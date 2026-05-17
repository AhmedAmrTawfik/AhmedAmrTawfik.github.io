<!DOCTYPE html>
<html lang="ar" dir="rtl">
<head>
  <meta charset="UTF-8" />
  <meta name="viewport" content="width=device-width, initial-scale=1.0" />
  <title>عداد الصلوات</title>
  <link href="https://fonts.googleapis.com/css2?family=Amiri:wght@400;700&family=Cairo:wght@300;400;600;700&display=swap" rel="stylesheet" />
  <style>
    *, *::before, *::after { box-sizing: border-box; margin: 0; padding: 0; }

    :root {
      --bg:        #0d1117;
      --surface:   #161b22;
      --border:    #30363d;
      --gold:      #c9a84c;
      --gold-dim:  #7a6230;
      --green:     #3fb950;
      --text:      #e6edf3;
      --muted:     #8b949e;
      --radius:    14px;
    }

    body {
      background: var(--bg);
      color: var(--text);
      font-family: 'Cairo', sans-serif;
      min-height: 100vh;
      display: flex;
      flex-direction: column;
      align-items: center;
      padding: 40px 16px 60px;
    }

    /* ── Header ── */
    header {
      text-align: center;
      margin-bottom: 48px;
    }

    header .arabic-title {
      font-family: 'Amiri', serif;
      font-size: clamp(2rem, 6vw, 3.2rem);
      font-weight: 700;
      color: var(--gold);
      letter-spacing: 0.02em;
      line-height: 1.2;
    }

    header .subtitle {
      font-size: 0.85rem;
      color: var(--muted);
      margin-top: 6px;
      direction: ltr;
    }

    /* ── Grid ── */
    .grid {
      display: grid;
      grid-template-columns: repeat(auto-fill, minmax(300px, 1fr));
      gap: 20px;
      width: 100%;
      max-width: 1020px;
    }

    /* ── Card ── */
    .card {
      background: var(--surface);
      border: 1px solid var(--border);
      border-radius: var(--radius);
      padding: 24px 20px 20px;
      display: flex;
      flex-direction: column;
      gap: 16px;
      transition: border-color 0.2s, transform 0.15s;
    }
    .card:hover {
      border-color: var(--gold-dim);
      transform: translateY(-2px);
    }

    /* prayer name */
    .prayer-name {
      font-family: 'Amiri', serif;
      font-size: 1.75rem;
      font-weight: 700;
      color: var(--gold);
    }

    /* counter row */
    .counter-row {
      display: flex;
      align-items: center;
      gap: 10px;
    }

    .btn-stack {
      display: flex;
      flex-direction: column;
      gap: 3px;
    }

    .btn {
      width: 18px;
      height: 18px;
      border-radius: 4px;
      border: 1px solid var(--border);
      background: transparent;
      color: var(--text);
      font-size: 0.65rem;
      cursor: pointer;
      display: flex;
      align-items: center;
      justify-content: center;
      transition: background 0.15s, border-color 0.15s, color 0.15s;
      flex-shrink: 0;
      line-height: 1;
      padding: 0;
    }
    .btn:hover { background: var(--border); }
    .btn.plus:hover { border-color: var(--green); color: var(--green); }
    .btn.minus:hover { border-color: #f85149; color: #f85149; }
    .btn:active { transform: scale(0.9); }

    .count-display {
      font-size: 2.4rem;
      font-weight: 700;
      color: var(--text);
      letter-spacing: -0.02em;
      min-width: 80px;
      text-align: center;
      transition: color 0.2s;
    }
    .count-display.bump {
      animation: bump 0.25s ease;
    }
    @keyframes bump {
      0%   { transform: scale(1); }
      40%  { transform: scale(1.18); }
      100% { transform: scale(1); }
    }

    /* out-of label */
    .out-of {
      font-size: 0.78rem;
      color: var(--muted);
      text-align: center;
      direction: ltr;
    }

    /* progress bar */
    .progress-wrap {
      display: flex;
      flex-direction: column;
      gap: 6px;
    }
    .progress-meta {
      display: flex;
      justify-content: space-between;
      font-size: 0.75rem;
      color: var(--muted);
      direction: ltr;
    }
    .bar-track {
      width: 100%;
      height: 8px;
      background: #21262d;
      border-radius: 4px;
      overflow: hidden;
    }
    .bar-fill {
      height: 100%;
      border-radius: 4px;
      background: linear-gradient(90deg, var(--gold-dim), var(--gold));
      transition: width 0.4s cubic-bezier(0.34, 1.56, 0.64, 1);
      min-width: 0;
    }
    .bar-fill.complete {
      background: linear-gradient(90deg, #238636, var(--green));
    }

    /* reset all button */
    .reset-all {
      margin-top: 32px;
      padding: 10px 28px;
      border-radius: 8px;
      border: 1px solid #f85149;
      background: transparent;
      color: #f85149;
      font-family: 'Cairo', sans-serif;
      font-size: 0.9rem;
      cursor: pointer;
      transition: background 0.15s;
    }
    .reset-all:hover { background: rgba(248,81,73,0.1); }

    /* completion badge */
    .badge {
      font-size: 0.72rem;
      padding: 2px 8px;
      border-radius: 20px;
      background: rgba(63,185,80,0.15);
      color: var(--green);
      border: 1px solid rgba(63,185,80,0.35);
      display: none;
    }
    .badge.show { display: inline-block; }
  </style>
</head>
<body>

<header>
  <div class="arabic-title">عداد الصلوات</div>
  <div class="subtitle">Prayer Counter · Target: 3605</div>
</header>

<div class="grid" id="grid"></div>

<button class="reset-all" onclick="resetAll()">إعادة تعيين الكل</button>

<script>
  const PRAYERS = ['فجر','ظهر','عصر','مغرب','عشاء'];
  const MAX = 3605;
  const KEY = 'prayer_counts_v1';

  // Load or init
  function load() {
    try { return JSON.parse(localStorage.getItem(KEY)) || {}; } catch { return {}; }
  }
  function save(data) { localStorage.setItem(KEY, JSON.stringify(data)); }

  let counts = load();
  PRAYERS.forEach(p => { if (counts[p] === undefined) counts[p] = 0; });

  function render() {
    const grid = document.getElementById('grid');
    grid.innerHTML = '';
    PRAYERS.forEach(p => buildCard(p, grid));
  }

  function buildCard(prayer, container) {
    const val = counts[prayer];
    const pct = Math.min(val / MAX * 100, 100).toFixed(2);
    const complete = val >= MAX;

    const card = document.createElement('div');
    card.className = 'card';
    card.id = 'card-' + prayer;

    card.innerHTML = `
      <div style="display:flex;align-items:center;justify-content:space-between;">
        <span class="prayer-name">${prayer}</span>
        <span class="badge ${complete ? 'show' : ''}" id="badge-${prayer}">✓ مكتمل</span>
      </div>
      <div class="counter-row">
        <div class="count-display" id="count-${prayer}">${val.toLocaleString('ar-EG')}</div>
        <div class="out-of">${val.toLocaleString()} / ${MAX.toLocaleString()}</div>
        <div class="btn-stack">
          <button class="btn plus" onclick="change('${prayer}', 1)" title="+1">+</button>
          <button class="btn minus" onclick="change('${prayer}', -1)" title="-1">−</button>
        </div>
      </div>
      <div class="progress-wrap">
        <div class="progress-meta">
          <span>${pct}%</span>
          <span>${Math.max(MAX - val, 0).toLocaleString()} متبقي</span>
        </div>
        <div class="bar-track">
          <div class="bar-fill ${complete ? 'complete' : ''}" id="bar-${prayer}" style="width:${pct}%"></div>
        </div>
      </div>
    `;
    container.appendChild(card);
  }

  function change(prayer, delta) {
    counts[prayer] = Math.max(0, counts[prayer] + delta);
    save(counts);
    updateCard(prayer);
  }

  function updateCard(prayer) {
    const val = counts[prayer];
    const pct = Math.min(val / MAX * 100, 100).toFixed(2);
    const complete = val >= MAX;

    // count
    const countEl = document.getElementById('count-' + prayer);
    countEl.textContent = val.toLocaleString('ar-EG');
    countEl.classList.remove('bump');
    void countEl.offsetWidth; // reflow to restart animation
    countEl.classList.add('bump');

    // out-of (sibling div)
    countEl.nextElementSibling.textContent = `${val.toLocaleString()} / ${MAX.toLocaleString()}`;

    // bar
    const bar = document.getElementById('bar-' + prayer);
    bar.style.width = pct + '%';
    bar.className = 'bar-fill' + (complete ? ' complete' : '');

    // progress meta
    const meta = bar.closest('.progress-wrap').querySelector('.progress-meta');
    meta.children[0].textContent = pct + '%';
    meta.children[1].textContent = Math.max(MAX - val, 0).toLocaleString() + ' متبقي';

    // badge
    const badge = document.getElementById('badge-' + prayer);
    badge.classList.toggle('show', complete);
  }

  function resetAll() {
    if (!confirm('هل تريد إعادة تعيين جميع العدادات؟')) return;
    PRAYERS.forEach(p => counts[p] = 0);
    save(counts);
    render();
  }

  render();
</script>
</body>
</html>
