/\*\*  
 \* RateLimit Pro \- Application Controller & Visual Orchestrator  
 \*/

document.addEventListener('DOMContentLoaded', () \=\> {  
  const engine \= new RateLimitEngine();

  // State  
  let currentAlgorithm \= 'token\_bucket';  
  let currentState \= null;  
  let history \= \[\];  
  let autoStreamInterval \= null;  
  let activeTab \= 'redis-lua';

  // Config DOM Elements  
  const selectAlgorithm \= document.getElementById('select-algorithm');  
  const inputCapacity \= document.getElementById('input-capacity');  
  const valCapacity \= document.getElementById('val-capacity');  
  const inputRefillRate \= document.getElementById('input-refill-rate');  
  const valRefillRate \= document.getElementById('val-refill-rate');  
  const labelRefillRate \= document.getElementById('label-refill-rate');  
  const inputWindow \= document.getElementById('input-window');  
  const valWindow \= document.getElementById('val-window');  
  const inputCost \= document.getElementById('input-cost');  
  const valCost \= document.getElementById('val-cost');  
  const inputBurstCount \= document.getElementById('input-burst-count');  
  const valBurstCount \= document.getElementById('val-burst-count');

  // Parameter Groups  
  const paramCapacityGroup \= document.getElementById('param-capacity-group');  
  const paramRateGroup \= document.getElementById('param-rate-group');  
  const paramWindowGroup \= document.getElementById('param-window-group');  
  const paramCostGroup \= document.getElementById('param-cost-group');

  // KPI Elements  
  const kpiTotal \= document.getElementById('kpi-total');  
  const kpiAccepted \= document.getElementById('kpi-accepted');  
  const kpiRejected \= document.getElementById('kpi-rejected');  
  const kpiRate \= document.getElementById('kpi-rate');

  // Visualizer Elements  
  const bucketStatusText \= document.getElementById('bucket-status-text');  
  const bucketFillBar \= document.getElementById('bucket-fill-bar');  
  const markerCapacity \= document.getElementById('marker-capacity');  
  const timelineContainer \= document.getElementById('timeline-container');

  // HTTP Inspector Elements  
  const httpStatusBadge \= document.getElementById('http-status-badge');  
  const headerLimit \= document.getElementById('header-limit');  
  const headerRemaining \= document.getElementById('header-remaining');  
  const headerReset \= document.getElementById('header-reset');  
  const headerPolicy \= document.getElementById('header-policy');  
  const rowRetryAfter \= document.getElementById('row-retry-after');  
  const headerRetryAfter \= document.getElementById('header-retry-after');

  // Code Tab Elements  
  const codeSnippetDisplay \= document.getElementById('code-snippet-display');  
  const btnCopyCode \= document.getElementById('btn-copy-code');  
  const btnDownloadSnippet \= document.getElementById('btn-download-snippet');  
  const tabBtns \= document.querySelectorAll('.tab-btn');

  // Preset Buttons  
  const presetButtons \= document.querySelectorAll('.btn-preset');

  // Presets Dictionary  
  const PRESETS \= {  
    'public-ai-api': {  
      algorithm: 'token\_bucket',  
      capacity: 20,  
      refillRate: 2.0,  
      cost: 1,  
      window: 60  
    },  
    'fintech-payment-gateway': {  
      algorithm: 'sliding\_window\_counter',  
      limit: 5,  
      window: 1,  
      cost: 1  
    },  
    'ecommerce-flash-sale': {  
      algorithm: 'fixed\_window',  
      limit: 60,  
      window: 60,  
      cost: 1  
    }  
  };

  function getConfig() {  
    return {  
      capacity: Number(inputCapacity.value) || 20,  
      refillRate: Number(inputRefillRate.value) || 2.0,  
      leakRate: Number(inputRefillRate.value) || 2.0,  
      limit: Number(inputCapacity.value) || 20,  
      windowSeconds: Number(inputWindow.value) || 60,  
      cost: Number(inputCost.value) || 1  
    };  
  }

  function updateAlgorithmFormVisibility() {  
    const algo \= selectAlgorithm.value;  
    currentAlgorithm \= algo;

    if (algo \=== 'token\_bucket') {  
      paramCapacityGroup.style.display \= 'flex';  
      paramRateGroup.style.display \= 'flex';  
      paramWindowGroup.style.display \= 'none';  
      paramCostGroup.style.display \= 'flex';  
      labelRefillRate.textContent \= 'Refill Rate (tokens/sec)';  
      markerCapacity.textContent \= \`Capacity: \${inputCapacity.value}\`;  
    } else if (algo \=== 'leaky\_bucket') {  
      paramCapacityGroup.style.display \= 'flex';  
      paramRateGroup.style.display \= 'flex';  
      paramWindowGroup.style.display \= 'none';  
      paramCostGroup.style.display \= 'flex';  
      labelRefillRate.textContent \= 'Leak Rate (units/sec)';  
      markerCapacity.textContent \= \`Buffer Size: \${inputCapacity.value}\`;  
    } else if (algo \=== 'fixed\_window' || algo \=== 'sliding\_window\_counter' || algo \=== 'sliding\_window\_log') {  
      paramCapacityGroup.style.display \= 'flex';  
      paramRateGroup.style.display \= 'none';  
      paramWindowGroup.style.display \= 'flex';  
      paramCostGroup.style.display \= 'none';  
      document.querySelector('\#param-capacity-group label span').textContent \= 'Window Max Requests';  
      markerCapacity.textContent \= \`Limit: \${inputCapacity.value}\`;  
    }  
    updateCodeDisplay();  
  }

  function resetState() {  
    currentState \= null;  
    history \= \[\];  
    renderDashboard(null);  
  }

  function handleRequest(now \= Date.now() / 1000\) {  
    const config \= getConfig();  
    let result;

    switch (currentAlgorithm) {  
      case 'token\_bucket':  
        result \= engine.evaluateTokenBucket(config, currentState, now);  
        break;  
      case 'leaky\_bucket':  
        result \= engine.evaluateLeakyBucket(config, currentState, now);  
        break;  
      case 'fixed\_window':  
        result \= engine.evaluateFixedWindow(config, currentState, now);  
        break;  
      case 'sliding\_window\_counter':  
        result \= engine.evaluateSlidingWindowCounter(config, currentState, now);  
        break;  
      case 'sliding\_window\_log':  
        result \= engine.evaluateSlidingWindowLog(config, currentState, now);  
        break;  
      default:  
        result \= engine.evaluateTokenBucket(config, currentState, now);  
    }

    currentState \= result.newState;  
    history.push({  
      timestamp: now,  
      allowed: result.allowed,  
      remaining: result.remaining,  
      limit: result.limit,  
      reset: result.reset,  
      retryAfter: result.retryAfter  
    });

    if (history.length \> 50\) {  
      history.shift();  
    }

    renderDashboard(result);  
  }

  function renderDashboard(latestResult) {  
    // 1\. KPI calculations  
    const total \= history.length;  
    const accepted \= history.filter(h \=\> h.allowed).length;  
    const rejected \= total \- accepted;  
    const rate \= total \> 0 ? Math.round((accepted / total) \* 100\) : 100;

    kpiTotal.textContent \= total;  
    kpiAccepted.textContent \= accepted;  
    kpiRejected.textContent \= rejected;  
    kpiRate.textContent \= \`\${rate}%\`;

    // 2\. Bucket visualizer  
    const config \= getConfig();  
    let fillPct \= 100;  
    let label \= '';

    if (currentAlgorithm \=== 'token\_bucket') {  
      const tokens \= currentState ? currentState.tokens : config.capacity;  
      fillPct \= Math.max(0, Math.min(100, (tokens / config.capacity) \* 100));  
      label \= \`\${tokens.toFixed(1)} / \${config.capacity} tokens\`;  
    } else if (currentAlgorithm \=== 'leaky\_bucket') {  
      const water \= currentState ? currentState.waterLevel : 0;  
      fillPct \= Math.max(0, Math.min(100, (1 \- (water / config.capacity)) \* 100));  
      label \= \`\${Math.floor(config.capacity \- water)} available buffer\`;  
    } else {  
      const remaining \= latestResult ? latestResult.remaining : config.limit;  
      fillPct \= Math.max(0, Math.min(100, (remaining / config.limit) \* 100));  
      label \= \`\${remaining} / \${config.limit} remaining in window\`;  
    }

    bucketFillBar.style.width \= \`\${fillPct}%\`;  
    bucketStatusText.textContent \= label;  
    if (fillPct \< 25\) {  
      bucketFillBar.classList.add('low');  
    } else {  
      bucketFillBar.classList.remove('low');  
    }

    // 3\. Render Timeline  
    timelineContainer.innerHTML \= '';  
    history.forEach(item \=\> {  
      const bar \= document.createElement('div');  
      bar.className \= \`timeline-bar \${item.allowed ? 'accepted' : 'rejected'}\`;  
      bar.style.height \= item.allowed ? '75%' : '100%';  
      bar.title \= \`\${item.allowed ? '200 OK' : '429 Blocked'} | Remaining: \${item.remaining}\`;  
      timelineContainer.appendChild(bar);  
    });  
    timelineContainer.scrollLeft \= timelineContainer.scrollWidth;

    // 4\. Update HTTP Inspector  
    if (latestResult) {  
      const headers \= engine.formatHttpHeaders(latestResult);  
      if (latestResult.allowed) {  
        httpStatusBadge.className \= 'status-badge status-200';  
        httpStatusBadge.textContent \= '200 OK';  
        rowRetryAfter.style.display \= 'none';  
      } else {  
        httpStatusBadge.className \= 'status-badge status-429';  
        httpStatusBadge.textContent \= '429 Too Many Requests';  
        rowRetryAfter.style.display \= 'table-row';  
        headerRetryAfter.textContent \= \`\${headers\['Retry-After'\]} seconds\`;  
      }

      headerLimit.textContent \= headers\['RateLimit-Limit'\];  
      headerRemaining.textContent \= headers\['RateLimit-Remaining'\];  
      headerReset.textContent \= \`\${headers\['RateLimit-Reset'\]}s\`;  
      headerPolicy.textContent \= headers\['RateLimit-Policy'\];  
    }  
  }

  function updateCodeDisplay() {  
    const config \= getConfig();  
    let code \= '';

    switch (activeTab) {  
      case 'redis-lua':  
        if (currentAlgorithm \=== 'sliding\_window\_counter') {  
          code \= CodeGenEngine.generateRedisLuaSlidingWindow(config);  
        } else {  
          code \= CodeGenEngine.generateRedisLuaTokenBucket(config);  
        }  
        break;  
      case 'express':  
        code \= CodeGenEngine.generateExpressMiddleware(config, currentAlgorithm);  
        break;  
      case 'fastapi':  
        code \= CodeGenEngine.generateFastApiMiddleware(config);  
        break;  
      case 'nextjs':  
        code \= CodeGenEngine.generateNextEdgeMiddleware(config);  
        break;  
      case 'golang':  
        code \= CodeGenEngine.generateGoMiddleware(config);  
        break;  
      default:  
        code \= CodeGenEngine.generateRedisLuaTokenBucket(config);  
    }

    codeSnippetDisplay.textContent \= code;  
  }

  // Event Listeners  
  selectAlgorithm.addEventListener('change', () \=\> {  
    updateAlgorithmFormVisibility();  
    resetState();  
  });

  inputCapacity.addEventListener('input', (e) \=\> {  
    valCapacity.textContent \= e.target.value;  
    markerCapacity.textContent \= \`Capacity: \${e.target.value}\`;  
    updateCodeDisplay();  
  });

  inputRefillRate.addEventListener('input', (e) \=\> {  
    valRefillRate.textContent \= Number(e.target.value).toFixed(1);  
    updateCodeDisplay();  
  });

  inputWindow.addEventListener('input', (e) \=\> {  
    valWindow.textContent \= e.target.value;  
    updateCodeDisplay();  
  });

  inputCost.addEventListener('input', (e) \=\> {  
    valCost.textContent \= e.target.value;  
    updateCodeDisplay();  
  });

  inputBurstCount.addEventListener('input', (e) \=\> {  
    valBurstCount.textContent \= e.target.value;  
  });

  // Buttons  
  document.getElementById('btn-send-single').addEventListener('click', () \=\> {  
    handleRequest();  
  });

  document.getElementById('btn-send-burst').addEventListener('click', () \=\> {  
    const count \= Number(inputBurstCount.value) || 30;  
    const now \= Date.now() / 1000;  
    for (let i \= 0; i \< count; i++) {  
      handleRequest(now \+ (i \* 0.02)); // 20ms burst step  
    }  
  });

  const btnAutoStream \= document.getElementById('btn-auto-stream');  
  btnAutoStream.addEventListener('click', () \=\> {  
    if (autoStreamInterval) {  
      clearInterval(autoStreamInterval);  
      autoStreamInterval \= null;  
      btnAutoStream.innerHTML \= \`\<svg width="16" height="16" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2"\>\<polygon points="5 3 19 12 5 21 5 3"/\>\</svg\> Auto Stream (3 RPS)\`;  
      btnAutoStream.classList.remove('btn-danger');  
      btnAutoStream.classList.add('btn-secondary');  
    } else {  
      autoStreamInterval \= setInterval(() \=\> {  
        handleRequest();  
      }, 333); // \~3 req/sec  
      btnAutoStream.innerHTML \= \`\<svg width="16" height="16" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2"\>\<rect x="6" y="4" width="4" height="16"/\>\<rect x="14" y="4" width="4" height="16"/\>\</svg\> Stop Streaming\`;  
      btnAutoStream.classList.remove('btn-secondary');  
      btnAutoStream.classList.add('btn-danger');  
    }  
  });

  document.getElementById('btn-reset-sim').addEventListener('click', () \=\> {  
    if (autoStreamInterval) {  
      clearInterval(autoStreamInterval);  
      autoStreamInterval \= null;  
      btnAutoStream.innerHTML \= \`\<svg width="16" height="16" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2"\>\<polygon points="5 3 19 12 5 21 5 3"/\>\</svg\> Auto Stream (3 RPS)\`;  
      btnAutoStream.classList.remove('btn-danger');  
      btnAutoStream.classList.add('btn-secondary');  
    }  
    resetState();  
  });

  // Tab Navigation  
  tabBtns.forEach(btn \=\> {  
    btn.addEventListener('click', () \=\> {  
      tabBtns.forEach(b \=\> b.classList.remove('active'));  
      btn.classList.add('active');  
      activeTab \= btn.getAttribute('data-tab');  
      updateCodeDisplay();  
    });  
  });

  // Copy Code  
  btnCopyCode.addEventListener('click', () \=\> {  
    navigator.clipboard.writeText(codeSnippetDisplay.textContent).then(() \=\> {  
      const originalText \= btnCopyCode.innerHTML;  
      btnCopyCode.innerHTML \= \`\<svg width="15" height="15" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2"\>\<polyline points="20 6 9 17 4 12"/\>\</svg\> Copied\!\`;  
      setTimeout(() \=\> {  
        btnCopyCode.innerHTML \= originalText;  
      }, 2000);  
    });  
  });

  // Download Code  
  btnDownloadSnippet.addEventListener('click', () \=\> {  
    let filename \= 'ratelimit.lua';  
    let mime \= 'text/plain';

    if (activeTab \=== 'express') filename \= 'ratelimit-express.js';  
    else if (activeTab \=== 'fastapi') filename \= 'ratelimit\_fastapi.py';  
    else if (activeTab \=== 'nextjs') filename \= 'middleware.ts';  
    else if (activeTab \=== 'golang') filename \= 'ratelimit.go';

    const blob \= new Blob(\[codeSnippetDisplay.textContent\], { type: mime });  
    const url \= URL.createObjectURL(blob);  
    const a \= document.createElement('a');  
    a.href \= url;  
    a.download \= filename;  
    a.click();  
    URL.revokeObjectURL(url);  
  });

  // Preset Buttons Click  
  presetButtons.forEach(btn \=\> {  
    btn.addEventListener('click', () \=\> {  
      presetButtons.forEach(b \=\> b.classList.remove('active'));  
      btn.classList.add('active');  
      const presetKey \= btn.getAttribute('data-preset');  
      const p \= PRESETS\[presetKey\];  
      if (p) {  
        selectAlgorithm.value \= p.algorithm;  
        if (p.capacity \!== undefined) {  
          inputCapacity.value \= p.capacity;  
          valCapacity.textContent \= p.capacity;  
        } else if (p.limit \!== undefined) {  
          inputCapacity.value \= p.limit;  
          valCapacity.textContent \= p.limit;  
        }

        if (p.refillRate \!== undefined) {  
          inputRefillRate.value \= p.refillRate;  
          valRefillRate.textContent \= Number(p.refillRate).toFixed(1);  
        }

        if (p.window \!== undefined) {  
          inputWindow.value \= p.window;  
          valWindow.textContent \= p.window;  
        }

        updateAlgorithmFormVisibility();  
        resetState();  
      }  
    });  
  });

  // Initial setup  
  updateAlgorithmFormVisibility();  
  resetState();  
});  
