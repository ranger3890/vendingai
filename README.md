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
text-transform: uppercase; margin-bottom: 1rem; }

/* ── mode switcher ── */
#mode-bar {
margin-bottom: 1rem; width: min(640px, 100%);
border: 1px solid var(–border); border-radius: 4px; overflow: hidden;
}
#mode-bar-title {
background: rgba(255,255,255,0.03); padding: 0.4rem 0.8rem;
color: var(–dim); border-bottom: 1px solid var(–border);
font-size: 0.62rem; text-transform: uppercase; letter-spacing: 0.14em;
}
#mode-buttons {
display: flex; flex-wrap: wrap; gap: 0;
}
.mode-btn {
flex: 1; min-width: 140px;
font-family: inherit; font-size: 0.68rem; font-weight: 700;
letter-spacing: 0.1em; text-transform: uppercase;
padding: 0.7rem 0.5rem; border: none; border-right: 1px solid var(–border);
background: transparent; color: var(–dim); cursor: pointer;
transition: all 0.2s; position: relative; text-align: center;
}
.mode-btn:last-child { border-right: none; }
.mode-btn:hover { color: var(–accent); background: rgba(0,255,224,0.04); }
.mode-btn.active { color: var(–bg); background: var(–accent); }
.mode-btn.disabled-mode { opacity: 0.3; cursor: not-allowed; }
.mode-btn .badge {
display: block; font-size: 0.52rem; opacity: 0.7;
font-weight: 400; letter-spacing: 0.06em; margin-top: 0.15rem;
}
.mode-btn.active .badge { opacity: 0.8; }

/* ── video wrapper ── */
.wrapper {
position: relative; display: inline-block;
border: 1px solid var(–border); border-radius: 4px; overflow: hidden;
box-shadow: 0 0 40px rgba(0,255,224,0.07);
}
.wrapper::before, .wrapper::after, .cbr, .cbl {
content: ‘’; position: absolute; width: 20px; height: 20px;
z-index: 10; pointer-events: none;
}
.wrapper::before { top:8px; left:8px; border-top:2px solid var(–accent); border-left:2px solid var(–accent); }
.wrapper::after { top:8px; right:8px; border-top:2px solid var(–accent); border-right:2px solid var(–accent); }
.cbr { bottom:8px; right:8px; border-bottom:2px solid var(–accent); border-right:2px solid var(–accent); }
.cbl { bottom:8px; left:8px; border-bottom:2px solid var(–accent); border-left:2px solid var(–accent); }
#videoWrap { position: relative; display: block; }
#v { display: block; max-width: min(640px, 100%); transform: scaleX(-1); }
#oc { position: absolute; top: 0; left: 0; width: 100%; height: 100%; pointer-events: none; }
.scan { position:absolute; left:0; right:0; height:2px;
background: linear-gradient(90deg, transparent, var(–accent), transparent);
opacity: 0.5; animation: scan 3s linear infinite; pointer-events:none; z-index:5; }
@keyframes scan { 0%{top:0%} 100%{top:100%} }

.controls { margin-top:1.2rem; display:flex; gap:1rem; flex-wrap:wrap; justify-content:center; }
button {
font-family: inherit; font-size:0.78rem; font-weight:700;
letter-spacing:0.12em; text-transform:uppercase; padding:0.65rem 1.6rem;
border:1px solid var(–accent); background:transparent; color:var(–accent);
cursor:pointer; border-radius:2px; transition:all 0.2s; position:relative; overflow:hidden;
}
button::before {
content:’’; position:absolute; inset:0; background:var(–accent);
transform:scaleX(0); transform-origin:left; transition:transform 0.2s; z-index:-1;
}
button:hover { color: var(–bg); }
button:hover::before { transform: scaleX(1); }
button:disabled { opacity:0.35; cursor:not-allowed; }
button:disabled::before { display:none; }

#count {
margin-top:0.8rem; font-size:2.4rem; font-weight:700; color:var(–accent);
text-align:center; min-height:3rem; text-shadow:0 0 20px rgba(0,255,224,0.4);
}
#count span { font-size:0.8rem; color:var(–dim); display:block; margin-top:-0.4rem; }
#status { margin-top:0.5rem; font-size:0.72rem; letter-spacing:0.1em; color:var(–dim); text-align:center; }
#status.ok { color: var(–accent); }
#status.error { color: var(–accent2); }
#status.warn { color: #ffe600; }

/* ── diag ── */
#diag {
margin-top: 0.8rem; width: min(640px, 100%);
border: 1px solid #2a2a3a; border-radius: 4px; overflow: hidden;
font-size: 0.65rem; letter-spacing: 0.06em;
}
#diag-title { background: rgba(255,255,255,0.03); padding: 0.4rem 0.8rem;
color: var(–dim); border-bottom: 1px solid #2a2a3a;
font-size: 0.62rem; text-transform: uppercase; letter-spacing: 0.14em; }
#diag-body { padding: 0.5rem 0.8rem; display: flex; flex-direction: column; gap: 0.3rem; }
.drow { display: flex; justify-content: space-between; gap: 1rem; }
.drow .dk { color: #555570; }
.drow .dv { color: #e8e8f0; }
.drow .dv.ok { color: #00ffe0; }
.drow .dv.err { color: #ff3cac; }
.drow .dv.warn { color: #ffe600; }
#diag-log { margin-top: 0.3rem; border-top: 1px solid #1a1a2a;
padding: 0.4rem 0.8rem; max-height: 80px; overflow-y: auto;
display: flex; flex-direction: column; gap: 0.15rem; }
.dlog { font-size: 0.6rem; color: #555570; }
.dlog.info { color: #00ffe0; }
.dlog.warn { color: #ffe600; }
.dlog.err { color: #ff3cac; }

/* ── items panel ── */
#items-panel {
margin-top: 1rem; width: min(640px, 100%);
border: 1px solid var(–border); border-radius: 4px; overflow: hidden;
}
#items-panel h2 {
font-size: 0.7rem; letter-spacing: 0.15em; text-transform: uppercase;
color: var(–dim); padding: 0.6rem 1rem;
border-bottom: 1px solid var(–border); background: rgba(255,255,255,0.02);
display: flex; justify-content: space-between; align-items: center;
}
#items-panel h2 .model-badge { font-size: 0.58rem; color: #444466; }
#items-grid { display: flex; flex-wrap: wrap; gap: 0.5rem; padding: 0.8rem 1rem; min-height: 52px; }
#items-grid:empty::after { content: ‘No items detected yet.’; color: var(–dim); font-size: 0.68rem; }
.item-chip {
display: flex; align-items: center; gap: 0.4rem;
padding: 0.25rem 0.7rem; border-radius: 2px; font-size: 0.68rem;
border: 1px solid; white-space: nowrap; animation: chipIn 0.25s ease;
}
@keyframes chipIn { from{opacity:0;transform:scale(0.92)} to{opacity:1;transform:none} }
.item-chip .dot { width:7px; height:7px; border-radius:50%; flex-shrink:0; }
.item-chip .cat { opacity:0.55; font-size:0.58rem; margin-left:0.2rem; }
.item-chip .conf { opacity:0.45; font-size:0.58rem; }

/* ── notif ── */
#notif-log {
margin-top: 1rem; width: min(480px, 100%);
border: 1px solid var(–border); border-radius: 4px; overflow: hidden;
}
#notif-log h2 {
font-size: 0.7rem; letter-spacing: 0.15em; text-transform: uppercase;
color: var(–dim); padding: 0.6rem 1rem;
border-bottom: 1px solid var(–border); background: rgba(255,255,255,0.02);
}
#notif-list { max-height: 120px; overflow-y: auto; padding: 0.4rem 0; }
#notif-list:empty::after {
content: ‘No alerts sent yet.’; display: block; text-align: center;
color: var(–dim); font-size: 0.68rem; padding: 0.8rem;
}
.notif-item {
display: flex; justify-content: space-between; align-items: center;
padding: 0.35rem 1rem; font-size: 0.68rem;
border-bottom: 1px solid rgba(255,255,255,0.04); animation: fadeIn 0.3s ease;
}
@keyframes fadeIn { from{opacity:0;transform:translateY(-4px)} to{opacity:1;transform:none} }
.notif-item .msg { color: var(–accent); }
.notif-item .time { color: var(–dim); font-size: 0.62rem; }
.notif-item.fail .msg { color: var(–accent2); }
</style>

</head>
<body>

<h1>FACE + OBJECT SCAN</h1>
<p class="sub">Vending Machine Prototype</p>

<!-- MODE SWITCHER -->

<div id="mode-bar">
<div id="mode-bar-title">⚡ Detection Mode</div>
<div id="mode-buttons">
<button class="mode-btn active" data-mode="builtin" id="btn-builtin">
Chrome Built-in
<span class="badge">No CDN · Face only</span>
</button>
<button class="mode-btn" data-mode="faceapi" id="btn-faceapi">
Face-API.js
<span class="badge">CDN · Face only</span>
</button>
<button class="mode-btn" data-mode="full" id="btn-full">
Face + Objects
<span class="badge">CDN · TF.js + COCO-SSD</span>
</button>
</div>
</div>

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
<div id="status" class="ok">Select a mode above, then click Start Camera.</div>

<div id="diag">
<div id="diag-title">⚙ Diagnostics</div>
<div id="diag-body">
<div class="drow"><span class="dk">Active mode</span> <span class="dv ok" id="d-mode">Chrome Built-in</span></div>
<div class="drow"><span class="dk">Built-in API</span> <span class="dv" id="d-builtin">—</span></div>
<div class="drow"><span class="dk">TF.js</span> <span class="dv" id="d-tf">not loaded</span></div>
<div class="drow"><span class="dk">Face model</span> <span class="dv" id="d-face">not loaded</span></div>
<div class="drow"><span class="dk">Object model</span> <span class="dv" id="d-obj">not loaded</span></div>
<div class="drow"><span class="dk">Canvas size</span> <span class="dv" id="d-canvas">—</span></div>
<div class="drow"><span class="dk">Frames run</span> <span class="dv" id="d-frames">0</span></div>
<div class="drow"><span class="dk">Last faces</span> <span class="dv" id="d-faces">—</span></div>
<div class="drow"><span class="dk">Last objects</span> <span class="dv" id="d-objs">—</span></div>
<div class="drow"><span class="dk">Last error</span> <span class="dv err" id="d-err">none</span></div>
</div>
<div id="diag-log"></div>
</div>

<div id="items-panel">
<h2>🔍 Detected Items <span class="model-badge" id="model-badge">Chrome FaceDetector API</span></h2>
<div id="items-grid"></div>
</div>

<div id="notif-log">
<h2>📱 Phone Alerts</h2>
<div id="notif-list"></div>
</div>

<!-- CDN scripts — loaded on demand when user switches to a CDN mode -->

<script>
/* ════════════════════════════════════════
DOM REFS
════════════════════════════════════════ */
const video = document.getElementById('v');
const overlayC = document.getElementById('oc');
const ctx = overlayC.getContext('2d');
const statusEl = document.getElementById('status');
const countEl = document.getElementById('count');
const startBtn = document.getElementById('startBtn');
const stopBtn = document.getElementById('stopBtn');
const notifList = document.getElementById('notif-list');
const itemsGrid = document.getElementById('items-grid');
const modelBadge = document.getElementById('model-badge');

const dMode = document.getElementById('d-mode');
const dBuiltin = document.getElementById('d-builtin');
const dTf = document.getElementById('d-tf');
const dFace = document.getElementById('d-face');
const dObj = document.getElementById('d-obj');
const dCanvas = document.getElementById('d-canvas');
const dFrames = document.getElementById('d-frames');
const dFaces = document.getElementById('d-faces');
const dObjs = document.getElementById('d-objs');
const dErr = document.getElementById('d-err');
const diagLog = document.getElementById('diag-log');

function dset(el, text, cls) {
el.textContent = text;
el.className = 'dv' + (cls ? ' ' + cls : '');
}
function dlog(msg, cls = '') {
const d = document.createElement('div');
d.className = 'dlog ' + cls;
d.textContent = new Date().toLocaleTimeString() + ' — ' + msg;
diagLog.prepend(d);
}
function setStatus(msg, cls = '') {
statusEl.textContent = msg;
statusEl.className = cls;
}

/* ════════════════════════════════════════
STATE
════════════════════════════════════════ */
let stream = null, running = false, animId = null;
let frameCount = 0, lastNotifTime = 0;
let lastObjDetections = [];
let currentMode = 'builtin'; // 'builtin' | 'faceapi' | 'full'

// CDN model state
let tfLoaded = false, faceApiLoaded = false, cocoLoaded = false;
let faceReady = false, objReady = false, cocoModel = null;
let builtinDetector = null;

const snapCanvas = document.createElement('canvas');
const snapCtx = snapCanvas.getContext('2d');

const NTFY_TOPIC = 'myfacealert123';
const FACE_MODEL_URL = 'https://cdn.jsdelivr.net/npm/@vladmandic/face-api/model';

/* ════════════════════════════════════════
CHECK BUILT-IN API
════════════════════════════════════════ */
if ('FaceDetector' in window) {
builtinDetector = new FaceDetector({ fastMode: false, maxDetectedFaces: 10 });
dset(dBuiltin, 'Supported ✓', 'ok');
dlog('Chrome FaceDetector API available.', 'info');
} else {
dset(dBuiltin, 'Not supported', 'err');
dlog('Chrome FaceDetector API not available. Enable chrome://flags → Experimental Web Platform Features.', 'warn');
// Disable builtin button and auto-switch to faceapi
document.getElementById('btn-builtin').classList.add('disabled-mode');
document.getElementById('btn-builtin').title = 'Not supported in this browser';
}

/* ════════════════════════════════════════
MODE SWITCHER
════════════════════════════════════════ */
document.querySelectorAll('.mode-btn').forEach(btn => {
btn.addEventListener('click', () => {
if (btn.classList.contains('disabled-mode')) return;
if (running) {
setStatus('⚠ Stop the camera before switching modes.', 'warn');
return;
}
document.querySelectorAll('.mode-btn').forEach(b => b.classList.remove('active'));
btn.classList.add('active');
currentMode = btn.dataset.mode;
onModeChange();
});
});

function onModeChange() {
dset(dMode, currentMode === 'builtin' ? 'Chrome Built-in' :
currentMode === 'faceapi' ? 'Face-API.js (CDN)' :
'Face + Objects (CDN)', 'ok');

if (currentMode === 'builtin') {
modelBadge.textContent = 'Chrome FaceDetector API';
if (!builtinDetector) {
setStatus('❌ Built-in API not supported — switch to a CDN mode.', 'error');
startBtn.disabled = true;
} else {
setStatus('Built-in mode ready — click Start Camera.', 'ok');
startBtn.disabled = false;
}
itemsGrid.innerHTML = '';
} else if (currentMode === 'faceapi') {
modelBadge.textContent = 'face-api.js · TinyFaceDetector';
setStatus('Loading Face-API from CDN…', 'warn');
startBtn.disabled = true;
loadFaceApi().then(() => {
setStatus('Face-API ready — click Start Camera.', 'ok');
startBtn.disabled = false;
}).catch(e => {
setStatus('❌ CDN failed: ' + e.message, 'error');
});
itemsGrid.innerHTML = '';
} else if (currentMode === 'full') {
modelBadge.textContent = 'TF.js + COCO-SSD + face-api.js';
setStatus('Loading models from CDN…', 'warn');
startBtn.disabled = true;
loadFullModels().then(() => {
setStatus('All models ready — click Start Camera.', 'ok');
startBtn.disabled = false;
}).catch(e => {
setStatus('❌ CDN failed: ' + e.message, 'error');
});
}
}

// Set initial mode label
onModeChange();

/* ════════════════════════════════════════
DYNAMIC CDN LOADER
════════════════════════════════════════ */
function loadScript(src) {
return new Promise((resolve, reject) => {
if (document.querySelector(`script[src="${src}"]`)) { resolve(); return; }
const s = document.createElement('script');
s.src = src;
s.onload = resolve;
s.onerror = () => reject(new Error('Failed to load: ' + src));
document.head.appendChild(s);
});
}

async function loadFaceApi() {
if (faceApiLoaded && faceReady) return;
dset(dFace, 'loading CDN…', 'warn');
if (!tfLoaded) {
dset(dTf, 'loading CDN…', 'warn');
await loadScript('https://cdn.jsdelivr.net/npm/@tensorflow/tfjs@4.17.0/dist/tf.min.js');
tfLoaded = true;
dset(dTf, 'v' + tf.version.tfjs + ' ✓', 'ok');
dlog('TF.js loaded from CDN.', 'info');
}
if (!faceApiLoaded) {
await loadScript('https://cdn.jsdelivr.net/npm/face-api.js@0.22.2/dist/face-api.min.js');
faceApiLoaded = true;
dlog('face-api.js loaded from CDN.', 'info');
}
if (!faceReady) {
await faceapi.nets.tinyFaceDetector.loadFromUri(FACE_MODEL_URL);
faceReady = true;
dset(dFace, 'loaded ✓', 'ok');
dlog('Face model weights loaded.', 'info');
}
}

async function loadFullModels() {
await loadFaceApi();
if (!cocoLoaded) {
dset(dObj, 'loading CDN…', 'warn');
await loadScript('https://cdn.jsdelivr.net/npm/@tensorflow-models/coco-ssd@2.2.3/dist/coco-ssd.min.js');
cocoLoaded = true;
dlog('COCO-SSD script loaded from CDN.', 'info');
}
if (!objReady) {
cocoModel = await cocoSsd.load({ base: 'lite_mobilenet_v2' });
objReady = true;
dset(dObj, 'loaded ✓', 'ok');
dlog('COCO-SSD model loaded.', 'info');
}
}

/* ════════════════════════════════════════
NTFY ALERTS
════════════════════════════════════════ */
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
res.ok ? logNotif(`✅ Alert: ${count} face${count > 1 ? 's' : ''} detected`)
: logNotif(`❌ ntfy error ${res.status}`, false);
} catch { logNotif('❌ Could not reach ntfy.sh', false); }
}

/* ════════════════════════════════════════
CATEGORY MAP
════════════════════════════════════════ */
const CATEGORY_MAP = {
person: { cat: 'Person', color: '#00ffe0' },
bicycle: { cat: 'Vehicle', color: '#ffe600' },
car: { cat: 'Vehicle', color: '#ffe600' },
motorcycle: { cat: 'Vehicle', color: '#ffe600' },
airplane: { cat: 'Vehicle', color: '#ffe600' },
bus: { cat: 'Vehicle', color: '#ffe600' },
train: { cat: 'Vehicle', color: '#ffe600' },
truck: { cat: 'Vehicle', color: '#ffe600' },
boat: { cat: 'Vehicle', color: '#ffe600' },
'traffic light': { cat: 'Street', color: '#fdcb6e' },
'fire hydrant': { cat: 'Street', color: '#fdcb6e' },
'stop sign': { cat: 'Street', color: '#fdcb6e' },
bench: { cat: 'Street', color: '#fdcb6e' },
bird: { cat: 'Animal', color: '#55efc4' },
cat: { cat: 'Animal', color: '#55efc4' },
dog: { cat: 'Animal', color: '#55efc4' },
horse: { cat: 'Animal', color: '#55efc4' },
cow: { cat: 'Animal', color: '#55efc4' },
elephant: { cat: 'Animal', color: '#55efc4' },
bear: { cat: 'Animal', color: '#55efc4' },
backpack: { cat: 'Bag', color: '#e17055' },
umbrella: { cat: 'Accessory', color: '#e17055' },
handbag: { cat: 'Bag', color: '#e17055' },
tie: { cat: 'Clothing', color: '#e17055' },
suitcase: { cat: 'Bag', color: '#e17055' },
'sports ball': { cat: 'Sports', color: '#00cec9' },
skateboard: { cat: 'Sports', color: '#00cec9' },
bottle: { cat: 'Drink', color: '#fd79a8' },
'wine glass': { cat: 'Drink', color: '#fd79a8' },
cup: { cat: 'Drink', color: '#fd79a8' },
fork: { cat: 'Utensil', color: '#ffeaa7' },
knife: { cat: 'Utensil', color: '#ffeaa7' },
spoon: { cat: 'Utensil', color: '#ffeaa7' },
bowl: { cat: 'Utensil', color: '#ffeaa7' },
banana: { cat: 'Food', color: '#ff9f43' },
apple: { cat: 'Food', color: '#ff9f43' },
sandwich: { cat: 'Food', color: '#ff9f43' },
orange: { cat: 'Food', color: '#ff9f43' },
broccoli: { cat: 'Food', color: '#ff9f43' },
carrot: { cat: 'Food', color: '#ff9f43' },
'hot dog': { cat: 'Food', color: '#ff9f43' },
pizza: { cat: 'Food', color: '#ff9f43' },
donut: { cat: 'Food', color: '#ff9f43' },
cake: { cat: 'Food', color: '#ff9f43' },
chair: { cat: 'Furniture', color: '#a29bfe' },
couch: { cat: 'Furniture', color: '#a29bfe' },
'potted plant': { cat: 'Plant', color: '#6ab04c' },
bed: { cat: 'Furniture', color: '#a29bfe' },
'dining table': { cat: 'Furniture', color: '#a29bfe' },
tv: { cat: 'Electronics', color: '#b084ff' },
laptop: { cat: 'Electronics', color: '#b084ff' },
mouse: { cat: 'Electronics', color: '#b084ff' },
remote: { cat: 'Electronics', color: '#b084ff' },
keyboard: { cat: 'Electronics', color: '#b084ff' },
'cell phone': { cat: 'Electronics', color: '#b084ff' },
microwave: { cat: 'Appliance', color: '#636e72' },
oven: { cat: 'Appliance', color: '#636e72' },
toaster: { cat: 'Appliance', color: '#636e72' },
sink: { cat: 'Appliance', color: '#636e72' },
refrigerator: { cat: 'Appliance', color: '#636e72' },
book: { cat: 'Books', color: '#74b9ff' },
clock: { cat: 'Decor', color: '#badc58' },
vase: { cat: 'Decor', color: '#badc58' },
scissors: { cat: 'Tool', color: '#dfe6e9' },
'teddy bear': { cat: 'Toy', color: '#fab1a0' },
toothbrush: { cat: 'Personal', color: '#dfe6e9' },
};
function getCategory(label) {
return CATEGORY_MAP[label.toLowerCase()] || { cat: 'Object', color: '#aaaaaa' };
}

/* ════════════════════════════════════════
CAMERA
════════════════════════════════════════ */
startBtn.addEventListener('click', async () => {
try {
dlog('Requesting camera…', 'info');
stream = await navigator.mediaDevices.getUserMedia({
video: { width: 640, height: 480, facingMode: 'user' }, audio: false
});
video.srcObject = stream;
await video.play();

await new Promise(resolve => {
if (video.videoWidth > 0) { resolve(); return; }
video.addEventListener('loadedmetadata', resolve, { once: true });
});
await new Promise(r => setTimeout(r, 300));

const W = video.videoWidth || 640;
const H = video.videoHeight || 480;
overlayC.width = W; overlayC.height = H;
snapCanvas.width = W; snapCanvas.height = H;

dset(dCanvas, `${W}×${H}`, 'ok');
dlog(`Camera started: ${W}×${H}`, 'info');

running = true;
startBtn.disabled = true;
stopBtn.disabled = false;
document.querySelectorAll('.mode-btn').forEach(b => b.style.pointerEvents = 'none');
setStatus('Camera active — scanning…', 'ok');
loop();
} catch(e) {
const msg = e.name === 'NotAllowedError' ? 'Camera permission denied.' : e.message;
setStatus('❌ ' + msg, 'error');
dset(dErr, msg, '');
dlog('Camera error: ' + e.message, 'err');
}
});

stopBtn.addEventListener('click', () => {
running = false;
cancelAnimationFrame(animId);
if (stream) stream.getTracks().forEach(t => t.stop());
ctx.clearRect(0, 0, overlayC.width, overlayC.height);
countEl.innerHTML = '';
itemsGrid.innerHTML = '';
lastObjDetections = [];
startBtn.disabled = false;
stopBtn.disabled = true;
document.querySelectorAll('.mode-btn').forEach(b => b.style.pointerEvents = '');
setStatus('Stopped. Select a mode and click Start Camera.');
dlog('Stopped.', '');
});

/* ════════════════════════════════════════
DRAW
════════════════════════════════════════ */
function drawBox(x, y, w, h, color, label, conf) {
ctx.strokeStyle = color; ctx.lineWidth = 2;
ctx.shadowColor = color; ctx.shadowBlur = 10;
ctx.strokeRect(x, y, w, h); ctx.shadowBlur = 0;

// corner accents
const cs = 12; ctx.lineWidth = 3;
[[x,y,1,1],[x+w,y,-1,1],[x,y+h,1,-1],[x+w,y+h,-1,-1]].forEach(([cx,cy,dx,dy]) => {
ctx.beginPath();
ctx.moveTo(cx+dx*cs, cy); ctx.lineTo(cx, cy); ctx.lineTo(cx, cy+dy*cs);
ctx.stroke();
});

ctx.lineWidth = 1;
ctx.font = 'bold 11px monospace';
const text = conf ? `${label} ${conf}%` : label;
const tw = ctx.measureText(text).width + 8;
ctx.fillStyle = color + '33';
ctx.fillRect(x, y > 18 ? y - 18 : y, tw, 18);
ctx.fillStyle = color;
ctx.fillText(text, x + 4, y > 18 ? y - 5 : y + 13);
}

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
(conf ? `<span class="conf">${conf}%</span>` : '');
itemsGrid.appendChild(chip);
});
}

/* ════════════════════════════════════════
MAIN LOOP
════════════════════════════════════════ */
async function loop() {
if (!running) return;

if (video.readyState >= 2) {
frameCount++;
dFrames.textContent = frameCount;

const W = overlayC.width;
const H = overlayC.height;
ctx.clearRect(0, 0, W, H);
const allItems = [];

/* ── MODE: Chrome Built-in ── */
if (currentMode === 'builtin' && builtinDetector) {
try {
const faces = await builtinDetector.detect(video);
dset(dFaces, faces.length + ' detected', faces.length > 0 ? 'ok' : '');
faces.forEach(face => {
const { x, y, width, height } = face.boundingBox;
const mx = W - x - width;
drawBox(mx, y, width, height, '#00ffe0', 'FACE', null);
allItems.push({ label: 'Face', cat: 'Face', color: '#00ffe0', conf: null });

// landmarks
if (face.landmarks) {
face.landmarks.forEach(lm => {
ctx.beginPath();
ctx.arc(W - lm.location.x, lm.location.y, 3, 0, Math.PI * 2);
ctx.fillStyle = '#ff3cac';
ctx.shadowColor = '#ff3cac'; ctx.shadowBlur = 6;
ctx.fill(); ctx.shadowBlur = 0;
});
}
});
const n = faces.length;
if (n > 0) sendPhoneAlert(n);
countEl.innerHTML = n > 0 ? `${n}<span>${n === 1 ? 'face detected' : 'faces detected'}</span>` : '';
} catch(e) {
dset(dErr, e.message.slice(0, 80), '');
dlog('Built-in detect error: ' + e.message, 'err');
}

/* ── MODE: face-api.js only ── */
} else if (currentMode === 'faceapi' && faceReady) {
try {
snapCtx.drawImage(video, 0, 0, W, H);
const opts = new faceapi.TinyFaceDetectorOptions({ inputSize: 224, scoreThreshold: 0.35 });
const faces = await faceapi.detectAllFaces(snapCanvas, opts);
const resized = faceapi.resizeResults(faces, { width: W, height: H });

dset(dFaces, resized.length + ' detected', resized.length > 0 ? 'ok' : '');
resized.forEach(det => {
const { x, y, width, height } = det.box;
const mx = W - x - width;
const conf = Math.round(det.score * 100);
drawBox(mx, y, width, height, '#00ffe0', 'FACE', conf);
allItems.push({ label: 'Face', cat: 'Face', color: '#00ffe0', conf });
});
const n = resized.length;
if (n > 0) sendPhoneAlert(n);
countEl.innerHTML = n > 0 ? `${n}<span>${n === 1 ? 'face detected' : 'faces detected'}</span>` : '';
} catch(e) {
dset(dErr, e.message.slice(0, 80), '');
dlog('face-api error: ' + e.message, 'err');
}

/* ── MODE: full (face + objects) ── */
} else if (currentMode === 'full' && faceReady) {
try {
snapCtx.drawImage(video, 0, 0, W, H);

// faces
const opts = new faceapi.TinyFaceDetectorOptions({ inputSize: 224, scoreThreshold: 0.35 });
const faces = await faceapi.detectAllFaces(snapCanvas, opts);
const resized = faceapi.resizeResults(faces, { width: W, height: H });

dset(dFaces, resized.length + ' detected', resized.length > 0 ? 'ok' : '');
resized.forEach(det => {
const { x, y, width, height } = det.box;
const mx = W - x - width;
const conf = Math.round(det.score * 100);
drawBox(mx, y, width, height, '#00ffe0', 'FACE', conf);
allItems.push({ label: 'Face', cat: 'Face', color: '#00ffe0', conf });
});
const n = resized.length;
if (n > 0) sendPhoneAlert(n);
countEl.innerHTML = n > 0 ? `${n}<span>${n === 1 ? 'face detected' : 'faces detected'}</span>` : '';

// objects every 3rd frame
if (frameCount % 3 === 0 && objReady && cocoModel) {
const preds = await cocoModel.detect(snapCanvas, 10, 0.35);
const fresh = [];
preds.forEach(pred => {
const label = pred.class.toLowerCase();
if (label === 'person') return;
const [bx, by, bw, bh] = pred.bbox;
const mx = W - bx - bw;
const conf = Math.round(pred.score * 100);
const { cat, color } = getCategory(label);
drawBox(mx, by, bw, bh, color, label, conf);
fresh.push({ label, conf, color, cat });
});
lastObjDetections = fresh;
dset(dObjs, fresh.length + ' detected', fresh.length > 0 ? 'ok' : '');
}
lastObjDetections.forEach(item => allItems.push(item));
} catch(e) {
dset(dErr, e.message.slice(0, 80), '');
dlog('Full mode error: ' + e.message, 'err');
}
}

updateItemsPanel(allItems);
}

animId = requestAnimationFrame(loop);
}
</script>

</body>
</html>
