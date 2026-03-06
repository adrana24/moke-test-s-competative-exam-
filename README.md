# moke-test-s-competative-exam-
education program 
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8" />
  <meta name="viewport" content="width=device-width, initial-scale=1.0"/>
  <title>MockTest – Instant MCQ Exam Creator & Proctor</title>

  <!-- pdf.js -->
  <script src="https://cdnjs.cloudflare.com/ajax/libs/pdf.js/4.7.432/pdf.min.mjs" type="module"></script>
  <!-- Tesseract.js for OCR -->
  <script src="https://cdn.jsdelivr.net/npm/tesseract.js@5/dist/tesseract.min.js"></script>

  <style>
    body {
      font-family: system-ui, -apple-system, sans-serif;
      background: #f9fafb;
      margin: 0;
      padding: 20px;
      color: #1f2937;
    }
    .container { max-width: 960px; margin: 0 auto; }
    h1 { color: #111827; text-align: center; }
    .subtitle { text-align: center; color: #4b5563; margin-bottom: 2rem; }
    .card {
      background: white;
      border-radius: 12px;
      box-shadow: 0 4px 20px rgba(0,0,0,0.07);
      padding: 2rem;
      margin-bottom: 2rem;
    }
    textarea, input, select {
      width: 100%;
      padding: 0.9rem;
      margin: 0.8rem 0;
      border: 1px solid #d1d5db;
      border-radius: 6px;
      font-size: 1rem;
    }
    textarea { min-height: 220px; font-family: ui-monospace, monospace; }
    button {
      background: #3b82f6;
      color: white;
      border: none;
      padding: 0.9rem 1.8rem;
      border-radius: 8px;
      font-size: 1.05rem;
      cursor: pointer;
    }
    button:hover:not(:disabled) { background: #2563eb; }
    button:disabled { background: #9ca3af; cursor: not-allowed; opacity: 0.7; }

    .input-mode {
      display: flex;
      gap: 1.5rem;
      justify-content: center;
      margin: 1.5rem 0;
      flex-wrap: wrap;
    }
    .input-mode label { font-size: 1.05rem; cursor: pointer; }
    .tab { display: none; }
    .tab.active { display: block; }

    .status {
      padding: 0.8rem;
      border-radius: 6px;
      margin: 0.8rem 0;
      font-weight: 500;
    }
    .status.error   { background: #fee2e2; color: #991b1b; border: 1px solid #fecaca; }
    .status.warning { background: #fef3c7; color: #92400e; border: 1px solid #fde68a; }
    .status.success { background: #dcfce7; color: #166534; border: 1px solid #bbf7d0; }

    progress { width: 100%; height: 10px; margin: 1rem 0; }

    #result-link {
      margin-top: 1.5rem;
      padding: 1rem;
      background: #eff6ff;
      border: 1px solid #bfdbfe;
      border-radius: 8px;
      word-break: break-all;
      font-family: monospace;
    }

    /* ── EXAM ──────────────────────────────────────── */
    .exam-container { max-width: 800px; margin: 2rem auto; }
    .timer { font-size: 2.5rem; font-weight: bold; text-align: center; color: #e11d48; margin: 1.5rem 0; }
    .question {
      background: white;
      padding: 1.6rem;
      margin-bottom: 1.6rem;
      border-radius: 10px;
      box-shadow: 0 2px 12px rgba(0,0,0,0.06);
    }
    .option {
      display: block;
      padding: 0.9rem;
      margin: 0.5rem 0;
      border: 1px solid #e5e7eb;
      border-radius: 6px;
      cursor: pointer;
    }
    .option:hover { background: #f3f4f6; }
    #submit-btn { display: block; margin: 3rem auto; padding: 1rem 4rem; font-size: 1.3rem; }

    .certificate-container {
      margin: 2rem auto;
      max-width: 800px;
      text-align: center;
      background: white;
      padding: 2rem;
      border-radius: 12px;
      box-shadow: 0 4px 25px rgba(0,0,0,0.1);
    }
    #certificateCanvas { border: 2px solid #ddd; border-radius: 8px; max-width: 100%; }
  </style>
</head>
<body>

<div class="container" id="home">

  <h1>MockTest</h1>
  <p class="subtitle">Create timed MCQ exams • JSON / Text / PDF / Scanned OCR • Certificate & Video Proof</p>

  <div class="card">
    <div class="input-mode">
      <label><input type="radio" name="inputMode" value="json" checked> JSON</label>
      <label><input type="radio" name="inputMode" value="text"> Plain Text</label>
      <label><input type="radio" name="inputMode" value="pdf"> Clean PDF</label>
      <label><input type="radio" name="inputMode" value="ocr"> Scanned PDF / Image (OCR)</label>
    </div>

    <div id="json-tab" class="tab active">
      <label>Time (minutes):</label>
      <input type="number" id="minutesJson" value="30" min="1" max="180">
      <label>Paste JSON array:</label>
      <textarea id="jsonInput" placeholder='[{ "question": "Example?", "options": ["A","B","C","D"], "correct": 1 }, ...]'></textarea>
    </div>

    <div id="text-tab" class="tab">
      <label>Time (minutes):</label>
      <input type="number" id="minutesText" value="30" min="1" max="180">
      <label>Paste questions (numbered + A) B) etc.):</label>
      <textarea id="textInput" placeholder="1. Question here?\nA) opt1\nB) opt2\nC) opt3\nD) opt4\n\n2. Next..."></textarea>
    </div>

    <div id="pdf-tab" class="tab">
      <label>Time (minutes):</label>
      <input type="number" id="minutesPdf" value="30" min="1" max="180">
      <label>Upload clean (text-selectable) PDF:</label>
      <input type="file" id="pdfFile" accept=".pdf">
      <button onclick="parseCleanPdf()" id="pdfBtn" disabled>Extract & Parse</button>
      <textarea id="pdfPreview" readonly placeholder="Parsed questions will appear here..."></textarea>
      <div class="status" id="pdfStatus"></div>
    </div>

    <div id="ocr-tab" class="tab">
      <label>Time (minutes):</label>
      <input type="number" id="minutesOcr" value="30" min="1" max="180">
      <label>OCR Language:</label>
      <select id="ocrLang">
        <option value="eng">English</option>
        <option value="hin">Hindi</option>
        <option value="guj">Gujarati</option>
        <option value="eng+hin">English + Hindi</option>
      </select>
      <label>Upload scanned PDF or image:</label>
      <input type="file" id="ocrFile" accept=".pdf,image/*">
      <button onclick="runOcr()" id="ocrBtn" disabled>Run OCR & Parse</button>
      <progress id="ocrProgress" value="0" max="100" style="display:none"></progress>
      <div class="status" id="ocrStatus"></div>
      <textarea id="ocrPreview" readonly></textarea>
    </div>

    <button onclick="createExamLink()" style="margin-top:2rem; width:100%; padding:1.2rem; font-size:1.2rem;">
      Create & Get Shareable Link
    </button>
    <div id="result-link"></div>
  </div>

</div>


<!-- ── EXAM INTERFACE ──────────────────────────────────────── -->
<div class="container exam-container" id="exam" style="display:none">
  <h1>Mock Test</h1>
  <div class="timer" id="timer">--:--</div>

  <div id="proctorNotice" class="status warning" style="display:none; margin-bottom:1.5rem;">
    <strong>Recording active:</strong> Webcam + microphone are being recorded.
  </div>

  <form id="quizForm"></form>
  <button id="submitBtn" style="display:none; margin:3rem auto; padding:1rem 4rem; font-size:1.3rem;" onclick="finishExam()">
    Submit Exam
  </button>

  <div id="resultArea" style="display:none"></div>

  <!-- Proof & Certificate -->
  <div id="proofArea" style="display:none">
    <div class="certificate-container">
      <h2>Certificate of Completion</h2>
      <canvas id="certCanvas" width="800" height="600"></canvas><br><br>
      <button onclick="downloadCertificate()">Download Certificate (PNG)</button>
    </div>

    <div style="margin-top:3rem; text-align:center;">
      <h3>Video Proof (optional)</h3>
      <p id="videoMsg">No video recorded.</p>
      <video id="proofVideo" controls style="max-width:100%; display:none;"></video><br>
      <button id="dlVideoBtn" style="display:none" onclick="downloadVideoProof()">Download Video Proof (WebM)</button>
    </div>
  </div>
</div>

<script type="module">
// ──────────────────────────────────────────────
// Globals
// ──────────────────────────────────────────────
import * as pdfjs from 'https://cdnjs.cloudflare.com/ajax/libs/pdf.js/4.7.432/pdf.min.mjs';
pdfjs.GlobalWorkerOptions.workerSrc = 'https://cdnjs.cloudflare.com/ajax/libs/pdf.js/4.7.432/pdf.worker.min.mjs';

let questions = [];
let timeLeft = 0;
let timerInterval = null;
let mediaRecorder = null;
let recordedChunks = [];
let videoBlobUrl = null;
let studentName = "Student";

// ──────────────────────────────────────────────
// Helpers
// ──────────────────────────────────────────────
function showStatus(id, msg, type = '') {
  const el = document.getElementById(id + 'Status') || document.getElementById(id);
  if (el) {
    el.textContent = msg;
    el.className = `status ${type}`;
  }
}

function compressData(obj) {
  return btoa(encodeURIComponent(JSON.stringify(obj)));
}

function decompressData(str) {
  try {
    return JSON.parse(decodeURIComponent(atob(str)));
  } catch {
    return null;
  }
}

function formatQuestionsPreview(qs) {
  return qs.map((q, i) => 
    `${i+1}. \( {q.question}\n \){q.options.map((o,j) => `   ${String.fromCharCode(65+j)}) ${o}`).join('\n')}`
  ).join('\n\n');
}

// ──────────────────────────────────────────────
// Text → Questions parser (basic version)
// ──────────────────────────────────────────────
function parseTextQuestions(text) {
  const qList = [];
  const blocks = text.split(/\n\s*\n/).filter(b => b.trim());

  for (let block of blocks) {
    const lines = block.split('\n').map(l => l.trim()).filter(Boolean);
    if (lines.length < 3) continue;

    let question = lines[0].replace(/^\d+[.)]\s*/, '').trim();
    let opts = [];
    let correct = -1;

    for (let i = 1; i < lines.length; i++) {
      const m = lines[i].match(/^([A-D1-4])\s*[).]\s*(.+)/i);
      if (m) {
        opts.push(m[2].trim());
        if (/correct|ans|key|✓|✔/i.test(lines[i])) correct = opts.length - 1;
      }
    }

    if (opts.length >= 2) {
      qList.push({ question, options: opts, correct: correct >= 0 ? correct : 0 });
    }
  }
  return qList;
}

// ──────────────────────────────────────────────
// Create link
// ──────────────────────────────────────────────
async function createExamLink() {
  let mins, qs;

  const mode = document.querySelector('input[name="inputMode"]:checked').value;

  try {
    switch (mode) {
      case 'json':
        mins = +document.getElementById('minutesJson').value;
        const raw = document.getElementById('jsonInput').value.trim();
        qs = JSON.parse(raw);
        if (!Array.isArray(qs)) throw new Error("Must be array");
        break;

      case 'text':
        mins = +document.getElementById('minutesText').value;
        qs = parseTextQuestions(document.getElementById('textInput').value);
        break;

      case 'pdf':
      case 'ocr':
        mins = +(document.getElementById(mode === 'pdf' ? 'minutesPdf' : 'minutesOcr').value);
        qs = questions; // set by parse functions
        break;
    }

    if (!mins || mins < 1 || mins > 180) throw new Error("Invalid time");
    if (!qs?.length) throw new Error("No questions parsed");

    const payload = { t: mins, q: qs };
    const hash = compressData(payload);
    const url = new URL(location);
    url.searchParams.set('e', hash);

    document.getElementById('result-link').innerHTML = `
      <strong>Exam link:</strong><br>
      <a href="\( {url}" target="_blank"> \){url.href}</a><br><br>
      <small>${qs.length} questions • ${mins} min</small>
    `;
  } catch (err) {
    alert("Error: " + err.message);
  }
}

// ──────────────────────────────────────────────
// Clean PDF parser (simplified)
// ──────────────────────────────────────────────
async function parseCleanPdf() {
  const file = document.getElementById('pdfFile').files[0];
  if (!file) return alert("Select PDF first");

  showStatus('pdf', "Reading PDF...", "");

  try {
    const buf = await file.arrayBuffer();
    const pdf = await pdfjs.getDocument({data: buf}).promise;

    let text = "";
    for (let i = 1; i <= pdf.numPages; i++) {
      const page = await pdf.getPage(i);
      const cont = await page.getTextContent();
      text += cont.items.map(it => it.str).join(" ") + "\n\n";
    }

    questions = parseTextQuestions(text);
    document.getElementById('pdfPreview').value = formatQuestionsPreview(questions) || "No questions found";
    showStatus('pdf', `${questions.length} questions parsed`, questions.length ? "success" : "error");
  } catch (e) {
    showStatus('pdf', "Error: " + e.message, "error");
  }
}

// ──────────────────────────────────────────────
// OCR (simplified)
// ──────────────────────────────────────────────
async function runOcr() {
  // Very basic version – full implementation would be longer
  showStatus('ocr', "OCR not fully implemented in this combined version", "warning");
  // You can paste the detailed OCR code from previous messages here if needed
}

// ──────────────────────────────────────────────
// Exam start
// ──────────────────────────────────────────────
function startExam(data) {
  document.getElementById('home').style.display = 'none';
  document.getElementById('exam').style.display = 'block';

  questions = data.q;
  timeLeft = data.t * 60;
  studentName = prompt("Your name (for certificate):")?.trim() || "Student";

  renderQuestions();
  startTimer();

  // Optional recording
  if (confirm("Record webcam + mic as proof? (video saved locally)")) {
    navigator.mediaDevices.getUserMedia({video: true, audio: true})
      .then(stream => {
        mediaRecorder = new MediaRecorder(stream);
        mediaRecorder.ondataavailable = e => recordedChunks.push(e.data);
        mediaRecorder.start();
        document.getElementById('proctorNotice').style.display = 'block';
      })
      .catch(err => console.warn("Recording denied", err));
  }
}

function renderQuestions() {
  const form = document.getElementById('quizForm');
  form.innerHTML = "";

  questions.forEach((q, i) => {
    const div = document.createElement('div');
    div.className = 'question';
    div.innerHTML = `
      <h3>${i+1}. ${q.question}</h3>
      ${q.options.map((opt, j) => `
        <label class="option">
          <input type="radio" name="q\( {i}" value=" \){j}">
          ${String.fromCharCode(65 + j)}) ${opt}
        </label>
      `).join('')}
    `;
    form.appendChild(div);
  });

  document.getElementById('submitBtn').style.display = 'block';
}

function startTimer() {
  const el = document.getElementById('timer');
  timerInterval = setInterval(() => {
    if (timeLeft <= 0) {
      clearInterval(timerInterval);
      finishExam(true);
      return;
    }
    const m = String(Math.floor(timeLeft / 60)).padStart(2, '0');
    const s = String(timeLeft % 60).padStart(2, '0');
    el.textContent = `\( {m}: \){s}`;
    timeLeft--;
  }, 1000);
}

function finishExam(timeup = false) {
  clearInterval(timerInterval);
  if (mediaRecorder?.state === 'recording') mediaRecorder.stop();

  let score = 0;
  let html = '<h2>Results</h2>';

  questions.forEach((q, i) => {
    const ans = document.querySelector(`input[name="q${i}"]:checked`);
    const idx = ans ? +ans.value : -1;
    const isCorrect = idx === q.correct;
    if (isCorrect) score++;

    html += `
      <div class="question">
        <h3>${i+1}. ${q.question}</h3>
        <p>Your answer: <span style="color:${isCorrect?'green':'red'}">
          ${idx >= 0 ? String.fromCharCode(65+idx) + ") " + q.options[idx] : "—"}
        </span></p>
        <p>Correct: <span style="color:green">${String.fromCharCode(65 + q.correct)}) ${q.options[q.correct]}</span></p>
      </div>`;
  });

  const perc = Math.round(score / questions.length * 100);
  html += `<h2 style="text-align:center">Score: ${score} / \( {questions.length} ( \){perc}%)</h2>`;

  document.getElementById('resultArea').innerHTML = html;
  document.getElementById('resultArea').style.display = 'block';
  document.getElementById('proofArea').style.display = 'block';

  if (timeup) alert("Time's up!");

  prepareCertificate(score, perc);
  if (recordedChunks.length) {
    const blob = new Blob(recordedChunks, {type: 'video/webm'});
    videoBlobUrl = URL.createObjectURL(blob);
    document.getElementById('proofVideo').src = videoBlobUrl;
    document.getElementById('proofVideo').style.display = 'block';
    document.getElementById('dlVideoBtn').style.display = 'inline-block';
    document.getElementById('videoMsg').textContent = "Video proof ready";
  }
}

// ──────────────────────────────────────────────
// Certificate
// ──────────────────────────────────────────────
function prepareCertificate(score, perc) {
  const c = document.getElementById('certCanvas');
  const ctx = c.getContext('2d');
  ctx.fillStyle = '#ffffff';
  ctx.fillRect(0,0,c.width,c.height);
  ctx.strokeStyle = '#3b82f6';
  ctx.lineWidth = 12;
  ctx.strokeRect(30,30,c.width-60,c.height-60);

  ctx.font = 'bold 48px serif';
  ctx.fillStyle = '#1e40af';
  ctx.textAlign = 'center';
  ctx.fillText('Certificate of Completion', c.width/2, 140);

  ctx.font = '32px serif';
  ctx.fillStyle = '#374151';
  ctx.fillText('This is to certify that', c.width/2, 220);

  ctx.font = 'bold 44px serif';
  ctx.fillText(studentName, c.width/2, 300);

  ctx.font = '28px serif';
  ctx.fillText(`scored \( {score}/ \){questions.length} (${perc}%)`, c.width/2, 360);

  const date = new Date().toLocaleDateString('en-IN');
  ctx.font = '24px serif';
  ctx.fillText(`Date: ${date} • MockTest`, c.width/2, 500);
}

function downloadCertificate() {
  const link = document.createElement('a');
  link.download = `certificate_${studentName.replace(/\s+/g,'_')}.png`;
  link.href = document.getElementById('certCanvas').toDataURL();
  link.click();
}

function downloadVideoProof() {
  if (!videoBlobUrl) return;
  const a = document.createElement('a');
  a.href = videoBlobUrl;
  a.download = `exam_proof_${studentName.replace(/\s+/g,'_')}.webm`;
  a.click();
}

// ──────────────────────────────────────────────
// Init – check if exam mode
// ──────────────────────────────────────────────
window.addEventListener('load', () => {
  const params = new URLSearchParams(location.search);
  const examData = params.get('e');
  if (examData) {
    const payload = decompressData(examData);
    if (payload?.t && payload?.q?.length) {
      startExam(payload);
    } else {
      alert("Invalid or corrupted exam link");
    }
  }

  // Enable buttons when files selected
  document.getElementById('pdfFile').onchange = () => document.getElementById('pdfBtn').disabled = false;
  document.getElementById('ocrFile').onchange = () => document.getElementById('ocrBtn').disabled = false;
});
</script>
</body>
</html>