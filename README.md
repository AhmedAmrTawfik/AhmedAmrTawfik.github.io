<!DOCTYPE html>
<html lang="ar" dir="rtl">
<head>
  <meta charset="UTF-8" />
  <meta name="viewport" content="width=device-width, initial-scale=1.0, maximum-scale=1.0" />
  <title>عداد الصلوات</title>
  <link href="https://fonts.googleapis.com/css2?family=Amiri:wght@400;700&family=Cairo:wght@400;600;700&display=swap" rel="stylesheet" />
  <style>
    *, *::before, *::after { box-sizing: border-box; margin: 0; padding: 0; }

    :root {
      --bg:       #0d1117;
      --surface:  #161b22;
      --border:   #30363d;
      --gold:     #c9a84c;
      --gold-dim: #7a6230;
      --green:    #3fb950;
      --text:     #e6edf3;
      --muted:    #8b949e;
    }

    body {
      background: var(--bg);
      color: var(--text);
      font-family: 'Cairo', sans-serif;
      min-height: 100vh;
      display: flex;
      flex-direction: column;
      align-items: center;
      padding: 24px 12px 40px;
    }

    header {
      text-align: center;
      margin-bottom: 20px;
    }
    header .title {
      font-family: 'Amiri', serif;
      font-size: 1.6rem;
      font-weight: 700;
      color: var(--gold);
    }

    .list {
      width: 100%;
      max-width: 420px;
      display: flex;
      flex-direction: column;
      gap: 8px;
    }

    .card {
      background: var(--surface);
      border: 1px solid var(--border);
      border-radius: 10px;
      padding: 8px 12px;
      display: flex;
      flex-direction: column;
      gap: 5px;
    }

    .top-line {
      display: flex;
      align-items: center;
      gap: 8px;
    }

    .prayer-name {
      font-family: 'Amiri', serif;
      font-size: 1.15rem;
      font-weight: 700;
      color: var(--gold);
      min-width: 42px;
    }

    .count-display {
      font-size: 1.05rem;
      font-weight: 700;
      color: var(--text);
      min-width: 38px;
      text-align: center;
    }
    .count-display.bump { animation: bump 0.2s ease; }
    @keyframes bump {
      0%  { transform: scale(1); }
      40% { transform: scale(1.2); }
      100%{ transform: scale(1); }
    }

    .out-of {
      font-size: 0.62rem;
      color: var(--muted);
      direction: ltr;
      white-space: nowrap;
    }

    .spacer { flex: 1; }

    .btn-stack {
      display: flex;
      flex-direction: column;
      gap: 2px;
    }
    .btn {
      width: 20px;
      height: 16px;
      border-radius: 3px;
      border: 1px solid var(--border);
      background: transparent;
      color: var(--text);
      font-size: 0.65rem;
      cursor: pointer;
      display: flex;
      align-items: center;
      justify-content: center;
      padding: 0;
      line-height: 1;
      -webkit-tap-highlight-color: transparent;
      touch-action: manipulation;
    }
    .btn:active { opacity: 0.6; transform: scale(0.92); }
    .btn.plus  { border-color: #238636; color: var(--green); }
    .btn.minus { border-color: #6e3630; color: #f85149; }

    .badge {
      font-size: 0.6rem;
      padding: 1px 5px;
      border-radius: 20px;
      background: rgba(63,185,80,0.15);
      color: var(--green);
      border: 1px solid rgba(63,185,80,0.35);
      white-space: nowrap;
      display: none;
    }
    .badge.show { display: inline-block; }

    .bar-track {
      width: 100%;
      height: 5px;
      background: #21262d;
      border-radius: 3px;
      overflow: hidden;
    }
    .bar-fill {
      height: 100%;
      border-radius: 3px;
      background: linear-gradient(90deg, var(--gold-dim), var(--gold));
      transition: width 0.35s cubic-bezier(0.34, 1.56, 0.64, 1);
    }
    .bar-fill.complete {
      background: linear-gradient(90deg, #238636, var(--green));
    }

    .pct-label {
      font-size: 0.6rem;
      color: var(--muted);
      direction: ltr;
      text-align: left;
    }

    .reset-all {
      margin-top: 20px;
      padding: 7px 22px;
      border-radius: 7px;
      border: 1px solid #6e3630;
      background: transparent;
      color: #f85149;
      font-family: 'Cairo', sans-serif;
      font-size: 0.8rem;
      cursor: pointer;
      -webkit-tap-highlight-color: transparent;
    }
    .reset-all:active { background: rgba(248,81,73,0.1); }
  </style>
</head>
<body>

<header>
  <div class="title">عداد الصلوات</div>
</header>

<div class="list" id="list"></div>
<button class="reset-all" onclick="resetAll()">إعادة تعيين الكل</button>

<script>
  const PRAYERS = ['فجر','ظهر','عصر','مغرب','عشاء'];
  const MAX = 3605;
  const KEY = 'prayer_counts_v1';

  function load() { try { return JSON.parse(localStorage.getItem(KEY)) || {}; } catch { return {}; } }
  function save(d) { localStorage.setItem(KEY, JSON.stringify(d)); }

  let counts = load();
  PRAYERS.forEach(p => { if (counts[p] === undefined) counts[p] = 0; });

  function render() {
    const list = document.getElementById('list');
    list.innerHTML = '';
    PRAYERS.forEach(p => buildCard(p, list));
  }

  function buildCard(prayer, container) {
    const val = counts[prayer];
    const pct = Math.min(val / MAX * 100, 100).toFixed(1);
    const complete = val >= MAX;

    const card = document.createElement('div');
    card.className = 'card';
    card.id = 'card-' + prayer;
    card.innerHTML =
      '<div class="top-line">' +
        '<span class="prayer-name">' + prayer + '</span>' +
        '<span class="count-display" id="count-' + prayer + '">' + val.toLocaleString() + '</span>' +
        '<span class="out-of" id="outof-' + prayer + '">/ ' + MAX.toLocaleString() + '</span>' +
        '<span class="spacer"></span>' +
        '<div class="btn-stack">' +
          '<button class="btn plus"  onclick="change(\'' + prayer + '\', 1)"  title="+1">+</button>' +
          '<button class="btn minus" onclick="change(\'' + prayer + '\', -1)" title="-1">\u2212</button>' +
        '</div>' +
        '<span class="badge ' + (complete ? 'show' : '') + '" id="badge-' + prayer + '">\u2713</span>' +
      '</div>' +
      '<div class="bar-track"><div class="bar-fill ' + (complete ? 'complete' : '') + '" id="bar-' + prayer + '" style="width:' + pct + '%"></div></div>' +
      '<div class="pct-label" id="pct-' + prayer + '">' + pct + '%</div>';
    container.appendChild(card);
  }

  function change(prayer, delta) {
    counts[prayer] = Math.max(0, counts[prayer] + delta);
    save(counts);
    updateCard(prayer);
  }

  function updateCard(prayer) {
    const val = counts[prayer];
    const pct = Math.min(val / MAX * 100, 100).toFixed(1);
    const complete = val >= MAX;

    const countEl = document.getElementById('count-' + prayer);
    countEl.textContent = val.toLocaleString();
    countEl.classList.remove('bump');
    void countEl.offsetWidth;
    countEl.classList.add('bump');

    const bar = document.getElementById('bar-' + prayer);
    bar.style.width = pct + '%';
    bar.className = 'bar-fill' + (complete ? ' complete' : '');

    document.getElementById('pct-' + prayer).textContent = pct + '%';
    document.getElementById('badge-' + prayer).classList.toggle('show', complete);
  }

  function resetAll() {
    if (!confirm('إعادة تعيين جميع العدادات؟')) return;
    PRAYERS.forEach(p => counts[p] = 0);
    save(counts);
    render();
  }

  render();
</script>
</body>
</html>
