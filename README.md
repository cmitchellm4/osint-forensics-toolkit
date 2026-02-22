# osint-forensics-toolkit
osint-forensics-toolkit
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8"/>
  <meta name="viewport" content="width=device-width, initial-scale=1.0"/>
  <title>OSINT & Forensics Toolkit</title>
  <link href="https://fonts.googleapis.com/css2?family=VT323&family=Courier+Prime:wght@400;700&display=swap" rel="stylesheet"/>
  <style>
    :root {
      --bg:       #010b04;
      --surface:  #030f06;
      --border:   #0d2914;
      --green:    #00ff41;
      --green2:   #00cc33;
      --green3:   #008f11;
      --dim:      #1a4a22;
      --dimtext:  #2d7a3a;
      --text:     #a0ffb0;
      --white:    #e0ffe8;
      --amber:    #ffa500;
      --red:      #ff2244;
      --mono:     'Courier Prime', monospace;
      --vt:       'VT323', monospace;
    }

    *, *::before, *::after { box-sizing: border-box; margin: 0; padding: 0; }

    html { scroll-behavior: smooth; }

    body {
      background: var(--bg);
      color: var(--text);
      font-family: var(--mono);
      min-height: 100vh;
      overflow-x: hidden;
      cursor: crosshair;
    }

    /* ── Matrix rain canvas ─────────────────────────────────────────────── */
    #matrix-canvas {
      position: fixed;
      top: 0; left: 0;
      width: 100%; height: 100%;
      pointer-events: none;
      z-index: 0;
      opacity: 0.07;
    }

    /* ── Scanlines ──────────────────────────────────────────────────────── */
    body::after {
      content: '';
      position: fixed;
      inset: 0;
      background: repeating-linear-gradient(
        0deg,
        transparent,
        transparent 3px,
        rgba(0, 255, 65, 0.012) 3px,
        rgba(0, 255, 65, 0.012) 4px
      );
      pointer-events: none;
      z-index: 9998;
    }

    /* ── CRT vignette ───────────────────────────────────────────────────── */
    body::before {
      content: '';
      position: fixed;
      inset: 0;
      background: radial-gradient(ellipse at center, transparent 60%, rgba(0,0,0,0.85) 100%);
      pointer-events: none;
      z-index: 9997;
    }

    .page-wrap {
      position: relative;
      z-index: 1;
      max-width: 1200px;
      margin: 0 auto;
      padding: 0 32px 80px;
    }

    /* ── Boot sequence ──────────────────────────────────────────────────── */
    #boot-screen {
      position: fixed;
      inset: 0;
      background: var(--bg);
      z-index: 9999;
      display: flex;
      flex-direction: column;
      justify-content: center;
      align-items: flex-start;
      padding: 64px;
      font-family: var(--vt);
      font-size: 22px;
      color: var(--green);
    }

    .boot-line {
      display: block;
      opacity: 0;
      animation: bootFade 0.1s forwards;
      letter-spacing: 1px;
      line-height: 1.6;
    }

    @keyframes bootFade { to { opacity: 1; } }

    .boot-cursor {
      display: inline-block;
      width: 12px;
      height: 18px;
      background: var(--green);
      animation: blink 0.7s step-end infinite;
      vertical-align: middle;
      margin-left: 4px;
    }

    @keyframes blink { 0%,100%{opacity:1} 50%{opacity:0} }

    /* ── Navigation ─────────────────────────────────────────────────────── */
    .nav {
      position: sticky;
      top: 0;
      z-index: 100;
      background: rgba(1, 11, 4, 0.95);
      border-bottom: 1px solid var(--border);
      backdrop-filter: blur(4px);
      padding: 0 32px;
      display: flex;
      align-items: center;
      height: 52px;
      gap: 0;
    }

    .nav-brand {
      font-family: var(--vt);
      font-size: 22px;
      color: var(--green);
      letter-spacing: 2px;
      margin-right: auto;
      text-shadow: 0 0 10px var(--green3);
    }

    .nav-brand::before { content: "> "; color: var(--green3); }

    .nav-links {
      display: flex;
      gap: 0;
    }

    .nav-link {
      font-size: 11px;
      letter-spacing: 2px;
      text-transform: uppercase;
      color: var(--dimtext);
      text-decoration: none;
      padding: 0 20px;
      height: 52px;
      display: flex;
      align-items: center;
      border-left: 1px solid var(--border);
      transition: all 0.15s;
    }

    .nav-link:hover {
      color: var(--green);
      background: rgba(0,255,65,0.04);
    }

    .nav-status {
      display: flex;
      align-items: center;
      gap: 8px;
      font-size: 10px;
      letter-spacing: 1px;
      color: var(--green3);
      border-left: 1px solid var(--border);
      padding-left: 20px;
      margin-left: 8px;
    }

    .status-dot {
      width: 6px; height: 6px;
      border-radius: 50%;
      background: var(--green);
      box-shadow: 0 0 6px var(--green);
      animation: pulse 2s ease-in-out infinite;
    }
    @keyframes pulse { 0%,100%{opacity:1;box-shadow:0 0 6px var(--green)} 50%{opacity:0.5;box-shadow:0 0 2px var(--green3)} }

    /* ── Hero ────────────────────────────────────────────────────────────── */
    .hero {
      padding: 80px 0 64px;
      border-bottom: 1px solid var(--border);
      position: relative;
    }

    .hero-eyebrow {
      font-family: var(--vt);
      font-size: 16px;
      letter-spacing: 4px;
      color: var(--green3);
      margin-bottom: 16px;
    }

    .hero-eyebrow::before { content: "// "; }

    .hero-title {
      font-family: var(--vt);
      font-size: clamp(52px, 8vw, 96px);
      color: var(--white);
      line-height: 1;
      letter-spacing: 4px;
      text-shadow: 0 0 40px rgba(0,255,65,0.3), 0 0 80px rgba(0,255,65,0.1);
      margin-bottom: 8px;
    }

    .hero-title .accent { color: var(--green); text-shadow: 0 0 20px var(--green), 0 0 40px var(--green3); }

    .hero-subtitle {
      font-family: var(--vt);
      font-size: clamp(24px, 4vw, 42px);
      color: var(--green3);
      letter-spacing: 3px;
      margin-bottom: 32px;
    }

    .hero-desc {
      font-size: 13px;
      color: var(--dimtext);
      line-height: 1.9;
      max-width: 580px;
      letter-spacing: 0.5px;
    }

    .hero-desc::before {
      content: "$ cat README.txt\n";
      display: block;
      color: var(--green3);
      margin-bottom: 8px;
      font-size: 12px;
    }

    .hero-stats {
      display: flex;
      gap: 0;
      margin-top: 48px;
      border: 1px solid var(--border);
      width: fit-content;
    }

    .hero-stat {
      padding: 16px 32px;
      border-right: 1px solid var(--border);
      text-align: center;
    }

    .hero-stat:last-child { border-right: none; }

    .stat-num {
      font-family: var(--vt);
      font-size: 36px;
      color: var(--green);
      display: block;
      text-shadow: 0 0 10px var(--green3);
    }

    .stat-label {
      font-size: 9px;
      letter-spacing: 2px;
      text-transform: uppercase;
      color: var(--dimtext);
      display: block;
      margin-top: 4px;
    }

    /* ── Section ─────────────────────────────────────────────────────────── */
    .section {
      padding: 64px 0;
      border-bottom: 1px solid var(--border);
    }

    .section-header {
      display: flex;
      align-items: baseline;
      gap: 16px;
      margin-bottom: 40px;
    }

    .section-tag {
      font-family: var(--vt);
      font-size: 13px;
      letter-spacing: 3px;
      color: var(--green3);
    }

    .section-title {
      font-family: var(--vt);
      font-size: 32px;
      color: var(--white);
      letter-spacing: 3px;
    }

    .section-line {
      flex: 1;
      height: 1px;
      background: var(--border);
    }

    /* ── Tool cards ──────────────────────────────────────────────────────── */
    .tools-grid {
      display: grid;
      grid-template-columns: repeat(auto-fill, minmax(340px, 1fr));
      gap: 16px;
    }

    .tool-card {
      background: var(--surface);
      border: 1px solid var(--border);
      padding: 0;
      position: relative;
      overflow: hidden;
      transition: all 0.2s;
      animation: cardReveal 0.5s ease both;
    }

    @keyframes cardReveal {
      from { opacity: 0; transform: translateY(16px); }
      to   { opacity: 1; transform: translateY(0); }
    }

    .tool-card:nth-child(1) { animation-delay: 0.1s; }
    .tool-card:nth-child(2) { animation-delay: 0.15s; }
    .tool-card:nth-child(3) { animation-delay: 0.2s; }
    .tool-card:nth-child(4) { animation-delay: 0.25s; }
    .tool-card:nth-child(5) { animation-delay: 0.3s; }

    .tool-card::before {
      content: '';
      position: absolute;
      top: 0; left: 0; right: 0;
      height: 1px;
      background: linear-gradient(90deg, transparent, var(--green), transparent);
      opacity: 0;
      transition: opacity 0.3s;
    }

    .tool-card:hover {
      border-color: var(--green3);
      box-shadow: 0 0 30px rgba(0,255,65,0.06), inset 0 0 30px rgba(0,255,65,0.02);
      transform: translateY(-3px);
    }

    .tool-card:hover::before { opacity: 1; }

    .card-terminal-bar {
      background: var(--border);
      padding: 8px 16px;
      display: flex;
      align-items: center;
      gap: 8px;
      border-bottom: 1px solid var(--dim);
    }

    .terminal-dot {
      width: 8px; height: 8px;
      border-radius: 50%;
    }

    .td-red    { background: #ff5f57; }
    .td-yellow { background: #febc2e; }
    .td-green  { background: #28c840; }

    .terminal-title {
      font-size: 10px;
      letter-spacing: 2px;
      color: var(--dimtext);
      margin-left: 4px;
      flex: 1;
      text-align: center;
    }

    .card-body { padding: 24px; }

    .card-icon {
      font-size: 28px;
      display: block;
      margin-bottom: 12px;
      filter: drop-shadow(0 0 8px rgba(0,255,65,0.3));
    }

    .card-num {
      font-family: var(--vt);
      font-size: 13px;
      color: var(--green3);
      letter-spacing: 3px;
      margin-bottom: 4px;
    }

    .card-title {
      font-family: var(--vt);
      font-size: 26px;
      color: var(--white);
      letter-spacing: 2px;
      margin-bottom: 12px;
    }

    .card-desc {
      font-size: 12px;
      color: var(--dimtext);
      line-height: 1.8;
      margin-bottom: 20px;
    }

    .card-tags {
      display: flex;
      flex-wrap: wrap;
      gap: 6px;
      margin-bottom: 20px;
    }

    .tag {
      font-size: 9px;
      letter-spacing: 1.5px;
      text-transform: uppercase;
      padding: 3px 10px;
      border: 1px solid var(--dim);
      color: var(--green3);
      background: rgba(0,255,65,0.03);
    }

    .card-links {
      display: flex;
      gap: 10px;
      padding-top: 16px;
      border-top: 1px solid var(--border);
    }

    .card-link {
      font-size: 10px;
      letter-spacing: 2px;
      text-transform: uppercase;
      text-decoration: none;
      padding: 7px 16px;
      transition: all 0.15s;
    }

    .link-primary {
      background: rgba(0,255,65,0.08);
      border: 1px solid var(--green3);
      color: var(--green);
    }

    .link-primary:hover {
      background: rgba(0,255,65,0.15);
      box-shadow: 0 0 12px rgba(0,255,65,0.2);
    }

    .link-ghost {
      border: 1px solid var(--border);
      color: var(--dimtext);
    }

    .link-ghost:hover { border-color: var(--dimtext); color: var(--text); }

    /* ── Terminal window (about section) ────────────────────────────────── */
    .terminal-window {
      border: 1px solid var(--border);
      background: var(--surface);
    }

    .terminal-titlebar {
      background: var(--border);
      padding: 10px 16px;
      display: flex;
      align-items: center;
      gap: 8px;
      border-bottom: 1px solid var(--dim);
    }

    .terminal-body {
      padding: 32px;
      font-family: var(--vt);
      font-size: 18px;
      line-height: 2;
      color: var(--text);
    }

    .t-prompt { color: var(--green3); }
    .t-cmd    { color: var(--green); }
    .t-out    { color: var(--dimtext); font-size: 16px; }
    .t-comment{ color: var(--dim); }
    .t-string { color: var(--amber); }
    .t-key    { color: var(--text); }
    .t-cursor-line::after {
      content: '█';
      color: var(--green);
      animation: blink 0.7s step-end infinite;
    }

    /* ── Skills section ──────────────────────────────────────────────────── */
    .skills-grid {
      display: grid;
      grid-template-columns: repeat(3, 1fr);
      gap: 16px;
    }

    .skill-item {
      border: 1px solid var(--border);
      padding: 20px;
      background: var(--surface);
      position: relative;
      overflow: hidden;
    }

    .skill-label {
      font-size: 10px;
      letter-spacing: 2px;
      text-transform: uppercase;
      color: var(--dimtext);
      margin-bottom: 10px;
    }

    .skill-bar-wrap {
      height: 3px;
      background: var(--dim);
      position: relative;
      overflow: hidden;
    }

    .skill-bar {
      height: 100%;
      background: linear-gradient(90deg, var(--green3), var(--green));
      box-shadow: 0 0 8px var(--green3);
      animation: growBar 1.5s ease both;
      transform-origin: left;
    }

    @keyframes growBar { from { width: 0 !important; } }

    .skill-pct {
      font-family: var(--vt);
      font-size: 20px;
      color: var(--green);
      margin-top: 6px;
      display: block;
    }

    /* ── Contact / footer ────────────────────────────────────────────────── */
    .contact-section {
      padding: 64px 0 0;
    }

    .contact-grid {
      display: grid;
      grid-template-columns: 1fr 1fr;
      gap: 32px;
    }

    .contact-block {
      border: 1px solid var(--border);
      padding: 28px;
      background: var(--surface);
    }

    .contact-label {
      font-size: 9px;
      letter-spacing: 3px;
      text-transform: uppercase;
      color: var(--green3);
      margin-bottom: 12px;
    }

    .contact-value {
      font-family: var(--vt);
      font-size: 22px;
      color: var(--white);
      letter-spacing: 1px;
    }

    .contact-value a { color: var(--green); text-decoration: none; }
    .contact-value a:hover { text-shadow: 0 0 8px var(--green); }

    .footer {
      margin-top: 48px;
      padding: 24px 0;
      border-top: 1px solid var(--border);
      display: flex;
      justify-content: space-between;
      align-items: center;
      font-size: 10px;
      letter-spacing: 2px;
      color: var(--dimtext);
    }

    .footer-brand { font-family: var(--vt); font-size: 16px; color: var(--green3); }

    /* ── Typewriter effect ───────────────────────────────────────────────── */
    .typewriter {
      overflow: hidden;
      border-right: 2px solid var(--green);
      white-space: nowrap;
      animation: typing 2.5s steps(40) 0.5s both, blinkCaret 0.75s step-end infinite;
      display: inline-block;
      max-width: 100%;
    }

    @keyframes typing { from { width: 0; } to { width: 100%; } }
    @keyframes blinkCaret { 0%,100%{border-color:var(--green)} 50%{border-color:transparent} }

    /* ── Glitch effect ───────────────────────────────────────────────────── */
    .glitch {
      position: relative;
    }
    .glitch::before, .glitch::after {
      content: attr(data-text);
      position: absolute;
      top: 0; left: 0;
      width: 100%;
    }
    .glitch::before {
      color: #0ff;
      animation: glitch1 4s infinite;
      clip-path: polygon(0 0, 100% 0, 100% 35%, 0 35%);
    }
    .glitch::after {
      color: #f0f;
      animation: glitch2 4s infinite;
      clip-path: polygon(0 65%, 100% 65%, 100% 100%, 0 100%);
    }
    @keyframes glitch1 {
      0%,90%,100% { transform: translate(0); opacity: 0; }
      92% { transform: translate(-2px, 1px); opacity: 0.8; }
      94% { transform: translate(2px, -1px); opacity: 0.8; }
    }
    @keyframes glitch2 {
      0%,90%,100% { transform: translate(0); opacity: 0; }
      93% { transform: translate(2px, 1px); opacity: 0.8; }
      95% { transform: translate(-2px, -1px); opacity: 0.8; }
    }

    /* ── Hex grid decoration ─────────────────────────────────────────────── */
    .hex-grid {
      position: absolute;
      right: -20px;
      top: 40px;
      opacity: 0.06;
      font-family: var(--vt);
      font-size: 11px;
      color: var(--green);
      line-height: 1.4;
      letter-spacing: 2px;
      pointer-events: none;
      user-select: none;
    }

    @media (max-width: 768px) {
      .skills-grid { grid-template-columns: 1fr 1fr; }
      .contact-grid { grid-template-columns: 1fr; }
      .tools-grid   { grid-template-columns: 1fr; }
      .hero-stats   { flex-wrap: wrap; }
      .hero-stat    { flex: 1; min-width: 100px; }
      .nav-links    { display: none; }
      .hex-grid     { display: none; }
      #boot-screen  { padding: 32px; font-size: 16px; }
    }
  </style>
</head>
<body>

<!-- Matrix rain -->
<canvas id="matrix-canvas"></canvas>

<!-- Boot screen -->
<div id="boot-screen">
  <div id="boot-lines"></div>
</div>

<!-- Navigation -->
<nav class="nav">
  <div class="nav-brand">OSINT_TOOLKIT</div>
  <div class="nav-links">
    <a class="nav-link" href="#tools">Tools</a>
    <a class="nav-link" href="#about">About</a>
    <a class="nav-link" href="#skills">Skills</a>
    <a class="nav-link" href="#contact">Contact</a>
  </div>
  <div class="nav-status">
    <span class="status-dot"></span>
    <span id="live-time"></span>
  </div>
</nav>

<!-- Hero -->
<div class="page-wrap">
  <section class="hero">
    <div class="hex-grid" id="hex-grid"></div>
    <div class="hero-eyebrow">OSINT &amp; DIGITAL FORENSICS</div>
    <h1 class="hero-title glitch" data-text="FORENSICS">
      <span class="accent">FORENSICS</span>
    </h1>
    <div class="hero-subtitle typewriter">TOOLKIT_v1.0 // OPEN SOURCE</div>
    <p class="hero-desc">
      A collection of open-source tools for digital investigations,
      forensic analysis, and OSINT research. Built from scratch
      to understand the techniques used by real security professionals.
    </p>
    <div class="hero-stats">
      <div class="hero-stat">
        <span class="stat-num" id="counter-tools">0</span>
        <span class="stat-label">Tools Built</span>
      </div>
      <div class="hero-stat">
        <span class="stat-num" id="counter-langs">0</span>
        <span class="stat-label">Languages</span>
      </div>
      <div class="hero-stat">
        <span class="stat-num" id="counter-apis">0</span>
        <span class="stat-label">Live APIs</span>
      </div>
      <div class="hero-stat">
        <span class="stat-num">100%</span>
        <span class="stat-label">Open Source</span>
      </div>
    </div>
  </section>

  <!-- Tools -->
  <section class="section" id="tools">
    <div class="section-header">
      <span class="section-tag">// 01</span>
      <h2 class="section-title">TOOLS</h2>
      <div class="section-line"></div>
    </div>

    <div class="tools-grid">

      <!-- Username Tracker -->
      <div class="tool-card">
        <div class="card-terminal-bar">
          <span class="terminal-dot td-red"></span>
          <span class="terminal-dot td-yellow"></span>
          <span class="terminal-dot td-green"></span>
          <span class="terminal-title">username_tracker.py</span>
        </div>
        <div class="card-body">
          <span class="card-icon">🔍</span>
          <div class="card-num">TOOL_01</div>
          <div class="card-title">USERNAME TRACKER</div>
          <div class="card-desc">
            Searches for a target username across 15+ platforms simultaneously.
            Reports where the username exists, flags false positives, and saves
            a timestamped investigation report.
          </div>
          <div class="card-tags">
            <span class="tag">Python</span>
            <span class="tag">OSINT</span>
            <span class="tag">HTTP</span>
            <span class="tag">Reconnaissance</span>
          </div>
          <div class="card-links">
            <a class="card-link link-primary" href="https://github.com/cmitchellm4/osint-username-tracker" target="_blank">↗ GitHub</a>
            <a class="card-link link-ghost" href="#">◉ View Tool</a>
          </div>
        </div>
      </div>

      <!-- EXIF Extractor -->
      <div class="tool-card">
        <div class="card-terminal-bar">
          <span class="terminal-dot td-red"></span>
          <span class="terminal-dot td-yellow"></span>
          <span class="terminal-dot td-green"></span>
          <span class="terminal-title">metadata_extractor.py — index.html</span>
        </div>
        <div class="card-body">
          <span class="card-icon">🔬</span>
          <div class="card-num">TOOL_02</div>
          <div class="card-title">EXIF EXTRACTOR</div>
          <div class="card-desc">
            Extracts hidden metadata embedded in image files — GPS coordinates,
            device model, timestamps, and camera settings. Converts GPS to
            Google Maps links. Web + CLI interfaces.
          </div>
          <div class="card-tags">
            <span class="tag">Python</span>
            <span class="tag">EXIF</span>
            <span class="tag">Geolocation</span>
            <span class="tag">Binary Parsing</span>
          </div>
          <div class="card-links">
            <a class="card-link link-primary" href="https://github.com/cmitchellm4/exif-metadata-extractor" target="_blank">↗ GitHub</a>
            <a class="card-link link-ghost" href="#">◉ View Tool</a>
          </div>
        </div>
      </div>

      <!-- Disk Forensics -->
      <div class="tool-card">
        <div class="card-terminal-bar">
          <span class="terminal-dot td-red"></span>
          <span class="terminal-dot td-yellow"></span>
          <span class="terminal-dot td-green"></span>
          <span class="terminal-title">disk_forensics.py — index.html</span>
        </div>
        <div class="card-body">
          <span class="card-icon">💾</span>
          <div class="card-num">TOOL_03</div>
          <div class="card-title">DISK FORENSICS</div>
          <div class="card-desc">
            Two-module toolkit: File Signature Analyzer reads magic bytes to
            detect disguised executables and mismatched extensions. File Tracer
            surfaces zero-byte remnants, hidden files, and deletion artifacts.
          </div>
          <div class="card-tags">
            <span class="tag">Python</span>
            <span class="tag">Magic Bytes</span>
            <span class="tag">File Analysis</span>
            <span class="tag">Forensics</span>
          </div>
          <div class="card-links">
            <a class="card-link link-primary" href="https://github.com/cmitchellm4/disk-forensics-toolkit" target="_blank">↗ GitHub</a>
            <a class="card-link link-ghost" href="#">◉ View Tool</a>
          </div>
        </div>
      </div>

      <!-- Email Analyzer -->
      <div class="tool-card">
        <div class="card-terminal-bar">
          <span class="terminal-dot td-red"></span>
          <span class="terminal-dot td-yellow"></span>
          <span class="terminal-dot td-green"></span>
          <span class="terminal-title">email_analyzer.py — index.html</span>
        </div>
        <div class="card-body">
          <span class="card-icon">📧</span>
          <div class="card-num">TOOL_04</div>
          <div class="card-title">EMAIL ANALYZER</div>
          <div class="card-desc">
            Parses raw email headers to reconstruct the full relay chain, detect
            spoofing via domain mismatches, verify SPF/DKIM/DMARC authentication,
            and geolocate originating IP addresses.
          </div>
          <div class="card-tags">
            <span class="tag">Python</span>
            <span class="tag">SMTP</span>
            <span class="tag">SPF/DKIM</span>
            <span class="tag">Phishing</span>
          </div>
          <div class="card-links">
            <a class="card-link link-primary" href="https://github.com/cmitchellm4/email-header-analyzer" target="_blank">↗ GitHub</a>
            <a class="card-link link-ghost" href="#">◉ View Tool</a>
          </div>
        </div>
      </div>

      <!-- Wayback Scraper -->
      <div class="tool-card">
        <div class="card-terminal-bar">
          <span class="terminal-dot td-red"></span>
          <span class="terminal-dot td-yellow"></span>
          <span class="terminal-dot td-green"></span>
          <span class="terminal-title">wayback_scraper.py — index.html</span>
        </div>
        <div class="card-body">
          <span class="card-icon">🕰️</span>
          <div class="card-num">TOOL_05</div>
          <div class="card-title">WAYBACK SCRAPER</div>
          <div class="card-desc">
            Queries the Internet Archive CDX API to browse snapshots of any URL,
            compare content between dates with a line-by-line diff, view cached
            pages, and download archived content locally.
          </div>
          <div class="card-tags">
            <span class="tag">Python</span>
            <span class="tag">Wayback API</span>
            <span class="tag">Diff</span>
            <span class="tag">Web Archive</span>
          </div>
          <div class="card-links">
            <a class="card-link link-primary" href="https://github.com/cmitchellm4/wayback-scraper" target="_blank">↗ GitHub</a>
            <a class="card-link link-ghost" href="#">◉ View Tool</a>
          </div>
        </div>
      </div>

    </div>
  </section>

  <!-- About -->
  <section class="section" id="about">
    <div class="section-header">
      <span class="section-tag">// 02</span>
      <h2 class="section-title">ABOUT</h2>
      <div class="section-line"></div>
    </div>
    <div class="terminal-window">
      <div class="terminal-titlebar">
        <span class="terminal-dot td-red"></span>
        <span class="terminal-dot td-yellow"></span>
        <span class="terminal-dot td-green"></span>
        <span style="font-size:10px;color:var(--dimtext);margin-left:8px;letter-spacing:2px">about.sh</span>
      </div>
      <div class="terminal-body">
        <div><span class="t-prompt">user@kali:~$ </span><span class="t-cmd">cat about.txt</span></div>
        <div class="t-out">&nbsp;</div>
        <div><span class="t-comment"># OSINT &amp; Forensics Toolkit</span></div>
        <div><span class="t-comment"># ================================</span></div>
        <div class="t-out">&nbsp;</div>
        <div><span class="t-key">focus    </span><span class="t-comment">:</span><span class="t-string"> "cybersecurity / digital forensics / osint"</span></div>
        <div><span class="t-key">goal     </span><span class="t-comment">:</span><span class="t-string"> "build real tools, learn real techniques"</span></div>
        <div><span class="t-key">stack    </span><span class="t-comment">:</span><span class="t-string"> "python · html · javascript · bash"</span></div>
        <div><span class="t-key">status   </span><span class="t-comment">:</span><span class="t-string"> "actively building"</span></div>
        <div class="t-out">&nbsp;</div>
        <div><span class="t-comment"># Each tool in this kit teaches a real skill:</span></div>
        <div><span class="t-comment"># binary file parsing, SMTP protocol internals,</span></div>
        <div><span class="t-comment"># HTTP header analysis, archive APIs, and more.</span></div>
        <div class="t-out">&nbsp;</div>
        <div><span class="t-prompt">user@kali:~$ </span><span class="t-cursor-line"></span></div>
      </div>
    </div>
  </section>

  <!-- Skills -->
  <section class="section" id="skills">
    <div class="section-header">
      <span class="section-tag">// 03</span>
      <h2 class="section-title">SKILLS</h2>
      <div class="section-line"></div>
    </div>
    <div class="skills-grid">
      <div class="skill-item">
        <div class="skill-label">Python</div>
        <div class="skill-bar-wrap"><div class="skill-bar" style="width:85%"></div></div>
        <span class="skill-pct">85%</span>
      </div>
      <div class="skill-item">
        <div class="skill-label">HTML / CSS / JS</div>
        <div class="skill-bar-wrap"><div class="skill-bar" style="width:75%"></div></div>
        <span class="skill-pct">75%</span>
      </div>
      <div class="skill-item">
        <div class="skill-label">Network Protocols</div>
        <div class="skill-bar-wrap"><div class="skill-bar" style="width:65%"></div></div>
        <span class="skill-pct">65%</span>
      </div>
      <div class="skill-item">
        <div class="skill-label">OSINT Techniques</div>
        <div class="skill-bar-wrap"><div class="skill-bar" style="width:80%"></div></div>
        <span class="skill-pct">80%</span>
      </div>
      <div class="skill-item">
        <div class="skill-label">Digital Forensics</div>
        <div class="skill-bar-wrap"><div class="skill-bar" style="width:70%"></div></div>
        <span class="skill-pct">70%</span>
      </div>
      <div class="skill-item">
        <div class="skill-label">Linux / Bash</div>
        <div class="skill-bar-wrap"><div class="skill-bar" style="width:72%"></div></div>
        <span class="skill-pct">72%</span>
      </div>
    </div>
  </section>

  <!-- Contact -->
  <section class="contact-section" id="contact">
    <div class="section-header">
      <span class="section-tag">// 04</span>
      <h2 class="section-title">CONTACT</h2>
      <div class="section-line"></div>
    </div>
    <div class="contact-grid">
      <div class="contact-block">
        <div class="contact-label">GitHub</div>
        <div class="contact-value"><a href="https://github.com/cmitchellm4" target="_blank">github.com/cmitchellm4</a></div>
      </div>
      <div class="contact-block">
        <div class="contact-label">Email</div>
        <div class="contact-value"><a href="mailto:cmitchellm4@gmail.com">cmitchellm4@gmail.com</a></div>
      </div>
      <div class="contact-block">
        <div class="contact-label">LinkedIn</div>
        <div class="contact-value"><a href="https://linkedin.com/in/chris-mitchell-536ba752" target="_blank">linkedin.com/in/chris-mitchell-536ba752</a></div>
      </div>
      <div class="contact-block">
        <div class="contact-label">Status</div>
        <div class="contact-value" style="color:var(--green)">
          <span class="status-dot" style="display:inline-block;margin-right:10px"></span>
          Open to opportunities
        </div>
      </div>
    </div>

    <div class="footer">
      <div class="footer-brand">> OSINT_TOOLKIT_v1.0</div>
      <div>// ALL TOOLS OPEN SOURCE · MIT LICENSE · <span id="year"></span></div>
    </div>
  </section>

</div><!-- end page-wrap -->

<script>
// ── Boot sequence ─────────────────────────────────────────────────────────
const bootLines = [
  "BIOS v2.4.1 // Initializing hardware...",
  "CPU: Intel Core i7 // RAM: 16GB // OK",
  "Loading GRUB bootloader...",
  "Mounting filesystem... [OK]",
  "Starting network interfaces... eth0 [UP]",
  "Launching OSINT_TOOLKIT v1.0...",
  "Loading modules: [username_tracker] [exif_extractor] [disk_forensics]",
  "Loading modules: [email_analyzer] [wayback_scraper]",
  "All systems nominal.",
  "Welcome.",
];

const bootEl = document.getElementById("boot-lines");
let bi = 0;
function showBootLine() {
  if (bi >= bootLines.length) {
    setTimeout(() => {
      document.getElementById("boot-screen").style.transition = "opacity 0.6s";
      document.getElementById("boot-screen").style.opacity = "0";
      setTimeout(() => document.getElementById("boot-screen").style.display = "none", 600);
    }, 400);
    return;
  }
  const span = document.createElement("span");
  span.className = "boot-line";
  span.style.animationDelay = "0s";
  const prefix = bi === bootLines.length - 1 ? ">>> " : bi < 3 ? "" : "[ OK ] ";
  const color = bi === bootLines.length - 1 ? "var(--green)" : bi < 3 ? "var(--dimtext)" : "var(--green3)";
  span.style.color = color;
  span.textContent = prefix + bootLines[bi];
  bootEl.appendChild(span);
  bi++;
  setTimeout(showBootLine, bi < 3 ? 80 : 120);
}
showBootLine();

// ── Matrix rain ───────────────────────────────────────────────────────────
const canvas = document.getElementById("matrix-canvas");
const ctx    = canvas.getContext("2d");
const chars  = "アイウエオカキクケコサシスセソタチツテトナニヌネノ0123456789ABCDEF<>/\\|{}[]";

function resizeCanvas() {
  canvas.width  = window.innerWidth;
  canvas.height = window.innerHeight;
}
resizeCanvas();
window.addEventListener("resize", resizeCanvas);

const fontSize = 14;
let columns = Math.floor(canvas.width / fontSize);
let drops   = new Array(columns).fill(1);

function drawMatrix() {
  ctx.fillStyle = "rgba(1, 11, 4, 0.05)";
  ctx.fillRect(0, 0, canvas.width, canvas.height);
  ctx.fillStyle = "#00ff41";
  ctx.font = fontSize + "px monospace";
  for (let i = 0; i < drops.length; i++) {
    const char = chars[Math.floor(Math.random() * chars.length)];
    ctx.fillStyle = drops[i] * fontSize < 50 ? "#00ff41" : "#008f11";
    ctx.fillText(char, i * fontSize, drops[i] * fontSize);
    if (drops[i] * fontSize > canvas.height && Math.random() > 0.975) drops[i] = 0;
    drops[i]++;
  }
}
setInterval(drawMatrix, 50);

// ── Live clock ────────────────────────────────────────────────────────────
function updateClock() {
  const now = new Date();
  const h = String(now.getHours()).padStart(2,"0");
  const m = String(now.getMinutes()).padStart(2,"0");
  const s = String(now.getSeconds()).padStart(2,"0");
  document.getElementById("live-time").textContent = `${h}:${m}:${s} UTC`;
}
setInterval(updateClock, 1000);
updateClock();

// ── Year ──────────────────────────────────────────────────────────────────
document.getElementById("year").textContent = new Date().getFullYear();

// ── Counters ──────────────────────────────────────────────────────────────
function animateCounter(el, target, duration = 1500) {
  let start = null;
  function step(ts) {
    if (!start) start = ts;
    const progress = Math.min((ts - start) / duration, 1);
    el.textContent = Math.floor(progress * target);
    if (progress < 1) requestAnimationFrame(step);
    else el.textContent = target;
  }
  requestAnimationFrame(step);
}

// Trigger when hero is visible
const heroObs = new IntersectionObserver((entries) => {
  if (entries[0].isIntersecting) {
    animateCounter(document.getElementById("counter-tools"), 5);
    animateCounter(document.getElementById("counter-langs"), 2);
    animateCounter(document.getElementById("counter-apis"),  3);
    heroObs.disconnect();
  }
}, { threshold: 0.3 });
heroObs.observe(document.querySelector(".hero-stats"));

// ── Hex decoration ────────────────────────────────────────────────────────
const hexEl = document.getElementById("hex-grid");
const hexLines = [];
for (let i = 0; i < 18; i++) {
  let line = "";
  for (let j = 0; j < 8; j++) {
    line += Math.floor(Math.random()*256).toString(16).padStart(2,"0").toUpperCase() + " ";
  }
  hexLines.push(line);
}
hexEl.innerHTML = hexLines.join("<br>");
</script>
</body>
</html>
