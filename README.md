<!DOCTYPE html>
<html lang="vi">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1.0, maximum-scale=1.0, user-scalable=no">
<meta name="apple-mobile-web-app-capable" content="yes">
<meta name="apple-mobile-web-app-status-bar-style" content="black-translucent">
<meta name="apple-mobile-web-app-title" content="2048">
<meta name="theme-color" content="#9FE1CB">
<link rel="manifest" href="manifest.json">
<title>2048 Chaos Mode</title>
<style>
  :root {
    --bg: #f5f4ef;
    --surface: #eceae3;
    --card: #ffffff;
    --text: #2c2c2a;
    --muted: #888780;
    --border: rgba(0,0,0,0.10);
    --radius: 12px;
    --radius-sm: 8px;
  }
  @media (prefers-color-scheme: dark) {
    :root {
      --bg: #1a1a18;
      --surface: #252522;
      --card: #2e2e2b;
      --text: #eceae3;
      --muted: #888780;
      --border: rgba(255,255,255,0.10);
    }
  }
  * { box-sizing: border-box; margin: 0; padding: 0; -webkit-tap-highlight-color: transparent; }
  html, body {
    height: 100%;
    background: var(--bg);
    color: var(--text);
    font-family: 'Courier New', Courier, monospace;
    overflow: hidden;
  }
  .wrap {
    display: flex;
    flex-direction: column;
    align-items: center;
    justify-content: center;
    min-height: 100vh;
    min-height: 100dvh;
    padding: 16px;
    gap: 10px;
  }

  /* Header */
  .hdr {
    width: 100%;
    max-width: 400px;
    display: flex;
    align-items: center;
    justify-content: space-between;
  }
  .ttl {
    font-size: 38px;
    font-weight: bold;
    letter-spacing: -2px;
    color: var(--text);
    line-height: 1;
  }
  .ttl small {
    display: block;
    font-size: 11px;
    font-weight: normal;
    letter-spacing: 0;
    color: var(--muted);
    margin-top: -2px;
  }
  .sc-row { display: flex; gap: 6px; align-items: center; }
  .sc {
    background: var(--surface);
    border: 0.5px solid var(--border);
    border-radius: var(--radius-sm);
    padding: 5px 12px;
    text-align: center;
    min-width: 60px;
  }
  .sc-l { font-size: 9px; color: var(--muted); text-transform: uppercase; letter-spacing: .08em; }
  .sc-v { font-size: 16px; font-weight: bold; color: var(--text); }
  .newbtn {
    background: var(--surface);
    border: 0.5px solid var(--border);
    border-radius: var(--radius-sm);
    padding: 8px 14px;
    font-family: inherit;
    font-size: 12px;
    color: var(--text);
    cursor: pointer;
  }

  /* Chaos bar */
  .chaos-bar {
    width: 100%;
    max-width: 400px;
    display: flex;
    align-items: center;
    gap: 8px;
    background: var(--surface);
    border: 0.5px solid var(--border);
    border-radius: var(--radius-sm);
    padding: 7px 12px;
  }
  .badge {
    font-size: 9px;
    font-weight: bold;
    padding: 3px 8px;
    border-radius: 6px;
    text-transform: uppercase;
    letter-spacing: .06em;
    white-space: nowrap;
  }
  .badge-chaos { background: #FAECE7; color: #993C1D; }
  .badge-split { background: #E1F5EE; color: #0F6E56; }
  .track {
    flex: 1;
    height: 5px;
    background: var(--card);
    border: 0.5px solid var(--border);
    border-radius: 3px;
    overflow: hidden;
  }
  .fill {
    height: 100%;
    border-radius: 3px;
    transition: width .92s linear, background .3s;
  }
  .tnum { font-size: 13px; font-weight: bold; min-width: 16px; text-align: right; }
  .elog { font-size: 10px; color: var(--muted); min-width: 90px; text-align: right; }

  /* Board */
  .board-wrap { position: relative; width: 100%; max-width: 400px; }
  .board {
    display: grid;
    grid-template-columns: repeat(4, 1fr);
    gap: 7px;
    background: var(--surface);
    border: 0.5px solid var(--border);
    border-radius: var(--radius);
    padding: 9px;
    width: 100%;
  }
  .cell {
    aspect-ratio: 1/1;
    border-radius: var(--radius-sm);
    display: flex;
    align-items: center;
    justify-content: center;
    font-weight: bold;
    font-size: 22px;
    transition: background .12s;
  }
  .cell.pop { animation: pop .18s ease-out; }
  .cell.merge { animation: merge .18s ease-out; }
  @keyframes pop { 0%{transform:scale(.4)} 60%{transform:scale(1.12)} 100%{transform:scale(1)} }
  @keyframes merge { 0%{transform:scale(1)} 40%{transform:scale(1.15)} 100%{transform:scale(1)} }

  /* Overlay */
  .overlay {
    position: absolute;
    inset: 0;
    border-radius: var(--radius);
    background: rgba(245, 244, 239, 0.95);
    display: none;
    flex-direction: column;
    align-items: center;
    justify-content: center;
    gap: 10px;
  }
  @media (prefers-color-scheme: dark) {
    .overlay { background: rgba(26,26,24,0.95); }
  }
  .overlay.show { display: flex; }
  .ov-t { font-size: 26px; font-weight: bold; color: var(--text); }
  .ov-s { font-size: 13px; color: var(--muted); }
  .ov-btn {
    background: var(--surface);
    border: 0.5px solid var(--border);
    border-radius: var(--radius-sm);
    padding: 10px 24px;
    font-family: inherit;
    font-size: 14px;
    font-weight: bold;
    color: var(--text);
    cursor: pointer;
    margin-top: 4px;
  }

  /* Hint */
  .hint { font-size: 11px; color: var(--muted); text-align: center; line-height: 1.7; }
  .hint b { color: var(--text); font-weight: bold; }
</style>
</head>
<body>
<div class="wrap">
  <div class="hdr">
    <div class="ttl">2048<small>chaos mode</small></div>
    <div class="sc-row">
      <div class="sc"><div class="sc-l">score</div><div class="sc-v" id="sv">0</div></div>
      <div class="sc"><div class="sc-l">best</div><div class="sc-v" id="bv">0</div></div>
      <button class="newbtn" onclick="newGame()">new</button>
    </div>
  </div>

  <div class="chaos-bar">
    <span class="badge badge-chaos" id="mbadge">chaos</span>
    <div class="track"><div class="fill" id="cfill" style="width:100%"></div></div>
    <span class="tnum" id="cnum">5</span>
    <span class="elog" id="elog">event #1 in 5s</span>
  </div>

  <div class="board-wrap">
    <div class="board" id="board"></div>
    <div class="overlay" id="overlay">
      <div class="ov-t" id="ov-t">game over</div>
      <div class="ov-s" id="ov-s"></div>
      <button class="ov-btn" onclick="newGame()">chơi lại</button>
    </div>
  </div>

  <div class="hint">
    vuốt để di chuyển &nbsp;·&nbsp; chaos dịch chuyển tile lớn nhất<br>
    <b>sau 2048:</b> tile lớn nhất tách đôi mỗi 5 giây
  </div>
</div>

<script>
const COLORS = {
  0:  {bg:'transparent',cl:'transparent',br:'0.5px solid rgba(0,0,0,0.08)',sz:'22px'},
  2:  {bg:'#F1EFE8',cl:'#444441',sz:'22px'},
  4:  {bg:'#D3D1C7',cl:'#2C2C2A',sz:'22px'},
  8:  {bg:'#FAC775',cl:'#633806',sz:'22px'},
  16: {bg:'#EF9F27',cl:'#412402',sz:'22px'},
  32: {bg:'#F5C4B3',cl:'#712B13',sz:'22px'},
  64: {bg:'#F0997B',cl:'#4A1B0C',sz:'22px'},
  128:{bg:'#F4C0D1',cl:'#4B1528',sz:'20px'},
  256:{bg:'#ED93B1',cl:'#4B1528',sz:'20px'},
  512:{bg:'#CECBF6',cl:'#26215C',sz:'20px'},
  1024:{bg:'#AFA9EC',cl:'#26215C',sz:'16px'},
  2048:{bg:'#9FE1CB',cl:'#04342C',sz:'16px'},
  4096:{bg:'#5DCAA5',cl:'#04342C',sz:'14px'},
};
function cs(v){ return COLORS[v]||{bg:'#1D9E75',cl:'#ffffff',sz:'13px'}; }

let grid, score, best=0, hasPassed, chaosCount, gameOver, newCells, mergedCells;
let tim, tLeft, tTotal;

function newGame(){
  grid = Array(4).fill(null).map(()=>Array(4).fill(0));
  score=0; hasPassed=false; chaosCount=0; gameOver=false;
  newCells=new Set(); mergedCells=new Set();
  add(); add();
  render();
  clearInterval(tim); startTimer();
  document.getElementById('overlay').classList.remove('show');
}

function add(){
  let e=[];
  for(let r=0;r<4;r++) for(let c=0;c<4;c++) if(grid[r][c]===0) e.push(r*4+c);
  if(!e.length) return;
  const idx=e[Math.floor(Math.random()*e.length)];
  grid[Math.floor(idx/4)][idx%4]=Math.random()<.9?2:4;
  newCells.add(idx);
}

function timerDur(){ return hasPassed?5:(chaosCount===0?5:4); }

function startTimer(){
  tTotal=timerDur(); tLeft=tTotal;
  updateTimer();
  tim=setInterval(()=>{ tLeft--; updateTimer(); if(tLeft<=0){clearInterval(tim);fireChaos();} },1000);
}

function fireChaos(){
  if(gameOver) return;
  if(hasPassed) splitLargest(); else teleportLargest();
  chaosCount++;
  render(); checkOver();
  if(!gameOver){ startTimer(); document.getElementById('elog').textContent='event #'+(chaosCount+1)+' in '+timerDur()+'s'; }
}

function largest(){
  let mx=0,mr=-1,mc=-1;
  for(let r=0;r<4;r++) for(let c=0;c<4;c++) if(grid[r][c]>mx){mx=grid[r][c];mr=r;mc=c;}
  return{v:mx,r:mr,c:mc};
}

function teleportLargest(){
  const{v,r,c}=largest();
  let e=[];
  for(let er=0;er<4;er++) for(let ec=0;ec<4;ec++) if(!(er===r&&ec===c)&&grid[er][ec]===0) e.push([er,ec]);
  if(!e.length) return;
  const[nr,nc]=e[Math.floor(Math.random()*e.length)];
  grid[r][c]=0; grid[nr][nc]=v; newCells.add(nr*4+nc);
}

function splitLargest(){
  const{v,r,c}=largest();
  if(v<2) return;
  const h=v/2; grid[r][c]=0;
  const cands=[[r-1,c],[r,c+1],[r,c-1],[r+1,c]];
  let placed=0;
  for(const[nr,nc]of cands){
    if(placed>=2) break;
    if(nr>=0&&nr<4&&nc>=0&&nc<4&&grid[nr][nc]===0){grid[nr][nc]=h;newCells.add(nr*4+nc);placed++;}
  }
  for(let er=0;er<4&&placed<2;er++) for(let ec=0;ec<4&&placed<2;ec++) if(grid[er][ec]===0){grid[er][ec]=h;newCells.add(er*4+ec);placed++;}
}

function updateTimer(){
  const f=document.getElementById('cfill'), n=document.getElementById('cnum');
  f.style.width=Math.max(0,tLeft/tTotal*100)+'%';
  f.style.background=tLeft<=2?'#E24B4A':tLeft<=3?'#EF9F27':'#378ADD';
  n.textContent=tLeft;
  n.style.color=tLeft<=2?'#E24B4A':'';
}

function render(){
  const b=document.getElementById('board');
  b.innerHTML='';
  for(let r=0;r<4;r++){
    for(let c=0;c<4;c++){
      const v=grid[r][c], s=cs(v), idx=r*4+c;
      const d=document.createElement('div');
      d.className='cell';
      if(newCells.has(idx)) d.classList.add('pop');
      if(mergedCells.has(idx)) d.classList.add('merge');
      d.style.background=s.bg;
      d.style.color=s.cl;
      d.style.border=s.br||'none';
      d.style.fontSize=s.sz;
      if(v>0) d.textContent=v;
      b.appendChild(d);
    }
  }
  newCells=new Set(); mergedCells=new Set();
  document.getElementById('sv').textContent=score;
  if(score>best){best=score; try{localStorage.setItem('2048best',best);}catch(e){}}
  document.getElementById('bv').textContent=best;
  const mb=document.getElementById('mbadge');
  mb.className='badge '+(hasPassed?'badge-split':'badge-chaos');
  mb.textContent=hasPassed?'split':'chaos';
}

function slideRow(row){
  let t=row.filter(x=>x!==0), res=[], i=0;
  while(i<t.length){
    if(i+1<t.length&&t[i]===t[i+1]){
      const m=t[i]*2; res.push(m); score+=m;
      if(m>=2048&&!hasPassed){hasPassed=true; clearInterval(tim); startTimer();}
      i+=2;
    } else{ res.push(t[i]); i++; }
  }
  while(res.length<4) res.push(0);
  return res;
}
function tp(g){return g[0].map((_,c)=>g.map(r=>r[c]));}
function rv(g){return g.map(r=>[...r].reverse());}

function move(dir){
  if(gameOver) return;
  let g=grid.map(r=>[...r]), prev=JSON.stringify(g);
  if(dir==='left') g=g.map(slideRow);
  else if(dir==='right') g=rv(rv(g).map(slideRow));
  else if(dir==='up') g=tp(tp(g).map(slideRow));
  else if(dir==='down') g=tp(rv(tp(rv(g).map(slideRow))));
  if(JSON.stringify(g)!==prev){ grid=g; add(); render(); checkOver(); }
}

function checkOver(){
  for(let r=0;r<4;r++) for(let c=0;c<4;c++) if(grid[r][c]===0) return;
  for(let r=0;r<4;r++) for(let c=0;c<4;c++){
    if(c+1<4&&grid[r][c]===grid[r][c+1]) return;
    if(r+1<4&&grid[r][c]===grid[r+1][c]) return;
  }
  gameOver=true; clearInterval(tim);
  document.getElementById('ov-t').textContent='game over';
  document.getElementById('ov-s').textContent='điểm cuối: '+score;
  document.getElementById('overlay').classList.add('show');
}

// Keyboard
const KEYS={ArrowLeft:'left',ArrowRight:'right',ArrowUp:'up',ArrowDown:'down',
  a:'left',d:'right',w:'up',s:'down',A:'left',D:'right',W:'up',S:'down'};
document.addEventListener('keydown',e=>{ if(KEYS[e.key]){e.preventDefault();move(KEYS[e.key]);} });

// Touch swipe
let tx,ty;
document.addEventListener('touchstart',e=>{tx=e.touches[0].clientX;ty=e.touches[0].clientY;},{passive:true});
document.addEventListener('touchend',e=>{
  const dx=e.changedTouches[0].clientX-tx, dy=e.changedTouches[0].clientY-ty;
  if(Math.abs(dx)<20&&Math.abs(dy)<20) return;
  if(Math.abs(dx)>Math.abs(dy)) move(dx>0?'right':'left');
  else move(dy>0?'down':'up');
},{passive:true});

// Load best
try{best=parseInt(localStorage.getItem('2048best'))||0;}catch(e){}
document.getElementById('bv').textContent=best;

newGame();

// Register service worker
if('serviceWorker' in navigator){
  window.addEventListener('load',()=>{ navigator.serviceWorker.register('/sw.js'); });
}
</script>
</body>
</html>
[index.html](https://github.com/user-attachments/files/27265702/index.html)
[manifest.json](https://github.com/user-attachments/files/27265695/manifest.json)
{
  "name": "2048 Chaos Mode",
  "short_name": "2048",
  "description": "2048 với cơ chế chaos và split",
  "start_url": "/index.html",
  "display": "standalone",
  "background_color": "#ffffff",
  "theme_color": "#9FE1CB",
  "orientation": "portrait",
  "icons": [
    {
      "src": "icon-192.png",
      "sizes": "192x192",
      "type": "image/png",
      "purpose": "any maskable"
    },
    {
      "src": "icon-512.png",
      "sizes": "512x512",
      "type": "image/png",
      "purpose": "any maskable"
    }
  ]
  const CACHE = '2048-chaos-v1';
const FILES = ['/', '/index.html', '/manifest.json'];

self.addEventListener('install', e => {
  e.waitUntil(caches.open(CACHE).then(c => c.addAll(FILES)));
});

self.addEventListener('fetch', e => {
  e.respondWith(
    caches.match(e.request).then(r => r || fetch(e.request))
  );
});

}
[sw.js](https://github.com/user-attachments/files/27265698/sw.js)
