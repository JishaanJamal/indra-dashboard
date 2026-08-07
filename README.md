<!DOCTYPE html>
<html lang="en">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1.0">
<title>Jamal Ahmed — INDRA Terminal</title>
<style>
:root{--bg:#0a0a0f;--panel:#111118;--border:#1f1f2e;--text:#c9d1d9;--dim:#6e7681;--green:#00d4aa;--red:#ff5555;--yellow:#f0ad4e;--blue:#58a6ff}
*{box-sizing:border-box;margin:0;padding:0}
body{background:var(--bg);color:var(--text);font-family:monospace;font-size:11px;min-height:100vh;overflow-y:auto;overflow-x:hidden}
#header{height:36px;background:var(--panel);border-bottom:1px solid var(--border);display:flex;align-items:center;justify-content:space-between;padding:0 12px;position:sticky;top:0;z-index:1000}
.logo{color:var(--green);font-weight:bold;font-size:12px}
.status{display:flex;align-items:center;gap:8px;color:var(--dim);font-size:10px}
#mode-btn{background:#0a0a0f;border:1px solid var(--border);color:var(--green);padding:2px 8px;cursor:pointer;font-family:inherit;font-size:10px;border-radius:3px}
#mode-btn.live{border-color:var(--green);background:rgba(0,212,170,0.1)}
#tf-bar{height:28px;background:var(--panel);border-bottom:1px solid var(--border);display:flex;align-items:center;justify-content:center;gap:8px;padding:0 12px}
.tf-btn{background:#0a0a0f;border:1px solid var(--border);color:var(--dim);padding:2px 10px;cursor:pointer;font-family:inherit;font-size:10px;border-radius:3px}
.tf-btn.active{border-color:var(--green);color:var(--green);background:rgba(0,212,170,0.08)}
#sentiment{height:24px;display:flex;align-items:center;justify-content:center;gap:16px;font-size:10px;border-bottom:1px solid var(--border);background:rgba(0,212,170,0.03)}
#sentiment.bullish{color:var(--green)}#sentiment.bearish{color:var(--red)}#sentiment.neutral{color:var(--yellow)}
.signal-buy{background:rgba(0,212,170,0.15);color:var(--green);padding:1px 6px;border-radius:2px;font-weight:bold}
.signal-sell{background:rgba(255,85,85,0.15);color:var(--red);padding:1px 6px;border-radius:2px;font-weight:bold}
.signal-hold{background:rgba(240,173,78,0.15);color:var(--yellow);padding:1px 6px;border-radius:2px;font-weight:bold}
#dashboard{display:grid;grid-template-columns:repeat(auto-fit,minmax(320px,1fr));gap:10px;padding:10px;max-width:1400px;margin:0 auto}
.widget{background:var(--panel);border:1px solid var(--border);border-radius:4px;overflow:hidden;min-height:200px;display:flex;flex-direction:column}
.widget-header{height:24px;background:linear-gradient(90deg,var(--panel),#161620);border-bottom:1px solid var(--border);display:flex;align-items:center;justify-content:space-between;padding:0 8px;font-size:10px;color:var(--green);font-weight:bold}
.w-btn{background:none;border:none;color:var(--dim);cursor:pointer;font-family:inherit;font-size:9px;padding:1px 4px}
.w-btn:hover{color:var(--green)}
.widget-body{flex:1;overflow:auto;padding:6px;font-size:10px}
table{width:100%;border-collapse:collapse;font-size:10px}
th{text-align:left;padding:3px 5px;color:var(--dim);border-bottom:1px solid var(--border);font-size:9px;text-transform:uppercase;background:var(--panel);position:sticky;top:0}
td{padding:3px 5px;border-bottom:1px solid rgba(31,31,46,0.5)}
.up{color:var(--green)}.down{color:var(--red)}.muted{color:var(--dim);font-size:9px}
.heat{display:grid;grid-template-columns:repeat(4,1fr);gap:3px}
.heat-cell{display:flex;flex-direction:column;align-items:center;padding:6px;border-radius:2px;font-size:9px;text-align:center;background:rgba(0,0,0,0.2);border:1px solid var(--border);cursor:pointer}
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
#menu{position:fixed;top:64px;right:8px;background:var(--panel);border:1px solid var(--border);border-radius:4px;padding:6px;z-index:10001;display:none;min-width:180px;box-shadow:0 8px 30px rgba(0,0,0,0.8)}
#menu.show{display:block}
.mi{padding:5px 8px;cursor:pointer;border-radius:2px;font-size:10px;color:var(--text)}
.mi:hover{background:rgba(0,212,170,0.08);color:var(--green)}
.disclaimer{font-size:9px;color:var(--dim);text-align:center;padding:4px;border-top:1px solid var(--border);background:rgba(0,0,0,0.3)}
.ta-grid{display:grid;grid-template-columns:1fr 1fr;gap:6px;margin-bottom:8px}
.ta-card{background:rgba(0,0,0,0.2);border:1px solid var(--border);border-radius:3px;padding:8px;text-align:center}
.ta-label{font-size:9px;color:var(--dim);text-transform:uppercase;margin-bottom:4px}
.ta-value{font-size:16px;font-weight:bold}
.ta-signal{font-size:9px;margin-top:3px;padding:2px 6px;border-radius:2px;display:inline-block}
.pred-box{text-align:center;padding:10px;background:rgba(0,0,0,0.2);border:1px solid var(--border);border-radius:3px;margin-top:6px}
.pred-title{font-size:9px;color:var(--dim);text-transform:uppercase;margin-bottom:4px}
.pred-val{font-size:14px;font-weight:bold;margin-bottom:4px}
.pred-conf{font-size:9px;color:var(--dim)}
.news-item{padding:6px;border-bottom:1px solid var(--border);cursor:pointer}
.news-item:hover{background:rgba(0,212,170,0.05)}
.news-title{font-size:10px;color:var(--text);margin-bottom:2px}
.news-meta{font-size:9px;color:var(--dim)}
.news-link{color:var(--green);text-decoration:none}
.news-link:hover{text-decoration:underline}
.news-source{display:inline-block;padding:1px 4px;border-radius:2px;font-size:8px;margin-right:4px;background:rgba(0,212,170,0.1);color:var(--green)}
</style>
</head>
<body>

<div id="header">
  <div class="logo">JAMAL AHMED — INDRA TERMINAL</div>
  <div class="status">
    <span id="conn">● DEMO</span>
    <span id="clk">--:--:--</span>
    <button id="mode-btn" onclick="toggleMode()">SWITCH TO LIVE</button>
    <button onclick="togMenu()" style="background:#0a0a0f;border:1px solid var(--border);color:var(--green);padding:2px 8px;cursor:pointer;font-family:inherit;font-size:10px;border-radius:3px">+</button>
    <button onclick="resetAll()" style="background:#0a0a0f;border:1px solid var(--border);color:var(--text);padding:2px 6px;cursor:pointer;font-family:inherit;font-size:10px;border-radius:3px">RESET</button>
  </div>
</div>

<div id="tf-bar">
  <span style="color:var(--dim);font-size:9px">TIMEFRAME:</span>
  <button class="tf-btn" id="tf-1m" onclick="setTF('1m')">1M</button>
  <button class="tf-btn active" id="tf-15m" onclick="setTF('15m')">15M</button>
  <button class="tf-btn" id="tf-30m" onclick="setTF('30m')">30M</button>
  <button class="tf-btn" id="tf-1d" onclick="setTF('1d')">1D</button>
</div>

<div id="sentiment" class="neutral">
  <span>NIFTY: <b id="s-nifty">22,534.50</b></span>
  <span>BIAS: <b id="s-bias">NEUTRAL</b></span>
  <span>1M: <span id="s-1m" class="signal-hold">HOLD</span></span>
  <span>15M: <span id="s-15m" class="signal-hold">HOLD</span></span>
  <span>30M: <span id="s-30m" class="signal-hold">HOLD</span></span>
  <span>RSI: <b id="s-rsi">--</b></span>
</div>

<div id="menu">
  <div class="mi" onclick="add('v')">Indian Top Volume</div>
  <div class="mi" onclick="add('g')">Indian Top Gainers</div>
  <div class="mi" onclick="add('i')">Global Indices</div>
  <div class="mi" onclick="add('h')">Sector Heatmap</div>
  <div class="mi" onclick="add('n')">NIFTY Chart</div>
  <div class="mi" onclick="add('a')">AAPL Chart</div>
  <div class="mi" onclick="add('m')">Precious Metals</div>
  <div class="mi" onclick="add('c')">World Clocks</div>
  <div class="mi" onclick="add('t')">Technical Analysis</div>
  <div class="mi" onclick="add('x')">World News</div>
</div>

<div id="dashboard"></div>

<div class="disclaimer">
  ⚠ DEMO: Simulated data. LIVE: Yahoo Finance (15m delay). Real-time NSE requires broker API. News via Reuters RSS.
</div>

<script>
let MODE='demo',TF='15m',N=1, niftyC=[], newsData=[];
const S=[{s:'RELIANCE',n:'Reliance',p:2456.3,v:4520000},{s:'TCS',n:'TCS',p:3890.5,v:1230000},{s:'HDFCBANK',n:'HDFC Bank',p:1423.8,v:3890000},{s:'INFY',n:'Infosys',p:1456.2,v:2100000},{s:'ICICIBANK',n:'ICICI Bank',p:945.6,v:5670000},{s:'SBIN',n:'SBI',p:567.4,v:8900000},{s:'BHARTIARTL',n:'Bharti Airtel',p:876.3,v:1780000},{s:'ITC',n:'ITC',p:423.5,v:6540000},{s:'LT',n:'L&T',p:2345.6,v:980000},{s:'KOTAKBANK',n:'Kotak',p:1678.9,v:1450000},{s:'AXISBANK',n:'Axis Bank',p:987.4,v:2340000},{s:'MARUTI',n:'Maruti',p:9876.5,v:560000},{s:'HCLTECH',n:'HCL Tech',p:1234.5,v:890000},{s:'WIPRO',n:'Wipro',p:456.7,v:1230000},{s:'SUNPHARMA',n:'Sun Pharma',p:1123.4,v:780000}];
const I=[{n:'NIFTY 50',v:22534.5,c:0.45},{n:'SENSEX',v:74123.8,c:0.38},{n:'S&P 500',v:5566.2,c:-0.12},{n:'NASDAQ',v:17890.4,c:0.67},{n:'DOW',v:41123.8,c:-0.05},{n:'FTSE',v:8345.3,c:0.23},{n:'DAX',v:18678.9,c:0.15},{n:'NIKKEI',v:38456.7,c:-0.34},{n:'HANG SENG',v:17890.5,c:0.89},{n:'SHANGHAI',v:2956.4,c:-0.21}];
const H=[{n:'AI/ML',sym:'AI',c:2.34},{n:'Solar',sym:'SOL',c:-1.23},{n:'EV',sym:'EV',c:0.87},{n:'Banking',sym:'BNK',c:0.45},{n:'Fintech',sym:'FIN',c:-0.67},{n:'Insurance',sym:'INS',c:0.12},{n:'Oil',sym:'OIL',c:1.45},{n:'Gas',sym:'GAS',c:0.34},{n:'Uranium',sym:'URN',c:3.21},{n:'Wind',sym:'WND',c:-0.89},{n:'Grid',sym:'GRD',c:0.23},{n:'Battery',sym:'BAT',c:-2.11}];
const M=[{n:'Gold',sym:'XAU',p:2345.6,c:0.34},{n:'Silver',sym:'XAG',p:28.45,c:-0.56},{n:'Platinum',sym:'XPT',p:978.3,c:0.12},{n:'Palladium',sym:'XPD',p:1023.4,c:-1.23},{n:'Copper',sym:'COP',p:4.56,c:0.78}];
function r(b,v){return b*(1+(Math.random()-0.5)*v)}

function sma(d,p){let R=[];for(let i=p-1;i<d.length;i++){let s=0;for(let j=0;j<p;j++)s+=d[i-j].c;R.push({t:d[i].t,v:s/p})}return R}
function ema(d,p){let k=2/(p+1),R=[],e=d.slice(0,p).reduce((s,x)=>s+x.c,0)/p;for(let i=p;i<d.length;i++){e=d[i].c*k+e*(1-k);R.push({t:d[i].t,v:e})}return R}
function rsi(d,p=14){let g=0,l=0,R=[];for(let i=1;i<=p;i++){let ch=d[i].c-d[i-1].c;if(ch>0)g+=ch;else l-=ch}let ag=g/p,al=l/p;for(let i=p+1;i<d.length;i++){let ch=d[i].c-d[i-1].c,cg=ch>0?ch:0,cl=ch<0?-ch:0;ag=(ag*(p-1)+cg)/p;al=(al*(p-1)+cl)/p;let rs=al===0?100:100-(100/(1+ag/al));R.push({t:d[i].t,v:rs})}return R}
function macd(d,f=12,s=26,sg=9){let ef=ema(d,f),es=ema(d,s),md=[],o=s-f;for(let i=0;i<es.length;i++)if(i+o<ef.length)md.push({t:es[i].t,v:ef[i+o].v-es[i].v});let sgL=[],k=2/(sg+1),em=md.slice(0,sg).reduce((s,x)=>s+x.v,0)/sg;for(let i=sg;i<md.length;i++){em=md[i].v*k+em*(1-k);sgL.push({t:md[i].t,v:em})}let hi=[];for(let i=0;i<sgL.length;i++)if(i+sg<md.length)hi.push({t:sgL[i].t,v:md[i+sg].v-sgL[i].v});return{md,sg:sgL,hi}}
function vwap(d){let pv=0,vl=0;return d.map(x=>{let tp=(x.h+x.l+x.c)/3;pv+=tp*x.v;vl+=x.v;return{t:x.t,v:pv/vl}})}

function analyze(d){
  if(d.length<30)return{bias:'NEUTRAL',conf:0,score:0,sig:[],rsi:50,macd:0,signals:{'1m':'HOLD','15m':'HOLD','30m':'HOLD'},confidences:{'1m':0,'15m':0,'30m':0}};
  let l=d[d.length-1],rs=rsi(d,14),rl=rs.length?rs[rs.length-1].v:50;
  let mc=macd(d,12,26,9),mh=mc.hi,ml=mh.length?mh[mh.length-1].v:0,mp=mh.length>1?mh[mh.length-2].v:0;
  let vw=vwap(d),vl=vw[vw.length-1].v;
  let s20=sma(d,20),sl=s20.length?s20[s20.length-1].v:l.c;
  let e9=ema(d,9),el=e9.length?e9[e9.length-1].v:l.c;
  let sc=0,sig=[];
  if(rl<30){sc+=2;sig.push('RSI oversold')}else if(rl<40){sc+=1;sig.push('RSI weak bullish')}else if(rl>70){sc-=2;sig.push('RSI overbought')}else if(rl>60){sc-=1;sig.push('RSI weak bearish')}
  if(ml>0&&mp<=0){sc+=2;sig.push('MACD bullish cross')}else if(ml>0){sc+=1;sig.push('MACD positive')}else if(ml<0&&mp>=0){sc-=2;sig.push('MACD bearish cross')}else if(ml<0){sc-=1;sig.push('MACD negative')}
  if(l.c>vl){sc+=1;sig.push('Above VWAP')}else{sc-=1;sig.push('Below VWAP')}
  if(l.c>sl){sc+=1;sig.push('Above SMA20')}else{sc-=1;sig.push('Below SMA20')}
  if(el>sl){sc+=1;sig.push('EMA9>SMA20')}else{sc-=1;sig.push('EMA9<SMA20')}
  if(l.c>l.o)sc+=0.5;else sc-=0.5;
  let cf=Math.min(Math.abs(sc)/4.5*100,100);
  let bi='NEUTRAL';if(sc>=2)bi='BULLISH';else if(sc<=-2)bi='BEARISH';else if(sc>0)bi='SLIGHTLY BULLISH';else if(sc<0)bi='SLIGHTLY BEARISH';
  let s1m=bi.includes('BULL')?'BUY':bi.includes('BEAR')?'SELL':'HOLD';
  let s15m=bi.includes('BULL')&&cf>60?'BUY':bi.includes('BEAR')&&cf>60?'SELL':'HOLD';
  let s30m=bi.includes('BULL')&&cf>75?'BUY':bi.includes('BEAR')&&cf>75?'SELL':'HOLD';
  return{bias:bi,conf:Math.round(cf),score:sc,sig,rsi:rl,macd:ml,signals:{'1m':s1m,'15m':s15m,'30m':s30m},confidences:{'1m':Math.round(cf*0.7),'15m':Math.round(cf*0.85),'30m':cf}}
}

async function fetchYahoo(sym,intv,range){
  try{let u=`https://query1.finance.yahoo.com/v8/finance/chart/${encodeURIComponent(sym)}?interval=${intv}&range=${range}`;let R=await fetch(u,{cache:'no-store'});if(!R.ok)return null;let j=await R.json();if(!j.chart||!j.chart.result||!j.chart.result[0])return null;let re=j.chart.result[0],ts=re.timestamp,q=re.indicators.quote[0];let out=[];for(let i=0;i<ts.length;i++)if(q.open[i]!=null)out.push({o:q.open[i],h:q.high[i],l:q.low[i],c:q.close[i],v:q.volume[i]||0,t:new Date(ts[i]*1000)});return out}catch(e){return null}
}

async function fetchNews(){
  try{
    let u='https://api.rss2json.com/v1/api.json?rss_url=https://www.reuters.com/business/markets/rss.xml&count=8';
    let R=await fetch(u,{cache:'no-store'});if(!R.ok)return[];let j=await R.json();return j.items||[];
  }catch(e){return[]}
}

const R={
  v(b){let d=S.map(s=>({s:s.s,n:s.n,p:r(s.p,0.01),v:Math.floor(s.v*(0.8+Math.random()*0.4)),c:(Math.random()-0.5)*4})).sort((a,b)=>b.v-a.v).slice(0,10);let h='<table><thead><tr><th>Sym</th><th>Price</th><th>Vol</th><th>Chg%</th></tr></thead><tbody>';d.forEach(x=>{let cl=x.c>=0?'up':'down',sg=x.c>=0?'+':'';h+=`<tr><td><b>${x.s}</b><br><span class="muted">${x.n}</span></td><td>₹${x.p.toFixed(2)}</td><td>${(x.v/1e6).toFixed(2)}M</td><td class="${cl}">${sg}${x.c.toFixed(2)}%</td></tr>`});b.innerHTML=h+'</tbody></table>'},
  g(b){let d=S.map(s=>({s:s.s,n:s.n,p:r(s.p,0.01),c:(Math.random()-0.3)*5,v:Math.floor(s.v*(0.5+Math.random()))})).sort((a,b)=>b.c-a.c).slice(0,10);let h='<table><thead><tr><th>Sym</th><th>Price</th><th>Chg%</th><th>Vol</th></tr></thead><tbody>';d.forEach(x=>{let cl=x.c>=0?'up':'down',sg=x.c>=0?'+':'';h+=`<tr><td><b>${x.s}</b><br><span class="muted">${x.n}</span></td><td>₹${x.p.toFixed(2)}</td><td class="${cl}">${sg}${x.c.toFixed(2)}%</td><td>${(x.v/1e6).toFixed(2)}M</td></tr>`});b.innerHTML=h+'</tbody></table>'},
  i(b){let d=I.map(x=>({n:x.n,v:r(x.v,0.005),c:x.c+(Math.random()-0.5)*0.3}));let h='<div class="indices">';d.forEach(x=>{let cl=x.c>=0?'up':'down',sg=x.c>=0?'+':'';h+=`<div class="pill"><div class="pill-name">${x.n}</div><div class="pill-val">${x.v.toLocaleString('en-IN',{maximumFractionDigits:2})}</div><div class="pill-chg ${cl}">${sg}${x.c.toFixed(2)}%</div></div>`});b.innerHTML=h+'</div>'},
  h(b){let d=H.map(s=>({...s,c:s.c+(Math.random()-0.5)*1.5}));let h='<div class="heat">';d.forEach(x=>{let i=Math.min(Math.abs(x.c)/3,1),bg=x.c>=0?`rgba(0,${Math.floor(212*i)},${Math.floor(170*i)},${0.15+i*0.25})`:`rgba(${Math.floor(255*i)},${Math.floor(85*i)},${Math.floor(85*i)},${0.15+i*0.25})`;h+=`<div class="heat-cell" style="background:${bg};border:1px solid ${x.c>=0?'rgba(0,212,170,0.3)':'rgba(255,85,85,0.3)'}" title="${x.n}"><b>${x.sym}</b><span class="${x.c>=0?'up':'down'}">${x.c>=0?'+':''}${x.c.toFixed(2)}%</span></div>`});b.innerHTML=h+'</div>'},
  m(b){let d=M.map(x=>({...x,p:r(x.p,0.008),c:x.c+(Math.random()-0.5)*0.5}));let h='';d.forEach(x=>{let cl=x.c>=0?'up':'down',sg=x.c>=0?'+':'';h+=`<div class="metal"><div><div class="muted">${x.n} (${x.sym})</div><div style="font-size:14px;font-weight:bold">$${x.p.toFixed(2)}</div><div class="${cl}">${sg}${x.c.toFixed(2)}%</div></div></div>`});b.innerHTML=h;},
  c(b){b.innerHTML='<div class="clocks" id="cg'+b.id+'"></div>';clk(b.id);},
  a(b){b.innerHTML='<canvas id="c'+b.id+'" width="400" height="180"></canvas>';setTimeout(()=>dr('c'+b.id,175,20,'AAPL'),50);},
  n(b){b.innerHTML='<canvas id="c'+b.id+'" width="400" height="180"></canvas>';setTimeout(()=>{let p=niftyC.length&&MODE==='live'?niftyC[niftyC.length-1].c:22534;dr('c'+b.id,p,200,'NIFTY 50 '+TF.toUpperCase())},50);},
  t(b){
    let a=analyze(niftyC.length&&MODE==='live'?niftyC:genDemo());
    let l=niftyC.length&&MODE==='live'?niftyC[niftyC.length-1]:{c:22534.5,o:22520,h:22550,l:22510};
    let h=`<div class="pred-box"><div class="pred-title">NIFTY 50 — BUY / SELL SIGNALS</div><div style="display:flex;gap:8px;justify-content:center;margin-bottom:6px">`;
    ['1m','15m','30m'].forEach(tf=>{let s=a.signals[tf],cl=s==='BUY'?'signal-buy':s==='SELL'?'signal-sell':'signal-hold';h+=`<div><div style="font-size:9px;color:var(--dim)">${tf.toUpperCase()}</div><div class="${cl}">${s}</div><div style="font-size:9px;color:var(--dim)">${a.confidences[tf]}% conf</div></div>`});
    h+=`</div></div><div class="ta-grid">`;
    let rc=a.rsi>70?'var(--red)':a.rsi<30?'var(--green)':'var(--text)';h+=`<div class="ta-card"><div class="ta-label">RSI (14)</div><div class="ta-value" style="color:${rc}">${a.rsi.toFixed(1)}</div><div class="ta-signal ${a.rsi>70?'signal-sell':a.rsi<30?'signal-buy':'signal-hold'}">${a.rsi>70?'SELL':a.rsi<30?'BUY':'HOLD'}</div></div>`;
    let mc=a.macd>0?'var(--green)':'var(--red)';h+=`<div class="ta-card"><div class="ta-label">MACD Hist</div><div class="ta-value" style="color:${mc}">${a.macd>0?'+':''}${a.macd.toFixed(2)}</div><div class="ta-signal ${a.macd>0?'signal-buy':'signal-sell'}">${a.macd>0?'BUY':'SELL'}</div></div>`;
    let vd=((l.c-a.vwap)/a.vwap*100).toFixed(2);h+=`<div class="ta-card"><div class="ta-label">VWAP</div><div class="ta-value">${a.vwap.toFixed(2)}</div><div class="ta-signal ${vd>0?'signal-buy':'signal-sell'}">${vd>0?'ABOVE':'BELOW'}</div></div>`;
    let sd=((l.c-a.sma20)/a.sma20*100).toFixed(2);h+=`<div class="ta-card"><div class="ta-label">SMA20</div><div class="ta-value">${a.sma20.toFixed(2)}</div><div class="ta-signal ${sd>0?'signal-buy':'signal-sell'}">${sd>0?'ABOVE':'BELOW'}</div></div>`;
    h+=`</div><div style="margin-top:6px;padding:6px;background:rgba(0,0,0,0.2);border:1px solid var(--border);border-radius:3px"><div style="font-size:9px;color:var(--dim);margin-bottom:4px">ACTIVE SIGNALS</div>`;
    a.sig.forEach(s=>h+=`<div style="font-size:10px;padding:1px 0">• ${s}</div>`);h+=`</div>`;
    h+=`<div class="pred-box"><div class="pred-title">15-MIN PREDICTION</div><div class="pred-val" style="color:${a.bias.includes('BULL')?'var(--green)':a.bias.includes('BEAR')?'var(--red)':'var(--yellow)'}">${a.bias}</div><div class="pred-conf">Score: ${a.score>0?'+':''}${a.score.toFixed(1)} | Confidence: ${a.conf}%</div></div>`;
    h+=`<div style="margin-top:6px;padding:6px;background:rgba(0,0,0,0.2);border:1px solid var(--border);border-radius:3px;font-size:9px;color:var(--dim);text-align:center">${MODE==='live'?'LIVE: Yahoo Finance (15m delay)':'DEMO: Simulated data — not real prices'}<br>For exact NSE prices, use broker API (Upstox/Angel One/Zerodha)</div>`;
    b.innerHTML=h;
  },
  x(b){
    let h='<div style="padding:6px"><div style="font-size:9px;color:var(--dim);text-transform:uppercase;margin-bottom:6px">REUTERS MARKETS HEADLINES</div>';
    if(newsData.length===0){h+='<div style="color:var(--dim);font-size:10px">Loading news...</div>';}
    else{newsData.slice(0,6).forEach(item=>{let date=new Date(item.pubDate).toLocaleDateString();h+=`<div class="news-item" onclick="window.open('${item.link}','_blank')"><div class="news-title"><span class="news-source">REUTERS</span>${item.title}</div><div class="news-meta">${date}</div></div>`;});}
    h+=`<div style="margin-top:8px;padding-top:8px;border-top:1px solid var(--border)"><div style="font-size:9px;color:var(--dim);text-transform:uppercase;margin-bottom:4px">QUICK LINKS</div>`;
    const links=[{n:'Reuters World',u:'https://www.reuters.com/world/'},{n:'Google News India',u:'https://news.google.co.in'},{n:'The Hindu',u:'https://www.thehindu.com/news/international/'},{n:'BBC World',u:'https://www.bbc.com/news/world'},{n:'CNN World',u:'https://edition.cnn.com/world'},{n:'Al Jazeera',u:'https://www.aljazeera.com/'},{n:'Sky News',u:'https://news.sky.com/world'}];
    links.forEach(l=>h+=`<div style="padding:3px 0"><a href="${l.u}" target="_blank" class="news-link" style="font-size:10px">→ ${l.n}</a></div>`);
    h+=`</div></div>`;
    b.innerHTML=h;
  }
};

function genDemo(){let d=[],p=22534;for(let i=0;i<60;i++){let o=p,c=p+(Math.random()-0.48)*30,h=Math.max(o,c)+Math.random()*15,l=Math.min(o,c)-Math.random()*15,v=2e7+Math.random()*3e7;d.push({o,h,l,c,v,t:new Date(Date.now()-(59-i)*9e5)});p=c}return d}

function dr(cid,b,s,t){
  let c=document.getElementById(cid);if(!c)return;
  let x=c.getContext('2d'),w=c.width,h=c.height;
  x.fillStyle='#111118';x.fillRect(0,0,w,h);
  x.strokeStyle='rgba(31,31,46,0.6)';x.lineWidth=0.5;
  for(let i=0;i<=4;i++){let y=20+(h-40)/4*i;x.beginPath();x.moveTo(30,y);x.lineTo(w-50,y);x.stroke();x.fillStyle='#6e7681';x.font='8px monospace';x.textAlign='right';x.fillText((b+s-(i*s/4)).toFixed(0),w-5,y+3)}
  let p=b;const bw=5;
  for(let i=0;i<40;i++){let o=p,c=p+(Math.random()-0.48)*s/10,hg=Math.max(o,c)+Math.random()*s/20,l=Math.min(o,c)-Math.random()*s/20,up=c>=o;x.strokeStyle=up?'#00d4aa':'#ff5555';x.fillStyle=up?'#00d4aa':'#ff5555';x.beginPath();x.moveTo(35+i*bw+2,20+(b+s-hg)/s*(h-40));x.lineTo(35+i*bw+2,20+(b+s-l)/s*(h-40));x.stroke();x.fillRect(35+i*bw,20+(b+s-Math.max(o,c))/s*(h-40),4,Math.max(Math.abs((c-o)/s*(h-40)),1));p=c;}
  x.fillStyle='#00d4aa';x.font='bold 10px monospace';x.fillText(t,8,14);x.fillStyle='#6e7681';x.font='8px monospace';x.fillText(MODE==='live'?'Live Yahoo (15m delay)':'DEMO DATA',8,24);
}

function clk(id){
  let el=document.getElementById('cg'+id);if(!el)return;
  let mk=[{c:'Tokyo',tz:'Asia/Tokyo',o:9,cl:15},{c:'Mumbai',tz:'Asia/Kolkata',o:9,cl:15.5},{c:'London',tz:'Europe/London',o:8,cl:16.5},{c:'New York',tz:'America/New_York',o:9.5,cl:16}];
  let h='';mk.forEach(m=>{let n=new Date(),ts=n.toLocaleTimeString('en-US',{timeZone:m.tz,hour12:false,hour:'2-digit',minute:'2-digit'}),hr=parseInt(n.toLocaleTimeString('en-US',{timeZone:m.tz,hour12:false,hour:'numeric'})),mn=parseInt(n.toLocaleTimeString('en-US',{timeZone:m.tz,hour12:false,minute:'numeric'})),dt=hr+mn/60,st='closed',txt='CLOSED';if(dt>=m.o&&dt<m.cl){st='open';txt='OPEN'}else if(Math.abs(dt-m.o)<0.5){st='pre';txt='PRE-OPEN'}h+=`<div class="clock"><div class="clock-city">${m.c}</div><div class="clock-time">${ts}</div><div class="clock-status ${st}">${txt}</div></div>`});el.innerHTML=h;
}

function add(t){
  let id='w'+N++,e=document.createElement('div');e.className='widget';e.id=id;
  let T={v:'INDIAN TOP VOLUME',g:'INDIAN TOP GAINERS',i:'GLOBAL INDICES',h:'SECTOR HEATMAP',a:'AAPL CHART',n:'NIFTY 50 — '+TF.toUpperCase(),m:'PRECIOUS METALS',c:'WORLD SESSIONS',t:'TECHNICAL ANALYSIS',x:'WORLD NEWS'};
  e.innerHTML=`<div class="widget-header"><span>${T[t]||t.toUpperCase()}</span><div><button class="w-btn" onclick="ref('${id}','${t}')">↻</button><button class="w-btn" onclick="document.getElementById('${id}').remove()">×</button></div></div><div class="widget-body" id="b${id}"></div>`;
  document.getElementById('dashboard').appendChild(e);ref(id,t);togMenu();
}
function ref(id,t){let b=document.getElementById('b'+id);if(!b)return;let fn=R[t];if(fn)fn(b)}
function togMenu(){document.getElementById('menu').classList.toggle('show')}
document.addEventListener('click',e=>{if(!e.target.closest('#menu')&&!e.target.closest('[onclick="togMenu()"]'))document.getElementById('menu').classList.remove('show')});
function resetAll(){document.getElementById('dashboard').innerHTML='';N=1;['v','g','i','h','n','a','m','c','t','x'].forEach(t=>add(t))}

async function setTF(tf){
  TF=tf;document.querySelectorAll('.tf-btn').forEach(b=>b.classList.remove('active'));document.getElementById('tf-'+tf).classList.add('active');
  if(MODE==='live'){await loadLive()}updateSentiment();
  document.querySelectorAll('.widget').forEach(w=>{let t=w.querySelector('.widget-header span').textContent;if(t.includes('NIFTY')){let id=w.id,b=document.getElementById('b'+id);if(b){b.innerHTML='<canvas id="c'+b.id+'" width="400" height="180"></canvas>';setTimeout(()=>{let p=niftyC.length?niftyC[niftyC.length-1].c:22534;dr('c'+b.id,p,200,'NIFTY 50 '+TF.toUpperCase())},50)}}})
}

async function toggleMode(){
  MODE=MODE==='demo'?'live':'demo';
  let btn=document.getElementById('mode-btn'),conn=document.getElementById('conn'),disc=document.querySelector('.disclaimer');
  if(MODE==='live'){btn.textContent='SWITCH TO DEMO';btn.className='live';conn.innerHTML='<span style="color:var(--green)">● LIVE (15m delay)</span>';disc.innerHTML='⚠ LIVE MODE: Yahoo Finance data (15-minute delayed). Not real-time NSE. Use broker API for exact prices. News via Reuters RSS.';await loadLive()}
  else{btn.textContent='SWITCH TO LIVE';btn.className='demo';conn.innerHTML='<span style="color:var(--yellow)">● DEMO</span>';disc.innerHTML='⚠ DEMO MODE: Simulated data. Not real market prices. Switch to LIVE for delayed Yahoo Finance data. News via Reuters RSS.';niftyC=[]}
  updateSentiment();document.querySelectorAll('.widget').forEach(w=>{let t=w.querySelector('.widget-header span').textContent;if(t.includes('NIFTY')){let id=w.id;ref(id,'n')}if(t.includes('TECHNICAL')){let id=w.id;ref(id,'t')}})
}

async function loadLive(){
  let ranges={'1m':'1d','15m':'5d','30m':'1mo','1d':'3mo'};
  let d=await fetchYahoo('^NSEI',TF,ranges[TF]||'5d');
  if(d&&d.length>0){niftyC=d;updateSentiment()}
  else{console.warn('Yahoo fetch failed, using demo');niftyC=genDemo()}
}

function updateSentiment(){
  let a=analyze(niftyC.length&&MODE==='live'?niftyC:genDemo());
  let l=niftyC.length&&MODE==='live'?niftyC[niftyC.length-1]:{c:22534.5};
  document.getElementById('s-nifty').textContent=l.c.toFixed(2);
  let b=document.getElementById('s-bias'),sent=document.getElementById('sentiment');
  b.textContent=a.bias;b.style.color=a.bias.includes('BULL')?'var(--green)':a.bias.includes('BEAR')?'var(--red)':'var(--yellow)';
  sent.className=a.bias.includes('BULL')?'bullish':a.bias.includes('BEAR')?'bearish':'neutral';
  document.getElementById('s-rsi').textContent=a.rsi.toFixed(1);
  ['1m','15m','30m'].forEach(tf=>{let el=document.getElementById('s-'+tf),s=a.signals[tf];el.textContent=s;el.className=s==='BUY'?'signal-buy':s==='SELL'?'signal-sell':'signal-hold'});
}

async function loadNews(){
  newsData=await fetchNews();
  document.querySelectorAll('.widget').forEach(w=>{let t=w.querySelector('.widget-header span').textContent;if(t.includes('NEWS')){let id=w.id;ref(id,'x')}});
}

setInterval(()=>{document.getElementById('clk').textContent=new Date().toLocaleTimeString('en-IN',{hour12:false})},1000);
setInterval(()=>{document.querySelectorAll('[id^="cg"]').forEach(el=>{let id=el.id.replace('cg','');if(id)clk(id)})},1000);
setInterval(()=>{if(MODE==='live')loadLive()},30000);
setInterval(loadNews,300000);

function init(){['v','g','i','h','n','a','m','c','t','x'].forEach(t=>add(t));updateSentiment();loadNews()}
init();
</script>
</body>
</html>
