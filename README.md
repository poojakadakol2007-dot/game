# game
application for game
 <img width="690" height="677" alt="Screenshot 2026-05-15 104229" src="https://github.com/user-attachments/assets/4a925853-d319-4f4c-9da9-5b84c6968405" />
 <img width="671" height="649" alt="Screenshot 2026-05-15 104240" src="https://github.com/user-attachments/assets/96a99595-c14e-40d9-b4eb-43e06e40e44d" />





cat > /mnt/user-data/outputs/temple_runner.html << 'HTMLEOF'
<!DOCTYPE html>
<html lang="en">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1.0">
<title>Temple Runner</title>
<style>
  *{margin:0;padding:0;box-sizing:border-box}
  body{background:#0d0500;display:flex;flex-direction:column;align-items:center;justify-content:center;min-height:100vh;font-family:monospace}
  #gameWrapper{background:#1a0a00;border-radius:12px;overflow:hidden;width:480px}
  canvas{display:block}
  #hud{width:100%;display:flex;justify-content:space-between;align-items:center;padding:10px 16px;background:#110600}
  #hud span{color:#f5c842;font-size:14px;font-weight:500}
  #lives{color:#e24b4a!important}
  #overlay{width:480px;min-height:420px;display:flex;flex-direction:column;align-items:center;justify-content:center;background:#1a0a00;padding:2rem}
  #overlay h1{color:#f5c842;font-size:28px;font-weight:500;margin-bottom:8px;text-align:center}
  #overlay p{color:#c8a060;font-size:14px;margin-bottom:1.5rem;text-align:center;line-height:1.7}
  #overlay button{background:#c8780a;color:#fff8e0;border:none;padding:10px 28px;border-radius:8px;font-size:15px;cursor:pointer;font-family:monospace;margin:6px}
  #overlay button:hover{background:#e8920c}
  #ctrl{display:flex;gap:10px;padding:10px;background:#110600;width:100%}
  #ctrl button{flex:1;background:#2a1200;color:#f5c842;border:1px solid #c8780a;border-radius:8px;padding:12px 0;font-size:14px;cursor:pointer;font-family:monospace}
  #ctrl button:hover{background:#3a1a00}
  #ctrl button:active{transform:scale(0.97)}
  #controls-hint{color:#6a4020;font-size:12px;text-align:center;padding:8px;background:#110600}
</style>
</head>
<body>
<div id="gameWrapper">
  <div id="hud">
    <span id="scoreDisplay">Score: 0</span>
    <span id="coinsDisplay">Coins: 0</span>
    <span id="lives">Lives: 3</span>
    <span id="levelDisplay">Speed: 1</span>
  </div>
  <div id="overlay">
    <h1>&#127981; Temple Runner</h1>
    <p>Run through the ancient temple!<br>Dodge obstacles &amp; collect golden coins.<br><br>
      <strong style="color:#f5c842">Barrier</strong> &#8594; Jump over it<br>
      <strong style="color:#f5c842">Low Beam</strong> &#8594; Slide under it<br>
      <strong style="color:#f5c842">Fire</strong> &#8594; Jump over it<br>
    </p>
    <button onclick="startGame()">&#9654; Start Running</button>
    <p style="font-size:12px;margin-top:1rem;color:#8a6030">
      Arrow Up / Space = Jump &nbsp;|&nbsp; Arrow Down = Slide<br>
      Arrow Left / Right = Switch Lane
    </p>
  </div>
  <canvas id="gc" width="480" height="380" style="display:none"></canvas>
  <div id="ctrl" style="display:none">
    <button onclick="handleKey('left')">&#8592; Left</button>
    <button onclick="handleKey('up')">&#8593; Jump</button>
    <button onclick="handleKey('down')">&#8595; Slide</button>
    <button onclick="handleKey('right')">&#8594; Right</button>
  </div>
  <div id="controls-hint" style="display:none">Keyboard: Arrow Keys or WASD &nbsp;|&nbsp; Mobile: Swipe on canvas</div>
</div>

<script>
const cv=document.getElementById('gc');
const ctx=cv.getContext('2d');
const W=480,H=380;
const LANES=[120,240,360];
const GROUND=290;
const PLAYER_W=28,PLAYER_H=40;

let state,raf;

function initState(){
  return{
    running:false,score:0,coins:0,lives:3,frame:0,speed:4,
    player:{lane:1,x:LANES[1],y:GROUND,vy:0,jumping:false,sliding:false,slideTimer:0,invTimer:0},
    obstacles:[],coinObjs:[],particles:[],bgTiles:[],
    laneChanging:false,targetLane:1
  };
}

function startGame(){
  document.getElementById('overlay').style.display='none';
  document.getElementById('gc').style.display='block';
  document.getElementById('ctrl').style.display='flex';
  document.getElementById('controls-hint').style.display='block';
  state=initState();
  state.running=true;
  for(let i=0;i<8;i++) state.bgTiles.push({x:i*80,speed:1});
  raf=requestAnimationFrame(loop);
}

function endGame(){
  state.running=false;
  cancelAnimationFrame(raf);
  document.getElementById('gc').style.display='none';
  document.getElementById('ctrl').style.display='none';
  document.getElementById('controls-hint').style.display='none';
  const ov=document.getElementById('overlay');
  ov.style.display='flex';
  ov.innerHTML=`
    <h1>Game Over!</h1>
    <p>Your Score: <strong style="color:#f5c842;font-size:22px">${Math.floor(state.score)}</strong><br><br>
       Coins Collected: <strong style="color:#f5c842">${state.coins}</strong></p>
    <button onclick="startGame()">&#9654; Play Again</button>`;
}

function loop(){
  if(!state.running) return;
  update();
  draw();
  raf=requestAnimationFrame(loop);
}

function update(){
  const s=state;
  s.frame++;
  s.score+=s.speed*0.05;
  s.speed=4+Math.floor(s.score/150)*0.5;
  s.speed=Math.min(s.speed,10);

  updatePlayer();
  if(s.laneChanging){
    const tx=LANES[s.targetLane];
    const dx=tx-s.player.x;
    s.player.x+=dx*0.18;
    if(Math.abs(dx)<2){s.player.x=tx;s.laneChanging=false;}
  }

  if(s.frame%55===0) spawnObstacle();
  if(s.frame%40===0) spawnCoin();

  s.obstacles.forEach(o=>{o.x-=s.speed;});
  s.coinObjs.forEach(c=>{c.x-=s.speed;});
  s.bgTiles.forEach(t=>{t.x-=s.speed*0.3;if(t.x<-80)t.x+=640;});
  s.particles=s.particles.filter(p=>{p.x+=p.vx;p.y+=p.vy;p.vy+=0.15;p.life--;return p.life>0;});

  s.obstacles=s.obstacles.filter(o=>o.x>-60);
  s.coinObjs=s.coinObjs.filter(c=>c.x>-30);

  checkCollisions();
  updateHUD();
}

function updatePlayer(){
  const p=state.player;
  if(p.jumping){
    p.vy+=0.7;
    p.y+=p.vy;
    if(p.y>=GROUND){p.y=GROUND;p.jumping=false;p.vy=0;}
  }
  if(p.sliding){
    p.slideTimer--;
    if(p.slideTimer<=0){p.sliding=false;}
  }
  if(p.invTimer>0) p.invTimer--;
}

function spawnObstacle(){
  const types=['barrier','beam','fire'];
  const t=types[Math.floor(Math.random()*types.length)];
  const lane=Math.floor(Math.random()*3);
  let h=t==='beam'?18:t==='barrier'?46:30;
  let w=t==='beam'?60:36;
  let yOff=t==='beam'?-h:0;
  state.obstacles.push({x:W+40,lane,type:t,w,h,yOff});
}

function spawnCoin(){
  const lane=Math.floor(Math.random()*3);
  const airCoin=Math.random()>0.5;
  state.coinObjs.push({x:W+20,lane,y:airCoin?GROUND-60:GROUND-10});
}

function checkCollisions(){
  const p=state.player;
  if(p.invTimer>0) return;
  const ph=p.sliding?PLAYER_H/2:PLAYER_H;
  const py=p.sliding?p.y-ph:p.y-PLAYER_H;
  const px=p.x-PLAYER_W/2;

  state.obstacles.forEach(o=>{
    if(o.lane!==p.lane) return;
    const ox=o.x-o.w/2;
    const oy=GROUND+o.yOff-o.h;
    if(px<ox+o.w&&px+PLAYER_W>ox&&py<oy+o.h&&py+ph>oy){
      hit();
    }
  });

  state.coinObjs=state.coinObjs.filter(c=>{
    if(c.lane!==p.lane) return true;
    const dist=Math.abs(c.x-p.x);
    const ydist=Math.abs(c.y-p.y+PLAYER_H/2);
    if(dist<24&&ydist<30){
      state.coins++;
      spawnParticles(c.x,c.y,'#f5c842');
      return false;
    }
    return true;
  });
}

function hit(){
  state.lives--;
  state.player.invTimer=90;
  spawnParticles(state.player.x,state.player.y-20,'#e24b4a');
  if(state.lives<=0) endGame();
}

function spawnParticles(x,y,color){
  for(let i=0;i<8;i++){
    state.particles.push({x,y,vx:(Math.random()-0.5)*4,vy:-Math.random()*4,life:20,color});
  }
}

function draw(){
  ctx.clearRect(0,0,W,H);
  drawBg();
  drawLanes();
  state.coinObjs.forEach(drawCoin);
  state.obstacles.forEach(drawObstacle);
  drawPlayer();
  state.particles.forEach(p=>{
    ctx.fillStyle=p.color;
    ctx.globalAlpha=p.life/20;
    ctx.fillRect(p.x-3,p.y-3,6,6);
    ctx.globalAlpha=1;
  });
}

function drawBg(){
  ctx.fillStyle='#0d0500';
  ctx.fillRect(0,0,W,H);
  ctx.fillStyle='#1a0800';
  ctx.fillRect(0,H-110,W,110);
  state.bgTiles.forEach(t=>{
    ctx.fillStyle='#2a1400';
    ctx.fillRect(t.x,H-115,72,8);
  });
  ctx.fillStyle='#3a2000';
  for(let i=0;i<6;i++){
    ctx.fillRect(60+i*70,20,12,H-130);
  }
  ctx.fillStyle='#261000';
  ctx.fillRect(60,0,360,30);
  for(let i=0;i<5;i++){
    ctx.fillStyle='#1e0e00';
    ctx.fillRect(80+i*72,0,24,30);
  }
}

function drawLanes(){
  LANES.forEach(lx=>{
    ctx.strokeStyle='#3d2200';
    ctx.lineWidth=1;
    ctx.setLineDash([10,14]);
    ctx.beginPath();ctx.moveTo(lx,150);ctx.lineTo(lx,GROUND+20);ctx.stroke();
    ctx.setLineDash([]);
  });
  ctx.fillStyle='#2a1400';
  ctx.fillRect(60,GROUND+20,360,8);
}

function drawPlayer(){
  const p=state.player;
  if(p.invTimer>0&&Math.floor(p.invTimer/6)%2===0) return;
  const ph=p.sliding?PLAYER_H/2:PLAYER_H;
  const py=p.y-ph;
  ctx.fillStyle='#c8780a';
  ctx.fillRect(p.x-PLAYER_W/2,py,PLAYER_W,ph);
  if(!p.sliding){
    ctx.fillStyle='#f5c842';
    ctx.beginPath();ctx.arc(p.x,py-10,12,0,Math.PI*2);ctx.fill();
    ctx.fillStyle='#c8780a';
    ctx.fillRect(p.x-6,py-8,12,6);
  }
  ctx.fillStyle='#8b4c00';
  if(!p.sliding){
    ctx.fillRect(p.x-PLAYER_W/2-6,py+6,8,PLAYER_H*0.55);
    ctx.fillRect(p.x+PLAYER_W/2-2,py+6,8,PLAYER_H*0.55);
  }
  ctx.fillStyle='#a05c00';
  ctx.fillRect(p.x-10,py+ph-4,10,10);
  ctx.fillRect(p.x,py+ph-4,10,10);
}

function drawObstacle(o){
  const ox=o.x-o.w/2;
  const oy=GROUND+o.yOff-o.h;
  if(o.type==='barrier'){
    ctx.fillStyle='#5a2d00';
    ctx.fillRect(ox,oy,o.w,o.h);
    ctx.fillStyle='#8b4c00';
    ctx.fillRect(ox+4,oy+4,o.w-8,6);
    ctx.fillRect(ox+4,oy+o.h-10,o.w-8,6);
  }else if(o.type==='beam'){
    ctx.fillStyle='#3a1a00';
    ctx.fillRect(ox,oy,o.w,o.h);
    ctx.fillStyle='#8b4c00';
    ctx.fillRect(ox,oy,o.w,4);
    ctx.fillRect(ox,oy+o.h-4,o.w,4);
  }else{
    ctx.fillStyle='#cc3300';
    ctx.fillRect(ox+8,oy+10,o.w-16,o.h-10);
    ctx.fillStyle='#ff6600';
    const fl=Math.sin(state.frame*0.2)*4;
    ctx.beginPath();
    ctx.moveTo(ox+8,oy+10);
    ctx.lineTo(o.x,oy-10+fl);
    ctx.lineTo(ox+o.w-8,oy+10);
    ctx.fill();
  }
}

function drawCoin(c){
  const pulse=Math.sin(state.frame*0.15)*2;
  ctx.fillStyle='#f5c842';
  ctx.beginPath();ctx.arc(c.x,c.y,8+pulse*0.3,0,Math.PI*2);ctx.fill();
  ctx.fillStyle='#c8a010';
  ctx.beginPath();ctx.arc(c.x-2,c.y-2,3,0,Math.PI*2);ctx.fill();
}

function updateHUD(){
  document.getElementById('scoreDisplay').textContent='Score: '+Math.floor(state.score);
  document.getElementById('coinsDisplay').textContent='Coins: '+state.coins;
  document.getElementById('lives').textContent='Lives: '+state.lives;
  document.getElementById('levelDisplay').textContent='Speed: '+state.speed.toFixed(1);
}

function handleKey(dir){
  if(!state||!state.running) return;
  const p=state.player;
  if(dir==='up'||dir==='space'){
    if(!p.jumping&&!p.sliding){p.jumping=true;p.vy=-13;}
  }else if(dir==='down'){
    if(!p.jumping){p.sliding=true;p.slideTimer=35;}
  }else if(dir==='left'){
    if(p.lane>0&&!state.laneChanging){p.lane--;state.targetLane=p.lane;state.laneChanging=true;}
  }else if(dir==='right'){
    if(p.lane<2&&!state.laneChanging){p.lane++;state.targetLane=p.lane;state.laneChanging=true;}
  }
}

document.addEventListener('keydown',e=>{
  if(e.code==='ArrowUp'||e.code==='Space'){e.preventDefault();handleKey('up');}
  else if(e.code==='ArrowDown'){e.preventDefault();handleKey('down');}
  else if(e.code==='ArrowLeft'){e.preventDefault();handleKey('left');}
  else if(e.code==='ArrowRight'){e.preventDefault();handleKey('right');}
  else if(e.code==='KeyW'){e.preventDefault();handleKey('up');}
  else if(e.code==='KeyS'){e.preventDefault();handleKey('down');}
  else if(e.code==='KeyA'){e.preventDefault();handleKey('left');}
  else if(e.code==='KeyD'){e.preventDefault();handleKey('right');}
});

let touchStartX=0,touchStartY=0;
cv.addEventListener('touchstart',e=>{touchStartX=e.touches[0].clientX;touchStartY=e.touches[0].clientY;},{passive:true});
cv.addEventListener('touchend',e=>{
  const dx=e.changedTouches[0].clientX-touchStartX;
  const dy=e.changedTouches[0].clientY-touchStartY;
  if(Math.abs(dy)>Math.abs(dx)){
    handleKey(dy<-20?'up':'down');
  }else{
    if(Math.abs(dx)>20) handleKey(dx<0?'left':'right');
  }
},{passive:true});
</script>
</body>
</html>
HTMLEOF
echo "Done"


