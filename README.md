<!DOCTYPE html>
<html lang="fr">
<head>
<meta charset="utf-8" />
<meta name="viewport" content="width=device-width,initial-scale=1" />
<title>La Malédiction du Manoir — Version Immersive</title>
<style>
  @import url('https://fonts.googleapis.com/css2?family=Creepster&display=swap');

  :root{
    --accent:#ff3c00;
    --bg-dark:#060607;
    --panel:rgba(0,0,0,0.72);
    --sanity-high: linear-gradient(90deg,#00ff99,#0099ff);
    --sanity-mid: linear-gradient(90deg,#ffcc00,#ff6600);
    --sanity-low: linear-gradient(90deg,#ff0000,#660000);
  }

  html,body{
    height:100%;
    margin:0;
    background:radial-gradient(circle at 20% 10%, #050506 10%, #0b0b0e 60%);
    color:#eee;
    font-family:"Creepster", cursive;
    -webkit-font-smoothing:antialiased;
    -moz-osx-font-smoothing:grayscale;
  }

  /* Background image + fog canvas overlay */
  #bg {
    position:fixed; inset:0; z-index:-3;
    background: url('https://i.ibb.co/ZTDCxmr/haunted-mansion-night.jpg') center/cover no-repeat;
    filter:brightness(0.28) contrast(1.05);
  }
  #fog {
    position:fixed; inset:0; z-index:-2; pointer-events:none;
    mix-blend-mode:screen; opacity:0.9;
  }

  /* flash for thunder */
  #flash {
    position:fixed; inset:0; background:#fff; z-index:9999; opacity:0; pointer-events:none;
    transition: opacity 0.1s linear;
  }

  /* top controls */
  header{
    display:flex; gap:12px; align-items:center; justify-content:space-between;
    padding:14px 18px; position:relative;
  }
  .title { font-size:clamp(22px,5vw,36px); color:var(--accent); text-shadow:0 0 12px rgba(255,60,0,0.6); }
  .controls { display:flex; gap:8px; align-items:center; }

  .toggle {
    background:transparent; border:2px solid rgba(255,255,255,0.06); color:#fff; padding:8px 12px; border-radius:10px;
    font-family:inherit; cursor:pointer;
  }
  .toggle.active { background:var(--accent); color:#000; border-color:transparent; }

  /* main story panel */
  main{
    max-width:900px; margin:10px auto 40px; padding:18px; box-sizing:border-box;
  }
  #story {
    background:var(--panel);
    border:3px solid rgba(255,60,0,0.22);
    border-radius:14px; padding:22px; font-size:1.1rem; line-height:1.6;
    box-shadow:0 12px 40px rgba(0,0,0,0.6);
    transition: transform 0.25s ease, box-shadow 0.25s ease;
  }

  .choices{ display:flex; flex-direction:column; gap:12px; margin-top:18px; }
  button.choice{
    background:#141414; color:var(--accent); border:2px solid var(--accent); font-size:1rem;
    padding:12px; border-radius:10px; cursor:pointer; transition: transform .12s ease, background .12s;
    text-align:center;
  }
  button.choice:hover{ transform:scale(1.04); background:var(--accent); color:#111; }

  /* sanity bar */
  .sanity-wrap{ width:90%; max-width:420px; margin:10px auto 6px; text-align:center; }
  #sanity-bar{ height:18px; width:100%; background:#141414; border-radius:10px; overflow:hidden; border:2px solid rgba(255,255,255,0.06); }
  #sanity-fill{ height:100%; width:100%; background:var(--sanity-high); transition:width .35s ease, background .35s; }

  #sanity-label{ font-size:0.9rem; color:rgba(255,255,255,0.7); margin-top:6px; }

  /* shake animation (applied to #story) */
  .shake { animation:shake 0.45s; }
  @keyframes shake {
    0%{ transform:translateX(0); } 25%{ transform:translateX(-12px); } 50%{ transform:translateX(10px);} 75%{ transform:translateX(-8px);} 100%{ transform:translateX(0);}
  }

  /* pulse when sanity low */
  .pulse { animation: pulse 1s infinite; }
  @keyframes pulse {
    0%{ box-shadow:0 0 8px rgba(255,0,0,0.25); } 50%{ box-shadow:0 0 20px rgba(255,0,0,0.6);} 100%{ box-shadow:0 0 8px rgba(255,0,0,0.25); }
  }

  /* small footer */
  footer{ text-align:center; margin-top:12px; color:rgba(255,255,255,0.5); font-size:0.9rem; }

  /* custom cursor */
  body.custom-cursor *{ cursor: url('data:image/svg+xml;utf8,<svg xmlns="http://www.w3.org/2000/svg" width="48" height="48"><text y="35" font-size="36">🖐️</text></svg>') 16 16, auto; }

  /* responsive */
  @media (max-width:640px){
    #story{ padding:16px; font-size:1rem; }
    button.choice{ font-size:0.95rem; padding:10px; }
  }
</style>
</head>
<body class="custom-cursor">

<div id="bg"></div>
<canvas id="fog"></canvas>
<div id="flash"></div>

<header>
  <div class="title">🏚️ La Malédiction du Manoir — Immersive</div>
  <div class="controls">
    <button id="soundToggle" class="toggle">🔊 Son</button>
    <button id="fastToggle" class="toggle">⚡ Mode rapide</button>
    <button id="cursorToggle" class="toggle">🖐️ Curseur</button>
  </div>
</header>

<div class="sanity-wrap" aria-hidden="false">
  <div id="sanity-bar"><div id="sanity-fill"></div></div>
  <div id="sanity-label">Santé mentale : <span id="sanityValue">100</span>/100</div>
</div>

<main>
  <div id="story" role="main" aria-live="polite"></div>
</main>

<footer>© Manoir — Clique pour choisir. Bonne chance... si tu l'oses.</footer>

<!-- audio -->
<audio id="music" src="https://cdn.pixabay.com/download/audio/2022/10/28/audio_5e6f6e7f2c.mp3?filename=horror-ambient-113870.mp3" loop></audio>
<audio id="thunder" src="https://actions.google.com/sounds/v1/weather/thunder_strike.ogg"></audio>
<audio id="shock" src="https://actions.google.com/sounds/v1/foley/metal_drop.ogg"></audio>

<script>
/* ============================
   Données & état
   ============================ */

const storyEl = document.getElementById('story');
const music = document.getElementById('music');
const thunder = document.getElementById('thunder');
const shock = document.getElementById('shock');
const flash = document.getElementById('flash');
const fogCanvas = document.getElementById('fog');
const sanityFill = document.getElementById('sanity-fill');
const sanityValue = document.getElementById('sanityValue');

let sanity = 100;
let fastMode = false;
let soundOn = true;
let customCursor = true;

/* Scenes with optional effects:
   - sanity: integer change applied when entering scene
   - flash: boolean trigger thunder flash
   - shake: boolean trigger screen shake
   - fog: intensity 0..1
   - autoNext: if fastMode enabled, auto-advance to first choice after delay (ms)
*/
const scenes = {
  start: {
    text: "La pluie bat son plein. Tu t’abrites dans un vieux manoir... Une porte grince. 🕯️<br>Souhaites-tu entrer ou fuir ?",
    sanity: 0,
    choices: [
      { text: "Entrer dans le manoir", next: "hall" },
      { text: "Rester dehors", next: "forest" }
    ]
  },
  hall: {
    text: "Tu entres dans le grand hall. Des portraits te fixent... Une ombre passe au fond du couloir. 😨<br>Que fais-tu ?",
    sanity: -5, flash:true, fog:0.25,
    choices: [
      { text: "Suivre l’ombre", next: "shadow" },
      { text: "Explorer la bibliothèque", next: "library" },
      { text: "Inspecter le tapis au sol", next: "trapdoor" }
    ]
  },
  forest: {
    text: "Tu décides de rester dehors. La foudre frappe un arbre. 🌩️ La porte s’est refermée. Tu es coincé dehors... à jamais.",
    sanity: +5, flash:true,
    end: "Tu as survécu, mais le mystère restera entier. 🌲"
  },
  shadow: {
    text: "Tu avances dans le couloir sombre. L’ombre t’attendait... c’est un miroir ! Ton reflet te sourit avec des crocs. 🪞",
    sanity: -12, flash:true, shake:true, fog:0.5,
    choices: [
      { text: "Briser le miroir", next: "curse" },
      { text: "Regarder de plus près", next: "trap" }
    ]
  },
  library: {
    text: "La bibliothèque est pleine de livres poussiéreux. Une voix murmure ton nom... 📖",
    sanity: -6, fog:0.15,
    choices: [
      { text: "Lire le grimoire", next: "ritualPage" },
      { text: "Refermer vite et sortir", next: "escape" }
    ]
  },
  trapdoor: {
    text: "Sous le tapis, une trappe s’ouvre vers les profondeurs. Un souffle froid s’en échappe... 💨",
    sanity: -6, fog:0.3,
    choices: [
      { text: "Descendre dans le sous-sol", next: "cellar" },
      { text: "Ignorer et retourner explorer", next: "hall" }
    ]
  },
  cellar: {
    text: "Tu descends dans le sous-sol humide. Des chaînes pendent au mur et une inscription sanglante t'effraie... 🔥",
    sanity: -12, fog:0.5, shake:true,
    choices: [
      { text: "Avancer vers la lumière", next: "crypt" },
      { text: "Remonter discrètement", next: "hall" }
    ]
  },
  crypt: {
    text: "Tu découvres une crypte. Une inscription dit : « Celui qui réveillera les âmes devra accomplir le rituel ». 💀",
    sanity: -10, fog:0.6,
    choices: [
      { text: "Accomplir le rituel", next: "ritual" },
      { text: "Fuir avant qu’il ne soit trop tard", next: "escape" }
    ]
  },
  ritualPage: {
    text: "Le grimoire évoque la clef d’un ancien rite. Un dessin indique comment invoquer ou libérer les âmes... 📜",
    sanity: -8, fog:0.25,
    choices:[
      { text:"Suivre les instructions du livre", next:"ritual" },
      { text:"Ne pas toucher au rituel", next:"escape" }
    ]
  },
  ritual: {
    text: "Tu allumes les cierges... Les murs tremblent, les âmes s’éveillent. 👁️ Tu sens une force t’envahir.",
    sanity: -18, flash:true, fog:0.8, shake:true,
    choices: [
      { text: "Libérer les esprits", next: "ghost" },
      { text: "Canaliser leur pouvoir", next: "curse" },
      { text: "Tenter d’arrêter le rituel", next: "collapse" }
    ]
  },
  curse: {
    text: "Le miroir se brise. Une énergie t’enveloppe... Tu deviens le nouveau gardien du manoir. 🩸",
    sanity: -30, flash:true, end: "Tu es maudit à jamais. 👁️"
  },
  trap: {
    text: "Ton reflet t’attrape et t’entraîne dans le miroir. 🌑",
    sanity: -25, shake:true, end: "Tu as disparu dans une autre dimension."
  },
  ghost: {
    text: "Les esprits hurlent de joie en retrouvant leur liberté. Une lumière envahit le manoir... 👻",
    sanity: +18, fog:0.1, end: "Bravo, tu as brisé la malédiction ! Les âmes sont libres. 🕊️"
  },
  collapse: {
    text: "Le rituel échoue. Les murs s’effondrent... 🌪️",
    sanity: -20, shake:true, end: "Le manoir s’effondre sur toi. Ton nom sera murmuré à chaque orage. ⚡"
  },
  escape: {
    text: "Tu cours vers la sortie, mais la porte claque ! Était-ce un rêve ? 🌧️",
    sanity: +6, end: "Tu es libre, mais hanté à jamais par ce souvenir..."
  },
  insanity: {
    text: "Ton esprit se fissure. Tu entends des voix... les murs respirent... 🌀",
    sanity: -100, flash:true, shake:true, end: "Tu as sombré dans la folie. Le manoir t’a pris ton âme. 💀"
  }
};

/* ============================
   Affichage & logique
   ============================ */

function clamp(v,min,max){ return Math.max(min,Math.min(max,v)); }

function setSanity(delta){
  sanity = clamp(sanity + delta, 0, 100);
  sanityFill.style.width = sanity + "%";
  sanityValue.innerText = sanity;
  // color
  if(sanity>60) sanityFill.style.background = getComputedStyle(document.documentElement).getPropertyValue('--sanity-high') || 'linear-gradient(90deg,#00ff99,#0099ff)';
  else if(sanity>30) sanityFill.style.background = getComputedStyle(document.documentElement).getPropertyValue('--sanity-mid') || 'linear-gradient(90deg,#ffcc00,#ff6600)';
  else sanityFill.style.background = getComputedStyle(document.documentElement).getPropertyValue('--sanity-low') || 'linear-gradient(90deg,#ff0000,#660000)';
  // pulse when low
  if(sanity <= 30) storyEl.classList.add('pulse'); else storyEl.classList.remove('pulse');
  // adjust music volume by sanity (low sanity => more intense volume)
  if(soundOn){
    const vol = clamp(0.12 + (100 - sanity)/220, 0.05, 0.45); // sanity low => louder
    music.volume = vol;
  } else {
    music.volume = 0;
  }
  // if sanity zero -> insanity
  if(sanity <= 0){
    showScene('insanity');
  }
}

function triggerFlash(){
  flash.style.opacity = '1';
  thunder.currentTime = 0;
  if(soundOn) thunder.play();
  setTimeout(()=>{ flash.style.opacity = '0'; }, 120);
}

function triggerShake(){
  storyEl.classList.add('shake');
  if(soundOn){ shock.currentTime = 0; shock.play(); }
  setTimeout(()=> storyEl.classList.remove('shake'), 460);
}

/* Fog: animated Perlin-like blobs using canvas */
const fctx = fogCanvas.getContext('2d');
let FWIDTH, FHEIGHT, fogParticles = [];
function resizeFog(){
  fogCanvas.width = window.innerWidth;
  fogCanvas.height = window.innerHeight;
  FWIDTH = fogCanvas.width; FHEIGHT = fogCanvas.height;
}
window.addEventListener('resize', resizeFog, {passive:true});
function spawnFog(intensity=0.4){
  fogParticles = [];
  const count = Math.round(25 + intensity*120);
  for(let i=0;i<count;i++){
    fogParticles.push({
      x: Math.random()*FWIDTH,
      y: Math.random()*FHEIGHT,
      r: 60 + Math.random()*140,
      vx: (Math.random()-0.5)*0.2,
      vy: (Math.random()-0.5)*0.2,
      a: 0.05 + Math.random()*0.25,
      hue: Math.round(200 - Math.random()*40)
    });
  }
}
function animateFog(){
  fctx.clearRect(0,0,FWIDTH,FHEIGHT);
  fogParticles.forEach(p=>{
    fctx.beginPath();
    const g = fctx.createRadialGradient(p.x,p.y,p.r*0.1,p.x,p.y,p.r);
    g.addColorStop(0, `hsla(${p.hue},80%,90%,${p.a*0.12})`);
    g.addColorStop(1, `hsla(${p.hue},80%,50%,0)`);
    fctx.fillStyle = g;
    fctx.fillRect(p.x-p.r,p.y-p.r,p.r*2,p.r*2);
    p.x += p.vx; p.y += p.vy;
    if(p.x < -p.r) p.x = FWIDTH+p.r;
    if(p.x > FWIDTH+p.r) p.x = -p.r;
    if(p.y < -p.r) p.y = FHEIGHT+p.r;
    if(p.y > FHEIGHT+p.r) p.y = -p.r;
  });
  requestAnimationFrame(animateFog);
}

/* initialize fog */
resizeFog();
spawnFog(0.2);
animateFog();

/* show scene */
let currentScene = 'start';
function showScene(name){
  const scene = scenes[name];
  if(!scene) return;
  currentScene = name;

  // apply visual effects
  if(scene.flash) triggerFlash();
  if(scene.shake) triggerShake();
  const fogIntensity = (typeof scene.fog === 'number') ? scene.fog : 0.18;
  spawnFog(fogIntensity);

  // update sanity from scene
  if(typeof scene.sanity !== 'undefined') setSanity(scene.sanity);

  // build HTML
  let html = `<p>${scene.text}</p>`;
  if(scene.choices){
    html += `<div class="choices">`;
    scene.choices.forEach(choice=>{
      html += `<button class="choice" onclick="showScene('${choice.next}')">${choice.text}</button>`;
    });
    html += `</div>`;
    // auto-advance when fastMode is on: pick first choice after short delay
    if(fastMode){
      // small visual delay so user can see
      const delay = 600; // ms
      setTimeout(()=>{
        if(currentScene === name && scenes[name].choices && scenes[name].choices[0]){
          showScene(scenes[name].choices[0].next);
        }
      }, delay);
    }
  } else if(scene.end){
    html += `<div id="end" style="margin-top:18px">${scene.end}</div>`;
    html += `<div class="choices" style="margin-top:18px"><button class="choice" onclick="restart()">🔁 Rejouer</button></div>`;
  }
  storyEl.innerHTML = html;
}

/* restart */
function restart(){
  sanity = 100;
  setSanity(0);
  spawnFog(0.2);
  showScene('start');
}

/* ============================
   UI Controls
   ============================ */
document.getElementById('fastToggle').addEventListener('click', ()=>{
  fastMode = !fastMode;
  document.getElementById('fastToggle').classList.toggle('active', fastMode);
  document.getElementById('fastToggle').innerText = fastMode ? '⚡ Mode rapide (ON)' : '⚡ Mode rapide';
});

document.getElementById('soundToggle').addEventListener('click', ()=>{
  soundOn = !soundOn;
  document.getElementById('soundToggle').classList.toggle('active', soundOn);
  document.getElementById('soundToggle').innerText = soundOn ? '🔊 Son (ON)' : '🔊 Son';
  if(soundOn) { music.volume = 0.18; music.play(); } else { music.volume = 0; }
});

document.getElementById('cursorToggle').addEventListener('click', ()=>{
  customCursor = !customCursor;
  document.getElementById('cursorToggle').classList.toggle('active', customCursor);
  document.body.classList.toggle('custom-cursor', customCursor);
});

/* start music */
music.volume = 0.18;
if(soundOn) music.play().catch(()=>{/* autoplay blocked maybe */});

/* initial show */
setSanity(0);
showScene('start');

/* keyboard shortcuts: Enter to pick first choice */
window.addEventListener('keydown', (e)=>{
  if(e.key === 'Enter'){
    const first = storyEl.querySelector('.choice');
    if(first) first.click();
  }
});

/* small helper: clicking background toggles a thunder */
document.getElementById('bg').addEventListener('click', ()=> { triggerFlash(); });

</script>
</body>
</html>
