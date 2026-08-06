<!DOCTYPE html>
<html lang="en">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1.0">
<title>Jamal Ahmed — INDRA Terminal</title>
<style>
:root{--bg:#0a0a0f;--panel:#111118;--border:#1f1f2e;--text:#c9d1d9;--dim:#6e7681;--green:#00d4aa;--red:#ff5555;--yellow:#f0ad4e}
*{box-sizing:border-box;margin:0;padding:0}
html,body{background:var(--bg);color:var(--text);font-family:monospace;font-size:11px;min-height:100vh;width:100vw}
body{overflow-y:auto;overflow-x:hidden}
#header{height:36px;background:var(--panel);border-bottom:1px solid var(--border);display:flex;align-items:center;justify-content:space-between;padding:0 12px;position:sticky;top:0;z-index:1000}
.logo{color:var(--green);font-weight:bold;font-size:12px}
#search-box{background:#0a0a0f;border:1px solid var(--border);color:var(--green);padding:2px 6px;font-family:inherit;font-size:10px;border-radius:3px;width:100px;outline:none}
#nifty-btn{background:#0a0a0f;border:1px solid var(--border);color:var(--green);padding:2px 8px;cursor:pointer;font-family:inherit;font-size:10px;border-radius:3px}
#nifty-btn.active{background:rgba(0,212,170,0.15);border-color:var(--green)}
#sentiment{height:24px;display:flex;align-items:center;justify-content:center;gap:12px;font-size:10px;border-bottom:1px solid var(--border);background:rgba(0,212,170,0.05)}
#sentiment.bullish{color:var(--green)}#sentiment.bearish{color:var(--red)}#sentiment.neutral{color:var(--yellow)}
#dashboard{position:relative;width:100%;min-height:calc(100vh - 60px);padding:10px}
.widget{position:absolute;background:var(--panel);border:1px solid var(--border);border-radius:4px;overflow:hidden;display:flex;flex-direction:column;min-width:220px;min-height:140px;box-shadow:0 4px 20px rgba(0,0,0,0.5)}
.widget-header{height:24px;background:linear-gradient(90deg,var(--panel),#161620);border-bottom:1px solid var(--border);display:flex;align-items:center;justify-content:space-between;padding:0 8px;cursor:grab;user-select:none;font-size:10px;color:var(--green)}
.w-btn{background:none;border:none;color:var(--dim);cursor:pointer;font-family:inherit;font-size:9px;padding:1px 4px}
.widget-body{flex:1;overflow:auto;padding:6px;font-size:10px}
.resize{position:absolute;bottom:0;right:0;width:12px;height:12px;cursor:se-resize;background:linear-gradient(135deg,transparent 50%,var(--border) 50%);border-bottom-right-radius:3px;opacity:0.5}
table{width:100%;border-collapse:collapse;font-size:10px}
th{text-align:left;padding:3px 5px;color:var(--dim);border-bottom:1px solid var(--border);font-size:9px;text-transform:uppercase;position:sticky;top:0;background:var(--panel)}
td{padding:3px 5px;border-bottom:1px solid rgba(31,31,46,0.5)}
.up{color:var(--green)}.down{color:var(--red)}.muted{color:var(--dim);font-size:9px}
.heat{display:grid;grid-template-columns:repeat(auto-fill,minmax(70px,1fr));gap:3px}
.heat-cell{display:flex;flex-direction:column;align-items:center;justify-content:center;padding:3px;border-radius:2px;font-size:9px;text-align:center;cursor:pointer}
.clocks{display:grid;grid-template-columns:repeat(2,1fr);gap:6px}
.clock{background:rgba(0,0,0,0.3);border:1px solid var(--border);border-radius:3px;padding:6px;text-align:center}
.clock-city{font-size:9px;color:var(--green)}.clock-time{font-size:15px;font-weight:bold}
.clock-status{font-size:8px;margin-top:2px;padding:1px 5px;border-radius:2px;display:inline-block}
.open{background:rgba(0,212,170,0.15);color:var(--green)}.closed{background:rgba(255,85,85,0.15);color:var(--red)}
.indices{display:flex;gap:6px;flex-wrap:wrap}
.pill{background:rgba(0,0,0,0.3);border:1px solid var(--border);border-radius:3px;padding:4px 8px;min-width:90px}
.pill-name{font-size:9px;color:var(--dim)}.pill-val{font-size:12px;font-weight:bold}.pill-chg{font-size:9px}
.metal{display:flex;align-items:center;justify-content:space-between;padding:5px;background:rgba(0,0,0,0.2);border:1px solid var(--border);border-radius:3px;margin-bottom:4px}
canvas{display:block;width:100%;height:100%}
#tooltip{position:fixed;background:var(--panel);border:1px solid var(--green);padding:5px 8px;border-radius:3px;font-size:10px;pointer-events:none;z-index:10000;display:none}
#add-menu{position:fixed;top:58px;right:8px;background:var(--panel);border:1px solid var(--border);border-radius:4px;padding:6px;z-index:10001;display:none;min-width:160px}
#add-menu.show{display:block}
.menu-item{padding:5px 8px;cursor:pointer;border-radius:2px;font-size:10px}
.menu-item:hover{background:rgba(0,212,170,0.08);color:var(--green)}
@media(max-width:768px){.widget{position:relative!important;width:100%!important;left:0!important;top:auto!important;margin-bottom:8px;height:auto!important;min-height:180px}#dashboard{height:auto;display:flex;flex-direction:column}.resize{display:none}}
</style>
</head>
<body>

<div id="header">
  <div class="logo">JAMAL AHMED — INDRA TERMINAL</div>
  <div style="display:flex;align-items:center;gap:8px">
    <input type="text" id="search-box" placeholder="Search..." oninput="filterWidgets(this.value)">
    <button id="nifty-btn" onclick="toggleNifty()">NIFTY 50</button>
    <button class="w-btn" style="border:1px solid var(--border);padding:2px 6px" onclick="toggleAddMenu()">+</button>
    <button class="w-btn" style="border:1px solid var(--border);padding:2px 6px" onclick="resetLayout()">RESET</button>
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
const CONFIG={defaults:[{t:'ta',x:10,y:10,w:340,h:260},{t:'nifty15',x:360,y:10,w:520,h:260},{t:'volume',x:10,y:280,w:320,h:260},{t:'gainers',x:340,y:280,w:320,h:260},{t:'indices',x:670,y:10,w:440,h:140},{t:'heatmap',x:670,y:160,w:440,h:220},{t:'aapl',x:10,y:550,w:520,h:300},{t:'metals',x:540,y:550,w:280,h:180},{t:'clocks',x:540,y:740,w:280,h:180}],refresh:5000};
let W=[],nextId=1,dragW=null,resizeW=null,isDrag=false,isResize=false,dOff={x:0,y:0},rStart={x:0,y:0,w:0,h:0};
const dash=document.getElementById('dashboard'),tip=document.getElementById('tooltip');

const stocks=[{s:'RELIANCE',n:'Reliance',p:2456.3,v:4520000},{s:'TCS',n:'TCS',p:3890.5,v:1230000},{s:'HDFCBANK',n:'HDFC Bank',p:1423.8,v:3890000},{s:'INFY',n:'Infosys',p:1456.2,v:2100000},{s:'ICICIBANK',n:'ICICI Bank',p:945.6,v:5670000},{s:'SBIN',n:'SBI',p:567.4,v:8900000},{s:'BHARTIARTL',n:'Bharti Airtel',p:876.3,v:1780000},{s:'ITC',n:'ITC',p:423.5,v:6540000},{s:'LT',n:'L&T',p:2345.6,v:980000},{s:'KOTAKBANK',n:'Kotak',p:1678.9,v:1450000},{s:'AXISBANK',n:'Axis Bank',p:987.4,v:2340000},{s:'MARUTI',n:'Maruti',p:9876.5,v:560000},{s:'HCLTECH',n:'HCL Tech',p:1234.5,v:890000},{s:'WIPRO',n:'Wipro',p:456.7,v:1230000},{s:'SUNPHARMA',n:'Sun Pharma',p:1123.4,v:780000}];
const indices=[{n:'NIFTY 50',v:22534.5,c:0.45},{n:'SENSEX',v:74123.8,c:0.38},{n:'S&P 500',v:5566.2,c:-0.12},{n:'NASDAQ',v:17890.4,c:0.67},{n:'DOW',v:41123.8,c:-0.05},{n:'FTSE',v:8345.3,c:0.23},{n:'DAX',v:18678.9,c:0.15},{n:'NIKKEI',v:38456.7,c:-0.34},{n:'HANG SENG',v:17890.5,c:0.89},{n:'SHANGHAI',v:2956.4,c:-0.21}];
const sectors=[{n:'AI/ML',sym:'AI',c:2.34},{n:'Solar',sym:'SOL',c:-1.23},{n:'EV',sym:'EV',c:0.87},{n:'Banking',sym:'BNK',c:0.45},{n:'Fintech',sym:'FIN',c:-0.67},{n:'Insurance',sym:'INS',c:0.12},{n:'Oil',sym:'OIL',c:1.45},{n:'Gas',sym:'GAS',c:0.34},{n:'Uranium',sym:'URN',c:3.21},{n:'Wind',sym:'WND',c:-0.89},{n:'Grid',sym:'GRD',c:0.23},{n:'Battery',sym:'BAT',c:-2.11}];
const metals=[{n:'Gold',sym:'XAU',p:2345.6,c:0.34},{n:'Silver',sym:'XAG',p:28.45,c:-0.56},{n:'Platinum',sym:'XPT',p:978.3,c:0.12},{n:'Palladium',sym:'XPD',p:1023.4,c:-1.23},{n:'Copper',sym:'COP',p:4.56,c:0.78}];

function r(base,v){return base*(1+(Math.random()-0.5)*v)}
function genCandles(count,start){
  let d=[],p=start;
  for(let i=0;i<count;i++){let o=p,c=p+(Math.random()-0.48)*4,h=Math.max(o,c)+Math.random()*2,l=Math.min(o,c)-Math.random()*2;d.push({o,h,l,c,v:2e7+Math.random()*3e7,date:new Date(Date.now()-(count-1-i)*9e5)});p=c}
  return d;
}

const R={
  volume(w){let data=stocks.map(s=>({s:s.s,n:s.n,p:r(s.p,0.01),v:Math.floor(s.v*(0.8+Math.random()*0.4)),c:(Math.random()-0.5)*4})).sort((a,b)=>b.v-a.v).slice(0,10);let h='<table><thead><tr><th>Sym</th><th>Price</th><th>Vol</th><th>Chg%</th></tr></thead><tbody>';data.forEach(d=>{let cl=d.c>=0?'up':'down',sg=d.c>=0?'+':'';h+=`<tr><td><b>${d.s}</b><br><span class="muted">${d.n}</span></td><td>₹${d.p.toFixed(2)}</td><td>${(d.v/1e6).toFixed(2)}M</td><td class="${cl}">${sg}${d.c.toFixed(2)}%</td></tr>`});w.body.innerHTML=h+'</tbody></table>'},
  gainers(w){let data=stocks.map(s=>({s:s.s,n:s.n,p:r(s.p,0.01),c:(Math.random()-0.3)*5,v:Math.floor(s.v*(0.5+Math.random()))})).sort((a,b)=>b.c-a.c).slice(0,10);let h='<table><thead><tr><th>Sym</th><th>Price</th><th>Chg%</th><th>Vol</th></tr></thead><tbody>';data.forEach(d=>{let cl=d.c>=0?'up':'down',sg=d.c>=0?'+':'';h+=`<tr><td><b>${d.s}</b><br><span class="muted">${d.n}</span></td><td>₹${d.p.toFixed(2)}</td><td class="${cl}">${sg}${d.c.toFixed(2)}%</td><td>${(d.v/1e6).toFixed(2)}M</td></tr>`});w.body.innerHTML=h+'</tbody></table>'},
  indices(w){let data=indices.map(i=>({n:i.n,v:r(i.v,0.005),c:i.c+(Math.random()-0.5)*0.3}));let h='<div class="indices">';data.forEach(d=>{let cl=d.c>=0?'up':'down',sg=d.c>=0?'+':'';h+=`<div class="pill"><div class="pill-name">${d.n}</div><div class="pill-val">${d.v.toLocaleString('en-IN',{maximumFractionDigits:2})}</div><div class="pill-chg ${cl}">${sg}${d.c.toFixed(2)}%</div></div>`});w.body.innerHTML=h+'</div>'},
  heatmap(w){let data=sectors.map(s=>({...s,c:s.c+(Math.random()-0.5)*1.5}));let h='<div class="heat">';data.forEach(d=>{let int=Math.min(Math.abs(d.c)/3,1),bg=d.c>=0?`rgba(0,${Math.floor(212*int)},${Math.floor(170*int)},${0.15+int*0.25})`:`rgba(${Math.floor(255*int)},${Math.floor(85*int)},${Math.floor(85*int)},${0.15+int*0.25})`,sg=d.c>=0?'+':'';h+=`<div class="heat-cell" style="background:${bg};border:1px solid ${d.c>=0?'rgba(0,212,170,0.3)':'rgba(255,85,85,0.3)'}"><b>${d.sym}</b><span class="${d.c>=0?'up':'down'}">${sg}${d.c.toFixed(2)}%</span></div>`});w.body.innerHTML=h+'</div>'},
  metals(w){let data=metals.map(m=>({...m,p:r(m.p,0.008),c:m.c+(Math.random()-0.5)*0.5}));let h='';data.forEach(m=>{let cl=m.c>=0?'up':'down',sg=m.c>=0?'+':'';h+=`<div class="metal"><div><div class="metal-name">${m.n} (${m.sym})</div><div class="metal-price">$${m.p.toFixed(2)}</div><div class="metal-chg ${cl}">${sg}${m.c.toFixed(2)}%</div></div></div>`});w.body.innerHTML=h},
  clocks(w){w.body.innerHTML='<div class="clocks" id="cg"></div>';updateClocks(w)},
  aapl(w){let c=document.createElement('canvas');c.className='chart';w.body.innerHTML='';w.body.appendChild(c);w.canvas=c;let ro=new ResizeObserver(()=>drawCandles(w,genCandles(60,175),'AAPL — 60 Sessions','Demo Data'));ro.observe(w.body);w._ro=ro;drawCandles(w,genCandles(60,175),'AAPL — 60 Sessions','Demo Data')},
  nifty15(w){let c=document.createElement('canvas');c.className='chart';w.body.innerHTML='';w.body.appendChild(c);w.canvas=c;let ro=new ResizeObserver(()=>drawCandles(w,genCandles(60,22534),'NIFTY 50 — 15 Min','Demo Data ~22,534'));ro.observe(w.body);w._ro=ro;drawCandles(w,genCandles(60,22534),'NIFTY 50 — 15 Min','Demo Data ~22,534')},
  ta(w){let h='<div style="text-align:center;padding:20px;color:var(--dim)"><b>TECHNICAL ANALYSIS</b><br><br>NIFTY 50 Demo Base: 22,534.50<br><br>This is demo data.<br><br>For live data, you need a broker API<br>(Upstox, Angel One, Zerodha Kite)</div>';w.body.innerHTML=h}
};

function drawCandles(w,data,title,sub){
  let c=w.canvas;if(!c)return;
  let r=w.body.getBoundingClientRect();c.width=r.width*window.devicePixelRatio;c.height=r.height*window.devicePixelRatio;
  c.style.width=r.width+'px';c.style.height=r.height+'px';
  let ctx=c.getContext('2d');ctx.scale(window.devicePixelRatio,window.devicePixelRatio);
  let W=r.width,H=r.height,pad={t:28,r:45,b:28,l:8},cw=W-pad.l-pad.r,ch=H-pad.t-pad.b;
  let minP=Math.min(...data.map(d=>d.l))*0.998,maxP=Math.max(...data.map(d=>d.h))*1.002,range=maxP-minP;
  ctx.fillStyle='#111118';ctx.fillRect(0,0,W,H);
  ctx.strokeStyle='rgba(31,31,46,0.6)';ctx.lineWidth=0.5;
  for(let i=0;i<=4;i++){let y=pad.t+(ch/4)*i;ctx.beginPath();ctx.moveTo(pad.l,y);ctx.lineTo(W-pad.r,y);ctx.stroke();ctx.fillStyle='#6e7681';ctx.font='8px monospace';ctx.textAlign='right';ctx.fillText((maxP-(range/4)*i).toFixed(1),W-4,y+3)}
  let bw=Math.max(cw/data.length*0.6,2),sp=cw/data.length;
  data.forEach((d,i)=>{let x=pad.l+i*sp+sp*0.2,yO=pad.t+((maxP-d.o)/range)*ch,yC=pad.t+((maxP-d.c)/range)*ch,yH=pad.t+((maxP-d.h)/range)*ch,yL=pad.t+((maxP-d.l)/range)*ch,up=d.c>=d.o;ctx.strokeStyle=up?'#00d4aa':'#ff5555';ctx.fillStyle=up?'#00d4aa':'#ff5555';ctx.beginPath();ctx.moveTo(x+bw/2,yH);ctx.lineTo(x+bw/2,yL);ctx.stroke();ctx.fillRect(x,Math.min(yO,yC),bw,Math.max(Math.abs(yC-yO),1))});
  let maxV=Math.max(...data.map(d=>d.v||0));if(maxV>0){let vh=36;data.forEach((d,i)=>{let x=pad.l+i*sp+sp*0.2,v=(d.v/maxV)*vh;ctx.fillStyle=d.c>=d.o?'rgba(0,212,170,0.25)':'rgba(255,85,85,0.25)';ctx.fillRect(x,H-pad.b-v,bw,v)})}
  ctx.fillStyle='#00d4aa';ctx.font='bold 10px monospace';ctx.textAlign='left';ctx.fillText(title,pad.l,16);ctx.fillStyle='#6e7681';ctx.font='8px monospace';ctx.fillText(sub,pad.l,26);
  c.onmousemove=(e)=>{let rc=c.getBoundingClientRect(),mx=e.clientX-rc.left,my=e.clientY-rc.top;if(mx>=pad.l&&mx<=W-pad.r&&my>=pad.t&&my<=H-pad.b){let idx=Math.floor((mx-pad.l)/sp);if(idx>=0&&idx<data.length){let d=data[idx];tip.innerHTML=`${d.date.toLocaleString()}<br>O:${d.o.toFixed(2)} H:${d.h.toFixed(2)}<br>L:${d.l.toFixed(2)} C:${d.c.toFixed(2)}<br>V:${((d.v||0)/1e6).toFixed(2)}M`;tip.style.display='block';tip.style.left=(e.clientX+12)+'px';tip.style.top=(e.clientY+12)+'px'}}else{tip.style.display='none'}};
  c.onmouseleave=()=>tip.style.display='none';
}

function updateClocks(w){
  let g=w.body.querySelector('#cg');if(!g)return;
  let m=[{c:'Tokyo',tz:'Asia/Tokyo',o:9,cl:15},{c:'Mumbai',tz:'Asia/Kolkata',o:9,cl:15.5},{c:'London',tz:'Europe/London',o:8,cl:16.5},{c:'New York',tz:'America/New_York',o:9.5,cl:16}];
  let h='';m.forEach(mk=>{let n=new Date(),ts=n.toLocaleTimeString('en-US',{timeZone:mk.tz,hour12:false,hour:'2-digit',minute:'2-digit'}),hr=parseInt(n.toLocaleTimeString('en-US',{timeZone:mk.tz,hour12:false,hour:'numeric'})),mn=parseInt(n.toLocaleTimeString('en-US',{timeZone:mk.tz,hour12:false,minute:'numeric'})),dt=hr+mn/60,st='closed',txt='CLOSED';if(dt>=mk.o&&dt<mk.cl){st='open';txt='OPEN'}else if(Math.abs(dt-mk.o)<0.5){st='pre';txt='PRE-OPEN'}h+=`<div class="clock"><div class="clock-city">${mk.c}</div><div class="clock-time">${ts}</div><div class="clock-status ${st}">${txt}</div></div>`});g.innerHTML=h;
}

function create(t,x,y,w,h){
  let id='w-'+nextId++,el=document.createElement('div');el.className='widget';el.id=id;
  el.style.left=x+'px';el.style.top=y+'px';el.style.width=w+'px';el.style.height=h+'px';
  let titles={volume:'INDIAN TOP VOLUME',gainers:'INDIAN TOP GAINERS',indices:'GLOBAL INDICES',heatmap:'SECTOR HEATMAP',aapl:'AAPL CHART',nifty15:'NIFTY 50 — 15 MIN',metals:'PRECIOUS METALS',clocks:'WORLD SESSIONS',ta:'TECHNICAL ANALYSIS'};
  el.innerHTML=`<div class="widget-header" data-id="${id}"><span>${titles[t]||t.toUpperCase()}</span><div><button class="w-btn" onclick="refresh('${id}')">↻</button><button class="w-btn" onclick="remove('${id}')">×</button></div></div><div class="widget-body"></div><div class="resize" data-id="${id}"></div>`;
  dash.appendChild(el);
  let widget={id,t,el,body:el.querySelector('.widget-body'),hdr:el.querySelector('.widget-header'),rs:el.querySelector('.resize'),x,y,w,h};
  widget.hdr.addEventListener('mousedown',e=>{if(e.target.closest('.w-btn'))return;isDrag=true;dragW=widget;dOff.x=e.clientX-widget.el.offsetLeft;dOff.y=e.clientY-widget.el.offsetTop;e.preventDefault()});
  widget.rs.addEventListener('mousedown',e=>{isResize=true;resizeW=widget;rStart.x=e.clientX;rStart.y=e.clientY;rStart.w=widget.el.offsetWidth;rStart.h=widget.el.offsetHeight;e.stopPropagation()});
  W.push(widget);refresh(id);return widget;
}
function refresh(id){let w=W.find(x=>x.id===id);if(!w)return;let fn=R[w.t];if(fn)fn(w)}
function remove(id){let i=W.findIndex(x=>x.id===id);if(i===-1)return;let w=W[i];if(w._ro)w._ro.disconnect();w.el.remove();W.splice(i,1);save()}
function addW(t){let ex=W.filter(x=>x.t===t).length,x=10+ex*20,y=10+ex*20,defs={volume:[320,260],gainers:[320,260],indices:[440,140],heatmap:[440,220],aapl:[520,300],nifty15:[520,260],metals:[280,180],clocks:[280,180],ta:[340,260]};let[w,h]=defs[t]||[300,200];create(t,x,y,w,h);save();document.getElementById('add-menu').classList.remove('show')}
function save(){try{localStorage.setItem('indra',JSON.stringify(W.map(w=>({t:w.t,x:w.x,y:w.y,w:w.w,h:w.h}))))}catch(e){}}
function load(){try{let s=localStorage.getItem('indra');if(s){JSON.parse(s).forEach(l=>create(l.t,l.x,l.y,l.w,l.h));return true}}catch(e){}return false}
function reset(){W.forEach(w=>{if(w._ro)w._ro.disconnect();w.el.remove()});W=[];try{localStorage.removeItem('indra')}catch(e){}CONFIG.defaults.forEach(d=>create(d.t,d.x,d.y,d.w,d.h))}

document.addEventListener('mousemove',e=>{if(isDrag&&dragW){let nx=Math.max(0,e.clientX-dOff.x),ny=Math.max(0,e.clientY-dOff.y);dragW.el.style.left=nx+'px';dragW.el.style.top=ny+'px';dragW.x=nx;dragW.y=ny}if(isResize&&resizeW){let nw=Math.max(200,rStart.w+(e.clientX-rStart.x)),nh=Math.max(120,rStart.h+(e.clientY-rStart.y));resizeW.el.style.width=nw+'px';resizeW.el.style.height=nh+'px';resizeW.w=nw;resizeW.h=nh}});
document.addEventListener('mouseup',()=>{if(isDrag&&dragW)save();if(isResize&&resizeW){refresh(resizeW.id);save()}isDrag=false;isResize=false;dragW=null;resizeW=null});

function filterWidgets(term){term=(term||'').toLowerCase().trim();document.querySelectorAll('.widget').forEach(w=>{let txt=w.textContent.toLowerCase(),title=(w.querySelector('.widget-header span')?.textContent||'').toLowerCase();let isNifty=txt.includes('nifty')||title.includes('nifty')||title.includes('technical')||title.includes('global');let match=!term||txt.includes(term)||title.includes(term)||(term==='nifty'&&isNifty);w.style.opacity=term&&!match?'0.08':'1';w.style.pointerEvents=term&&!match?'none':'auto';w.style.transition='opacity 0.3s'});let b=document.getElementById('sentiment');if(term.includes('nifty')){b.style.borderBottom='1px solid #00d4aa';b.style.boxShadow='0 0 15px rgba(0,212,170,0.15)'}else{b.style.borderBottom='1px solid var(--border)';b.style.boxShadow='none'}}
function toggleNifty(){let btn=document.getElementById('nifty-btn'),sb=document.getElementById('search-box');let on=btn.classList.toggle('active');if(on){sb.value='nifty';filterWidgets('nifty')}else{sb.value='';filterWidgets('')}}
function toggleAddMenu(){document.getElementById('add-menu').classList.toggle('show')}
document.addEventListener('click',e=>{if(!e.target.closest('#add-menu')&&!e.target.closest('[onclick="toggleAddMenu()"]'))document.getElementById('add-menu').classList.remove('show')});
setInterval(()=>{let now=new Date();document.getElementById('s-nifty').textContent=(22534.50+(Math.random()-0.5)*50).toFixed(2)},5000);

function init(){try{if(!load())CONFIG.defaults.forEach(d=>create(d.t,d.x,d.y,d.w,d.h));setInterval(()=>W.forEach(w=>{if(w.t!=='aapl'&&w.t!=='nifty15'&&w.t!=='clocks'&&w.t!=='ta')refresh(w.id)}),CONFIG.refresh);setInterval(()=>W.filter(w=>w.t==='clocks').forEach(w=>updateClocks(w)),1000)}catch(e){console.error('Init error:',e)}}
window.refresh=refresh;window.remove=remove;window.addW=addW;window.resetLayout=reset;
window.toggleAddMenu=toggleAddMenu;window.toggleNifty=toggleNifty;window.filterWidgets=filterWidgets;
if(document.readyState==='loading')document.addEventListener('DOMContentLoaded',init);else init();
</script>
</body>
</html>
