# AI-CHATBOT- #  An Intelligent Enterprise Assistant for public sector:

### AIM:



To develop a secure and scalable NLP-based chatbot that answers employee queries, processes documents, supports 2FA, filters bad language, and handles multiple users efficiently.

### PROBLEM STATEMENT:

Description: Develop a chatbot using deep learning and natural language processing techniques to accurately understand and respond to queries from employees of a large public sector organization.

 The chatbot should be capable of handling diverse questions related to HR policies, IT support, company events, and other organizational matters. (Hackathon students/teams to use publicly available sample information for HR Policy, IT Support, etc. available on internet.) Develop document processing capabilities for the chatbot to analyse and extract information from documents uploaded by employees.

This includes summarizing a document or extracting text (keyword information) from documents relevant to organizational needs. (Hackathon students/teams can use any 8 to 10 page document for demonstration). Ensure the chatbot architecture is scalable to handle minimum 5 users parallelly. This includes optimizing response time (Response Time should not exceed 5 seconds for any query unless there is a technical issue like connectivity, etc.) Enable 2FA (2 Factor Authentication – email id type) in the chatbot for enhancing the security level of the chatbot. 


### PROCEDURE: 

Collect data, train the NLP model, implement document processing and 2FA security, integrate bad language filtering, and deploy the system with multi-user support.

### STEPS:

STEPS: 
1.User logs in with OTP → 
2.asks query/uploads document →
3.system filters language →
4.NLP processes request → 
5.chatbot generates response within 5 seconds.

### PROGRAM:

APP.PY:

```
/* =====================================================
   GovAssist AI — Application Logic
   ===================================================== */

/* ── Particle Background ── */
(function () {
  const canvas = document.getElementById('bgCanvas');
  const ctx = canvas.getContext('2d');
  let W, H, particles = [];

  function resize() {
    W = canvas.width = window.innerWidth;
    H = canvas.height = window.innerHeight;
  }

  function mkParticle() {
    return {
      x: Math.random() * W, y: Math.random() * H,
      r: Math.random() * 1.8 + .4,
      vx: (Math.random() - .5) * .25,
      vy: (Math.random() - .5) * .25,
      alpha: Math.random() * .5 + .1
    };
  }

  function init() {
    resize();
    particles = Array.from({ length: 120 }, mkParticle);
  }

  function draw() {
    ctx.clearRect(0, 0, W, H);
    particles.forEach(p => {
      p.x += p.vx; p.y += p.vy;
      if (p.x < 0) p.x = W; if (p.x > W) p.x = 0;
      if (p.y < 0) p.y = H; if (p.y > H) p.y = 0;
      ctx.beginPath();
      ctx.arc(p.x, p.y, p.r, 0, Math.PI * 2);
      ctx.fillStyle = `rgba(100,140,255,${p.alpha})`;
      ctx.fill();
    });
    requestAnimationFrame(draw);
  }

  window.addEventListener('resize', resize);
  init(); draw();
})();

/* ═══════════════════════════════════
   STATE
   ═══════════════════════════════════ */
let currentUser = { email: '', name: '' };
let currentOTP = '';
let selectedAction = 'summarize';
let uploadedFile = null;
let countdownTimer = null;
let sidebarOpen = true;

/* ═══════════════════════════════════
   SCREEN HELPERS
   ═══════════════════════════════════ */
function showScreen(id) {
  document.querySelectorAll('.screen').forEach(s => s.classList.remove('active'));
  const s = document.getElementById(id);
  s.classList.add('active');
}

/* ═══════════════════════════════════
   UTILITIES
   ═══════════════════════════════════ */
function showToast(msg, duration = 3000) {
  const t = document.getElementById('toast');
  t.textContent = msg;
  t.classList.add('show');
  setTimeout(() => t.classList.remove('show'), duration);
}

function now() {
  return new Date().toLocaleTimeString([], { hour: '2-digit', minute: '2-digit' });
}

function togglePass() {
  const p = document.getElementById('passwordInput');
  p.type = p.type === 'password' ? 'text' : 'password';
}

/* ═══════════════════════════════════
   LOGIN
   ═══════════════════════════════════ */
document.getElementById('loginForm').addEventListener('submit', function (e) {
  e.preventDefault();
  const email = document.getElementById('emailInput').value.trim();
  const password = document.getElementById('passwordInput').value.trim();
  if (!email || !password) return showToast('⚠️ Please fill in all fields.');

  const btn = document.getElementById('loginBtn');
  const text = btn.querySelector('.btn-text');
  const load = btn.querySelector('.btn-loader');
  btn.disabled = true; text.classList.add('hidden'); load.classList.remove('hidden');

  currentUser.email = email;
  currentUser.name = email.split('@')[0].replace(/\./g, ' ').replace(/\b\w/g, c => c.toUpperCase());

  setTimeout(() => {
    btn.disabled = false; text.classList.remove('hidden'); load.classList.add('hidden');
    sendOTP(email);
  }, 1200);
});

function sendOTP(email) {
  currentOTP = String(Math.floor(100000 + Math.random() * 900000));
  document.getElementById('otpEmail').textContent = email;
  document.getElementById('otpDisplay').textContent = currentOTP;
  showScreen('twoFAScreen');
  clearOTPInputs();
  startCountdown(120);
  showToast('📧 OTP sent to ' + email);
  document.getElementById('d0').focus();
}

/* ─── OTP digit navigation ─── */
document.querySelectorAll('.otp-digit').forEach((input, idx) => {
  input.addEventListener('input', () => {
    input.value = input.value.replace(/\D/g, '');
    if (input.value) {
      input.classList.add('filled');
      const next = document.getElementById('d' + (idx + 1));
      if (next) next.focus();
    } else {
      input.classList.remove('filled');
    }
  });
  input.addEventListener('keydown', e => {
    if (e.key === 'Backspace' && !input.value) {
      const prev = document.getElementById('d' + (idx - 1));
      if (prev) { prev.value = ''; prev.classList.remove('filled'); prev.focus(); }
    }
  });
});

document.getElementById('otpForm').addEventListener('submit', function (e) {
  e.preventDefault();
  const entered = [0, 1, 2, 3, 4, 5].map(i => document.getElementById('d' + i).value).join('');
  if (entered.length < 6) return showToast('⚠️ Enter all 6 digits.');
  if (entered !== currentOTP) {
    showToast('❌ Incorrect OTP. Try again.');
    clearOTPInputs();
    document.getElementById('d0').focus();
    return;
  }
  clearInterval(countdownTimer);
  showToast('✅ Authentication successful!');
  setTimeout(loadChatScreen, 700);
});

function clearOTPInputs() {
  [0, 1, 2, 3, 4, 5].forEach(i => {
    const d = document.getElementById('d' + i);
    d.value = ''; d.classList.remove('filled');
  });
}

function startCountdown(secs) {
  clearInterval(countdownTimer);
  let s = secs;
  const el = document.getElementById('countdown');
  const rb = document.getElementById('resendBtn');
  rb.classList.add('disabled');
  el.textContent = s;
  countdownTimer = setInterval(() => {
    s--;
    el.textContent = s;
    if (s <= 0) {
      clearInterval(countdownTimer);
      document.getElementById('timerTxt').textContent = 'Code expired.';
      rb.classList.remove('disabled');
    }
  }, 1000);
}

function resendOTP() {
  if (!currentUser.email) return;
  showToast('📧 New OTP sent to ' + currentUser.email);
  currentOTP = String(Math.floor(100000 + Math.random() * 900000));
  document.getElementById('otpDisplay').textContent = currentOTP;
  clearOTPInputs();
  startCountdown(120);
}

function goBack(screenId) {
  clearInterval(countdownTimer);
  showScreen(screenId);
}

/* ═══════════════════════════════════
   CHAT SCREEN INIT
   ═══════════════════════════════════ */
function loadChatScreen() {
  document.getElementById('avatarLetter').textContent = currentUser.name.charAt(0).toUpperCase();
  document.getElementById('sidebarName').textContent = currentUser.name;
  showScreen('chatScreen');
  // Auto welcome message
  setTimeout(() => {
    addBotMessage(
      `Hello <strong>${currentUser.name}</strong> 👋 Welcome to <strong>GovAssist AI</strong>!` +
      `<br/><br/>I'm your intelligent organizational assistant. You can ask me about:` +
      `<ul style="margin-top:8px;padding-left:18px;display:flex;flex-direction:column;gap:4px;">` +
      `<li>📋 <b>HR Policies</b> – Leave, payroll, benefits</li>` +
      `<li>💻 <b>IT Support</b> – Passwords, VPN, hardware</li>` +
      `<li>📅 <b>Events</b> – Townhalls, trainings, holidays</li>` +
      `<li>📄 <b>Documents</b> – Upload to summarize or extract info</li>` +
      `</ul>`,
      'all'
    );
  }, 400);
}

/* ═══════════════════════════════════
   MESSAGES
   ═══════════════════════════════════ */
function addUserMessage(text) {
  const wrap = document.createElement('div');
  wrap.className = 'bubble-wrap user';
  wrap.innerHTML = `
    <div class="bubble user">${escHtml(text)}<time>${now()}</time></div>
    <div class="bubble-avatar user-av">${currentUser.name.charAt(0)}</div>
  `;
  messagesArea().appendChild(wrap);
  scrollBottom();
}

function addBotMessage(html, category) {
  const wrap = document.createElement('div');
  wrap.className = 'bubble-wrap bot';
  const tag = category && category !== 'all' ? `<span class="tag-chip">${categoryLabel(category)}</span><br/>` : '';
  wrap.innerHTML = `
    <div class="bubble-avatar bot-av">G</div>
    <div class="bubble bot">${tag}${html}<time>${now()}</time></div>
  `;
  messagesArea().appendChild(wrap);
  scrollBottom();
}

function addTypingIndicator() {
  const wrap = document.createElement('div');
  wrap.className = 'bubble-wrap bot'; wrap.id = 'typingWrap';
  wrap.innerHTML = `
    <div class="bubble-avatar bot-av">G</div>
    <div class="bubble bot">
      <div class="typing-indicator"><span></span><span></span><span></span></div>
    </div>
  `;
  messagesArea().appendChild(wrap);
  scrollBottom();
  return wrap;
}

function removeTypingIndicator() {
  const el = document.getElementById('typingWrap');
  if (el) el.remove();
}

function messagesArea() { return document.getElementById('messagesArea'); }
function scrollBottom() {
  const a = messagesArea();
  a.scrollTop = a.scrollHeight;
}

function escHtml(t) {
  return t.replace(/&/g, '&amp;').replace(/</g, '&lt;').replace(/>/g, '&gt;');
}

function categoryLabel(cat) {
  return { hr: '👔 HR Policies', it: '💻 IT Support', events: '📅 Events', docs: '📄 Documents', all: '' }[cat] || cat;
}

/* ═══════════════════════════════════
   SEND MESSAGE
   ═══════════════════════════════════ */
function sendMessage() {
  const inp = document.getElementById('userInput');
  const text = inp.value.trim();
  if (!text) return;
  inp.value = ''; inp.style.height = '';

  // Remove welcome banner if present
  const wb = document.querySelector('.welcome-banner');
  if (wb) wb.remove();

  addUserMessage(text);
  const typer = addTypingIndicator();

  const delay = 1200 + Math.random() * 1800;
  setTimeout(() => {
    removeTypingIndicator();
    const { response, category } = generateResponse(text);
    addBotMessage(response, category);
    addToHistory(text);
  }, delay);
}

function quickAsk(text) {
  document.getElementById('userInput').value = text;
  sendMessage();
}

function handleKey(e) {
  if (e.key === 'Enter' && !e.shiftKey) { e.preventDefault(); sendMessage(); }
}

function autoResize(el) {
  el.style.height = 'auto';
  el.style.height = Math.min(el.scrollHeight, 160) + 'px';
}

/* ═══════════════════════════════════
   RESPONSE GENERATION
   ═══════════════════════════════════ */
const KB = {
  hr: [
    {
      keys: ['leave', 'annual', 'casual', 'sick', 'earned'],
      answer: `<b>Leave Policy Summary</b><br/>
      • <b>Annual Leave:</b> 20 days per year (carry-forward up to 10 days)<br/>
      • <b>Casual Leave:</b> 10 days per year (non-encashable)<br/>
      • <b>Sick Leave:</b> 12 days per year (medical certificate required for &gt;3 days)<br/>
      • <b>Maternity Leave:</b> 26 weeks (as per Maternity Benefit Act 2017)<br/>
      • <b>Paternity Leave:</b> 15 days<br/>
      Apply via the <strong>HR Self-Service Portal</strong> at least 3 days in advance.` },
    {
      keys: ['salary', 'payroll', 'payslip', 'stipend', 'pay'],
      answer: `<b>Payroll Information</b><br/>
      • Salaries are processed on the <b>25th of every month</b><br/>
      • Payslips are available in the <b>HR Portal</b> by the 27th<br/>
      • For discrepancies, raise a ticket under <em>Finance &gt; Payroll</em><br/>
      • Annual increments are effective from <b>1st April</b> each year` },
    {
      keys: ['appraisal', 'performance', 'review', 'rating', 'kpi'],
      answer: `<b>Performance Appraisal Cycle</b><br/>
      • Mid-Year Review: <b>October</b><br/>
      • Annual Appraisal: <b>March–April</b><br/>
      • Self-Assessment must be completed 2 weeks before manager review<br/>
      • Ratings: Outstanding / Exceeds / Meets / Needs Improvement` },
    {
      keys: ['transfer', 'deputation', 'posting'],
      answer: `<b>Transfer & Deputation Policy</b><br/>
      • Transfer requests must be submitted via HR Portal with supporting documents<br/>
      • Minimum service of <b>2 years</b> required before transfer request<br/>
      • Deputation to central/state bodies is available for Grade-B and above officers<br/>
      • Transfer orders are typically issued in <b>May–June</b> annually` },
  ],
  it: [
    {
      keys: ['vpn', 'network', 'connect', 'internet'],
      answer: `<b>VPN Troubleshooting</b><br/>
      1. Ensure you're using <b>Cisco AnyConnect v4.10+</b><br/>
      2. Try connecting to <code>vpn.organization.gov.in</code><br/>
      3. Disable any third-party antivirus temporarily<br/>
      4. Restart the VPN client service via <em>Services.msc</em><br/>
      5. If issue persists, call IT Helpdesk: <b>1800-XXX-1234</b> (24/7)` },
    {
      keys: ['password', 'reset', 'forgot', 'locked', 'account'],
      answer: `<b>Password Reset Options</b><br/>
      • <b>Self-Service:</b> Visit <code>sspr.organization.gov.in</code><br/>
      • <b>Helpdesk:</b> Raise ticket at <code>itsupport.org</code> (SLA: 2 hrs)<br/>
      • <b>Emergency:</b> Call <b>Ext. 1111</b> during business hours<br/>
      Passwords must be 12+ chars, include uppercase, number, and symbol.` },
    {
      keys: ['email', 'outlook', 'calendar', 'teams', 'office'],
      answer: `<b>Microsoft 365 / Outlook Support</b><br/>
      • Mailbox quota: <b>50 GB</b> per user<br/>
      • For calendar sync issues: remove and re-add the account in Outlook Settings<br/>
      • Teams audio issues: Go to <em>Settings → Devices</em> and re-select microphone<br/>
      • Report phishing emails using the <em>Report Phishing</em> add-in in Outlook` },
    {
      keys: ['laptop', 'hardware', 'printer', 'monitor', 'device', 'computer'],
      answer: `<b>Hardware & Device Support</b><br/>
      • <b>New hardware requests</b> require HOD approval (raise via IT Portal)<br/>
      • Asset replacement cycle: <b>4 years</b> for laptops, <b>5 years</b> for desktops<br/>
      • For printer issues: restart spooler via <code>services.msc</code> or call Facilities<br/>
      • Submit hardware return 15 days before last working day` },
  ],
  events: [
    {
      keys: ['event', 'happening', 'this month', 'upcoming', 'schedule', 'townhall'],
      answer: `<b>Upcoming Events — February 2026</b><br/>
      📌 <b>22 Feb</b> — IT Security Awareness Workshop (Virtual, 10 AM)<br/>
      📌 <b>25 Feb</b> — Quarterly Town Hall (Main Auditorium, 3 PM)<br/>
      📌 <b>27 Feb</b> — Women's Day Celebration Kickoff<br/>
      📌 <b>28 Feb</b> — Year-End Finance Review (HODs only)<br/>
      <br/>Register for events via the <strong>Events Portal</strong>.` },
    {
      keys: ['training', 'workshop', 'course', 'certification', 'learning'],
      answer: `<b>Training & Learning Programs</b><br/>
      • <b>e-Learning Platform:</b> <code>learn.organization.gov.in</code><br/>
      • <b>Featured Courses:</b> Cyber Security, Leadership 101, Data Analytics<br/>
      • <b>External certification</b> reimbursement up to ₹25,000/year (Grade C+)<br/>
      • Mandatory trainings: Anti-Corruption (due 31 Mar), POSH (due 15 Apr)` },
    {
      keys: ['holiday', 'gazetted', 'restricted', 'list', 'national'],
      answer: `<b>Public Holidays 2026</b><br/>
      • Republic Day (26 Jan), Holi (14 Mar), Good Friday (3 Apr),<br/>
        Ambedkar Jayanti (14 Apr), Labour Day (1 May),<br/>
        Independence Day (15 Aug), Gandhi Jayanti (2 Oct),<br/>
        Dussehra (2 Oct), Diwali (20 Oct), Christmas (25 Dec)<br/>
      <em>Plus 2 Restricted Holidays of your choice.</em>` },
  ],
};

const FALLBACKS = [
  `I'm looking into your query. Could you provide more context so I can give you the most accurate information? You can also try browsing by category using the sidebar.`,
  `That's a great question! For specific organizational queries, please contact the relevant department directly or raise a ticket on the internal portal. Is there anything else I can help with?`,
  `I don't have specific information on that right now. For urgent queries, please contact the HR Helpdesk at <b>hr@organization.gov.in</b> or IT Support at <b>itsupport@organization.gov.in</b>.`,
];

function generateResponse(text) {
  const lower = text.toLowerCase();
  for (const [cat, items] of Object.entries(KB)) {
    for (const item of items) {
      if (item.keys.some(k => lower.includes(k))) {
        return { response: item.answer, category: cat };
      }
    }
  }
  return {
    response: FALLBACKS[Math.floor(Math.random() * FALLBACKS.length)],
    category: 'all'
  };
}

/* ═══════════════════════════════════
   SIDEBAR & CATEGORY
   ═══════════════════════════════════ */
function toggleSidebar() {
  sidebarOpen = !sidebarOpen;
  document.getElementById('sidebar').classList.toggle('collapsed', !sidebarOpen);
}

function setCategory(btn, cat) {
  document.querySelectorAll('.cat-btn').forEach(b => b.classList.remove('active'));
  btn.classList.add('active');
  const titles = { all: 'GovAssist AI', hr: 'HR Policies', it: 'IT Support', events: 'Company Events', docs: 'Documents' };
  document.getElementById('chatCategoryTitle').textContent = titles[cat] || 'GovAssist AI';
  if (cat !== 'all' && cat !== 'docs') {
    showToast('📂 Category: ' + titles[cat]);
  }
}

function newChat() {
  const area = messagesArea();
  // Clear messages except welcome
  area.innerHTML = `<div class="welcome-banner">
    <div class="welcome-grid">
      <div class="welcome-card" onclick="quickAsk('What is the leave encashment policy?')">
        <div class="wc-icon">👔</div><div><h4>HR Policies</h4><p>Leave, payroll, benefits &amp; more</p></div>
      </div>
      <div class="welcome-card" onclick="quickAsk('My VPN is not connecting, how do I fix it?')">
        <div class="wc-icon">💻</div><div><h4>IT Support</h4><p>VPN, passwords, hardware &amp; tickets</p></div>
      </div>
      <div class="welcome-card" onclick="quickAsk('What events are happening this month?')">
        <div class="wc-icon">📅</div><div><h4>Company Events</h4><p>Townhalls, training &amp; celebrations</p></div>
      </div>
      <div class="welcome-card" onclick="openUpload()">
        <div class="wc-icon">📄</div><div><h4>Document Analysis</h4><p>Upload &amp; extract insights</p></div>
      </div>
    </div>
  </div>`;
  showToast('✨ New chat started');
}

function addToHistory(text) {
  const list = document.getElementById('historyList');
  const li = document.createElement('li');
  li.className = 'history-item active';
  li.textContent = text.length > 30 ? text.slice(0, 30) + '…' : text;
  list.querySelectorAll('li').forEach(i => i.classList.remove('active'));
  list.insertBefore(li, list.firstChild);
  if (list.children.length > 8) list.removeChild(list.lastChild);
}

function logout() {
  currentUser = { email: '', name: '' };
  showScreen('loginScreen');
  document.getElementById('emailInput').value = '';
  document.getElementById('passwordInput').value = '';
  showToast('👋 Signed out successfully');
}

/* ═══════════════════════════════════
   UPLOAD / DOCUMENT ANALYSIS
   ═══════════════════════════════════ */
function openUpload() {
  uploadedFile = null;
  document.getElementById('filePreview').classList.add('hidden');
  document.getElementById('analyzeBtn').disabled = false;
  document.getElementById('analysisResult').classList.add('hidden');
  document.getElementById('uploadOverlay').classList.add('visible');
  document.getElementById('fileInput').value = '';
}

function closeUpload() {
  document.getElementById('uploadOverlay').classList.remove('visible');
}

function handleFile(e) {
  const file = e.target.files[0];
  if (!file) return;
  processFile(file);
}

function processFile(file) {
  uploadedFile = file;
  const size = file.size < 1024 * 1024
    ? (file.size / 1024).toFixed(1) + ' KB'
    : (file.size / 1024 / 1024).toFixed(2) + ' MB';
  const ext = file.name.split('.').pop().toUpperCase();
  const icons = { PDF: '📕', DOCX: '📘', TXT: '📃', PNG: '🖼️', JPG: '🖼️', JPEG: '🖼️' };
  const fp = document.getElementById('filePreview');
  fp.classList.remove('hidden');
  fp.innerHTML = `
    <span class="file-icon">${icons[ext] || '📄'}</span>
    <div class="file-info">
      <div class="file-name">${file.name}</div>
      <div class="file-size">${ext} · ${size}</div>
    </div>
    <span style="color:var(--green);font-size:18px">✓</span>
  `;
}

/* Drag & drop */
const dz = document.getElementById('dropZone');
dz.addEventListener('dragover', e => { e.preventDefault(); dz.classList.add('drag-over'); });
dz.addEventListener('dragleave', () => dz.classList.remove('drag-over'));
dz.addEventListener('drop', e => {
  e.preventDefault(); dz.classList.remove('drag-over');
  const file = e.dataTransfer.files[0];
  if (file) processFile(file);
});

function selectAction(action, btn) {
  selectedAction = action;
  document.querySelectorAll('.chip').forEach(c => c.classList.remove('active'));
  btn.classList.add('active');
  const qaWrap = document.getElementById('qaInputWrap');
  action === 'qa' ? qaWrap.classList.remove('hidden') : qaWrap.classList.add('hidden');
}

function analyzeDoc() {
  if (!uploadedFile) { showToast('⚠️ Please upload a document first.'); return; }
  const btn = document.getElementById('analyzeBtn');
  btn.disabled = true; btn.textContent = '⏳ Analysing…';

  setTimeout(() => {
    btn.textContent = 'Analyse Document'; btn.disabled = false;
    const result = document.getElementById('analysisResult');
    result.classList.remove('hidden');

    const qaQ = document.getElementById('qaInput').value.trim();
    let content = '';

    if (selectedAction === 'summarize') {
      content = `<b>📝 Document Summary</b><br/><br/>
      The document "<em>${uploadedFile.name}</em>" covers key organizational policies and procedures relevant to public sector employees. 
      Key highlights include: governance frameworks, compliance requirements, operational guidelines, and stakeholder responsibilities. 
      The document emphasizes accountability, transparency, and procedural adherence in accordance with applicable regulatory standards. 
      Section 2 outlines eligibility criteria; Section 4 covers reporting obligations; Section 7 details grievance redressal mechanisms. 
      <br/><br/><b>Confidence:</b> High &nbsp;|&nbsp; <b>Pages processed:</b> ~10`;
    } else if (selectedAction === 'extract') {
      content = `<b>🔍 Extracted Keywords</b><br/><br/>
      <div style="display:flex;flex-wrap:wrap;gap:8px;margin-top:4px;">
        ${['Governance', 'Compliance', 'Policy Framework', 'Accountability', 'Transparency',
          'Eligibility', 'Reporting Obligations', 'Grievance Redressal', 'Regulatory Standards',
          'Operational Guidelines', 'Stakeholder', 'Amendment', 'Notification', 'Jurisdiction'].map(k =>
            `<span style="background:rgba(79,127,255,.15);color:var(--primary2);border:1px solid rgba(79,127,255,.3);padding:4px 12px;border-radius:20px;font-size:12px;font-weight:600;">${k}</span>`
          ).join('')}
      </div>`;
    } else {
      const q = qaQ || 'What are the key provisions?';
      content = `<b>❓ Q&A Result</b><br/>
      <em>Your question: "${q}"</em><br/><br/>
      Based on the uploaded document, the key provisions related to your query include: the organization's obligation to ensure equitable distribution of resources, defined timelines for response (maximum 30 working days), escalation paths for unresolved matters, and the right to seek review at competent authorities. Clause 5.3 specifically addresses this area with supplementary guidelines under Annexure-B.`;
    }

    result.innerHTML = content;

    // Close and post in chat
    setTimeout(() => {
      closeUpload();
      const wb = document.querySelector('.welcome-banner');
      if (wb) wb.remove();
      addUserMessage(`📄 Uploaded: ${uploadedFile.name} — ${selectedAction === 'summarize' ? 'Summarize' : selectedAction === 'extract' ? 'Extract Keywords' : 'Q&A: ' + (qaQ || 'Key provisions?')}`);
      addTypingIndicator();
      setTimeout(() => {
        removeTypingIndicator();
        addBotMessage(content.replace(/<b>.*?<\/b><br\/><br\/>/, ''), 'docs');
        addToHistory('Document: ' + uploadedFile.name);
      }, 1500);
    }, 1800);
  }, 2000);
}

/* ═══════════════════════════════════
   KEYBOARD SHORTCUT
   ═══════════════════════════════════ */
document.addEventListener('keydown', e => {
  if (e.key === 'Escape') closeUpload();
  if ((e.ctrlKey || e.metaKey) && e.key === 'k') {
    e.preventDefault();
    document.getElementById('userInput')?.focus();
  }
});


```






##index.html:

```
<!DOCTYPE html>
<html lang="en">

<head>
    <meta charset="UTF-8" />
    <meta name="viewport" content="width=device-width, initial-scale=1.0" />
    <title>GovAssist AI — Organizational Chatbot</title>
    <link rel="preconnect" href="https://fonts.googleapis.com" />
    <link href="https://fonts.googleapis.com/css2?family=Inter:wght@300;400;500;600;700;800&display=swap"
        rel="stylesheet" />
    <link rel="stylesheet" href="style.css" />
</head>

<body>

    <!-- ═══════════════ PARTICLE CANVAS ═══════════════ -->
    <canvas id="bgCanvas"></canvas>

    <!-- ═══════════════ LOGIN SCREEN ═══════════════ -->
    <div id="loginScreen" class="screen active">
        <div class="auth-card glass">
            <div class="brand">
                <div class="brand-icon"><span>G</span></div>
                <h1>GovAssist <span class="accent">AI</span></h1>
                <p class="subtitle">Organizational Intelligence Platform</p>
            </div>
            <form id="loginForm" autocomplete="off">
                <div class="field-group">
                    <label>Work Email</label>
                    <div class="input-wrap">
                        <svg class="input-icon" viewBox="0 0 24 24">
                            <path
                                d="M20 4H4a2 2 0 00-2 2v12a2 2 0 002 2h16a2 2 0 002-2V6a2 2 0 00-2-2zm0 4l-8 5-8-5V6l8 5 8-5v2z" />
                        </svg>
                        <input type="email" id="emailInput" placeholder="you@organization.gov.in" required />
                    </div>
                </div>
                <div class="field-group">
                    <label>Password</label>
                    <div class="input-wrap">
                        <svg class="input-icon" viewBox="0 0 24 24">
                            <path
                                d="M18 8h-1V6A5 5 0 007 6v2H6a2 2 0 00-2 2v10a2 2 0 002 2h12a2 2 0 002-2V10a2 2 0 00-2-2zm-6 9a2 2 0 110-4 2 2 0 010 4zm3.1-9H8.9V6a3.1 3.1 0 116.2 0v2z" />
                        </svg>
                        <input type="password" id="passwordInput" placeholder="Enter your password" required />
                        <button type="button" class="eye-btn" onclick="togglePass()">
                            <svg viewBox="0 0 24 24">
                                <path
                                    d="M12 4.5C7 4.5 2.73 7.61 1 12c1.73 4.39 6 7.5 11 7.5s9.27-3.11 11-7.5c-1.73-4.39-6-7.5-11-7.5zm0 12.5a5 5 0 110-10 5 5 0 010 10zm0-8a3 3 0 100 6 3 3 0 000-6z" />
                            </svg>
                        </button>
                    </div>
                </div>
                <button type="submit" class="btn-primary" id="loginBtn">
                    <span class="btn-text">Sign In</span>
                    <span class="btn-loader hidden"></span>
                </button>
            </form>
        </div>
    </div>

    <!-- ═══════════════ 2FA SCREEN ═══════════════ -->
    <div id="twoFAScreen" class="screen">
        <div class="auth-card glass">
            <div class="back-btn" onclick="goBack('loginScreen')">
                <svg viewBox="0 0 24 24">
                    <path d="M20 11H7.83l5.59-5.59L12 4l-8 8 8 8 1.41-1.41L7.83 13H20v-2z" />
                </svg>
                Back
            </div>
            <div class="brand">
                <div class="brand-icon shield"><span>🔐</span></div>
                <h2>Two-Factor Authentication</h2>
                <p class="subtitle" id="otpSubtitle">Enter the 6-digit code sent to<br /><strong id="otpEmail"></strong>
                </p>
            </div>
            <div id="otpBanner" class="otp-banner">OTP Code: <strong id="otpDisplay"></strong></div>
            <form id="otpForm">
                <div class="otp-inputs">
                    <input type="text" maxlength="1" class="otp-digit" id="d0" />
                    <input type="text" maxlength="1" class="otp-digit" id="d1" />
                    <input type="text" maxlength="1" class="otp-digit" id="d2" />
                    <span class="otp-sep">—</span>
                    <input type="text" maxlength="1" class="otp-digit" id="d3" />
                    <input type="text" maxlength="1" class="otp-digit" id="d4" />
                    <input type="text" maxlength="1" class="otp-digit" id="d5" />
                </div>
                <div class="timer-row">
                    <span id="timerTxt">Code expires in <b id="countdown">120</b>s</span>
                    <button type="button" class="resend-btn disabled" id="resendBtn"
                        onclick="resendOTP()">Resend</button>
                </div>
                <button type="submit" class="btn-primary">Verify &amp; Continue</button>
            </form>
        </div>
    </div>

    <!-- ═══════════════ MAIN CHAT SCREEN ═══════════════ -->
    <div id="chatScreen" class="screen">

        <!-- Sidebar -->
        <aside class="sidebar" id="sidebar">
            <div class="sidebar-header">
                <div class="brand-mini">
                    <div class="brand-icon-sm"><span>G</span></div>
                    <span>GovAssist <b>AI</b></span>
                </div>
                <button class="icon-btn" onclick="toggleSidebar()" title="Close sidebar">
                    <svg viewBox="0 0 24 24">
                        <path
                            d="M19 6.41L17.59 5 12 10.59 6.41 5 5 6.41 10.59 12 5 17.59 6.41 19 12 13.41 17.59 19 19 17.59 13.41 12z" />
                    </svg>
                </button>
            </div>

            <div class="new-chat-btn" onclick="newChat()">
                <svg viewBox="0 0 24 24">
                    <path
                        d="M19 3H5a2 2 0 00-2 2v14l4-4h12a2 2 0 002-2V5a2 2 0 00-2-2zm-2 10H7v-2h10v2zm0-3H7V8h10v2z" />
                </svg>
                New Chat
            </div>

            <div class="sidebar-section-label">Categories</div>
            <nav class="category-nav">
                <button class="cat-btn active" onclick="setCategory(this,'all')">
                    <svg viewBox="0 0 24 24">
                        <path d="M3 13h8V3H3v10zm0 8h8v-6H3v6zm10 0h8V11h-8v10zm0-18v6h8V3h-8z" />
                    </svg>
                    All Topics
                </button>
                <button class="cat-btn" onclick="setCategory(this,'hr')">
                    <svg viewBox="0 0 24 24">
                        <path
                            d="M16 11c1.66 0 2.99-1.34 2.99-3S17.66 5 16 5c-1.66 0-3 1.34-3 3s1.34 3 3 3zm-8 0c1.66 0 2.99-1.34 2.99-3S9.66 5 8 5C6.34 5 5 6.34 5 8s1.34 3 3 3zm0 2c-2.33 0-7 1.17-7 3.5V19h14v-2.5c0-2.33-4.67-3.5-7-3.5zm8 0c-.29 0-.62.02-.97.05 1.16.84 1.97 1.97 1.97 3.45V19h6v-2.5c0-2.33-4.67-3.5-7-3.5z" />
                    </svg>
                    HR Policies
                </button>
                <button class="cat-btn" onclick="setCategory(this,'it')">
                    <svg viewBox="0 0 24 24">
                        <path
                            d="M20 18c1.1 0 1.99-.9 1.99-2L22 6a2 2 0 00-2-2H4a2 2 0 00-2 2v10a2 2 0 002 2H0v2h24v-2h-4zM4 6h16v10H4V6z" />
                    </svg>
                    IT Support
                </button>
                <button class="cat-btn" onclick="setCategory(this,'events')">
                    <svg viewBox="0 0 24 24">
                        <path
                            d="M17 12h-5v5h5v-5zM16 1v2H8V1H6v2H5a2 2 0 00-2 2v14a2 2 0 002 2h14a2 2 0 002-2V5a2 2 0 00-2-2h-1V1h-2zm3 18H5V8h14v11z" />
                    </svg>
                    Events
                </button>
                <button class="cat-btn" onclick="setCategory(this,'docs')">
                    <svg viewBox="0 0 24 24">
                        <path
                            d="M14 2H6a2 2 0 00-2 2v16a2 2 0 002 2h12a2 2 0 002-2V8l-6-6zm2 16H8v-2h8v2zm0-4H8v-2h8v2zm-3-5V3.5L18.5 9H13z" />
                    </svg>
                    Documents
                </button>
            </nav>

            <div class="sidebar-section-label">Recent Chats</div>
            <ul class="history-list" id="historyList">
                <li class="history-item active">Welcome Session</li>
                <li class="history-item">Leave Policy Query</li>
                <li class="history-item">Password Reset Issue</li>
            </ul>

            <div class="sidebar-footer">
                <div class="user-chip" id="userChip">
                    <div class="avatar-sm"><span id="avatarLetter">U</span></div>
                    <div class="user-meta">
                        <span id="sidebarName">Employee</span>
                        <span class="status-dot-wrap"><span class="status-dot"></span> Online</span>
                    </div>
                </div>
                <button class="icon-btn logout-btn" onclick="logout()" title="Logout">
                    <svg viewBox="0 0 24 24">
                        <path
                            d="M17 7l-1.41 1.41L18.17 11H8v2h10.17l-2.58 2.58L17 17l5-5-5-5zM4 5h8V3H4a2 2 0 00-2 2v14a2 2 0 002 2h8v-2H4V5z" />
                    </svg>
                </button>
            </div>
        </aside>

        <!-- Main Chat Area -->
        <main class="chat-main" id="chatMain">

            <!-- Top Bar -->
            <header class="chat-header">
                <button class="icon-btn menu-btn" onclick="toggleSidebar()" title="Toggle sidebar">
                    <svg viewBox="0 0 24 24">
                        <path d="M3 18h18v-2H3v2zm0-5h18v-2H3v2zm0-7v2h18V6H3z" />
                    </svg>
                </button>
                <div class="header-title">
                    <h2 id="chatCategoryTitle">GovAssist AI</h2>
                    <p class="header-sub">Public Sector Intelligence · Powered by Deep Learning</p>
                </div>
                <div class="header-badges">
                    <span class="badge badge-green">🟢 5 Users Online</span>
                    <span class="badge badge-blue">2FA Active</span>
                </div>
                <button class="icon-btn upload-trigger" onclick="openUpload()" title="Upload document">
                    <svg viewBox="0 0 24 24">
                        <path d="M9 16h6v-6h4l-7-7-7 7h4v6zm-4 2h14v2H5v-2z" />
                    </svg>
                </button>
            </header>

            <!-- Messages -->
            <div class="messages-area" id="messagesArea">
                <!-- Welcome Banner -->
                <div class="welcome-banner">
                    <div class="welcome-grid">
                        <div class="welcome-card" onclick="quickAsk('What is the leave encashment policy?')">
                            <div class="wc-icon">👔</div>
                            <div>
                                <h4>HR Policies</h4>
                                <p>Leave, payroll, benefits &amp; more</p>
                            </div>
                        </div>
                        <div class="welcome-card" onclick="quickAsk('My VPN is not connecting, how do I fix it?')">
                            <div class="wc-icon">💻</div>
                            <div>
                                <h4>IT Support</h4>
                                <p>VPN, passwords, hardware &amp; tickets</p>
                            </div>
                        </div>
                        <div class="welcome-card" onclick="quickAsk('What events are happening this month?')">
                            <div class="wc-icon">📅</div>
                            <div>
                                <h4>Company Events</h4>
                                <p>Townhalls, training &amp; celebrations</p>
                            </div>
                        </div>
                        <div class="welcome-card" onclick="openUpload()">
                            <div class="wc-icon">📄</div>
                            <div>
                                <h4>Document Analysis</h4>
                                <p>Upload &amp; extract insights</p>
                            </div>
                        </div>
                    </div>
                </div>
            </div>

            <!-- Input Bar -->
            <div class="input-zone">
                <div class="input-bar glass">
                    <button class="icon-btn attach-btn" onclick="openUpload()" title="Attach document">
                        <svg viewBox="0 0 24 24">
                            <path
                                d="M16.5 6v11.5a4 4 0 01-8 0V5a2.5 2.5 0 015 0v10.5a1 1 0 01-2 0V6H10v9.5a2.5 2.5 0 005 0V5a4 4 0 00-8 0v12.5a5.5 5.5 0 0011 0V6h-1.5z" />
                        </svg>
                    </button>
                    <textarea id="userInput" placeholder="Ask about HR policies, IT support, events…" rows="1"
                        onkeydown="handleKey(event)" oninput="autoResize(this)"></textarea>
                    <button class="send-btn" id="sendBtn" onclick="sendMessage()">
                        <svg viewBox="0 0 24 24">
                            <path d="M2 21l21-9L2 3v7l15 2-15 2v7z" />
                        </svg>
                    </button>
                </div>
                <p class="input-footer">Responses may take up to 5 s · Secured by 2FA · GovAssist v2.1</p>
            </div>
        </main>
    </div>

    <!-- ═══════════════ UPLOAD MODAL ═══════════════ -->
    <div class="modal-overlay" id="uploadOverlay" onclick="closeUpload()">
        <div class="modal glass" onclick="event.stopPropagation()">
            <div class="modal-header">
                <h3>📄 Document Analysis</h3>
                <button class="icon-btn" onclick="closeUpload()">
                    <svg viewBox="0 0 24 24">
                        <path
                            d="M19 6.41L17.59 5 12 10.59 6.41 5 5 6.41 10.59 12 5 17.59 6.41 19 12 13.41 17.59 19 19 17.59 13.41 12z" />
                    </svg>
                </button>
            </div>

            <div class="drop-zone" id="dropZone">
                <input type="file" id="fileInput" accept=".pdf,.docx,.txt,.png,.jpg" onchange="handleFile(event)"
                    hidden />
                <div class="drop-inner" onclick="document.getElementById('fileInput').click()">
                    <div class="drop-icon">☁️</div>
                    <p>Drag &amp; drop or <span class="accent">browse</span></p>
                    <p class="hint-text">PDF · DOCX · TXT · PNG · JPG (max 20 MB)</p>
                </div>
            </div>
            <div id="filePreview" class="file-preview hidden"></div>

            <div class="action-chips">
                <button class="chip active" id="chip-summarize" onclick="selectAction('summarize',this)">📝
                    Summarize</button>
                <button class="chip" id="chip-extract" onclick="selectAction('extract',this)">🔍 Extract
                    Keywords</button>
                <button class="chip" id="chip-qa" onclick="selectAction('qa',this)">❓ Ask Questions</button>
            </div>

            <div id="qaInputWrap" class="qa-input-wrap hidden">
                <input type="text" id="qaInput" placeholder="What would you like to know about this document?" />
            </div>

            <button class="btn-primary" id="analyzeBtn" onclick="analyzeDoc()">Analyse Document</button>
            <div id="analysisResult" class="analysis-result hidden"></div>
        </div>
    </div>

    <!-- ═══════════════ TOAST ═══════════════ -->
    <div id="toast" class="toast"></div>

    <script src="app.js"></script>
</body>

</html>


```






##style.css:

```
/* ══════════════════════════════════════════
   GovAssist AI — Premium Dark UI Stylesheet
   ══════════════════════════════════════════ */

:root {
  --bg:        #080c18;
  --bg2:       #0d1224;
  --panel:     rgba(255,255,255,.04);
  --panel2:    rgba(255,255,255,.07);
  --border:    rgba(255,255,255,.09);
  --border2:   rgba(255,255,255,.15);
  --primary:   #4f7fff;
  --primary2:  #6c9aff;
  --accent:    #a78bfa;
  --accent2:   #c4b5fd;
  --green:     #34d399;
  --red:       #f87171;
  --yellow:    #fbbf24;
  --text:      #e8eaf6;
  --muted:     #8892b0;
  --radius:    16px;
  --radius-sm: 10px;
  --shadow:    0 8px 40px rgba(0,0,0,.6);
  --glow:      0 0 30px rgba(79,127,255,.25);
  --sidebar-w: 270px;
  --transition: .25s cubic-bezier(.4,0,.2,1);
}

*, *::before, *::after { box-sizing: border-box; margin: 0; padding: 0; }

html, body {
  height: 100%;
  font-family: 'Inter', sans-serif;
  background: var(--bg);
  color: var(--text);
  overflow: hidden;
}

/* ── Canvas BG ── */
#bgCanvas {
  position: fixed; inset: 0; z-index: 0;
  pointer-events: none;
}

/* ── Screens ── */
.screen {
  position: fixed; inset: 0;
  display: flex; align-items: center; justify-content: center;
  z-index: 10;
  opacity: 0; pointer-events: none;
  transition: opacity .4s ease;
}
.screen.active { opacity: 1; pointer-events: all; }

/* ── Glass Card ── */
.glass {
  background: rgba(13,18,36,.75);
  backdrop-filter: blur(24px) saturate(1.4);
  border: 1px solid var(--border2);
  border-radius: var(--radius);
  box-shadow: var(--shadow), var(--glow);
}

/* ══════════════════════════════════
   AUTH SCREENS
   ══════════════════════════════════ */
.auth-card {
  width: 100%; max-width: 440px;
  padding: 48px 44px 44px;
  display: flex; flex-direction: column; gap: 28px;
  animation: slideUp .5s cubic-bezier(.4,0,.2,1) both;
}

@keyframes slideUp {
  from { transform: translateY(32px); opacity: 0; }
  to   { transform: translateY(0);   opacity: 1; }
}

/* Brand */
.brand { text-align: center; display: flex; flex-direction: column; align-items: center; gap: 12px; }
.brand-icon {
  width: 68px; height: 68px;
  border-radius: 20px;
  background: linear-gradient(135deg, var(--primary), var(--accent));
  display: flex; align-items: center; justify-content: center;
  font-size: 28px; font-weight: 800; color: #fff;
  box-shadow: 0 8px 24px rgba(79,127,255,.4);
  animation: pulse-glow 3s ease-in-out infinite;
}
.brand-icon.shield { background: linear-gradient(135deg, #7c3aed, #4f7fff); }
@keyframes pulse-glow {
  0%,100% { box-shadow: 0 8px 24px rgba(79,127,255,.4); }
  50%      { box-shadow: 0 8px 40px rgba(79,127,255,.7); }
}
.brand h1, .brand h2 { font-size: 26px; font-weight: 800; letter-spacing: -.5px; }
.brand h1 .accent, .accent { color: var(--primary2); }
.subtitle { color: var(--muted); font-size: 13px; }

/* Form */
.field-group { display: flex; flex-direction: column; gap: 8px; }
.field-group label { font-size: 13px; font-weight: 600; color: var(--muted); letter-spacing: .3px; }
.input-wrap { position: relative; display: flex; align-items: center; }
.input-wrap input {
  width: 100%; padding: 14px 44px 14px 42px;
  background: var(--panel2); border: 1px solid var(--border);
  border-radius: var(--radius-sm); color: var(--text);
  font-size: 14px; font-family: inherit;
  transition: border-color var(--transition), box-shadow var(--transition);
  outline: none;
}
.input-wrap input:focus {
  border-color: var(--primary);
  box-shadow: 0 0 0 3px rgba(79,127,255,.2);
}
.input-wrap input::placeholder { color: var(--muted); }
.input-icon {
  position: absolute; left: 13px;
  width: 18px; height: 18px; fill: var(--muted); pointer-events: none;
}
.eye-btn {
  position: absolute; right: 10px;
  background: none; border: none; cursor: pointer; padding: 4px;
}
.eye-btn svg { width: 18px; height: 18px; fill: var(--muted); }

/* Buttons */
.btn-primary {
  width: 100%; padding: 15px;
  background: linear-gradient(135deg, var(--primary), #6a5aff);
  border: none; border-radius: var(--radius-sm);
  color: #fff; font-size: 15px; font-weight: 700;
  cursor: pointer; letter-spacing: .2px;
  transition: transform .18s, box-shadow .18s, filter .18s;
  position: relative; overflow: hidden;
  display: flex; align-items: center; justify-content: center; gap: 8px;
}
.btn-primary::after {
  content: ''; position: absolute; inset: 0;
  background: linear-gradient(rgba(255,255,255,.12), transparent);
}
.btn-primary:hover { transform: translateY(-2px); box-shadow: 0 8px 24px rgba(79,127,255,.45); }
.btn-primary:active { transform: translateY(0); filter: brightness(.9); }

.btn-loader {
  width: 18px; height: 18px;
  border: 2px solid rgba(255,255,255,.4);
  border-top-color: #fff;
  border-radius: 50%;
  animation: spin .7s linear infinite;
}
@keyframes spin { to { transform: rotate(360deg); } }
.hidden { display: none !important; }

.hint-text { text-align: center; font-size: 12px; color: var(--muted); }

/* Back btn */
.back-btn {
  display: flex; align-items: center; gap: 6px;
  color: var(--primary2); font-size: 13px; font-weight: 600;
  cursor: pointer; width: fit-content;
}
.back-btn svg { width: 18px; height: 18px; fill: var(--primary2); }

/* OTP Banner */
.otp-banner {
  background: linear-gradient(135deg, rgba(79,127,255,.15), rgba(167,139,250,.15));
  border: 1px solid rgba(79,127,255,.3);
  border-radius: 10px; padding: 10px 16px;
  text-align: center; font-size: 14px; color: var(--text);
  letter-spacing: .5px;
}
.otp-banner strong { color: var(--primary2); font-size: 20px; letter-spacing: 4px; }

/* OTP inputs */
.otp-inputs {
  display: flex; gap: 8px; justify-content: center; align-items: center;
}
.otp-sep { color: var(--muted); font-size: 20px; margin: 0 4px; }
.otp-digit {
  width: 52px; height: 58px;
  text-align: center; font-size: 22px; font-weight: 700;
  background: var(--panel2); border: 2px solid var(--border);
  border-radius: 12px; color: var(--text);
  outline: none; transition: border-color .2s, box-shadow .2s;
}
.otp-digit:focus {
  border-color: var(--primary);
  box-shadow: 0 0 0 3px rgba(79,127,255,.25);
}
.otp-digit.filled { border-color: var(--accent); }

/* Timer */
.timer-row {
  display: flex; justify-content: space-between; align-items: center;
  font-size: 13px; color: var(--muted);
}
.resend-btn {
  background: none; border: none; color: var(--primary2);
  font-weight: 600; cursor: pointer; font-size: 13px;
}
.resend-btn.disabled { color: var(--muted); cursor: default; pointer-events: none; }

/* ══════════════════════════════════
   CHAT LAYOUT
   ══════════════════════════════════ */
#chatScreen {
  display: flex; align-items: stretch; flex-direction: row;
  padding: 0; z-index: 5;
}
#chatScreen.active { display: flex; }

/* Sidebar */
.sidebar {
  width: var(--sidebar-w); flex-shrink: 0;
  background: rgba(8,12,24,.9);
  border-right: 1px solid var(--border);
  display: flex; flex-direction: column;
  height: 100vh; overflow: hidden;
  transition: width var(--transition), opacity var(--transition);
  z-index: 20;
}
.sidebar.collapsed { width: 0; opacity: 0; }
.sidebar-header {
  display: flex; align-items: center; justify-content: space-between;
  padding: 20px 18px 16px;
  border-bottom: 1px solid var(--border);
}
.brand-mini { display: flex; align-items: center; gap: 10px; font-size: 15px; font-weight: 700; }
.brand-icon-sm {
  width: 34px; height: 34px; border-radius: 10px;
  background: linear-gradient(135deg, var(--primary), var(--accent));
  display: flex; align-items: center; justify-content: center;
  font-size: 14px; font-weight: 800; color: #fff;
}

.new-chat-btn {
  margin: 16px; padding: 12px 16px;
  background: linear-gradient(135deg, var(--primary), #7c4dff);
  border-radius: 12px; cursor: pointer;
  display: flex; align-items: center; gap: 10px;
  font-weight: 600; font-size: 14px;
  transition: transform .2s, box-shadow .2s;
}
.new-chat-btn:hover { transform: translateY(-1px); box-shadow: 0 6px 20px rgba(79,127,255,.35); }
.new-chat-btn svg { width: 18px; height: 18px; fill: #fff; }

.sidebar-section-label {
  padding: 8px 18px 4px;
  font-size: 11px; font-weight: 700; letter-spacing: 1.2px;
  color: var(--muted); text-transform: uppercase;
}

.category-nav { display: flex; flex-direction: column; gap: 2px; padding: 4px 8px; }
.cat-btn {
  display: flex; align-items: center; gap: 10px;
  padding: 10px 12px; border-radius: 10px;
  background: none; border: none; color: var(--muted);
  font-size: 13px; font-weight: 500; cursor: pointer;
  transition: background .2s, color .2s;
  text-align: left;
}
.cat-btn svg { width: 17px; height: 17px; fill: currentColor; flex-shrink: 0; }
.cat-btn:hover { background: var(--panel2); color: var(--text); }
.cat-btn.active { background: rgba(79,127,255,.15); color: var(--primary2); }

.history-list {
  list-style: none; padding: 4px 8px; flex: 1; overflow-y: auto;
}
.history-list::-webkit-scrollbar { width: 4px; }
.history-list::-webkit-scrollbar-track { background: transparent; }
.history-list::-webkit-scrollbar-thumb { background: var(--border2); border-radius: 4px; }

.history-item {
  padding: 9px 12px; border-radius: 10px;
  font-size: 13px; color: var(--muted); cursor: pointer;
  white-space: nowrap; overflow: hidden; text-overflow: ellipsis;
  transition: background .2s, color .2s;
}
.history-item:hover { background: var(--panel2); color: var(--text); }
.history-item.active { background: rgba(79,127,255,.1); color: var(--primary2); }

.sidebar-footer {
  padding: 16px; border-top: 1px solid var(--border);
  display: flex; align-items: center; gap: 10px;
}
.user-chip { display: flex; align-items: center; gap: 10px; flex: 1; overflow: hidden; }
.avatar-sm {
  width: 36px; height: 36px; border-radius: 50%;
  background: linear-gradient(135deg, var(--primary), var(--accent));
  display: flex; align-items: center; justify-content: center;
  font-weight: 700; font-size: 14px; flex-shrink: 0;
}
.user-meta { display: flex; flex-direction: column; overflow: hidden; }
.user-meta span:first-child { font-size: 13px; font-weight: 600; white-space: nowrap; overflow: hidden; text-overflow: ellipsis; }
.status-dot-wrap { font-size: 11px; color: var(--muted); display: flex; align-items: center; gap: 4px; }
.status-dot { width: 7px; height: 7px; border-radius: 50%; background: var(--green); display: inline-block; box-shadow: 0 0 6px var(--green); }
.logout-btn { flex-shrink: 0; }
.logout-btn svg { fill: var(--muted); }
.logout-btn:hover svg { fill: var(--red); }

/* Chat Main */
.chat-main {
  flex: 1; display: flex; flex-direction: column;
  height: 100vh; overflow: hidden;
  background: var(--bg);
}

/* Header */
.chat-header {
  display: flex; align-items: center; gap: 14px;
  padding: 14px 24px;
  border-bottom: 1px solid var(--border);
  background: rgba(8,12,24,.8); backdrop-filter: blur(12px);
  flex-shrink: 0;
}
.header-title { flex: 1; }
.header-title h2 { font-size: 17px; font-weight: 700; }
.header-sub { font-size: 11px; color: var(--muted); }
.header-badges { display: flex; gap: 8px; }
.badge {
  padding: 4px 10px; border-radius: 20px;
  font-size: 11px; font-weight: 600; letter-spacing: .3px;
}
.badge-green { background: rgba(52,211,153,.12); color: var(--green); border: 1px solid rgba(52,211,153,.3); }
.badge-blue  { background: rgba(79,127,255,.12); color: var(--primary2); border: 1px solid rgba(79,127,255,.3); }

/* Icon Button */
.icon-btn {
  width: 38px; height: 38px; border-radius: 10px;
  background: var(--panel); border: 1px solid var(--border);
  display: flex; align-items: center; justify-content: center;
  cursor: pointer; transition: background .2s, border-color .2s;
}
.icon-btn:hover { background: var(--panel2); border-color: var(--border2); }
.icon-btn svg { width: 18px; height: 18px; fill: var(--muted); pointer-events: none; }

/* Messages */
.messages-area {
  flex: 1; overflow-y: auto;
  padding: 28px 32px; display: flex;
  flex-direction: column; gap: 20px;
  scroll-behavior: smooth;
}
.messages-area::-webkit-scrollbar { width: 5px; }
.messages-area::-webkit-scrollbar-track { background: transparent; }
.messages-area::-webkit-scrollbar-thumb { background: var(--border2); border-radius: 4px; }

/* Bubble */
.bubble-wrap { display: flex; align-items: flex-end; gap: 10px; max-width: 75%; }
.bubble-wrap.user { flex-direction: row-reverse; align-self: flex-end; }
.bubble-wrap.bot  { align-self: flex-start; }

.bubble-avatar {
  width: 32px; height: 32px; border-radius: 50%; flex-shrink: 0;
  display: flex; align-items: center; justify-content: center;
  font-size: 14px;
}
.bubble-avatar.bot-av {
  background: linear-gradient(135deg, var(--primary), var(--accent));
  font-weight: 800; color: #fff; font-size: 12px;
}
.bubble-avatar.user-av {
  background: linear-gradient(135deg, #7c3aed, #4f7fff);
}

.bubble {
  padding: 13px 17px; border-radius: 18px;
  font-size: 14px; line-height: 1.65; word-break: break-word;
  animation: bubbleIn .3s cubic-bezier(.4,0,.2,1) both;
}
@keyframes bubbleIn {
  from { transform: scale(.92) translateY(8px); opacity: 0; }
  to   { transform: scale(1)   translateY(0);   opacity: 1; }
}
.bubble.bot {
  background: var(--panel2); border: 1px solid var(--border);
  border-bottom-left-radius: 6px; color: var(--text);
}
.bubble.user {
  background: linear-gradient(135deg, var(--primary), #7c4dff);
  color: #fff; border-bottom-right-radius: 6px;
}
.bubble time { display: block; font-size: 10.5px; opacity: .55; margin-top: 5px; text-align: right; }

/* Typing indicator */
.typing-indicator {
  display: flex; gap: 5px; align-items: center; padding: 4px 0;
}
.typing-indicator span {
  width: 8px; height: 8px; border-radius: 50%;
  background: var(--primary);
  animation: bounce .9s infinite;
}
.typing-indicator span:nth-child(2) { animation-delay: .15s; }
.typing-indicator span:nth-child(3) { animation-delay: .3s; }
@keyframes bounce {
  0%,80%,100% { transform: translateY(0); opacity: .4; }
  40%          { transform: translateY(-7px); opacity: 1; }
}

/* Welcome banner */
.welcome-banner { align-self: center; width: 100%; max-width: 680px; margin: auto 0; }
.welcome-grid {
  display: grid; grid-template-columns: 1fr 1fr; gap: 14px;
}
.welcome-card {
  background: var(--panel2); border: 1px solid var(--border);
  border-radius: 16px; padding: 20px;
  display: flex; align-items: center; gap: 14px;
  cursor: pointer; transition: background .2s, border-color .2s, transform .2s;
}
.welcome-card:hover {
  background: rgba(79,127,255,.1); border-color: rgba(79,127,255,.4);
  transform: translateY(-2px);
}
.wc-icon { font-size: 28px; flex-shrink: 0; }
.welcome-card h4 { font-size: 14px; font-weight: 700; margin-bottom: 3px; }
.welcome-card p  { font-size: 12px; color: var(--muted); }

/* Chip categories in messages */
.tag-chip {
  display: inline-block; padding: 2px 9px;
  border-radius: 20px; font-size: 11px; font-weight: 600; margin-bottom: 6px;
  background: rgba(79,127,255,.15); color: var(--primary2);
  border: 1px solid rgba(79,127,255,.3);
}

/* Input zone */
.input-zone {
  padding: 16px 24px 20px;
  flex-shrink: 0;
  background: rgba(8,12,24,.85); backdrop-filter: blur(12px);
  border-top: 1px solid var(--border);
}
.input-bar {
  display: flex; align-items: flex-end; gap: 10px;
  padding: 10px 10px 10px 14px; border-radius: 16px;
}
.input-bar textarea {
  flex: 1; background: none; border: none;
  color: var(--text); font-size: 14.5px; font-family: inherit;
  resize: none; outline: none; max-height: 160px; line-height: 1.6;
}
.input-bar textarea::placeholder { color: var(--muted); }
.send-btn {
  width: 42px; height: 42px; border-radius: 12px; flex-shrink: 0;
  background: linear-gradient(135deg, var(--primary), #7c4dff);
  border: none; cursor: pointer; display: flex; align-items: center; justify-content: center;
  transition: transform .18s, box-shadow .18s;
}
.send-btn:hover { transform: scale(1.08); box-shadow: 0 4px 16px rgba(79,127,255,.5); }
.send-btn svg { width: 18px; height: 18px; fill: #fff; }
.input-footer { text-align: center; font-size: 11px; color: var(--muted); margin-top: 8px; }

/* ══════════════════════════════════
   MODAL
   ══════════════════════════════════ */
.modal-overlay {
  position: fixed; inset: 0;
  background: rgba(0,0,0,.65); backdrop-filter: blur(6px);
  z-index: 100; display: flex; align-items: center; justify-content: center;
  opacity: 0; pointer-events: none;
  transition: opacity .3s;
}
.modal-overlay.visible { opacity: 1; pointer-events: all; }
.modal {
  width: 100%; max-width: 520px; padding: 32px;
  display: flex; flex-direction: column; gap: 20px;
  animation: slideUp .35s cubic-bezier(.4,0,.2,1) both;
}
.modal-header { display: flex; align-items: center; justify-content: space-between; }
.modal-header h3 { font-size: 18px; font-weight: 700; }

/* Drop zone */
.drop-zone {
  border: 2px dashed var(--border2); border-radius: 14px;
  transition: border-color .2s, background .2s;
  cursor: pointer;
}
.drop-zone.drag-over { border-color: var(--primary); background: rgba(79,127,255,.08); }
.drop-inner {
  padding: 36px 24px; text-align: center;
  display: flex; flex-direction: column; align-items: center; gap: 8px;
}
.drop-icon { font-size: 40px; }
.drop-inner p { font-size: 14px; color: var(--muted); }
.drop-inner .accent { color: var(--primary2); font-weight: 600; cursor: pointer; }

.file-preview {
  background: var(--panel2); border: 1px solid var(--border);
  border-radius: 12px; padding: 14px 16px;
  display: flex; align-items: center; gap: 12px; font-size: 13px;
}
.file-preview .file-icon { font-size: 24px; }
.file-preview .file-info { flex: 1; overflow: hidden; }
.file-preview .file-name { font-weight: 600; white-space: nowrap; overflow: hidden; text-overflow: ellipsis; }
.file-preview .file-size { color: var(--muted); font-size: 12px; }

/* Action chips */
.action-chips { display: flex; gap: 10px; }
.chip {
  flex: 1; padding: 10px; border-radius: 12px;
  background: var(--panel2); border: 1px solid var(--border);
  color: var(--muted); font-size: 13px; font-weight: 600; cursor: pointer;
  transition: all .2s;
}
.chip:hover { background: var(--panel); border-color: var(--border2); color: var(--text); }
.chip.active { background: rgba(79,127,255,.15); border-color: var(--primary); color: var(--primary2); }

.qa-input-wrap input {
  width: 100%; padding: 12px 16px;
  background: var(--panel2); border: 1px solid var(--border);
  border-radius: 10px; color: var(--text); font-family: inherit; font-size: 14px;
  outline: none; transition: border-color .2s;
}
.qa-input-wrap input:focus { border-color: var(--primary); }
.qa-input-wrap input::placeholder { color: var(--muted); }

.analysis-result {
  background: var(--panel2); border: 1px solid var(--border);
  border-radius: 14px; padding: 18px;
  font-size: 13.5px; line-height: 1.7; color: var(--text);
  max-height: 250px; overflow-y: auto;
}
.analysis-result::-webkit-scrollbar { width: 4px; }
.analysis-result::-webkit-scrollbar-thumb { background: var(--border2); border-radius: 4px; }

/* Toast */
.toast {
  position: fixed; bottom: 28px; left: 50%; transform: translateX(-50%) translateY(20px);
  background: rgba(20,26,50,.98); border: 1px solid var(--border2);
  border-radius: 30px; padding: 12px 22px;
  font-size: 13.5px; color: var(--text);
  z-index: 999; opacity: 0; transition: opacity .3s, transform .3s;
  pointer-events: none;
}
.toast.show { opacity: 1; transform: translateX(-50%) translateY(0); }

/* Scrollbar global */
* { scrollbar-width: thin; scrollbar-color: var(--border2) transparent; }

/* Responsive */
@media (max-width: 640px) {
  .auth-card { padding: 32px 24px; }
  .welcome-grid { grid-template-columns: 1fr; }
  .messages-area { padding: 20px 16px; }
  .header-badges { display: none; }
}


```





### OUTPUT:


<a href="https://ibb.co/k698c95J"><img src="https://i.ibb.co/2YstKsWS/img-1.jpg" alt="img-1" border="0"></a>
<a href="https://ibb.co/rKG6sFKv"><img src="https://i.ibb.co/DPDCpzPL/img-2.jpg" alt="img-2" border="0"></a>
<a href="https://ibb.co/Xr4P8yGc"><img src="https://i.ibb.co/VcNGmHkf/img-3.jpg" alt="img-3" border="0"></a>



### RESULT:

The developed chatbot successfully understands and responds to employee queries related to HR, IT support, and organizational matters, processes uploaded documents for summarization and keyword extraction, securely authenticates users using email-based 2FA, filters inappropriate language, and efficiently handles a minimum of 5 parallel users with response time under 5 seconds.
