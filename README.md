<!doctype html>
<html lang="ko">
<head>
<meta charset="utf-8">
<meta name="viewport" content="width=device-width,initial-scale=1">
<title>2인용 사다리타기</title>
<style>
body{font-family:'Segoe UI',Tahoma,Geneva,Verdana,sans-serif;margin:0;min-height:100vh;background:#f0f4f8;color:#1f2937;display:flex;align-items:center;justify-content:center;padding:24px}
.wrap{width:760px;max-width:96%;background:#fff;border-radius:16px;padding:24px;box-shadow:0 8px 20px rgba(0,0,0,0.2);position:relative}
h1{margin:0 0 12px;font-size:24px;text-align:center;color:#111827;position:relative;top:-10px}
.btn{background:#3b82f6;border:none;padding:10px 16px;border-radius:10px;color:white;font-weight:600;cursor:pointer;transition:background 0.2s}
.btn:hover{background:#2563eb}
.btn.secondary{background:#e5e7eb;color:#374151;border:1px solid #d1d5db}
.btn.secondary:hover{background:#d1d5db}
#ladderCanvas{background:#e5e7eb;border-radius:12px;display:block;margin:16px auto;width:680px;height:380px}
.input-inline{width:80px;text-align:center;background:#f9fafb;border:1px solid #d1d5db;color:#111827;border-radius:8px;padding:4px;font-size:14px;position:absolute;box-shadow:0 2px 4px rgba(0,0,0,0.1)}
#finalResults{display:flex;justify-content:space-between;margin-top:12px;font-weight:700;color:#111827}
</style>
</head>
<body>
<div class="wrap">
<h1>2인용 사다리타기</h1>
<canvas id="ladderCanvas" width="680" height="380"></canvas>

<!-- 출발 이름 입력 -->
<input id="name1" class="input-inline" placeholder="1번 이름" style="top:70px; left:30%;">
<input id="name2" class="input-inline" placeholder="2번 이름" style="top:70px; left:60%;">

<!-- 결과 입력란 -->
<input id="resInput1Bottom" class="input-inline" placeholder="1번 결과" style="bottom:100px; left:30%;">
<input id="resInput2Bottom" class="input-inline" placeholder="2번 결과" style="bottom:100px; left:60%;">

<div style="text-align:center;margin-top:12px;">
  <button class="btn" id="runBtn">사다리타기</button>
  <button class="btn secondary" id="resetBtn">초기화</button>
</div>

<div id="finalResults">
  <div id="res1"></div>
  <div id="res2"></div>
</div>
</div>

<script>
(function(){
const canvas=document.getElementById('ladderCanvas');
const ctx=canvas.getContext('2d');
const runBtn=document.getElementById('runBtn');
const resetBtn=document.getElementById('resetBtn');
const res1=document.getElementById('res1');
const res2=document.getElementById('res2');
const resInput1Bottom=document.getElementById('resInput1Bottom');
const resInput2Bottom=document.getElementById('resInput2Bottom');

const columns=2;
const rows=12;
let rungs=[];
let isRunning=false;

function fitCanvas(){
  const dpr=window.devicePixelRatio||1;
  const w=canvas.clientWidth||canvas.width;
  const h=canvas.clientHeight||canvas.height;
  canvas.width=w*dpr;
  canvas.height=h*dpr;
  canvas.style.width=w+'px';
  canvas.style.height=h+'px';
  ctx.setTransform(dpr,0,0,dpr,0,0);
}

function xForCol(i){return (i+1)*canvas.width/(columns+1)/(window.devicePixelRatio||1);}
function yForRow(i){const top=30;const bottom=(canvas.height/(window.devicePixelRatio||1))-60;const h=bottom-top;return top+(i+1)*(h/(rows+1));}

function generateRungs(){
  rungs=[];
  if(isRunning){
    for(let i=0;i<rows;i++){
      rungs.push(i%2===0?true:false);
    }
  }else{
    for(let i=0;i<rows;i++){
      rungs.push(false);
    }
  }
}

function drawLadder(){
  fitCanvas();
  const w=canvas.width/(window.devicePixelRatio||1);
  const h=canvas.height/(window.devicePixelRatio||1);
  ctx.clearRect(0,0,w,h);
  ctx.lineWidth=2;
  ctx.strokeStyle='rgba(107,114,128,0.3)';

  for(let c=0;c<columns;c++){const x=xForCol(c);ctx.beginPath();ctx.moveTo(x,10);ctx.lineTo(x,h-40);ctx.stroke();}
  for(let i=0;i<rows;i++){if(!rungs[i])continue;const y=yForRow(i);const x1=xForCol(0);const x2=xForCol(1);ctx.beginPath();ctx.moveTo(x1,y);ctx.lineTo(x2,y);ctx.stroke();}
}

function computePath(startCol){
  const path=[];let col=startCol;let x=xForCol(col);let y=10;path.push({x,y});
  for(let i=0;i<rows;i++){
    const targetY=yForRow(i);path.push({x,y:targetY});
    if(rungs[i]){col=1-col;x=xForCol(col);path.push({x,y:targetY});}
    y=targetY;
  }
  path.push({x,y:(canvas.height/(window.devicePixelRatio||1))-40});
  return path;
}

function animatePath(path,color,onComplete){
  const speed=0.5;let segIndex=0;let segPos=0;const segs=[];
  for(let i=0;i<path.length-1;i++){const dx=path[i+1].x-path[i].x;const dy=path[i+1].y-path[i].y;segs.push(Math.hypot(dx,dy));}
  let lastTime=null;let currentPos={x:path[0].x,y:path[0].y};

  function step(ts){
    if(!lastTime)lastTime=ts;const dt=ts-lastTime;lastTime=ts;let move=dt*speed;
    while(move>0 && segIndex<segs.length){
      const remain=segs[segIndex]-segPos;
      if(move<remain){segPos+=move;move=0;const t=segPos/segs[segIndex];currentPos.x=path[segIndex].x+(path[segIndex+1].x-path[segIndex].x)*t;currentPos.y=path[segIndex].y+(path[segIndex+1].y-path[segIndex].y)*t;}
      else{move-=remain;segIndex++;segPos=0;if(segIndex<segs.length){currentPos.x=path[segIndex].x;currentPos.y=path[segIndex].y;}else{currentPos.x=path[path.length-1].x;currentPos.y=path[path.length-1].y;}}
    }
    drawLadder();
    ctx.strokeStyle=color;
    ctx.lineWidth=4;ctx.lineCap='round';ctx.beginPath();ctx.moveTo(path[0].x,path[0].y);
    for(let i=1;i<=segIndex && i<path.length;i++){ctx.lineTo(path[i].x,path[i].y);}
    if(segIndex<segs.length){ctx.lineTo(currentPos.x,currentPos.y);}
    ctx.stroke();ctx.fillStyle=color;ctx.beginPath();ctx.arc(currentPos.x,currentPos.y,9,0,Math.PI*2);ctx.fill();
    if(segIndex>=segs.length){onComplete&&onComplete(currentPos);}else{requestAnimationFrame(step);}
  }
  requestAnimationFrame(step);
}

function runOnce(){
  isRunning=true;
  generateRungs();
  drawLadder();
  res1.textContent='';res2.textContent='';

  const path1=computePath(0);
  const path2=computePath(1);

  // 왼쪽 플레이어 진행 후 오른쪽 플레이어 진행
  animatePath(path1,'#3b82f6',()=>{
    res1.textContent=resInput1Bottom.value;
    animatePath(path2,'#10b981',()=>{
      res2.textContent=resInput2Bottom.value;
    });
  });
}

function init(){fitCanvas();generateRungs();drawLadder();}
runBtn.addEventListener('click',runOnce);
resetBtn.addEventListener('click',()=>{isRunning=false;generateRungs();drawLadder();res1.textContent='';res2.textContent='';});
window.addEventListener('resize',()=>{init();});
init();
})();
</script>
</body>
</html>
