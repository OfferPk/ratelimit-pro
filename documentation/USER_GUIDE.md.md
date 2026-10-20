# RateLimit Pro — User & Integration Guide

Welcome to **RateLimit Pro**. This guide covers visual studio usage, framework integration (Node.js, FastAPI, Next.js Edge, Go), and command-line automation.

---

## 1\. Quick Start with the Visual Studio

Open `source-code/index.html` in any modern web browser (Chrome, Edge, Safari, Firefox). No installation, build step, or server is required.

### 1.1 Selecting a Presets

Click any of the top preset buttons to load battle-tested configurations:

- **AI Inference API**: Token bucket (capacity 20, 2.0 refill/sec) protecting expensive OpenAI/Claude/Gemini proxy endpoints.  
- **FinTech Gateway**: 5 requests/sec rolling sliding window protecting Stripe, Adyen, or banking webhooks from replay attacks.  
- **E-Commerce Shield**: Fixed window (60 req/min) blocking aggressive catalog scrapers and scalper bots during flash sales.

### 1.2 Running Traffic Simulations

1. Adjust the **Burst Spikes** slider (e.g. 30 requests).  
2. Click **Fire Burst** to observe how the visual bucket meter drains in real-time.  
3. Observe the timeline bars transition from green (`200 OK`) to red (`429 Blocked`).  
4. Click **Auto Stream (3 RPS)** to view sustained equilibrium dynamics.  
5. Inspect the **Live HTTP Response Inspector** to verify that headers match the IETF Draft specifications.

---

## 2\. Framework Integration Walkthroughs

### 2.1 Node.js / Express.js

Copy the generated code from the **Node.js Express** tab:

import express from 'express';

import { rateLimiter } from './ratelimit-express.js';

const app \= express();

// Apply rate limiting globally or per route

app.use('/api/v1/generate', rateLimiter({

  capacity: 20,

  refillRate: 2.0,

  cost: 1

}));

app.post('/api/v1/generate', (req, res) \=\> {

  res.json({ message: 'Success' });

});

### 2.2 Python FastAPI

Copy from the **Python FastAPI** tab:

from fastapi import FastAPI

from ratelimit\_fastapi import RateLimitMiddleware

app \= FastAPI()

app.add\_middleware(

    RateLimitMiddleware,

    redis\_url="redis://localhost:6379",

    capacity=20,

    refill\_rate=2.0

)

### 2.3 Next.js Edge Middleware

Save to `middleware.ts` in your Next.js project root. Uses Upstash Redis REST API to avoid Node.js TCP cold starts on Cloudflare Workers / Vercel Edge.

---

## 3\. Command-Line Interface (`ratelimit-cli.py`)

### 3.1 Simulating Traffic from the Terminal

python3 ratelimit-cli.py simulate \--algo token\_bucket \--capacity 20 \--rate 2.0 \--requests 100 \--burst 25

### 3.2 Exporting Atomic Redis Lua Scripts

python3 ratelimit-cli.py export-lua \--algo token\_bucket \--capacity 50 \--rate 5.0 \-o ratelimit.lua

### 3.3 Validating Configurations in CI/CD

python3 ratelimit-cli.py validate presets/public-ai-api.json

Exit code `0` indicates compliance; exit code `1` prints actionable schema errors to `stderr`.