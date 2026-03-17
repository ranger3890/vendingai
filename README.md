[face-scan.html](https://github.com/user-attachments/files/26042021/face-scan.html)
<!DOCTYPE html>
<html lang="en">
<head>
<meta charset="UTF-8"/>
<title>Face + Object Scan</title>
<style>
@import url('https://fonts.googleapis.com/css2?family=Space+Mono:wght@400;700&display=swap');
*, *::before, *::after { box-sizing: border-box; margin: 0; padding: 0; }
:root {
  --bg: #0a0a0f; --accent: #00ffe0; --accent2: #ff3cac;
  --dim: #666680; --border: #2a2a3a;
}
body {
  background: var(--bg); color: #e8e8f0;
  font-family: 'Space Mono', monospace, 'Courier New';
  min-height: 100vh; display: flex; flex-direction: column;
  align-items: center; padding: 2rem 1rem;
}
body::before {
  content: ''; position: fixed; inset: 0; pointer-events: none; z-index: 999;
  background: repeating-linear-gradient(0deg, transparent, transparent 2px,
    rgba(0,255,224,0.012) 2px, rgba(0,255,224,0.012) 4px);
}
h1 {
  font-size: clamp(1.6rem, 5vw, 2.8rem); font-weight: 700; letter-spacing: -0.02em;
  background: linear-gradient(90deg, var(--accent), var(--accent2));
  -webkit-background-clip: text; -webkit-text-fill-color: transparent;
  margin-bottom: 0.3rem;
}
.sub { color: var(--dim); font-size: 0.72rem; letter-spacing: 0.18em;
  text-transform: uppercase; margin-bottom: 2rem; }
.wrapper {
  position: relative; display: inline-block;
  border: 1px solid var(--border); border-radius: 4px; overflow: hidden;
  box-shadow: 0 0 40px rgba(0,255,224,0.07);
}
.wrapper::before, .wrapper::after, .cbr, .cbl {
  content: ''; position: absolute; width: 20px; height: 20px;
  z-index: 10; pointer-events: none;
}
.wrapper::before { top:8px; left:8px; border-top:2px solid var(--accent); border-left:2px solid var(--accent); }
.wrapper::after  { top:8px; right:8px; border-top:2px solid var(--accent); border-right:2px solid var(--accent); }
.cbr { bottom:8px; right:8px; border-bottom:2px solid var(--accent); border-right:2px solid var(--accent); }
.cbl { bottom:8px; left:8px; border-bottom:2px solid var(--accent); border-left:2px solid var(--accent); }
#videoWrap { position: relative; display: block; }
#v  { display: block; max-width: min(640px, 100%); transform: scaleX(-1); }
#oc { position: absolute; top: 0; left: 0; width: 100%; height: 100%; pointer-events: none; }
.scan { position:absolute; left:0; right:0; height:2px;
  background: linear-gradient(90deg, transparent, var(--accent), transparent);
  opacity: 0.5; animation: scan 3s linear infinite; pointer-events:none; z-index:5; }
@keyframes scan { 0%{top:0%} 100%{top:100%} }
.controls { margin-top:1.5rem; display:flex; gap:1rem; flex-wrap:wrap; justify-content:center; }
button {
  font-family: inherit; font-size:0.78rem; font-weight:700;
  letter-spacing:0.12em; text-transform:uppercase; padding:0.65rem 1.6rem;
  border:1px solid var(--accent); background:transparent; color:var(--accent);
  cursor:pointer; border-radius:2px; transition:all 0.2s; position:relative; overflow:hidden;
}
button::before {
  content:''; position:absolute; inset:0; background:var(--accent);
  transform:scaleX(0); transform-origin:left; transition:transform 0.2s; z-index:-1;
}
button:hover { color: var(--bg); }
button:hover::before { transform: scaleX(1); }
button:disabled { opacity:0.35; cursor:not-allowed; }
button:disabled::before { display:none; }
#count {
  margin-top:0.8rem; font-size:2.4rem; font-weight:700; color:var(--accent);
  text-align:center; min-height:3rem; text-shadow:0 0 20px rgba(0,255,224,0.4);
}
#count span { font-size:0.8rem; color:var(--dim); display:block; margin-top:-0.4rem; }
#status { margin-top:0.6rem; font-size:0.72rem; letter-spacing:0.1em; color:var(--dim); text-align:center; }
#status.ok { color: var(--accent); }
#status.error { color: var(--accent2); }
#items-panel {
  margin-top: 1.4rem; width: min(640px, 100%);
  border: 1px solid var(--border); border-radius: 4px; overflow: hidden;
}
#items-panel h2 {
  font-size: 0.7rem; letter-spacing: 0.15em; text-transform: uppercase;
  color: var(--dim); padding: 0.6rem 1rem;
  border-bottom: 1px solid var(--border); background: rgba(255,255,255,0.02);
  display: flex; justify-content: space-between; align-items: center;
}
#items-panel h2 .model-badge { font-size: 0.58rem; color: #444466; letter-spacing: 0.08em; }
#items-grid {
  display: flex; flex-wrap: wrap; gap: 0.5rem;
  padding: 0.8rem 1rem; min-height: 52px;
}
#items-grid:empty::after {
  content: 'No items detected yet.'; color: var(--dim);
  font-size: 0.68rem;
}
.item-chip {
  display: flex; align-items: center; gap: 0.4rem;
  padding: 0.25rem 0.7rem; border-radius: 2px; font-size: 0.68rem;
  border: 1px solid; white-space: nowrap;
  animation: chipIn 0.25s ease;
}
@keyframes chipIn { from { opacity:0; transform:scale(0.92); } to { opacity:1; transform:none; } }
.item-chip .dot { width: 7px; height: 7px; border-radius: 50%; flex-shrink: 0; }
.item-chip .cat { opacity: 0.55; font-size: 0.58rem; margin-left: 0.2rem; }
.item-chip .conf { opacity: 0.45; font-size: 0.58rem; }
#notif-log {
  margin-top: 1.5rem; width: min(480px, 100%);
  border: 1px solid var(--border); border-radius: 4px; overflow: hidden;
}
#notif-log h2 {
  font-size: 0.7rem; letter-spacing: 0.15em; text-transform: uppercase;
  color: var(--dim); padding: 0.6rem 1rem;
  border-bottom: 1px solid var(--border); background: rgba(255,255,255,0.02);
}
#notif-list { max-height: 140px; overflow-y: auto; padding: 0.4rem 0; }
#notif-list:empty::after {
  content: 'No alerts sent yet.'; display: block; text-align: center;
  color: var(--dim); font-size: 0.68rem; padding: 0.8rem;
}
.notif-item {
  display: flex; justify-content: space-between; align-items: center;
  padding: 0.35rem 1rem; font-size: 0.68rem;
  border-bottom: 1px solid rgba(255,255,255,0.04);
  animation: fadeIn 0.3s ease;
}
@keyframes fadeIn { from { opacity:0; transform:translateY(-4px); } to { opacity:1; transform:none; } }
.notif-item .msg { color: var(--accent); }
.notif-item .time { color: var(--dim); font-size: 0.62rem; }
.notif-item.fail .msg { color: var(--accent2); }
#tip {
  margin-top: 1rem; max-width: 480px; text-align: center;
  font-size: 0.68rem; color: var(--dim); line-height: 1.7;
  border: 1px solid var(--border); padding: 0.8rem 1.2rem; border-radius: 4px;
}
</style>
</head>
<body>

<h1>FACE + OBJECT SCAN</h1>
<p class="sub">Vending Machine Prototype</p>

<div class="wrapper">
  <div id="videoWrap">
    <video id="v" width="640" height="480" muted playsinline autoplay></video>
    <canvas id="oc"></canvas>
  </div>
  <div class="scan"></div>
  <div class="cbr"></div><div class="cbl"></div>
</div>

<div class="controls">
  <button id="startBtn">Start Camera</button>
  <button id="stopBtn" disabled>Stop</button>
</div>

<div id="count"></div>
<div id="status" class="ok">Loading models…</div>

<div id="items-panel">
  <h2>🔍 Detected Items <span class="model-badge">COCO-SSD · 80 classes</span></h2>
  <div id="items-grid"></div>
</div>

<div id="notif-log">
  <h2>📱 Phone Alerts</h2>
  <div id="notif-list"></div>
</div>

<div id="tip"></div>

<script src="https://cdn.jsdelivr.net/npm/@tensorflow/tfjs@4.17.0/dist/tf.min.js"></script>
<script src="https://cdn.jsdelivr.net/npm/@tensorflow-models/coco-ssd@2.2.3/dist/coco-ssd.min.js"></script>
<script src="https://cdn.jsdelivr.net/npm/face-api.js@0.22.2/dist/face-api.min.js"></script>
<script>
const video     = document.getElementById('v');
const overlayC  = document.getElementById('oc');
const ctx       = overlayC.getContext('2d');
const statusEl  = document.getElementById('status');
const countEl   = document.getElementById('count');
const tipEl     = document.getElementById('tip');
const startBtn  = document.getElementById('startBtn');
const stopBtn   = document.getElementById('stopBtn');
const notifList = document.getElementById('notif-list');
const itemsGrid = document.getElementById('items-grid');

let stream = null, running = false, animId = null;
let faceReady = false, objReady = false, cocoModel = null;
let lastNotifTime = 0, frameCount = 0;
let lastObjDetections = []; // persisted between frames

const NTFY_TOPIC = 'myfacealert123';
const FACE_MODEL_URL = 'https://cdn.jsdelivr.net/npm/@vladmandic/face-api/model';

/* ── Category map: COCO label → { cat, color } ── */
const CATEGORY_MAP = {
  person:            { cat: 'Person',      color: '#00ffe0' },
  bicycle:           { cat: 'Vehicle',     color: '#ffe600' },
  car:               { cat: 'Vehicle',     color: '#ffe600' },
  motorcycle:        { cat: 'Vehicle',     color: '#ffe600' },
  airplane:          { cat: 'Vehicle',     color: '#ffe600' },
  bus:               { cat: 'Vehicle',     color: '#ffe600' },
  train:             { cat: 'Vehicle',     color: '#ffe600' },
  truck:             { cat: 'Vehicle',     color: '#ffe600' },
  boat:              { cat: 'Vehicle',     color: '#ffe600' },
  'traffic light':   { cat: 'Street',      color: '#fdcb6e' },
  'fire hydrant':    { cat: 'Street',      color: '#fdcb6e' },
  'stop sign':       { cat: 'Street',      color: '#fdcb6e' },
  'parking meter':   { cat: 'Street',      color: '#fdcb6e' },
  bench:             { cat: 'Street',      color: '#fdcb6e' },
  bird:              { cat: 'Animal',      color: '#55efc4' },
  cat:               { cat: 'Animal',      color: '#55efc4' },
  dog:               { cat: 'Animal',      color: '#55efc4' },
  horse:             { cat: 'Animal',      color: '#55efc4' },
  sheep:             { cat: 'Animal',      color: '#55efc4' },
  cow:               { cat: 'Animal',      color: '#55efc4' },
  elephant:          { cat: 'Animal',      color: '#55efc4' },
  bear:              { cat: 'Animal',      color: '#55efc4' },
  zebra:             { cat: 'Animal',      color: '#55efc4' },
  giraffe:           { cat: 'Animal',      color: '#55efc4' },
  backpack:          { cat: 'Bag',         color: '#e17055' },
  umbrella:          { cat: 'Accessory',   color: '#e17055' },
  handbag:           { cat: 'Bag',         color: '#e17055' },
  tie:               { cat: 'Clothing',    color: '#e17055' },
  suitcase:          { cat: 'Bag',         color: '#e17055' },
  frisbee:           { cat: 'Sports',      color: '#00cec9' },
  skis:              { cat: 'Sports',      color: '#00cec9' },
  snowboard:         { cat: 'Sports',      color: '#00cec9' },
  'sports ball':     { cat: 'Sports',      color: '#00cec9' },
  kite:              { cat: 'Sports',      color: '#00cec9' },
  'baseball bat':    { cat: 'Sports',      color: '#00cec9' },
  'baseball glove':  { cat: 'Sports',      color: '#00cec9' },
  skateboard:        { cat: 'Sports',      color: '#00cec9' },
  surfboard:         { cat: 'Sports',      color: '#00cec9' },
  'tennis racket':   { cat: 'Sports',      color: '#00cec9' },
  bottle:            { cat: 'Drink',       color: '#fd79a8' },
  'wine glass':      { cat: 'Drink',       color: '#fd79a8' },
  cup:               { cat: 'Drink',       color: '#fd79a8' },
  fork:              { cat: 'Utensil',     color: '#ffeaa7' },
  knife:             { cat: 'Utensil',     color: '#ffeaa7' },
  spoon:             { cat: 'Utensil',     color: '#ffeaa7' },
  bowl:              { cat: 'Utensil',     color: '#ffeaa7' },
  banana:            { cat: 'Food',        color: '#ff9f43' },
  apple:             { cat: 'Food',        color: '#ff9f43' },
  sandwich:          { cat: 'Food',        color: '#ff9f43' },
  orange:            { cat: 'Food',        color: '#ff9f43' },
  broccoli:          { cat: 'Food',        color: '#ff9f43' },
  carrot:            { cat: 'Food',        color: '#ff9f43' },
  'hot dog':         { cat: 'Food',        color: '#ff9f43' },
  pizza:             { cat: 'Food',        color: '#ff9f43' },
  donut:             { cat: 'Food',        color: '#ff9f43' },
  cake:              { cat: 'Food',        color: '#ff9f43' },
  chair:             { cat: 'Furniture',   color: '#a29bfe' },
  couch:             { cat: 'Furniture',   color: '#a29bfe' },
  'potted plant':    { cat: 'Plant',       color: '#6ab04c' },
  bed:               { cat: 'Furniture',   color: '#a29bfe' },
  'dining table':    { cat: 'Furniture',   color: '#a29bfe' },
  toilet:            { cat: 'Furniture',   color: '#a29bfe' },
  tv:                { cat: 'Electronics', color: '#b084ff' },
  laptop:            { cat: 'Electronics', color: '#b084ff' },
  mouse:             { cat: 'Electronics', color: '#b084ff' },
  remote:            { cat: 'Electronics', color: '#b084ff' },
  keyboard:          { cat: 'Electronics', color: '#b084ff' },
  'cell phone':      { cat: 'Electronics', color: '#b084ff' },
  microwave:         { cat: 'Appliance',   color: '#636e72' },
  oven:              { cat: 'Appliance',   color: '#636e72' },
  toaster:           { cat: 'Appliance',   color: '#636e72' },
  sink:              { cat: 'Appliance',   color: '#636e72' },
  refrigerator:      { cat: 'Appliance',   color: '#636e72' },
  book:              { cat: 'Books',       color: '#74b9ff' },
  clock:             { cat: 'Decor',       color: '#badc58' },
  vase:              { cat: 'Decor',       color: '#badc58' },
  scissors:          { cat: 'Tool',        color: '#dfe6e9' },
  'teddy bear':      { cat: 'Toy',         color: '#fab1a0' },
  'hair drier':      { cat: 'Appliance',   color: '#dfe6e9' },
  toothbrush:        { cat: 'Personal',    color: '#dfe6e9' },
};

function getCategory(label) {
  return CATEGORY_MAP[label.toLowerCase()] || { cat: 'Object', color: '#aaaaaa' };
}

/* ── helpers ── */
function setStatus(msg, cls = '') { statusEl.textContent = msg; statusEl.className = cls; }

function logNotif(msg, ok = true) {
  const el = document.createElement('div');
  el.className = 'notif-item' + (ok ? '' : ' fail');
  el.innerHTML = `<span class="msg">${msg}</span><span class="time">${new Date().toLocaleTimeString()}</span>`;
  notifList.prepend(el);
}

async function sendPhoneAlert(count) {
  const now = Date.now();
  if (now - lastNotifTime < 10000) return;
  lastNotifTime = now;
  const body = `📸 Customer detected! ${count} person${count > 1 ? 's' : ''} at the vending machine.`;
  try {
    const res = await fetch(`https://ntfy.sh/${NTFY_TOPIC}`, {
      method: 'POST', body,
      headers: { 'Title': 'Vending Machine Alert', 'Priority': 'default', 'Tags': 'vending,face,camera' }
    });
    res.ok ? logNotif(`✅ Alert sent: ${count} face${count > 1 ? 's' : ''} detected`)
           : logNotif(`❌ ntfy error ${res.status}`, false);
  } catch { logNotif('❌ Could not reach ntfy.sh', false); }
}

/* ── model loading ── */
function checkReady() {
  if (faceReady && objReady) { setStatus('Models ready — click Start Camera', 'ok'); startBtn.disabled = false; }
}

async function loadModels() {
  startBtn.disabled = true;
  setStatus('Loading face + object models…');
  await Promise.all([
    faceapi.nets.tinyFaceDetector.loadFromUri(FACE_MODEL_URL)
      .then(() => { faceReady = true; checkReady(); })
      .catch(e => { console.error(e); setStatus('❌ Failed to load face model.', 'error'); }),
    cocoSsd.load({ base: 'lite_mobilenet_v2' })
      .then(m => { cocoModel = m; objReady = true; checkReady(); })
      .catch(e => { console.error(e); setStatus('❌ Failed to load object model.', 'error'); }),
  ]);
}

/* ── camera ── */
startBtn.addEventListener('click', async () => {
  try {
    stream = await navigator.mediaDevices.getUserMedia({ video: { width:640, height:480, facingMode:'user' }, audio:false });
    video.srcObject = stream;
    await video.play();
    video.addEventListener('loadedmetadata', () => {
      overlayC.width = video.videoWidth; overlayC.height = video.videoHeight;
    }, { once: true });
    running = true; startBtn.disabled = true; stopBtn.disabled = false;
    setStatus('Camera active — scanning…', 'ok');
    loop();
  } catch (e) {
    setStatus(e.name === 'NotAllowedError'
      ? '❌ Camera permission denied. Click the camera icon in the address bar.'
      : '❌ ' + e.message, 'error');
  }
});

stopBtn.addEventListener('click', () => {
  running = false; cancelAnimationFrame(animId);
  if (stream) stream.getTracks().forEach(t => t.stop());
  ctx.clearRect(0, 0, overlayC.width, overlayC.height);
  countEl.innerHTML = ''; itemsGrid.innerHTML = '';
  startBtn.disabled = false; stopBtn.disabled = true;
  setStatus('Stopped.');
});

/* ── draw box helper ── */
function drawBox(x, y, w, h, color, topLabel, conf) {
  ctx.strokeStyle = color; ctx.lineWidth = 2;
  ctx.shadowColor = color; ctx.shadowBlur = 10;
  ctx.strokeRect(x, y, w, h); ctx.shadowBlur = 0;
  ctx.font = 'bold 11px monospace';
  const text = `${topLabel} ${conf}%`;
  const tw = ctx.measureText(text).width + 8;
  ctx.fillStyle = color + '33';
  ctx.fillRect(x, y > 18 ? y - 18 : y, tw, 18);
  ctx.fillStyle = color;
  ctx.fillText(text, x + 4, y > 18 ? y - 5 : y + 13);
}

/* ── items panel ── */
function updateItemsPanel(items) {
  const best = {};
  items.forEach(({ label, conf, color, cat }) => {
    if (!best[label] || conf > best[label].conf) best[label] = { conf, color, cat };
  });
  itemsGrid.innerHTML = '';
  Object.entries(best).forEach(([label, { conf, color, cat }]) => {
    const chip = document.createElement('div');
    chip.className = 'item-chip';
    chip.style.cssText = `border-color:${color}88;color:${color};background:${color}11`;
    chip.innerHTML =
      `<span class="dot" style="background:${color}"></span>` +
      `<strong>${label}</strong><span class="cat">[${cat}]</span>` +
      `<span class="conf">${conf}%</span>`;
    itemsGrid.appendChild(chip);
  });
}

/* ── main loop ── */
async function loop() {
  if (!running) return;

  if (video.readyState >= 2) {
    frameCount++;
    ctx.clearRect(0, 0, overlayC.width, overlayC.height);
    const W = overlayC.width, H = overlayC.height;
    const allItems = [];

    /* Face detection — every frame */
    try {
      const faceOpts = new faceapi.TinyFaceDetectorOptions({ inputSize: 320, scoreThreshold: 0.45 });
      const faces = await faceapi.detectAllFaces(video, faceOpts);
      const resized = faceapi.resizeResults(faces, { width: W, height: H });
      resized.forEach(det => {
        const { x, y, width, height } = det.box;
        const mx = W - x - width;
        const conf = Math.round(det.score * 100);
        drawBox(mx, y, width, height, '#00ffe0', 'FACE', conf);
        allItems.push({ label: 'person', conf, color: '#00ffe0', cat: 'Person' });
      });
      const n = resized.length;
      if (n > 0) sendPhoneAlert(n);
      countEl.innerHTML = n > 0 ? `${n}<span>${n === 1 ? 'face detected' : 'faces detected'}</span>` : '';
    } catch { /* skip frame */ }

    /* Object detection — every 3rd frame (CPU-friendly for Chromebook) */
    if (frameCount % 3 === 0 && cocoModel) {
      try {
        const preds = await cocoModel.detect(video, 10, 0.40);
        const objItems = [];
        preds.forEach(pred => {
          const label = pred.class.toLowerCase();
          if (label === 'person') return; // already handled above
          const [bx, by, bw, bh] = pred.bbox;
          const mx = W - bx - bw;
          const conf = Math.round(pred.score * 100);
          const { cat, color } = getCategory(label);
          drawBox(mx, by, bw, bh, color, label, conf);
          objItems.push({ label, conf, color, cat });
        });
        lastObjDetections = objItems;
      } catch { /* skip */ }
    } else {
      // Redraw last known object boxes from stored data (no bbox — just panel)
      lastObjDetections.forEach(item => allItems.push(item));
    }

    lastObjDetections.forEach(item => allItems.push(item));
    updateItemsPanel(allItems);
  }

  animId = requestAnimationFrame(loop);
}

loadModels();
</script>
</body>
</html>
