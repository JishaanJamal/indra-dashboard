# index.html
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
    --font-mono: 'SFMono-Regular', 'Menlo', 'Monaco', 'Consolas', 'Liberation Mono', 'Courier New', monospace;
  }

  * { box-sizing: border-box; margin: 0; padding: 0; }

  body {
    background: var(--bg);
    color: var(--text);
    font-family: var(--font-mono);
    font-size: 12px;
    line-height: 1.5;
    overflow: hidden;
    height: 100vh;
    width: 100vw;
  }

  ::-webkit-scrollbar { width: 6px; height: 6px; }
  ::-webkit-scrollbar-track { background: var(--bg); }
  ::-webkit-scrollbar-thumb { background: var(--border-bright); border-radius: 3px; }
  ::-webkit-scrollbar-thumb:hover { background: var(--accent-dim); }

  #header {
    height: 40px;
    background: var(--bg-panel);
    border-bottom: 1px solid var(--border);
    display: flex;
    align-items: center;
    justify-content: space-between;
    padding: 0 16px;
    position: relative;
    z-index: 1000;
    flex-shrink: 0;
  }

  #header .logo {
    display: flex;
    align-items: center;
    gap: 8px;
    color: var(--accent);
    font-weight: bold;
    font-size: 13px;
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
    gap: 16px;
    color: var(--text-dim);
  }

  #header .status span { display: flex; align-items: center; gap: 6px; }

  .dot {
    width: 7px;
    height: 7px;
    border-radius: 50%;
    display: inline-block;
  }
  .dot.live { background: var(--green); box-shadow: 0 0 6px var(--green); }
  .dot.demo { background: var(--yellow); box-shadow: 0 0 6px var(--yellow); }

  #mode-toggle {
    background: var(--bg);
    border: 1px solid var(--border);
    color: var(--text);
    padding: 3px 10px;
    cursor: pointer;
    font-family: inherit;
    font-size: 11px;
    border-radius: 3px;
    transition: all 0.2s;
  }
  #mode-toggle:hover { border-color: var(--accent); color: var(--accent); }

  #dashboard {
    position: relative;
    width: 100%;
    height: calc(100vh - 40px);
    overflow: auto;
    background:
      linear-gradient(rgba(0,212,170,0.03) 1px, transparent 1px),
      linear-gradient(90deg, rgba(0,212,170,0.03) 1px, transparent 1px);
    background-size: 20px 20px;
  }

  .widget {
    position: absolute;
    background: var(--bg-panel);
    border: 1px solid var(--border);
    border-radius: 4px;
    overflow: hidden;
    display: flex;
    flex-direction: column;
    min-width: 240px;
    min-height: 160px;
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
    height: 28px;
    background: linear-gradient(90deg, var(--bg-panel), var(--bg-panel-hover));
    border-bottom: 1px solid var(--border);
    display: flex;
    align-items: center;
    justify-content: space-between;
    padding: 0 10px;
    cursor: grab;
    user-select: none;
    flex-shrink: 0;
  }

  .widget-header:active { cursor: grabbing; }

  .widget-title {
    color: var(--accent);
    font-size: 11px;
    font-weight: bold;
    letter-spacing: 0.5px;
    display: flex;
    align-items: center;
    gap: 6px;
  }

  .widget-title::before {
    content: '■';
    font-size: 8px;
    color: var(--accent-dim);
  }

  .widget-controls {
    display: flex;
    gap: 6px;
  }

  .widget-btn {
    background: none;
    border: none;
    color: var(--text-dim);
    cursor: pointer;
    font-family: inherit;
    font-size: 10px;
    padding: 2px 4px;
    border-radius: 2px;
    transition: color 0.2s;
  }
  .widget-btn:hover { color: var(--accent); }

  .widget-body {
    flex: 1;
    overflow: auto;
    padding: 8px;
    position: relative;
  }

  .resize-handle {
    position: absolute;
    bottom: 0;
    right: 0;
    width: 14px;
    height: 14px;
    cursor: se-resize;
    background: linear-gradient(135deg, transparent 50%, var(--border-bright) 50%);
    border-bottom-right-radius: 3px;
    opacity: 0.5;
    transition: opacity 0.2s;
  }
  .resize-handle:hover { opacity: 1; background: linear-gradient(135deg, transparent 50%, var(--accent) 50%); }

  table {
    width: 100%;
    border-collapse: collapse;
    font-size: 11px;
  }

  th {
    text-align: left;
    padding: 4px 6px;
    color: var(--text-dim);
    border-bottom: 1px solid var(--border);
    font-weight: normal;
    font-size: 10px;
    text-transform: uppercase;
    letter-spacing: 0.5px;
    position: sticky;
    top: 0;
    background: var(--bg-panel);
  }

  td {
    padding: 5px 6px;
    border-bottom: 1px solid rgba(31,31,46,0.5);
    color: var(--text);
  }

  tr:hover td { background: rgba(0,212,170,0.05); }

  .up { color: var(--green); }
  .down { color: var(--red); }
  .muted { color: var(--text-dim); }

  canvas.chart-canvas {
    width: 100%;
    height: 100%;
    display: block;
  }

  .heatmap-grid {
    display: grid;
    grid-template-columns: repeat(auto-fill, minmax(80px, 1fr));
    gap: 3px;
    height: 100%;
  }

  .heatmap-cell {
    display: flex;
    flex-direction: column;
    align-items: center;
    justify-content: center;
    padding: 4px;
    border-radius: 2px;
    font-size: 10px;
    text-align: center;
    cursor: pointer;
    transition: transform 0.1s;
    position: relative;
    overflow: hidden;
  }

  .heatmap-cell:hover { transform: scale(1.05); z-index: 2; }
  .heatmap-cell .hm-symbol { font-weight: bold; font-size: 10px; margin-bottom: 2px; }
  .heatmap-cell .hm-change { font-size: 9px; }

  .clock-grid {
    display: grid;
    grid-template-columns: repeat(2, 1fr);
    gap: 8px;
    height: 100%;
  }

  .clock-item {
    background: rgba(0,0,0,0.3);
    border: 1px solid var(--border);
    border-radius: 3px;
    padding: 8px;
    display: flex;
    flex-direction: column;
    align-items: center;
    justify-content: center;
    text-align: center;
  }

  .clock-city { font-size: 10px; color: var(--accent); margin-bottom: 4px; }
  .clock-time { font-size: 18px; font-weight: bold; color: var(--text); }
  .clock-status { font-size: 9px; margin-top: 4px; padding: 1px 6px; border-radius: 2px; }
  .clock-status.open { background: rgba(0,212,170,0.15); color: var(--green); }
  .clock-status.closed { background: rgba(255,85,85,0.15); color: var(--red); }
  .clock-status.pre { background: rgba(240,173,78,0.15); color: var(--yellow); }

  .indices-row {
    display: flex;
    gap: 12px;
    flex-wrap: wrap;
  }

  .index-pill {
    background: rgba(0,0,0,0.3);
    border: 1px solid var(--border);
    border-radius: 3px;
    padding: 6px 10px;
    min-width: 120px;
  }

  .index-name { font-size: 10px; color: var(--text-dim); margin-bottom: 2px; }
  .index-value { font-size: 14px; font-weight: bold; }
  .index-change { font-size: 10px; margin-top: 2px; }

  .metal-card {
    display: flex;
    align-items: center;
    justify-content: space-between;
    padding: 8px;
    background: rgba(0,0,0,0.2);
    border: 1px solid var(--border);
    border-radius: 3px;
    margin-bottom: 6px;
  }

  .metal-info { display: flex; flex-direction: column; }
  .metal-name { font-size: 11px; color: var(--text-dim); }
  .metal-price { font-size: 14px; font-weight: bold; color: var(--text); }
  .metal-change { font-size: 10px; }

  .sparkline {
    width: 60px;
    height: 24px;
  }

  #tooltip {
    position: fixed;
    background: var(--bg-panel);
    border: 1px solid var(--accent);
    padding: 6px 10px;
    border-radius: 3px;
    font-size: 11px;
    pointer-events: none;
    z-index: 10000;
    display: none;
    box-shadow: 0 4px 20px rgba(0,0,0,0.8);
    max-width: 200px;
  }

  #loading {
    position: fixed;
    inset: 0;
    background: var(--bg);
    display: flex;
    flex-direction: column;
    align-items: center;
    justify-content: center;
    z-index: 20000;
    gap: 16px;
  }

  #loading.hidden { display: none; }

  .loader-text {
    color: var(--accent);
    font-size: 12px;
    letter-spacing: 2px;
    animation: blink 1s infinite;
  }

  @keyframes blink { 0%, 100% { opacity: 1; } 50% { opacity: 0.3; } }

  .loader-bar {
    width: 200px;
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

  #add-menu {
    position: fixed;
    top: 40px;
    right: 10px;
    background: var(--bg-panel);
    border: 1px solid var(--border);
    border-radius: 4px;
    padding: 8px;
    z-index: 10001;
    display: none;
    min-width: 180px;
    box-shadow: 0 8px 30px rgba(0,0,0,0.8);
  }

  #add-menu.show { display: block; }

  .menu-item {
    padding: 6px 10px;
    cursor: pointer;
    border-radius: 2px;
    font-size: 11px;
    color: var(--text);
    transition: background 0.15s;
  }
  .menu-item:hover { background: var(--accent-glow); color: var(--accent); }

  @media (max-width: 768px) {
    .widget { position: relative !important; width: 100% !important; left: 0 !important; top: auto !important; margin-bottom: 8px; }
    #dashboard { height: auto; overflow: visible; }
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
    <button id="add-widget-btn" class="widget-btn" style="border:1px solid var(--border); padding:3px 10px;" onclick="toggleAddMenu()">+ WIDGET</button>
    <button class="widget-btn" style="border:1px solid var(--border); padding:3px 10px;" onclick="resetLayout()">RESET</button>
  </div>
</div>

<div id="add-menu">
  <div class="menu-item" onclick="addWidget('indianTopVolume')">Indian Top Volume</div>
  <div class="menu-item" onclick="addWidget('indianTopGainers')">Indian Top Gainers</div>
  <div class="menu-item" onclick="addWidget('globalIndices')">Global Indices</div>
  <div class="menu-item" onclick="addWidget('sectorHeatmap')">Sector Heatmap</div>
  <div class="menu-item" onclick="addWidget('aaplChart')">AAPL 60-Session</div>
  <div class="menu-item" onclick="addWidget('preciousMetals')">Precious Metals</div>
  <div class="menu-item" onclick="addWidget('worldClocks')">World Clocks</div>
</div>

<div id="dashboard"></div>
<div id="tooltip"></div>

<script>
(function() {
  'use strict';

  const CONFIG = {
    gridSize: 20,
    defaultWidgets: [
      { type: 'indianTopVolume', x: 20, y: 20, w: 340, h: 320 },
      { type: 'indianTopGainers', x: 380, y: 20, w: 340, h: 320 },
      { type: 'globalIndices', x: 740, y: 20, w: 480, h: 160 },
      { type: 'sectorHeatmap', x: 20, y: 360, w: 500, h: 280 },
      { type: 'aaplChart', x: 540, y: 200, w: 680, h: 440 },
      { type: 'preciousMetals', x: 20, y: 660, w: 340, h: 200 },
      { type: 'worldClocks', x: 380, y: 660, w: 340, h: 200 }
    ],
    refreshInterval: 3000,
    demoMode: true
  };

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

  const dashboard = document.getElementById('dashboard');
  const tooltip = document.getElementById('tooltip');

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

  const Renderers = {
    indianTopVolume(widget) {
      const data = DataAdapter.getIndianTopVolume();
      let html = '<table><thead><tr><th>Symbol</th><th>Price</th><th>Vol</th><th>Chg%</th></tr></thead><tbody>';
      data.forEach(d => {
        const cls = d.change >= 0 ? 'up' : 'down';
        const sign = d.change >= 0 ? '+' : '';
        html += `<tr><td><strong>${d.symbol}</strong><br><span class="muted">${d.name}</span></td><td>₹${d.price.toFixed(2)}</td><td>${(d.volume/1000000).toFixed(2)}M</td><td class="${cls}">${sign}${d.change.toFixed(2)}%</td></tr>`;
      });
      html += '</tbody></table>';
      widget.body.innerHTML = html;
    },

    indianTopGainers(widget) {
      const data = DataAdapter.getIndianTopGainers();
      let html = '<table><thead><tr><th>Symbol</th><th>Price</th><th>Chg%</th><th>Vol</th></tr></thead><tbody>';
      data.forEach(d => {
        const cls = d.change >= 0 ? 'up' : 'down';
        const sign = d.change >= 0 ? '+' : '';
        html += `<tr><td><strong>${d.symbol}</strong><br><span class="muted">${d.name}</span></td><td>₹${d.price.toFixed(2)}</td><td class="${cls}">${sign}${d.change.toFixed(2)}%</td><td>${(d.volume/1000000).toFixed(2)}M</td></tr>`;
      });
      html += '</tbody></table>';
      widget.body.innerHTML = html;
    },

    globalIndices(widget) {
      const data = DataAdapter.getGlobalIndices();
      let html = '<div class="indices-row">';
      data.forEach(d => {
        const cls = d.change >= 0 ? 'up' : 'down';
        const sign = d.change >= 0 ? '+' : '';
        html += `<div class="index-pill"><div class="index-name">${d.name}</div><div class="index-value">${d.value.toLocaleString('en-IN', {maximumFractionDigits: 2})}</div><div class="index-change ${cls}">${sign}${d.change.toFixed(2)}%</div></div>`;
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
        html += `<div class="heatmap-cell" style="background:${bg}; border: 1px solid ${d.change>=0?'rgba(0,212,170,0.3)':'rgba(255,85,85,0.3)'}" title="${d.stocks.join(', ')}"><div class="hm-symbol">${d.symbol}</div><div class="hm-change ${d.change>=0?'up':'down'}">${sign}${d.change.toFixed(2)}%</div></div>`;
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

      const resizeObserver = new ResizeObserver(() => drawAAPLChart(widget));
      resizeObserver.observe(widget.body);
      widget._observer = resizeObserver;

      drawAAPLChart(widget);
    },

    preciousMetals(widget) {
      const data = DataAdapter.getMetals();
      let html = '';
      data.forEach(m => {
        const cls = m.change >= 0 ? 'up' : 'down';
        const sign = m.change >= 0 ? '+' : '';
        html += `<div class="metal-card"><div class="metal-info"><div class="metal-name">${m.name} (${m.symbol})</div><div class="metal-price">$${m.price.toFixed(m.price < 10 ? 2 : 2)}</div><div class="metal-change ${cls}">${sign}${m.change.toFixed(2)}%</div></div><canvas class="sparkline" id="spark-${m.symbol}"></canvas></div>`;
      });
      widget.body.innerHTML = html;
      setTimeout(() => {
        data.forEach(m => {
          const c = document.getElementById(`spark-${m.symbol}`);
          if (c) drawSparkline(c, m.change >= 0);
        });
      }, 10);
    },

    worldClocks(widget) {
      widget.body.innerHTML = '<div class="clock-grid" id="clock-grid"></div>';
      updateClocks(widget);
    }
  };

  function drawAAPLChart(widget) {
    const canvas = widget.canvas;
    if (!canvas) return;
    const rect = widget.body.getBoundingClientRect();
    canvas.width = rect.width * window.devicePixelRatio;
    canvas.height = rect.height * window.devicePixelRatio;
    canvas.style.width = rect.width + 'px';
    canvas.style.height = rect.height + 'px';
    const ctx = canvas.getContext('2d');
    ctx.scale(window.devicePixelRatio, window.devicePixelRatio);

    const data = DataAdapter.generateAAPLData();
    const w = rect.width, h = rect.height;
    const pad = { top: 30, right: 50, bottom: 30, left: 10 };
    const cw = w - pad.left - pad.right;
    const ch = h - pad.top - pad.bottom;

    const prices = data.map(d => d.close);
    const minP = Math.min(...prices) * 0.995;
    const maxP = Math.max(...prices) * 1.005;
    const range = maxP - minP;

    ctx.fillStyle = '#111118';
    ctx.fillRect(0, 0, w, h);

    ctx.strokeStyle = 'rgba(31,31,46,0.6)';
    ctx.lineWidth = 0.5;
    for (let i = 0; i <= 4; i++) {
      const y = pad.top + (ch / 4) * i;
      ctx.beginPath(); ctx.moveTo(pad.left, y); ctx.lineTo(w - pad.right, y); ctx.stroke();
      const price = maxP - (range / 4) * i;
      ctx.fillStyle = '#6e7681';
      ctx.font = '9px monospace';
      ctx.textAlign = 'right';
      ctx.fillText('$' + price.toFixed(2), w - 5, y + 3);
    }

    const barW = cw / data.length * 0.7;
    const spacing = cw / data.length;

    data.forEach((d, i) => {
      const x = pad.left + i * spacing + spacing * 0.15;
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

    const maxVol = Math.max(...data.map(d => d.volume));
    const volH = 40;
    data.forEach((d, i) => {
      const x = pad.left + i * spacing + spacing * 0.15;
      const vh = (d.volume / maxVol) * volH;
      const isUp = d.close >= d.open;
      ctx.fillStyle = isUp ? 'rgba(0,212,170,0.3)' : 'rgba(255,85,85,0.3)';
      ctx.fillRect(x, h - pad.bottom - vh, barW, vh);
    });

    ctx.fillStyle = '#00d4aa';
    ctx.font = 'bold 11px monospace';
    ctx.textAlign = 'left';
    ctx.fillText('AAPL — 60 Session Candlestick', pad.left, 16);
    ctx.fillStyle = '#6e7681';
    ctx.font = '9px monospace';
    ctx.fillText('Volume overlay | Demo Data', pad.left, 28);

    canvas.onmousemove = (e) => {
      const r = canvas.getBoundingClientRect();
      const mx = e.clientX - r.left;
      const my = e.clientY - r.top;
      if (mx >= pad.left && mx <= w - pad.right && my >= pad.top && my <= h - pad.bottom) {
        const idx = Math.floor((mx - pad.left) / spacing);
        if (idx >= 0 && idx < data.length) {
          const d = data[idx];
          showTooltip(e.clientX, e.clientY, `${d.date.toDateString()}<br>O: $${d.open.toFixed(2)} H: $${d.high.toFixed(2)}<br>L: $${d.low.toFixed(2)} C: $${d.close.toFixed(2)}<br>Vol: ${(d.volume/1000000).toFixed(2)}M`);
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

  function showTooltip(x, y, html) {
    tooltip.innerHTML = html;
    tooltip.style.display = 'block';
    tooltip.style.left = (x + 12) + 'px';
    tooltip.style.top = (y + 12) + 'px';
  }
  function hideTooltip() { tooltip.style.display = 'none'; }

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
      aaplChart: 'AAPL 60-SESSION',
      preciousMetals: 'PRECIOUS METALS',
      worldClocks: 'WORLD SESSIONS'
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
    const x = 20 + existing * 30;
    const y = 20 + existing * 30;
    const defaults = { indianTopVolume: [340,320], indianTopGainers: [340,320], globalIndices: [480,160], sectorHeatmap: [500,280], aaplChart: [680,440], preciousMetals: [340,200], worldClocks: [340,200] };
    const [w, h] = defaults[type] || [300, 200];
    createWidget(type, x, y, w, h);
    saveLayout();
    document.getElementById('add-menu').classList.remove('show');
  }

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

  window.toggleMode = function() {
    dataMode = dataMode === 'demo' ? 'live' : 'demo';
    CONFIG.demoMode = dataMode === 'demo';
    const dot = document.getElementById('status-dot');
    const text = document.getElementById('status-text');
    const btn = document.getElementById('mode-toggle');
    if (dataMode === 'live') {
      dot.className = 'dot live';
      text.textContent = 'LIVE MODE';
      btn.textContent = 'SWITCH TO DEMO';
    } else {
      dot.className = 'dot demo';
      text.textContent = 'DEMO MODE';
      btn.textContent = 'SWITCH TO LIVE';
    }
    widgets.forEach(w => refreshWidget(w.id));
  };

  window.toggleAddMenu = function() {
    document.getElementById('add-menu').classList.toggle('show');
  };

  document.addEventListener('click', (e) => {
    if (!e.target.closest('#add-menu') && !e.target.closest('#add-widget-btn')) {
      document.getElementById('add-menu').classList.remove('show');
    }
  });

  function updateHeaderClock() {
    const now = new Date();
    document.getElementById('clock').textContent = now.toLocaleTimeString('en-IN', { hour12: false });
  }
  setInterval(updateHeaderClock, 1000);
  updateHeaderClock();

  function init() {
    if (!loadLayout()) {
      CONFIG.defaultWidgets.forEach(w => createWidget(w.type, w.x, w.y, w.w, w.h));
    }

    updateTimer = setInterval(() => {
      widgets.forEach(w => {
        if (w.type !== 'aaplChart' && w.type !== 'worldClocks') {
          refreshWidget(w.id);
        }
      });
    }, CONFIG.refreshInterval);

    setInterval(() => {
      widgets.filter(w => w.type === 'worldClocks').forEach(w => updateClocks(w));
    }, 1000);

    setTimeout(() => {
      document.getElementById('loading').classList.add('hidden');
    }, 1500);
  }

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
