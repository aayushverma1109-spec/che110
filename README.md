<!DOCTYPE html>
<html lang="en">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1.0">
<title>Terra Memoria — Local Land Use Change Viewer</title>
<link href="https://fonts.googleapis.com/css2?family=Playfair+Display:ital,wght@0,700;1,400&family=DM+Mono:wght@300;400;500&family=Lora:ital,wght@0,400;0,600;1,400&display=swap" rel="stylesheet">
<style>
  :root {
    --soil: #2d1b0e;
    --clay: #6b3f1e;
    --sand: #c8a97a;
    --moss: #4a6741;
    --sage: #8aad7c;
    --sky: #7baec3;
    --parchment: #f5efe4;
    --paper: #faf7f2;
    --ink: #1a1208;
    --rust: #b85c2a;
    --gold: #d4a017;
    --mist: rgba(245,239,228,0.85);
  }

  *, *::before, *::after { box-sizing: border-box; margin: 0; padding: 0; }

  body {
    background: var(--paper);
    color: var(--ink);
    font-family: 'Lora', Georgia, serif;
    overflow-x: hidden;
  }

  /* ── NOISE OVERLAY ── */
  body::before {
    content: '';
    position: fixed; inset: 0;
    background-image: url("data:image/svg+xml,%3Csvg viewBox='0 0 200 200' xmlns='http://www.w3.org/2000/svg'%3E%3Cfilter id='n'%3E%3CfeTurbulence type='fractalNoise' baseFrequency='0.75' numOctaves='4' stitchTiles='stitch'/%3E%3C/filter%3E%3Crect width='100%25' height='100%25' filter='url(%23n)' opacity='0.04'/%3E%3C/svg%3E");
    pointer-events: none; z-index: 9999; opacity: 0.6;
  }

  /* ── HEADER ── */
  header {
    background: var(--soil);
    color: var(--parchment);
    padding: 2.5rem 4rem;
    position: relative;
    overflow: hidden;
    border-bottom: 4px solid var(--gold);
  }
  header::after {
    content: '';
    position: absolute; bottom: -1px; left: 0; right: 0;
    height: 6px;
    background: repeating-linear-gradient(90deg, var(--moss) 0 18px, var(--rust) 18px 22px, var(--sand) 22px 40px, var(--soil) 40px 44px);
  }
  .header-inner {
    max-width: 1200px; margin: 0 auto;
    display: flex; align-items: flex-end; justify-content: space-between; gap: 2rem; flex-wrap: wrap;
  }
  .logo-block {}
  .logo-eyebrow {
    font-family: 'DM Mono', monospace;
    font-size: 0.68rem; letter-spacing: 0.25em;
    color: var(--sand); text-transform: uppercase; margin-bottom: 0.5rem;
  }
  h1 {
    font-family: 'Playfair Display', serif;
    font-size: clamp(2rem, 4vw, 3.2rem); font-weight: 700;
    line-height: 1;
    color: var(--parchment);
  }
  h1 em { color: var(--gold); font-style: italic; }
  .header-desc {
    font-family: 'Lora', serif; font-style: italic;
    font-size: 0.95rem; color: var(--sand); margin-top: 0.5rem;
  }
  .header-stats {
    display: flex; gap: 2rem;
  }
  .stat {
    text-align: center;
    background: rgba(255,255,255,0.06);
    border: 1px solid rgba(200,169,122,0.25);
    border-radius: 4px; padding: 0.8rem 1.2rem;
  }
  .stat-num {
    font-family: 'Playfair Display', serif;
    font-size: 1.8rem; font-weight: 700; color: var(--gold);
    display: block;
  }
  .stat-label {
    font-family: 'DM Mono', monospace;
    font-size: 0.62rem; letter-spacing: 0.15em;
    color: var(--sand); text-transform: uppercase;
  }

  /* ── NAV TABS ── */
  nav {
    background: var(--parchment);
    border-bottom: 2px solid var(--sand);
    position: sticky; top: 0; z-index: 100;
    box-shadow: 0 2px 12px rgba(45,27,14,0.12);
  }
  .nav-inner {
    max-width: 1200px; margin: 0 auto;
    display: flex; gap: 0;
  }
  .tab-btn {
    font-family: 'DM Mono', monospace;
    font-size: 0.75rem; letter-spacing: 0.12em; text-transform: uppercase;
    background: none; border: none; cursor: pointer;
    color: var(--clay); padding: 1rem 2rem;
    border-bottom: 3px solid transparent;
    transition: all 0.2s; position: relative;
  }
  .tab-btn:hover { color: var(--soil); background: rgba(45,27,14,0.04); }
  .tab-btn.active {
    color: var(--soil); font-weight: 500;
    border-bottom-color: var(--rust);
    background: white;
  }
  .tab-badge {
    display: inline-block; background: var(--moss); color: white;
    font-size: 0.6rem; padding: 1px 5px; border-radius: 10px; margin-left: 6px;
    vertical-align: middle;
  }

  /* ── SECTIONS ── */
  .section { display: none; }
  .section.active { display: block; }

  /* ── GALLERY SECTION ── */
  .gallery-wrap {
    max-width: 1200px; margin: 0 auto; padding: 3rem 2rem;
  }
  .section-title {
    font-family: 'Playfair Display', serif;
    font-size: 1.6rem; color: var(--soil);
    margin-bottom: 0.4rem;
  }
  .section-sub {
    font-family: 'DM Mono', monospace;
    font-size: 0.72rem; color: var(--clay); letter-spacing: 0.1em;
    text-transform: uppercase; margin-bottom: 2rem;
  }

  /* Filter bar */
  .filter-bar {
    display: flex; gap: 0.6rem; flex-wrap: wrap; margin-bottom: 2rem;
    padding-bottom: 1.5rem; border-bottom: 1px solid var(--sand);
  }
  .filter-btn {
    font-family: 'DM Mono', monospace; font-size: 0.7rem;
    letter-spacing: 0.1em; text-transform: uppercase;
    border: 1.5px solid var(--sand); background: white;
    color: var(--clay); padding: 0.4rem 1rem; border-radius: 2px;
    cursor: pointer; transition: all 0.18s;
  }
  .filter-btn:hover, .filter-btn.active {
    background: var(--soil); color: var(--parchment); border-color: var(--soil);
  }

  /* Cards */
  .gallery-grid {
    display: grid;
    grid-template-columns: repeat(auto-fill, minmax(340px, 1fr));
    gap: 2rem;
  }
  .card {
    background: white;
    border: 1px solid #e0d8cc;
    border-radius: 4px;
    overflow: hidden;
    box-shadow: 0 2px 8px rgba(45,27,14,0.07);
    cursor: pointer;
    transition: transform 0.2s, box-shadow 0.2s;
  }
  .card:hover {
    transform: translateY(-4px);
    box-shadow: 0 10px 32px rgba(45,27,14,0.14);
  }
  .card-img-wrap {
    position: relative; height: 200px; overflow: hidden;
    background: #e0d8cc;
  }
  .card-img-wrap canvas { width: 100%; height: 100%; display: block; }
  .card-tag {
    position: absolute; top: 10px; right: 10px;
    font-family: 'DM Mono', monospace; font-size: 0.62rem;
    letter-spacing: 0.1em; text-transform: uppercase;
    background: var(--soil); color: var(--gold);
    padding: 3px 8px; border-radius: 2px;
  }
  .card-body { padding: 1.2rem 1.4rem; }
  .card-location {
    font-family: 'DM Mono', monospace; font-size: 0.68rem;
    color: var(--clay); letter-spacing: 0.12em; text-transform: uppercase;
    margin-bottom: 0.4rem;
  }
  .card-title {
    font-family: 'Playfair Display', serif;
    font-size: 1.05rem; color: var(--soil); margin-bottom: 0.5rem;
  }
  .card-desc {
    font-family: 'Lora', serif; font-size: 0.84rem;
    color: #5a4535; line-height: 1.6;
  }
  .card-meta {
    display: flex; gap: 1rem; margin-top: 0.8rem;
    padding-top: 0.8rem; border-top: 1px solid #e8e0d4;
    font-family: 'DM Mono', monospace; font-size: 0.65rem;
    color: var(--clay); text-transform: uppercase; letter-spacing: 0.08em;
  }
  .card-meta span::before { content: '· '; color: var(--sand); }

  /* ── COMPARE SECTION ── */
  .compare-wrap {
    max-width: 900px; margin: 0 auto; padding: 3rem 2rem;
  }
  .compare-select {
    display: flex; gap: 1rem; margin-bottom: 2rem; flex-wrap: wrap;
  }
  .compare-select select {
    flex: 1; min-width: 200px;
    font-family: 'DM Mono', monospace; font-size: 0.78rem;
    color: var(--soil); background: white;
    border: 1.5px solid var(--sand); border-radius: 3px;
    padding: 0.6rem 1rem; cursor: pointer;
    appearance: none;
    background-image: url("data:image/svg+xml,%3Csvg xmlns='http://www.w3.org/2000/svg' width='12' height='8' viewBox='0 0 12 8'%3E%3Cpath d='M1 1l5 5 5-5' stroke='%236b3f1e' fill='none' stroke-width='2'/%3E%3C/svg%3E");
    background-repeat: no-repeat; background-position: right 12px center;
    padding-right: 2rem;
  }

  /* Slider */
  .slider-container {
    position: relative; width: 100%; height: 480px;
    overflow: hidden; border-radius: 4px;
    border: 2px solid var(--sand);
    user-select: none;
    cursor: ew-resize;
    box-shadow: 0 8px 32px rgba(45,27,14,0.15);
  }
  .slider-after, .slider-before {
    position: absolute; inset: 0;
    width: 100%; height: 100%;
  }
  .slider-after canvas, .slider-before canvas {
    width: 100%; height: 100%; object-fit: cover; display: block;
  }
  .slider-before {
    clip-path: inset(0 50% 0 0);
  }
  .slider-divider {
    position: absolute; top: 0; bottom: 0;
    left: 50%; width: 3px;
    background: white;
    box-shadow: 0 0 12px rgba(0,0,0,0.3);
    z-index: 10;
    transform: translateX(-50%);
  }
  .slider-handle {
    position: absolute; top: 50%; left: 50%;
    transform: translate(-50%, -50%);
    width: 44px; height: 44px;
    background: white;
    border-radius: 50%;
    box-shadow: 0 2px 12px rgba(0,0,0,0.25);
    display: flex; align-items: center; justify-content: center;
    z-index: 11; cursor: ew-resize;
    border: 2px solid var(--sand);
  }
  .slider-handle svg { color: var(--clay); }
  .slider-label {
    position: absolute; top: 12px;
    font-family: 'DM Mono', monospace; font-size: 0.68rem;
    letter-spacing: 0.12em; text-transform: uppercase;
    padding: 4px 10px; border-radius: 2px; z-index: 12;
    pointer-events: none;
  }
  .label-before { left: 14px; background: rgba(45,27,14,0.85); color: var(--parchment); }
  .label-after  { right: 14px; background: rgba(74,103,65,0.9); color: white; }
  .compare-info {
    margin-top: 1.5rem;
    background: white; border: 1px solid #e0d8cc;
    border-radius: 4px; padding: 1.4rem 1.6rem;
  }
  .compare-info h3 {
    font-family: 'Playfair Display', serif; font-size: 1.2rem; color: var(--soil);
    margin-bottom: 0.5rem;
  }
  .compare-info p { font-size: 0.9rem; color: #5a4535; line-height: 1.7; }
  .changes-list {
    margin-top: 1rem; display: flex; gap: 0.6rem; flex-wrap: wrap;
  }
  .change-pill {
    font-family: 'DM Mono', monospace; font-size: 0.65rem;
    letter-spacing: 0.08em; text-transform: uppercase;
    padding: 3px 10px; border-radius: 12px; border: 1.5px solid;
  }
  .change-pill.loss { border-color: var(--rust); color: var(--rust); }
  .change-pill.gain { border-color: var(--moss); color: var(--moss); }
  .change-pill.neutral { border-color: var(--sky); color: #4a7a92; }

  /* ── TIMELINE ── */
  .timeline-wrap {
    max-width: 900px; margin: 0 auto; padding: 3rem 2rem;
  }
  .timeline {
    position: relative; padding-left: 2.5rem;
  }
  .timeline::before {
    content: ''; position: absolute; left: 7px; top: 0; bottom: 0;
    width: 2px; background: linear-gradient(to bottom, var(--sand), var(--moss));
  }
  .timeline-item {
    position: relative; margin-bottom: 2.5rem;
    opacity: 0; transform: translateX(-20px);
    animation: slideIn 0.5s forwards;
  }
  .timeline-item:nth-child(1) { animation-delay: 0.1s; }
  .timeline-item:nth-child(2) { animation-delay: 0.2s; }
  .timeline-item:nth-child(3) { animation-delay: 0.3s; }
  .timeline-item:nth-child(4) { animation-delay: 0.4s; }
  .timeline-item:nth-child(5) { animation-delay: 0.5s; }
  .timeline-item:nth-child(6) { animation-delay: 0.6s; }
  @keyframes slideIn {
    to { opacity: 1; transform: translateX(0); }
  }
  .timeline-dot {
    position: absolute; left: -2.05rem; top: 4px;
    width: 16px; height: 16px; border-radius: 50%;
    border: 2.5px solid var(--sand); background: white;
    z-index: 1;
  }
  .timeline-dot.highlight {
    background: var(--rust); border-color: var(--rust);
    box-shadow: 0 0 0 4px rgba(184,92,42,0.2);
  }
  .timeline-year {
    font-family: 'DM Mono', monospace; font-size: 0.7rem;
    color: var(--clay); text-transform: uppercase; letter-spacing: 0.15em;
    margin-bottom: 0.3rem;
  }
  .timeline-card {
    background: white; border: 1px solid #e0d8cc;
    border-radius: 4px; padding: 1.1rem 1.4rem;
    box-shadow: 0 2px 8px rgba(45,27,14,0.05);
  }
  .timeline-card h4 {
    font-family: 'Playfair Display', serif; font-size: 1rem;
    color: var(--soil); margin-bottom: 0.4rem;
  }
  .timeline-card p { font-size: 0.85rem; color: #5a4535; line-height: 1.65; }

  /* ── SUBMIT SECTION ── */
  .submit-wrap {
    max-width: 680px; margin: 0 auto; padding: 3rem 2rem;
  }
  .form-card {
    background: white; border: 1px solid #e0d8cc;
    border-radius: 4px; padding: 2.5rem;
    box-shadow: 0 4px 20px rgba(45,27,14,0.08);
  }
  .form-grid { display: grid; grid-template-columns: 1fr 1fr; gap: 1.2rem; }
  .form-group { display: flex; flex-direction: column; gap: 0.4rem; }
  .form-group.full { grid-column: 1 / -1; }
  label {
    font-family: 'DM Mono', monospace; font-size: 0.7rem;
    letter-spacing: 0.1em; text-transform: uppercase; color: var(--clay);
  }
  input[type="text"], input[type="email"], input[type="number"],
  select.form-select, textarea {
    font-family: 'Lora', serif; font-size: 0.9rem;
    color: var(--soil); background: var(--paper);
    border: 1.5px solid #d0c8bc; border-radius: 3px;
    padding: 0.65rem 0.9rem; transition: border-color 0.2s;
    outline: none; width: 100%;
  }
  input:focus, select:focus, textarea:focus {
    border-color: var(--clay); background: white;
  }
  textarea { min-height: 100px; resize: vertical; }
  select.form-select { appearance: none; cursor: pointer;
    background-image: url("data:image/svg+xml,%3Csvg xmlns='http://www.w3.org/2000/svg' width='12' height='8' viewBox='0 0 12 8'%3E%3Cpath d='M1 1l5 5 5-5' stroke='%236b3f1e' fill='none' stroke-width='2'/%3E%3C/svg%3E");
    background-repeat: no-repeat; background-position: right 12px center; padding-right: 2rem;
  }

  /* Drop zone */
  .dropzone {
    border: 2px dashed var(--sand); border-radius: 4px;
    padding: 2.5rem; text-align: center; cursor: pointer;
    transition: all 0.2s; background: var(--paper);
  }
  .dropzone:hover, .dropzone.drag-over {
    border-color: var(--clay); background: white;
  }
  .dropzone-icon { font-size: 2rem; margin-bottom: 0.6rem; }
  .dropzone-text {
    font-family: 'DM Mono', monospace; font-size: 0.73rem;
    color: var(--clay); text-transform: uppercase; letter-spacing: 0.1em;
  }
  .dropzone-sub { font-size: 0.7rem; color: #a09080; margin-top: 0.3rem; }
  .file-list { margin-top: 0.8rem; }
  .file-item {
    display: flex; align-items: center; gap: 0.6rem;
    font-family: 'DM Mono', monospace; font-size: 0.72rem;
    color: var(--moss); padding: 0.3rem 0;
  }
  .file-item::before { content: '✓ '; }

  .submit-btn {
    width: 100%; margin-top: 1.8rem;
    font-family: 'DM Mono', monospace; font-size: 0.8rem;
    letter-spacing: 0.15em; text-transform: uppercase;
    background: var(--soil); color: var(--gold);
    border: none; padding: 1rem; border-radius: 3px;
    cursor: pointer; transition: all 0.2s;
  }
  .submit-btn:hover { background: var(--clay); transform: translateY(-1px); }
  .submit-success {
    display: none; text-align: center; padding: 2rem;
  }
  .submit-success .check { font-size: 3rem; margin-bottom: 1rem; }
  .submit-success h3 { font-family: 'Playfair Display', serif; color: var(--moss); margin-bottom: 0.5rem; }
  .submit-success p { font-size: 0.9rem; color: #5a4535; }

  /* ── MODAL ── */
  .modal-overlay {
    display: none; position: fixed; inset: 0;
    background: rgba(26,18,8,0.88); z-index: 1000;
    align-items: center; justify-content: center; padding: 2rem;
  }
  .modal-overlay.open { display: flex; }
  .modal {
    background: var(--paper); border-radius: 6px;
    max-width: 820px; width: 100%; max-height: 90vh;
    overflow-y: auto;
    border: 1px solid var(--sand);
    animation: modalIn 0.25s ease;
  }
  @keyframes modalIn {
    from { opacity: 0; transform: scale(0.95) translateY(20px); }
    to   { opacity: 1; transform: scale(1) translateY(0); }
  }
  .modal-header {
    display: flex; justify-content: space-between; align-items: flex-start;
    padding: 1.6rem 2rem 0;
    border-bottom: 1px solid var(--sand); padding-bottom: 1rem;
  }
  .modal-header h2 { font-family: 'Playfair Display', serif; font-size: 1.3rem; color: var(--soil); }
  .modal-close {
    background: none; border: none; cursor: pointer;
    font-size: 1.4rem; color: var(--clay); line-height: 1;
    transition: color 0.15s;
  }
  .modal-close:hover { color: var(--soil); }
  .modal-body { padding: 1.6rem 2rem 2rem; }
  .modal-slider-wrap {
    position: relative; height: 320px; border-radius: 4px;
    overflow: hidden; margin-bottom: 1.2rem;
    border: 1.5px solid var(--sand); cursor: ew-resize;
    user-select: none;
  }
  .modal-slider-wrap .slider-after,
  .modal-slider-wrap .slider-before { position: absolute; inset: 0; }
  .modal-slider-wrap canvas { width: 100%; height: 100%; display: block; }

  /* ── FOOTER ── */
  footer {
    background: var(--soil); color: var(--sand);
    text-align: center; padding: 2rem;
    font-family: 'DM Mono', monospace; font-size: 0.7rem;
    letter-spacing: 0.1em; text-transform: uppercase;
    margin-top: 4rem;
    border-top: 4px solid var(--gold);
  }
  footer span { color: var(--gold); }

  @media (max-width: 600px) {
    header { padding: 1.5rem; }
    .header-stats { display: none; }
    .form-grid { grid-template-columns: 1fr; }
    .slider-container { height: 280px; }
  }
</style>
</head>
<body>

<header>
  <div class="header-inner">
    <div class="logo-block">
      <div class="logo-eyebrow">Community Project · Since 2018</div>
      <h1>Terra <em>Memoria</em></h1>
      <div class="header-desc">Documenting how our land transforms, one place at a time.</div>
    </div>
    <div class="header-stats">
      <div class="stat"><span class="stat-num" id="stat-sites">12</span><span class="stat-label">Sites</span></div>
      <div class="stat"><span class="stat-num" id="stat-years">7</span><span class="stat-label">Years</span></div>
      <div class="stat"><span class="stat-num" id="stat-contrib">34</span><span class="stat-label">Contributors</span></div>
    </div>
  </div>
</header>

<nav>
  <div class="nav-inner">
    <button class="tab-btn active" onclick="switchTab('gallery',this)">📷 Gallery</button>
    <button class="tab-btn" onclick="switchTab('compare',this)">⇔ Compare</button>
    <button class="tab-btn" onclick="switchTab('timeline',this)">📅 Timeline</button>
    <button class="tab-btn" onclick="switchTab('submit',this)">＋ Submit <span class="tab-badge">New</span></button>
  </div>
</nav>

<!-- ════════════════════ GALLERY ════════════════════ -->
<div class="section active" id="tab-gallery">
  <div class="gallery-wrap">
    <h2 class="section-title">Local Land Records</h2>
    <p class="section-sub">Click any record to view the before/after comparison</p>
    <div class="filter-bar">
      <button class="filter-btn active" onclick="filterCards('all',this)">All</button>
      <button class="filter-btn" onclick="filterCards('urban',this)">Urban Development</button>
      <button class="filter-btn" onclick="filterCards('agriculture',this)">Agriculture</button>
      <button class="filter-btn" onclick="filterCards('green',this)">Green Space</button>
      <button class="filter-btn" onclick="filterCards('industrial',this)">Industrial</button>
    </div>
    <div class="gallery-grid" id="gallery-grid"></div>
  </div>
</div>

<!-- ════════════════════ COMPARE ════════════════════ -->
<div class="section" id="tab-compare">
  <div class="compare-wrap">
    <h2 class="section-title">Before / After Viewer</h2>
    <p class="section-sub">Drag the slider to reveal changes over time</p>
    <div class="compare-select">
      <select id="compare-select" onchange="loadComparison()">
      </select>
    </div>
    <div class="slider-container" id="main-slider">
      <div class="slider-after"><canvas id="canvas-after"></canvas></div>
      <div class="slider-before" id="before-clip"><canvas id="canvas-before"></canvas></div>
      <div class="slider-divider" id="divider"></div>
      <div class="slider-handle" id="handle">
        <svg width="20" height="16" viewBox="0 0 20 16" fill="none">
          <path d="M6 1L1 8l5 7M14 1l5 7-5 7" stroke="currentColor" stroke-width="2" stroke-linecap="round"/>
        </svg>
      </div>
      <div class="slider-label label-before">Before</div>
      <div class="slider-label label-after">After</div>
    </div>
    <div class="compare-info" id="compare-info">
      <h3 id="ci-title">—</h3>
      <p id="ci-desc">—</p>
      <div class="changes-list" id="ci-changes"></div>
    </div>
  </div>
</div>

<!-- ════════════════════ TIMELINE ════════════════════ -->
<div class="section" id="tab-timeline">
  <div class="timeline-wrap">
    <h2 class="section-title">Chronology of Change</h2>
    <p class="section-sub">A record of documented transformations</p>
    <div class="timeline" id="timeline"></div>
  </div>
</div>

<!-- ════════════════════ SUBMIT ════════════════════ -->
<div class="section" id="tab-submit">
  <div class="submit-wrap">
    <h2 class="section-title">Submit an Observation</h2>
    <p class="section-sub">Help document your local landscape</p>
    <div class="form-card">
      <div id="form-body">
        <div class="form-grid">
          <div class="form-group">
            <label for="f-name">Your Name</label>
            <input type="text" id="f-name" placeholder="e.g. Priya Sharma">
          </div>
          <div class="form-group">
            <label for="f-email">Email</label>
            <input type="email" id="f-email" placeholder="you@example.com">
          </div>
          <div class="form-group">
            <label for="f-location">Location / Area Name</label>
            <input type="text" id="f-location" placeholder="e.g. Near Bus Stand, Phagwara">
          </div>
          <div class="form-group">
            <label for="f-year-before">Year (Before)</label>
            <input type="number" id="f-year-before" placeholder="e.g. 2018" min="1980" max="2025">
          </div>
          <div class="form-group">
            <label for="f-year-after">Year (After)</label>
            <input type="number" id="f-year-after" placeholder="e.g. 2024" min="1980" max="2025">
          </div>
          <div class="form-group">
            <label for="f-category">Category</label>
            <select id="f-category" class="form-select">
              <option value="">— Select —</option>
              <option>Urban Development</option>
              <option>Agriculture</option>
              <option>Green Space</option>
              <option>Industrial</option>
              <option>Water Body</option>
              <option>Other</option>
            </select>
          </div>
          <div class="form-group full">
            <label for="f-desc">Description of Change</label>
            <textarea id="f-desc" placeholder="Describe what changed and why it matters to the community…"></textarea>
          </div>
          <div class="form-group full">
            <label>Upload Photos (Before & After)</label>
            <div class="dropzone" id="dropzone"
                 ondragover="dzOver(event)" ondragleave="dzLeave(event)" ondrop="dzDrop(event)" onclick="document.getElementById('file-input').click()">
              <div class="dropzone-icon">🗂️</div>
              <div class="dropzone-text">Drag & drop photos here</div>
              <div class="dropzone-sub">or click to browse · JPG, PNG · Max 5MB each</div>
              <div class="file-list" id="file-list"></div>
            </div>
            <input type="file" id="file-input" multiple accept="image/*" style="display:none" onchange="handleFiles(this.files)">
          </div>
        </div>
        <button class="submit-btn" onclick="submitForm()">Submit Observation →</button>
      </div>
      <div class="submit-success" id="submit-success">
        <div class="check">🌱</div>
        <h3>Thank you for contributing!</h3>
        <p>Your observation has been received and will be reviewed by our team before being added to the archive.</p>
      </div>
    </div>
  </div>
</div>

<!-- MODAL -->
<div class="modal-overlay" id="modal" onclick="closeModal(event)">
  <div class="modal">
    <div class="modal-header">
      <div>
        <div style="font-family:'DM Mono',monospace;font-size:0.67rem;color:var(--clay);text-transform:uppercase;letter-spacing:0.12em;margin-bottom:4px" id="m-location"></div>
        <h2 id="m-title"></h2>
      </div>
      <button class="modal-close" onclick="closeModal()">✕</button>
    </div>
    <div class="modal-body">
      <div class="modal-slider-wrap" id="modal-slider">
        <div class="slider-after"><canvas id="m-canvas-after"></canvas></div>
        <div class="slider-before" id="m-before-clip"><canvas id="m-canvas-before"></canvas></div>
        <div class="slider-divider" id="m-divider"></div>
        <div class="slider-handle" id="m-handle">
          <svg width="18" height="14" viewBox="0 0 20 16" fill="none">
            <path d="M6 1L1 8l5 7M14 1l5 7-5 7" stroke="currentColor" stroke-width="2" stroke-linecap="round"/>
          </svg>
        </div>
        <div class="slider-label label-before" id="m-label-b">Before</div>
        <div class="slider-label label-after" id="m-label-a">After</div>
      </div>
      <p id="m-desc" style="font-size:0.9rem;color:#5a4535;line-height:1.7;"></p>
      <div class="changes-list" id="m-changes" style="margin-top:1rem;"></div>
    </div>
  </div>
</div>

<footer>
  <span>Terra Memoria</span> · Community Land Use Archive · Built with HTML & JavaScript
</footer>

<script>
/* ─── DATA ─── */
const SITES = [
  {
    id: 1, title: "Grain Fields to Housing Colony",
    location: "Kapurthala Road, Phagwara", category: "urban",
    yearBefore: 2017, yearAfter: 2024,
    desc: "A 12-acre wheat-growing plot was gradually converted into a residential colony with 200+ plots. The shift replaced seasonal agriculture with permanent concrete construction.",
    changes: [{t:"Agriculture lost",k:"loss"},{t:"200+ new homes",k:"gain"},{t:"12 acres converted",k:"neutral"}],
    palette: { before: ["#b5c98a","#8aad7c","#c8a97a","#a0b870"], after: ["#d0c8c0","#a09898","#c0b8b0","#e0d8d0"] }
  },
  {
    id: 2, title: "River Bank Encroachment",
    location: "Beas River Edge, Nawanshahr", category: "green",
    yearBefore: 2015, yearAfter: 2023,
    desc: "Natural floodplain vegetation along the Beas bank has been replaced by unauthorised construction and dumping, reducing the natural flood buffer and biodiversity corridor.",
    changes: [{t:"Vegetation cleared",k:"loss"},{t:"Flood risk ↑",k:"loss"},{t:"Construction added",k:"neutral"}],
    palette: { before: ["#7baec3","#8aad7c","#4a9070","#5a9ab0"], after: ["#c0b8b0","#a08878","#c8c0b8","#b0a8a0"] }
  },
  {
    id: 3, title: "Paddy Fields → Industrial Zone",
    location: "NH-44 Corridor, Ludhiana", category: "industrial",
    yearBefore: 2019, yearAfter: 2024,
    desc: "Three contiguous paddy farming tracts were acquired for a new industrial park. The transition brought employment but eliminated traditional water-table recharge zones.",
    changes: [{t:"1200 jobs created",k:"gain"},{t:"Paddy farming ended",k:"loss"},{t:"Water recharge lost",k:"loss"}],
    palette: { before: ["#b5c98a","#8aad7c","#c0d0a0","#90b870"], after: ["#808888","#606870","#989898","#707878"] }
  },
  {
    id: 4, title: "Orchard Preserved as Park",
    location: "Gurudwara Road, Hoshiarpur", category: "green",
    yearBefore: 2018, yearAfter: 2023,
    desc: "An aging mango and guava orchard was rescued from development and converted into a public green space with walking paths, benches, and heritage tree tagging.",
    changes: [{t:"Public park created",k:"gain"},{t:"Heritage trees saved",k:"gain"},{t:"Development halted",k:"gain"}],
    palette: { before: ["#7a9460","#90a868","#b0b870","#60884a"], after: ["#8aad7c","#6a9060","#a8c880","#5a8050"] }
  },
  {
    id: 5, title: "Village Pond Reclamation",
    location: "Adampur, Jalandhar", category: "agriculture",
    yearBefore: 2016, yearAfter: 2022,
    desc: "A traditional village pond used for livestock and groundwater recharge was illegally filled and turned into agricultural land. Community activists have documented its disappearance.",
    changes: [{t:"Pond eliminated",k:"loss"},{t:"Groundwater impact",k:"loss"},{t:"Agricultural gain",k:"neutral"}],
    palette: { before: ["#5a9ab0","#7baec3","#4a7890","#90c0d0"], after: ["#c8a97a","#b09060","#d0b888","#a08858"] }
  },
  {
    id: 6, title: "Derelict Factory → Community Hub",
    location: "G.T. Road, Phagwara", category: "urban",
    yearBefore: 2020, yearAfter: 2024,
    desc: "An abandoned textile mill from the 1980s was rehabilitated into a cultural centre and co-working space, preserving the original brick façade while adding modern amenities.",
    changes: [{t:"Mill restored",k:"gain"},{t:"Heritage preserved",k:"gain"},{t:"Community space",k:"gain"}],
    palette: { before: ["#807060","#908070","#a09080","#706050"], after: ["#c0a880","#a89070","#d0b898","#906840"] }
  }
];

const TIMELINE_EVENTS = [
  { year: "2015", title: "Project Founded", desc: "Terra Memoria starts as a student documentation initiative at a local college, with 3 founding contributors and 2 initial sites.", highlight: false },
  { year: "2017", title: "Mobile Photography Drive", desc: "Community volunteers equipped with smartphones document 15 new sites across Doaba region, tripling the archive.", highlight: false },
  { year: "2019", title: "Municipal Partnership", desc: "Formal data-sharing agreement signed with the Municipal Council. Official land records now cross-referenced with community photos.", highlight: true },
  { year: "2021", title: "35-Site Milestone & COVID Gap", desc: "Archive reaches 35 documented sites. COVID restrictions pause field work for 8 months, highlighting need for remote sensing.", highlight: false },
  { year: "2023", title: "River Bank Campaign", desc: "A focused campaign documents illegal encroachments on 12km of Beas riverbank. Report submitted to PPCB.", highlight: true },
  { year: "2024", title: "Present Day", desc: "Active archive with 12 active comparison sites, 34 contributors, and ongoing community submissions via this platform.", highlight: true },
];

/* ─── CANVAS DRAWING ─── */
function drawScene(canvas, palette, label, year) {
  const ctx = canvas.getContext('2d');
  canvas.width = 800; canvas.height = 480;
  const w = 800, h = 480;
  const p = palette;

  // Sky/ground split
  const grad = ctx.createLinearGradient(0,0,0,h*0.45);
  grad.addColorStop(0, label==='before' ? '#d4e8f0' : '#c8d8e4');
  grad.addColorStop(1, label==='before' ? '#e8f4f8' : '#d8e4ec');
  ctx.fillStyle = grad; ctx.fillRect(0,0,w,h*0.45);

  // Ground
  const gGrad = ctx.createLinearGradient(0,h*0.45,0,h);
  gGrad.addColorStop(0, p[0]); gGrad.addColorStop(1, p[1]);
  ctx.fillStyle = gGrad; ctx.fillRect(0,h*0.45,w,h*0.55);

  // Patches
  ctx.globalAlpha = 0.55;
  for(let i=0;i<8;i++) {
    ctx.fillStyle = p[i%p.length];
    const x = (i*111)%w, y = h*0.45 + (i*73)%(h*0.4);
    ctx.beginPath();
    ctx.ellipse(x+80, y+40, 90+i*8, 45+i*4, (i*0.4), 0, Math.PI*2);
    ctx.fill();
  }
  ctx.globalAlpha = 1;

  // Horizon line features
  if(label==='before') {
    // Trees
    for(let i=0;i<6;i++) {
      const tx = 80+i*130, ty = h*0.38;
      ctx.fillStyle = '#4a6741';
      ctx.beginPath(); ctx.arc(tx,ty,18,0,Math.PI*2); ctx.fill();
      ctx.fillStyle = '#3a5531';
      ctx.beginPath(); ctx.arc(tx-5,ty+8,12,0,Math.PI*2); ctx.fill();
      ctx.fillStyle = '#5a4535'; ctx.fillRect(tx-3,ty+16,6,24);
    }
  } else {
    // Buildings
    for(let i=0;i<5;i++) {
      const bx = 60+i*155, by = h*0.28;
      const bw = 80+i*5, bh = h*0.18+i*8;
      ctx.fillStyle = `hsl(${220+i*10},8%,${55+i*4}%)`;
      ctx.fillRect(bx, by, bw, bh);
      // Windows
      ctx.fillStyle = `rgba(200,200,180,0.6)`;
      for(let r=0;r<3;r++) for(let c=0;c<3;c++) {
        ctx.fillRect(bx+10+c*22, by+10+r*22, 12, 12);
      }
    }
  }

  // Year watermark
  ctx.font = 'bold 120px Georgia';
  ctx.fillStyle = 'rgba(0,0,0,0.04)';
  ctx.textAlign = 'center'; ctx.textBaseline = 'middle';
  ctx.fillText(year, w/2, h*0.72);
}

function renderCard(site, canvas) {
  // Just draw "after" view for gallery thumb
  drawScene(canvas, site.palette.after, 'after', site.yearAfter);
}

/* ─── GALLERY ─── */
function buildGallery(filter='all') {
  const grid = document.getElementById('gallery-grid');
  grid.innerHTML = '';
  SITES.filter(s => filter==='all' || s.category===filter).forEach(site => {
    const div = document.createElement('div');
    div.className = 'card'; div.dataset.cat = site.category;
    div.innerHTML = `
      <div class="card-img-wrap">
        <canvas id="thumb-${site.id}"></canvas>
        <div class="card-tag">${site.category}</div>
      </div>
      <div class="card-body">
        <div class="card-location">${site.location}</div>
        <div class="card-title">${site.title}</div>
        <div class="card-desc">${site.desc.substring(0,110)}…</div>
        <div class="card-meta">
          <span>${site.yearBefore} → ${site.yearAfter}</span>
          <span>${site.changes.length} changes noted</span>
        </div>
      </div>`;
    div.onclick = () => openModal(site);
    grid.appendChild(div);
    requestAnimationFrame(() => {
      const c = document.getElementById(`thumb-${site.id}`);
      if(c) drawScene(c, site.palette.after, 'after', site.yearAfter);
    });
  });
}

function filterCards(cat, btn) {
  document.querySelectorAll('.filter-btn').forEach(b => b.classList.remove('active'));
  btn.classList.add('active');
  buildGallery(cat);
}

/* ─── SLIDER LOGIC ─── */
function makeSlider(container, beforeClip, dividerEl, handleEl) {
  let dragging = false;
  function setPos(pct) {
    pct = Math.max(5, Math.min(95, pct));
    beforeClip.style.clipPath = `inset(0 ${100-pct}% 0 0)`;
    dividerEl.style.left = pct + '%';
    handleEl.style.left = pct + '%';
  }
  function getPercent(e) {
    const rect = container.getBoundingClientRect();
    const x = (e.touches ? e.touches[0].clientX : e.clientX) - rect.left;
    return (x / rect.width) * 100;
  }
  container.addEventListener('mousedown', e => { dragging = true; setPos(getPercent(e)); });
  container.addEventListener('touchstart', e => { dragging = true; setPos(getPercent(e)); }, {passive:true});
  window.addEventListener('mousemove', e => { if(dragging) setPos(getPercent(e)); });
  window.addEventListener('touchmove', e => { if(dragging) setPos(getPercent(e)); }, {passive:true});
  window.addEventListener('mouseup', () => dragging = false);
  window.addEventListener('touchend', () => dragging = false);
  setPos(50);
}

/* ─── COMPARE TAB ─── */
function buildCompareSelect() {
  const sel = document.getElementById('compare-select');
  SITES.forEach(s => {
    const opt = document.createElement('option');
    opt.value = s.id; opt.textContent = s.title;
    sel.appendChild(opt);
  });
  loadComparison();
  makeSlider(
    document.getElementById('main-slider'),
    document.getElementById('before-clip'),
    document.getElementById('divider'),
    document.getElementById('handle')
  );
}

function loadComparison() {
  const id = +document.getElementById('compare-select').value;
  const site = SITES.find(s=>s.id===id);
  if(!site) return;
  drawScene(document.getElementById('canvas-before'), site.palette.before, 'before', site.yearBefore);
  drawScene(document.getElementById('canvas-after'), site.palette.after, 'after', site.yearAfter);
  document.getElementById('ci-title').textContent = site.title;
  document.getElementById('ci-desc').textContent = site.desc;
  const ch = document.getElementById('ci-changes');
  ch.innerHTML = site.changes.map(c=>`<span class="change-pill ${c.k}">${c.t}</span>`).join('');
  // reset slider
  document.getElementById('before-clip').style.clipPath = 'inset(0 50% 0 0)';
  document.getElementById('divider').style.left = '50%';
  document.getElementById('handle').style.left = '50%';
}

/* ─── TIMELINE ─── */
function buildTimeline() {
  const tl = document.getElementById('timeline');
  TIMELINE_EVENTS.forEach(ev => {
    const item = document.createElement('div');
    item.className = 'timeline-item';
    item.innerHTML = `
      <div class="timeline-dot ${ev.highlight?'highlight':''}"></div>
      <div class="timeline-year">${ev.year}</div>
      <div class="timeline-card">
        <h4>${ev.title}</h4>
        <p>${ev.desc}</p>
      </div>`;
    tl.appendChild(item);
  });
}

/* ─── MODAL ─── */
let modalSliderInit = false;
function openModal(site) {
  document.getElementById('m-location').textContent = site.location;
  document.getElementById('m-title').textContent = site.title;
  document.getElementById('m-desc').textContent = site.desc;
  document.getElementById('m-label-b').textContent = `Before · ${site.yearBefore}`;
  document.getElementById('m-label-a').textContent = `After · ${site.yearAfter}`;
  document.getElementById('m-changes').innerHTML = site.changes.map(c=>`<span class="change-pill ${c.k}">${c.t}</span>`).join('');
  requestAnimationFrame(() => {
    drawScene(document.getElementById('m-canvas-before'), site.palette.before, 'before', site.yearBefore);
    drawScene(document.getElementById('m-canvas-after'), site.palette.after, 'after', site.yearAfter);
  });
  document.getElementById('modal').classList.add('open');
  if(!modalSliderInit) {
    makeSlider(
      document.getElementById('modal-slider'),
      document.getElementById('m-before-clip'),
      document.getElementById('m-divider'),
      document.getElementById('m-handle')
    );
    modalSliderInit = true;
  } else {
    document.getElementById('m-before-clip').style.clipPath = 'inset(0 50% 0 0)';
    document.getElementById('m-divider').style.left = '50%';
    document.getElementById('m-handle').style.left = '50%';
  }
}
function closeModal(e) {
  if(!e || e.target===document.getElementById('modal') || e.currentTarget.classList.contains('modal-close')) {
    document.getElementById('modal').classList.remove('open');
  }
}

/* ─── TABS ─── */
function switchTab(name, btn) {
  document.querySelectorAll('.section').forEach(s => s.classList.remove('active'));
  document.querySelectorAll('.tab-btn').forEach(b => b.classList.remove('active'));
  document.getElementById('tab-'+name).classList.add('active');
  btn.classList.add('active');
}

/* ─── DROPZONE ─── */
const uploadedFiles = [];
function dzOver(e) { e.preventDefault(); document.getElementById('dropzone').classList.add('drag-over'); }
function dzLeave()  { document.getElementById('dropzone').classList.remove('drag-over'); }
function dzDrop(e)  { e.preventDefault(); dzLeave(); handleFiles(e.dataTransfer.files); }
function handleFiles(files) {
  const list = document.getElementById('file-list');
  Array.from(files).forEach(f => {
    uploadedFiles.push(f.name);
    const item = document.createElement('div');
    item.className = 'file-item'; item.textContent = f.name;
    list.appendChild(item);
  });
}

/* ─── FORM SUBMIT ─── */
function submitForm() {
  const name = document.getElementById('f-name').value.trim();
  const loc  = document.getElementById('f-location').value.trim();
  if(!name || !loc) { alert('Please fill in at least your name and location.'); return; }
  document.getElementById('form-body').style.display = 'none';
  document.getElementById('submit-success').style.display = 'block';
}

/* ─── INIT ─── */
buildGallery();
buildCompareSelect();
buildTimeline();
</script>
</body>
</html>
