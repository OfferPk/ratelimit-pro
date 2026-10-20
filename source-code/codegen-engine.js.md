/\*\*  
 \* RateLimit Pro \- Production Code & Redis Lua Synthesizer  
 \* Generates zero-dependency, copy-pasteable production implementations.  
 \*/

class CodeGenEngine {  
  /\*\*  
   \* Generates atomic Redis Lua script for Token Bucket algorithm.  
   \*/  
  static generateRedisLuaTokenBucket(config) {  
    const capacity \= Number(config.capacity) || 10;  
    const refillRate \= Number(config.refillRate) || 1;  
    const cost \= Number(config.cost) || 1;

    return \`-- RateLimit Pro: Atomic Redis Lua Token Bucket  
\-- KEYS\[1\] : Rate limit bucket key (e.g., 'ratelimit:user\_123')  
\-- ARGV\[1\] : Current UNIX timestamp in seconds (floating point or integer)  
\-- ARGV\[2\] : Token cost for this operation (default: \${cost})  
\-- ARGV\[3\] : Bucket capacity (default: \${capacity})  
\-- ARGV\[4\] : Refill rate tokens/sec (default: \${refillRate})

local key \= KEYS\[1\]  
local now \= tonumber(ARGV\[1\])  
local cost \= tonumber(ARGV\[2\]) or \${cost}  
local capacity \= tonumber(ARGV\[3\]) or \${capacity}  
local refill\_rate \= tonumber(ARGV\[4\]) or \${refillRate}

\-- Calculate TTL to automatically purge inactive keys  
local ttl \= math.ceil(capacity / refill\_rate) \* 2

local data \= redis.call("HMGET", key, "tokens", "last\_updated")  
local tokens \= tonumber(data\[1\])  
local last\_updated \= tonumber(data\[2\])

if tokens \== nil then  
    tokens \= capacity  
    last\_updated \= now  
else  
    local elapsed \= math.max(0, now \- last\_updated)  
    tokens \= math.min(capacity, tokens \+ (elapsed \* refill\_rate))  
    last\_updated \= now  
end

local allowed \= 0  
local retry\_after \= 0

if tokens \>= cost then  
    allowed \= 1  
    tokens \= tokens \- cost  
else  
    allowed \= 0  
    local deficit \= cost \- tokens  
    retry\_after \= math.ceil(deficit / refill\_rate)  
end

redis.call("HMSET", key, "tokens", tokens, "last\_updated", last\_updated)  
redis.call("EXPIRE", key, ttl)

local remaining \= math.floor(tokens)  
local reset\_in \= math.ceil((capacity \- tokens) / refill\_rate)

\-- Return: \[allowed, remaining, reset\_in, retry\_after\]  
return { allowed, remaining, reset\_in, retry\_after }  
\`;  
  }

  /\*\*  
   \* Generates atomic Redis Lua script for Sliding Window Counter.  
   \*/  
  static generateRedisLuaSlidingWindow(config) {  
    const limit \= Number(config.limit) || 100;  
    const windowSeconds \= Number(config.windowSeconds) || 60;

    return \`-- RateLimit Pro: Atomic Redis Lua Sliding Window Counter  
\-- KEYS\[1\] : Key prefix for user rate limit (e.g., 'ratelimit:ip\_1.2.3.4')  
\-- ARGV\[1\] : Current UNIX timestamp in seconds  
\-- ARGV\[2\] : Window size in seconds (default: \${windowSeconds})  
\-- ARGV\[3\] : Max limit per window (default: \${limit})

local key\_prefix \= KEYS\[1\]  
local now \= tonumber(ARGV\[1\])  
local window \= tonumber(ARGV\[2\]) or \${windowSeconds}  
local limit \= tonumber(ARGV\[3\]) or \${limit}

local current\_window \= math.floor(now / window)  
local previous\_window \= current\_window \- 1

local current\_key \= key\_prefix .. ":" .. current\_window  
local previous\_key \= key\_prefix .. ":" .. previous\_window

local current\_count \= tonumber(redis.call("GET", current\_key) or "0")  
local previous\_count \= tonumber(redis.call("GET", previous\_key) or "0")

local time\_in\_window \= now \- (current\_window \* window)  
local progress \= time\_in\_window / window  
local estimated\_count \= current\_count \+ (previous\_count \* (1 \- progress))

local allowed \= 0  
local retry\_after \= 0

if estimated\_count \< limit then  
    allowed \= 1  
    redis.call("INCR", current\_key)  
    redis.call("EXPIRE", current\_key, window \* 2\)  
    current\_count \= current\_count \+ 1  
else  
    allowed \= 0  
    retry\_after \= math.ceil(window \- time\_in\_window)  
end

local remaining \= math.max(0, math.floor(limit \- (current\_count \+ (previous\_count \* (1 \- progress)))))  
local reset\_in \= math.max(1, math.ceil(window \- time\_in\_window))

return { allowed, remaining, reset\_in, retry\_after }  
\`;  
  }

  /\*\*  
   \* Generates Node.js Express middleware.  
   \*/  
  static generateExpressMiddleware(config, algorithm \= 'token\_bucket') {  
    return \`// Express.js Rate Limiting Middleware (RateLimit Pro)  
// Compatible with ioredis / node-redis  
import Redis from 'ioredis';

const redis \= new Redis(process.env.REDIS\_URL || 'redis://localhost:6379');

const LUA\_TOKEN\_BUCKET \= \\\`\${CodeGenEngine.generateRedisLuaTokenBucket(config).replace(/\`/g, '\\\\\`')}\\\`;

export function rateLimiter(options \= {}) {  
  const capacity \= options.capacity || \${config.capacity || 20};  
  const refillRate \= options.refillRate || \${config.refillRate || 1};  
  const cost \= options.cost || \${config.cost || 1};

  return async (req, res, next) \=\> {  
    try {  
      const clientIp \= req.headers\['x-forwarded-for'\] || req.socket.remoteAddress || '127.0.0.1';  
      const key \= \\\`ratelimit:\\\${req.user?.id || clientIp}\\\`;  
      const now \= Date.now() / 1000;

      const \[allowed, remaining, resetIn, retryAfter\] \= await redis.eval(  
        LUA\_TOKEN\_BUCKET,  
        1,  
        key,  
        now,  
        cost,  
        capacity,  
        refillRate  
      );

      res.setHeader('RateLimit-Limit', capacity);  
      res.setHeader('RateLimit-Remaining', remaining);  
      res.setHeader('RateLimit-Reset', resetIn);  
      res.setHeader('RateLimit-Policy', \\\`\\\${capacity};w=\\\${resetIn}\\\`);

      if (allowed \=== 0\) {  
        res.setHeader('Retry-After', retryAfter);  
        return res.status(429).json({  
          error: 'rate\_limit\_exceeded',  
          message: 'Too many requests. Please slow down.',  
          limit: capacity,  
          remaining: 0,  
          reset\_in\_seconds: resetIn,  
          retry\_after: retryAfter  
        });  
      }

      next();  
    } catch (err) {  
      console.error('Rate limiter failure, failing open:', err);  
      next(); // Fail open to protect availability  
    }  
  };  
}  
\`;  
  }

  /\*\*  
   \* Generates Python FastAPI middleware.  
   \*/  
  static generateFastApiMiddleware(config) {  
    const capacity \= config.capacity || 20;  
    const refillRate \= config.refillRate || 1;  
    const cost \= config.cost || 1;

    return \`\# Python FastAPI Rate Limiting Middleware (RateLimit Pro)  
import time  
import math  
from fastapi import Request, Response, HTTPException  
from starlette.middleware.base import BaseHTTPMiddleware  
import redis.asyncio as aioredis

LUA\_TOKEN\_BUCKET \= """\${CodeGenEngine.generateRedisLuaTokenBucket(config)}"""

class RateLimitMiddleware(BaseHTTPMiddleware):  
    def \_\_init\_\_(self, app, redis\_url: str \= "redis://localhost:6379", capacity: int \= \${capacity}, refill\_rate: float \= \${refillRate}):  
        super().\_\_init\_\_(app)  
        self.redis \= aioredis.from\_url(redis\_url, decode\_responses=False)  
        self.capacity \= capacity  
        self.refill\_rate \= refill\_rate  
        self.lua\_script \= self.redis.register\_script(LUA\_TOKEN\_BUCKET)

    async def dispatch(self, request: Request, call\_next):  
        client\_ip \= request.headers.get("x-forwarded-for") or request.client.host  
        key \= f"ratelimit:{client\_ip}"  
        now \= time.time()

        try:  
            allowed, remaining, reset\_in, retry\_after \= await self.lua\_script(  
                keys=\[key\],  
                args=\[str(now), "1", str(self.capacity), str(self.refill\_rate)\]  
            )  
        except Exception as e:  
            \# Fail open strategy  
            return await call\_next(request)

        headers \= {  
            "RateLimit-Limit": str(self.capacity),  
            "RateLimit-Remaining": str(remaining),  
            "RateLimit-Reset": str(reset\_in),  
            "RateLimit-Policy": f"{self.capacity};w={reset\_in}",  
        }

        if allowed \== 0:  
            headers\["Retry-After"\] \= str(retry\_after)  
            return Response(  
                content='{"error":"rate\_limit\_exceeded","message":"Too many requests"}',  
                status\_code=429,  
                media\_type="application/json",  
                headers=headers  
            )

        response \= await call\_next(request)  
        for k, v in headers.items():  
            response.headers\[k\] \= v  
        return response  
\`;  
  }

  /\*\*  
   \* Generates Next.js Edge Middleware / Cloudflare Workers.  
   \*/  
  static generateNextEdgeMiddleware(config) {  
    const limit \= config.limit || config.capacity || 20;  
    const window \= config.windowSeconds || 60;

    return \`// Next.js Edge Middleware (Edge Runtime / Cloudflare Workers)  
// Uses Upstash Redis REST API for zero TCP cold starts  
import { NextResponse } from 'next/server';

const UPSTASH\_URL \= process.env.UPSTASH\_REDIS\_REST\_URL;  
const UPSTASH\_TOKEN \= process.env.UPSTASH\_REDIS\_REST\_TOKEN;

export async function middleware(req) {  
  const ip \= req.headers.get('x-forwarded-for') || req.ip || '127.0.0.1';  
  const key \= \\\`ratelimit:\\\${ip}\\\`;  
  const windowSeconds \= \${window};  
  const maxRequests \= \${limit};

  try {  
    // Atomic INCR \+ EXPIRE via Upstash REST pipeline  
    const res \= await fetch(\\\`\\\${UPSTASH\_URL}/pipeline\\\`, {  
      method: 'POST',  
      headers: { Authorization: \\\`Bearer \\\${UPSTASH\_TOKEN}\\\` },  
      body: JSON.stringify(\[  
        \['INCR', key\],  
        \['EXPIRE', key, windowSeconds, 'NX'\]  
      \])  
    });

    const \[incrResult\] \= await res.json();  
    const currentCount \= incrResult.result;

    const remaining \= Math.max(0, maxRequests \- currentCount);  
    const headers \= new Headers();  
    headers.set('RateLimit-Limit', String(maxRequests));  
    headers.set('RateLimit-Remaining', String(remaining));  
    headers.set('RateLimit-Reset', String(windowSeconds));

    if (currentCount \> maxRequests) {  
      headers.set('Retry-After', String(windowSeconds));  
      return new NextResponse(  
        JSON.stringify({ error: 'rate\_limit\_exceeded', limit: maxRequests, retry\_after: windowSeconds }),  
        { status: 429, headers }  
      );  
    }

    const nextResponse \= NextResponse.next();  
    nextResponse.headers.set('RateLimit-Limit', String(maxRequests));  
    nextResponse.headers.set('RateLimit-Remaining', String(remaining));  
    return nextResponse;  
  } catch (err) {  
    return NextResponse.next(); // Fail open  
  }  
}

export const config \= {  
  matcher: '/api/:path\*',  
};  
\`;  
  }

  /\*\*  
   \* Generates Golang middleware.  
   \*/  
  static generateGoMiddleware(config) {  
    const capacity \= config.capacity || 20;  
    const refillRate \= config.refillRate || 1;

    return \`// Golang Fiber / Gin / Chi Rate Limit Middleware  
package main

import (  
	"context"  
	"fmt"  
	"net/http"  
	"time"

	"github.com/redis/go-redis/v9"  
)

var ctx \= context.Background()

var tokenBucketLua \= redis.NewScript(\\\`  
\${CodeGenEngine.generateRedisLuaTokenBucket(config)}  
\\\`)

func RateLimitMiddleware(rdb \*redis.Client, capacity int, refillRate float64) func(http.Handler) http.Handler {  
	return func(next http.Handler) http.Handler {  
		return http.HandlerFunc(func(w http.ResponseWriter, r \*http.Request) {  
			ip := r.RemoteAddr  
			key := fmt.Sprintf("ratelimit:%s", ip)  
			now := float64(time.Now().Unix())

			res, err := tokenBucketLua.Run(ctx, rdb, \[\]string{key}, now, 1, capacity, refillRate).Slice()  
			if err \!= nil {  
				// Fail open  
				next.ServeHTTP(w, r)  
				return  
			}

			allowed := res\[0\].(int64)  
			remaining := res\[1\].(int64)  
			resetIn := res\[2\].(int64)  
			retryAfter := res\[3\].(int64)

			w.Header().Set("RateLimit-Limit", fmt.Sprintf("%d", capacity))  
			w.Header().Set("RateLimit-Remaining", fmt.Sprintf("%d", remaining))  
			w.Header().Set("RateLimit-Reset", fmt.Sprintf("%d", resetIn))

			if allowed \== 0 {  
				w.Header().Set("Retry-After", fmt.Sprintf("%d", retryAfter))  
				w.WriteHeader(http.StatusStatusTooManyRequests)  
				w.Write(\[\]byte(\\\`{"error":"rate\_limit\_exceeded","message":"Too many requests"}\\\`))  
				return  
			}

			next.ServeHTTP(w, r)  
		})  
	}  
}  
\`;  
  }  
}

if (typeof module \!== 'undefined' && module.exports) {  
  module.exports \= CodeGenEngine;  
}  
