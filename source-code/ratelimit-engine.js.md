/\*\*  
 \* RateLimit Pro \- Core Algorithmic Simulation & Rate Limiting Engine  
 \* Pure ES6+, Zero External Dependencies  
 \*/

class RateLimitEngine {  
  constructor() {  
    this.algorithms \= {  
      TOKEN\_BUCKET: 'token\_bucket',  
      LEAKY\_BUCKET: 'leaky\_bucket',  
      FIXED\_WINDOW: 'fixed\_window',  
      SLIDING\_WINDOW\_COUNTER: 'sliding\_window\_counter',  
      SLIDING\_WINDOW\_LOG: 'sliding\_window\_log'  
    };  
  }

  /\*\*  
   \* Evaluates a Token Bucket rate limit state.  
   \* @param {Object} config \- { capacity, refillRate, cost }  
   \* @param {Object} state \- { tokens, lastRefillTimestamp }  
   \* @param {number} currentTimestamp \- in milliseconds or seconds (consistent)  
   \*/  
  evaluateTokenBucket(config, state, currentTimestamp \= Date.now() / 1000\) {  
    const capacity \= Math.max(1, Number(config.capacity) || 10);  
    const refillRate \= Math.max(0.001, Number(config.refillRate) || 1); // tokens per second  
    const cost \= Math.max(1, Number(config.cost) || 1);

    let currentTokens \= state && typeof state.tokens \=== 'number' ? state.tokens : capacity;  
    let lastRefill \= state && typeof state.lastRefillTimestamp \=== 'number' ? state.lastRefillTimestamp : currentTimestamp;

    // Elapsed time in seconds  
    const elapsed \= Math.max(0, currentTimestamp \- lastRefill);  
    const refilledTokens \= elapsed \* refillRate;  
    currentTokens \= Math.min(capacity, currentTokens \+ refilledTokens);

    let allowed \= false;  
    let retryAfter \= 0;  
    let remaining \= 0;

    if (currentTokens \>= cost) {  
      allowed \= true;  
      currentTokens \-= cost;  
      remaining \= Math.floor(currentTokens);  
    } else {  
      allowed \= false;  
      const deficit \= cost \- currentTokens;  
      retryAfter \= Math.ceil(deficit / refillRate);  
      remaining \= Math.floor(currentTokens);  
    }

    const resetSeconds \= Math.ceil((capacity \- currentTokens) / refillRate);

    return {  
      allowed,  
      algorithm: 'token\_bucket',  
      limit: capacity,  
      remaining: Math.max(0, remaining),  
      reset: Math.max(0, resetSeconds),  
      retryAfter: allowed ? 0 : Math.max(1, retryAfter),  
      newState: {  
        tokens: currentTokens,  
        lastRefillTimestamp: currentTimestamp  
      }  
    };  
  }

  /\*\*  
   \* Evaluates a Leaky Bucket rate limit state (Traffic Shaper / Queue buffer).  
   \* @param {Object} config \- { capacity, leakRate, cost }  
   \* @param {Object} state \- { waterLevel, lastLeakTimestamp }  
   \*/  
  evaluateLeakyBucket(config, state, currentTimestamp \= Date.now() / 1000\) {  
    const capacity \= Math.max(1, Number(config.capacity) || 10);  
    const leakRate \= Math.max(0.001, Number(config.leakRate) || 1); // leak per second  
    const cost \= Math.max(1, Number(config.cost) || 1);

    let waterLevel \= state && typeof state.waterLevel \=== 'number' ? state.waterLevel : 0;  
    let lastLeak \= state && typeof state.lastLeakTimestamp \=== 'number' ? state.lastLeakTimestamp : currentTimestamp;

    const elapsed \= Math.max(0, currentTimestamp \- lastLeak);  
    const leaked \= elapsed \* leakRate;  
    waterLevel \= Math.max(0, waterLevel \- leaked);

    let allowed \= false;  
    let retryAfter \= 0;

    if (waterLevel \+ cost \<= capacity) {  
      allowed \= true;  
      waterLevel \+= cost;  
    } else {  
      allowed \= false;  
      const overflow \= (waterLevel \+ cost) \- capacity;  
      retryAfter \= Math.ceil(overflow / leakRate);  
    }

    const remaining \= Math.max(0, Math.floor(capacity \- waterLevel));  
    const resetSeconds \= Math.ceil(waterLevel / leakRate);

    return {  
      allowed,  
      algorithm: 'leaky\_bucket',  
      limit: capacity,  
      remaining,  
      reset: resetSeconds,  
      retryAfter: allowed ? 0 : Math.max(1, retryAfter),  
      newState: {  
        waterLevel,  
        lastLeakTimestamp: currentTimestamp  
      }  
    };  
  }

  /\*\*  
   \* Evaluates Fixed Window Counter rate limit state.  
   \* @param {Object} config \- { limit, windowSeconds }  
   \* @param {Object} state \- { windowIndex, count }  
   \*/  
  evaluateFixedWindow(config, state, currentTimestamp \= Date.now() / 1000\) {  
    const limit \= Math.max(1, Number(config.limit) || 10);  
    const windowSeconds \= Math.max(1, Number(config.windowSeconds) || 60);

    const currentWindowIndex \= Math.floor(currentTimestamp / windowSeconds);  
    const windowStart \= currentWindowIndex \* windowSeconds;  
    const windowEnd \= windowStart \+ windowSeconds;  
    const resetSeconds \= Math.max(1, Math.ceil(windowEnd \- currentTimestamp));

    let count \= 0;  
    if (state && state.windowIndex \=== currentWindowIndex) {  
      count \= state.count;  
    }

    let allowed \= false;  
    if (count \< limit) {  
      allowed \= true;  
      count \+= 1;  
    } else {  
      allowed \= false;  
    }

    const remaining \= Math.max(0, limit \- count);

    return {  
      allowed,  
      algorithm: 'fixed\_window',  
      limit,  
      remaining,  
      reset: resetSeconds,  
      retryAfter: allowed ? 0 : resetSeconds,  
      newState: {  
        windowIndex: currentWindowIndex,  
        count  
      }  
    };  
  }

  /\*\*  
   \* Evaluates Sliding Window Counter (Approximation via weighted previous \+ current window).  
   \* @param {Object} config \- { limit, windowSeconds }  
   \* @param {Object} state \- { currentWindowIndex, currentCount, previousCount }  
   \*/  
  evaluateSlidingWindowCounter(config, state, currentTimestamp \= Date.now() / 1000\) {  
    const limit \= Math.max(1, Number(config.limit) || 10);  
    const windowSeconds \= Math.max(1, Number(config.windowSeconds) || 60);

    const currentWindowIndex \= Math.floor(currentTimestamp / windowSeconds);  
    const windowStart \= currentWindowIndex \* windowSeconds;  
    const timeIntoCurrentWindow \= currentTimestamp \- windowStart;  
    const progressInWindow \= Math.min(1, Math.max(0, timeIntoCurrentWindow / windowSeconds));

    let currentCount \= 0;  
    let previousCount \= 0;

    if (state) {  
      if (state.currentWindowIndex \=== currentWindowIndex) {  
        currentCount \= state.currentCount || 0;  
        previousCount \= state.previousCount || 0;  
      } else if (state.currentWindowIndex \=== currentWindowIndex \- 1\) {  
        previousCount \= state.currentCount || 0;  
        currentCount \= 0;  
      } else {  
        currentCount \= 0;  
        previousCount \= 0;  
      }  
    }

    // Weighted count calculation  
    const weightedPrevious \= previousCount \* (1 \- progressInWindow);  
    const estimatedCount \= currentCount \+ weightedPrevious;

    let allowed \= false;  
    if (estimatedCount \< limit) {  
      allowed \= true;  
      currentCount \+= 1;  
    } else {  
      allowed \= false;  
    }

    const newEstimated \= currentCount \+ (previousCount \* (1 \- progressInWindow));  
    const remaining \= Math.max(0, Math.floor(limit \- newEstimated));  
    const resetSeconds \= Math.max(1, Math.ceil(windowSeconds \- timeIntoCurrentWindow));

    return {  
      allowed,  
      algorithm: 'sliding\_window\_counter',  
      limit,  
      remaining,  
      reset: resetSeconds,  
      retryAfter: allowed ? 0 : resetSeconds,  
      newState: {  
        currentWindowIndex,  
        currentCount,  
        previousCount  
      }  
    };  
  }

  /\*\*  
   \* Evaluates Sliding Window Log (Exact sliding window via timestamp log).  
   \* @param {Object} config \- { limit, windowSeconds }  
   \* @param {Object} state \- { log: Array\<number\> }  
   \*/  
  evaluateSlidingWindowLog(config, state, currentTimestamp \= Date.now() / 1000\) {  
    const limit \= Math.max(1, Number(config.limit) || 10);  
    const windowSeconds \= Math.max(1, Number(config.windowSeconds) || 60);  
    const cutoff \= currentTimestamp \- windowSeconds;

    let log \= state && Array.isArray(state.log) ? state.log.filter(ts \=\> ts \> cutoff) : \[\];

    let allowed \= false;  
    let retryAfter \= 0;

    if (log.length \< limit) {  
      allowed \= true;  
      log.push(currentTimestamp);  
    } else {  
      allowed \= false;  
      const oldestInWindow \= log\[0\] || cutoff;  
      retryAfter \= Math.max(1, Math.ceil((oldestInWindow \+ windowSeconds) \- currentTimestamp));  
    }

    const remaining \= Math.max(0, limit \- log.length);  
    const oldest \= log.length \> 0 ? log\[0\] : currentTimestamp;  
    const resetSeconds \= Math.max(1, Math.ceil((oldest \+ windowSeconds) \- currentTimestamp));

    return {  
      allowed,  
      algorithm: 'sliding\_window\_log',  
      limit,  
      remaining,  
      reset: resetSeconds,  
      retryAfter: allowed ? 0 : retryAfter,  
      newState: {  
        log  
      }  
    };  
  }

  /\*\*  
   \* Formats standard IETF Draft RFC HTTP headers.  
   \*/  
  formatHttpHeaders(evalResult) {  
    const headers \= {  
      'RateLimit-Limit': String(evalResult.limit),  
      'RateLimit-Remaining': String(evalResult.remaining),  
      'RateLimit-Reset': String(evalResult.reset),  
      'RateLimit-Policy': \`\${evalResult.limit};w=\${evalResult.reset || 60}\`  
    };

    if (\!evalResult.allowed) {  
      headers\['Retry-After'\] \= String(evalResult.retryAfter || 1);  
    }

    return headers;  
  }

  /\*\*  
   \* Run simulation over an array of arrival timestamps.  
   \*/  
  simulateTraffic(algorithm, config, arrivalTimes) {  
    let state \= null;  
    const results \= \[\];

    for (const ts of arrivalTimes) {  
      let res;  
      switch (algorithm) {  
        case this.algorithms.TOKEN\_BUCKET:  
          res \= this.evaluateTokenBucket(config, state, ts);  
          break;  
        case this.algorithms.LEAKY\_BUCKET:  
          res \= this.evaluateLeakyBucket(config, state, ts);  
          break;  
        case this.algorithms.FIXED\_WINDOW:  
          res \= this.evaluateFixedWindow(config, state, ts);  
          break;  
        case this.algorithms.SLIDING\_WINDOW\_COUNTER:  
          res \= this.evaluateSlidingWindowCounter(config, state, ts);  
          break;  
        case this.algorithms.SLIDING\_WINDOW\_LOG:  
          res \= this.evaluateSlidingWindowLog(config, state, ts);  
          break;  
        default:  
          res \= this.evaluateTokenBucket(config, state, ts);  
      }  
      state \= res.newState;  
      results.push({  
        timestamp: ts,  
        allowed: res.allowed,  
        remaining: res.remaining,  
        reset: res.reset,  
        retryAfter: res.retryAfter  
      });  
    }

    const total \= results.length;  
    const accepted \= results.filter(r \=\> r.allowed).length;  
    const rejected \= total \- accepted;  
    const acceptanceRate \= total \> 0 ? ((accepted / total) \* 100).toFixed(1) : 0;

    return {  
      summary: {  
        totalRequests: total,  
        accepted,  
        rejected,  
        acceptanceRate: Number(acceptanceRate)  
      },  
      timeline: results  
    };  
  }  
}

if (typeof module \!== 'undefined' && module.exports) {  
  module.exports \= RateLimitEngine;  
}  
