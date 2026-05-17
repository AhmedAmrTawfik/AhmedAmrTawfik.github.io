<html lang="ar" dir="rtl">
<head>
  <meta charset="UTF-8" />
  <meta name="viewport" content="width=device-width, initial-scale=1.0, maximum-scale=1.0" />
  <title>عداد الصلوات</title>
  <link href="https://fonts.googleapis.com/css2?family=Amiri:wght@400;700&family=Cairo:wght@400;600;700&display=swap" rel="stylesheet" />
  <style>
    *, *::before, *::after { box-sizing: border-box; margin: 0; padding: 0; }

    body {
      background: #f0f4ff;
      color: #1a1a2e;
      font-family: 'Cairo', sans-serif;
      min-height: 100vh;
      display: flex;
      flex-direction: column;
      align-items: center;
      padding: 28px 14px 48px;
    }

    header { text-align: center; margin-bottom: 24px; }
    header .title {
      font-family: 'Amiri', serif;
      font-size: 1.9rem;
      font-weight: 700;
      color: #1e1b4b;
    }
    header .subtitle { font-size: 0.65rem; color: #888; direction: ltr; margin-top: 2px; }

    .list { width: 100%; max-width: 420px; display: flex; flex-direction: column; gap: 10px; }

    /* Per-prayer solid colors */
    .card[data-prayer="فجر"]  { --bg: #6c47ff; --dark: #4f33c0; --light: #ede9ff; --text-col: #fff; }
    .card[data-prayer="ظهر"]  { --bg: #f59e0b; --dark: #b87908; --light: #fff8e1; --text-col: #fff; }
    .card[data-prayer="عصر"]  { --bg: #10b981; --dark: #0a8c61; --light: #d1fae5; --text-col: #fff; }
    .card[data-prayer="مغرب"] { --bg: #ef4444; --dark: #b91c1c; --light: #fee2e2; --text-col: #fff; }
    .card[data-prayer="عشاء"] { --bg: #ec4899; --dark: #be185d; --light: #fce7f3; --text-col: #fff; }

    .card {
      background: var(--bg);
      border-radius: 14px;
      padding: 10px 14px 10px;
      display: flex;
      flex-direction: column;
      gap: 7px;
      box-shadow: 0 4px 14px color-mix(in srgb, var(--bg) 40%, transparent);
    }

    .top-line { display: flex; align-items: center; gap: 8px; }

    .prayer-name {
      font-family: 'Amiri', serif;
      font-size: 1.25rem;
      font-weight: 700;
      color: #fff;
      min-width: 44px;
    }

    .count-display {
      font-size: 1.15rem;
      font-weight: 700;
      color: #fff;
      min-width: 36px;
      text-align: center;
    }
    .count-display.bump { animation: bump 0.2s ease; }
    @keyframes bump { 0%{transform:scale(1)} 40%{transform:scale(1.3)} 100%{transform:scale(1)} }

    .out-of { font-size: 0.6rem; color: rgba(255,255,255,0.65); direction: ltr; white-space: nowrap; }

    .spacer { flex: 1; }

    .btn-stack { display: flex; flex-direction: column; gap: 3px; }
    .btn {
      width: 24px; height: 18px;
      border-radius: 5px;
      border: none;
      background: rgba(255,255,255,0.25);
      color: #fff;
      font-size: 0.75rem; font-weight: 700;
      cursor: pointer;
      display: flex; align-items: center; justify-content: center;
      padding: 0; line-height: 1;
      -webkit-tap-highlight-color: transparent;
      touch-action: manipulation;
    }
    .btn.minus { background: rgba(0,0,0,0.18); }
    .btn:active { opacity: 0.6; transform: scale(0.88); }

    .badge {
      font-size: 0.58rem; padding: 2px 6px; border-radius: 20px;
      background: rgba(255,255,255,0.9);
      color: var(--bg);
      font-weight: 700;
      white-space: nowrap; display: none;
    }
    .badge.show { display: inline-block; }

    /* progress bar sits on a lighter band */
    .bar-wrap {
      background: rgba(0,0,0,0.15);
      border-radius: 6px;
      padding: 4px 6px;
      display: flex;
      align-items: center;
      gap: 7px;
    }
    .bar-track {
      flex: 1; height: 6px;
      background: rgba(0,0,0,0.2);
      border-radius: 3px; overflow: hidden;
    }
    .bar-fill {
      height: 100%; border-radius: 3px;
      background: rgba(255,255,255,0.9);
      transition: width 0.4s cubic-bezier(0.34,1.56,0.64,1);
    }
    .pct-label {
      font-size: 0.6rem; color: rgba(255,255,255,0.85);
      direction: ltr; white-space: nowrap; min-width: 32px; text-align: right;
    }

    .reset-all {
      margin-top: 22px; padding: 8px 26px;
      border-radius: 8px; border: 2px solid #ef4444;
      background: #fff; color: #ef4444;
      font-family: 'Cairo', sans-serif; font-size: 0.85rem; font-weight: 600;
      cursor: pointer; -webkit-tap-highlight-color: transparent;
      box-shadow: 0 2px 8px rgba(239,68,68,0.2);
    }
    .reset-all:active { background: #fee2e2; }
  </style>
</head>
<body>

<header>
  <div class="title">عداد الصلوات</div>
  <div class="subtitle">Prayer Counter · 3605</div>
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
    card.setAttribute('data-prayer', prayer);
    card.id = 'card-' + prayer;
    card.innerHTML =
      '<div class="top-line">' +
        '<span class="prayer-name">' + prayer + '</span>' +
        '<span class="count-display" id="count-' + prayer + '">' + val.toLocaleString() + '</span>' +
        '<span class="out-of">/ ' + MAX.toLocaleString() + '</span>' +
        '<span class="spacer"></span>' +
        '<div class="btn-stack">' +
          '<button class="btn plus"  onclick="change(\'' + prayer + '\', 1)">+</button>' +
          '<button class="btn minus" onclick="change(\'' + prayer + '\', -1)">\u2212</button>' +
        '</div>' +
        '<span class="badge ' + (complete ? 'show' : '') + '" id="badge-' + prayer + '">\u2713 مكتمل</span>' +
      '</div>' +
      '<div class="bar-wrap">' +
        '<div class="bar-track"><div class="bar-fill" id="bar-' + prayer + '" style="width:' + pct + '%"></div></div>' +
        '<div class="pct-label" id="pct-' + prayer + '">' + pct + '%</div>' +
      '</div>';
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
    document.getElementById('bar-' + prayer).style.width = pct + '%';
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
