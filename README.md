<!DOCTYPE html>
<html lang="en">
<head>
<meta charset="UTF-8">
<title>Bridge Deck Optimizer</title>
<style>
:root{
  --bg:#402215; --surface:#523B28; --accent:#DBCAAE;
  --gold:#FFC857; --cyan:#5BD2E0; --green:#7DDB87;
  --muted:#A89272; --soft:#6A4A2F;
  --concrete:#A89178; --steel:#7C5A3A; --barrier:#8B6F47;
}
*{box-sizing:border-box}
body{margin:0;background:var(--bg);color:var(--accent);
     font-family:Calibri,system-ui,-apple-system,sans-serif;line-height:1.55}
h1,h2,h3{font-family:Georgia,serif;color:var(--accent);margin:0 0 .5em 0}
h1{font-size:2.4em}
h2{font-size:1.5em;border-bottom:1px solid var(--muted);padding-bottom:.3em;margin-top:1.2em}
h3{font-size:1.05em;margin-top:0}
.container{max-width:1200px;margin:0 auto;padding:30px}
.header{padding:20px 0;border-bottom:2px solid var(--muted);margin-bottom:20px}
.subtitle{color:var(--muted);font-size:0.95em}
.card{background:var(--surface);padding:22px 24px;border-radius:10px;margin:16px 0;
      box-shadow:0 2px 10px rgba(0,0,0,0.28)}
.grid{display:grid;grid-template-columns:repeat(auto-fit,minmax(260px,1fr));gap:14px}
.kv{display:flex;justify-content:space-between;padding:6px 0;
    border-bottom:1px dotted rgba(219,202,174,0.25)}
.kv:last-child{border-bottom:none}
.kv .k{color:var(--muted)}

label{display:flex;flex-direction:column;gap:6px;font-size:0.95em}
.field{background:var(--soft);padding:10px 12px;border-radius:8px}
.field-label{color:var(--muted);font-size:0.85em;text-transform:uppercase;letter-spacing:0.5px}
input[type="number"],input[type="text"]{
  width:100%;padding:8px 10px;border:1px solid var(--muted);border-radius:6px;
  background:#2D170D;color:var(--accent);font-family:inherit;font-size:1em;
}
input[type="number"]:focus,input[type="text"]:focus{
  outline:none;border-color:var(--gold);box-shadow:0 0 0 2px rgba(255,200,87,0.25);
}
input[type="range"]{width:100%;accent-color:var(--gold)}
.range-row{display:flex;gap:12px;align-items:center}
.range-readout{min-width:170px;text-align:right;font-weight:600;color:var(--gold)}
.btn{display:inline-block;padding:12px 26px;border:none;border-radius:8px;
     background:var(--gold);color:#402215;font-family:Georgia,serif;font-size:1.1em;
     font-weight:700;cursor:pointer;transition:transform .1s, box-shadow .1s}
.btn:hover{transform:translateY(-1px);box-shadow:0 4px 14px rgba(255,200,87,0.3)}
.btn:disabled{opacity:0.55;cursor:not-allowed;transform:none;box-shadow:none}
.btn-secondary{background:transparent;color:var(--accent);border:1px solid var(--muted)}
.btn-secondary:hover{background:rgba(219,202,174,0.08)}
.actions{display:flex;gap:12px;align-items:center;margin-top:18px;flex-wrap:wrap}

details{background:var(--soft);border-radius:8px;padding:0;margin-top:18px}
details summary{list-style:none;cursor:pointer;padding:14px 18px;font-weight:bold;
                font-family:Georgia,serif;color:var(--gold)}
details summary::-webkit-details-marker{display:none}
details summary::before{content:"▸ ";display:inline-block;margin-right:6px}
details[open] summary::before{content:"▾ "}
details[open] summary{border-bottom:1px solid rgba(219,202,174,0.2)}
details .grid{padding:16px 18px}

table{width:100%;border-collapse:collapse;margin-top:10px}
th,td{padding:8px 10px;text-align:left;border-bottom:1px solid rgba(219,202,174,0.25);
      font-size:0.92em}
th{color:var(--gold);font-weight:bold;font-family:Georgia,serif}
.tag{display:inline-block;padding:3px 9px;border-radius:4px;font-size:0.85em;font-weight:bold}
.tag-user  {background:var(--gold); color:#402215}
.tag-cost  {background:var(--cyan); color:#402215}
.tag-carbon{background:var(--green);color:#402215}

.scenarios-grid{display:grid;gap:18px;
                grid-template-columns:repeat(auto-fit,minmax(310px,1fr))}
.scenario-card{background:var(--bg);border:1px solid var(--muted);
               border-radius:10px;padding:14px;display:flex;flex-direction:column;
               gap:10px}
.scenario-card.user-card  {border-color:var(--gold)}
.scenario-card.cost-card  {border-color:var(--cyan)}
.scenario-card.carbon-card{border-color:var(--green)}
.scenario-head{display:flex;justify-content:space-between;align-items:center}
.scenario-title{font-family:Georgia,serif;font-weight:bold;font-size:1.1em}
.bignum{display:flex;justify-content:space-between;font-size:1.5em;
        font-family:Georgia,serif;font-weight:bold;margin:6px 0}
.bignum .lbl{font-size:0.6em;color:var(--muted);font-weight:normal;
             font-family:Calibri,sans-serif;text-transform:uppercase;letter-spacing:0.5px}
.spec-list{display:grid;grid-template-columns:1fr 1fr;gap:4px 12px;font-size:0.86em}
.spec-list .sk{color:var(--muted)}
.spec-list .sv{color:var(--accent);text-align:right}

.plot-wrap{position:relative;width:100%;background:var(--bg);border-radius:8px;
           padding:6px;box-shadow:inset 0 0 0 1px rgba(168,146,114,0.2)}
canvas.plot{display:block;width:100%;height:460px;background:transparent}
canvas.section-canvas{display:block;width:100%;height:240px;background:transparent}
.tooltip{position:absolute;pointer-events:none;background:#2D170D;color:var(--accent);
         border:1px solid var(--gold);padding:8px 10px;border-radius:6px;
         font-size:0.85em;line-height:1.4;display:none;white-space:nowrap;
         box-shadow:0 4px 14px rgba(0,0,0,0.5);z-index:10}

footer{margin-top:40px;padding-top:20px;border-top:1px solid var(--muted);
       color:var(--muted);font-size:0.85em}
footer ol{padding-left:20px}
.small{font-size:0.85em;color:var(--muted)}
.hidden{display:none !important}
.error{background:#5C2222;color:#FFD7D7;border-left:4px solid #FF6B6B;
       padding:14px 18px;border-radius:6px;margin:14px 0;white-space:pre-wrap;
       font-family:Consolas,Monaco,monospace;font-size:0.9em;max-height:280px;overflow:auto}

.spinner{width:60px;height:60px;border:5px solid rgba(219,202,174,0.2);
         border-top-color:var(--gold);border-radius:50%;animation:spin 1s linear infinite;
         margin:14px auto}
@keyframes spin{to{transform:rotate(360deg)}}
.progress-text{text-align:center;color:var(--gold);font-family:Georgia,serif;font-size:1.15em}
.progress-sub {text-align:center;color:var(--muted);font-size:0.9em;margin-top:4px}
</style>
</head>

<body>
<div class="container">

<div class="header">
  <h1>Bridge Deck Optimizer</h1>
  <p class="subtitle">
    Multi-Objective Optimization of RC Bridge Deck Design&nbsp;·&nbsp;
    Self-tuning Genetic Algorithm<br>
    The American University in Cairo · Construction Engineering · Fall 2026
  </p>
</div>

<!-- ─── INPUT FORM ────────────────────────────────────────────── -->
<div class="card" id="input-card">
  <h2>Inputs</h2>
  <p class="small">Provide the three basic parameters; optionally expand
    <i>Advanced</i> for finer control. Click <b>Run Optimization</b> to launch
    the GA — it will simultaneously optimise for <b>user-priority</b>,
    <b>pure cost</b>, and <b>pure carbon</b> in a single run.</p>

  <form id="input-form">
    <div class="grid">
      <div class="field"><label>
        <span class="field-label">Bridge length (m)  ·  5–500</span>
        <input type="number" name="bridge_length" min="5" max="500" step="0.1" value="30" required>
      </label></div>
      <div class="field"><label>
        <span class="field-label">Number of lanes (total)  ·  1–12</span>
        <input type="number" name="num_lanes" min="1" max="12" step="1" value="4" required>
      </label></div>
      <div class="field">
        <span class="field-label">Cost ↔ Carbon weight (%)</span>
        <div class="range-row">
          <input type="range" name="weight_pct" id="weight-range" min="0" max="100" step="1" value="50">
          <span class="range-readout" id="weight-readout">50% cost / 50% carbon</span>
        </div>
        <div class="small">0 = pure carbon · 100 = pure cost · 50 = balanced</div>
      </div>
    </div>

    <details>
      <summary>Advanced inputs (optional)</summary>
      <div class="grid">
        <div class="field"><label>
          <span class="field-label">Cantilever length (m)  ·  0.5–3.0</span>
          <input type="number" name="cantilever_length" min="0.5" max="3.0" step="0.05" value="1.5">
        </label></div>
        <div class="field"><label>
          <span class="field-label">Concrete cover (mm)  ·  20–75</span>
          <input type="number" name="cover_mm" min="20" max="75" step="1" value="40">
        </label></div>
        <div class="field"><label>
          <span class="field-label">AASHTO HL-93 wheel (kN)  ·  50–300</span>
          <input type="number" name="wheel_kN" min="50" max="300" step="1" value="145">
        </label></div>
        <div class="field"><label>
          <span class="field-label">Barrier SID per side (kN/m)  ·  0–10</span>
          <input type="number" name="sid_kN" min="0" max="10" step="0.1" value="1.5">
        </label></div>
        <div class="field"><label>
          <span class="field-label">Asphalt thickness (mm)  ·  0–200</span>
          <input type="number" name="asphalt_mm" min="0" max="200" step="1" value="80">
        </label></div>
        <div class="field"><label>
          <span class="field-label">Lane width (m)  ·  2.5–4.5</span>
          <input type="number" name="lane_width" min="2.5" max="4.5" step="0.05" value="3.65">
        </label></div>
        <div class="field"><label>
          <span class="field-label">Shoulder per side (m)  ·  0–2.0</span>
          <input type="number" name="shoulder" min="0" max="2.0" step="0.05" value="0.5">
        </label></div>
      </div>
    </details>

    <details>
      <summary>GA hyperparameters (advanced — leave defaults if unsure)</summary>
      <div class="grid">
        <div class="field"><label>
          <span class="field-label">Population size  ·  10–500</span>
          <input type="number" name="pop" min="10" max="500" step="1" value="50">
        </label></div>
        <div class="field"><label>
          <span class="field-label">Max generations  ·  5–500</span>
          <input type="number" name="max_gens" min="5" max="500" step="1" value="100">
        </label></div>
        <div class="field"><label>
          <span class="field-label">Stagnation limit  ·  3–100</span>
          <input type="number" name="stagnation_limit" min="3" max="100" step="1" value="15">
        </label></div>
        <div class="field"><label>
          <span class="field-label">Mutation probability  ·  0.0–1.0</span>
          <input type="number" name="mut" min="0" max="1" step="0.01" value="0.20">
        </label></div>
        <div class="field"><label>
          <span class="field-label">Crossover probability  ·  0.0–1.0</span>
          <input type="number" name="cx" min="0" max="1" step="0.01" value="0.80">
        </label></div>
        <div class="field"><label>
          <span class="field-label">Tournament size (k)  ·  2–20</span>
          <input type="number" name="tournament" min="2" max="20" step="1" value="3">
        </label></div>
        <div class="field"><label>
          <span class="field-label">Elite count  ·  0–10</span>
          <input type="number" name="elite" min="0" max="10" step="1" value="2">
        </label></div>
      </div>
    </details>

    <div class="actions">
      <button type="submit" class="btn" id="run-btn">Run Optimization</button>
      <button type="reset" class="btn btn-secondary" id="reset-btn">Reset to defaults</button>
    </div>
  </form>

  <div id="error-box" class="error hidden"></div>
</div>

<!-- ─── PROGRESS ────────────────────────────────────────────── -->
<div class="card hidden" id="progress-card">
  <h2>Optimizing…</h2>
  <div class="spinner"></div>
  <p class="progress-text" id="progress-text">Initialising in-browser optimizer…</p>
  <p class="progress-sub" id="progress-sub">Engine + GA run entirely in your browser. No data leaves this tab.</p>
</div>

<!-- ─── RESULTS ────────────────────────────────────────────── -->
<div class="hidden" id="results">
  <div class="card">
    <h2>Inputs used</h2>
    <div class="grid" id="inputs-echo"></div>
  </div>

  <div class="card">
    <h2>Three scenarios — visual comparison</h2>
    <p class="small">Each card shows the optimum cross-section the GA found for that
       objective. Drawn to scale: slab + cantilevers + main girders + cross-girder
       indication.</p>
    <div class="scenarios-grid" id="scenario-cards"></div>
  </div>

  <div class="card">
    <h2>Side-by-side numbers</h2>
    <div id="comparison-table"></div>
  </div>

  <div class="card">
    <h2>Pareto Front — Cost vs Carbon</h2>
    <div class="plot-wrap"><canvas id="cv-pareto" class="plot"></canvas><div class="tooltip" id="tt-pareto"></div></div>
    <p class="small">Each dot is a feasible design the GA explored. The three highlighted
       points are the chosen scenarios. Hover for details.</p>
  </div>

  <div class="card">
    <h2>GA Convergence</h2>
    <div class="plot-wrap"><canvas id="cv-convergence" class="plot"></canvas></div>
  </div>

  <div class="card">
    <h2>GA hyperparameters used</h2>
    <div id="hp-card"></div>
  </div>

  <div class="actions">
    <button class="btn" id="run-again-btn">Run again with new inputs</button>
  </div>
</div>

<footer>
<p>Made by Ahmed Tawfik, Ali Mohamed, Carla Farah, Jana Refaie</p>
</footer>
</div>

<noscript><p style="color:#FFC857;text-align:center">This page needs JavaScript.</p></noscript>

<script>
'use strict';

// ╔══════════════════════════════════════════════════════════════════════╗
// ║  EMBEDDED ENGINE  —  JS port of bridge_optimizer.py                  ║
// ║  Self-contained: no network, no server. Runs entirely in browser.    ║
// ╚══════════════════════════════════════════════════════════════════════╝

const CODE = {
  gamma_c: 25, gamma_asp: 22, gamma_pc: 22,
  t_asp_default: 80, t_pc: 200,
  SID_default: 1.5, ped_LL: 5.0,
  AASHTO_WHEEL_KN: 145,           // single source of truth — was 150 in xg
  IM_pct: 0.33, LF: 1.5, phi_f: 0.9,
  gamma_s: 1.15, gamma_cb: 1.5,
  cvr_slab: 40, phi_stirrup: 10,
  m_cont: 0.76, h_barrier: 1.2, E_cant: 2.6,
  P_min_wheel: 200,
  lane_ld: 2.5, truck_ld: 9.0, axle_gap: 1.2,
  bar_dia_assume: 22, b_mg: 400, b_xg: 300, ts_default: 250,
};
const FC_OPTIONS = [30, 35, 40, 50];
const FY_OPTIONS = [280, 400, 420, 550];     // 400 added (matches Python)
const S_MG_RANGE = [2.0, 3.0];
const S_XG_RANGE = [4.0, 6.0];
const K_MG_RANGE = [12.0, 14.0];
const K_XG_RANGE = [0.67, 0.75];
const COST_CONCRETE = {30: 90, 35: 105, 40: 120, 50: 140};
const COST_STEEL    = {280: 0.95, 400: 1.05, 420: 1.10, 550: 1.30};
const COST_FORMWORK = 10;
const CARBON_CONCRETE = {30: 260, 35: 300, 40: 340, 50: 375};
const CARBON_STEEL    = {280: 1.46, 400: 1.83, 420: 1.99, 550: 2.10};
const CARBON_FORMWORK = 0.18;
const BAR_TABLE = {10:78.5, 12:113.1, 16:201.1, 18:254.5, 20:314.2, 22:380.1, 25:490.9, 28:615.8, 32:804.2};

// ── Bar selection helpers ──
function pickBar(As_req, ts_mm, label, bar_dia) {
  const order = [10,12,16,18,20,22,25,28,32];
  const spLim = (d) => [Math.round(Math.max(1.5*d, 38)*10)/10,
                        Math.round(Math.min(1.5*ts_mm, 450)*10)/10];
  let dia = 32;
  if (bar_dia == null) {
    for (const d of order) {
      const Ab = BAR_TABLE[d];
      const n = Math.ceil(As_req/Ab);
      const s = Math.round(1000/n*10)/10;
      const [smn, smx] = spLim(d);
      if (smn <= s && s <= smx) { dia = d; break; }
    }
  } else dia = bar_dia;
  const Ab = BAR_TABLE[dia];
  const n = Math.ceil(As_req/Ab);
  const s = Math.round(1000/n*10)/10;
  const As_prov = Math.round(n*Ab*10)/10;
  const [smn, smx] = spLim(dia);
  return {dia, Ab, n, s, As_prov, s_min: smn, s_max: smx,
          spacing_check: (smn<=s && s<=smx) ? '✅ Spacing OK' : '❌ Adjust spacing',
          summary: `${n}Ø${dia}mm @ ${s}mm`};
}
function pickSlab(As, b, opts, mx, mn) {
  b = b || 1000; mx = mx || 300; mn = mn || 75;
  opts = opts || [10,12,16,18,20,22,25,28,32];
  for (const dia of opts) {
    const Ab = Math.PI*dia*dia/4;
    const n = Math.ceil(As/Ab);
    const s = Math.max(mn, Math.min(Math.floor(b/n/25)*25, mx));
    if (Math.ceil(b/s)*Ab >= As) return [dia, s, Math.ceil(b/s)*Ab];
  }
  const dia = opts[opts.length-1];
  const Ab = Math.PI*dia*dia/4;
  const na = Math.ceil(b/mn);
  return [dia, mn, na*Ab];
}
function pickBeam(As_req, bw, cover, stir_dia) {
  // Multi-layer beam picker (RESTORED — was dead in original engine)
  stir_dia = stir_dia || 10;
  const opts = [10,12,16,18,20,22,25,28,32];
  const aw = bw - 2*cover - 2*stir_dia;
  for (const dia of opts) {
    const Ab = Math.PI*dia*dia/4;
    const n = Math.ceil(As_req/Ab);
    const clear = Math.max(dia, 25);
    const per_layer = Math.max(1, Math.floor((aw + clear)/(dia + clear)));
    const layers = Math.ceil(n/per_layer);
    if (n*Ab >= As_req && layers <= 4) {
      return {dia, n, per_layer, layers, Ab, As_prov: n*Ab,
              summary: `${n}Ø${dia}mm in ${layers} layer(s) of ${per_layer}`};
    }
  }
  const dia = 32, Ab = Math.PI*dia*dia/4;
  const n = Math.ceil(As_req/Ab);
  const clear = Math.max(dia, 25);
  const per_layer = Math.max(1, Math.floor((aw + clear)/(dia + clear)));
  const layers = Math.ceil(n/per_layer);
  return {dia, n, per_layer, layers, Ab, As_prov: n*Ab,
          summary: `${n}Ø${dia}mm in ${layers} layer(s) of ${per_layer}`};
}

// ── AASHTO iterative flexural design ──
function designAASHTO(Mu, b, d, fc, fy, As_min, ts_mm, top_dia, btm_dia, phi, tol, max_iter, As_init) {
  phi = phi || 0.9; tol = tol || 0.01; max_iter = max_iter || 15;
  if (Mu <= 0) {
    return {As: As_min, a:0, c:0, phi_Mn:0, eps_t:0.005,
            ductility:'tension controlled ✓', converged:true, n_iter:0,
            top: pickBar(As_min, ts_mm, 'TOP', top_dia),
            btm: pickBar(As_min, ts_mm, 'BOTTOM', btm_dia)};
  }
  const beta1 = 0.85;
  let As_iter = (As_init && As_init > 0) ? As_init : (Mu*1e6)/(phi*fy*0.9*d);
  let conv=false, n_iter=1, a=0, Mn=0, phi_Mn=0;
  for (n_iter=2; n_iter <= max_iter; n_iter++) {
    a = (As_iter*fy)/(0.85*fc*b);
    Mn = As_iter*fy*(d-a/2)/1e6;
    phi_Mn = phi*Mn;
    const As_new = (Mu*1e6)/(phi*fy*(d-a/2));
    if (Math.abs(phi_Mn - Mu)/Mu < tol) { As_iter = As_new; conv = true; break; }
    As_iter = As_new;
  }
  const As = Math.max(As_iter, As_min);
  a = (As*fy)/(0.85*fc*b);
  const c = a/beta1;
  Mn = As*fy*(d-a/2);
  phi_Mn = phi*Mn/1e6;
  const eps_t = 0.003*(d-c)/c;
  const ductility = eps_t >= 0.005 ? 'tension controlled ✓'
                   : eps_t >= 0.004 ? 'transition zone ⚠'
                   : 'compression controlled ✗';
  return {As, a, c, phi_Mn, eps_t, ductility, converged: conv, n_iter,
          top: pickBar(As, ts_mm, 'TOP', top_dia),
          btm: pickBar(As_min, ts_mm, 'BOTTOM', btm_dia)};
}

// ── BMD solver (general n>=2) ──
function getDeckSpans(n, L_int, L_cant) {
  if (n < 2) throw new Error('Need at least 2 girders');
  return {L_ba: L_int, L_bc: L_cant, ba_far_free: false, bc_far_free: true,
          critical_joint: 'B (first interior girder)'};
}
function green(xi, xj, L) {
  const a = Math.min(xi, xj), b = Math.max(xi, xj);
  if (L === 0) return 0;
  return a*(L-b)/(6*L) * (L*L - (L-b)*(L-b) - a*a);
}
function d0_udl(xi, aL, aR, w_span, L) {
  if (w_span === 0 || aR <= aL) return 0;
  let res = 0;
  let lo = aL, hi = Math.min(aR, xi);
  if (hi > lo) {
    const c = (L-xi)/(6*L)*w_span;
    res += c * ((2*L*xi - xi*xi)*(hi*hi - lo*lo)/2 - (Math.pow(hi,4) - Math.pow(lo,4))/4);
  }
  let lo2 = Math.max(aL, xi), hi2 = aR;
  if (hi2 > lo2) {
    const c2 = xi/(6*L)*w_span;
    const F = s => L*L*s*s - L*Math.pow(s,3) - L*xi*xi*s + Math.pow(s,4)/4 + xi*xi*s*s/2;
    res += c2*(F(hi2) - F(lo2));
  }
  return res;
}
function solveLin(A, b) {
  const n = A.length;
  const M = A.map((r,i) => [...r, b[i]]);
  for (let i = 0; i < n; i++) {
    let mr = i;
    for (let k = i+1; k < n; k++) if (Math.abs(M[k][i]) > Math.abs(M[mr][i])) mr = k;
    [M[i], M[mr]] = [M[mr], M[i]];
    if (Math.abs(M[i][i]) < 1e-15) throw new Error('singular');
    for (let k = i+1; k < n; k++) {
      const f = M[k][i]/M[i][i];
      for (let j = i; j <= n; j++) M[k][j] -= f*M[i][j];
    }
  }
  const x = new Array(n).fill(0);
  for (let i = n-1; i >= 0; i--) {
    let s = M[i][n];
    for (let j = i+1; j < n; j++) s -= M[i][j]*x[j];
    x[i] = s/M[i][i];
  }
  return x;
}
function solveDeckSlabDL(n, Ls, Lc, w, wc, SID) {
  if (n < 2) throw new Error('n must be >= 2');
  const Lss = (n-1)*Ls;
  const P_end = wc*Lc + SID;
  let R;
  if (n === 2) {
    R = [0.5*w*Ls + P_end, 0.5*w*Ls + P_end];
  } else {
    const n_red = n - 2;
    const x_red = []; for (let i=1; i<=n_red; i++) x_red.push(i*Ls);
    const F_mat = x_red.map(xi => x_red.map(xj => green(xi, xj, Lss)));
    const d0 = x_red.map(xi => {
      let v = 0;
      for (let k = 0; k < n-1; k++) v += d0_udl(xi, k*Ls, (k+1)*Ls, w, Lss);
      v += P_end * green(xi, 0, Lss);
      v += P_end * green(xi, Lss, Lss);
      return v;
    });
    const R_red = solveLin(F_mat, d0);
    const R_Gn = (w*Lss*Lss/2 + P_end*Lss - x_red.reduce((s,xi,i)=>s+R_red[i]*xi, 0))/Lss;
    const R_G1 = w*Lss + 2*P_end - R_Gn - R_red.reduce((a,b)=>a+b, 0);
    R = [R_G1, ...R_red, R_Gn];
  }
  const M_cant = -(wc*Lc*Lc/2 + SID*Lc);
  const M_at = (k) => {
    const x = k*Ls;
    let M = R[0]*x - P_end*x - w*x*x/2;
    for (let j = 1; j < k; j++) M += R[j]*(x - j*Ls);
    return M;
  };
  const M_supports = [];
  for (let k = 0; k < n; k++) M_supports.push(Math.round(M_at(k)*1e10)/1e10);
  const shear_left = (span_idx) => {
    let v = R[0] - P_end;
    for (let k = 1; k <= span_idx; k++) v += R[k] - w*Ls;
    return v;
  };
  const M_peaks = [];
  for (let span = 0; span < n-1; span++) {
    const V0 = shear_left(span), M_left = M_supports[span];
    M_peaks.push(V0 > 0 ? M_left + V0*V0/(2*w) : M_left);
  }
  return {reactions: R, M_cant, M_supports, M_peaks,
          M_hog: Math.min(...M_supports), M_sag: Math.max(...M_peaks)};
}

// ── Design phases ──
function designCantilever(Lc_m, fc, fy, cover_mm) {
  const ts_mm = CODE.ts_default;
  const tpc_m = CODE.t_pc/1000;
  const w_cant_sw = (ts_mm/1000)*CODE.gamma_c + tpc_m*CODE.gamma_pc;
  const ws_c1 = w_cant_sw + CODE.ped_LL;
  const M_c1 = ws_c1*Lc_m*Lc_m/2 + CODE.SID_default*CODE.h_barrier + CODE.SID_default*Lc_m;
  const P_wh = Math.max(CODE.AASHTO_WHEEL_KN*(1+CODE.IM_pct), CODE.P_min_wheel);
  const a_wh = Math.min(Lc_m - 0.2, 1.3);
  const M_c2 = w_cant_sw*Lc_m*Lc_m/2 + P_wh*a_wh/CODE.E_cant;
  const M_unfact = Math.max(M_c1, M_c2);
  const Mu = CODE.LF*M_unfact;
  let ts_cant = ts_mm;
  const d_assume = CODE.bar_dia_assume/2;
  let d_cant = ts_cant - cover_mm - CODE.phi_stirrup - d_assume;
  const As_est = (Mu*1e6)/(0.9*0.9*d_cant*fy);
  const a_est = (As_est*fy)/(0.85*fc*1000);
  const d_req = a_est/2 + Math.sqrt((a_est/2)*(a_est/2) + (Mu*1e6)/(0.85*fc*1000));
  if (d_cant < d_req) {
    ts_cant = Math.ceil((d_req + cover_mm + d_assume)/25)*25;
    d_cant = ts_cant - cover_mm - CODE.phi_stirrup - d_assume;
  }
  const As_min = Math.max(0.002*1000*ts_cant, Math.max(0.0018, 1.4/fy)*1000*d_cant);
  const r = designAASHTO(Mu, 1000, d_cant, fc, fy, As_min, ts_cant, 22, 16);
  return {ts: ts_cant, d: d_cant, M_c1, M_c2, M_unfact, Mu,
          As_min, As_req: r.As, a: r.a, c: r.c, phi_Mn: r.phi_Mn,
          eps_t: r.eps_t, ductility: r.ductility, converged: r.converged,
          top_bar: r.top, btm_bar: r.btm,
          flex_check: r.phi_Mn >= Mu ? 'PASS ✓' : 'FAIL ✗'};
}

function designSlab(n_g, Ls_m, Lc_m, fc, fy, cover_mm) {
  const ts_mm = CODE.ts_default, ts_m = ts_mm/1000;
  const ta_m = CODE.t_asp_default/1000, tpc_m = CODE.t_pc/1000;
  const Ls_mm = Ls_m*1000;
  const w_DC = ts_m*CODE.gamma_c + ta_m*CODE.gamma_asp;
  const w_bc = ts_m*CODE.gamma_c + tpc_m*CODE.gamma_pc;
  const spans = getDeckSpans(n_g, Ls_m, Lc_m);     // restored helper
  const bmd = solveDeckSlabDL(n_g, Ls_m, Lc_m, w_DC, w_bc, CODE.SID_default);
  const M_DL_neg = bmd.M_hog, M_DL_pos = bmd.M_sag;
  const S1 = 0.35 + ta_m + ts_m, S2 = 0.60 + ta_m + ts_m;
  const Be = Math.min(S1+1.8, S1+0.2*CODE.m_cont*Ls_m);
  const w_LL = 200/(S2*Be), ap = S2;
  const R_ss = w_LL*ap*ap/2;
  const M_LL = Math.max(Math.abs(-R_ss), Math.abs(R_ss*Ls_m/2 - w_LL*ap*ap/8));
  const Mu_bot = CODE.LF*M_DL_pos + CODE.LF*M_LL;
  const Mu_top = CODE.LF*Math.abs(M_DL_neg) + CODE.LF*M_LL*0.8;
  const d_s = ts_mm - cover_mm - 11 - 10;
  const As_min = Math.max(0.002*1000*ts_mm, Math.max(0.0018, 1.4/fy)*1000*d_s);
  const pct = Math.min(67, 3840/Math.sqrt(Ls_mm));
  const bot = designAASHTO(Mu_bot, 1000, d_s, fc, fy, As_min, ts_mm, 22, 16);
  const top = designAASHTO(Mu_top, 1000, d_s, fc, fy, As_min, ts_mm, 22, 16);
  const _arr = (As_dir, dia) => {
    const Ab = BAR_TABLE[dia];
    let n = Math.ceil(As_dir/Ab);
    let s = Math.ceil((1000/n)/5)*5;
    n = Math.ceil(1000/s);
    return {dia, n, s, As: Math.round(n*Ab*10)/10};
  };
  const by = pickSlab(Math.max(bot.As*pct/100, As_min));
  const ty = pickSlab(Math.max(top.As*pct/100, As_min));
  return {ts: ts_mm, d: d_s, spans, bmd, M_LL,
          M_DL_pos, M_DL_neg, M_DL_cant: bmd.M_cant,
          Mu_bot, Mu_top, As_min, pct_dist: pct,
          As_bot: bot.As, As_top: top.As,
          phi_Mn_bot: bot.phi_Mn, phi_Mn_top: top.phi_Mn,
          eps_t_bot: bot.eps_t, eps_t_top: top.eps_t,
          duct_bot: bot.ductility, duct_top: top.ductility,
          conv_bot: bot.converged, conv_top: top.converged,
          flex_check_bot: bot.phi_Mn >= Mu_bot ? 'PASS ✓' : 'FAIL ✗',
          flex_check_top: top.phi_Mn >= Mu_top ? 'PASS ✓' : 'FAIL ✗',
          bot_x: _arr(bot.As, bot.top.dia),
          bot_y: _arr(Math.max(bot.As*pct/100, As_min), by[0]),
          top_x: _arr(top.As, top.top.dia),
          top_y: _arr(Math.max(top.As*pct/100, As_min), ty[0])};
}

function computeILLoads(n_g, Ls_m, Lc_m, ts_mm, h_xg_mm, b_xg) {
  const ts_m = ts_mm/1000, ta_m = CODE.t_asp_default/1000, tpc_m = CODE.t_pc/1000;
  const n_int = n_g - 1;
  const w_DC = ts_m*CODE.gamma_c + ta_m*CODE.gamma_asp;
  const w_cant_sw = ts_m*CODE.gamma_c + tpc_m*CODE.gamma_pc;
  const R1 = (2*CODE.SID_default + w_cant_sw*2*Lc_m + w_DC*Ls_m*n_int)/n_g;
  const ow_xg = CODE.gamma_c*(b_xg/1000)*(h_xg_mm-ts_mm)/1000;
  const P1 = ow_xg*Ls_m*n_int/n_g;
  const positions = [];
  for (let i = 0; i < n_g; i++) positions.push((i - (n_g-1)/2)*Ls_m);
  const Sa2 = 2 * positions.filter(p => p > 0).reduce((s,p)=>s+p*p, 0);
  const IL = (a, b) => Sa2 === 0 ? 1/n_g : 1/n_g + (a*b)/Sa2;
  const cant_right = positions[positions.length-1] + Lc_m;
  const cant_left = positions[0] - Lc_m;
  const wheel_heavy = CODE.AASHTO_WHEEL_KN + 5;     // ≈150 (kept for parity)
  const wheel_light = Math.max(0, wheel_heavy - 50);
  const R2_all = [], P2_all = [];
  for (const gp of positions) {
    let r2 = 0;
    for (const lp of positions) r2 += (CODE.lane_ld + CODE.truck_ld)*Ls_m*Math.max(0, IL(gp, lp));
    for (const tip of [cant_right, cant_left]) {
      const io = IL(gp, tip);
      if (io > 0) r2 += CODE.ped_LL*Lc_m*io;
    }
    R2_all.push(r2);
    const ords = positions.map(lp => IL(gp, lp)).sort((a,b)=>b-a);
    const p2 = ords.slice(0,2).reduce((s,o)=>s + wheel_heavy*Math.max(0,o), 0)
             + ords.slice(2,4).reduce((s,o)=>s + wheel_light*Math.max(0,o), 0);
    P2_all.push(p2);
  }
  return {R1, P1, ow_xg, R2_all, P2_all, positions, Sa2};
}

function designMainGirders(n_g, sp_m, h_mg, b_mg, ts_mm, num_xg, fc, fy, cover_mm, Ls_m, IL) {
  const bar_dia = CODE.bar_dia_assume;
  const Ab22 = Math.PI*bar_dia*bar_dia/4;
  const sp_mm = sp_m*1000;
  const out = [];
  for (let gi = 0; gi < n_g; gi++) {
    const R2 = IL.R2_all[gi], P2 = IL.P2_all[gi];
    const h = h_mg, bw = b_mg;
    const B = Math.min(16*ts_mm + bw, Math.floor(Ls_m*1000), Math.floor(sp_mm/5 + bw));
    const ow_g = CODE.gamma_c*(bw/1000)*(h-ts_mm)/1000;
    const wd = IL.R1 + ow_g;
    let d_g = Math.floor(h - cover_mm - CODE.phi_stirrup - bar_dia/2);
    const R_DL = (wd*sp_m + IL.P1*num_xg)/2;
    const M_DL = wd*sp_m*sp_m/8;
    const R_LL = (R2*sp_m + 2*P2)/2;
    const M_LL = R_LL*(sp_m/2) - P2*(CODE.axle_gap/2) - R2*(sp_m/2)*(sp_m/2)/2;
    const Mu = Math.max(0, CODE.LF*(M_DL + M_LL));
    const As_min = Math.max(0.0033, 1.4/fy)*bw*d_g;
    const a_seed = Ab22*fy/(0.85*fc*B);
    const lever = d_g - a_seed/2;
    const As_init = Mu > 0 ? (Mu*1e6)/(CODE.phi_f*fy*lever) : null;
    const r = designAASHTO(Mu, B, d_g, fc, fy, As_min, h, bar_dia, null, CODE.phi_f, undefined, undefined, As_init);
    const beam = pickBeam(r.As, bw, cover_mm, CODE.phi_stirrup);  // restored helper
    const n_bars = beam.n, layers = beam.layers;
    if (layers > 1) d_g = Math.floor(d_g - (layers-1)*(bar_dia + 25)/2);
    const Ap = n_bars*Ab22;
    const ab = Ap*fy/(0.85*fc*B);
    let pMn, tsec;
    if (ab <= ts_mm) {
      pMn = CODE.phi_f*Ap*fy*(d_g - ab/2)/1e6;
      tsec = 'rectangular';
    } else {
      const Cf = 0.85*fc*(B-bw)*ts_mm;
      const Cw = Ap*fy - Cf;
      const aw = Cw/(0.85*fc*bw);
      pMn = CODE.phi_f*(Cf*(d_g-ts_mm/2) + Cw*(d_g-aw/2))/1e6;
      tsec = 'T-section';
    }
    const flex = pMn >= Mu ? 'PASS' : 'FAIL';
    const Qu = CODE.LF*(R_DL + R_LL);
    const qsu = Qu*1e3/(bw*d_g);
    const qcu = 0.24*Math.sqrt(fc/CODE.gamma_cb);
    const qmx = 0.70*Math.sqrt(fc/CODE.gamma_cb);
    const qn = qsu - qcu/2;
    const n_legs = 4, sd = 16;
    const Av = n_legs*Math.PI*sd*sd/4;
    let ss;
    if (qn > 0) {
      const ss_raw = Av*fy/(CODE.gamma_s*bw*qn);
      ss = Math.max(75, Math.min(Math.floor(ss_raw/25)*25, Math.floor(d_g/2), 300));
    } else ss = Math.floor(Math.min(d_g/2, 300));
    const qsv = Av*fy/(CODE.gamma_s*bw*ss);
    const shear = (qsu <= qmx && (qn <= 0 || qsv >= qn)) ? 'PASS' : 'FAIL';
    out.push({idx: gi, width: bw, depth: h, B_eff: B, ow_g, wd, d_g,
              R_DL, M_DL_g: M_DL, R2, P2, R_LL, M_LL_g: M_LL, Mu_g: Mu,
              As_min, As_req: r.As, long_dia: bar_dia, long_n: n_bars, layers,
              Ap_prov: Ap, ab, phi_Mn: pMn, t_section: tsec, flex_check: flex,
              Qu, qsu, qcu, qmax: qmx, qn,
              stirrup_dia: sd, stirrup_s: ss, n_legs, Av, qsv_prov: qsv,
              shear_check: shear,
              eps_t: r.eps_t, ductility: r.ductility, converged: r.converged});
  }
  return out;
}

function designCrossGirder(Ls_m, Sb_m, h_xg, b_xg, ts_mm, fc, fy, cover_mm, num_xg) {
  const Lx = Ls_m;
  const bar_dia = CODE.bar_dia_assume;
  const d_xg = Math.floor(h_xg - cover_mm - CODE.phi_stirrup - bar_dia/2);
  const ow_xg = CODE.gamma_c*(b_xg/1000)*(h_xg-ts_mm)/1000;
  const w_DC = (ts_mm/1000)*CODE.gamma_c + (CODE.t_asp_default/1000)*CODE.gamma_asp;
  const wD = ow_xg + w_DC*Sb_m;
  const wL = (CODE.lane_ld + CODE.truck_ld)*Sb_m;
  const Pt = CODE.AASHTO_WHEEL_KN*(1+CODE.IM_pct);     // single source of truth
  const MD = wD*Lx*Lx/8;
  const ML = wL*Lx*Lx/8 + Pt*Lx/4;
  const Mu = CODE.LF*(MD + ML);
  const As_min = Math.max(0.0033, 1.4/fy)*b_xg*d_xg;
  const Ab22 = Math.PI*bar_dia*bar_dia/4;
  const a_seed = Ab22*fy/(0.85*fc*b_xg);
  const lever = d_xg - a_seed/2;
  const As_init = Mu > 0 ? (Mu*1e6)/(CODE.phi_f*fy*lever) : null;
  const r = designAASHTO(Mu, b_xg, d_xg, fc, fy, As_min, h_xg, bar_dia, null, CODE.phi_f, undefined, undefined, As_init);
  const beam = pickBeam(r.As, b_xg, cover_mm, CODE.phi_stirrup);
  const n_bars = beam.n, layers = beam.layers;
  const Ap = n_bars*Ab22;
  const ab = Ap*fy/(0.85*fc*b_xg);
  const pMn = CODE.phi_f*Ap*fy*(d_xg - ab/2)/1e6;
  const flex = pMn >= Mu ? 'PASS' : 'FAIL';
  const VD = wD*Lx/2;
  const VL = wL*Lx/2 + Pt;
  const Vu = CODE.LF*(VD + VL);
  const qsu = Vu*1e3/(b_xg*d_xg);
  const qcu = 0.24*Math.sqrt(fc/CODE.gamma_cb);
  const qmx = 0.70*Math.sqrt(fc/CODE.gamma_cb);
  const qn = qsu - qcu/2;
  const n_legs = 2, sd = 10;
  const Av = n_legs*Math.PI*sd*sd/4;
  let ss;
  if (qn > 0) {
    const ss_raw = Av*fy/(CODE.gamma_s*b_xg*qn);
    ss = Math.max(75, Math.min(Math.floor(ss_raw/25)*25, Math.floor(d_xg/2), 300));
  } else ss = Math.floor(Math.min(d_xg/2, 300));
  const qsv = Av*fy/(CODE.gamma_s*b_xg*ss);
  const shear = (qsu <= qmx && (qn <= 0 || qsv >= qn)) ? 'PASS' : 'FAIL';
  return {depth: h_xg, width: b_xg, d_xg, Lx, Sb_m, ow_xg, wD_xg: wD, wL_xg: wL,
          Pt_xg: Pt, MD_xg: MD, ML_xg: ML, Mu_xg: Mu, As_min, As_req: r.As,
          long_dia: bar_dia, long_n: n_bars, layers, Ap_prov: Ap, ab_xg: ab,
          phi_Mn: pMn, flex_check: flex,
          VD_xg: VD, VL_xg: VL, Vu_xg: Vu,
          qsu, qcu, qmax: qmx, qn, stirrup_dia: sd, stirrup_legs: n_legs,
          stirrup_s: ss, Av, qsv_prov: qsv, shear_check: shear,
          eps_t: r.eps_t, ductility: r.ductility, converged: r.converged};
}

function analyzeDeck(IN, k_mg, k_xg) {
  const [bridge_length, num_spans, main_girder_spacing, num_main_girders,
         secondary_beam_spacing, num_secondary_beams, fc, fy, cover, cant_len] = IN;
  const L_m = bridge_length/1000;
  const sp_m = bridge_length/num_spans/1000;
  const Ls_m = main_girder_spacing/1000;
  const Lc_m = cant_len/1000;
  const Sb_m = secondary_beam_spacing/1000;
  const n_g = num_main_girders;
  const ts_mm = CODE.ts_default;
  const raw_hmg = L_m / k_mg;
  const h_mg = 1000*Math.ceil(raw_hmg/0.5)*0.5;
  const raw_hxg = k_xg*h_mg;
  const h_xg = Math.ceil(raw_hxg/0.05)*0.05;
  const b_mg = CODE.b_mg, b_xg = CODE.b_xg;
  if (h_mg <= ts_mm || h_xg <= ts_mm) throw new Error('Girder depth ≤ slab thickness');
  const cantilever = designCantilever(Lc_m, fc, fy, cover);
  const slab = designSlab(n_g, Ls_m, Lc_m, fc, fy, cover);
  const IL = computeILLoads(n_g, Ls_m, Lc_m, ts_mm, h_xg, b_xg);
  const main_girders = designMainGirders(n_g, sp_m, h_mg, b_mg, ts_mm,
                                          num_secondary_beams, fc, fy, cover, Ls_m, IL);
  const cross_girder = designCrossGirder(Ls_m, Sb_m, h_xg, b_xg, ts_mm,
                                          fc, fy, CODE.cvr_slab, num_secondary_beams);
  return {cantilever, slab, main_girders, cross_girder, IL_loads: IL,
          derived: {L_m, sp_m, Ls_m, Lc_m, Sb_m, h_mg_mm: h_mg, h_xg_mm: h_xg,
                    b_mg, b_xg, ts_slab: ts_mm, ts_cant: cantilever.ts,
                    n_g, num_secondary_beams, main_girder_spacing_mm: main_girder_spacing,
                    fc_prime: fc, fy: fy, cover_mm: cover, IN}};
}

// ── Cost / carbon ──
function computeCostCarbon(result, ui) {
  const der = result.derived;
  const h_mg = der.h_mg_mm, h_xg = der.h_xg_mm;
  const n_g = der.n_g, n_xg = der.num_secondary_beams;
  const fc = der.fc_prime, fy = der.fy;
  const L_m = ui.bridge_length, Bw_m = ui.deck_width;
  const Lc_m = ui.cantilever_length, Int_m = ui.interior_m;
  const spans = ui.num_spans;
  const ts_slab = der.ts_slab, ts_cant = der.ts_cant;
  const b_mg = der.b_mg, b_xg = der.b_xg;
  const A_int = (Bw_m - 2*Lc_m)*L_m, A_cant = 2*Lc_m*L_m;
  const V_slab = A_int*(ts_slab/1000) + A_cant*(ts_cant/1000);
  const V_mg = n_g*((h_mg - ts_slab)*b_mg)*1e-6*L_m;
  const V_xg_each = ((h_xg - ts_slab)*b_xg)*1e-6*Int_m;
  const V_xg = n_xg*spans*V_xg_each;
  const V_total = V_slab + V_mg + V_xg;
  const rho = 7850;
  const slab = result.slab, cant = result.cantilever;
  const mgs = result.main_girders, xg = result.cross_girder;
  const As_slab = slab.bot_x.As + slab.bot_y.As + slab.top_x.As + slab.top_y.As;
  const M_slab_steel = (As_slab*1e-6)*Bw_m*L_m*rho;
  const As_cant = cant.top_bar.As_prov + cant.btm_bar.As_prov;
  const M_cant_steel = (As_cant*1e-6)*(2*Lc_m)*L_m*rho;
  let M_mg_long = 0, M_mg_stir = 0;
  for (const mg of mgs) {
    M_mg_long += mg.Ap_prov*1e-6*L_m*rho;
    const d_st = mg.stirrup_dia, sp_st = mg.stirrup_s;
    const n_st = Math.max(1, Math.round(L_m*1000/Math.max(50, sp_st)));
    const perim = 2*(b_mg + (h_mg - 50))*1e-3;
    const A_bar = Math.PI*d_st*d_st/4*1e-6;
    M_mg_stir += A_bar*perim*n_st*rho;
  }
  const M_mg_steel = M_mg_long + M_mg_stir;
  const M_xg_long = xg.Ap_prov*1e-6*Int_m*n_xg*spans*rho;
  const d_st_xg = xg.stirrup_dia, sp_st_xg = xg.stirrup_s;
  const n_st_xg = Math.max(1, Math.round(Int_m*1000/Math.max(50, sp_st_xg)));
  const perim_xg = 2*(b_xg + (h_xg - 50))*1e-3;
  const A_bar_xg = Math.PI*d_st_xg*d_st_xg/4*1e-6;
  const M_xg_stir = A_bar_xg*perim_xg*n_st_xg*n_xg*spans*rho;
  const M_xg_steel = M_xg_long + M_xg_stir;
  const M_steel = M_slab_steel + M_cant_steel + M_mg_steel + M_xg_steel;
  const A_form = (Bw_m - 2*Lc_m)*L_m + 2*Lc_m*L_m
              + n_g*2*((h_mg - ts_slab)/1000)*L_m + n_g*(b_mg/1000)*L_m
              + n_xg*spans*2*((h_xg - ts_slab)/1000)*Int_m + n_xg*spans*(b_xg/1000)*Int_m;
  const M_form = A_form*(500*0.020*0.4);
  const cost = V_total*COST_CONCRETE[fc] + M_steel*COST_STEEL[fy] + A_form*COST_FORMWORK;
  const carbon = V_total*CARBON_CONCRETE[fc] + M_steel*CARBON_STEEL[fy] + M_form*CARBON_FORMWORK;
  return {cost, carbon,
    q: {V_concrete_m3: V_total, M_steel_kg: M_steel, A_formwork_m2: A_form,
        cost_concrete: V_total*COST_CONCRETE[fc],
        cost_steel: M_steel*COST_STEEL[fy],
        cost_formwork: A_form*COST_FORMWORK,
        carbon_concrete: V_total*CARBON_CONCRETE[fc],
        carbon_steel: M_steel*CARBON_STEEL[fy],
        carbon_formwork: M_form*CARBON_FORMWORK}};
}

// ── GA ──
function makeRng(seed) {
  let a = seed >>> 0;
  const rnd = () => {
    a |= 0; a = a + 0x6D2B79F5 | 0;
    let t = a;
    t = Math.imul(t ^ t >>> 15, t | 1);
    t ^= t + Math.imul(t ^ t >>> 7, t | 61);
    return ((t ^ t >>> 14) >>> 0)/4294967296;
  };
  return {rand: rnd,
    randint: (lo, hi) => Math.floor(lo + rnd()*(hi-lo+1)),
    uniform: (lo, hi) => lo + rnd()*(hi-lo),
    gauss: (mu, sigma) => {
      mu = mu||0; sigma = sigma==null?1:sigma;
      let u1 = 0, u2 = 0;
      while (u1 === 0) u1 = rnd();
      while (u2 === 0) u2 = rnd();
      return mu + sigma*Math.sqrt(-2*Math.log(u1))*Math.cos(2*Math.PI*u2);
    },
    sample: (n, k) => {
      const arr = Array.from({length: n}, (_, i) => i);
      for (let i = n-1; i > 0; i--) {
        const j = Math.floor(rnd()*(i+1));
        [arr[i], arr[j]] = [arr[j], arr[i]];
      }
      return arr.slice(0, k);
    }};
}
function randomChrom(rng) {
  return [rng.uniform(...S_MG_RANGE), rng.uniform(...S_XG_RANGE),
          rng.uniform(...K_MG_RANGE), rng.uniform(...K_XG_RANGE),
          rng.randint(0, FC_OPTIONS.length-1), rng.randint(0, FY_OPTIONS.length-1)];
}
const cacheMap = new Map();
function cacheKey(c) {
  return [Math.round(c[0]*1000)/1000, Math.round(c[1]*1000)/1000,
          Math.round(c[2]*1000)/1000, Math.round(c[3]*1000)/1000,
          Math.round(c[4]), Math.round(c[5])].join('|');
}
function evalChrom(chrom, ui) {
  const [s_mg, s_xg, k_mg, k_xg, fc_idx, fy_idx] = chrom;
  const fc = FC_OPTIONS[Math.round(fc_idx)];
  const fy = FY_OPTIONS[Math.round(fy_idx)];
  if (ui.bridge_length < 5 || ui.num_spans < 1 || ui.interior_m <= 0) return null;
  const n_int_spans = Math.max(1, Math.ceil(ui.interior_m/s_mg));
  const n_main = Math.max(2, n_int_spans + 1);
  if (n_main < 2 || n_main > 16) return null;
  const main_sp = Math.round(ui.interior_m*1000/(n_main - 1));
  const num_xg = Math.max(1, Math.ceil(ui.span_length/s_xg) - 1);
  if (num_xg < 1 || num_xg > 20) return null;
  const sec_sp = Math.round(ui.span_length*1000/(num_xg + 1));
  const h_mg_pre = ui.bridge_length/k_mg;
  const h_xg_pre = k_xg*h_mg_pre;
  if (h_mg_pre < 1.2*h_xg_pre) return null;
  if (h_mg_pre*1000 <= 250 || h_xg_pre*1000 <= 250) return null;
  const IN = [Math.round(ui.bridge_length*1000), ui.num_spans, main_sp, n_main,
              sec_sp, num_xg, fc, fy, ui.cover_mm,
              Math.round(ui.cantilever_length*1000)];
  let result;
  try { result = analyzeDeck(IN, k_mg, k_xg); }
  catch (e) { return null; }
  let viol = 0;
  const cant = result.cantilever, slab = result.slab;
  const mgs = result.main_girders, xg = result.cross_girder;
  if (!String(cant.flex_check||'').includes('PASS')) viol++;
  if (!String(slab.flex_check_bot||'').includes('PASS')) viol++;
  if (!String(slab.flex_check_top||'').includes('PASS')) viol++;
  if (cant.ductility !== 'tension controlled ✓') viol++;
  if (slab.duct_top !== 'tension controlled ✓') viol++;
  if (slab.duct_bot !== 'tension controlled ✓') viol++;
  if (cant.converged === false) viol++;
  if (slab.conv_top === false) viol++;
  if (slab.conv_bot === false) viol++;
  for (const mg of mgs) {
    if (mg.flex_check !== 'PASS') viol++;
    if (mg.shear_check !== 'PASS') viol++;
    if (mg.ductility !== 'tension controlled ✓') viol++;
    if (mg.converged === false) viol++;
  }
  if (xg.flex_check !== 'PASS') viol++;
  if (xg.shear_check !== 'PASS') viol++;
  if (xg.ductility !== 'tension controlled ✓') viol++;
  if (xg.converged === false) viol++;
  if (viol > 0) return {_infeasible: true, violations: viol, derived: result.derived};
  const cc = computeCostCarbon(result, ui);
  return {_infeasible: false, result, cost: cc.cost, carbon: cc.carbon, q: cc.q,
          derived: result.derived};
}
function evalCached(chrom, ui) {
  const key = cacheKey(chrom);
  if (cacheMap.has(key)) return cacheMap.get(key);
  const out = evalChrom(chrom, ui);
  cacheMap.set(key, out);
  return out;
}
function fitness(chrom, w_cost, w_carbon, ref, ui) {
  const out = evalCached(chrom, ui);
  if (!out) return 1e9 + 50;
  if (out._infeasible) return 1e9 + (out.violations||1);
  const [cmin, cmax, kmin, kmax] = ref;
  const cn = (out.cost - cmin)/Math.max(1e-9, cmax - cmin);
  const kn = (out.carbon - kmin)/Math.max(1e-9, kmax - kmin);
  return w_cost*cn + w_carbon*kn;
}
function findRefBounds(ui) {
  // Expanded grid search (was 5 candidates, now ~60)
  const candidates = [];
  for (const s_mg of [2.0, 2.25, 2.5, 2.75, 3.0])
    for (const s_xg of [4.0, 5.0, 6.0])
      for (const k_mg of [12.0, 13.0, 14.0])
        for (const k_xg of [0.67, 0.71, 0.75])
          for (const fc_i of [0, 1, 2, 3])
            candidates.push([s_mg, s_xg, k_mg, k_xg, fc_i, 1]);  // fy=400 default
  for (const c of candidates) {
    const out = evalCached(c, ui);
    if (out && !out._infeasible) {
      return [out.cost*0.8, out.cost*1.2, out.carbon*0.8, out.carbon*1.2];
    }
  }
  throw new Error('No feasible midpoint chromosome — try different inputs.');
}
function tournament(rng, pop, fits, k) {
  const idxs = rng.sample(pop.length, k);
  let bi = idxs[0];
  for (const i of idxs) if (fits[i] < fits[bi]) bi = i;
  return [...pop[bi]];
}
function crossover(rng, p1, p2, cx) {
  const out = [];
  for (let i = 0; i < p1.length; i++) {
    if (rng.rand() < cx) {
      if (i < 4) { const a = rng.rand(); out.push(a*p1[i] + (1-a)*p2[i]); }
      else out.push(rng.rand() < 0.5 ? p1[i] : p2[i]);
    } else out.push(p1[i]);
  }
  return out;
}
function mutate(rng, chrom, mut) {
  const ranges = [S_MG_RANGE, S_XG_RANGE, K_MG_RANGE, K_XG_RANGE];
  const out = [...chrom];
  for (let i = 0; i < 4; i++) {
    if (rng.rand() < mut) {
      const [lo, hi] = ranges[i];
      const sigma = 0.05*(hi - lo);
      out[i] = Math.max(lo, Math.min(hi, out[i] + rng.gauss(0, sigma)));
    }
  }
  if (rng.rand() < mut) out[4] = rng.randint(0, FC_OPTIONS.length-1);
  if (rng.rand() < mut) out[5] = rng.randint(0, FY_OPTIONS.length-1);
  return out;
}
async function runGA(rng, w_cost, w_carbon, cfg, ref, ui, onProg, label) {
  let pop = []; let attempts = 0; const cap = cfg.pop*50;
  while (pop.length < cfg.pop && attempts < cap) {
    const c = randomChrom(rng);
    const out = evalCached(c, ui);
    if (out && !out._infeasible) pop.push(c);
    attempts++;
  }
  while (pop.length < cfg.pop) pop.push(randomChrom(rng));
  let fits = pop.map(c => fitness(c, w_cost, w_carbon, ref, ui));
  let bi = 0; for (let i=1; i<fits.length; i++) if (fits[i] < fits[bi]) bi = i;
  let best = [...pop[bi]], bestFit = fits[bi];
  const history = [bestFit]; let stag = 0;
  for (let g = 0; g < cfg.gens; g++) {
    const order = Array.from({length: cfg.pop}, (_, i) => i).sort((a,b) => fits[a] - fits[b]);
    const newPop = [];
    for (let i = 0; i < cfg.elite; i++) newPop.push([...pop[order[i]]]);
    while (newPop.length < cfg.pop) {
      const p1 = tournament(rng, pop, fits, cfg.tournament);
      const p2 = tournament(rng, pop, fits, cfg.tournament);
      let child = crossover(rng, p1, p2, cfg.cx);
      child = mutate(rng, child, cfg.mut);
      newPop.push(child);
    }
    pop = newPop;
    fits = pop.map(c => fitness(c, w_cost, w_carbon, ref, ui));
    let cur = 0; for (let i=1; i<fits.length; i++) if (fits[i] < fits[cur]) cur = i;
    if (fits[cur] < bestFit - 1e-9) { bestFit = fits[cur]; best = [...pop[cur]]; stag = 0; }
    else stag++;
    history.push(bestFit);
    onProg(`GA — ${label}`,
           `gen ${g+1}/${cfg.gens} · best fit ${bestFit.toFixed(4)} · stagnation ${stag}`);
    await new Promise(r => setTimeout(r, 0));
    if (stag >= cfg.stagnation_limit) break;
  }
  return {best, bestFit, history};
}
function paretoFront(pts) {
  const sorted = [...pts].sort((a,b) => a.cost - b.cost || a.carbon - b.carbon);
  const front = []; let bestC = Infinity;
  for (const p of sorted) {
    if (p.carbon < bestC - 1e-6) { front.push(p); bestC = p.carbon; }
  }
  return front;
}
function downsample(arr, n) {
  n = n||200;
  if (arr.length <= n) return arr;
  const step = arr.length/n;
  const out = []; for (let i = 0; i < n; i++) out.push(arr[Math.floor(i*step)]);
  return out;
}
function packageScenario(chrom, ui) {
  const out = evalCached(chrom, ui);
  const der = out.derived;
  return {s_mg: chrom[0], s_xg: chrom[1], k_mg: chrom[2], k_xg: chrom[3],
          fc: FC_OPTIONS[Math.round(chrom[4])],
          fy: FY_OPTIONS[Math.round(chrom[5])],
          n_g: der.n_g, n_xg: der.num_secondary_beams,
          h_mg_mm: der.h_mg_mm, h_xg_mm: der.h_xg_mm,
          b_mg: der.b_mg, b_xg: der.b_xg,
          ts_slab: der.ts_slab, ts_cant: der.ts_cant,
          cost: out.cost, carbon: out.carbon, q: out.q, IN: der.IN};
}
function collectPareto(ui) {
  const pts = [];
  for (const [key, out] of cacheMap.entries()) {
    if (!out || out._infeasible) continue;
    const parts = key.split('|');
    pts.push({s_mg:+parts[0], s_xg:+parts[1], k_mg:+parts[2], k_xg:+parts[3],
              fc: FC_OPTIONS[+parts[4]], fy: FY_OPTIONS[+parts[5]],
              n_g: out.derived.n_g, n_xg: out.derived.num_secondary_beams,
              cost: out.cost, carbon: out.carbon});
  }
  return downsample(paretoFront(pts), 200);
}
async function runOptimization(raw, onProgress) {
  cacheMap.clear();
  const onProg = onProgress || (()=>{});
  const num = (k, def) => {
    const v = raw[k];
    if (v == null || v === '') return def;
    const n = Number(v); return isFinite(n) ? n : def;
  };
  const ui = {
    bridge_length: num('bridge_length', 30),
    num_lanes: Math.round(num('num_lanes', 4)),
    weight_pct: Math.round(num('weight_pct', 50)),
    cantilever_length: num('cantilever_length', 1.5),
    // num_spans auto-calculated from bridge length (≈10 m per span)
    num_spans: Math.max(1, Math.min(20, Math.round(num('bridge_length', 30)/10))),
    cover_mm: Math.round(num('cover_mm', 40)),
    wheel_kN: num('wheel_kN', 145),
    sid_kN: num('sid_kN', 1.5),
    asphalt_mm: Math.round(num('asphalt_mm', 80)),
    lane_width: num('lane_width', 3.65),
    shoulder: num('shoulder', 0.5),
  };
  if (ui.bridge_length < 5 || ui.bridge_length > 500) throw new Error('Bridge length must be 5–500 m');
  if (ui.num_lanes < 1 || ui.num_lanes > 12) throw new Error('Lanes must be 1–12');
  if (ui.weight_pct < 0 || ui.weight_pct > 100) throw new Error('Weight % 0–100');
  ui.deck_width = ui.num_lanes*ui.lane_width + 2*ui.shoulder + 2*ui.cantilever_length;
  ui.span_length = ui.bridge_length/ui.num_spans;
  ui.interior_m = Math.max(0, ui.deck_width - 2*ui.cantilever_length);
  const ga_def = {pop:50, max_gens:100, stagnation_limit:15, mut:0.20, cx:0.80,
                  tournament:3, elite:2};
  const ga = {};
  for (const k of Object.keys(ga_def)) {
    const v = raw[k];
    if (v == null || v === '') ga[k] = ga_def[k];
    else {
      const n = Number(v);
      if (!isFinite(n)) throw new Error(`'${k}' must be numeric`);
      ga[k] = (k === 'mut' || k === 'cx') ? n : Math.round(n);
    }
  }
  if (ga.elite >= ga.pop) throw new Error('elite < pop required');
  if (ga.tournament > ga.pop) throw new Error('tournament ≤ pop required');
  ui.ga = ga;
  onProg('Finding feasible reference midpoint…',
         'Searching ~60 candidates for cost / carbon normalisation bounds.');
  await new Promise(r => setTimeout(r, 0));
  const ref = findRefBounds(ui);
  const cfg = {pop: ga.pop, gens: ga.max_gens, mut: ga.mut, cx: ga.cx,
               tournament: ga.tournament, elite: ga.elite,
               stagnation_limit: ga.stagnation_limit};
  const userWc = ui.weight_pct/100, userWk = 1 - userWc;
  const runs = [['user', userWc, userWk, 'user-priority'],
                ['cost', 1.0, 0.0, 'pure-cost'],
                ['carbon', 0.0, 1.0, 'pure-carbon']];
  const scenarios = {}, histories = {};
  for (const [label, wc, wk, pretty] of runs) {
    onProg(`GA run — ${pretty}`,
           `w_cost=${wc.toFixed(2)}, w_carbon=${wk.toFixed(2)} · pop=${cfg.pop}, max gens=${cfg.gens}`);
    await new Promise(r => setTimeout(r, 0));
    const seed = (42 + Math.abs([...label].reduce((s,c)=>s + c.charCodeAt(0)*31, 0)))%1000;
    const rng = makeRng(seed);
    const r = await runGA(rng, wc, wk, cfg, ref, ui, onProg, pretty);
    scenarios[label] = packageScenario(r.best, ui);
    histories[label] = r.history;
  }
  onProg('Building Pareto front…', 'Filtering non-dominated designs.');
  await new Promise(r => setTimeout(r, 0));
  const pareto = collectPareto(ui);
  return {inputs: ui, scenarios, histories, pareto,
          ga_config: {pop: ga.pop, max_gens: ga.max_gens,
                      stagnation_limit: ga.stagnation_limit,
                      mut: ga.mut, cx: ga.cx, tournament: ga.tournament,
                      elite: ga.elite,
                      gens_used: {user: histories.user.length,
                                  cost: histories.cost.length,
                                  carbon: histories.carbon.length}},
          meta: {timestamp: new Date().toISOString().slice(0, 19),
                 cache_size: cacheMap.size}};
}

// ╔══════════════════════════════════════════════════════════════════════╗
// ║  UI WIRING                                                           ║
// ╚══════════════════════════════════════════════════════════════════════╝
const $ = (id) => document.getElementById(id);
const form = $('input-form');
const inputCard = $('input-card');
const progressCard = $('progress-card');
const progressText = $('progress-text');
const progressSub = $('progress-sub');
const resultsBox = $('results');
const errorBox = $('error-box');
const weightRange = $('weight-range');
const weightReadout = $('weight-readout');

weightRange.addEventListener('input', () => {
  const w = +weightRange.value;
  weightReadout.textContent = `${w}% cost / ${100-w}% carbon`;
});
form.addEventListener('reset', () => setTimeout(() => {
  const w = +weightRange.value;
  weightReadout.textContent = `${w}% cost / ${100-w}% carbon`;
}, 0));

let LAST = null;
function setProgress(text, sub) {
  if (text != null) progressText.textContent = text;
  if (sub != null)  progressSub.textContent  = sub;
}

form.addEventListener('submit', async (e) => {
  e.preventDefault();
  errorBox.classList.add('hidden');
  const fd = new FormData(form);
  const obj = {};
  for (const [k, v] of fd.entries()) obj[k] = v;
  inputCard.classList.add('hidden');
  resultsBox.classList.add('hidden');
  progressCard.classList.remove('hidden');
  setProgress('Initialising in-browser optimizer…', '');
  try {
    await new Promise(r => setTimeout(r, 50));
    const data = await runOptimization(obj, setProgress);
    progressCard.classList.add('hidden');
    LAST = data;
    // ── BUG FIX: unhide BEFORE rendering, give layout one frame to settle.
    // Otherwise canvas.getBoundingClientRect() returns 0 width and the
    // charts come out blank (this was D1 in the diagnostic).
    resultsBox.classList.remove('hidden');
    await new Promise(r => requestAnimationFrame(r));
    renderResults(data);
    resultsBox.scrollIntoView({behavior: 'smooth', block: 'start'});
  } catch (err) {
    progressCard.classList.add('hidden');
    inputCard.classList.remove('hidden');
    errorBox.textContent = (err && err.message ? err.message : String(err)) +
      (err && err.stack ? '\n\n' + err.stack : '');
    errorBox.classList.remove('hidden');
  }
});

$('run-again-btn').addEventListener('click', () => {
  resultsBox.classList.add('hidden');
  inputCard.classList.remove('hidden');
  inputCard.scrollIntoView({behavior: 'smooth', block: 'start'});
});

let resizeTimer = null;
window.addEventListener('resize', () => {
  if (!LAST) return;
  if (resizeTimer) clearTimeout(resizeTimer);
  resizeTimer = setTimeout(() => renderResults(LAST), 100);
});

// ╔══════════════════════════════════════════════════════════════════════╗
// ║  RENDERING                                                           ║
// ╚══════════════════════════════════════════════════════════════════════╝
const TC = {bg:'#402215', surface:'#523B28', text:'#DBCAAE', muted:'#A89272',
            gold:'#FFC857', cyan:'#5BD2E0', green:'#7DDB87',
            grid:'rgba(168,146,114,0.22)',
            concrete:'#A89178', steel:'#7C5A3A', barrier:'#8B6F47'};
const fmtMoney = v => '$' + Math.round(v).toLocaleString();
const fmtKg    = v => Math.round(v).toLocaleString();

function fitCanvas(c) {
  const dpr = window.devicePixelRatio || 1;
  const rect = c.getBoundingClientRect();
  c.width = Math.max(1, Math.round(rect.width*dpr));
  c.height = Math.max(1, Math.round(rect.height*dpr));
  const ctx = c.getContext('2d');
  ctx.setTransform(1,0,0,1,0,0);
  ctx.scale(dpr, dpr);
  return {ctx, w: rect.width, h: rect.height};
}
function niceTicks(min, max, target) {
  target = target || 6;
  if (!isFinite(min) || !isFinite(max)) { min = 0; max = 1; }
  if (max === min) max = min + 1;
  const range = max - min;
  const rough = range/Math.max(1, target - 1);
  const exp = Math.pow(10, Math.floor(Math.log10(rough)));
  const f = rough/exp;
  const nice = f < 1.5 ? 1 : f < 3 ? 2 : f < 7 ? 5 : 10;
  const step = nice*exp;
  const lo = Math.floor(min/step)*step;
  const hi = Math.ceil(max/step)*step;
  const ticks = [];
  for (let v = lo; v <= hi + step/2; v += step) ticks.push(parseFloat(v.toPrecision(12)));
  return {ticks, lo, hi};
}
function drawAxes(ctx, w, h, pad, xR, yR, xLabel, yLabel, fmtX, fmtY, opts) {
  opts = opts || {};
  const x0 = pad.l, y0 = h - pad.b, x1 = w - pad.r, y1 = pad.t;
  ctx.strokeStyle = TC.grid; ctx.lineWidth = 1;
  ctx.font = '11px Calibri, system-ui, sans-serif';
  ctx.fillStyle = TC.muted;
  ctx.textAlign = 'center'; ctx.textBaseline = 'top';
  for (const t of xR.ticks) {
    if (opts.skipXTicks) continue;
    const px = x0 + (t - xR.lo)/(xR.hi - xR.lo)*(x1 - x0);
    ctx.beginPath(); ctx.moveTo(px, y0); ctx.lineTo(px, y1); ctx.stroke();
    ctx.fillText(fmtX ? fmtX(t) : t, px, y0 + 6);
  }
  ctx.textAlign = 'right'; ctx.textBaseline = 'middle';
  for (const t of yR.ticks) {
    const py = y0 - (t - yR.lo)/(yR.hi - yR.lo)*(y0 - y1);
    ctx.beginPath(); ctx.moveTo(x0, py); ctx.lineTo(x1, py); ctx.stroke();
    ctx.fillText(fmtY ? fmtY(t) : t, x0 - 8, py);
  }
  ctx.strokeStyle = TC.muted; ctx.beginPath();
  ctx.moveTo(x0, y0); ctx.lineTo(x1, y0);
  ctx.moveTo(x0, y0); ctx.lineTo(x0, y1); ctx.stroke();
  ctx.fillStyle = TC.text;
  ctx.font = 'bold 12px Georgia, serif';
  ctx.textAlign = 'center'; ctx.textBaseline = 'bottom';
  ctx.fillText(xLabel, (x0 + x1)/2, h - 6);
  ctx.save(); ctx.translate(16, (y0 + y1)/2); ctx.rotate(-Math.PI/2);
  ctx.textBaseline = 'top'; ctx.fillText(yLabel, 0, 0); ctx.restore();
  return {x0, y0, x1, y1,
          px: v => x0 + (v - xR.lo)/(xR.hi - xR.lo)*(x1 - x0),
          py: v => y0 - (v - yR.lo)/(yR.hi - yR.lo)*(y0 - y1)};
}
function drawSymbol(ctx, x, y, kind, size, color) {
  ctx.fillStyle = color; ctx.strokeStyle = '#402215'; ctx.lineWidth = 1.5;
  if (kind === 'circle') {
    ctx.beginPath(); ctx.arc(x, y, size/2, 0, 2*Math.PI); ctx.fill();
  } else if (kind === 'star') {
    const outer = size*0.7, inner = outer*0.45;
    ctx.beginPath();
    for (let i = 0; i < 10; i++) {
      const r = i%2 ? inner : outer;
      const a = Math.PI/2 + i*Math.PI/5;
      const xx = x + r*Math.cos(a), yy = y - r*Math.sin(a);
      if (i === 0) ctx.moveTo(xx, yy); else ctx.lineTo(xx, yy);
    }
    ctx.closePath(); ctx.fill(); ctx.stroke();
  } else if (kind === 'diamond') {
    const r = size*0.6;
    ctx.beginPath();
    ctx.moveTo(x, y-r); ctx.lineTo(x+r, y);
    ctx.lineTo(x, y+r); ctx.lineTo(x-r, y);
    ctx.closePath(); ctx.fill(); ctx.stroke();
  }
}
function drawLegend(ctx, x, y, items) {
  ctx.font = '12px Calibri, system-ui, sans-serif';
  ctx.textBaseline = 'middle';
  const padX = 10, padY = 8, lh = 18;
  let maxW = 0;
  for (const it of items) maxW = Math.max(maxW, ctx.measureText(it.label).width);
  const w = maxW + 36 + padX*2, h = items.length*lh + padY*2;
  ctx.fillStyle = 'rgba(45,23,13,0.85)'; ctx.strokeStyle = TC.muted;
  ctx.lineWidth = 1; ctx.beginPath(); ctx.rect(x, y, w, h);
  ctx.fill(); ctx.stroke();
  ctx.textAlign = 'left';
  for (let i = 0; i < items.length; i++) {
    const it = items[i];
    const yy = y + padY + lh/2 + i*lh;
    drawSymbol(ctx, x + padX + 8, yy, it.symbol||'circle', it.size||10, it.color);
    ctx.fillStyle = TC.text;
    ctx.fillText(it.label, x + padX + 22, yy);
  }
}

// ── Cross-section diagram ──
function drawCrossSection(canvas, scen, ui, accentColor) {
  const {ctx, w, h} = fitCanvas(canvas);
  ctx.fillStyle = TC.bg; ctx.fillRect(0, 0, w, h);

  const Bw = ui.deck_width;          // m
  const Lc = ui.cantilever_length;
  const ts = scen.ts_slab/1000;      // m
  const tc = scen.ts_cant/1000;
  const h_mg_m = scen.h_mg_mm/1000;
  const b_mg_m = scen.b_mg/1000;
  const n_g = scen.n_g;

  // Compute scale: fit Bw m of width into (w - 2*pad) px and (h_mg + ts) m vert into (h - 2*pad) px
  const padX = 24, padY = 36;
  const usableW = w - 2*padX;
  const usableH = h - 2*padY;
  const total_h = h_mg_m + ts;          // total height (slab + girder)
  const sx = usableW / Bw;
  const sy = usableH / total_h * 0.80;  // leave room for labels
  const s = Math.min(sx, sy);           // unified scale (so it looks proportional)

  const drawW = Bw * s;
  const drawH = total_h * s;
  const ox = (w - drawW)/2;
  const oy = padY + (usableH - drawH)/2 + 12;

  // Slab (thicker over cantilevers if upsized)
  ctx.fillStyle = TC.concrete;
  // Interior slab (uniform thickness)
  ctx.fillRect(ox + Lc*s, oy, (Bw - 2*Lc)*s, ts*s);
  // Cantilevers (may be thicker)
  ctx.fillRect(ox, oy, Lc*s, tc*s);
  ctx.fillRect(ox + (Bw - Lc)*s, oy, Lc*s, tc*s);
  ctx.strokeStyle = TC.muted; ctx.lineWidth = 0.8;
  ctx.strokeRect(ox, oy, Lc*s, tc*s);
  ctx.strokeRect(ox + Lc*s, oy, (Bw - 2*Lc)*s, ts*s);
  ctx.strokeRect(ox + (Bw - Lc)*s, oy, Lc*s, tc*s);

  // Main girders (positioned at evenly spaced positions across interior)
  ctx.fillStyle = TC.steel;
  const Int_m = Bw - 2*Lc;
  const girder_top_y = oy + ts*s;
  const girder_h = (h_mg_m - ts) * s;
  for (let i = 0; i < n_g; i++) {
    const xc_m = Lc + (Int_m * i)/(n_g - 1);
    const px_c = ox + xc_m*s;
    ctx.fillRect(px_c - b_mg_m*s/2, girder_top_y, b_mg_m*s, girder_h);
    ctx.strokeRect(px_c - b_mg_m*s/2, girder_top_y, b_mg_m*s, girder_h);
  }

  // Barriers (small markers at ends)
  ctx.fillStyle = TC.barrier;
  const barH = 1.2*s;
  ctx.fillRect(ox - 4, oy - barH, 4, barH);
  ctx.fillRect(ox + drawW, oy - barH, 4, barH);

  // Dimension lines
  ctx.fillStyle = TC.muted;
  ctx.strokeStyle = TC.muted;
  ctx.font = '10px Calibri';
  ctx.textAlign = 'center';
  ctx.textBaseline = 'top';
  // Bw label below
  ctx.beginPath();
  ctx.moveTo(ox, oy + drawH + 6); ctx.lineTo(ox + drawW, oy + drawH + 6);
  ctx.stroke();
  ctx.fillText(`Bw = ${Bw.toFixed(2)} m`, ox + drawW/2, oy + drawH + 9);
  // h_mg label on right
  ctx.textAlign = 'left';
  ctx.fillText(`h_mg = ${Math.round(scen.h_mg_mm)} mm`, ox + drawW + 8, girder_top_y + girder_h/2 - 5);
  ctx.fillText(`ts = ${Math.round(scen.ts_slab)} mm`,    ox + drawW + 8, oy + 4);

  // n_g label
  ctx.textAlign = 'center';
  ctx.fillStyle = accentColor;
  ctx.font = 'bold 11px Calibri';
  ctx.fillText(`${n_g} main girders`, ox + drawW/2, girder_top_y + girder_h + 4);

  // Title (top)
  ctx.font = 'bold 13px Georgia';
  ctx.textAlign = 'left';
  ctx.fillStyle = accentColor;
  ctx.fillText(canvas.dataset.title || '', 8, 20);
}

// ── Main render entry ──
function renderResults(data) {
  renderInputsEcho(data.inputs);
  renderScenarioCards(data);                     // NEW primary visual
  renderComparison(data.scenarios);
  drawPareto('cv-pareto', 'tt-pareto', data.pareto, data.scenarios);
  drawConvergence('cv-convergence', data.histories);
  renderHpCard(data.ga_config);
}

function renderInputsEcho(ui) {
  const rows = [
    ['Bridge length', ui.bridge_length.toFixed(1) + ' m'],
    ['Number of lanes', ui.num_lanes],
    ['Cantilever length', ui.cantilever_length.toFixed(2) + ' m'],
    ['Number of spans (auto)', ui.num_spans],
    ['Span length', ui.span_length.toFixed(2) + ' m'],
    ['Deck width', ui.deck_width.toFixed(2) + ' m'],
    ['Interior width', ui.interior_m.toFixed(2) + ' m'],
    ['Cover', ui.cover_mm + ' mm'],
    ['AASHTO wheel', Math.round(ui.wheel_kN) + ' kN'],
    ['Cost↔Carbon weight', `${ui.weight_pct}% cost / ${100-ui.weight_pct}% carbon`],
  ];
  $('inputs-echo').innerHTML = rows.map(([k,v]) =>
    `<div class="kv"><span class="k">${k}</span><span>${v}</span></div>`
  ).join('');
}

// NEW: Scenario cards with cross-section diagrams
function renderScenarioCards(data) {
  const scen = data.scenarios, ui = data.inputs;
  const meta = [
    ['user',   'User priority', TC.gold,  'user-card',   'tag-user',
     `${ui.weight_pct}% cost · ${100-ui.weight_pct}% carbon`],
    ['cost',   'Pure cost',     TC.cyan,  'cost-card',   'tag-cost',
     'minimum construction cost'],
    ['carbon', 'Pure carbon',   TC.green, 'carbon-card', 'tag-carbon',
     'minimum embodied carbon'],
  ];
  const u = scen.user;
  const cards = meta.map(([key, title, color, cardClass, tagClass, sub]) => {
    const s = scen[key];
    const dC = key === 'user' ? null : (u.cost - s.cost)/Math.max(1, u.cost)*100;
    const dK = key === 'user' ? null : (u.carbon - s.carbon)/Math.max(1, u.carbon)*100;
    const fmtPct = v => (v >= 0 ? '+' : '') + v.toFixed(2) + '%';
    const deltas = key === 'user' ? '' :
      `<div class="small">vs user: cost ${fmtPct(dC)} · carbon ${fmtPct(dK)}</div>`;
    return `
      <div class="scenario-card ${cardClass}">
        <div class="scenario-head">
          <span class="scenario-title">${title}</span>
          <span class="tag ${tagClass}">scenario</span>
        </div>
        <div class="small">${sub}</div>
        <canvas class="section-canvas" data-title="" data-key="${key}"></canvas>
        <div class="bignum">
          <span><span class="lbl">cost</span><br>${fmtMoney(s.cost)}</span>
          <span><span class="lbl">carbon</span><br>${fmtKg(s.carbon)} kg</span>
        </div>
        <div class="spec-list">
          <span class="sk">f'c</span><span class="sv">${s.fc} MPa</span>
          <span class="sk">fy</span><span class="sv">${s.fy} MPa</span>
          <span class="sk">main girders</span><span class="sv">${s.n_g}</span>
          <span class="sk">cross-girders</span><span class="sv">${s.n_xg}</span>
          <span class="sk">h<sub>mg</sub></span><span class="sv">${Math.round(s.h_mg_mm)} mm</span>
          <span class="sk">h<sub>xg</sub></span><span class="sv">${Math.round(s.h_xg_mm)} mm</span>
          <span class="sk">main spacing</span><span class="sv">${s.s_mg.toFixed(2)} m</span>
          <span class="sk">cross spacing</span><span class="sv">${s.s_xg.toFixed(2)} m</span>
        </div>
        ${deltas}
      </div>`;
  }).join('');
  $('scenario-cards').innerHTML = cards;
  // After insertion, draw each cross-section
  const colors = {user: TC.gold, cost: TC.cyan, carbon: TC.green};
  for (const card of $('scenario-cards').querySelectorAll('canvas.section-canvas')) {
    const key = card.dataset.key;
    drawCrossSection(card, scen[key], ui, colors[key]);
  }
}

function renderComparison(scen) {
  const tag = {user:'<span class="tag tag-user">User</span>',
               cost:'<span class="tag tag-cost">Pure cost</span>',
               carbon:'<span class="tag tag-carbon">Pure carbon</span>'};
  const keys = ['user','cost','carbon'];
  const rows = [];
  rows.push(`<table><thead><tr><th>Metric</th>` +
            keys.map(k => `<th>${tag[k]}</th>`).join('') + `</tr></thead><tbody>`);
  const m = (label, fn) =>
    `<tr><td>${label}</td>` + keys.map(k => `<td>${fn(scen[k])}</td>`).join('') + `</tr>`;
  rows.push(m('Total cost (USD)',     s => fmtMoney(s.cost)));
  rows.push(m('Total carbon (kg CO₂)', s => fmtKg(s.carbon)));
  rows.push(m('Concrete vol. (m³)',   s => s.q.V_concrete_m3.toFixed(1)));
  rows.push(m('Steel mass (kg)',       s => fmtKg(s.q.M_steel_kg)));
  rows.push(m('Formwork (m²)',         s => s.q.A_formwork_m2.toFixed(1)));
  rows.push(m('Cost: concrete',  s => fmtMoney(s.q.cost_concrete)));
  rows.push(m('Cost: steel',     s => fmtMoney(s.q.cost_steel)));
  rows.push(m('Cost: formwork',  s => fmtMoney(s.q.cost_formwork)));
  rows.push(m('Carbon: concrete', s => fmtKg(s.q.carbon_concrete)));
  rows.push(m('Carbon: steel',    s => fmtKg(s.q.carbon_steel)));
  rows.push(m('Carbon: formwork', s => fmtKg(s.q.carbon_formwork)));
  rows.push(`</tbody></table>`);
  $('comparison-table').innerHTML = rows.join('');
}

function drawPareto(canvasId, ttId, pareto, scen) {
  const canvas = $(canvasId), tt = $(ttId);
  const {ctx, w, h} = fitCanvas(canvas);
  if (w < 50 || h < 50) return;     // safety
  const all = pareto.concat([
    {cost: scen.user.cost, carbon: scen.user.carbon},
    {cost: scen.cost.cost, carbon: scen.cost.carbon},
    {cost: scen.carbon.cost, carbon: scen.carbon.carbon},
  ]);
  const xs = all.map(p => p.cost), ys = all.map(p => p.carbon);
  const xR = niceTicks(Math.min(...xs), Math.max(...xs), 6);
  const yR = niceTicks(Math.min(...ys), Math.max(...ys), 6);
  const pad = {l: 90, r: 30, t: 30, b: 60};
  const fr = drawAxes(ctx, w, h, pad, xR, yR,
    'Cost (USD)', 'Carbon (kg CO₂-eq)', fmtMoney, fmtKg);
  const plotted = pareto.map(p => ({
    px: fr.px(p.cost), py: fr.py(p.carbon),
    cost: p.cost, carbon: p.carbon, p,
  }));
  ctx.fillStyle = TC.text; ctx.strokeStyle = TC.muted; ctx.lineWidth = 0.5;
  for (const pt of plotted) {
    ctx.beginPath(); ctx.arc(pt.px, pt.py, 3.8, 0, 2*Math.PI);
    ctx.fill(); ctx.stroke();
  }
  const highlights = [
    {p: scen.user,   color: TC.gold,  symbol: 'star',    size: 22, label: 'User'},
    {p: scen.cost,   color: TC.cyan,  symbol: 'diamond', size: 18, label: 'Pure cost'},
    {p: scen.carbon, color: TC.green, symbol: 'diamond', size: 18, label: 'Pure carbon'},
  ];
  const hpts = highlights.map(h => ({
    px: fr.px(h.p.cost), py: fr.py(h.p.carbon),
    cost: h.p.cost, carbon: h.p.carbon, p: h.p,
    color: h.color, symbol: h.symbol, size: h.size, label: h.label, isHl: true,
  }));
  for (const hp of hpts) drawSymbol(ctx, hp.px, hp.py, hp.symbol, hp.size, hp.color);
  drawLegend(ctx, fr.x1 - 170, fr.y1 + 6, [
    {color: TC.text,  symbol: 'circle',  label: 'Explored designs'},
    {color: TC.gold,  symbol: 'star',    label: 'User'},
    {color: TC.cyan,  symbol: 'diamond', label: 'Pure cost'},
    {color: TC.green, symbol: 'diamond', label: 'Pure carbon'},
  ]);
  const all_pts = plotted.concat(hpts);
  canvas.onmousemove = (e) => {
    const rect = canvas.getBoundingClientRect();
    const mx = e.clientX - rect.left, my = e.clientY - rect.top;
    let best = null, bestD = 18*18;
    for (const pt of all_pts) {
      const dx = pt.px - mx, dy = pt.py - my;
      const d = dx*dx + dy*dy;
      if (d < bestD) { bestD = d; best = pt; }
    }
    if (best) {
      const p = best.p;
      const hdr = best.isHl ? `<b style="color:${best.color}">${best.label}</b><br>` : '';
      tt.innerHTML = hdr +
        `Cost ${fmtMoney(best.cost)}<br>Carbon ${fmtKg(best.carbon)} kg<br>` +
        (p.s_mg !== undefined ?
          `s_mg=${p.s_mg.toFixed(2)} m, s_xg=${p.s_xg.toFixed(2)} m<br>`+
          `k_mg=${p.k_mg.toFixed(2)}, k_xg=${p.k_xg.toFixed(2)}<br>`+
          `f'c=${p.fc} MPa, fy=${p.fy} MPa<br>n_g=${p.n_g}, n_xg=${p.n_xg}`
          : '');
      tt.style.display = 'block';
      const tw = tt.offsetWidth, th = tt.offsetHeight;
      let tx = mx + 14, ty = my - th - 8;
      if (tx + tw > rect.width) tx = mx - tw - 14;
      if (ty < 0) ty = my + 14;
      tt.style.left = tx + 'px'; tt.style.top = ty + 'px';
    } else { tt.style.display = 'none'; }
  };
  canvas.onmouseleave = () => { tt.style.display = 'none'; };
}

function drawConvergence(canvasId, hist) {
  const canvas = $(canvasId);
  const {ctx, w, h} = fitCanvas(canvas);
  if (w < 50 || h < 50) return;
  const series = [
    {name: 'User',   color: TC.gold,  data: hist.user},
    {name: 'Cost',   color: TC.cyan,  data: hist.cost},
    {name: 'Carbon', color: TC.green, data: hist.carbon},
  ];
  const allY = [].concat(...series.map(s => s.data));
  const maxLen = Math.max(...series.map(s => s.data.length));
  const xR = niceTicks(0, Math.max(1, maxLen - 1), 6);
  const yR = niceTicks(Math.min(...allY), Math.max(...allY), 6);
  const pad = {l: 90, r: 30, t: 30, b: 60};
  const fr = drawAxes(ctx, w, h, pad, xR, yR, 'Generation', 'Best fitness',
    v => Math.round(v).toString(), v => v.toFixed(3));
  for (const sr of series) {
    ctx.strokeStyle = sr.color; ctx.lineWidth = 2;
    ctx.beginPath();
    sr.data.forEach((y, i) => {
      const px = fr.px(i), py = fr.py(y);
      if (i === 0) ctx.moveTo(px, py); else ctx.lineTo(px, py);
    });
    ctx.stroke();
  }
  drawLegend(ctx, fr.x1 - 160, fr.y1 + 6,
    series.map(s => ({color: s.color, symbol: 'circle', label: s.name})));
}

function renderHpCard(ga) {
  const used = ga.gens_used || {};
  const rows = [
    ['Population size', ga.pop],
    ['Max generations', ga.max_gens],
    ['Stagnation limit', ga.stagnation_limit],
    ['Mutation prob.', ga.mut],
    ['Crossover prob.', ga.cx],
    ['Tournament size (k)', ga.tournament],
    ['Elites kept', ga.elite],
    ['Generations actually run',
     `user ${used.user||'—'} · cost ${used.cost||'—'} · carbon ${used.carbon||'—'}`],
  ];
  $('hp-card').innerHTML =
    `<div class="grid">` + rows.map(([k,v]) =>
      `<div class="kv"><span class="k">${k}</span><span>${v}</span></div>`
    ).join('') + `</div>`;
}
</script>
</body>
</html>
