<!DOCTYPE html>
<html lang="en">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1.0">
<title>Jamal Ahmed — INDRA Terminal</title>
<style>
:root{--bg:#0a0a0f;--panel:#111118;--border:#1f1f2e;--text:#c9d1d9;--dim:#6e7681;--green:#00d4aa;--red:#ff5555;--yellow:#f0ad4e}
*{box-sizing:border-box;margin:0;padding:0}
body{background:var(--bg);color:var(--text);font-family:monospace;font-size:11px;min-height:100vh;overflow-y:auto;overflow-x:hidden}
#header{height:36px;background:var(--panel);border-bottom:1px solid var(--border);display:flex;align-items:center;justify-content:space-between;padding:0 12px;position:sticky;top:0;z-index:1000}
.logo{color:var(--green);font-weight:bold;font-size:12px}
.status{display:flex;align-items:center;gap:8px;color:var(--dim);font-size:10px}
#search-box{background:#0a0a0f;border:1px solid var(--border);color:var(--green);padding:2px 6px;font-family:inherit;font-size:10px;border-radius:3px;width:100px;outline:none}
#nifty-btn{background:#0a0a0f;border:1px solid var(--border);color:var(--green);padding:2px 8px;cursor:pointer;font-family:inherit;font-size:10px;border-radius:3px}
#nifty-btn.active{background:rgba(0,212,170,0.15);border-color:var(--green)}
#sentiment{height:24px;display:flex;align-items:center;justify-content:center;gap:12px;font-size:10px;border-bottom:1px solid var(--border);background:rgba(0,212,170,0.05);color:var(--yellow)}
#dashboard{display:grid;grid-template-columns:repeat(auto-fit,minmax(320px,1fr));gap:10px;padding:10px;max-width:1400px;margin:0 auto}
.widget{background:var(--panel);border:1px solid var(--border);border-radius:4px;overflow:hidden;min-height:200px;display:flex;flex-direction:column}
.widget-header{height:24px;background:linear-gradient(90deg,var(--panel),#161620);border-bottom:1px solid var(--border);display:flex;align-items:center;justify-content:space-between;padding:0 8px;font-size:10px;color:var(--green);font-weight:bold}
.w-btn{background:none;border:none;color:var(--dim);cursor:pointer;font-family:inherit;font-size:9px;padding:1px 4px}
.w-btn:hover{color:var(--green)}
.widget-body{flex:1;overflow:auto;padding:6px;font-size:10px}
table{width:100%;border-collapse:collapse;font-size:10px}
th{text-align:left;padding:3px 5px;color:var(--dim);border-bottom:1px solid var(--border);font-size:9px;text-transform:uppercase;position:sticky;top:0;background:var(--panel)}
td{padding:3px 5px;border-bottom:1px solid rgba(31,31,46,0.5)}
.up{color:var(--green)}.down{color:var(--red)}.muted{color:var(--dim);font-size:9px}
.heat{display:grid;grid-template-columns:repeat(4,1fr);gap:3px}
.heat-cell{display:flex;flex-direction:column;align-items:center;justify-content:center;padding:6px;border-radius:2px;font-size:9px;text-align:center;background:rgba(0,0,0,0.2);border:1px solid var(--border)}
.clocks{display:grid;grid-template-columns:repeat(2,1fr);gap:6px}
.clock{background:rgba(0,0,0,0.3);border:1px solid var(--border);border-radius:3px;padding:6px;text-align:center}
.clock-city{font-size:9px;color:var(--green)}.clock-time{font-size:15px;font-weight:bold}
.clock-status{font-size:8px;margin-top:2px;padding:1px 5px;border-radius:2px;display:inline-block}
.open{background:rgba(0,212,170,0.15);color:var(--green)}.closed{background:rgba(255,85,85,0.15);color:var(--red)}
.indices{display:flex;gap:6px;flex-wrap:wrap}
.pill{background:rgba(0,0,0,0.3);border:1px solid var(--border);border-radius:3px;padding:4px 8px;min-width:90px}
.pill-name{font-size:9px;color:var(--dim)}.pill-val{font-size:12px;font-weight:bold}.pill-chg{font-size:9px}
.metal{display:flex;align-items:center;justify-content:space-between;padding:5px;background:rgba(0,0,0,0.2);border:1px solid var(--border);border-radius:3px;margin-bottom:4px}
canvas{display:block;width:100%;height:180px}
#tooltip{position:fixed;background:var(--panel);border:1px solid var(--green);padding:5px 8px;border-radius:3px;font-size:10px;pointer-events:none;z-index:10000;display:none}
#add-menu{position:fixed;top:58px;right:8px;background:var(--panel);border:1px solid var(--border);border-radius:4px;padding:6px;z-index:10001;display:none;min-width:180px;box-shadow:0 8px 30px rgba(0,0,0,0.8)}
#add-menu.show{display:block}
.menu-item{padding:5px 8px;cursor:pointer;border-radius:2px;font-size:10px;color:var(--text)}
.menu-item:hover{background:rgba(0,212,170,0.08);color:var(--green)}
</style>
</head>
<body>

<div id="header">
  <div class="logo">JAMAL AHMED — INDRA TERMINAL</div>
  <div class="status">
    <span id="clock">--:--:--</span>
    <input type="text" id="search-box" placeholder="Search..." oninput="filterWidgets(this.value)">
    <button id="nifty-btn" onclick="toggleNifty()">NIFTY 50</button>
    <button onclick="toggleAddMenu()" style="background:#0a0a0f;border:1px solid var(--border);color:var(--text);padding:2px 6px;font-family:inherit;font-size:10px;border-radius:3px;cursor:pointer">+</button>
    <button onclick="resetLayout()" style="background:#0a0a0f;border:1px solid var(--border);color:var(--text);padding:2px 6px;font-family:inherit;font-size:10px;border-radius:3px;cursor:pointer">RESET</button>
  </div>
</div>

<div id="sentiment" class="neutral">
  <span>NIFTY: <b id="s-nifty">22,534.50</b></span>
  <span>BIAS: <b id="s-bias">NEUTRAL</b></span>
  <span>RSI: <b>--</b></span>
</div>

<div id="add-menu">
  <div class="menu-item" onclick="addW('volume')">Indian Top Volume</div>
  <div class="menu-item" onclick="addW('gainers')">Indian Top Gainers</div>
  <div class="menu-item" onclick="addW('indices')">Global Indices</div>
  <div class="menu-item" onclick="addW('heatmap')">Sector Heatmap</div>
  <div class="menu-item" onclick="addW('nifty15')">NIFTY 15-Min Chart</div>
  <div class="menu-item" onclick="addW('aapl')">AAPL Chart</div>
  <div class="menu-item" onclick="addW('metals')">Precious Metals</div>
  <div class="menu-item" onclick="addW('clocks')">World Clocks</div>
  <div class="menu-item" onclick="addW('ta')">Technical Analysis</div>
</div>

<div id="dashboard"></div>
<div id="tooltip"></div>

<script>
// ===================== DATA =====================
const stocks=[{s:'RELIANCE',n:'Reliance',p:2456.3,v:4520000},{s:'TCS',n:'TCS',p:3890.5,v:1230000},{s:'HDFCBANK',n:'HDFC Bank',p:1423.8,v:3890000},{s:'INFY',n:'Infosys',p:1456.2,v:2100000},{s:'ICICIBANK',n:'ICICI Bank',p:945.6,v:5670000},{s:'SBIN',n:'SBI',p:567.4,v:8900000},{s:'BHARTIARTL',n:'Bharti Airtel',p:876.3,v:1780000},{s:'ITC',n:'ITC',p:423.5,v:6540000},{s:'LT',n:'L&T',p:2345.6,v:980000},{s:'KOTAKBANK',n:'Kotak',p:1678.9,v:1450000},{s:'AXISBANK',n:'Axis Bank',p:987.4,v:2340000},{s:'MARUTI',n:'Maruti',p:9876.5,v:560000},{s:'HCLTECH',n:'HCL Tech',p:1234.5,v:890000},{s:'WIPRO',n:'Wipro',p:456.7,v:1230000},{s:'SUNPHARMA',n:'Sun Pharma',p:1123.4,v:780000}];
const indices=[{n:'NIFTY 50',v:22534.5,c:0.45},{n:'SENSEX',v:74123.8,c:0.38},{n:'S&P 500',v:5566.2,c:-0.12},{n:'NASDAQ',v:17890.4,c:0.67},{n:'DOW',v:41123.8,c:-0.05},{n:'FTSE',v:8345.3,c:0.23},{n:'DAX',v:18678.9,c:0.15},{n:'NIKKEI',v:38456.7,c:-0.34},{n:'HANG SENG',v:17890.5,c:0.89},{n:'SHANGHAI',v:2956.4,c:-0.21}];
const sectors=[{n:'AI/ML',sym:'AI',c:2.34},{n:'Solar',sym:'SOL',c:-1.23},{n:'EV',sym:'EV',c:0.87},{n:'Banking',sym:'BNK',c:0.45},{n:'Fintech',sym:'FIN',c:-0.67},{n:'Insurance',sym:'INS',c:0.12},{n:'Oil',sym:'OIL',c:1.45},{n:'Gas',sym:'GAS',c:0.34},{n:'Uranium',sym:'URN',c:3.21},{n:'Wind',sym:'WND',c:-0.89},{n:'Grid',sym:'GRD',c:0.23},{n:'Battery',sym:'BAT',c:-2.11}];
const metals=[{n:'Gold',sym:'XAU',p:2345.6,c:0.34},{n:'Silver',sym:'XAG',p:28.45,c:-0.56},{n:'Platinum',sym:'XPT',p:978.3,c:0.12},{n:'Palladium',sym:'XPD',p:1023.4,c:-1.23},{n:'Copper',sym:'COP',p:4.56,c:0.78}];
function r(base,v){return base*(1+(Math.random()-0.5)*v)}

// ===================== RENDERERS =====================
const R={
  volume(w){
    let data=stocks.map(s=>({s:s.s,n:s.n,p:r(s.p,0.01),v:Math.floor(s.v*(0.8+Math.random()*0.4)),c:(Math.random()-0.5)*4})).sort((a,b)=>b.v-a.v).slice(0,10);
    let h='<table><thead><tr><th>Sym</th><th>Price</th><th>Vol</th><th>Chg%</th></tr></thead><tbody>';
    data.forEach(d=>{let cl=d.c>=0?'up':'down',sg=d.c>=0?'+':'';h+=`<tr><td><b>${d.s}</b><br><span class="muted">${d.n}</span></td><td>₹${d.p.toFixed(2)}</td><td>${(d.v/1e6).toFixed(2)}M</td><td class="${cl}">${sg}${d.c.toFixed(2)}%</td></tr>`});
    w.innerHTML=h+'</tbody></table>';
  },
  gainers(w){
    let data=stocks.map(s=>({s:s.s,n:s.n,p:r(s.p,0.01),c:(Math.random()-0.3)*5,v:Math.floor(s.v*(0.5+Math.random()))})).sort((a,b)=>b.c-a.c).slice(0,10);
    let h='<table><thead><tr><th>Sym</th><th>Price</th><th>Chg%</th><th>Vol</th></tr></thead><tbody>';
    data.forEach(d=>{let cl=d.c>=0?'up':'down',sg=d.c>=0?'+':'';h+=`<tr><td><b>${d.s}</b><br><span class="muted">${d.n}</span></td><td>₹${d.p.toFixed(2)}</td><td class="${cl}">${sg}${d.c.toFixed(2)}%</td><td>${(d.v/1e6).toFixed(2)}M</td></tr>`});
    w.innerHTML=h+'</tbody></table>';
  },
  indices(w){
    let data=indices.map(i=>({n:i.n,v:r(i.v,0.005),c:i.c+(Math.random()-0.5)*0.3}));
    let h='<div class="indices">';
    data.forEach(d=>{let cl=d.c>=0?'up':'down',sg=d.c>=0?'+':'';h+=`<div class="pill"><div class="pill-name">${d.n}</div><div class="pill-val">${d.v.toLocaleString('en-IN',{maximumFractionDigits:2})}</div><div class="pill-chg ${cl}">${sg}${d.c.toFixed(2)}%</div></div>`});
    w.innerHTML=h+'</div>';
  },
  heatmap(w){
    let data=sectors.map(s=>({...s,c:s.c+(Math.random()-0.5)*1.5}));
    let h='<div class="heat">';
    data.forEach(d=>{let int=Math.min(Math.abs(d.c)/3,1),bg=d.c>=0?`rgba(0,${Math.floor(212*int)},${Math.floor(170*int)},${0.15+int*0.25})`:`rgba(${Math.floor(255*int)},${Math.floor(85*int)},${Math.floor(85*int)},${0.15+int*0.25})`,sg=d.c>=0?'+':'';h+=`<div class="heat-cell" style="background:${bg};border:1px solid ${d.c>=0?'rgba(0,212,170,0.3)':'rgba(255,85,85,0.3)'}"><b>${d.sym}</b><span class="${d.c>=0?'up':'down'}">${sg}${d.c.toFixed(2)}%</span></div>`});
    w.innerHTML=h+'</div>';
  },
  metals(w){
    let data=metals.map(m=>({...m,p:r(m.p,0.008),c:m.c+(Math.random()-0.5)*0.5}));
    let h='';
    data.forEach(m=>{let cl=m.c>=0?'up':'down',sg=m.c>=0?'+':'';h+=`<div class="metal"><div><div class="muted">${m.n} (${m.sym})</div><div style="font-size:14px;font-weight:bold">$${m.p.toFixed(2)}</div><div class="metal-chg ${cl}">${sg}${m.c.toFixed(2)}%</div></div></div>`});
    w.innerHTML=h;
  },
  clocks(w){
    w.innerHTML='<div class="clocks" id="cg"></div>';
    updateClocks(w);
  },
  aapl(w){
    w.innerHTML='<canvas id="aapl-canvas" width="400" height="180"></canvas>';
    setTimeout(drawAAPL,50);
  },
  nifty15(w){
    w.innerHTML='<canvas id="nifty-canvas" width="400" height="180"></canvas>';
    setTimeout(drawNifty,50);
  },
  ta(w){
    w.innerHTML='<div style="text-align:center;padding:20px;color:var(--dim)"><b>TECHNICAL ANALYSIS</b><br><br>NIFTY 50 Demo Base: 22,534.50<br><br>Switch to LIVE mode for real-time data.<br><br>For exact NSE India prices, use broker API (Upstox/Angel One/Zerodha).</div>';
  }
};

// ===================== CANVAS DRAWING =====================
function drawNifty(){
  const c=document.getElementById('nifty-canvas');if(!c)return;
  const ctx=c.getContext('2d'),w=c.width,h=c.height;
  ctx.fillStyle='#111118';ctx.fillRect(0,0,w,h);
  ctx.strokeStyle='rgba(31,31,46,0.6)';ctx.lineWidth=0.5;
  for(let i=0;i<=4;i++){let y=20+(h-40)/4*i;ctx.beginPath();ctx.moveTo(30,y);ctx.lineTo(w-50,y);ctx.stroke();ctx.fillStyle='#6e7681';ctx.font='8px monospace';ctx.textAlign='right';ctx.fillText((22550-(i*50)).toFixed(0),w-5,y+3)}
  let p=22500;const bw=5;
  for(let i=0;i<40;i++){
    let o=p,c=p+(Math.random()-0.48)*30,hg=Math.max(o,c)+Math.random()*15,l=Math.min(o,c)-Math.random()*15,up=c>=o;
    ctx.strokeStyle=up?'#00d4aa':'#ff5555';ctx.fillStyle=up?'#00d4aa':'#ff5555';
    ctx.beginPath();ctx.moveTo(35+i*bw+2,20+(22550-hg)/200*(h-40));ctx.lineTo(35+i*bw+2,20+(22550-l)/200*(h-40));ctx.stroke();
    ctx.fillRect(35+i*bw,20+(22550-Math.max(o,c))/200*(h-40),4,Math.max(Math.abs((c-o)/200*(h-40)),1));p=c;
  }
  ctx.fillStyle='#00d4aa';ctx.font='bold 10px monospace';ctx.fillText('NIFTY 50 — 15 Min Demo',8,14);
  ctx.fillStyle='#6e7681';ctx.font='8px monospace';ctx.fillText('Base: 22,534 | Demo Data',8,24);
}

function drawAAPL(){
  const c=document.getElementById('aapl-canvas');if(!c)return;
  const ctx=c.getContext('2d'),w=c.width,h=c.height;
  ctx.fillStyle='#111118';ctx.fillRect(0,0,w,h);
  ctx.strokeStyle='rgba(31,31,46,0.6)';ctx.lineWidth=0.5;
  for(let i=0;i<=4;i++){let y=20+(h-40)/4*i;ctx.beginPath();ctx.moveTo(30,y);ctx.lineTo(w-50,y);ctx.stroke();ctx.fillStyle='#6e7681';ctx.font='8px monospace';ctx.textAlign='right';ctx.fillText('$'+(180-(i*5)).toFixed(0),w-5,y+3)}
  let p=175;const bw=5;
  for(let i=0;i<40;i++){
    let o=p,c=p+(Math.random()-0.48)*4,hg=Math.max(o,c)+Math.random()*2,l=Math.min(o,c)-Math.random()*2,up=c>=o;
    ctx.strokeStyle=up?'#00d4aa':'#ff5555';ctx.fillStyle=up?'#00d4aa':'#ff5555';
    ctx.beginPath();ctx.moveTo(35+i*bw+2,20+(180-hg)/20*(h-40));ctx.lineTo(35+i*bw+2,20+(180-l)/20*(h-40));ctx.stroke();
    ctx.fillRect(35+i*bw,20+(180-Math.max(o,c))/20*(h-40),4,Math.max(Math.abs((c-o)/20*(h-40)),1));p=c;
  }
  ctx.fillStyle='#00d4aa';ctx.font='bold 10px monospace';ctx.fillText('AAPL — 60 Sessions',8,14);
  ctx.fillStyle='#6e7681';ctx.font='8px monospace';ctx.fillText('Demo Data',8,24);
}

// ===================== CLOCKS =====================
function updateClocks(w){
  const container=w.querySelector('#cg');if(!container)return;
  const markets=[{c:'Tokyo',tz:'Asia/Tokyo',o:9,cl:15},{c:'Mumbai',tz:'Asia/Kolkata',o:9,cl:15.5},{c:'London',tz:'Europe/London',o:8,cl:16.5},{c:'New York',tz:'America/New_York',o:9.5,cl:16}];
  let h='';markets.forEach(mk=>{
    const now=new Date();
    const ts=now.toLocaleTimeString('en-US',{timeZone:mk.tz,hour12:false,hour:'2-digit',minute:'2-digit'});
    const hr=parseInt(now.toLocaleTimeString('en-US',{timeZone:mk.tz,hour12:false,hour:'numeric'}));
    const mn=parseInt(now.toLocaleTimeString('en-US',{timeZone:mk.tz,hour12:false,minute:'numeric'}));
    const dt=hr+mn/60;let st='closed',txt='CLOSED';
    if(dt>=mk.o&&dt<mk.cl){st='open';txt='OPEN'}else if(Math.abs(dt-mk.o)<0.5){st='pre';txt='PRE-OPEN'}
    h+=`<div class="clock"><div class="clock-city">${mk.c}</div><div class="clock-time">${ts}</div><div class="clock-status ${st}">${txt}</div></div>`;
  });
  container.innerHTML=h;
}

// ===================== WIDGET SYSTEM =====================
let nextId=1;

function createWidget(type){
  const id='widget-'+nextId++;
  const el=document.createElement('div');
  el.className='widget';
  el.id=id;
  el.setAttribute('data-type',type);
  
  const titles={volume:'INDIAN TOP VOLUME',gainers:'INDIAN TOP GAINERS',indices:'GLOBAL INDICES',heatmap:'SECTOR HEATMAP',aapl:'AAPL CHART',nifty15:'NIFTY 50 — 15 MIN',metals:'PRECIOUS METALS',clocks:'WORLD SESSIONS',ta:'TECHNICAL ANALYSIS'};
  
  el.innerHTML=`<div class="widget-header"><span>${titles[type]||type.toUpperCase()}</span><div><button class="w-btn" onclick="refreshWidget('${id}')">↻</button><button class="w-btn" onclick="removeWidget('${id}')">×</button></div></div><div class="widget-body" id="body-${id}"></div>`;
  
  document.getElementById('dashboard').appendChild(el);
  refreshWidget(id);
  return el;
}

function refreshWidget(id){
  const el=document.getElementById(id);
  if(!el)return;
  const type=el.getAttribute('data-type');
  const body=el.querySelector('.widget-body');
  if(!body)return;
  const fn=R[type];
  if(fn)fn(body);
}

function removeWidget(id){
  const el=document.getElementById(id);
  if(el)el.remove();
}

function addW(type){
  createWidget(type);
  toggleAddMenu();
}

function toggleAddMenu(){
  document.getElementById('add-menu').classList.toggle('show');
}

function resetLayout(){
  document.getElementById('dashboard').innerHTML='';
  nextId=1;
  ['volume','gainers','indices','heatmap','nifty15','aapl','metals','clocks','ta'].forEach(t=>createWidget(t));
}

function filterWidgets(term){
  term=(term||'').toLowerCase().trim();
  document.querySelectorAll('.widget').forEach(w=>{
    const txt=w.textContent.toLowerCase(),type=w.getAttribute('data-type')||'';
    const isNifty=type.includes('nifty')||type.includes('ta')||type.includes('indices')||txt.includes('nifty');
    const match=!term||txt.includes(term)||type.includes(term)||(term==='nifty'&&isNifty);
    w.style.display=term&&!match?'none':'flex';
  });
  const b=document.getElementById('sentiment');
  if(term.includes('nifty')){b.style.borderBottom='1px solid #00d4aa';b.style.boxShadow='0 0 15px rgba(0,212,170,0.15)'}
  else{b.style.borderBottom='1px solid var(--border)';b.style.boxShadow='none'}
}

function toggleNifty(){
  const btn=document.getElementById('nifty-btn'),sb=document.getElementById('search-box');
  const on=btn.classList.toggle('active');
  if(on){sb.value='nifty';filterWidgets('nifty');btn.style.background='rgba(0,212,170,0.15)';btn.style.borderColor='var(--green)'}
  else{sb.value='';filterWidgets('');btn.style.background='#0a0a0f';btn.style.borderColor='var(--border)'}
}

// Close menu on outside click
document.addEventListener('click',e=>{
  const menu=document.getElementById('add-menu');
  if(!e.target.closest('#add-menu')&&!e.target.closest('[onclick="toggleAddMenu()"]'))menu.classList.remove('show');
});

// Clocks
setInterval(()=>{document.getElementById('clock').textContent=new Date().toLocaleTimeString('en-IN',{hour12:false})},1000);
setInterval(()=>{document.querySelectorAll('.widget[data-type="clocks"]').forEach(w=>updateClocks(w))},1000);

// NIFTY price jitter
setInterval(()=>{document.getElementById('s-nifty').textContent=(22534.50+(Math.random()-0.5)*50).toFixed(2)},5000);

// Init
function init(){
  ['volume','gainers','indices','heatmap','nifty15','aapl','metals','clocks','ta'].forEach(t=>createWidget(t));
}
init();
</script>
</body>
</html>
