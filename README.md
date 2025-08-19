# rover.
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8" />
  <meta name="viewport" content="width=device-width, initial-scale=1" />
  <title>Mining Rover Sensors — Arduino Uno Dashboard</title>
  <link rel="preconnect" href="https://fonts.googleapis.com">
  <link rel="preconnect" href="https://fonts.gstatic.com" crossorigin>
  <link href="https://fonts.googleapis.com/css2?family=Inter:wght@400;600;700&display=swap" rel="stylesheet">
  <script src="https://cdn.jsdelivr.net/npm/chart.js"></script>
  <style>
    :root { --bg:#0f172a; --card:#111827; --muted:#9ca3af; --accent:#38bdf8; --text:#e5e7eb; }
    *{box-sizing:border-box}
    body{margin:0;font-family:Inter,system-ui,Segoe UI,Roboto,Arial,sans-serif;background:linear-gradient(180deg,#0b1220,#0f172a 20%,#0f172a);color:var(--text)}
    header{display:flex;gap:12px;align-items:center;justify-content:space-between;padding:18px 22px;border-bottom:1px solid #1f2937;position:sticky;top:0;background:rgba(15,23,42,.85);backdrop-filter:blur(8px);}
    h1{font-size: clamp(18px, 2.5vw, 26px);margin:0;font-weight:700;letter-spacing:.3px}
    .chip{font-size:12px;color:#93c5fd;background:#0b3e56;border:1px solid #1e90bb;padding:4px 8px;border-radius:999px}
    .wrap{max-width:1100px;margin:0 auto;padding:18px}
    .toolbar{display:flex;flex-wrap:wrap;gap:10px;align-items:center}
    button{appearance:none;border:0;border-radius:14px;padding:10px 14px;font-weight:600;cursor:pointer;transition:.2s;background:#0ea5e9;color:white}
    button.secondary{background:#1f2937}
    button.ghost{background:transparent;border:1px solid #334155}
    button:disabled{opacity:.5;cursor:not-allowed}
    .grid{display:grid;grid-template-columns:repeat(4,minmax(0,1fr));gap:14px}
    @media (max-width:1000px){.grid{grid-template-columns:repeat(2,minmax(0,1fr));}}
    @media (max-width:560px){.grid{grid-template-columns:1fr}}
    .card{background:linear-gradient(180deg,#0b1220,#0e1729);border:1px solid #1f2937;border-radius:18px;padding:14px;box-shadow:0 8px 30px rgba(0,0,0,.35)}
    .card h3{margin:0 0 6px 0;font-size:14px;color:#a5b4fc;font-weight:600}
    .big{font-size: clamp(22px, 4vw, 38px);font-weight:700;letter-spacing:.3px}
    .unit{font-size:12px;color:var(--muted);margin-left:6px}
    .row{display:flex;align-items:center;gap:8px}
    .status{font-size:12px;color:var(--muted)}
    .kpi{display:flex;align-items:flex-end;gap:6px}
    .cards-2{display:grid;grid-template-columns:1fr 1fr;gap:14px}
    .charts{display:grid;grid-template-columns:1fr;gap:14px;margin-top:16px}
    canvas{background:#0a1220;border:1px solid #1f2937;border-radius:12px;padding:8px}
    footer{padding:20px;color:#94a3b8;text-align:center}
    .log{font-family:ui-monospace,SFMono-Regular,Menlo,monospace;font-size:12px;background:#0a1220;border:1px solid #1f2937;border-radius:12px;padding:10px;height:160px;overflow:auto}
    .badge{border:1px solid #334155;color:#a3a3a3;border-radius:999px;padding:2px 8px;font-size:11px}
    .notice{border:1px dashed #334155;background:#0a1220;border-radius:12px;padding:10px;color:#cbd5e1}
    .ok{color:#22c55e}
    .warn{color:#f59e0b}
    .err{color:#ef4444}
    details summary{cursor:pointer}
    .testlog{font-family:ui-monospace,SFMono-Regular,Menlo,monospace;font-size:12px;white-space:pre-wrap}
    .hidden{display:none}
  </style>
</head>
<body>
  <header>
    <div class="row">
      <h1>Mining Rover — Live Sensor Dashboard</h1>
      <span class="chip">Arduino Uno • DHT11 • MQ-2 • MQ-7 • MQ-135</span>
    </div>
    <div class="toolbar">
      <button id="connectBtn">🔌 Connect</button>
      <button id="disconnectBtn" class="secondary" disabled>⏏ Disconnect</button>
      <button id="simulateBtn" class="secondary">🧪 Demo Data</button>
      <button id="uploadBtn" class="ghost">📄 Upload Log</button>
      <input id="fileInput" class="hidden" type="file" accept=".jsonl,.txt,.csv" />
      <button id="pasteBtn" class="ghost">📋 Paste Data</button>
      <span class="status" id="status">Environment check…</span>
    </div>
  </header>

  <div class="wrap">
    <div class="notice" id="policyNote" style="display:none">
      <strong>Heads up:</strong> Serial access is blocked in this sandboxed environment (Permissions Policy). You can still test via <em>Demo Data</em>, <em>Upload Log</em>, or <em>Paste Data</em> now. To read directly from Arduino, open this file outside the sandbox in Chrome/Edge over <code>https://</code> or <code>http://localhost</code>, then click <em>Connect</em>.
    </div>

    <div class="grid" style="margin-top:12px">
      <div class="card">
        <h3>Temperature</h3>
        <div class="kpi"><span class="big" id="temp">--</span><span class="unit">°C</span></div>
        <div class="status" id="heatIndex">Heat index: --</div>
      </div>
      <div class="card">
        <h3>Humidity</h3>
        <div class="kpi"><span class="big" id="hum">--</span><span class="unit">%</span></div>
        <div class="status">Comfort: <span id="comfort">--</span></div>
      </div>
      <div class="card">
        <h3>MQ-2 (Smoke/LPG)</h3>
        <div class="kpi"><span class="big" id="mq2">--</span><span class="unit">ADC</span></div>
        <div class="status">Status: <span class="badge" id="mq2State">--</span></div>
      </div>
      <div class="card">
        <h3>MQ-7 (CO)</h3>
        <div class="kpi"><span class="big" id="mq7">--</span><span class="unit">ADC</span></div>
        <div class="status">Status: <span class="badge" id="mq7State">--</span></div>
      </div>
      <div class="card">
        <h3>MQ-135 (Air Quality)</h3>
        <div class="kpi"><span class="big" id="mq135">--</span><span class="unit">ADC</span></div>
        <div class="status">Status: <span class="badge" id="mq135State">--</span></div>
      </div>
      <div class="card">
        <h3>Connection</h3>
        <div class="status" id="portInfo">Checking…</div>
        <div class="status">Last packet: <span id="lastTs">–</span></div>
      </div>
      <div class="card" style="grid-column: span 2">
        <h3>Live Log</h3>
        <div class="log" id="log"></div>
      </div>
    </div>

    <div class="charts">
      <canvas id="chartTemp"></canvas>
      <canvas id="chartGas"></canvas>
    </div>

    <details class="card" style="margin-top:14px">
      <summary>✅ Built-in Tests</summary>
      <div class="testlog" id="testOut">Running tests…</div>
    </details>

    <footer>
      If <strong>Connect</strong> is disabled here, you're likely in a sandbox (e.g., embedded preview). Save this file and open it directly in Chrome/Edge from a normal tab served over <code>https://</code> or <code>http://localhost</code>.
    </footer>
  </div>

<script>
// ======= Dashboard State =======
let port, reader, keepReading = false, useDemo = false;
let serialBlocked = false; // updated by environment check
let replayTimer = null; // for file/paste replay
const statusEl = document.getElementById('status');
const portInfoEl = document.getElementById('portInfo');
const logEl = document.getElementById('log');
const lastTsEl = document.getElementById('lastTs');
const policyNote = document.getElementById('policyNote');
const ui = {
  temp: document.getElementById('temp'), hum: document.getElementById('hum'),
  mq2: document.getElementById('mq2'), mq7: document.getElementById('mq7'), mq135: document.getElementById('mq135'),
  mq2State: document.getElementById('mq2State'), mq7State: document.getElementById('mq7State'), mq135State: document.getElementById('mq135State'),
  heatIndex: document.getElementById('heatIndex'), comfort: document.getElementById('comfort')
};

// ======= Charts =======
const tempChart = new Chart(document.getElementById('chartTemp'), {
  type: 'line',
  data: { labels: [], datasets: [
    { label: 'Temperature (°C)', data: [], tension:.25},
    { label: 'Humidity (%)', data: [], tension:.25, yAxisID:'y1'}
  ]},
  options: { responsive:true, animation:false, scales:{ y:{ beginAtZero:true }, y1:{ position:'right', beginAtZero:true } } }
});

const gasChart = new Chart(document.getElementById('chartGas'), {
  type: 'line',
  data: { labels: [], datasets: [
    { label: 'MQ-2 (ADC)', data: [], tension:.25 },
    { label: 'MQ-7 (ADC)', data: [], tension:.25 },
    { label: 'MQ-135 (ADC)', data: [], tension:.25 }
  ]},
  options: { responsive:true, animation:false, scales:{ y:{ beginAtZero:true } } }
});

function pushCharts(ts, pkt){
  const t = new Date(ts).toLocaleTimeString();
  const maxPts = 60; // last 60 points
  // Temp/Hum chart
  tempChart.data.labels.push(t);
  tempChart.data.datasets[0].data.push(pkt.tempC);
  tempChart.data.datasets[1].data.push(pkt.hum);
  // Gas chart
  gasChart.data.labels.push(t);
  gasChart.data.datasets[0].data.push(pkt.mq2);
  gasChart.data.datasets[1].data.push(pkt.mq7);
  gasChart.data.datasets[2].data.push(pkt.mq135);
  // trim
  for (const c of [tempChart, gasChart]){
    if (c.data.labels.length>maxPts){ c.data.labels.shift(); c.data.datasets.forEach(d=>d.data.shift()); }
    c.update('none');
  }
}

// ======= Helpers =======
function comfortLevel(temp, hum){
  if (temp<20 && hum<30) return 'Dry/Cool';
  if (temp>32 && hum>60) return 'Hot & Humid';
  if (hum>70) return 'Humid';
  if (hum<30) return 'Dry';
  return 'Comfortable';
}
function assessGas(adc){
  if (adc>800) return 'Danger';
  if (adc>600) return 'High';
  if (adc>400) return 'Elevated';
  return 'Normal';
}
function appendLog(line){
  const atBottom = Math.abs(logEl.scrollHeight - logEl.scrollTop - logEl.clientHeight) < 4;
  logEl.textContent += line + '\n';
  if (atBottom) logEl.scrollTop = logEl.scrollHeight;
}

// Simple heat index (approx, Celsius)
function heatIndexC(T, RH){
  // Rothfusz simplified for C (approx)
  const T_F = T*9/5+32;
  const HI_F = -42.379 + 2.04901523*T_F + 10.14333127*RH - 0.22475541*T_F*RH - 0.00683783*T_F*T_F - 0.05481717*RH*RH + 0.00122874*T_F*T_F*RH + 0.00085282*T_F*RH*RH - 0.00000199*T_F*T_F*RH*RH;
  return ((HI_F-32)*5/9);
}

function updateUI(pkt){
  ui.temp.textContent = (pkt.tempC??NaN).toFixed(1);
  ui.hum.textContent = (pkt.hum??NaN).toFixed(0);
  ui.mq2.textContent = pkt.mq2;
  ui.mq7.textContent = pkt.mq7;
  ui.mq135.textContent = pkt.mq135;
  ui.mq2State.textContent = assessGas(pkt.mq2);
  ui.mq7State.textContent = assessGas(pkt.mq7);
  ui.mq135State.textContent = assessGas(pkt.mq135);
  const hi = heatIndexC(pkt.tempC, pkt.hum);
  ui.heatIndex.textContent = `Heat index: ${isFinite(hi)?hi.toFixed(1):'--'} °C`;
  ui.comfort.textContent = comfortLevel(pkt.tempC, pkt.hum);
}

function handlePacket(pkt){
  // coerce types in case strings arrive from CSV
  pkt.tempC = Number(pkt.tempC);
  pkt.hum = Number(pkt.hum);
  pkt.mq2 = Number(pkt.mq2);
  pkt.mq7 = Number(pkt.mq7);
  pkt.mq135 = Number(pkt.mq135);
  if ([pkt.tempC,pkt.hum,pkt.mq2,pkt.mq7,pkt.mq135].some(v=>!isFinite(v))) return; // skip bad rows
  const ts = Date.now();
  lastTsEl.textContent = new Date(ts).toLocaleTimeString();
  updateUI(pkt);
  pushCharts(ts, pkt);
}

// ======= Robust line parser (JSONL or CSV) =======
function parseLine(line){
  line = line.trim();
  if (!line) return null;
  // JSON object
  if (line[0] === '{'){
    try { return JSON.parse(line); } catch { return null; }
  }
  // CSV: either values or header+values
  const parts = line.split(/[,\t]/).map(s=>s.trim());
  if (parts.length >= 5){
    // If header row, ignore
    if (/tempC/i.test(parts[0]) || /hum/i.test(parts[1])) return null;
    const [tempC, hum, mq2, mq7, mq135] = parts;
    return { tempC: Number(tempC), hum: Number(hum), mq2: Number(mq2), mq7: Number(mq7), mq135: Number(mq135) };
  }
  return null;
}

// ======= Environment / Permissions-Policy preflight =======
async function checkEnvironment(){
  const hasSerialAPI = ('serial' in navigator);
  const secure = window.isSecureContext; // needed for Web Serial
  let canUse = false;
  let blocked = false;
  if (hasSerialAPI && secure){
    try {
      // getPorts() is a harmless call, but will throw if Permissions Policy blocks "serial"
      await navigator.serial.getPorts();
      canUse = true;
    } catch(e){
      blocked = true;
    }
  }
  serialBlocked = !canUse;
  const connectBtn = document.getElementById('connectBtn');
  const disconnectBtn = document.getElementById('disconnectBtn');
  if (serialBlocked){
    connectBtn.disabled = true;
    disconnectBtn.disabled = true;
    portInfoEl.textContent = 'Serial blocked';
    statusEl.textContent = 'Use Demo / Upload / Paste';
    policyNote.style.display = '';
  } else {
    connectBtn.disabled = false;
    portInfoEl.textContent = 'Ready';
    statusEl.textContent = 'Idle';
    policyNote.style.display = 'none';
  }
}

// ======= Web Serial =======
async function connect(){
  if (!('serial' in navigator)){
    alert('Web Serial API not supported. Use Chrome/Edge on desktop.');
    return;
  }
  try{
    port = await navigator.serial.requestPort();
    await port.open({ baudRate: 9600 });
    portInfoEl.textContent = 'Connected';
    document.getElementById('connectBtn').disabled = true;
    document.getElementById('disconnectBtn').disabled = false;
    statusEl.textContent = 'Reading…';
    keepReading = true;

    const decoder = new TextDecoderStream();
    const inputDone = port.readable.pipeTo(decoder.writable);
    const inputStream = decoder.readable;
    reader = inputStream.getReader();

    let buffer = '';
    while (keepReading){
      const { value, done } = await reader.read();
      if (done) break;
      buffer += value;
      let idx;
      while ((idx = buffer.indexOf('\n')) >= 0){
        const line = buffer.slice(0, idx).trim();
        buffer = buffer.slice(idx+1);
        if (!line) continue;
        appendLog(line);
        const pkt = parseLine(line);
        if (pkt) handlePacket(pkt);
      }
    }
  }catch(err){
    // Catch SecurityError thrown by Permissions Policy block and switch to guidance
    console.error(err);
    appendLog('ERR: '+err.message);
    statusEl.textContent = 'Error: '+err.message;
    if ((err && /disallowed by permissions policy/i.test(err.message)) || err.name === 'SecurityError'){
      serialBlocked = true;
      document.getElementById('connectBtn').disabled = true;
      document.getElementById('disconnectBtn').disabled = true;
      portInfoEl.textContent = 'Serial blocked';
      policyNote.style.display = '';
    }
  }
}

async function disconnect(){
  try{
    keepReading = false;
    if (reader){ await reader.cancel(); reader.releaseLock(); }
    if (port){ await port.close(); }
  }catch(e){}
  document.getElementById('connectBtn').disabled = serialBlocked; // keep disabled if blocked
  document.getElementById('disconnectBtn').disabled = true;
  statusEl.textContent = 'Disconnected';
  portInfoEl.textContent = serialBlocked ? 'Serial blocked' : 'No port';
}

document.getElementById('connectBtn').addEventListener('click', connect);

document.getElementById('disconnectBtn').addEventListener('click', disconnect);

// ======= Demo data (for testing UI without hardware) =======
document.getElementById('simulateBtn').addEventListener('click', () => {
  useDemo = !useDemo;
  document.getElementById('simulateBtn').textContent = useDemo ? '🧪 Demo ON' : '🧪 Demo Data';
  if (useDemo){
    stopReplay();
    window.demoTimer = window.demoTimer || null;
    if (window.demoTimer) clearInterval(window.demoTimer);
    window.demoTimer = setInterval(()=>{
      const pkt = {
        tempC: +(24 + Math.sin(Date.now()/30000)*6 + Math.random()).toFixed(1),
        hum: Math.min(95, Math.max(20, Math.round(60 + Math.sin(Date.now()/25000)*20 + (Math.random()*6-3)))) ,
        mq2: Math.round(350 + Math.random()*500),
        mq7: Math.round(300 + Math.random()*550),
        mq135: Math.round(280 + Math.random()*600)
      };
      appendLog(JSON.stringify(pkt));
      handlePacket(pkt);
    }, 1000);
  } else {
    if (window.demoTimer) clearInterval(window.demoTimer);
  }
});

// ======= Upload & Paste fallback =======
const uploadBtn = document.getElementById('uploadBtn');
const fileInput = document.getElementById('fileInput');
const pasteBtn = document.getElementById('pasteBtn');

uploadBtn.addEventListener('click', () => fileInput.click());
fileInput.addEventListener('change', async (e) => {
  const file = e.target.files?.[0];
  if (!file) return;
  const text = await file.text();
  startReplay(text);
});

pasteBtn.addEventListener('click', async () => {
  const text = prompt('Paste JSON Lines or CSV with columns: tempC,hum,mq2,mq7,mq135');
  if (text) startReplay(text);
});

function startReplay(text){
  stopReplay();
  const lines = text.split(/\r?\n/).filter(Boolean);
  if (!lines.length) return;
  appendLog(`[REPLAY] ${lines.length} lines loaded`);
  let i = 0;
  replayTimer = setInterval(() => {
    if (i >= lines.length){ stopReplay(); return; }
    const line = lines[i++];
    appendLog(line);
    const pkt = parseLine(line);
    if (pkt) handlePacket(pkt);
  }, 1000);
}

function stopReplay(){ if (replayTimer){ clearInterval(replayTimer); replayTimer = null; } }

// ======= Tests =======
function runTests(){
  const out = [];
  let pass = 0, fail = 0;
  function t(name, fn){
    try{ fn(); out.push(`✔ ${name}`); pass++; }
    catch(e){ out.push(`✖ ${name}\n   → ${e.message}`); fail++; }
  }
  function eq(a,b,eps=0){
    if (eps){ if (Math.abs(a-b)>eps) throw new Error(`Expected ${a} ≈ ${b} (±${eps})`); }
    else if (a!==b) throw new Error(`Expected ${a} === ${b}`);
  }
  function ok(v,msg='Expected truthy'){ if (!v) throw new Error(msg); }

  // --- assessGas thresholds
  t('assessGas 350 ⇒ Normal', () => eq(assessGas(350),'Normal'));
  t('assessGas 450 ⇒ Elevated', () => eq(assessGas(450),'Elevated'));
  t('assessGas 650 ⇒ High', () => eq(assessGas(650),'High'));
  t('assessGas 900 ⇒ Danger', () => eq(assessGas(900),'Danger'));

  // --- comfortLevel categories
  t('comfortLevel 25°C/50% ⇒ Comfortable', () => eq(comfortLevel(25,50),'Comfortable'));
  t('comfortLevel 35°C/70% ⇒ Hot & Humid', () => eq(comfortLevel(35,70),'Hot & Humid'));
  t('comfortLevel 18°C/25% ⇒ Dry/Cool', () => eq(comfortLevel(18,25),'Dry/Cool'));

  // --- heatIndex rough expectations
  t('heatIndexC increases with humidity', () => {
    const hi1 = heatIndexC(30,30);
    const hi2 = heatIndexC(30,70);
    ok(hi2 > hi1);
  });
  t('heatIndexC 30°C/70% within plausible range', () => {
    const hi = heatIndexC(30,70);
    ok(hi > 30 && hi < 50, 'HI should be > T and < 50');
  });

  // --- parseLine JSON
  t('parseLine JSON object', () => {
    const pkt = parseLine('{"tempC":28.5,"hum":60,"mq2":500,"mq7":450,"mq135":420}');
    eq(pkt.tempC,28.5); eq(pkt.hum,60); eq(pkt.mq2,500); eq(pkt.mq7,450); eq(pkt.mq135,420);
  });
  // --- parseLine CSV
  t('parseLine CSV values', () => {
    const pkt = parseLine('28.5,60,500,450,420');
    eq(pkt.tempC,28.5); eq(pkt.hum,60); eq(pkt.mq2,500); eq(pkt.mq7,450); eq(pkt.mq135,420);
  });
  t('parseLine ignores CSV header', () => {
    const pkt = parseLine('tempC,hum,mq2,mq7,mq135');
    eq(pkt,null);
  });

  // --- pushCharts trimming
  t('pushCharts caps to 60 points', () => {
    for (let i=0;i<65;i++) pushCharts(Date.now()+i*1000,{tempC:25,hum:50,mq2:400,mq7:400,mq135:400});
    ok(tempChart.data.labels.length<=60 && gasChart.data.labels.length<=60,'Charts should trim to 60 points');
  });

  document.getElementById('testOut').textContent = out.join('\n')+`\n\n${pass} passed, ${fail} failed.`;
}

window.addEventListener('DOMContentLoaded', async () => {
  await checkEnvironment();
  runTests();
});
</script>
</body>
</html>
