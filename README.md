<html lang="en">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1.0">
<title>INDRA Terminal — Indian Market Dashboard</title>
<style>
  :root {
    --bg: #0a0a0f;
    --bg-panel: #111118;
    --bg-panel-hover: #161620;
    --border: #1f1f2e;
    --border-bright: #2a2a3d;
    --text: #c9d1d9;
    --text-dim: #6e7681;
    --accent: #00d4aa;
    --accent-dim: #00a884;
    --accent-glow: rgba(0, 212, 170, 0.15);
    --red: #ff5555;
    --green: #00d4aa;
    --yellow: #f0ad4e;
    --blue: #58a6ff;
    --purple: #bc8cff;
    --orange: #ff9f43;
    --font-mono: 'SFMono-Regular', 'Menlo', 'Monaco', 'Consolas', 'Liberation Mono', 'Courier New', monospace;
  }

  * { box-sizing: border-box; margin: 0; padding: 0; }

  html, body {
    background: var(--bg);
    color: var(--text);
    font-family: var(--font-mono);
    font-size: 11px;
    line-height: 1.45;
    overflow-x: hidden;
    min-height: 100vh;
    width: 100vw;
  }

  ::-webkit-scrollbar { width: 5px; height: 5px; }
  ::-webkit-scrollbar-track { background: var(--bg); }
  ::-webkit-scrollbar-thumb { background: var(--border-bright); border-radius: 3px; }
  ::-webkit-scrollbar-thumb:hover { background: var(--accent-dim); }

  /* Header */
  #header {
    height: 42px;
    background: var(--bg-panel);
    border-bottom: 1px solid var(--border);
    display: flex;
    align-items: center;
    justify-content: space-between;
    padding: 0 12px;
    position: sticky;
    top: 0;
    z-index: 1000;
    flex-shrink: 0;
  }

  #header .logo {
    display: flex;
    align-items: center;
    gap: 6px;
    color: var(--accent);
    font-weight: bold;
    font-size: 12px;
    letter-spacing: 1px;
  }

  #header .logo::before {
    content: '▶';
    color: var(--accent);
    animation: pulse 2s infinite;
  }

  @keyframes pulse {
    0%, 100% { opacity: 1; }
    50% { opacity: 0.3; }
  }

  #header .status {
    display: flex;
    align-items: center;
    gap: 10px;
    color: var(--text-dim);
    font-size: 10px;
  }

  #header .status span { display: flex; align-items: center; gap: 4px; }

  .dot {
    width: 6px;
    height: 6px;
    border-radius: 50%;
    display: inline-block;
  }
  .dot.live { background: var(--green); box-shadow: 0 0 5px var(--green); }
  .dot.demo { background: var(--yellow); box-shadow: 0 0 5px var(--yellow); }
  .dot.error { background: var(--red); box-shadow: 0 0 5px var(--red); }

  #mode-toggle, #add-widget-btn, #reset-btn, #api-btn {
    background: var(--bg);
    border: 1px solid var(--border);
    color: var(--text);
    padding: 2px 8px;
    cursor: pointer;
    font-family: inherit;
    font-size: 10px;
    border-radius: 3px;
    transition: all 0.2s;
  }
  #mode-toggle:hover, #add-widget-btn:hover, #reset-btn:hover, #api-btn:hover { border-color: var(--accent); color: var(--accent); }

  /* Sentiment Banner */
  #sentiment-banner {
    height: 22px;
    display: flex;
    align-items: center;
    justify-content: center;
    gap: 16px;
    font-size: 10px;
    font-weight: bold;
    letter-spacing: 0.5px;
    border-bottom: 1px solid var(--border);
    transition: background 0.5s, color 0.5s;
  }
  #sentiment-banner.bullish { background: rgba(0,212,170,0.08); color: var(--green); }
  #sentiment-banner.bearish { background: rgba(255,85,85,0.08); color: var(--red); }
  #sentiment-banner.neutral { background: rgba(240,173,78,0.08); color: var(--yellow); }
  #sentiment-banner .sent-item { display: flex; align-items: center; gap: 4px; }
  #sentiment-banner .sent-label { color: var(--text-dim); font-weight: normal; font-size: 9px; }

  /* Dashboard */
  #dashboard {
    position: relative;
    width: 100%;
    min-height: calc(100vh - 64px);
    overflow-y: auto;
    overflow-x: hidden;
    background:
      linear-gradient(rgba(0,212,170,0.03) 1px, transparent 1px),
      linear-gradient(90deg, rgba(0,212,170,0.03) 1px, transparent 1px);
    background-size: 20px 20px;
    padding: 12px;
  }

  /* Widget Base */
  .widget {
    position: absolute;
    background: var(--bg-panel);
    border: 1px solid var(--border);
    border-radius: 4px;
    overflow: hidden;
    display: flex;
    flex-direction: column;
    min-width: 220px;
    min-height: 140px;
    box-shadow: 0 4px 20px rgba(0,0,0,0.5);
    transition: box-shadow 0.2s, border-color 0.2s;
  }

  .widget.dragging {
    opacity: 0.9;
    box-shadow: 0 8px 40px rgba(0,0,0,0.8), 0 0 0 1px var(--accent);
    z-index: 9999 !important;
  }

  .widget:hover { border-color: var(--border-bright); }

  .widget-header {
    height: 26px;
    background: linear-gradient(90deg, var(--bg-panel), var(--bg-panel-hover));
    border-bottom: 1px solid var(--border);
    display: flex;
    align-items: center;
    justify-content: space-between;
    padding: 0 8px;
    cursor: grab;
    user-select: none;
    flex-shrink: 0;
  }

  .widget-header:active { cursor: grabbing; }

  .widget-title {
    color: var(--accent);
    font-size: 10px;
    font-weight: bold;
    letter-spacing: 0.5px;
    display: flex;
    align-items: center;
    gap: 5px;
  }

  .widget-title::before {
    content: '■';
    font-size: 7px;
    color: var(--accent-dim);
  }

  .widget-controls {
    display: flex;
    gap: 4px;
  }

  .widget-btn {
    background: none;
    border: none;
    color: var(--text-dim);
    cursor: pointer;
    font-family: inherit;
    font-size: 9px;
    padding: 2px 4px;
    border-radius: 2px;
    transition: color 0.2s;
  }
  .widget-btn:hover { color: var(--accent); }

  .widget-body {
    flex: 1;
    overflow: auto;
    padding: 6px;
    position: relative;
    font-size: 10px;
  }

  .resize-handle {
    position: absolute;
    bottom: 0;
    right: 0;
    width: 12px;
    height: 12px;
    cursor: se-resize;
    background: linear-gradient(135deg, transparent 50%, var(--border-bright) 50%);
    border-bottom-right-radius: 3px;
    opacity: 0.5;
    transition: opacity 0.2s;
  }
  .resize-handle:hover { opacity: 1; background: linear-gradient(135deg, transparent 50%, var(--accent) 50%); }

  /* Tables */
  table {
    width: 100%;
    border-collapse: collapse;
    font-size: 10px;
  }

  th {
    text-align: left;
    padding: 3px 5px;
    color: var(--text-dim);
    border-bottom: 1px solid var(--border);
    font-weight: normal;
    font-size: 9px;
    text-transform: uppercase;
    letter-spacing: 0.5px;
    position: sticky;
    top: 0;
    background: var(--bg-panel);
  }

  td {
    padding: 4px 5px;
    border-bottom: 1px solid rgba(31,31,46,0.5);
    color: var(--text);
    font-size: 10px;
  }

  tr:hover td { background: rgba(0,212,170,0.05); }

  .up { color: var(--green); }
  .down { color: var(--red); }
  .muted { color: var(--text-dim); font-size: 9px; }

  /* Canvas Charts */
  canvas.chart-canvas {
    width: 100%;
    height: 100%;
    display: block;
  }

  /* Heatmap */
  .heatmap-grid {
    display: grid;
    grid-template-columns: repeat(auto-fill, minmax(70px, 1fr));
    gap: 3px;
    height: 100%;
  }

  .heatmap-cell {
    display: flex;
    flex-direction: column;
    align-items: center;
    justify-content: center;
    padding: 3px;
    border-radius: 2px;
    font-size: 9px;
    text-align: center;
    cursor: pointer;
    transition: transform 0.1s;
    position: relative;
    overflow: hidden;
  }

  .heatmap-cell:hover { transform: scale(1.05); z-index: 2; }
  .heatmap-cell .hm-symbol { font-weight: bold; font-size: 9px; margin-bottom: 1px; }
  .heatmap-cell .hm-change { font-size: 8px; }

  /* Clocks */
  .clock-grid {
    display: grid;
    grid-template-columns: repeat(2, 1fr);
    gap: 6px;
    height: 100%;
  }

  .clock-item {
    background: rgba(0,0,0,0.3);
    border: 1px solid var(--border);
    border-radius: 3px;
    padding: 6px;
    display: flex;
    flex-direction: column;
    align-items: center;
    justify-content: center;
    text-align: center;
  }

  .clock-city { font-size: 9px; color: var(--accent); margin-bottom: 3px; }
  .clock-time { font-size: 16px; font-weight: bold; color: var(--text); }
  .clock-status { font-size: 8px; margin-top: 3px; padding: 1px 5px; border-radius: 2px; }
  .clock-status.open { background: rgba(0,212,170,0.15); color: var(--green); }
  .clock-status.closed { background: rgba(255,85,85,0.15); color: var(--red); }
  .clock-status.pre { background: rgba(240,173,78,0.15); color: var(--yellow); }

  /* Indices row */
  .indices-row {
    display: flex;
    gap: 8px;
    flex-wrap: wrap;
  }

  .index-pill {
    background: rgba(0,0,0,0.3);
    border: 1px solid var(--border);
    border-radius: 3px;
    padding: 5px 8px;
    min-width: 100px;
  }

  .index-name { font-size: 9px; color: var(--text-dim); margin-bottom: 1px; }
  .index-value { font-size: 12px; font-weight: bold; }
  .index-change { font-size: 9px; margin-top: 1px; }

  /* Metals */
  .metal-card {
    display: flex;
    align-items: center;
    justify-content: space-between;
    padding: 6px;
    background: rgba(0,0,0,0.2);
    border: 1px solid var(--border);
    border-radius: 3px;
    margin-bottom: 5px;
  }

  .metal-info { display: flex; flex-direction: column; }
  .metal-name { font-size: 10px; color: var(--text-dim); }
  .metal-price { font-size: 13px; font-weight: bold; color: var(--text); }
  .metal-change { font-size: 9px; }

  /* Sparkline */
  .sparkline {
    width: 50px;
    height: 20px;
  }

  /* Technical Analysis Widget Styles */
  .ta-grid {
    display: grid;
    grid-template-columns: 1fr 1fr;
    gap: 6px;
  }

  .ta-card {
    background: rgba(0,0,0,0.2);
    border: 1px solid var(--border);
    border-radius: 3px;
    padding: 8px;
    text-align: center;
  }

  .ta-label {
    font-size: 9px;
    color: var(--text-dim);
    text-transform: uppercase;
    letter-spacing: 0.5px;
    margin-bottom: 4px;
  }

  .ta-value {
    font-size: 16px;
    font-weight: bold;
  }

  .ta-signal {
    font-size: 9px;
    margin-top: 3px;
    padding: 2px 6px;
    border-radius: 2px;
    display: inline-block;
  }

  .ta-signal.buy { background: rgba(0,212,170,0.15); color: var(--green); }
  .ta-signal.sell { background: rgba(255,85,85,0.15); color: var(--red); }
  .ta-signal.neutral { background: rgba(240,173,78,0.15); color: var(--yellow); }

  .ta-bar-container {
    width: 100%;
    height: 6px;
    background: var(--border);
    border-radius: 3px;
    margin-top: 4px;
    overflow: hidden;
  }

  .ta-bar {
    height: 100%;
    border-radius: 3px;
    transition: width 0.5s;
  }

  .bias-meter {
    display: flex;
    align-items: center;
    justify-content: center;
    gap: 8px;
    padding: 8px;
    background: rgba(0,0,0,0.2);
    border: 1px solid var(--border);
    border-radius: 3px;
    margin-bottom: 6px;
  }

  .bias-label {
    font-size: 9px;
    color: var(--text-dim);
    text-transform: uppercase;
  }

  .bias-value {
    font-size: 14px;
    font-weight: bold;
  }

  .bias-arrow {
    font-size: 16px;
  }

  .api-input-row {
    display: flex;
    gap: 6px;
    margin-bottom: 8px;
  }

  .api-input {
    flex: 1;
    background: var(--bg);
    border: 1px solid var(--border);
    color: var(--text);
    padding: 4px 8px;
    font-family: inherit;
    font-size: 10px;
    border-radius: 3px;
  }

  .api-btn-small {
    background: var(--bg-panel-hover);
    border: 1px solid var(--border);
    color: var(--accent);
    padding: 4px 10px;
    cursor: pointer;
    font-family: inherit;
    font-size: 10px;
    border-radius: 3px;
  }

  .api-status {
    font-size: 9px;
    color: var(--text-dim);
    margin-bottom: 6px;
  }

  /* Tooltip */
  #tooltip {
    position: fixed;
    background: var(--bg-panel);
    border: 1px solid var(--accent);
    padding: 5px 8px;
    border-radius: 3px;
    font-size: 10px;
    pointer-events: none;
    z-index: 10000;
    display: none;
    box-shadow: 0 4px 20px rgba(0,0,0,0.8);
    max-width: 180px;
  }

  /* Loading overlay */
  #loading {
    position: fixed;
    inset: 0;
    background: var(--bg);
    display: flex;
    flex-direction: column;
    align-items: center;
    justify-content: center;
    z-index: 20000;
    gap: 14px;
  }

  #loading.hidden { display: none; }

  .loader-text {
    color: var(--accent);
    font-size: 11px;
    letter-spacing: 2px;
    animation: blink 1s infinite;
  }

  @keyframes blink { 0%, 100% { opacity: 1; } 50% { opacity: 0.3; } }

  .loader-bar {
    width: 180px;
    height: 2px;
    background: var(--border);
    border-radius: 1px;
    overflow: hidden;
  }

  .loader-bar-inner {
    height: 100%;
    width: 0%;
    background: var(--accent);
    animation: load 1.5s ease-out forwards;
  }

  @keyframes load { to { width: 100%; } }

  /* Add widget menu */
  #add-menu {
    position: fixed;
    top: 64px;
    right: 8px;
    background: var(--bg-panel);
    border: 1px solid var(--border);
    border-radius: 4px;
    padding: 6px;
    z-index: 10001;
    display: none;
    min-width: 180px;
    box-shadow: 0 8px 30px rgba(0,0,0,0.8);
  }

  #add-menu.show { display: block; }

  .menu-item {
    padding: 5px 8px;
    cursor: pointer;
    border-radius: 2px;
    font-size: 10px;
    color: var(--text);
    transition: background 0.15s;
  }
  .menu-item:hover { background: var(--accent-glow); color: var(--accent); }

  /* API Config Modal */
  #api-modal {
    position: fixed;
    inset: 0;
    background: rgba(0,0,0,0.7);
    display: none;
    align-items: center;
    justify-content: center;
    z-index: 20001;
  }
  #api-modal.show { display: flex; }

  .modal-box {
    background: var(--bg-panel);
    border: 1px solid var(--border);
    border-radius: 6px;
    padding: 16px;
    width: 360px;
    max-width: 90vw;
  }

  .modal-title {
    color: var(--accent);
    font-size: 12px;
    font-weight: bold;
    margin-bottom: 12px;
    letter-spacing: 0.5px;
  }

  .modal-row {
    margin-bottom: 10px;
  }

  .modal-label {
    font-size: 9px;
    color: var(--text-dim);
    text-transform: uppercase;
    margin-bottom: 4px;
    display: block;
  }

  .modal-input {
    width: 100%;
    background: var(--bg);
    border: 1px solid var(--border);
    color: var(--text);
    padding: 6px 8px;
    font-family: inherit;
    font-size: 11px;
    border-radius: 3px;
  }

  .modal-btns {
    display: flex;
    justify-content: flex-end;
    gap: 8px;
    margin-top: 12px;
  }

  .modal-btn {
    padding: 4px 12px;
    border-radius: 3px;
    font-family: inherit;
    font-size: 10px;
    cursor: pointer;
    border: 1px solid var(--border);
  }
  .modal-btn.primary { background: var(--accent); color: var(--bg); border-color: var(--accent); }
  .modal-btn.secondary { background: transparent; color: var(--text); }

  @media (max-width: 768px) {
    .widget { position: relative !important; width: 100% !important; left: 0 !important; top: auto !important; margin-bottom: 8px; height: auto !important; min-height: 180px; }
    #dashboard { height: auto; overflow: visible; display: flex; flex-direction: column; }
    .resize-handle { display: none; }
    .widget-header { cursor: default; }
    #sentiment-banner { flex-wrap: wrap; height: auto; padding: 4px; gap: 8px; }
  }
</style>
</head>
<body>

<div id="loading">
  <div class="loader-text">INITIALIZING INDRA TERMINAL...</div>
  <div class="loader-bar"><div class="loader-bar-inner"></div></div>
</div>

<div id="header">
  <div class="logo">INDRA MARKET TERMINAL</div>
  <div class="status">
    <span id="conn-status"><span class="dot demo" id="status-dot"></span> <span id="status-text">DEMO MODE</span></span>
    <span id="clock">--:--:--</span>
    <button id="mode-toggle" onclick="toggleMode()">SWITCH TO LIVE</button>
    <button id="api-btn" onclick="toggleApiModal()">API KEY</button>
    <button id="add-widget-btn" onclick="toggleAddMenu()">+ WIDGET</button>
    <button id="reset-btn" onclick="resetLayout()">RESET</button>
  </div>
</div>

<div id="sentiment-banner" class="neutral">
  <div class="sent-item"><span class="sent-label">NIFTY 50</span><span id="sent-nifty">--</span></div>
  <div class="sent-item"><span class="sent-label">SENSEX</span><span id="sent-sensex">--</span></div>
  <div class="sent-item"><span class="sent-label">15M BIAS</span><span id="sent-bias">NEUTRAL</span></div>
  <div class="sent-item"><span class="sent-label">RSI(14)</span><span id="sent-rsi">--</span></div>
  <div class="sent-item"><span class="sent-label">MACD</span><span id="sent-macd">--</span></div>
  <div class="sent-item"><span class="sent-label">VWAP</span><span id="sent-vwap">--</span></div>
</div>

<div id="add-menu">
  <div class="menu-item" onclick="addWidget('indianTopVolume')">Indian Top Volume</div>
  <div class="menu-item" onclick="addWidget('indianTopGainers')">Indian Top Gainers</div>
  <div class="menu-item" onclick="addWidget('globalIndices')">Global Indices</div>
  <div class="menu-item" onclick="addWidget('sectorHeatmap')">Sector Heatmap</div>
  <div class="menu-item" onclick="addWidget('aaplChart')">AAPL 60-Session</div>
  <di
  v class="menu-item" onclick="addWidget('preciousMetals')">Precious Metals</div>
  <div class="menu-item" onclick="addWidget('worldClocks')">World Clocks</div>
  <div class="menu-item" onclick="addWidget('technicalAnalysis')">Technical Analysis</div>
  <div class="menu-item" onclick="addWidget('nifty15m')">NIFTY 15-Min Chart</div>
</div>

<div id="api-modal">
  <div class="modal-box">
    <div class="modal-title">API CONFIGURATION</div>
    <div class="modal-row">
      <label class="modal-label">Alpha Vantage API Key (optional)</label>
      <input class="modal-input" id="av-key" placeholder="Enter your Alpha Vantage API key" />
      <div style="font-size:9px;color:var(--text-dim);margin-top:4px;">Get free key at alphavantage.co/support/#api-key</div>
    </div>
    <div class="modal-row">
      <label class="modal-label">Data Source Preference</label>
      <select class="modal-input" id="data-source" style="cursor:pointer;">
        <option value="yahoo">Yahoo Finance (default, free)</option>
        <option value="alpha">Alpha Vantage (requires key)</option>
        <option value="demo">Demo Data (offline)</option>
      </select>
    </div>
    <div class="modal-btns">
      <button class="modal-btn secondary" onclick="toggleApiModal()">Cancel</button>
      <button class="modal-btn primary" onclick="saveApiConfig()">Save</button>
    </div>
  </div>
</div>

<div id="dashboard"></div>
<div id="tooltip"></div>

<script>
(function() {
  'use strict';

  // ===================== CONFIG =====================
  const CONFIG = {
    gridSize: 20,
    defaultWidgets: [
      { type: 'technicalAnalysis', x: 12, y: 12, w: 340, h: 260 },
      { type: 'nifty15m', x: 368, y: 12, w: 520, h: 260 },
      { type: 'indianTopVolume', x: 12, y: 284, w: 320, h: 260 },
      { type: 'indianTopGainers', x: 348, y: 284, w: 320, h: 260 },
      { type: 'globalIndices', x: 680, y: 12, w: 440, h: 140 },
      { type: 'sectorHeatmap', x: 680, y: 164, w: 440, h: 220 },
      { type: 'aaplChart', x: 12, y: 556, w: 520, h: 300 },
      { type: 'preciousMetals', x: 544, y: 556, w: 280, h: 180 },
      { type: 'worldClocks', x: 544, y: 748, w: 280, h: 180 }
    ],
    refreshInterval: 5000,
    demoMode: true,
    dataSource: 'yahoo',
    alphaVantageKey: '',
    yahooSymbols: {
      nifty: '^NSEI',
      sensex: '^BSESN',
      reliance: 'RELIANCE.NS',
      tcs: 'TCS.NS',
      hdfcbank: 'HDFCBANK.NS',
      infy: 'INFY.NS',
      icicibank: 'ICICIBANK.NS',
      sbin: 'SBIN.NS',
      bhartiartl: 'BHARTIARTL.NS',
      itc: 'ITC.NS',
      lt: 'LT.NS',
      kotakbank: 'KOTAKBANK.NS',
      axisbank: 'AXISBANK.NS',
      maruti: 'MARUTI.NS',
      hcltech: 'HCLTECH.NS',
      wipro: 'WIPRO.NS',
      sunpharma: 'SUNPHARMA.NS',
      sp500: '^GSPC',
      nasdaq: '^IXIC',
      dow: '^DJI',
      ftse: '^FTSE',
      dax: '^GDAXI',
      nikkei: '^N225',
      hangSeng: '^HSI',
      shanghai: '000001.SS',
      aapl: 'AAPL',
      gold: 'GC=F',
      silver: 'SI=F',
      platinum: 'PL=F',
      palladium: 'PA=F',
      copper: 'HG=F'
    }
  };

  // ===================== STATE =====================
  let widgets = [];
  let nextId = 1;
  let isDragging = false;
  let isResizing = false;
  let dragWidget = null;
  let resizeWidget = null;
  let dragOffset = { x: 0, y: 0 };
  let resizeStart = { x: 0, y: 0, w: 0, h: 0 };
  let dataMode = 'demo';
  let updateTimer = null;
  let liveDataCache = {};
  let lastFetchTime = 0;
  let technicalData = {};

  const dashboard = document.getElementById('dashboard');
  const tooltip = document.getElementById('tooltip');
  const sentimentBanner = document.getElementById('sentiment-banner');

  // ===================== DATA ADAPTERS =====================
  const DataAdapter = {
    randomChange(base, volatility = 0.02) {
      return base * (1 + (Math.random() - 0.5) * volatility);
    },

    indianStocks: [
      { symbol: 'RELIANCE', name: 'Reliance Ind.', price: 2456.30, vol: 4520000 },
      { symbol: 'TCS', name: 'Tata Consultancy', price: 3890.50, vol: 1230000 },
      { symbol: 'HDFCBANK', name: 'HDFC Bank', price: 1423.80, vol: 3890000 },
      { symbol: 'INFY', name: 'Infosys', price: 1456.20, vol: 2100000 },
      { symbol: 'ICICIBANK', name: 'ICICI Bank', price: 945.60, vol: 5670000 },
      { symbol: 'SBIN', name: 'SBI', price: 567.40, vol: 8900000 },
      { symbol: 'BHARTIARTL', name: 'Bharti Airtel', price: 876.30, vol: 1780000 },
      { symbol: 'ITC', name: 'ITC Ltd', price: 423.50, vol: 6540000 },
      { symbol: 'LT', name: 'Larsen & Toubro', price: 2345.60, vol: 980000 },
      { symbol: 'KOTAKBANK', name: 'Kotak Bank', price: 1678.90, vol: 1450000 },
      { symbol: 'AXISBANK', name: 'Axis Bank', price: 987.40, vol: 2340000 },
      { symbol: 'MARUTI', name: 'Maruti Suzuki', price: 9876.50, vol: 560000 },
      { symbol: 'HCLTECH', name: 'HCL Tech', price: 1234.50, vol: 890000 },
      { symbol: 'WIPRO', name: 'Wipro', price: 456.70, vol: 1230000 },
      { symbol: 'SUNPHARMA', name: 'Sun Pharma', price: 1123.40, vol: 780000 }
    ],

    globalIndices: [
      { name: 'NIFTY 50', value: 22456.30, change: 0.45 },
      { name: 'SENSEX', value: 73890.50, change: 0.38 },
      { name: 'S&P 500', value: 4456.20, change: -0.12 },
      { name: 'NASDAQ', value: 13890.40, change: 0.67 },
      { name: 'DOW JONES', value: 34567.80, change: -0.05 },
      { name: 'FTSE 100', value: 7456.30, change: 0.23 },
      { name: 'DAX', value: 15678.90, change: 0.15 },
      { name: 'NIKKEI 225', value: 33456.70, change: -0.34 },
      { name: 'HANG SENG', value: 17890.50, change: 0.89 },
      { name: 'SHANGHAI', value: 3056.40, change: -0.21 }
    ],

    sectors: [
      { name: 'AI/ML', symbol: 'AI', stocks: ['NVIDIA','AMD','PLTR','SMCI','AVGO'], change: 2.34 },
      { name: 'Solar', symbol: 'SOL', stocks: ['ENPH','SEDG','FSLR','NXT','ARRY'], change: -1.23 },
      { name: 'EV', symbol: 'EV', stocks: ['TSLA','RIVN','LCID','NIO','XPEV'], change: 0.87 },
      { name: 'Banking', symbol: 'BNK', stocks: ['JPM','BAC','WFC','C','GS'], change: 0.45 },
      { name: 'Fintech', symbol: 'FIN', stocks: ['SQ','PYPL','SOFI','UPST','AFRM'], change: -0.67 },
      { name: 'Insurance', symbol: 'INS', stocks: ['BRK.B','PGR','TRV','AIG','MET'], change: 0.12 },
      { name: 'Oil', symbol: 'OIL', stocks: ['XOM','CVX','COP','OXY','MPC'], change: 1.45 },
      { name: 'Gas', symbol: 'GAS', stocks: ['LNG','KMI','WMB','EPD','ET'], change: 0.34 },
      { name: 'Uranium', symbol: 'URN', stocks: ['CCJ','UUUU','DNN','LEU','URG'], change: 3.21 },
      { name: 'Wind', symbol: 'WND', stocks: ['GE','VWDRY','NPI','TPIC','AMSC'], change: -0.89 },
      { name: 'Grid', symbol: 'GRD', stocks: ['NEE','SO','D','AEP','SRE'], change: 0.23 },
      { name: 'Battery', symbol: 'BAT', stocks: ['ALB','SQM','LTHM','MP','CBAT'], change: -2.11 }
    ],

    metals: [
      { name: 'Gold', symbol: 'XAU', price: 2345.60, change: 0.34 },
      { name: 'Silver', symbol: 'XAG', price: 28.45, change: -0.56 },
      { name: 'Platinum', symbol: 'XPT', price: 978.30, change: 0.12 },
      { name: 'Palladium', symbol: 'XPD', price: 1023.40, change: -1.23 },
      { name: 'Copper', symbol: 'COP', price: 4.56, change: 0.78 }
    ],

    generateAAPLData() {
      const data = [];
      let price = 175;
      for (let i = 0; i < 60; i++) {
        const open = price;
        const close = price + (Math.random() - 0.48) * 4;
        const high = Math.max(open, close) + Math.random() * 2;
        const low = Math.min(open, close) - Math.random() * 2;
        const volume = 20000000 + Math.random() * 30000000;
        data.push({ open, high, low, close, volume, date: new Date(Date.now() - (59 - i) * 86400000) });
        price = close;
      }
      return data;
    },

    getIndianTopVolume() {
      return this.indianStocks.map(s => ({
        symbol: s.symbol,
        name: s.name,
        price: this.randomChange(s.price, 0.01),
        volume: Math.floor(s.vol * (0.8 + Math.random() * 0.4)),
        change: (Math.random() - 0.5) * 4
      })).sort((a, b) => b.volume - a.volume).slice(0, 10);
    },

    getIndianTopGainers() {
      return this.indianStocks.map(s => ({
        symbol: s.symbol,
        name: s.name,
        price: this.randomChange(s.price, 0.01),
        change: (Math.random() - 0.3) * 5,
        volume: Math.floor(s.vol * (0.5 + Math.random()))
      })).sort((a, b) => b.change - a.change).slice(0, 10);
    },

    getGlobalIndices() {
      return this.globalIndices.map(i => ({
        name: i.name,
        value: this.randomChange(i.value, 0.005),
        change: i.change + (Math.random() - 0.5) * 0.3
      }));
    },

    getSectors() {
      return this.sectors.map(s => ({
        ...s,
        change: s.change + (Math.random() - 0.5) * 1.5
      }));
    },

    getMetals() {
      return this.metals.map(m => ({
        ...m,
        price: this.randomChange(m.price, 0.008),
        change: m.change + (Math.random() - 0.5) * 0.5
      }));
    }
  };

  // ===================== LIVE DATA FETCHER =====================
  const LiveData = {
    async fetchYahooChart(symbol, interval = '15m', range = '5d') {
      try {
        const url = `https://query1.finance.yahoo.com/v8/finance/chart/${encodeURIComponent(symbol)}?interval=${interval}&range=${range}&includePrePost=false`;
        const response = await fetch(url, { cache: 'no-store' });
        if (!response.ok) throw new Error('HTTP ' + response.status);
        const data = await response.json();
        if (!data.chart || !data.chart.result || !data.chart.result[0]) throw new Error('No data');
        return data.chart.result[0];
      } catch (e) {
        console.warn('Yahoo fetch failed for', symbol, e.message);
        return null;
      }
    },

    async fetchYahooQuote(symbols) {
      try {
        const syms = Array.isArray(symbols) ? symbols.join(',') : symbols;
        const url = `https://query1.finance.yahoo.com/v7/finance/quote?symbols=${encodeURIComponent(syms)}`;
        const response = await fetch(url, { cache: 'no-store' });
        if (!response.ok) throw new Error('HTTP ' + response.status);
        const data = await response.json();
        if (!data.quoteResponse || !data.quoteResponse.result) throw new Error('No data');
        return data.quoteResponse.result;
      } catch (e) {
        console.warn('Yahoo quote fetch failed:', e.message);
        return null;
      }
    },

    processCandles(result) {
      if (!result || !result.timestamp) return [];
      const ts = result.timestamp;
      const q = result.indicators.quote[0];
      const candles = [];
      for (let i = 0; i < ts.length; i++) {
        if (q.open[i] != null && q.high[i] != null && q.low[i] != null && q.close[i] != null) {
          candles.push({
            date: new Date(ts[i] * 1000),
            open: q.open[i],
            high: q.high[i],
            low: q.low[i],
            close: q.close[i],
            volume: q.volume[i] || 0
          });
        }
      }
      return candles;
    }
  };

  // ===================== TECHNICAL INDICATORS =====================
  const Indicators = {
    sma(data, period) {
      if (data.length < period) return [];
      const result = [];
      for (let i = period - 1; i < data.length; i++) {
        let sum = 0;
        for (let j = 0; j < period; j++) sum += data[i - j].close;
        result.push({ date: data[i].date, value: sum / period });
      }
      return result;
    },

    ema(data, period) {
      if (data.length < period) return [];
      const k = 2 / (period + 1);
      const result = [];
      let ema = data.slice(0, period).reduce((s, d) => s + d.close, 0) / period;
      for (let i = period; i < data.length; i++) {
        ema = data[i].close * k + ema * (1 - k);
        result.push({ date: data[i].date, value: ema });
      }
      return result;
    },

    rsi(data, period = 14) {
      if (data.length < period + 1) return [];
      const result = [];
      let gains = 0, losses = 0;
      for (let i = 1; i <= period; i++) {
        const change = data[i].close - data[i - 1].close;
        if (change > 0) gains += change; else losses -= change;
      }
      let avgGain = gains / period;
      let avgLoss = losses / period;
      for (let i = period + 1; i < data.length; i++) {
        const change = data[i].close - data[i - 1].close;
        const gain = change > 0 ? change : 0;
        const loss = change < 0 ? -change : 0;
        avgGain = (avgGain * (period - 1) + gain) / period;
        avgLoss = (avgLoss * (period - 1) + loss) / period;
        const rs = avgLoss === 0 ? 100 : avgGain / avgLoss;
        const rsi = avgLoss === 0 ? 100 : 100 - (100 / (1 + rs));
        result.push({ date: data[i].date, value: rsi });
      }
      return result;
    },

    macd(data, fast = 12, slow = 26, signal = 9) {
      if (data.length < slow + signal) return { macd: [], signal: [], histogram: [] };
      const emaFast = this.ema(data, fast);
      const emaSlow = this.ema(data, slow);
      const macdLine = [];
      const offset = slow - fast;
      for (let i = 0; i < emaSlow.length; i++) {
        if (i + offset < emaFast.length) {
          macdLine.push({ date: emaSlow[i].date, value: emaFast[i + offset].value - emaSlow[i].value });
        }
      }
      const signalLine = [];
      let emaSignal = macdLine.slice(0, signal).reduce((s, d) => s + d.value, 0) / signal;
      const k = 2 / (signal + 1);
      for (let i = signal; i < macdLine.length; i++) {
        emaSignal = macdLine[i].value * k + emaSignal * (1 - k);
        signalLine.push({ date: macdLine[i].date, value: emaSignal });
      }
      const histogram = [];
      for (let i = 0; i < signalLine.length; i++) {
        if (i + signal < macdLine.length) {
          histogram.push({ date: signalLine[i].date, value: macdLine[i + signal].value - signalLine[i].value });
        }
      }
      return { macd: macdLine, signal: signalLine, histogram };
    },

    vwap(data) {
      let cumPV = 0, cumV = 0;
      return data.map(d => {
        const tp = (d.high + d.low + d.close) / 3;
        cumPV += tp * d.volume;
        cumV += d.volume;
        return { date: d.date, value: cumPV / cumV };
      });
    },

    bollinger(data, period = 20, mult = 2) {
      const sma = this.sma(data, period);
      const upper = [], lower = [];
      for (let i = period - 1; i < data.length; i++) {
        let sum = 0;
        for (let j = 0; j < period; j++) {
          sum += Math.pow(data[i - j].close - sma[i - period + 1].value, 2);
        }
        const std = Math.sqrt(sum / period);
        upper.push({ date: data[i].date, value: sma[i - period + 1].value + mult * std });
        lower.push({ date: data[i].date, value: sma[i - period + 1].value - mult * std });
      }
      return { sma, upper, lower };
    },

    analyze15mBias(candles) {
      if (candles.length < 30) return { bias: 'NEUTRAL', confidence: 0, signals: [] };
      const closes = candles.map(c => c.close);
      const latest = candles[candles.length - 1];
      const prev = candles[candles.length - 2];

      const rsiValues = this.rsi(candles, 14);
      const rsiLatest = rsiValues.length > 0 ? rsiValues[rsiValues.length - 1].value : 50;

      const macdResult = this.macd(candles, 12, 26, 9);
      const macdHist = macdResult.histogram;
      const macdLatest = macdHist.length > 0 ? macdHist[macdHist.length - 1].value : 0;
      const macdPrev = macdHist.length > 1 ? macdHist[macdHist.length - 2].value : 0;

      const vwapValues = this.vwap(candles);
      const vwapLatest = vwapValues[vwapValues.length - 1].value;

      const sma20 = this.sma(candles, 20);
      const sma20Latest = sma20.length > 0 ? sma20[sma20.length - 1].value : latest.close;

      const ema9 = this.ema(candles, 9);
      const ema9Latest = ema9.length > 0 ? ema9[ema9.length - 1].value : latest.close;

      let score = 0;
      const signals = [];

      // RSI
      if (rsiLatest < 30) { score += 2; signals.push('RSI oversold'); }
      else if (rsiLatest < 40) { score += 1; signals.push('RSI weak bullish'); }
      else if (rsiLatest > 70) { score -= 2; signals.push('RSI overbought'); }
      else if (rsiLatest > 60) { score -= 1; signals.push('RSI weak bearish'); }

      // MACD histogram
      if (macdLatest > 0 && macdPrev <= 0) { score += 2; signals.push('MACD bullish cross'); }
      else if (macdLatest > 0) { score += 1; signals.push('MACD positive'); }
      else if (macdLatest < 0 && macdPrev >= 0) { score -= 2; signals.push('MACD bearish cross'); }
      else if (macdLatest < 0) { score -= 1; signals.push('MACD negative'); }

      // Price vs VWAP
      if (latest.close > vwapLatest) { score += 1; signals.push('Above VWAP'); }
      else { score -= 1; signals.push('Below VWAP'); }

      // Price vs SMA20
      if (latest.close > sma20Latest) { score += 1; signals.push('Above SMA20'); }
      else { score -= 1; signals.push('Below SMA20'); }

      // EMA9 vs SMA20
      if 
      (ema9Latest > sma20Latest) { score += 1; signals.push('EMA9 > SMA20'); }
      else { score -= 1; signals.push('EMA9 < SMA20'); }

      // Recent candle
      if (latest.close > latest.open) { score += 0.5; }
      else { score -= 0.5; }

      const confidence = Math.min(Math.abs(score) / 4.5 * 100, 100);
      let bias = 'NEUTRAL';
      if (score >= 2) bias = 'BULLISH';
      else if (score <= -2) bias = 'BEARISH';
      else if (score > 0) bias = 'SLIGHTLY BULLISH';
      else if (score < 0) bias = 'SLIGHTLY BEARISH';

      return { bias, confidence: Math.round(confidence), score, signals, rsi: rsiLatest, macd: macdLatest, vwap: vwapLatest, sma20: sma20Latest, ema9: ema9Latest };
    }
  };

  // ===================== LIVE DATA UPDATE =====================
  async function updateLiveData() {
    if (dataMode !== 'live') return;

    const now = Date.now();
    if (now - lastFetchTime < 10000) return; // min 10s between fetches
    lastFetchTime = now;

    try {
      // Fetch NIFTY 50 15m data for technical analysis
      const niftyResult = await LiveData.fetchYahooChart(CONFIG.yahooSymbols.nifty, '15m', '5d');
      if (niftyResult) {
        const candles = LiveData.processCandles(niftyResult);
        if (candles.length > 0) {
          technicalData.nifty = candles;
          const analysis = Indicators.analyze15mBias(candles);
          technicalData.niftyAnalysis = analysis;
          updateSentimentBanner(analysis, candles[candles.length - 1]);
        }
      }

      // Fetch quotes for Indian stocks
      const indianSyms = [
        CONFIG.yahooSymbols.reliance, CONFIG.yahooSymbols.tcs, CONFIG.yahooSymbols.hdfcbank,
        CONFIG.yahooSymbols.infy, CONFIG.yahooSymbols.icicibank, CONFIG.yahooSymbols.sbin,
        CONFIG.yahooSymbols.bhartiartl, CONFIG.yahooSymbols.itc, CONFIG.yahooSymbols.lt,
        CONFIG.yahooSymbols.kotakbank, CONFIG.yahooSymbols.axisbank, CONFIG.yahooSymbols.maruti,
        CONFIG.yahooSymbols.hcltech, CONFIG.yahooSymbols.wipro, CONFIG.yahooSymbols.sunpharma
      ];
      const indianQuotes = await LiveData.fetchYahooQuote(indianSyms);
      if (indianQuotes) {
        liveDataCache.indian = indianQuotes.map(q => ({
          symbol: q.symbol.replace('.NS', ''),
          name: q.shortName || q.longName || q.symbol,
          price: q.regularMarketPrice || q.price || 0,
          change: q.regularMarketChangePercent || 0,
          volume: q.regularMarketVolume || 0
        }));
      }

      // Fetch global indices
      const globalSyms = [
        CONFIG.yahooSymbols.nifty, CONFIG.yahooSymbols.sensex, CONFIG.yahooSymbols.sp500,
        CONFIG.yahooSymbols.nasdaq, CONFIG.yahooSymbols.dow, CONFIG.yahooSymbols.ftse,
        CONFIG.yahooSymbols.dax, CONFIG.yahooSymbols.nikkei, CONFIG.yahooSymbols.hangSeng,
        CONFIG.yahooSymbols.shanghai
      ];
      const globalQuotes = await LiveData.fetchYahooQuote(globalSyms);
      if (globalQuotes) {
        liveDataCache.global = globalQuotes.map(q => ({
          name: q.shortName || q.longName || q.symbol,
          value: q.regularMarketPrice || q.price || 0,
          change: q.regularMarketChangePercent || 0
        }));
      }

      // Fetch metals
      const metalSyms = [CONFIG.yahooSymbols.gold, CONFIG.yahooSymbols.silver, CONFIG.yahooSymbols.platinum, CONFIG.yahooSymbols.palladium, CONFIG.yahooSymbols.copper];
      const metalQuotes = await LiveData.fetchYahooQuote(metalSyms);
      if (metalQuotes) {
        liveDataCache.metals = metalQuotes.map(q => ({
          name: q.shortName || q.symbol,
          symbol: q.symbol,
          price: q.regularMarketPrice || q.price || 0,
          change: q.regularMarketChangePercent || 0
        }));
      }

      // Fetch AAPL
      const aaplResult = await LiveData.fetchYahooChart(CONFIG.yahooSymbols.aapl, '1d', '3mo');
      if (aaplResult) {
        technicalData.aapl = LiveData.processCandles(aaplResult);
      }

      updateStatus('live');
      widgets.forEach(w => refreshWidget(w.id));
    } catch (e) {
      console.error('Live data update failed:', e);
      updateStatus('error');
    }
  }

  function updateSentimentBanner(analysis, latestCandle) {
    if (!analysis) return;
    const banner = document.getElementById('sentiment-banner');
    banner.className = analysis.bias.toLowerCase().includes('bull') ? 'bullish' : analysis.bias.toLowerCase().includes('bear') ? 'bearish' : 'neutral';

    document.getElementById('sent-nifty').textContent = latestCandle ? latestCandle.close.toFixed(2) : '--';
    document.getElementById('sent-sensex').textContent = liveDataCache.global && liveDataCache.global[1] ? liveDataCache.global[1].value.toFixed(2) : '--';
    document.getElementById('sent-bias').textContent = `${analysis.bias} (${analysis.confidence}%)`;
    document.getElementById('sent-rsi').textContent = analysis.rsi ? analysis.rsi.toFixed(1) : '--';
    document.getElementById('sent-macd').textContent = analysis.macd ? (analysis.macd > 0 ? '+' : '') + analysis.macd.toFixed(2) : '--';
    document.getElementById('sent-vwap').textContent = analysis.vwap ? (latestCandle.close > analysis.vwap ? 'ABOVE' : 'BELOW') : '--';
  }

  function updateStatus(mode) {
    const dot = document.getElementById('status-dot');
    const text = document.getElementById('status-text');
    const btn = document.getElementById('mode-toggle');
    if (mode === 'live') {
      dot.className = 'dot live';
      text.textContent = 'LIVE DATA';
      btn.textContent = 'SWITCH TO DEMO';
    } else if (mode === 'error') {
      dot.className = 'dot error';
      text.textContent = 'API ERROR';
      btn.textContent = 'RETRY LIVE';
    } else {
      dot.className = 'dot demo';
      text.textContent = 'DEMO MODE';
      btn.textContent = 'SWITCH TO LIVE';
    }
  }

  // ===================== WIDGET RENDERERS =====================
  const Renderers = {
    indianTopVolume(widget) {
      const data = liveDataCache.indian && dataMode === 'live'
        ? liveDataCache.indian.sort((a, b) => b.volume - a.volume).slice(0, 10)
        : DataAdapter.getIndianTopVolume();
      let html = '<table><thead><tr><th>Symbol</th><th>Price</th><th>Vol</th><th>Chg%</th></tr></thead><tbody>';
      data.forEach(d => {
        const cls = d.change >= 0 ? 'up' : 'down';
        const sign = d.change >= 0 ? '+' : '';
        const volStr = d.volume > 1000000 ? (d.volume/1000000).toFixed(2) + 'M' : (d.volume/1000).toFixed(1) + 'K';
        html += `<tr><td><strong>${d.symbol}</strong><br><span class="muted">${d.name}</span></td><td>₹${typeof d.price === 'number' ? d.price.toFixed(2) : d.price}</td><td>${volStr}</td><td class="${cls}">${sign}${typeof d.change === 'number' ? d.change.toFixed(2) : d.change}%</td></tr>`;
      });
      html += '</tbody></table>';
      widget.body.innerHTML = html;
    },

    indianTopGainers(widget) {
      const data = liveDataCache.indian && dataMode === 'live'
        ? liveDataCache.indian.sort((a, b) => b.change - a.change).slice(0, 10)
        : DataAdapter.getIndianTopGainers();
      let html = '<table><thead><tr><th>Symbol</th><th>Price</th><th>Chg%</th><th>Vol</th></tr></thead><tbody>';
      data.forEach(d => {
        const cls = d.change >= 0 ? 'up' : 'down';
        const sign = d.change >= 0 ? '+' : '';
        const volStr = d.volume > 1000000 ? (d.volume/1000000).toFixed(2) + 'M' : (d.volume/1000).toFixed(1) + 'K';
        html += `<tr><td><strong>${d.symbol}</strong><br><span class="muted">${d.name}</span></td><td>₹${typeof d.price === 'number' ? d.price.toFixed(2) : d.price}</td><td class="${cls}">${sign}${typeof d.change === 'number' ? d.change.toFixed(2) : d.change}%</td><td>${volStr}</td></tr>`;
      });
      html += '</tbody></table>';
      widget.body.innerHTML = html;
    },

    globalIndices(widget) {
      const data = liveDataCache.global && dataMode === 'live'
        ? liveDataCache.global
        : DataAdapter.getGlobalIndices();
      let html = '<div class="indices-row">';
      data.forEach(d => {
        const cls = d.change >= 0 ? 'up' : 'down';
        const sign = d.change >= 0 ? '+' : '';
        html += `<div class="index-pill"><div class="index-name">${d.name}</div><div class="index-value">${typeof d.value === 'number' ? d.value.toLocaleString('en-IN', {maximumFractionDigits: 2}) : d.value}</div><div class="index-change ${cls}">${sign}${typeof d.change === 'number' ? d.change.toFixed(2) : d.change}%</div></div>`;
      });
      html += '</div>';
      widget.body.innerHTML = html;
    },

    sectorHeatmap(widget) {
      const data = DataAdapter.getSectors();
      let html = '<div class="heatmap-grid">';
      data.forEach(d => {
        const intensity = Math.min(Math.abs(d.change) / 3, 1);
        const bg = d.change >= 0 ? `rgba(0, ${Math.floor(212*intensity)}, ${Math.floor(170*intensity)}, ${0.15 + intensity*0.25})` : `rgba(${Math.floor(255*intensity)}, ${Math.floor(85*intensity)}, ${Math.floor(85*intensity)}, ${0.15 + intensity*0.25})`;
        const sign = d.change >= 0 ? '+' : '';
        html += `<div class="heatmap-cell" style="background:${bg}; border: 1px solid ${d.change>=0?'rgba(0,212,170,0.3)':'rgba(255,85,85,0.3)'}"><div class="hm-symbol">${d.symbol}</div><div class="hm-change ${d.change>=0?'up':'down'}">${sign}${d.change.toFixed(2)}%</div></div>`;
      });
      html += '</div>';
      widget.body.innerHTML = html;
    },

    aaplChart(widget) {
      const canvas = document.createElement('canvas');
      canvas.className = 'chart-canvas';
      widget.body.innerHTML = '';
      widget.body.appendChild(canvas);
      widget.canvas = canvas;

      const resizeObserver = new ResizeObserver(() => drawChart(widget, 'aapl'));
      resizeObserver.observe(widget.body);
      widget._observer = resizeObserver;

      drawChart(widget, 'aapl');
    },

    nifty15m(widget) {
      const canvas = document.createElement('canvas');
      canvas.className = 'chart-canvas';
      widget.body.innerHTML = '';
      widget.body.appendChild(canvas);
      widget.canvas = canvas;

      const resizeObserver = new ResizeObserver(() => drawChart(widget, 'nifty'));
      resizeObserver.observe(widget.body);
      widget._observer = resizeObserver;

      drawChart(widget, 'nifty');
    },

    preciousMetals(widget) {
      const data = liveDataCache.metals && dataMode === 'live'
        ? liveDataCache.metals
        : DataAdapter.getMetals();
      let html = '';
      data.forEach(m => {
        const cls = m.change >= 0 ? 'up' : 'down';
        const sign = m.change >= 0 ? '+' : '';
        html += `<div class="metal-card"><div class="metal-info"><div class="metal-name">${m.name} (${m.symbol})</div><div class="metal-price">$${typeof m.price === 'number' ? m.price.toFixed(m.price < 10 ? 2 : 2) : m.price}</div><div class="metal-change ${cls}">${sign}${typeof m.change === 'number' ? m.change.toFixed(2) : m.change}%</div></div><canvas class="sparkline" id="spark-${m.symbol.replace('=','')}"></canvas></div>`;
      });
      widget.body.innerHTML = html;
      setTimeout(() => {
        data.forEach(m => {
          const c = document.getElementById(`spark-${m.symbol.replace('=','')}`);
          if (c) drawSparkline(c, m.change >= 0);
        });
      }, 10);
    },

    worldClocks(widget) {
      widget.body.innerHTML = '<div class="clock-grid" id="clock-grid"></div>';
      updateClocks(widget);
    },

    technicalAnalysis(widget) {
      const analysis = technicalData.niftyAnalysis;
      const candles = technicalData.nifty;
      let html = '';

      if (!analysis || !candles) {
        html = '<div style="text-align:center;padding:20px;color:var(--text-dim);">Switch to LIVE mode to fetch real-time technical analysis for NIFTY 50.<br><br><button onclick="toggleMode()" style="background:var(--accent);color:var(--bg);border:none;padding:6px 16px;border-radius:3px;cursor:pointer;font-family:inherit;font-size:11px;font-weight:bold;">SWITCH TO LIVE</button></div>';
        widget.body.innerHTML = html;
        return;
      }

      const latest = candles[candles.length - 1];
      const prev = candles[candles.length - 2];

      html += `<div class="bias-meter">`;
      html += `<span class="bias-label">15-MIN BIAS</span>`;
      html += `<span class="bias-arrow" style="color:${analysis.bias.includes('BULL') ? 'var(--green)' : analysis.bias.includes('BEAR') ? 'var(--red)' : 'var(--yellow)'}">${analysis.bias.includes('BULL') ? '▲' : analysis.bias.includes('BEAR') ? '▼' : '◆'}</span>`;
      html += `<span class="bias-value" style="color:${analysis.bias.includes('BULL') ? 'var(--green)' : analysis.bias.includes('BEAR') ? 'var(--red)' : 'var(--yellow)'}">${analysis.bias}</span>`;
      html += `<span class="bias-label">Confidence: ${analysis.confidence}%</span>`;
      html += `</div>`;

      html += `<div class="ta-grid">`;

      // RSI
      const rsiColor = analysis.rsi > 70 ? 'var(--red)' : analysis.rsi < 30 ? 'var(--green)' : 'var(--text)';
      const rsiSignal = analysis.rsi > 70 ? 'OVERBOUGHT' : analysis.rsi < 30 ? 'OVERSOLD' : analysis.rsi > 50 ? 'BULLISH' : 'BEARISH';
      const rsiClass = analysis.rsi > 70 ? 'sell' : analysis.rsi < 30 ? 'buy' : 'neutral';
      html += `<div class="ta-card"><div class="ta-label">RSI (14)</div><div class="ta-value" style="color:${rsiColor}">${analysis.rsi.toFixed(1)}</div><div class="ta-signal ${rsiClass}">${rsiSignal}</div><div class="ta-bar-container"><div class="ta-bar" style="width:${Math.min(analysis.rsi, 100)}%; background:${analysis.rsi > 70 ? 'var(--red)' : analysis.rsi < 30 ? 'var(--green)' : 'var(--yellow)'};"></div></div></div>`;

      // MACD
      const macdColor = analysis.macd > 0 ? 'var(--green)' : 'var(--red)';
      const macdSignal = analysis.macd > 0 && analysis.macd > (technicalData.niftyAnalysis.macdPrev || 0) ? 'BULLISH' : analysis.macd < 0 ? 'BEARISH' : 'NEUTRAL';
      const macdClass = analysis.macd > 0 ? 'buy' : 'sell';
      html += `<div class="ta-card"><div class="ta-label">MACD Histogram</div><div class="ta-value" style="color:${macdColor}">${analysis.macd > 0 ? '+' : ''}${analysis.macd.toFixed(2)}</div><div class="ta-signal ${macdClass}">${macdSignal}</div></div>`;

      // VWAP
      const vwapDiff = ((latest.close - analysis.vwap) / analysis.vwap * 100);
      const vwapColor = vwapDiff > 0 ? 'var(--green)' : 'var(--red)';
      html += `<div class="ta-card"><div class="ta-label">VWAP</div><div class="ta-value" style="color:${vwapColor}">${analysis.vwap.toFixed(2)}</div><div class="ta-signal ${vwapDiff > 0 ? 'buy' : 'sell'}">${vwapDiff > 0 ? 'ABOVE' : 'BELOW'} VWAP</div></div>`;

      // SMA20
      const smaDiff = ((latest.close - analysis.sma20) / analysis.sma20 * 100);
      const smaColor = smaDiff > 0 ? 'var(--green)' : 'var(--red)';
      html += `<div class="ta-card"><div class="ta-label">SMA (20)</div><div class="ta-value" style="color:${smaColor}">${analysis.sma20.toFixed(2)}</div><div class="ta-signal ${smaDiff > 0 ? 'buy' : 'sell'}">${smaDiff > 0 ? 'ABOVE' : 'BELOW'} SMA20</div></div>`;

      html += `</div>`;

      // Signals list
      html += `<div style="margin-top:8px;padding:6px;background:rgba(0,0,0,0.2);border:1px solid var(--border);border-radius:3px;">`;
      html += `<div style="font-size:9px;color:var(--text-dim);text-transform:uppercase;margin-bottom:4px;">Active Signals</div>`;
      analysis.signals.forEach(sig => {
        html += `<div style="font-size:10px;padding:2px 0;color:var(--text);">• ${sig}</div>`;
      });
      html += `</div>`;

      // 15-min prediction
      html += `<div style="margin-top:8px;padding:8px;background:rgba(0,0,0,0.2);border:1px solid var(--border);border-radius:3px;text-align:center;">`;
      html += `<div style="font-size:9px;color:var(--text-dim);text-transform:uppercase;margin-bottom:4px;">Next 15-Min Prediction</div>`;
      html += `<div style="font-size:14px;font-weight:bold;color:${analysis.bias.includes('BULL') ? 'var(--green)' : analysis.bias.includes('BEAR') ? 'var(--red)' : 'var(--yellow)'}">${analysis.bias}</div>`;
      html += `<div style="font-size:9px;color:var(--text-dim);margin-top:4px;">Based on RSI, MACD, VWAP, SMA20, EMA9 cross</div>`;
      html += `<div style="font-size:9px;color:var(--text-dim);margin-top:2px;">Confidence: ${analysis.confidence}% | Score: ${analysis.score > 0 ? '+' : ''}${analysis.score.toFixed(1)}</div>`;
      html += `</div>`;

      widget.body.innerHTML = html;
    }
  };

  // ===================== CANVAS DRAWING =====================
  function drawChart(widget, type) {
    const canvas = widget.canvas;
    if (!canvas) return;
    const rect = widget.body.getBoundingClientRect();
    canvas.width = rect.width * window.devicePixelRatio;
    canvas.height = rect.height * window.devicePixelRatio;
    canvas.style.width = rect.width + 'px';
    canvas.style.height = rect.height + 'px';
    const ctx = canvas.getContext('2d');
    ctx.scale(window.devicePixelRatio, window.devicePixelRatio);

    let data, title, subtitle;
    if (type =
    == 'aapl') {
      data = technicalData.aapl && dataMode === 'live' ? technicalData.aapl : DataAdapter.generateAAPLData();
      title = 'AAPL — Daily Candlestick';
      subtitle = dataMode === 'live' ? 'Live Yahoo Finance | 3 Months' : 'Demo Data | 60 Sessions';
    } else {
      data = technicalData.nifty && dataMode === 'live' ? technicalData.nifty : DataAdapter.generateAAPLData();
      title = 'NIFTY 50 — 15-Min Candlestick';
      subtitle = dataMode === 'live' ? 'Live Yahoo Finance | 5 Days' : 'Demo Data | Switch to LIVE';
    }

    if (!data || data.length === 0) return;

    const w = rect.width, h = rect.height;
    const pad = { top: 32, right: 50, bottom: 32, left: 8 };
    const cw = w - pad.left - pad.right;
    const ch = h - pad.top - pad.bottom;

    const prices = data.map(d => d.close);
    const minP = Math.min(...data.map(d => d.low)) * 0.998;
    const maxP = Math.max(...data.map(d => d.high)) * 1.002;
    const range = maxP - minP;

    // Background
    ctx.fillStyle = '#111118';
    ctx.fillRect(0, 0, w, h);

    // Draw indicators if live NIFTY
    if (type === 'nifty' && dataMode === 'live' && technicalData.nifty) {
      const candles = technicalData.nifty;
      // SMA20
      const sma20 = Indicators.sma(candles, 20);
      if (sma20.length > 0) {
        ctx.strokeStyle = 'rgba(88,166,255,0.6)';
        ctx.lineWidth = 1;
        ctx.beginPath();
        sma20.forEach((p, i) => {
          const x = pad.left + ((candles.length - sma20.length + i) / (candles.length - 1)) * cw;
          const y = pad.top + ((maxP - p.value) / range) * ch;
          if (i === 0) ctx.moveTo(x, y); else ctx.lineTo(x, y);
        });
        ctx.stroke();
      }
      // VWAP
      const vwap = Indicators.vwap(candles);
      if (vwap.length > 0) {
        ctx.strokeStyle = 'rgba(188,140,255,0.6)';
        ctx.lineWidth = 1;
        ctx.setLineDash([4, 4]);
        ctx.beginPath();
        vwap.forEach((p, i) => {
          const x = pad.left + (i / (candles.length - 1)) * cw;
          const y = pad.top + ((maxP - p.value) / range) * ch;
          if (i === 0) ctx.moveTo(x, y); else ctx.lineTo(x, y);
        });
        ctx.stroke();
        ctx.setLineDash([]);
      }
    }

    // Grid
    ctx.strokeStyle = 'rgba(31,31,46,0.6)';
    ctx.lineWidth = 0.5;
    for (let i = 0; i <= 4; i++) {
      const y = pad.top + (ch / 4) * i;
      ctx.beginPath(); ctx.moveTo(pad.left, y); ctx.lineTo(w - pad.right, y); ctx.stroke();
      const price = maxP - (range / 4) * i;
      ctx.fillStyle = '#6e7681';
      ctx.font = '8px monospace';
      ctx.textAlign = 'right';
      ctx.fillText(type === 'nifty' ? price.toFixed(1) : '$' + price.toFixed(2), w - 4, y + 3);
    }

    // Candlesticks
    const barW = Math.max(cw / data.length * 0.6, 2);
    const spacing = cw / data.length;

    data.forEach((d, i) => {
      const x = pad.left + i * spacing + spacing * 0.2;
      const yO = pad.top + ((maxP - d.open) / range) * ch;
      const yC = pad.top + ((maxP - d.close) / range) * ch;
      const yH = pad.top + ((maxP - d.high) / range) * ch;
      const yL = pad.top + ((maxP - d.low) / range) * ch;
      const isUp = d.close >= d.open;

      ctx.strokeStyle = isUp ? '#00d4aa' : '#ff5555';
      ctx.fillStyle = isUp ? '#00d4aa' : '#ff5555';
      ctx.lineWidth = 1;

      ctx.beginPath(); ctx.moveTo(x + barW/2, yH); ctx.lineTo(x + barW/2, yL); ctx.stroke();
      const bodyH = Math.max(Math.abs(yC - yO), 1);
      ctx.fillRect(x, Math.min(yO, yC), barW, bodyH);
    });

    // Volume bars
    const maxVol = Math.max(...data.map(d => d.volume || 0));
    const volH = 36;
    if (maxVol > 0) {
      data.forEach((d, i) => {
        const x = pad.left + i * spacing + spacing * 0.2;
        const vh = ((d.volume || 0) / maxVol) * volH;
        const isUp = d.close >= d.open;
        ctx.fillStyle = isUp ? 'rgba(0,212,170,0.25)' : 'rgba(255,85,85,0.25)';
        ctx.fillRect(x, h - pad.bottom - vh, barW, vh);
      });
    }

    // Title
    ctx.fillStyle = '#00d4aa';
    ctx.font = 'bold 10px monospace';
    ctx.textAlign = 'left';
    ctx.fillText(title, pad.left, 16);
    ctx.fillStyle = '#6e7681';
    ctx.font = '8px monospace';
    ctx.fillText(subtitle, pad.left, 26);

    // Legend for NIFTY
    if (type === 'nifty' && dataMode === 'live') {
      ctx.fillStyle = 'rgba(88,166,255,0.8)';
      ctx.fillText('SMA20', w - 80, 16);
      ctx.fillStyle = 'rgba(188,140,255,0.8)';
      ctx.fillText('VWAP', w - 40, 16);
    }

    // Crosshair
    canvas.onmousemove = (e) => {
      const r = canvas.getBoundingClientRect();
      const mx = e.clientX - r.left;
      const my = e.clientY - r.top;
      if (mx >= pad.left && mx <= w - pad.right && my >= pad.top && my <= h - pad.bottom) {
        const idx = Math.floor((mx - pad.left) / spacing);
        if (idx >= 0 && idx < data.length) {
          const d = data[idx];
          showTooltip(e.clientX, e.clientY, `${d.date.toLocaleString()}<br>O: ${d.open.toFixed(2)} H: ${d.high.toFixed(2)}<br>L: ${d.low.toFixed(2)} C: ${d.close.toFixed(2)}<br>Vol: ${((d.volume||0)/1000000).toFixed(2)}M`);
        }
      } else {
        hideTooltip();
      }
    };
    canvas.onmouseleave = hideTooltip;
  }

  function drawSparkline(canvas, isUp) {
    const rect = canvas.getBoundingClientRect();
    canvas.width = rect.width * window.devicePixelRatio;
    canvas.height = rect.height * window.devicePixelRatio;
    const ctx = canvas.getContext('2d');
    ctx.scale(window.devicePixelRatio, window.devicePixelRatio);
    const w = rect.width, h = rect.height;

    const points = [];
    let val = 50;
    for (let i = 0; i < 20; i++) {
      val += (Math.random() - (isUp ? 0.4 : 0.6)) * 10;
      points.push(val);
    }
    const min = Math.min(...points), max = Math.max(...points);
    const range = max - min || 1;

    ctx.strokeStyle = isUp ? '#00d4aa' : '#ff5555';
    ctx.lineWidth = 1.5;
    ctx.beginPath();
    points.forEach((p, i) => {
      const x = (i / (points.length - 1)) * w;
      const y = h - ((p - min) / range) * h * 0.8 - h * 0.1;
      if (i === 0) ctx.moveTo(x, y); else ctx.lineTo(x, y);
    });
    ctx.stroke();

    ctx.lineTo(w, h); ctx.lineTo(0, h); ctx.closePath();
    ctx.fillStyle = isUp ? 'rgba(0,212,170,0.1)' : 'rgba(255,85,85,0.1)';
    ctx.fill();
  }

  // ===================== CLOCKS =====================
  function updateClocks(widget) {
    const grid = widget.body.querySelector('#clock-grid');
    if (!grid) return;

    const markets = [
      { city: 'Tokyo', tz: 'Asia/Tokyo', open: 9, close: 15 },
      { city: 'Mumbai', tz: 'Asia/Kolkata', open: 9, close: 15.5 },
      { city: 'London', tz: 'Europe/London', open: 8, close: 16.5 },
      { city: 'New York', tz: 'America/New_York', open: 9.5, close: 16 }
    ];

    let html = '';
    markets.forEach(m => {
      const now = new Date();
      const timeStr = now.toLocaleTimeString('en-US', { timeZone: m.tz, hour12: false, hour: '2-digit', minute: '2-digit' });
      const hour = parseInt(now.toLocaleTimeString('en-US', { timeZone: m.tz, hour12: false, hour: 'numeric' }));
      const minute = parseInt(now.toLocaleTimeString('en-US', { timeZone: m.tz, hour12: false, minute: 'numeric' }));
      const decimalTime = hour + minute / 60;

      let status = 'closed', statusText = 'CLOSED';
      if (decimalTime >= m.open && decimalTime < m.close) { status = 'open'; statusText = 'OPEN'; }
      else if (Math.abs(decimalTime - m.open) < 0.5) { status = 'pre'; statusText = 'PRE-OPEN'; }

      html += `<div class="clock-item"><div class="clock-city">${m.city}</div><div class="clock-time">${timeStr}</div><div class="clock-status ${status}">${statusText}</div></div>`;
    });
    grid.innerHTML = html;
  }

  // ===================== TOOLTIP =====================
  function showTooltip(x, y, html) {
    tooltip.innerHTML = html;
    tooltip.style.display = 'block';
    tooltip.style.left = (x + 12) + 'px';
    tooltip.style.top = (y + 12) + 'px';
  }
  function hideTooltip() { tooltip.style.display = 'none'; }

  // ===================== WIDGET SYSTEM =====================
  function createWidget(type, x, y, w, h) {
    const id = 'widget-' + nextId++;
    const el = document.createElement('div');
    el.className = 'widget';
    el.id = id;
    el.style.left = x + 'px';
    el.style.top = y + 'px';
    el.style.width = w + 'px';
    el.style.height = h + 'px';

    const titles = {
      indianTopVolume: 'INDIAN TOP VOLUME',
      indianTopGainers: 'INDIAN TOP GAINERS',
      globalIndices: 'GLOBAL INDICES',
      sectorHeatmap: 'SECTOR HEATMAP',
      aaplChart: 'AAPL CHART',
      nifty15m: 'NIFTY 50 — 15 MIN',
      preciousMetals: 'PRECIOUS METALS',
      worldClocks: 'WORLD SESSIONS',
      technicalAnalysis: 'TECHNICAL ANALYSIS'
    };

    el.innerHTML = `
      <div class="widget-header" data-id="${id}">
        <div class="widget-title">${titles[type] || type.toUpperCase()}</div>
        <div class="widget-controls">
          <button class="widget-btn" onclick="refreshWidget('${id}')">↻</button>
          <button class="widget-btn" onclick="removeWidget('${id}')">×</button>
        </div>
      </div>
      <div class="widget-body"></div>
      <div class="resize-handle" data-id="${id}"></div>
    `;

    dashboard.appendChild(el);

    const widget = {
      id, type, el,
      header: el.querySelector('.widget-header'),
      body: el.querySelector('.widget-body'),
      handle: el.querySelector('.resize-handle'),
      x, y, w, h
    };

    widget.header.addEventListener('mousedown', (e) => {
      if (e.target.closest('.widget-btn')) return;
      isDragging = true;
      dragWidget = widget;
      dragOffset.x = e.clientX - widget.el.offsetLeft;
      dragOffset.y = e.clientY - widget.el.offsetTop;
      widget.el.classList.add('dragging');
      e.preventDefault();
    });

    widget.handle.addEventListener('mousedown', (e) => {
      isResizing = true;
      resizeWidget = widget;
      resizeStart.x = e.clientX;
      resizeStart.y = e.clientY;
      resizeStart.w = widget.el.offsetWidth;
      resizeStart.h = widget.el.offsetHeight;
      e.preventDefault();
      e.stopPropagation();
    });

    widgets.push(widget);
    refreshWidget(id);
    return widget;
  }

  function refreshWidget(id) {
    const widget = widgets.find(w => w.id === id);
    if (!widget) return;
    const renderer = Renderers[widget.type];
    if (renderer) renderer(widget);
  }

  function removeWidget(id) {
    const idx = widgets.findIndex(w => w.id === id);
    if (idx === -1) return;
    const widget = widgets[idx];
    if (widget._observer) widget._observer.disconnect();
    widget.el.remove();
    widgets.splice(idx, 1);
    saveLayout();
  }

  function addWidget(type) {
    const existing = widgets.filter(w => w.type === type).length;
    const x = 12 + existing * 24;
    const y = 12 + existing * 24;
    const defaults = {
      indianTopVolume: [320,260], indianTopGainers: [320,260], globalIndices: [440,140],
      sectorHeatmap: [460,240], aaplChart: [520,300], nifty15m: [520,260],
      preciousMetals: [280,180], worldClocks: [280,180], technicalAnalysis: [340,260]
    };
    const [w, h] = defaults[type] || [300, 200];
    createWidget(type, x, y, w, h);
    saveLayout();
    document.getElementById('add-menu').classList.remove('show');
  }

  // ===================== LAYOUT PERSISTENCE =====================
  function saveLayout() {
    const layout = widgets.map(w => ({ type: w.type, x: w.x, y: w.y, w: w.w, h: w.h }));
    try { localStorage.setItem('indra_layout', JSON.stringify(layout)); } catch(e) {}
  }

  function loadLayout() {
    try {
      const saved = localStorage.getItem('indra_layout');
      if (saved) {
        const layout = JSON.parse(saved);
        layout.forEach(l => createWidget(l.type, l.x, l.y, l.w, l.h));
        return true;
      }
    } catch(e) {}
    return false;
  }

  function resetLayout() {
    widgets.forEach(w => { if (w._observer) w._observer.disconnect(); w.el.remove(); });
    widgets = [];
    try { localStorage.removeItem('indra_layout'); } catch(e) {}
    CONFIG.defaultWidgets.forEach(w => createWidget(w.type, w.x, w.y, w.w, w.h));
  }

  // ===================== GLOBAL EVENTS =====================
  document.addEventListener('mousemove', (e) => {
    if (isDragging && dragWidget) {
      let nx = e.clientX - dragOffset.x;
      let ny = e.clientY - dragOffset.y;
      nx = Math.max(0, nx);
      ny = Math.max(0, ny);
      dragWidget.el.style.left = nx + 'px';
      dragWidget.el.style.top = ny + 'px';
      dragWidget.x = nx;
      dragWidget.y = ny;
    }
    if (isResizing && resizeWidget) {
      const nw = Math.max(200, resizeStart.w + (e.clientX - resizeStart.x));
      const nh = Math.max(120, resizeStart.h + (e.clientY - resizeStart.y));
      resizeWidget.el.style.width = nw + 'px';
      resizeWidget.el.style.height = nh + 'px';
      resizeWidget.w = nw;
      resizeWidget.h = nh;
    }
  });

  document.addEventListener('mouseup', () => {
    if (isDragging && dragWidget) {
      dragWidget.el.classList.remove('dragging');
      saveLayout();
    }
    if (isResizing && resizeWidget) {
      refreshWidget(resizeWidget.id);
      saveLayout();
    }
    isDragging = false;
    isResizing = false;
    dragWidget = null;
    resizeWidget = null;
  });

  // ===================== MODE TOGGLE =====================
  window.toggleMode = function() {
    dataMode = dataMode === 'demo' ? 'live' : 'demo';
    CONFIG.demoMode = dataMode === 'demo';
    if (dataMode === 'live') {
      updateStatus('live');
      updateLiveData();
    } else {
      updateStatus('demo');
      sentimentBanner.className = 'neutral';
      document.getElementById('sent-nifty').textContent = '--';
      document.getElementById('sent-sensex').textContent = '--';
      document.getElementById('sent-bias').textContent = 'NEUTRAL';
      document.getElementById('sent-rsi').textContent = '--';
      document.getElementById('sent-macd').textContent = '--';
      document.getElementById('sent-vwap').textContent = '--';
    }
    widgets.forEach(w => refreshWidget(w.id));
  };

  window.toggleAddMenu = function() {
    document.getElementById('add-menu').classList.toggle('show');
  };

  window.toggleApiModal = function() {
    document.getElementById('api-modal').classList.toggle('show');
  };

  window.saveApiConfig = function() {
    CONFIG.alphaVantageKey = document.getElementById('av-key').value.trim();
    CONFIG.dataSource = document.getElementById('data-source').value;
    try {
      localStorage.setItem('indra_api_key', CONFIG.alphaVantageKey);
      localStorage.setItem('indra_data_source', CONFIG.dataSource);
    } catch(e) {}
    toggleApiModal();
    if (dataMode === 'live') updateLiveData();
  };

  document.addEventListener('click', (e) => {
    if (!e.target.closest('#add-menu') && !e.target.closest('#add-widget-btn')) {
      document.getElementById('add-menu').classList.remove('show');
    }
    if (!e.target.closest('#api-modal') && !e.target.closest('#api-btn')) {
      document.getElementById('api-modal').classList.remove('show');
    }
  });

  // ===================== CLOCK =====================
  function updateHeaderClock() {
    const now = new Date();
    document.getElementById('clock').textContent = now.toLocaleTimeString('en-IN', { hour12: false });
  }
  setInterval(updateHeaderClock, 1000);
  updateHeaderClock();

  // ===================== INIT =====================
  function init() {
    // Load API config
    try {
      const savedKey = localStorage.getItem('indra_api_key');
      const savedSource = localStorage.getItem('indra_data_source');
      if (savedKey) { CONFIG.alphaVantageKey = savedKey; document.getElementById('av-key').value = savedKey; }
      if (savedSource) { CONFIG.dataSource = savedSource; document.getElementById('data-source').value = savedSource; }
    } catch(e) {}

    if (!loadLayout()) {
      CONFIG.defaultWidgets.forEach(w => createWidget(w.type, w.x, w.y, w.w, w.h));
    }

    // Periodic refresh
    updateTimer = setInterval(() => {
      if (dataMode === 'live') {
        updateLiveData();
      } else {
        widgets.forEach(w => {
          if (w.type !== 'aaplChart' && w.type !== 'nifty15m' && w.type !== 'worldClocks' && w.type !== 'technicalAnalysis') {
            refreshWidget(w.id);
          }
        });
      }
    }, CONFIG.refreshInterval);

    setInterval(() => {
      widgets.filter(w => w.type === 'worldClocks').forEach(w => updateClocks(w));
    }, 1000);

    setTimeout(() => {
      document.getElementById('loading').classList.add('hidden');
    }, 1500);
  }

  // Expose globals
  window.refreshWidget = refreshWidget;
  window.removeWidget = removeWidget;
  window.addWidget = addWidget;
  window.resetLayout = resetLayout;

  if (document.readyState === 'loading') {
    document.addEventListener('DOMContentLoaded', init);
  } else {
    init();
  }

})();
</script>
</body>
</html>
