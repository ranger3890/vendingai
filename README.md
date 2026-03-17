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
.wrapper::after { top:8px; right:8px; border-top:2px solid var(--accent); border-right:2px solid var(--accent); }
.cbr { bottom:8px; right:8px; border-bottom:2px solid var(--accent); border-right:2px solid var(--accent); }
.cbl { bottom:8px; left:8px; border-bottom:2px solid var(--accent); border-left:2px solid var(--accent); }
#videoWrap { position: relative; display: block; }
#v { display: block; max-width: min(640px, 100%); transform: scaleX(-1); }
#oc { position: absolute; top: 0; left: 0; width: 100%; height: 100%; pointer-events: none; }
.scan { position:absolute; left:0; right:0; height:2px;
background: linear-gradient(90deg, transparent, var(--accent), transparent);
opacity: 0; animation: scan 3s linear infinite; pointer-events:none; z-index:5; transition: opacity 0.5s; }
.scan.active { opacity: 0.5; }
@keyframes scan { 0%{top:0%} 100%{top:100%} }

/* API key input */
#api-bar {
margin-bottom: 1rem; width: min(640px, 100%);
border: 1px solid var(--border); border-radius: 4px; padding: 0.8rem 1rem;
background: rgba(255,255,255,0.02);
}
#api-bar label { font-size: 0.65rem; letter-spacing: 0.14em; text-transform: uppercase; color: var(--dim); display: block; margin-bottom: 0.4rem; }
#api-bar .row { display: flex; gap: 0.5rem; align-items: center; }
#api-key {
flex: 1; background: #0d0d18; border: 1px solid var(--border); border-radius: 2px;
color: #e8e8f0; font-family: inherit; font-size: 0.72rem; padding: 0.5rem 0.8rem;
outline: none; transition: border-color 0.2s;
}
#api-key:focus { border-color: var(--accent); }
#api-key::placeholder { color: #33334a; }
.api-hint { font-size: 0.62rem; color: var(--dim); margin-top: 0.4rem; }
.api-hint a { color: var(--accent); text-decoration: none; }

.controls { margin-top:1.2rem; display:flex; gap:1rem; flex-wrap:wrap; justify-content:center; }
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
#status { margin-top:0.5rem; font-size:0.72rem; letter-spacing:0.1em; color:var(--dim); text-align:center; }
#status.ok { color: var(--accent); }
#status.error { color: var(--accent2); }
#status.scanning { color: #ffe600; }

/* interval control */
#interval-bar {
margin-top: 0.6rem; display: flex; align-items: center; gap: 0.8rem;
font-size: 0.68rem; color: var(--dim);
}
#interval-bar label { letter-spacing: 0.1em; text-transform: uppercase; font-size: 0.62rem; }
#intervalRange { accent-color: var(--accent); width: 120px; }
#intervalVal { color: var(--accent); min-width: 3ch; }

/* diag */
#diag {
margin-top: 0.8rem; width: min(640px, 100%);
border: 1px solid #2a2a3a; border-radius: 4px; overflow: hidden;
font-size: 0.65rem; letter-spacing: 0.06em;
}
#diag-title { background: rgba(255,255,255,0.03); padding: 0.4rem 0.8rem;
color: var(--dim); border-bottom: 1px solid #2a2a3a;
font-size: 0.62rem; text-transform: uppercase; letter-spacing: 0.14em; }
#diag-body { padding: 0.5rem 0.8rem; display: flex; flex-direction: column; gap: 0.3rem; }
.drow { display: flex; justify-content: space-between; gap: 1rem; }
.drow .dk { color: #555570; }
.drow .dv { color: #e8e8f0; }
.drow .dv.ok { color: #00ffe0; }
.drow .dv.err { color: #ff3cac; }
.drow .dv.warn{ color: #ffe600; }
#diag-log { margin-top: 0.3rem; border-top: 1px solid #1a1a2a;
padding: 0.4rem 0.8rem; max-height: 80px; overflow-y: auto;
display: flex; flex-direction: column; gap: 0.15rem; }
.dlog { font-size: 0.6rem; color: #555570; }
.dlog.info { color: #00ffe0; }
.dlog.warn { color: #ffe600; }
.dlog.err { color: #ff3cac; }

#items-panel {
margin-top: 1rem; width: min(640px, 100%);
border: 1px solid var(--border); border-radius: 4px; overflow: hidden;
}
#items-panel h2 {
font-size: 0.7rem; letter-spacing: 0.15em; text-transform: uppercase;
color: var(--dim); padding: 0.6rem 1rem;
border-bottom: 1px solid var(--border); background: rgba(255,255,255,0.02);
display: flex; justify-content: space-between; align-items: center;
}
#items-panel h2 .model-badge { font-size: 0.58rem; color: #444466; }
#items-grid { display: flex; flex-wrap: wrap; gap: 0.5rem; padding: 0.8rem 1rem; min-height: 52px; }
#items-grid:empty::after { content: 'No items detected yet.'; color: var(--dim); font-size: 0.68rem; }
.item-chip {
display: flex; align-items: center; gap: 0.4rem;
padding: 0.25rem 0.7rem; border-radius: 2px; font-size: 0.68rem;
border: 1px solid; white-space: nowrap; animation: chipIn 0.25s ease;
}
@keyframes chipIn { from { opacity:0; transform:scale(0.92); } to { opacity:1; transform:none; } }
.item-chip .dot { width: 7px; height: 7px; border-radius: 50%; flex-shrink: 0; }
.item-chip .cat { opacity: 0.55; font-size: 0.58rem; margin-left: 0.2rem; }

#notif-log {
margin-top: 1rem; width: min(480px, 100%);
border: 1px solid var(--border); border-radius: 4px; overflow: hidden;
}
#notif-log h2 {
font-size: 0.7rem; letter-spacing: 0.15em; text-transform: uppercase;
color: var(--dim); padding: 0.6rem 1rem;
border-bottom: 1px solid var(--border); background: rgba(255,255,255,0.02);
}
#notif-list { max-height: 120px; overflow-y: auto; padding: 0.4rem 0; }
#notif-list:empty::after {
content: 'No alerts sent yet.'; display: block; text-align: center;
color: var(--dim); font-size: 0.68rem; padding: 0.8rem;
}
.notif-item {
display: flex; justify-content: space-between; align-items: center;
padding: 0.35rem 1rem; font-size: 0.68rem;
border-bottom: 1px solid rgba(255,255,255,0.04); animation: fadeIn 0.3s ease;
}
@keyframes fadeIn { from { opacity:0; transform:translateY(-4px); } to { opacity:1; transform:none; } }
.notif-item .msg { color: var(--accent); }
.notif-item .time { color: var(--dim); font-size: 0.62rem; }
.notif-item.fail .msg { color: var(--accent2); }

/* scanning pulse on overlay */
#oc.pulsing { animation: borderPulse 1s ease-in-out; }
@keyframes borderPulse {
0% { box-shadow: 0 0 0 0 rgba(0,255,224,0); }
50% { box-shadow: 0 0 20px 4px rgba(0,255,224,0.35); }
100% { box-shadow: 0 0 0 0 rgba(0,255,224,0); }
}
</style>
</head>
<body>

<h1>FACE + OBJECT SCAN</h1>
<p class="sub">Vending Machine Prototype · Powered by Claude Vision</p>

<!-- API Key input -->
<div id="api-bar">
<label>⚡ Anthropic API Key (required — runs in your browser only)</label>
<div class="row">
<input id="api-key" type="password" placeholder="sk-ant-api03-…" autocomplete="off" spellcheck="false"/>
</div>
<p class="api-hint">No CDN ML libraries needed. Frames are sent to the <strong>Claude vision API</strong> for detection. Your key is never stored. Get one at <a href="https://console.anthropic.com" target="_blank">console.anthropic.com</a>.</p>
</div>

<div class="wrapper">
<div id="videoWrap">
<video id="v" width="640" height="480" muted playsinline autoplay></video>
<canvas id="oc"></canvas>
</div>
<div class="scan" id="scanBar"></div>
<div class="cbr"></div><div class="cbl"></div>
</div>

<div class="controls">
<button id="startBtn">Start Camera</button>
<button id="stopBtn" disabled>Stop</button>
</div>

<div id="interval-bar">
<label>Scan Interval</label>
<input type="range" id="intervalRange" min="1" max="10" value="3" step="0.5"/>
<span id="intervalVal">3s</span>
</div>

<div id="count"></div>
<div id="status" class="ok">Enter your API key, then click Start Camera.</div>

<div id="diag">
<div id="diag-title">⚙ Diagnostics</div>
<div id="diag-body">
<div class="drow"><span class="dk">Claude API</span> <span class="dv" id="d-api">—</span></div>
<div class="drow"><span class="dk">Canvas size</span> <span class="dv" id="d-canvas">—</span></div>
<div class="drow"><span class="dk">Frames captured</span><span class="dv" id="d-frames">0</span></div>
<div class="drow"><span class="dk">API calls made</span> <span class="dv" id="d-calls">0</span></div>
<div class="drow"><span class="dk">Last faces</span> <span class="dv" id="d-faces">—</span></div>
<div class="drow"><span class="dk">Last objects</span> <span class="dv" id="d-objs">—</span></div>
<div class="drow"><span class="dk">Last latency</span> <span class="dv" id="d-lat">—</span></div>
<div class="drow"><span class="dk">Last error</span> <span class="dv err" id="d-err">none</span></div>
</div>
<div id="diag-log"></div>
</div>

<div id="items-panel">
<h2>🔍 Detected Items <span class="model-badge">Claude Vision · claude-sonnet-4-20250514</span></h2>
<div id="items-grid"></div>
</div>

<div id="notif-log">
<h2>📱 Phone Alerts</h2>
<div id="notif-list"></div>
</div>

<script>
/* ── DOM refs ── */
const video = document.getElementById('v');
const overlayC = document.getElementById('oc');
const ctx = overlayC.getContext('2d');
const statusEl = document.getElementById('status');
const countEl = document.getElementById('count');
const startBtn = document.getElementById('startBtn');
const stopBtn = document.getElementById('stopBtn');
const notifList = document.getElementById('notif-list');
const itemsGrid = document.getElementById('items-grid');
const scanBar = document.getElementById('scanBar');
const apiKeyInput = document.getElementById('api-key');
const intervalRange = document.getElementById('intervalRange');
const intervalVal = document.getElementById('intervalVal');

/* ── diag refs ── */
const dApi = document.getElementById('d-api');
const dCanvas = document.getElementById('d-canvas');
const dFrames = document.getElementById('d-frames');
const dCalls = document.getElementById('d-calls');
const dFaces = document.getElementById('d-faces');
const dObjs = document.getElementById('d-objs');
const dLat = document.getElementById('d-lat');
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

/* ── state ── */
let stream = null, running = false;
let frameCount = 0, callCount = 0;
let lastNotifTime = 0;
let scanTimer = null;

const snapCanvas = document.createElement('canvas');
const snapCtx = snapCanvas.getContext('2d');

const NTFY_TOPIC = 'myfacealert123';

/* ── interval slider ── */
intervalRange.addEventListener('input', () => {
intervalVal.textContent = intervalRange.value + 's';
});

function setStatus(msg, cls = '') {
statusEl.textContent = msg;
statusEl.className = cls;
}

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

/* ── colour palette for object categories ── */
const CAT_COLORS = {
'Person': '#00ffe0',
'Face': '#00ffe0',
'Vehicle': '#ffe600',
'Animal': '#55efc4',
'Food': '#ff9f43',
'Electronics': '#b084ff',
'Furniture': '#a29bfe',
'Bag': '#e17055',
'Clothing': '#e17055',
'Drink': '#fd79a8',
'Utensil': '#ffeaa7',
'Appliance': '#636e72',
'Sports': '#00cec9',
'Plant': '#6ab04c',
'Default': '#aaaaaa',
};
function catColor(cat) { return CAT_COLORS[cat] || CAT_COLORS['Default']; }

/* ── draw bounding box ── */
function drawBox(x, y, w, h, color, label, conf) {
ctx.strokeStyle = color; ctx.lineWidth = 2;
ctx.shadowColor = color; ctx.shadowBlur = 10;
ctx.strokeRect(x, y, w, h); ctx.shadowBlur = 0;
ctx.font = 'bold 11px monospace';
const text = conf ? `${label} ${conf}%` : label;
const tw = ctx.measureText(text).width + 8;
ctx.fillStyle = color + '33';
ctx.fillRect(x, y > 18 ? y - 18 : y, tw, 18);
ctx.fillStyle = color;
ctx.fillText(text, x + 4, y > 18 ? y - 5 : y + 13);
}

/* ── items panel ── */
function updateItemsPanel(items) {
itemsGrid.innerHTML = '';
items.forEach(({ label, cat, color, conf }) => {
const chip = document.createElement('div');
chip.className = 'item-chip';
chip.style.cssText = `border-color:${color}88;color:${color};background:${color}11`;
chip.innerHTML =
`<span class="dot" style="background:${color}"></span>` +
`<strong>${label}</strong><span class="cat">[${cat}]</span>` +
(conf ? `<span class="conf" style="opacity:.45;font-size:.58rem">${conf}%</span>` : '');
itemsGrid.appendChild(chip);
});
}

/* ════════════════════════════════════════
CLAUDE VISION API — analyse one frame
════════════════════════════════════════ */
async function analyseFrame(base64jpeg, apiKey) {
const prompt = `You are a computer vision system for a vending machine. Analyse this camera frame and return ONLY a JSON object, no markdown, no prose.

Schema:
{
"faces": <integer — number of human faces visible>,
"objects": [
{ "label": "<object name>", "category": "<category>", "confidence": <0-100 integer>,
"box": { "x": <0.0-1.0>, "y": <0.0-1.0>, "w": <0.0-1.0>, "h": <0.0-1.0> } }
]
}

Rules:
- box values are fractions of image width/height (0.0–1.0)
- category must be one of: Person, Face, Vehicle, Animal, Food, Electronics, Furniture, Bag, Clothing, Drink, Utensil, Appliance, Sports, Plant, Default
- include each visible face as a separate entry in objects with category "Face"
- do NOT include "person" as a whole-body object unless the full body is clearly visible
- list every clearly visible object, max 15 entries
- confidence is your certainty 0-100
- return ONLY the JSON, nothing else`;

const t0 = Date.now();
const resp = await fetch('https://api.anthropic.com/v1/messages', {
method: 'POST',
headers: {
'Content-Type': 'application/json',
'x-api-key': apiKey,
'anthropic-version': '2023-06-01',
'anthropic-dangerous-direct-browser-access': 'true',
},
body: JSON.stringify({
model: 'claude-sonnet-4-20250514',
max_tokens: 1000,
messages: [{
role: 'user',
content: [
{ type: 'image', source: { type: 'base64', media_type: 'image/jpeg', data: base64jpeg } },
{ type: 'text', text: prompt }
]
}]
})
});

const latency = Date.now() - t0;
dset(dLat, latency + 'ms', latency < 3000 ? 'ok' : 'warn');

if (!resp.ok) {
const err = await resp.text();
throw new Error(`API ${resp.status}: ${err.slice(0, 120)}`);
}

const data = await resp.json();
const raw = data.content.map(b => b.text || '').join('');
// strip markdown fences if model ignores the instruction
const clean = raw.replace(/```json|```/gi, '').trim();
return JSON.parse(clean);
}

/* ════════════════════════════════════════
CAPTURE + PROCESS ONE FRAME
════════════════════════════════════════ */
async function captureAndScan() {
if (!running) return;
const apiKey = apiKeyInput.value.trim();
if (!apiKey) { setStatus('❌ API key required', 'error'); return; }

if (video.readyState < 2 || snapCanvas.width === 0) return;

frameCount++;
dFrames.textContent = frameCount;

// Draw mirrored frame to snap canvas (match the CSS mirror transform)
const W = snapCanvas.width, H = snapCanvas.height;
snapCtx.save();
snapCtx.translate(W, 0); snapCtx.scale(-1, 1);
snapCtx.drawImage(video, 0, 0, W, H);
snapCtx.restore();

// Get JPEG base64
const base64 = snapCanvas.toDataURL('image/jpeg', 0.7).split(',')[1];

// Pulse the overlay
overlayC.classList.remove('pulsing');
void overlayC.offsetWidth;
overlayC.classList.add('pulsing');
scanBar.classList.add('active');
setStatus('🔍 Scanning frame…', 'scanning');

callCount++;
dCalls.textContent = callCount;
dset(dApi, 'Calling Claude…', 'warn');

try {
const result = await analyseFrame(base64, apiKey);
dset(dApi, 'OK ✓', 'ok');
dlog(`Frame ${frameCount}: ${result.faces} faces, ${result.objects.length} objects`, 'info');

ctx.clearRect(0, 0, W, H);
const allItems = [];

// Draw face boxes + collect
const faceObjs = result.objects.filter(o => o.category === 'Face');
faceObjs.forEach(o => {
const bx = o.box.x * W, by = o.box.y * H, bw = o.box.w * W, bh = o.box.h * H;
drawBox(bx, by, bw, bh, '#00ffe0', 'FACE', o.confidence);
allItems.push({ label: 'Face', cat: 'Face', color: '#00ffe0', conf: o.confidence });
});

// Draw other objects
result.objects.filter(o => o.category !== 'Face').forEach(o => {
const color = catColor(o.category);
const bx = o.box.x * W, by = o.box.y * H, bw = o.box.w * W, bh = o.box.h * H;
drawBox(bx, by, bw, bh, color, o.label, o.confidence);
allItems.push({ label: o.label, cat: o.category, color, conf: o.confidence });
});

const n = result.faces;
dset(dFaces, n + ' detected', n > 0 ? 'ok' : '');
dset(dObjs, (result.objects.length - faceObjs.length) + ' detected',
result.objects.length - faceObjs.length > 0 ? 'ok' : '');

if (n > 0) sendPhoneAlert(n);
countEl.innerHTML = n > 0
? `${n}<span>${n === 1 ? 'face detected' : 'faces detected'}</span>`
: '';

updateItemsPanel(allItems);
setStatus('Camera active — scanning…', 'ok');
} catch(e) {
dset(dApi, 'ERROR', 'err');
dset(dErr, e.message.slice(0, 80), '');
dlog('API error: ' + e.message, 'err');
setStatus('❌ ' + e.message.slice(0, 60), 'error');
}

scanBar.classList.remove('active');

// Schedule next scan
if (running) {
const interval = parseFloat(intervalRange.value) * 1000;
scanTimer = setTimeout(captureAndScan, interval);
}
}

/* ════════════════════════════════════════
CAMERA CONTROLS
════════════════════════════════════════ */
startBtn.addEventListener('click', async () => {
const apiKey = apiKeyInput.value.trim();
if (!apiKey) {
setStatus('❌ Please enter your Anthropic API key first.', 'error');
apiKeyInput.focus();
return;
}

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
apiKeyInput.disabled = true;
setStatus('Camera active — scanning…', 'ok');

captureAndScan(); // kick off first scan
} catch(e) {
const msg = e.name === 'NotAllowedError' ? 'Camera permission denied.' : e.message;
setStatus('❌ ' + msg, 'error');
dset(dErr, msg, '');
dlog('Camera error: ' + e.message, 'err');
}
});

stopBtn.addEventListener('click', () => {
running = false;
clearTimeout(scanTimer);
if (stream) stream.getTracks().forEach(t => t.stop());
ctx.clearRect(0, 0, overlayC.width, overlayC.height);
countEl.innerHTML = '';
itemsGrid.innerHTML = '';
scanBar.classList.remove('active');
startBtn.disabled = false;
stopBtn.disabled = true;
apiKeyInput.disabled = false;
setStatus('Stopped.');
dlog('Stopped.', '');
});
</script>
</body>
</html>
