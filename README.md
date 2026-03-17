<!DOCTYPE html>
<html lang="en">
<head>
<meta charset="UTF-8"/>
<title>Face Scan</title>
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
.wrapper::after { top:8px; right:8px; border-top:2px solid var(--accent); border-right:2px solid var(--accent); }
.cbr { bottom:8px; right:8px; border-bottom:2px solid var(--accent); border-right:2px solid var(--accent); }
.cbl { bottom:8px; left:8px; border-bottom:2px solid var(--accent); border-left:2px solid var(--accent); }
canvas { display: block; max-width: min(640px, 100%); }
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
#status { margin-top:1rem; font-size:0.72rem; letter-spacing:0.1em; color:var(--dim); text-align:center; }
#status.ok { color: var(--accent); }
#status.error { color: var(--accent2); }
#notif-log {
margin-top: 1.5rem; width: min(480px, 100%);
border: 1px solid var(--border); border-radius: 4px; overflow: hidden;
}
#notif-log h2 {
font-size: 0.7rem; letter-spacing: 0.15em; text-transform: uppercase;
color: var(--dim); padding: 0.6rem 1rem;
border-bottom: 1px solid var(--border); background: rgba(255,255,255,0.02);
}
#notif-list { max-height: 160px; overflow-y: auto; padding: 0.4rem 0; }
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

<h1>FACE SCAN</h1>
<p class="sub">Vending Machine Prototype</p>

<div class="wrapper">
<canvas id="c" width="640" height="480"></canvas>
<div class="scan"></div>
<div class="cbr"></div><div class="cbl"></div>
</div>

<div class="controls">
<button id="startBtn">Start Camera</button>
<button id="stopBtn" disabled>Stop</button>
</div>

<div id="count"></div>
<div id="status" class="ok">Ready</div>

<div id="notif-log">
<h2>📱 Phone Alerts</h2>
<div id="notif-list"></div>
</div>

<div id="tip"></div>

<script>
const canvas = document.getElementById('c');
const ctx = canvas.getContext('2d');
const statusEl = document.getElementById('status');
const countEl = document.getElementById('count');
const tipEl = document.getElementById('tip');
const startBtn = document.getElementById('startBtn');
const stopBtn = document.getElementById('stopBtn');
const notifList = document.getElementById('notif-list');

let stream = null, video = null, detector = null;
let running = false, animId = null;
let lastNotifTime = 0;

const NTFY_TOPIC = 'myfacealert123';

function setStatus(msg, cls='') {
statusEl.textContent = msg;
statusEl.className = cls;
}

function logNotif(msg, success = true) {
const item = document.createElement('div');
item.className = 'notif-item' + (success ? '' : ' fail');
const now = new Date().toLocaleTimeString();
item.innerHTML = `<span class="msg">${msg}</span><span class="time">${now}</span>`;
notifList.prepend(item);
}

async function sendPhoneAlert(count) {
const now = Date.now();
if (now - lastNotifTime < 10000) return;
lastNotifTime = now;

const body = `📸 Customer detected! ${count} person${count > 1 ? 's' : ''} at the vending machine.`;

try {
const res = await fetch(`https://ntfy.sh/${NTFY_TOPIC}`, {
method: 'POST',
body: body,
headers: {
'Title': 'Vending Machine Alert',
'Priority': 'default',
'Tags': 'vending,face,camera'
}
});
if (res.ok) {
logNotif(`✅ Alert sent: ${count} face${count > 1 ? 's' : ''} detected`, true);
} else {
logNotif(`❌ ntfy error ${res.status} — check topic name`, false);
}
} catch (e) {
logNotif(`❌ Could not reach ntfy.sh — may be blocked`, false);
}
}

const hasNativeDetector = ('FaceDetector' in window);
const hasMediaDevices = !!(navigator.mediaDevices && navigator.mediaDevices.getUserMedia);

if (!hasMediaDevices) {
setStatus('❌ Camera API not supported in this browser.', 'error');
startBtn.disabled = true;
} else if (!hasNativeDetector) {
tipEl.textContent =
"ℹ️ Chrome's built-in face detector isn't enabled — using skin-tone detection as fallback. " +
"For full face detection, ask your admin to enable Experimental Web Platform Features in Chrome.";
}

startBtn.addEventListener('click', async () => {
try {
stream = await navigator.mediaDevices.getUserMedia({
video: { width:640, height:480, facingMode:'user' },
audio: false
});

video = document.createElement('video');
video.srcObject = stream;
video.muted = true;
video.autoplay = true;
await new Promise(r => { video.onloadedmetadata = r; });
await video.play();

canvas.width = video.videoWidth || 640;
canvas.height = video.videoHeight || 480;

if (hasNativeDetector) {
detector = new FaceDetector({ fastMode: true, maxDetectedFaces: 10 });
}

running = true;
startBtn.disabled = true;
stopBtn.disabled = false;
setStatus('Camera active — scanning…', 'ok');
loop();

} catch (e) {
if (e.name === 'NotAllowedError') {
setStatus('❌ Camera permission denied. Click the camera icon in the address bar to allow it.', 'error');
} else {
setStatus('❌ ' + e.message, 'error');
}
}
});

stopBtn.addEventListener('click', () => {
running = false;
cancelAnimationFrame(animId);
if (stream) stream.getTracks().forEach(t => t.stop());
ctx.clearRect(0, 0, canvas.width, canvas.height);
countEl.innerHTML = '';
startBtn.disabled = false;
stopBtn.disabled = true;
setStatus('Stopped.');
});

function skinToneDetect() {
const w = canvas.width, h = canvas.height;
const frame = ctx.getImageData(0, 0, w, h);
const d = frame.data;

const skinMap = new Uint8Array(w * h);
for (let y = 0; y < h; y += 4) {
for (let x = 0; x < w; x += 4) {
const i = (y * w + x) * 4;
const r = d[i], g = d[i+1], b = d[i+2];
if (
r > 95 && g > 40 && b > 20 &&
r > g && r > b &&
(r - Math.min(g, b)) > 15 &&
Math.abs(r - g) > 15
) {
skinMap[y * w + x] = 1;
}
}
}

let minX = w, maxX = 0, minY = h, maxY = 0, count = 0;
for (let y = 0; y < h; y += 4) {
for (let x = 0; x < w; x += 4) {
if (skinMap[y * w + x]) {
if (x < minX) minX = x;
if (x > maxX) maxX = x;
if (y < minY) minY = y;
if (y > maxY) maxY = y;
count++;
}
}
}

if (count > 200) {
const bw = maxX - minX, bh = maxY - minY;
if (bw > 40 && bh > 40 && bh / bw > 0.5) {
ctx.strokeStyle = '#00ffe0';
ctx.lineWidth = 2;
ctx.shadowColor = '#00ffe0';
ctx.shadowBlur = 10;
ctx.strokeRect(minX, minY, bw, bh);
ctx.shadowBlur = 0;
ctx.font = 'bold 11px monospace';
ctx.fillStyle = '#00ffe0';
ctx.fillText('FACE', minX + 4, minY > 16 ? minY - 6 : minY + 14);
return 1;
}
}
return 0;
}

async function loop() {
if (!running) return;

if (video.readyState === 4) {
ctx.save();
ctx.translate(canvas.width, 0);
ctx.scale(-1, 1);
ctx.drawImage(video, 0, 0, canvas.width, canvas.height);
ctx.restore();

let n = 0;

if (hasNativeDetector && detector) {
try {
const faces = await detector.detect(video);
n = faces.length;

faces.forEach(face => {
const { left, top, width, height } = face.boundingBox;
const mx = canvas.width - left - width;

ctx.strokeStyle = '#00ffe0';
ctx.lineWidth = 2;
ctx.shadowColor = '#00ffe0';
ctx.shadowBlur = 12;
ctx.strokeRect(mx, top, width, height);
ctx.shadowBlur = 0;

ctx.font = 'bold 11px monospace';
ctx.fillStyle = '#00ffe0';
ctx.fillText('FACE', mx + 4, top > 16 ? top - 6 : top + 14);

if (face.landmarks) {
const lmColors = { eye: '#ff3cac', nose: '#ffe600', mouth: '#00ffe0' };
face.landmarks.forEach(lm => {
const lx = canvas.width - lm.location.x;
const ly = lm.location.y;
const col = lmColors[lm.type] || '#fff';
ctx.beginPath();
ctx.arc(lx, ly, 4, 0, Math.PI * 2);
ctx.fillStyle = col;
ctx.shadowColor = col;
ctx.shadowBlur = 6;
ctx.fill();
ctx.shadowBlur = 0;
});
}
});
} catch(e) { /* skip frame */ }

} else {
n = skinToneDetect();
}

if (n > 0) sendPhoneAlert(n);

countEl.innerHTML = n > 0
? `${n}<span>${n === 1 ? 'face detected' : 'faces detected'}</span>`
: '';
}

animId = requestAnimationFrame(loop);
}
</script>
</body>
</html>
