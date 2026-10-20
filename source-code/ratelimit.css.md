/\* RateLimit Pro \- Modern Dark Studio UI \*/  
:root {  
  \--bg-primary: \#0a0e17;  
  \--bg-secondary: \#111827;  
  \--bg-card: rgba(17, 24, 39, 0.75);  
  \--border-color: rgba(255, 255, 255, 0.1);  
  \--border-focus: \#3b82f6;  
  \--text-main: \#f9fafb;  
  \--text-muted: \#9ca3af;  
  \--accent-blue: \#3b82f6;  
  \--accent-cyan: \#06b6d4;  
  \--accent-green: \#10b981;  
  \--accent-red: \#ef4444;  
  \--accent-amber: \#f59e0b;  
  \--accent-purple: \#8b5cf6;  
  \--font-sans: 'Inter', \-apple-system, BlinkMacSystemFont, 'Segoe UI', Roboto, sans-serif;  
  \--font-mono: 'JetBrains Mono', 'Fira Code', Menlo, Monaco, Consolas, monospace;  
}

\* {  
  box-sizing: border-box;  
  margin: 0;  
  padding: 0;  
}

body {  
  font-family: var(--font-sans);  
  background-color: var(--bg-primary);  
  color: var(--text-main);  
  line-height: 1.5;  
  min-height: 100vh;  
  padding: 24px;  
}

.studio-container {  
  max-width: 1440px;  
  margin: 0 auto;  
  display: flex;  
  flex-direction: column;  
  gap: 24px;  
}

/\* Header \*/  
.studio-header {  
  display: flex;  
  justify-content: space-between;  
  align-items: center;  
  padding: 20px 24px;  
  background: var(--bg-card);  
  backdrop-filter: blur(16px);  
  border: 1px solid var(--border-color);  
  border-radius: 16px;  
  flex-wrap: wrap;  
  gap: 16px;  
}

.brand-wrapper {  
  display: flex;  
  align-items: center;  
  gap: 14px;  
}

.brand-icon {  
  width: 44px;  
  height: 44px;  
  background: linear-gradient(135deg, \#2563eb, \#06b6d4);  
  border-radius: 12px;  
  display: flex;  
  align-items: center;  
  justify-content: center;  
  color: white;  
  box-shadow: 0 4px 14px rgba(37, 99, 235, 0.4);  
}

.brand-title {  
  font-size: 1.35rem;  
  font-weight: 700;  
  letter-spacing: \-0.02em;  
  display: flex;  
  align-items: center;  
  gap: 8px;  
}

.version-badge {  
  font-size: 0.75rem;  
  padding: 2px 8px;  
  border-radius: 9999px;  
  background: rgba(59, 130, 246, 0.2);  
  color: \#60a5fa;  
  border: 1px solid rgba(59, 130, 246, 0.4);  
}

.brand-subtitle {  
  font-size: 0.85rem;  
  color: var(--text-muted);  
}

.header-actions {  
  display: flex;  
  align-items: center;  
  gap: 12px;  
  flex-wrap: wrap;  
}

.preset-pill-group {  
  display: flex;  
  gap: 8px;  
  background: rgba(0, 0, 0, 0.3);  
  padding: 4px;  
  border-radius: 10px;  
  border: 1px solid var(--border-color);  
}

.btn-preset {  
  background: transparent;  
  border: none;  
  color: var(--text-muted);  
  font-size: 0.8rem;  
  font-weight: 500;  
  padding: 6px 12px;  
  border-radius: 6px;  
  cursor: pointer;  
  transition: all 0.2s ease;  
}

.btn-preset:hover {  
  color: var(--text-main);  
  background: rgba(255, 255, 255, 0.05);  
}

.btn-preset.active {  
  color: \#fff;  
  background: var(--accent-blue);  
  box-shadow: 0 2px 8px rgba(59, 130, 246, 0.4);  
}

/\* Layout Grid \*/  
.studio-grid {  
  display: grid;  
  grid-template-columns: 440px 1fr;  
  gap: 24px;  
}

@media (max-width: 1024px) {  
  .studio-grid {  
    grid-template-columns: 1fr;  
  }  
}

/\* Studio Panels \*/  
.studio-panel {  
  background: var(--bg-card);  
  border: 1px solid var(--border-color);  
  border-radius: 16px;  
  padding: 24px;  
  display: flex;  
  flex-direction: column;  
  gap: 20px;  
  backdrop-filter: blur(12px);  
}

.panel-title {  
  font-size: 1.1rem;  
  font-weight: 600;  
  display: flex;  
  align-items: center;  
  gap: 8px;  
  padding-bottom: 12px;  
  border-bottom: 1px solid var(--border-color);  
}

/\* Form Controls \*/  
.form-group {  
  display: flex;  
  flex-direction: column;  
  gap: 8px;  
}

.form-label {  
  font-size: 0.85rem;  
  font-weight: 500;  
  color: var(--text-muted);  
  display: flex;  
  justify-content: space-between;  
}

.form-label .val-display {  
  color: var(--accent-cyan);  
  font-family: var(--font-mono);  
  font-weight: 600;  
}

.form-select, .form-input {  
  background: rgba(0, 0, 0, 0.4);  
  border: 1px solid var(--border-color);  
  color: var(--text-main);  
  padding: 10px 14px;  
  border-radius: 8px;  
  font-size: 0.9rem;  
  font-family: inherit;  
  transition: all 0.2s ease;  
}

.form-select:focus, .form-input:focus {  
  outline: none;  
  border-color: var(--border-focus);  
  box-shadow: 0 0 0 3px rgba(59, 130, 246, 0.2);  
}

.form-range {  
  accent-color: var(--accent-blue);  
  cursor: pointer;  
  height: 6px;  
  background: rgba(255, 255, 255, 0.1);  
  border-radius: 3px;  
}

/\* Simulation Controls \*/  
.sim-btn-group {  
  display: grid;  
  grid-template-columns: 1fr 1fr;  
  gap: 12px;  
  margin-top: 8px;  
}

.btn {  
  display: inline-flex;  
  align-items: center;  
  justify-content: center;  
  gap: 8px;  
  padding: 10px 16px;  
  font-size: 0.88rem;  
  font-weight: 600;  
  border-radius: 8px;  
  cursor: pointer;  
  border: 1px solid transparent;  
  transition: all 0.2s ease;  
}

.btn-primary {  
  background: linear-gradient(135deg, \#2563eb, \#1d4ed8);  
  color: white;  
  box-shadow: 0 4px 12px rgba(37, 99, 235, 0.3);  
}

.btn-primary:hover {  
  filter: brightness(1.1);  
  transform: translateY(-1px);  
}

.btn-danger {  
  background: rgba(239, 68, 68, 0.15);  
  color: \#f87171;  
  border: 1px solid rgba(239, 68, 68, 0.3);  
}

.btn-danger:hover {  
  background: rgba(239, 68, 68, 0.25);  
}

.btn-secondary {  
  background: rgba(255, 255, 255, 0.08);  
  color: var(--text-main);  
  border: 1px solid var(--border-color);  
}

.btn-secondary:hover {  
  background: rgba(255, 255, 255, 0.12);  
}

/\* KPI Summary Cards \*/  
.metrics-row {  
  display: grid;  
  grid-template-columns: repeat(4, 1fr);  
  gap: 14px;  
}

@media (max-width: 768px) {  
  .metrics-row {  
    grid-template-columns: repeat(2, 1fr);  
  }  
}

.metric-card {  
  background: rgba(0, 0, 0, 0.3);  
  border: 1px solid var(--border-color);  
  border-radius: 12px;  
  padding: 14px 16px;  
  display: flex;  
  flex-direction: column;  
  gap: 4px;  
}

.metric-label {  
  font-size: 0.75rem;  
  color: var(--text-muted);  
  text-transform: uppercase;  
  letter-spacing: 0.05em;  
  font-weight: 600;  
}

.metric-value {  
  font-size: 1.5rem;  
  font-weight: 700;  
  font-family: var(--font-mono);  
}

.metric-card.success .metric-value { color: var(--accent-green); }  
.metric-card.danger .metric-value { color: var(--accent-red); }  
.metric-card.info .metric-value { color: var(--accent-cyan); }  
.metric-card.warning .metric-value { color: var(--accent-amber); }

/\* Bucket Visualizer \*/  
.visualizer-wrapper {  
  background: rgba(0, 0, 0, 0.35);  
  border: 1px solid var(--border-color);  
  border-radius: 12px;  
  padding: 20px;  
  display: flex;  
  flex-direction: column;  
  gap: 12px;  
}

.bucket-bar-container {  
  height: 28px;  
  background: rgba(255, 255, 255, 0.05);  
  border-radius: 14px;  
  overflow: hidden;  
  position: relative;  
  border: 1px solid var(--border-color);  
}

.bucket-fill {  
  height: 100%;  
  width: 100%;  
  background: linear-gradient(90deg, \#10b981, \#06b6d4);  
  transition: width 0.3s cubic-bezier(0.4, 0, 0.2, 1);  
  border-radius: 14px;  
}

.bucket-fill.low {  
  background: linear-gradient(90deg, \#ef4444, \#f59e0b);  
}

.bucket-markers {  
  display: flex;  
  justify-content: space-between;  
  font-size: 0.8rem;  
  color: var(--text-muted);  
  font-family: var(--font-mono);  
}

/\* Timeline Chart \*/  
.timeline-container {  
  height: 110px;  
  background: rgba(0, 0, 0, 0.4);  
  border: 1px solid var(--border-color);  
  border-radius: 12px;  
  padding: 12px;  
  display: flex;  
  align-items: flex-end;  
  gap: 4px;  
  overflow-x: auto;  
  position: relative;  
}

.timeline-bar {  
  flex: 0 0 10px;  
  border-radius: 3px 3px 0 0;  
  transition: height 0.2s ease;  
  position: relative;  
}

.timeline-bar.accepted {  
  background: var(--accent-green);  
  box-shadow: 0 0 8px rgba(16, 185, 129, 0.4);  
}

.timeline-bar.rejected {  
  background: var(--accent-red);  
  box-shadow: 0 0 8px rgba(239, 68, 68, 0.4);  
}

/\* HTTP Headers Inspector \*/  
.http-inspector {  
  background: \#0d1117;  
  border: 1px solid var(--border-color);  
  border-radius: 12px;  
  overflow: hidden;  
}

.http-header-bar {  
  background: rgba(255, 255, 255, 0.03);  
  padding: 10px 16px;  
  display: flex;  
  justify-content: space-between;  
  align-items: center;  
  border-bottom: 1px solid var(--border-color);  
  font-size: 0.85rem;  
}

.status-badge {  
  font-size: 0.8rem;  
  font-weight: 700;  
  padding: 3px 10px;  
  border-radius: 6px;  
  font-family: var(--font-mono);  
}

.status-badge.status-200 {  
  background: rgba(16, 185, 129, 0.2);  
  color: \#34d399;  
  border: 1px solid rgba(16, 185, 129, 0.4);  
}

.status-badge.status-429 {  
  background: rgba(239, 68, 68, 0.2);  
  color: \#f87171;  
  border: 1px solid rgba(239, 68, 68, 0.4);  
}

.http-table {  
  width: 100%;  
  border-collapse: collapse;  
  font-family: var(--font-mono);  
  font-size: 0.8rem;  
}

.http-table td {  
  padding: 8px 16px;  
  border-bottom: 1px solid rgba(255, 255, 255, 0.04);  
}

.http-table td.header-name {  
  color: \#60a5fa;  
  width: 220px;  
}

.http-table td.header-val {  
  color: \#e5e7eb;  
}

/\* Code Synthesizer Tabs \*/  
.tabs-container {  
  display: flex;  
  flex-direction: column;  
  gap: 12px;  
}

.tab-nav {  
  display: flex;  
  gap: 6px;  
  background: rgba(0, 0, 0, 0.3);  
  padding: 4px;  
  border-radius: 10px;  
  border: 1px solid var(--border-color);  
  overflow-x: auto;  
}

.tab-btn {  
  background: transparent;  
  border: none;  
  color: var(--text-muted);  
  font-size: 0.82rem;  
  font-weight: 500;  
  padding: 8px 14px;  
  border-radius: 6px;  
  cursor: pointer;  
  white-space: nowrap;  
  transition: all 0.2s ease;  
}

.tab-btn:hover {  
  color: var(--text-main);  
  background: rgba(255, 255, 255, 0.05);  
}

.tab-btn.active {  
  color: \#fff;  
  background: rgba(255, 255, 255, 0.12);  
  border: 1px solid rgba(255, 255, 255, 0.15);  
}

.code-display {  
  background: \#0d1117;  
  border: 1px solid var(--border-color);  
  border-radius: 12px;  
  padding: 16px;  
  font-family: var(--font-mono);  
  font-size: 0.82rem;  
  color: \#c9d1d9;  
  overflow-x: auto;  
  max-height: 380px;  
  white-space: pre;  
  line-height: 1.45;  
}

.code-actions {  
  display: flex;  
  justify-content: flex-end;  
  gap: 10px;  
}

/\* Footer \*/  
.studio-footer {  
  text-align: center;  
  font-size: 0.8rem;  
  color: var(--text-muted);  
  padding: 16px;  
  border-top: 1px solid var(--border-color);  
}  
