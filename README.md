# RateLimit Pro (v1.0.0)

> Architectural Rate Limiting Simulator, Visual Debugger & Atomic Redis Code Synthesizer for Modern APIs.

---

## What is RateLimit Pro?

**RateLimit Pro** is a high-utility, zero-infrastructure developer studio designed to eliminate API rate limiting vulnerabilities, race conditions, and boundary spikes. It combines an interactive client-side visual simulation engine with production-ready atomic Redis Lua scripts, modern middleware generators (Node.js Express, Python FastAPI, Next.js Edge, Golang), and IETF Draft / RFC 6585 standard HTTP header formatters.

---

## Key Features

1. **5 Algorithmic Models**:  
     
   - **Token Bucket**: Handles bursty traffic with steady replenishment.  
   - **Leaky Bucket**: Smooths jagged traffic bursts into a steady output stream.  
   - **Fixed Window Counter**: Fast, low-overhead atomic Redis `INCR` counter.  
   - **Sliding Window Counter**: Memory-efficient (\$O(1)\$) rolling window using weighted interpolation to eliminate boundary spikes.  
   - **Sliding Window Log**: Exact timestamp history tracking using Redis Sorted Sets (`ZSET`).

   

2. **Live Visual Simulation Dashboard**:  
     
   - **Dynamic Bucket Meter**: Real-time visual progress bar tracking fill percentage, active token volume, and buffer headroom.  
   - **Interactive 50-Request Timeline**: Color-coded bar graph highlighting accepted requests (`200 OK`) and blocked requests (`429 Too Many Requests`).  
   - **Real-Time KPI Cards**: Total requests, accepted count, 429 blocked count, and overall acceptance rate.

   

3. **Production Code & Script Synthesizer**:  
     
   - **Atomic Redis Lua Script**: Eliminates TOCTOU race conditions via single-operation `EVALSHA` execution with automatic TTL cleanup.  
   - **Node.js Express**: Ready-to-mount middleware with automatic IP/Bearer token extraction.  
   - **Python FastAPI**: Starlette/ASGI middleware supporting async Redis connections.  
   - **Next.js Edge / Cloudflare Workers**: Low-latency edge middleware using Upstash Redis REST pipelines.  
   - **Golang**: High-throughput Go middleware using `go-redis`.

   

4. **IETF & RFC Compliance**:  
     
   - Generates standardized headers: `RateLimit-Limit`, `RateLimit-Remaining`, `RateLimit-Reset`, `RateLimit-Policy`.  
   - Injects RFC 6585 `Retry-After` backoff guidance and structured JSON error envelopes on HTTP 429 rejections.

   

5. **Standalone Python CLI Tool (`ratelimit-cli.py`)**:  
     
   - Traffic simulation with ASCII output directly in the terminal.  
   - Automated export of Lua scripts and middleware.  
   - Strict configuration validation for CI/CD pipelines (returns exit code `0` on pass, `1` on error).

---

## Directory Structure

ratelimit-pro/

├── source-code/

│   ├── index.html                 \# Complete Studio Interface

│   ├── ratelimit.css              \# Dark Glassmorphism Studio UI

│   ├── ratelimit-engine.js        \# Core Algorithmic Simulation Engine

│   ├── codegen-engine.js          \# Redis Lua & Framework Code Synthesizer

│   ├── app.js                     \# Visual Orchestrator & Event Controller

│   └── presets/

│       ├── public-ai-api.json           \# AI Inference API Preset

│       ├── fintech-payment-gateway.json \# FinTech Gateway Preset

│       └── ecommerce-flash-sale.json    \# E-Commerce Shield Preset

├── build/

│   ├── ratelimit-cli.py           \# Standalone Python CLI Simulator & Validator

│   └── ratelimit-pro-v1.0.0.zip   \# Distributable Product Archive

├── documentation/

│   ├── ARCHITECTURE.md            \# Mathematical Derivations & Concurrency Models

│   ├── USER\_GUIDE.md               \# User & Integration Guide

│   └── MONETIZATION\_AND\_MARKETING\_PLAYBOOK.md \# Commercial Strategy

├── screenshots/

│   ├── simulator-preview.svg      \# Vector Diagram of Bucket Meter & Timeline

│   └── studio-preview.svg         \# Studio Interface Preview

└── README.md                      \# Master Repository Documentation

---

## Quick Start

### Running the Web Studio

Double-click or open `source-code/index.html` in any browser. No build steps, Node packages, or backend servers are required.

### Running Automated Tests

python3 tests/test\_ratelimit.py

### Running the CLI Tool

python3 build/ratelimit-cli.py simulate \--preset source-code/presets/public-ai-api.json \--requests 60 \--burst 20

---

## Specifications & Compliance

- **Standards**: RFC 6585 (HTTP 429), IETF Draft HTTP RateLimit Headers.  
- **Atomicity**: Single-turn Redis Lua execution via `EVALSHA`.  
- **Latency**: Sub-0.05ms execution overhead in memory.  
- **Dependencies**: Zero external runtime dependencies.