[face-scan.html](https://github.com/user-attachments/files/26041979/face-scan.html)
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
.wrapper::after  { top:8px; right:8px; border-top:2px solid var(--accent); border-right:2px solid var(--accent); }
.cbr { bottom:8px; right:8px; border-bottom:2px solid var(--accent); border-right:2px solid var(--accent); }
.cbl { bottom:8px; left:8px; border-bottom:2px solid var(--accent); border-left:2px solid var(--accent); }

/* stacked video + overlay canvas */
#videoWrap { position: relative; display: block; }
#v  { display: block; max-width: min(640px, 100%); transform: scaleX(-1); }
#oc {
  position: absolute; top: 0; left: 0;
  width: 100%; height: 100%; pointer-events: none;
}

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
<div id="status" class="ok">Loading face detection models…</div>

<div id="notif-log">
  <h2>📱 Phone Alerts</h2>
  <div id="notif-list"></div>
</div>

<div id="tip"></div>

<!-- face-api.js: pure JS neural-net face detection, no browser flags needed -->
<script src="https://cdn.jsdelivr.net/npm/face-api.js@0.22.2/dist/face-api.min.js"></script>
<script>
const video      = document.getElementById('v');
const overlayC   = document.getElementById('oc');
const overlayCtx = overlayC.getContext('2d');
const statusEl   = document.getElementById('status');
const countEl    = document.getElementById('count');
const tipEl      = document.getElementById('tip');
const startBtn   = document.getElementById('startBtn');
const stopBtn    = document.getElementById('stopBtn');
const notifList  = document.getElementById('notif-list');

let stream = null, running = false, animId = null, modelsReady = false;
let lastNotifTime = 0;

const NTFY_TOPIC = 'myfacealert123';
// CDN that hosts the face-api model weights
const MODEL_URL  = 'https://cdn.jsdelivr.net/npm/@vladmandic/face-api/model';

/* ── status helper ─────────────────────────────────────────── */
function setStatus(msg, cls = '') {
  statusEl.textContent = msg;
  statusEl.className   = cls;
}

/* ── notification log ──────────────────────────────────────── */
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
      method: 'POST', body,
      headers: { 'Title': 'Vending Machine Alert', 'Priority': 'default', 'Tags': 'vending,face,camera' }
    });
    if (res.ok) logNotif(`✅ Alert sent: ${count} face${count > 1 ? 's' : ''} detected`, true);
    else        logNotif(`❌ ntfy error ${res.status} — check topic name`, false);
  } catch {
    logNotif('❌ Could not reach ntfy.sh — may be blocked', false);
  }
}

/* ── load neural-net models ────────────────────────────────── */
async function loadModels() {
  try {
    // Tiny face detector — fast, lightweight, works well on Chromebook CPUs
    await faceapi.nets.tinyFaceDetector.loadFromUri(MODEL_URL);
    modelsReady = true;
    setStatus('Models ready — click Start Camera', 'ok');
    startBtn.disabled = false;
  } catch (e) {
    setStatus('❌ Failed to load face-detection models. Check your internet connection.', 'error');
    tipEl.textContent = 'The face-api.js model weights could not be downloaded. Make sure you are online and try refreshing.';
    console.error(e);
  }
}

/* ── camera start / stop ───────────────────────────────────── */
startBtn.disabled = true; // disabled until models load
startBtn.addEventListener('click', async () => {
  if (!modelsReady) return;
  try {
    stream = await navigator.mediaDevices.getUserMedia({
      video: { width: 640, height: 480, facingMode: 'user' }, audio: false
    });
    video.srcObject = stream;
    await video.play();

    // match overlay canvas to actual video dimensions
    video.addEventListener('loadedmetadata', () => {
      overlayC.width  = video.videoWidth;
      overlayC.height = video.videoHeight;
    }, { once: true });

    running = true;
    startBtn.disabled = true;
    stopBtn.disabled  = false;
    setStatus('Camera active — scanning…', 'ok');
    loop();
  } catch (e) {
    if (e.name === 'NotAllowedError')
      setStatus('❌ Camera permission denied. Click the camera icon in the address bar.', 'error');
    else
      setStatus('❌ ' + e.message, 'error');
  }
});

stopBtn.addEventListener('click', () => {
  running = false;
  cancelAnimationFrame(animId);
  if (stream) stream.getTracks().forEach(t => t.stop());
  overlayCtx.clearRect(0, 0, overlayC.width, overlayC.height);
  countEl.innerHTML = '';
  startBtn.disabled = false;
  stopBtn.disabled  = true;
  setStatus('Stopped.');
});

/* ── detection + draw loop ─────────────────────────────────── */
async function loop() {
  if (!running) return;

  if (video.readyState >= 2) {
    // Tiny face detector options — lower scoreThreshold = more sensitive
    const opts = new faceapi.TinyFaceDetectorOptions({ inputSize: 320, scoreThreshold: 0.45 });

    try {
      const detections = await faceapi.detectAllFaces(video, opts);

      // Resize detections to match the display canvas size
      const resized = faceapi.resizeResults(detections, {
        width:  overlayC.width,
        height: overlayC.height
      });

      overlayCtx.clearRect(0, 0, overlayC.width, overlayC.height);

      const n = resized.length;

      resized.forEach(det => {
        const { x, y, width, height } = det.box;
        // Mirror the x coordinate to match the CSS-mirrored video
        const mx = overlayC.width - x - width;

        overlayCtx.strokeStyle = '#00ffe0';
        overlayCtx.lineWidth   = 2;
        overlayCtx.shadowColor = '#00ffe0';
        overlayCtx.shadowBlur  = 12;
        overlayCtx.strokeRect(mx, y, width, height);
        overlayCtx.shadowBlur  = 0;

        overlayCtx.font      = 'bold 11px monospace';
        overlayCtx.fillStyle = '#00ffe0';
        overlayCtx.fillText('FACE', mx + 4, y > 16 ? y - 6 : y + 14);

        // confidence badge
        const conf = Math.round(det.score * 100);
        overlayCtx.fillStyle = 'rgba(0,255,224,0.15)';
        overlayCtx.fillRect(mx, y, width, 18);
        overlayCtx.fillStyle = '#00ffe0';
        overlayCtx.font = '10px monospace';
        overlayCtx.fillText(`${conf}%`, mx + 4, y + 13);
      });

      if (n > 0) sendPhoneAlert(n);

      countEl.innerHTML = n > 0
        ? `${n}<span>${n === 1 ? 'face detected' : 'faces detected'}</span>`
        : '';

    } catch (e) { /* skip bad frame */ }
  }

  animId = requestAnimationFrame(loop);
}

/* ── kick off model loading immediately ────────────────────── */
loadModels();
</script>
</body>
</html>
