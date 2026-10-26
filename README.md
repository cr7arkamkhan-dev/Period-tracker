<!DOCTYPE html>
<html lang="en">
<head>
<meta charset="utf-8">
<meta name="viewport" content="width=device-width, initial-scale=1, viewport-fit=cover">
<title>Cycle Calendar</title>
<style>
:root{box-sizing:border-box;padding-top:env(safe-area-inset-top,0px);padding-bottom:env(safe-area-inset-bottom,0px);
--bg:#faf7f7;--card:#fff;--tx:#2a2226;--mu:#7d6f75;--bd:#eadfe3;--per:#e5486b;--pms:#f4a259;--post:#7fb7a4;--mid:#e9e4f3;--ac:#b8325a}
@media (prefers-color-scheme:dark){:root:not([data-theme="light"]){color-scheme:dark;--bg:#171214;--card:#221a1e;--tx:#f3eaee;--mu:#a89aa1;--bd:#3a2d33;--per:#ee5f80;--pms:#e59a4f;--post:#5e9c88;--mid:#35304a;--ac:#f08bab}}
html{scroll-padding-top:env(safe-area-inset-top,0px)}
body{margin:0;background:var(--bg);color:var(--tx);font:16px/1.4 system-ui,-apple-system,"Segoe UI",sans-serif}
.w{max-width:480px;margin:0 auto;padding:16px}
h1{font-size:20px;margin:0 0 12px}
.card{background:var(--card);border:1px solid var(--bd);border-radius:14px;padding:14px;margin-bottom:12px}
.big{font-size:26px;font-weight:700;color:var(--ac)}
.mu{color:var(--mu);font-size:13px}
.nav{display:flex;justify-content:space-between;align-items:center;margin-bottom:8px}
button{font:inherit;border:1px solid var(--bd);background:var(--card);color:var(--tx);border-radius:10px;padding:8px 12px;cursor:pointer}
button.p{background:var(--per);border-color:var(--per);color:#fff}
.g{display:grid;grid-template-columns:repeat(7,1fr);gap:4px}
.dh{text-align:center;font-size:12px;color:var(--mu)}
.d{aspect-ratio:1;border-radius:10px;display:flex;align-items:center;justify-content:center;font-size:14px;cursor:pointer;position:relative;border:2px solid transparent}
.d.per{background:var(--per);color:#fff}.d.pms{background:var(--pms);color:#fff}.d.post{background:var(--post);color:#fff}.d.mid{background:var(--mid)}
.d.pr{opacity:.6;border-style:dashed;border-color:var(--card)}
.d.today{border-color:var(--tx)}.d.sel{outline:3px solid var(--ac);outline-offset:1px}
.d.s::after{content:"●";position:absolute;bottom:0;font-size:8px}
.lg{display:flex;flex-wrap:wrap;gap:10px;font-size:12px;color:var(--mu);margin-top:10px}
.lg i{display:inline-block;width:11px;height:11px;border-radius:3px;margin-right:4px;vertical-align:-1px}
.row{display:flex;justify-content:space-between;align-items:center;padding:6px 0;border-bottom:1px solid var(--bd)}
label{display:flex;justify-content:space-between;align-items:center;margin:6px 0}
input[type=number]{width:64px;font:inherit;padding:6px;border:1px solid var(--bd);border-radius:8px;background:var(--bg);color:var(--tx)}
.acts{display:flex;gap:8px;margin-top:10px;flex-wrap:wrap}
</style>
</head>
<body>
<div class="w">
<h1>Cycle Calendar</h1>
<div class="card" id="sum"></div>
<div class="card">
 <div class="nav"><button id="pv">‹</button><b id="mt"></b><button id="nx">›</button></div>
 <div class="g" id="hd"></div><div class="g" id="cal" style="margin-top:4px"></div>
 <div class="lg"><span><i style="background:var(--per)"></i>Period</span><span><i style="background:var(--post)"></i>Post-period</span><span><i style="background:var(--mid)"></i>Mid-cycle</span><span><i style="background:var(--pms)"></i>PMS</span><span>Faded = predicted</span></div>
</div>
<div class="card" id="sel"></div>
<div class="card"><b>Logged periods</b><div id="log"></div></div>
<div class="card"><b>Settings</b>
 <label>Default cycle length (days)<input type="number" id="sl" min="21" max="90"></label>
 <label>Period length (days)<input type="number" id="sp" min="1" max="12"></label>
 <label>PMS days before period<input type="number" id="sm" min="1" max="14"></label>
 <label>Post-period days<input type="number" id="so" min="1" max="14"></label>
 <div class="mu">With PCOS, cycles are often long and irregular (commonly 35–45+ days). Once 2 or more periods are logged, the average of her real cycles is used instead of the default.</div>
</div>
<p class="mu">Predictions are estimates, not medical advice. Ovulation is uncertain with PCOS, so no fertile window is shown. Data is saved only in this browser.</p>
</div>
<script>
const DAY=864e5,$=id=>document.getElementById(id);
let S={starts:[],set:{len:35,per:5,pms:7,post:7}};
try{const r=localStorage.getItem('cyc1');if(r)S=Object.assign(S,JSON.parse(r))}catch(e){}
const save=()=>{try{localStorage.setItem('cyc1',JSON.stringify(S))}catch(e){}};
const N=(y,m,d)=>Math.round(Date.UTC(y,m,d)/DAY);
const D=n=>new Date(n*DAY);
const fmt=n=>D(n).toLocaleDateString('en-IN',{day:'numeric',month:'short',year:'numeric',timeZone:'UTC'});
const t=new Date(),today=N(t.getFullYear(),t.getMonth(),t.getDate());
let vy=t.getFullYear(),vm=t.getMonth(),sel=today;
function gaps(){const s=S.starts.slice().sort((a,b)=>a-b),g=[];for(let i=1;i<s.length;i++)g.push(s[i]-s[i-1]);return g.slice(-6)}
function avg(){const g=gaps();return g.length?Math.round(g.reduce((a,b)=>a+b,0)/g.length):S.set.len}
function all(){const s=S.starts.slice().sort((a,b)=>a-b),r=s.map(x=>({n:x,p:0}));if(s.length){const a=avg();for(let k=1;k<=8;k++)r.push({n:s[s.length-1]+a*k,p:1})}return r}
function info(n){const a=all();let c=null,i=-1;for(let j=0;j<a.length;j++)if(a[j].n<=n)c=j;if(c===null)return null;
 const s=a[c],nx=a[c+1]?a[c+1].n:s.n+avg(),L=nx-s.n,d=n-s.n,{per,pms,post}=S.set;let ph='mid';
 if(d<per)ph='per';else if(d>=L-pms)ph='pms';else if(d<per+post)ph='post';
 return{ph,pr:s.p||(a[c+1]&&false),day:d+1,L,pr2:s.p}}
const NM={per:'Period',pms:'PMS',post:'Post-period',mid:'Mid-cycle'};
function render(){
 $('hd').innerHTML='SMTWTFS'.split('').map(x=>`<div class="dh">${x}</div>`).join('');
 $('mt').textContent=new Date(vy,vm,1).toLocaleDateString('en-IN',{month:'long',year:'numeric'});
 const f=new Date(vy,vm,1).getDay(),dim=new Date(vy,vm+1,0).getDate();let h='';
 for(let i=0;i<f;i++)h+='<div></div>';
 for(let d=1;d<=dim;d++){const n=N(vy,vm,d),i=info(n);
  h+=`<div class="d ${i?i.ph:''} ${i&&i.pr2?'pr':''} ${n===today?'today':''} ${n===sel?'sel':''} ${S.starts.includes(n)?'s':''}" data-n="${n}">${d}</div>`}
 $('cal').innerHTML=h;
 document.querySelectorAll('.d[data-n]').forEach(e=>e.onclick=()=>{sel=+e.dataset.n;render()});
 const a=all(),nxt=a.find(x=>x.p&&x.n>=today)||a.find(x=>x.n>today),g=gaps(),cur=info(today);
 let s='';
 if(!S.starts.length)s='<div class="mu">Tap the first day of her last period on the calendar, then choose “Period started”.</div>';
 else{const dl=nxt.n-today,lo=g.length>1?Math.min(...g):null,hi=g.length>1?Math.max(...g):null,last=Math.max(...S.starts);
  s=`<div class="mu">Next period expected</div><div class="big">${fmt(nxt.n)}</div><div>${dl>0?'in '+dl+' day'+(dl==1?'':'s'):dl==0?'today':Math.abs(dl)+' days late'}</div>`;
  if(lo!==null)s+=`<div class="mu">Likely window: ${fmt(last+lo)} – ${fmt(last+hi)} (her shortest to longest recent cycle)</div>`;
  s+=`<div class="mu" style="margin-top:6px">Average cycle: ${avg()} days${g.length?'':' (default, nothing to average yet)'}${cur?' · Today: '+NM[cur.ph]+', day '+cur.day:''}</div>`;
  const pm=nxt.n-S.set.pms;if(pm>today)s+=`<div class="mu">PMS phase starts ${fmt(pm)}</div>`}
 $('sum').innerHTML=s;
 const si=info(sel),has=S.starts.includes(sel);
 $('sel').innerHTML=`<b>${fmt(sel)}</b><div class="mu">${si?NM[si.ph]+' · cycle day '+si.day+(si.pr2?' (predicted)':''):'No data for this date'}</div>
 <div class="acts">${has?'<button id="rm">Remove period start</button>':'<button class="p" id="ad">Period started this day</button>'}</div>`;
 if($('ad'))$('ad').onclick=()=>{S.starts.push(sel);save();render()};
 if($('rm'))$('rm').onclick=()=>{S.starts=S.starts.filter(x=>x!==sel);save();render()};
 const ls=S.starts.slice().sort((a,b)=>b-a);
 $('log').innerHTML=ls.length?ls.map((n,i)=>{const p=ls[i+1];return `<div class="row"><span>${fmt(n)}</span><span class="mu">${p?(n-p)+'-day cycle':'first logged'}</span></div>`}).join(''):'<div class="mu">Nothing logged yet.</div>';
}
$('pv').onclick=()=>{vm--;if(vm<0){vm=11;vy--}render()};
$('nx').onclick=()=>{vm++;if(vm>11){vm=0;vy++}render()};
[['sl','len'],['sp','per'],['sm','pms'],['so','post']].forEach(([id,k])=>{$(id).value=S.set[k];$(id).onchange=()=>{const v=parseInt($(id).value);if(v>0){S.set[k]=v;save();render()}}});
render();
</script>
</body>
</html>

