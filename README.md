<!DOCTYPE html>
<html lang="en">
<head>
<meta charset="utf-8">
<meta name="viewport" content="width=device-width, initial-scale=1">
<title>PRISM RANGE</title>
<style>
  :root { --red:#ff4655; --navy:#0f1923; --paper:#ece8e1; --cyan:#3ff5d9; --mut:#9aa7b4; --xc:#3ff5d9; --gap:4px; }
  * { box-sizing:border-box; margin:0; padding:0; }
  html, body { width:100%; height:100%; overflow:hidden; background:#05070b; color:var(--paper);
    font-family:"Bahnschrift","DIN Alternate","Arial Narrow",system-ui,sans-serif; user-select:none; -webkit-user-select:none; }
  #gl { position:fixed; inset:0; width:100%; height:100%; display:block; }
  .hidden { display:none !important; }

  /* ---------- HUD ---------- */
  #hud { position:fixed; inset:0; pointer-events:none; }
  #xh { position:fixed; left:50%; top:50%; width:0; height:0; }
  #xh i { position:absolute; background:var(--xc); box-shadow:0 0 0 1px rgba(0,0,0,.65); }
  #xh .t { width:2px; height:8px; left:-1px; top:calc(-8px - var(--gap)); }
  #xh .b { width:2px; height:8px; left:-1px; top:var(--gap); }
  #xh .l { width:8px; height:2px; top:-1px; left:calc(-8px - var(--gap)); }
  #xh .r { width:8px; height:2px; top:-1px; left:var(--gap); }
  #xh .dot { width:2px; height:2px; left:-1px; top:-1px; }
  #hit { position:fixed; left:50%; top:50%; opacity:0; transition:opacity .25s; }
  #hit i { position:absolute; width:9px; height:3px; left:-4px; top:-1px; background:#fff; box-shadow:0 0 0 1px rgba(0,0,0,.6); }
  #hit i:nth-child(1){ transform:rotate(45deg) translateX(13px); }
  #hit i:nth-child(2){ transform:rotate(135deg) translateX(13px); }
  #hit i:nth-child(3){ transform:rotate(225deg) translateX(13px); }
  #hit i:nth-child(4){ transform:rotate(315deg) translateX(13px); }
  #hit.head i { background:var(--red); }
  #hit.on { opacity:1; transition:none; }
  #scope { position:fixed; inset:0; display:none; background:radial-gradient(circle at 50% 50%, transparent 0, transparent 30vh, #000 30.4vh); }
  #scope::before, #scope::after { content:""; position:absolute; background:#000; }
  #scope::before { left:0; right:0; top:50%; height:1px; }
  #scope::after { top:0; bottom:0; left:50%; width:1px; }
  #dmg { position:fixed; inset:0; background:radial-gradient(ellipse at center, transparent 45%, rgba(255,40,60,.75) 100%); opacity:0; transition:opacity .5s; }
  #dmg.on { opacity:1; transition:none; }
  #dmgdir { position:fixed; left:50%; top:50%; width:0; height:0; opacity:0; transition:opacity .8s; }
  #dmgdir::after { content:""; position:absolute; left:-18px; top:-190px; border:18px solid transparent; border-bottom:34px solid rgba(255,70,85,.9); }
  #dmgdir.on { opacity:1; transition:none; }
  #stats { position:fixed; left:12px; top:10px; font:12px/1.5 ui-monospace,Menlo,Consolas,monospace; color:#cfe; text-shadow:0 1px 2px #000; white-space:pre; }
  #feed { position:fixed; right:16px; top:16px; display:flex; flex-direction:column; align-items:flex-end; gap:4px; }
  #feed div { background:rgba(15,25,35,.8); padding:5px 12px; border-left:3px solid var(--red); font-size:15px; letter-spacing:.06em; text-transform:uppercase; }
  #feed div.head { border-color:#ffd24a; }
  #toast { position:fixed; left:50%; top:16%; transform:translateX(-50%); font-size:22px; letter-spacing:.14em; text-transform:uppercase; text-shadow:0 2px 6px #000; opacity:0; transition:opacity .4s; }
  #toast.on { opacity:1; transition:none; }
  #score { position:fixed; left:50%; top:12px; transform:translateX(-50%); font-size:20px; letter-spacing:.2em; text-shadow:0 2px 4px #000; }
  .panel { position:fixed; background:rgba(15,25,35,.78); padding:10px 16px; text-transform:uppercase; }
  #bl { left:16px; bottom:16px; display:flex; gap:14px; align-items:flex-end; }
  #bl b { font-size:44px; line-height:1; }
  #bl small { display:block; font-size:11px; letter-spacing:.18em; color:var(--mut); }
  #ar b { color:var(--cyan); }
  #br { right:16px; bottom:16px; text-align:right; }
  #wname { font-size:13px; letter-spacing:.24em; color:var(--mut); }
  #ammo { font-size:48px; line-height:1; font-weight:700; }
  #ammo small { font-size:20px; color:var(--mut); display:inline; letter-spacing:0; }
  #abil { left:50%; bottom:16px; transform:translateX(-50%); display:flex; gap:10px; }
  .ab { position:relative; width:78px; height:58px; background:rgba(15,25,35,.85); border:1px solid rgba(255,255,255,.18); text-align:center; overflow:hidden; }
  .ab b { display:block; font-size:22px; margin-top:6px; }
  .ab span { font-size:10px; letter-spacing:.14em; color:var(--mut); }
  .ab u { position:absolute; left:0; right:0; bottom:0; height:0; background:rgba(255,70,85,.45); }
  #slots { right:16px; bottom:108px; display:flex; gap:6px; padding:6px; }
  #slots div { width:34px; height:28px; display:flex; align-items:center; justify-content:center; font-size:13px; border:1px solid rgba(255,255,255,.2); color:var(--mut); }
  #slots div.on { background:var(--red); color:#fff; border-color:var(--red); }
  #blockSel { left:50%; bottom:92px; transform:translateX(-50%); display:flex; gap:10px; align-items:center; font-size:14px; letter-spacing:.14em; }
  #blockSel i { width:22px; height:22px; border:2px solid #fff; display:inline-block; }

  /* ---------- Menu ---------- */
  #menu { position:fixed; inset:0; display:flex; align-items:center; justify-content:center; background:rgba(5,8,13,.72); backdrop-filter:blur(5px); overflow:auto; }
  .card { width:min(940px, calc(100vw - 28px)); background:var(--navy); border:1px solid rgba(255,255,255,.12); padding:28px 32px; margin:16px; }
  .card h1 { font-size:58px; letter-spacing:.12em; line-height:1; }
  .card h1 span { color:var(--red); }
  .sub { color:var(--mut); letter-spacing:.2em; text-transform:uppercase; font-size:13px; margin:6px 0 20px; }
  #play { width:100%; padding:16px; background:var(--red); border:0; color:#fff; font:inherit; font-size:22px; font-weight:700; letter-spacing:.2em; cursor:pointer; }
  #play:hover { filter:brightness(1.12); }
  .cols { display:grid; grid-template-columns:1fr 1fr; gap:28px; margin-top:22px; }
  @media (max-width:760px){ .cols { grid-template-columns:1fr; } }
  .cols h3 { font-size:13px; letter-spacing:.24em; color:var(--cyan); margin-bottom:10px; }
  table { border-collapse:collapse; width:100%; font-size:14px; }
  td { padding:3px 0; color:#d8d4cc; } td:first-child { color:#fff; font-weight:700; width:42%; letter-spacing:.06em; }
  .row { display:flex; align-items:center; justify-content:space-between; gap:12px; margin-bottom:9px; font-size:14px; }
  .row label { color:#d8d4cc; min-width:130px; }
  .row input[type=range] { flex:1; accent-color:var(--red); }
  .row output { width:44px; text-align:right; color:#fff; font-variant-numeric:tabular-nums; }
  select { background:#16222e; color:#fff; border:1px solid rgba(255,255,255,.2); padding:5px 8px; font:inherit; }
  .note { margin-top:14px; font-size:12px; line-height:1.55; color:var(--mut); text-transform:none; letter-spacing:0; }
  #err { position:fixed; left:10px; right:10px; bottom:10px; max-height:46vh; overflow:auto; background:rgba(60,0,10,.95); color:#ffd7dc;
    font:12px/1.4 ui-monospace,Menlo,Consolas,monospace; padding:12px 14px; white-space:pre-wrap; border:1px solid var(--red); z-index:50; user-select:text; -webkit-user-select:text; }
</style>
</head>
<body>
<canvas id="gl"></canvas>

<div id="hud" class="hidden">
  <div id="xh"><i class="t"></i><i class="b"></i><i class="l"></i><i class="r"></i><i class="dot"></i></div>
  <div id="hit"><i></i><i></i><i></i><i></i></div>
  <div id="scope"></div>
  <div id="dmg"></div>
  <div id="dmgdir"></div>
  <div id="stats"></div>
  <div id="score">KILLS 0</div>
  <div id="feed"></div>
  <div id="toast"></div>
  <div id="bl" class="panel"><div id="hpb"><b id="hp">100</b><small>HEALTH</small></div><div id="ar"><b id="arv">50</b><small>ARMOR</small></div></div>
  <div id="br" class="panel"><div id="wname">RIFLE</div><div id="ammo">25 <small>/ 50</small></div></div>
  <div id="slots" class="panel"><div>1</div><div>2</div><div>3</div><div>4</div></div>
  <div id="abil">
    <div class="ab" id="abQ"><b>Q</b><span>UPDRAFT</span><u></u></div>
    <div class="ab" id="abE"><b>E</b><span>DASH</span><u></u></div>
  </div>
  <div id="blockSel" class="panel hidden"><i id="swatch"></i><span id="bname"></span><span style="color:var(--mut)">SCROLL TO CHANGE</span></div>
</div>

<div id="menu">
  <div class="card">
    <h1>PRISM<span>RANGE</span></h1>
    <div class="sub">First-person ray-traced tactical sandbox</div>
    <button id="play">CLICK TO PLAY</button>
    <div class="cols">
      <div>
        <h3>CONTROLS</h3>
        <table>
          <tr><td>WASD</td><td>Move</td></tr>
          <tr><td>MOUSE</td><td>Aim · LMB shoot · RMB zoom</td></tr>
          <tr><td>SPACE / C</td><td>Jump / Crouch (hold)</td></tr>
          <tr><td>SHIFT</td><td>Walk (silent, accurate)</td></tr>
          <tr><td>Q / E</td><td>Updraft / Dash</td></tr>
          <tr><td>R</td><td>Reload</td></tr>
          <tr><td>1 · 2 · 3</td><td>Sidearm · Rifle · Sniper</td></tr>
          <tr><td>4</td><td>Build tool: LMB place, RMB break, wheel = block</td></tr>
          <tr><td>T / Y</td><td>Spawn bot / Clear bots</td></tr>
          <tr><td>U</td><td>Bots shoot back on/off</td></tr>
          <tr><td>F</td><td>Fly mode (Space up, C down)</td></tr>
          <tr><td>ESC</td><td>Pause / settings</td></tr>
        </table>
      </div>
      <div>
        <h3>GRAPHICS</h3>
        <div class="row"><label for="preset">Preset</label>
          <select id="preset"><option value="performance">Performance</option><option value="balanced">Balanced</option><option value="quality">Quality</option><option value="ultra">Ultra</option></select></div>
        <div class="row"><label for="scale">Render scale</label><input id="scale" type="range" min="30" max="100" step="5"><output id="scaleV"></output></div>
        <div class="row"><label for="bounces">Light bounces</label>
          <select id="bounces"><option value="0">0 (sun + AO)</option><option value="1">1</option><option value="2">2</option><option value="3">3</option></select></div>
        <div class="row"><label for="reflect">Reflections</label><input id="reflect" type="checkbox"></div>
        <div class="row"><label for="auto">Auto resolution</label><input id="auto" type="checkbox"></div>
        <div class="row"><label for="tod">Time of day</label><input id="tod" type="range" min="6" max="75" step="1"><output id="todV"></output></div>
        <div class="row"><label for="fov">Field of view</label><input id="fov" type="range" min="80" max="120" step="1"><output id="fovV"></output></div>
        <div class="row"><label for="sens">Sensitivity</label><input id="sens" type="range" min="0.05" max="2" step="0.01"><output id="sensV"></output></div>
        <p class="note">Everything on screen is path-traced per pixel in a shader (sun shadows, ambient occlusion, bounce lighting, reflections).
        NVIDIA DLSS can’t run in a browser, so the built-in temporal upscaler does a similar job: it renders fewer pixels, jitters the camera, reprojects the previous frame and blends it in.</p>
        <p class="note" id="gpuinfo"></p>
      </div>
    </div>
  </div>
</div>
<pre id="err" class="hidden"></pre>

<script>
(() => {
'use strict';
const $ = id => document.getElementById(id);
const clamp = (v, a, b) => Math.max(a, Math.min(b, v));
const lerp = (a, b, t) => a + (b - a) * t;
const rand = (a, b) => a + Math.random() * (b - a);
const approach = (v, t, d) => v < t ? Math.min(v + d, t) : Math.max(v - d, t);

function showError(msg) {
  const e = $('err'); e.classList.remove('hidden'); e.textContent += msg + '\n';
  console.error(msg);
}

/* ===================================================================
   Settings
   =================================================================== */
const PRESETS = {
  performance: { scale: 0.40, bounces: 0, reflect: false },
  balanced:    { scale: 0.55, bounces: 1, reflect: true },
  quality:     { scale: 0.75, bounces: 2, reflect: true },
  ultra:       { scale: 1.00, bounces: 3, reflect: true }
};
const S = { preset: 'balanced', scale: 0.55, bounces: 1, reflect: true, auto: true, fov: 103, sens: 0.45, tod: 38 };
try { Object.assign(S, JSON.parse(localStorage.getItem('prismRangeSettings') || '{}')); } catch (e) {}
const saveSettings = () => { try { localStorage.setItem('prismRangeSettings', JSON.stringify(S)); } catch (e) {} };

/* ===================================================================
   Voxel world  (128 x 40 x 128 voxels of 0.5 m  =  64 x 20 x 64 m)
   =================================================================== */
const NX = 128, NY = 40, NZ = 128, VS = 0.5, GROUND = 0.5;
const vox = new Uint8Array(NX * NY * NZ);
const inb = (i, j, k) => i >= 0 && j >= 0 && k >= 0 && i < NX && j < NY && k < NZ;
const getV = (i, j, k) => inb(i, j, k) ? vox[i + NX * (j + NY * k)] : 0;
const setV = (i, j, k, id) => { if (inb(i, j, k)) vox[i + NX * (j + NY * k)] = id; };

// materials: 1 concrete, 2 plaster, 3 steel, 4 wood, 5 light, 6 red, 7 teal, 8 chrome, 9 polished tile
const MATS = [
  { id: 2, name: 'PLASTER',  css: '#cdc2ad' }, { id: 1, name: 'CONCRETE', css: '#80808a' },
  { id: 3, name: 'STEEL',    css: '#6f7580' }, { id: 4, name: 'WOOD',     css: '#73492a' },
  { id: 5, name: 'LIGHT',    css: '#fff0b8' }, { id: 6, name: 'RED',      css: '#c71a22' },
  { id: 7, name: 'TEAL',     css: '#16a095' }, { id: 8, name: 'CHROME',   css: '#e8edf5' },
  { id: 9, name: 'POLISHED', css: '#c4ccd6' }
];

// box in metres: x/z are absolute, y is height above the ground surface
function box(x0, z0, x1, z1, y0, y1, id) {
  const i0 = Math.round(x0 / VS), i1 = Math.round(x1 / VS), k0 = Math.round(z0 / VS), k1 = Math.round(z1 / VS);
  const j0 = Math.round(y0 / VS) + 1, j1 = Math.round(y1 / VS) + 1;
  for (let k = k0; k < k1; k++) for (let j = j0; j < j1; j++) for (let i = i0; i < i1; i++) setV(i, j, k, id);
}
function floorPatch(x0, z0, x1, z1, id) {
  for (let k = Math.round(z0 / VS); k < Math.round(z1 / VS); k++)
    for (let i = Math.round(x0 / VS); i < Math.round(x1 / VS); i++) setV(i, 0, k, id);
}
// wall with a doorway:  axis 'x' = wall runs along z at fixed x
function wallX(x, z0, z1, d0, d1, h) { box(x, z0, x + 1, d0, 0, h, 2); box(x, d1, x + 1, z1, 0, h, 2); box(x, d0, x + 1, d1, 4.5, h, 2); }

function buildMap() {
  vox.fill(0);
  for (let k = 0; k < NZ; k++) for (let i = 0; i < NX; i++) setV(i, 0, k, 1);

  // outer walls
  box(0, 0, 64, 1, 0, 8, 2); box(0, 63, 64, 64, 0, 8, 2); box(0, 0, 1, 64, 0, 8, 2); box(63, 0, 64, 64, 0, 8, 2);

  // attacker spawn room (x 19..45, z 1..14) with side doorways
  wallX(18, 1, 14, 6, 10, 6); wallX(45, 1, 14, 6, 10, 6);
  floorPatch(26, 4, 38, 10, 3);               // reflective steel plates
  box(24, 1, 40, 1.5, 4, 4.5, 5);             // light strip on back wall

  // long walls forming lanes and halls
  wallX(15, 14, 40, 26, 30, 6); wallX(25, 14, 40, 20, 24, 6);
  wallX(38, 14, 40, 20, 24, 6); wallX(48, 14, 40, 26, 30, 6);

  // front wall with five entrances (A, west hall, mid, east hall, B)
  for (const [a, b] of [[1, 8], [14, 17], [23, 29], [35, 41], [47, 50], [56, 63]]) box(a, 40, b, 41, 0, 6, 2);

  // wall light strips
  box(14.5, 16, 15, 22, 3, 3.5, 5); box(14.5, 32, 15, 38, 3, 3.5, 5);
  box(16, 30, 16.5, 36, 3, 3.5, 5); box(24.5, 26, 25, 36, 3, 3.5, 5);
  box(39, 26, 39.5, 36, 3, 3.5, 5); box(48.5, 32, 49, 38, 3, 3.5, 5);

  // hall + mid cover
  box(18, 28, 20, 30, 0, 1, 4); box(21, 18, 23, 19, 0, 1.5, 4); box(20, 34, 22, 36, 0, 1, 4);
  box(41, 28, 43, 30, 0, 1, 4); box(44, 18, 46, 19, 0, 1.5, 4); box(42, 34, 44, 36, 0, 1, 4);
  box(29, 22, 31, 24, 0, 1.5, 4); box(33, 28, 35, 30, 0, 1, 4); box(30, 34, 32, 36, 0, 1, 4);
  box(31, 26, 32, 27, 0, 3, 3);

  // west + east lane cover
  box(5, 20, 8, 22, 0, 1, 4); box(9, 30, 11, 31, 0, 2, 4); box(54, 20, 57, 22, 0, 1, 4); box(52, 30, 54, 31, 0, 2, 4);

  // ---- A site (teal, covered by a roof) ----
  floorPatch(4, 44, 22, 60, 7);
  box(8, 46, 10, 48, 0, 1, 4); box(14, 54, 16, 56, 0, 1, 4); box(14, 54, 15, 55, 1, 2, 4); box(4, 57, 6, 59, 0, 1.5, 4);
  box(2, 50, 6, 62, 0, 1, 2); box(6, 52, 7, 56, 0, 0.5, 2);                      // platform + step
  box(10, 42, 23, 53, 5, 5.5, 2);                                                  // roof
  for (const [x, z] of [[10, 42], [22, 42], [10, 52], [22, 52]]) box(x, z, x + 1, z + 1, 0, 5, 3);
  box(13, 46, 19, 47, 4.5, 5, 5);                                                   // light under roof
  box(18, 58, 19, 59, 0, 6, 3);

  // ---- B site (red, polished tile + mirror wall) ----
  floorPatch(42, 44, 62, 62, 9); floorPatch(48, 48, 56, 56, 6);
  box(44, 46, 46, 48, 0, 1.5, 4); box(58, 46, 60, 48, 0, 1, 4); box(58, 46, 59, 47, 1, 1.5, 4); box(50, 58, 52, 60, 0, 1, 4);
  box(46, 50, 62, 62, 5, 5.5, 2);
  for (const [x, z] of [[46, 50], [54, 50], [46, 60], [54, 60]]) box(x, z, x + 1, z + 1, 0, 5, 3);
  box(48, 54, 60, 55, 4.5, 5, 5); box(48, 58, 60, 59, 4.5, 5, 5);
  box(62, 50, 63, 58, 1, 4, 8);                                                     // mirror panel

  // defenders' courtyard
  box(28, 52, 36, 53, 0, 1, 2); box(30, 56, 34, 58, 0, 2, 4);
}
buildMap();

/* ===================================================================
   Ray casting (CPU) – same grid walk the shader uses
   =================================================================== */
function raycast(ox, oy, oz, dx, dy, dz, maxT) {
  const px = ox / VS, py = oy / VS, pz = oz / VS;
  let ix = Math.floor(px), iy = Math.floor(py), iz = Math.floor(pz);
  const sx = dx > 0 ? 1 : -1, sy = dy > 0 ? 1 : -1, sz = dz > 0 ? 1 : -1;
  const tdx = dx !== 0 ? Math.abs(1 / dx) : Infinity, tdy = dy !== 0 ? Math.abs(1 / dy) : Infinity, tdz = dz !== 0 ? Math.abs(1 / dz) : Infinity;
  let tmx = dx !== 0 ? (dx > 0 ? ix + 1 - px : px - ix) * tdx : Infinity;
  let tmy = dy !== 0 ? (dy > 0 ? iy + 1 - py : py - iy) * tdy : Infinity;
  let tmz = dz !== 0 ? (dz > 0 ? iz + 1 - pz : pz - iz) * tdz : Infinity;
  const maxG = maxT / VS;
  let t = 0, nx = 0, ny = 0, nz = 0;
  for (let n = 0; n < 700; n++) {
    if (!inb(ix, iy, iz)) return null;
    const id = vox[ix + NX * (iy + NY * iz)];
    if (id) return { t: t * VS, nx, ny, nz, i: ix, j: iy, k: iz, id };
    if (tmx < tmy) {
      if (tmx < tmz) { t = tmx; if (t > maxG) return null; ix += sx; tmx += tdx; nx = -sx; ny = 0; nz = 0; }
      else { t = tmz; if (t > maxG) return null; iz += sz; tmz += tdz; nx = 0; ny = 0; nz = -sz; }
    } else {
      if (tmy < tmz) { t = tmy; if (t > maxG) return null; iy += sy; tmy += tdy; nx = 0; ny = -sy; nz = 0; }
      else { t = tmz; if (t > maxG) return null; iz += sz; tmz += tdz; nx = 0; ny = 0; nz = -sz; }
    }
  }
  return null;
}

function rayEllipsoid(ox, oy, oz, dx, dy, dz, cx, cy, cz, rx, ry, rz) {
  const ax = (ox - cx) / rx, ay = (oy - cy) / ry, az = (oz - cz) / rz;
  const bx = dx / rx, by = dy / ry, bz = dz / rz;
  const a = bx * bx + by * by + bz * bz, b = ax * bx + ay * by + az * bz, c = ax * ax + ay * ay + az * az - 1;
  const disc = b * b - a * c; if (disc < 0) return -1;
  const s = Math.sqrt(disc); let t = (-b - s) / a;
  if (t < 0) { t = (-b + s) / a; if (t < 0) return -1; }
  return t;
}
// bot hitboxes (must match the shader): body ellipsoid + head sphere
const bodyHit = (b, o, d) => rayEllipsoid(o[0], o[1], o[2], d[0], d[1], d[2], b.x, b.y + 0.8, b.z, 0.38, 0.8, 0.38);
const headHit = (b, o, d) => rayEllipsoid(o[0], o[1], o[2], d[0], d[1], d[2], b.x, b.y + 1.78, b.z, 0.2, 0.2, 0.2);

/* ===================================================================
   Physics (AABB vs voxels)
   =================================================================== */
const solid = (i, j, k) => (i < 0 || k < 0 || j < 0 || i >= NX || k >= NZ) ? true : (j >= NY ? false : vox[i + NX * (j + NY * k)] !== 0);
function boxHit(x, y, z, half, h) {
  const e = 0.001;
  const i0 = Math.floor((x - half + e) / VS), i1 = Math.floor((x + half - e) / VS);
  const j0 = Math.floor((y + e) / VS), j1 = Math.floor((y + h - e) / VS);
  const k0 = Math.floor((z - half + e) / VS), k1 = Math.floor((z + half - e) / VS);
  for (let k = k0; k <= k1; k++) for (let j = j0; j <= j1; j++) for (let i = i0; i <= i1; i++) if (solid(i, j, k)) return true;
  return false;
}
function moveHoriz(e, dx, dz, half, h, canStep) {
  let blocked = false;
  if (dx !== 0) {
    const nx = e.x + dx;
    if (!boxHit(nx, e.y, e.z, half, h)) e.x = nx;
    else if (canStep && !boxHit(nx, e.y + VS, e.z, half, h)) { e.x = nx; e.y += VS; }
    else { e.vx = 0; blocked = true; }
  }
  if (dz !== 0) {
    const nz = e.z + dz;
    if (!boxHit(e.x, e.y, nz, half, h)) e.z = nz;
    else if (canStep && !boxHit(e.x, e.y + VS, nz, half, h)) { e.z = nz; e.y += VS; }
    else { e.vz = 0; blocked = true; }
  }
  return blocked;
}
function moveVert(e, dy, half, h) {
  const n = Math.max(1, Math.ceil(Math.abs(dy) / 0.2)), sd = dy / n;
  for (let s = 0; s < n; s++) {
    const ny = e.y + sd;
    if (boxHit(e.x, ny, e.z, half, h)) {
      if (sd < 0) e.y = Math.floor(e.y / VS + 1e-6) * VS;
      e.vy = 0; return;
    }
    e.y = ny;
  }
}

/* ===================================================================
   Game state
   =================================================================== */
const SPAWN = { x: 32, z: 6 };
const P = { x: SPAWN.x, y: GROUND, z: SPAWN.z, vx: 0, vy: 0, vz: 0, yaw: 0, pitch: 0, onGround: true, h: 1.8, eyeH: 1.62,
  hp: 100, armor: 50, dead: false, respawn: 0, fly: false, dashT: 0, dashX: 0, dashZ: 0, cdQ: 0, cdE: 0, jumpBuf: 0, step: 0, stepSmooth: 0 };
const HALF = 0.3, H_STAND = 1.8, H_CROUCH = 1.35, EYE_STAND = 1.62, EYE_CROUCH = 1.2;
const GRAV = 24, JUMP_V = 7.4, RUN = 5.6, WALK = 2.8, CROUCH_SPEED = 2.4;

const W = [
  { name: 'SIDEARM', auto: false, interval: 0.17,  mag: 12, reserve: 36, body: 26,  head: 78,  reload: 1.75, zoom: [],         base: 0.25, run: 2.2, air: 3, hip: 0, snd: 'pistol', kickP: 1.0, kickY: 0.25 },
  { name: 'RIFLE',   auto: true,  interval: 0.1026, mag: 25, reserve: 50, body: 40,  head: 160, reload: 2.5,  zoom: [1.25],     base: 0.12, run: 3.2, air: 4, hip: 0, snd: 'rifle',  kickP: 0.5, kickY: 0.28 },
  { name: 'SNIPER',  auto: false, interval: 1.45,   mag: 5,  reserve: 10, body: 150, head: 255, reload: 3.6,  zoom: [2.5, 5],   base: 0.05, run: 6,   air: 8, hip: 5, snd: 'sniper', kickP: 3.2, kickY: 0.4 }
];
const ammo = W.map(w => ({ mag: w.mag, res: w.reserve }));
let cur = 1;            // 0..2 weapons, 3 build tool
let blockIdx = 0;
let reloading = false, reloadT = 0, fireCd = 0, switchT = 0, fireIdx = 0, lastFire = -9, flashT = 0, kickBack = 0;
let zoomLevel = 0, zoomCur = 1, mouseL = false, mouseR = false, fireEdge = false, hitT = 0, buildCd = 0;
let kills = 0, time = 0, botsAggressive = true, gunLight = 1, showStats = true;

const bots = [];
const fx = [];            // {x,y,z,life,max,r}
const edits = [];         // voxel edits waiting to be uploaded to the GPU

const eyePos = () => [P.x, P.y + P.eyeH + P.stepSmooth, P.z];
function basis(yaw, pitch) {
  const cp = Math.cos(pitch), sp = Math.sin(pitch), cy = Math.cos(yaw), sy = Math.sin(yaw);
  return { f: [cp * sy, sp, cp * cy], r: [cy, 0, -sy], u: [-sp * sy, cp, -sp * cy] };
}

function addFx(x, y, z, life, r) {
  fx.push({ x, y, z, life, max: life, r });
  if (fx.length > 12) fx.shift();
}
function setVoxel(i, j, k, id) { setV(i, j, k, id); edits.push([i, j, k, id]); }

/* ---------- audio (synthesised, no files) ---------- */
let actx = null, noiseBuf = null;
function audio() {
  if (!actx) { try { actx = new (window.AudioContext || window.webkitAudioContext)(); } catch (e) {} }
  if (actx && actx.state === 'suspended') actx.resume();
  return actx;
}
function sfxShot(kind, vol) {
  const a = audio(); if (!a || vol <= 0.01) return;
  const t = a.currentTime;
  if (!noiseBuf) { noiseBuf = a.createBuffer(1, a.sampleRate, a.sampleRate); const d = noiseBuf.getChannelData(0); for (let i = 0; i < d.length; i++) d[i] = Math.random() * 2 - 1; }
  const src = a.createBufferSource(); src.buffer = noiseBuf;
  const f = a.createBiquadFilter(); f.type = 'lowpass';
  f.frequency.setValueAtTime(kind === 'sniper' ? 2600 : kind === 'rifle' ? 3800 : 5200, t); f.frequency.exponentialRampToValueAtTime(260, t + 0.16);
  const g = a.createGain(); const dur = kind === 'sniper' ? 0.7 : 0.2;
  g.gain.setValueAtTime(0.55 * vol, t); g.gain.exponentialRampToValueAtTime(0.001, t + dur);
  src.connect(f); f.connect(g); g.connect(a.destination); src.start(t); src.stop(t + dur + 0.05);
  const o = a.createOscillator(); o.frequency.setValueAtTime(150, t); o.frequency.exponentialRampToValueAtTime(40, t + 0.13);
  const og = a.createGain(); og.gain.setValueAtTime(0.5 * vol, t); og.gain.exponentialRampToValueAtTime(0.001, t + 0.15);
  o.connect(og); og.connect(a.destination); o.start(t); o.stop(t + 0.2);
}
function sfxTone(freq, dur, vol, type) {
  const a = audio(); if (!a) return;
  const t = a.currentTime, o = a.createOscillator(), g = a.createGain();
  o.type = type || 'sine'; o.frequency.setValueAtTime(freq, t);
  g.gain.setValueAtTime(vol, t); g.gain.exponentialRampToValueAtTime(0.001, t + dur);
  o.connect(g); g.connect(a.destination); o.start(t); o.stop(t + dur + 0.02);
}
function sfxStep() {
  const a = audio(); if (!a) return;
  if (!noiseBuf) sfxTone(1, 0.01, 0.0001);
  if (!noiseBuf) return;
  const t = a.currentTime, s = a.createBufferSource(); s.buffer = noiseBuf;
  const f = a.createBiquadFilter(); f.type = 'lowpass'; f.frequency.value = 420;
  const g = a.createGain(); g.gain.setValueAtTime(0.18, t); g.gain.exponentialRampToValueAtTime(0.001, t + 0.07);
  s.connect(f); f.connect(g); g.connect(a.destination); s.start(t); s.stop(t + 0.09);
}

/* ---------- HUD helpers ---------- */
const hud = {
  hp: $('hp'), arv: $('arv'), wname: $('wname'), ammo: $('ammo'), score: $('score'), feed: $('feed'), toast: $('toast'),
  hit: $('hit'), dmg: $('dmg'), dmgdir: $('dmgdir'), scope: $('scope'), xh: $('xh'), stats: $('stats'),
  abQ: $('abQ'), abE: $('abE'), slots: $('slots'), blockSel: $('blockSel'), swatch: $('swatch'), bname: $('bname')
};
let toastTimer = 0;
function toast(msg) { hud.toast.textContent = msg; hud.toast.classList.add('on'); clearTimeout(toastTimer); toastTimer = setTimeout(() => hud.toast.classList.remove('on'), 1600); }
function feed(msg, head) {
  const d = document.createElement('div'); d.textContent = msg; if (head) d.className = 'head';
  hud.feed.appendChild(d); setTimeout(() => d.remove(), 4200);
  while (hud.feed.children.length > 5) hud.feed.firstChild.remove();
}
function flashClass(el, cls, ms) { el.classList.add(cls); void el.offsetWidth; setTimeout(() => el.classList.remove(cls), ms); }

/* ---------- weapons ---------- */
function viewDir() { return basis(P.yaw, P.pitch).f; }

function spreadDeg(w) {
  const sp = Math.hypot(P.vx, P.vz);
  let s = w.base;
  if (!P.onGround && !P.fly) s += w.air;
  else if (sp > 3.4) s += w.run * Math.min(sp / RUN, 1);
  if (P.h < 1.5) s *= 0.65;
  if (w.hip && zoomLevel === 0) s += w.hip;
  s += Math.min(fireIdx, 10) * (w.auto ? 0.07 : 0.1);
  return s;
}

function startReload() {
  const w = W[cur], a = ammo[cur];
  if (cur > 2 || reloading || a.mag >= w.mag || a.res <= 0) return;
  reloading = true; reloadT = w.reload; zoomLevel = 0;
  sfxTone(300, 0.08, 0.15, 'square');
}
function finishReload() {
  const w = W[cur], a = ammo[cur];
  const need = Math.min(w.mag - a.mag, a.res);
  a.mag += need; a.res -= need; reloading = false;
  sfxTone(520, 0.08, 0.15, 'square');
}

function shoot() {
  const w = W[cur], a = ammo[cur];
  a.mag--;
  const e = eyePos(), B = basis(P.yaw, P.pitch);
  let d = B.f.slice();
  const sp = spreadDeg(w) * Math.PI / 180;
  if (sp > 0) {
    const ang = Math.random() * Math.PI * 2, rr = Math.sqrt(Math.random()) * Math.tan(sp);
    for (let i = 0; i < 3; i++) d[i] += B.r[i] * rr * Math.cos(ang) + B.u[i] * rr * Math.sin(ang);
    const l = Math.hypot(d[0], d[1], d[2]); d = d.map(v => v / l);
  }
  // what did we hit?
  const wh = raycast(e[0], e[1], e[2], d[0], d[1], d[2], 200);
  let best = wh ? wh.t : 200, target = null;
  for (const b of bots) {
    if (!b.alive) continue;
    const th = headHit(b, e, d), tb = bodyHit(b, e, d);
    if (th > 0 && th < best) { best = th; target = { b, head: true }; }
    if (tb > 0 && tb < best) { best = tb; target = { b, head: false }; }
  }
  const hx = e[0] + d[0] * best, hy = e[1] + d[1] * best, hz = e[2] + d[2] * best;
  if (target) {
    damageBot(target.b, target.head ? w.head : w.body, target.head);
    addFx(hx - d[0] * 0.05, hy, hz - d[2] * 0.05, 0.12, 0.07);
  } else if (wh) {
    addFx(hx - d[0] * 0.03, hy - d[1] * 0.03, hz - d[2] * 0.03, 0.3, 0.05);
  }
  // feel: recoil, flash, sound
  const idx = Math.min(fireIdx, 14);
  let kp = w.kickP, ky = 0;
  if (w.auto) { kp = w.kickP + 0.035 * idx; ky = w.kickY * Math.sin(idx * 0.9) * (idx > 2 ? 1 : 0.2) + (idx > 10 ? rand(-0.15, 0.15) : 0); }
  else ky = rand(-w.kickY, w.kickY);
  P.pitch = clamp(P.pitch + kp * Math.PI / 180, -1.5, 1.5);
  P.yaw += ky * Math.PI / 180;
  recoilAcc += kp;
  fireIdx++; lastFire = time; flashT = 0.06; kickBack = Math.min(kickBack + (w.auto ? 0.03 : 0.06), 0.09);
  sfxShot(w.snd, 1);
  if (cur === 2) { zoomLevel = 0; }   // sniper un-scopes after the shot
  if (a.mag <= 0) setTimeout(() => { if (!reloading && ammo[cur] === a && cur < 3) startReload(); }, 250);
}
let recoilAcc = 0;

function damageBot(b, dmg, head) {
  b.hp -= dmg; b.hitFlash = 0.1;
  hitT = 0.25; hud.hit.classList.toggle('head', head);
  flashClass(hud.hit, 'on', 80);
  sfxTone(head ? 1900 : 1300, 0.06, 0.12, 'triangle');
  if (b.hp <= 0) {
    b.alive = false; b.respawn = 4; kills++;
    feed('YOU  ▸  BOT' + (head ? '   ◎ HEADSHOT' : ''), head);
    sfxTone(700, 0.18, 0.18, 'sawtooth');
    for (let i = 0; i < 4; i++) addFx(b.x + rand(-.3, .3), b.y + rand(.4, 1.6), b.z + rand(-.3, .3), 0.35, 0.09);
  }
}

function damagePlayer(dmg, sx, sz) {
  if (P.dead) return;
  const absorbed = Math.min(P.armor, dmg * 0.66);
  P.armor -= absorbed; P.hp -= dmg - absorbed;
  flashClass(hud.dmg, 'on', 90);
  const ang = Math.atan2(sx - P.x, sz - P.z) - P.yaw;
  hud.dmgdir.style.transform = 'rotate(' + (ang * 180 / Math.PI) + 'deg)';
  flashClass(hud.dmgdir, 'on', 120);
  sfxTone(180, 0.1, 0.2, 'square');
  if (P.hp <= 0) {
    P.hp = 0; P.dead = true; P.respawn = 2.8; mouseL = false; mouseR = false;
    feed('BOT  ▸  YOU'); toast('ELIMINATED');
  }
}
function respawnPlayer() {
  P.x = SPAWN.x; P.y = GROUND; P.z = SPAWN.z; P.vx = P.vy = P.vz = 0; P.hp = 100; P.armor = 50; P.dead = false;
  P.yaw = 0; P.pitch = 0; for (let i = 0; i < 3; i++) { ammo[i].mag = W[i].mag; ammo[i].res = W[i].reserve; }
  reloading = false; R.reset = true; toast('RESPAWNED');
}

/* ---------- bots ---------- */
function findSpot() {
  for (let n = 0; n < 80; n++) {
    const x = rand(3, 61), z = rand(3, 61);
    if (!boxHit(x, GROUND, z, 0.45, 1.95) && Math.hypot(x - P.x, z - P.z) > 12) return { x, z };
  }
  return { x: 32, z: 30 };
}
function spawnBot() {
  if (bots.length >= 8) { toast('MAX 8 BOTS'); return; }
  const s = findSpot();
  bots.push({ x: s.x, y: GROUND, z: s.z, vx: 0, vz: 0, hp: 100, alive: true, respawn: 0, strafeDir: 1, strafeT: 0, react: 0, shootT: 0.5, burst: 0, wanderT: 0, wanderA: 0, hitFlash: 0 });
}
function updateBots(dt) {
  const eyeP = P.y + P.eyeH;
  for (const b of bots) {
    if (!b.alive) { b.respawn -= dt; if (b.respawn <= 0) { const s = findSpot(); b.x = s.x; b.z = s.z; b.hp = 100; b.alive = true; b.react = 0; } continue; }
    b.hitFlash = Math.max(0, b.hitFlash - dt);
    const dx = P.x - b.x, dz = P.z - b.z, dist = Math.hypot(dx, dz) || 1;
    const nx = dx / dist, nz = dz / dist;
    let los = false;
    if (!P.dead && dist < 60) {
      const ey = eyeP - (b.y + 1.6), d3 = Math.hypot(dx, ey, dz);
      const h = raycast(b.x, b.y + 1.6, b.z, dx / d3, ey / d3, dz / d3, d3);
      los = !h || h.t >= d3 - 0.4;
    }
    if (!botsAggressive) continue;
    b.strafeT -= dt;
    if (b.strafeT <= 0) { b.strafeDir = Math.random() < 0.2 ? 0 : (Math.random() < 0.5 ? -1 : 1); b.strafeT = rand(0.6, 1.8); }
    let mx = 0, mz = 0, sp = 2.6;
    if (b.wanderT > 0) { b.wanderT -= dt; mx = Math.cos(b.wanderA); mz = Math.sin(b.wanderA); sp = 3; }
    else if (los) {
      mx = -nz * b.strafeDir; mz = nx * b.strafeDir;
      if (dist > 22) { mx += nx * 0.7; mz += nz * 0.7; } else if (dist < 7) { mx -= nx * 0.7; mz -= nz * 0.7; }
    } else { mx = nx; mz = nz; sp = 3.3; }
    if (moveHoriz(b, mx * sp * dt, mz * sp * dt, 0.38, 1.95, false) && !los) { b.wanderT = rand(0.8, 1.6); b.wanderA = Math.random() * Math.PI * 2; }
    if (los && dist < 55) {
      b.react += dt;
      if (b.react > 0.55) {
        b.shootT -= dt;
        if (b.shootT <= 0) {
          botFire(b, dist, nx, nz);
          b.burst++; if (b.burst >= 3) { b.burst = 0; b.shootT = rand(0.8, 1.4); } else b.shootT = 0.13;
        }
      }
    } else { b.react = 0; b.burst = 0; }
  }
}
function botFire(b, dist, nx, nz) {
  addFx(b.x + nx * 0.55, b.y + 1.3, b.z + nz * 0.55, 0.05, 0.1);
  sfxShot('rifle', clamp(1.1 - dist / 55, 0, 0.7));
  const moving = Math.hypot(P.vx, P.vz) > 3;
  const p = clamp(0.62 - dist * 0.014 - (moving ? 0.15 : 0), 0.05, 0.6);
  if (Math.random() < p) damagePlayer(Math.random() < 0.1 ? 40 : 9, b.x, b.z);
}
function updateFx(dt) { for (let i = fx.length - 1; i >= 0; i--) { fx[i].life -= dt; if (fx[i].life <= 0) fx.splice(i, 1); } }

/* ---------- building ---------- */
function targetVoxel() { const e = eyePos(), f = viewDir(); return raycast(e[0], e[1], e[2], f[0], f[1], f[2], 10); }
function voxelHitsBox(i, j, k, x, y, z, half, h) {
  return (i + 1) * VS > x - half && i * VS < x + half && (j + 1) * VS > y && j * VS < y + h && (k + 1) * VS > z - half && k * VS < z + half;
}
function placeBlock() {
  const h = targetVoxel(); if (!h) return;
  const i = h.i + h.nx, j = h.j + h.ny, k = h.k + h.nz;
  if (!inb(i, j, k) || j >= NY - 1 || getV(i, j, k)) return;
  if (voxelHitsBox(i, j, k, P.x, P.y, P.z, HALF, P.h)) return;
  for (const b of bots) if (b.alive && voxelHitsBox(i, j, k, b.x, b.y, b.z, 0.4, 1.95)) return;
  setVoxel(i, j, k, MATS[blockIdx].id); sfxTone(260, 0.07, 0.15, 'square');
}
function breakBlock() {
  const h = targetVoxel(); if (!h || h.j === 0) return;
  setVoxel(h.i, h.j, h.k, 0); sfxTone(150, 0.09, 0.2, 'sawtooth');
}

/* ---------- player update ---------- */
function equip(n) {
  if (n === cur || P.dead) return;
  cur = n; reloading = false; zoomLevel = 0; switchT = 0.3; fireIdx = 0;
  sfxTone(420, 0.05, 0.1, 'square');
  if (n === 3) toast('BUILD TOOL'); else toast(W[n].name);
}

function updatePlayer(dt) {
  if (P.dead) { P.respawn -= dt; if (P.respawn <= 0) respawnPlayer(); return; }
  P.cdQ = Math.max(0, P.cdQ - dt); P.cdE = Math.max(0, P.cdE - dt);
  P.jumpBuf = Math.max(0, P.jumpBuf - dt);

  const wantCrouch = keys.has('KeyC') || keys.has('ControlLeft');
  if (wantCrouch && !P.fly) { if (P.h !== H_CROUCH) P.h = H_CROUCH; }
  else if (P.h !== H_STAND && !boxHit(P.x, P.y, P.z, HALF, H_STAND)) P.h = H_STAND;
  P.eyeH = lerp(P.eyeH, P.h === H_STAND ? EYE_STAND : EYE_CROUCH, Math.min(1, dt * 14));
  P.stepSmooth = lerp(P.stepSmooth, 0, Math.min(1, dt * 12));

  let ix = (keys.has('KeyD') ? 1 : 0) - (keys.has('KeyA') ? 1 : 0), iz = (keys.has('KeyW') ? 1 : 0) - (keys.has('KeyS') ? 1 : 0);
  const il = Math.hypot(ix, iz); if (il > 0) { ix /= il; iz /= il; }
  const sy = Math.sin(P.yaw), cy = Math.cos(P.yaw);
  const wx = sy * iz + cy * ix, wz = cy * iz - sy * ix;

  if (P.fly) {
    const B = basis(P.yaw, P.pitch), sp = keys.has('ShiftLeft') ? 22 : 9;
    let vy = (keys.has('Space') ? 1 : 0) - (keys.has('KeyC') ? 1 : 0);
    P.vx = approach(P.vx, (B.f[0] * iz + B.r[0] * ix) * sp, 90 * dt);
    P.vz = approach(P.vz, (B.f[2] * iz + B.r[2] * ix) * sp, 90 * dt);
    P.vy = approach(P.vy, (B.f[1] * iz + vy) * sp, 90 * dt);
    const hh = P.h;
    moveHoriz(P, P.vx * dt, P.vz * dt, HALF, hh, false); moveVert(P, P.vy * dt, HALF, hh);
    P.onGround = false;
    return;
  }

  const walking = keys.has('ShiftLeft') || keys.has('ShiftRight');
  const maxSp = P.h === H_CROUCH ? CROUCH_SPEED : walking ? WALK : RUN;
  const accel = P.onGround ? 80 : 9;
  if (P.dashT > 0) { P.dashT -= dt; P.vx = P.dashX * 18; P.vz = P.dashZ * 18; P.vy = 0; }
  else { P.vx = approach(P.vx, wx * maxSp, accel * dt); P.vz = approach(P.vz, wz * maxSp, accel * dt); }

  if (P.jumpBuf > 0 && P.onGround && P.dashT <= 0) { P.vy = JUMP_V; P.jumpBuf = 0; P.onGround = false; }
  if (P.dashT <= 0) P.vy -= GRAV * dt;
  const fallV = P.vy, y0 = P.y;
  moveHoriz(P, P.vx * dt, P.vz * dt, HALF, P.h, P.onGround);
  if (P.y - y0 >= 0.4) P.stepSmooth -= (P.y - y0);
  moveVert(P, P.vy * dt, HALF, P.h);
  const was = P.onGround;
  P.onGround = P.vy <= 0 && boxHit(P.x, P.y - 0.02, P.z, HALF, P.h);
  if (P.onGround && !was && fallV < -8) sfxStep();

  if (P.onGround && !walking && P.h === H_STAND) {
    const sp = Math.hypot(P.vx, P.vz);
    if (sp > 3.5) { P.step += dt * sp; if (P.step > 2.3) { P.step = 0; sfxStep(); } }
  }
}

function useQ() { if (P.cdQ > 0 || P.dead) return; P.cdQ = 8; P.vy = 11; P.onGround = false; sfxTone(500, 0.3, 0.12, 'sine'); }
function useE() {
  if (P.cdE > 0 || P.dead) return; P.cdE = 6;
  let ix = (keys.has('KeyD') ? 1 : 0) - (keys.has('KeyA') ? 1 : 0), iz = (keys.has('KeyW') ? 1 : 0) - (keys.has('KeyS') ? 1 : 0);
  if (!ix && !iz) iz = 1;
  const l = Math.hypot(ix, iz); ix /= l; iz /= l;
  const sy = Math.sin(P.yaw), cy = Math.cos(P.yaw);
  P.dashX = sy * iz + cy * ix; P.dashZ = cy * iz - sy * ix; P.dashT = 0.2;
  sfxTone(240, 0.22, 0.14, 'sawtooth');
}

function updateWeapon(dt) {
  time += dt;
  flashT = Math.max(0, flashT - dt); kickBack = Math.max(0, kickBack - dt * 0.5);
  hitT = Math.max(0, hitT - dt); switchT = Math.max(0, switchT - dt); buildCd = Math.max(0, buildCd - dt);
  if (time - lastFire > 0.28) fireIdx = 0;
  // view recovers toward where it was before the spray
  if (time - lastFire > 0.12 && recoilAcc > 0) {
    const rec = Math.min(recoilAcc, recoilAcc * (1 - Math.exp(-dt * 7)) + 0.02 * dt);
    P.pitch -= rec * Math.PI / 180; recoilAcc -= rec;
  }
  const w = cur < 3 ? W[cur] : null;
  // zoom
  const zt = (w && zoomLevel > 0 && w.zoom.length) ? w.zoom[zoomLevel - 1] : 1;
  zoomCur = lerp(zoomCur, zt, Math.min(1, dt * 14));
  if (P.dead) return;
  if (w) {
    fireCd -= dt;
    if (reloading) { reloadT -= dt; if (reloadT <= 0) finishReload(); }
    const wantFire = mouseL && (w.auto || fireEdge);
    if (wantFire && fireCd <= 0 && !reloading && switchT <= 0) {
      if (ammo[cur].mag > 0) { shoot(); fireCd = w.interval; } else startReload();
    }
  } else if (mouseL && buildCd <= 0 && cur === 3) { placeBlock(); buildCd = 0.18; } else if (mouseR && buildCd <= 0 && cur === 3) { breakBlock(); buildCd = 0.18; }
  fireEdge = false;
}

function updateAll(dt) {
  updatePlayer(dt); updateWeapon(dt); updateBots(dt); updateFx(dt);
  // is the player in sunlight? (drives the first-person weapon lighting)
  const e = eyePos(), sd = R.sunDir;
  const blocked = raycast(e[0], e[1] + 0.2, e[2], sd[0], sd[1], sd[2], 80);
  gunLight = lerp(gunLight, blocked ? 0.2 : 1, Math.min(1, dt * 4));
}

/* ===================================================================
   Renderer (WebGL2)
   =================================================================== */
const R = { ok: false, reset: true, frame: 0, sunDir: [0, 1, 0], prev: null, zoomT: 1 };
const canvas = $('gl');
let gl, vao, progTrace, progResolve, progPresent, LT, LR, LP, voxTex, targets = null;

const VS_SRC = `#version 300 es
out vec2 vUv;
void main() {
  vec2 p = vec2(float((gl_VertexID << 1) & 2), float(gl_VertexID & 2));
  vUv = p;
  gl_Position = vec4(p * 2.0 - 1.0, 0.0, 1.0);
}`;

/* ---------- pass 1: path tracer on the voxel grid ---------- */
const FS_TRACE = `#version 300 es
precision highp float;
precision highp int;
precision highp sampler3D;
precision highp usampler3D;

in vec2 vUv;
out vec4 outColor;

const int NX = ${NX};
const int NY = ${NY};
const int NZ = ${NZ};
const float VS = ${VS.toFixed(3)};
const float PI = 3.14159265;

uniform usampler3D uVox;
uniform vec2 uRes;
uniform vec3 uCamPos;
uniform vec3 uCamRight;
uniform vec3 uCamUp;
uniform vec3 uCamFwd;
uniform float uTanHalf;
uniform vec2 uJitter;
uniform uint uFrame;
uniform vec3 uSunDir;
uniform vec3 uSunCol;
uniform vec3 uSkyZ;
uniform vec3 uSkyH;
uniform int uBounces;
uniform int uReflect;
uniform vec4 uBots[8];
uniform int uBotCount;
uniform vec4 uFx[12];
uniform int uFxCount;
uniform vec4 uSel;
uniform vec4 uFlash;
uniform float uTime;

uint rngState;
float hT;
vec3 hN;
int hKind;
int hId;
ivec3 hCell;
vec3 hAlb;
float hRough;
float hMetal;
vec3 hEmis;
int gB;

uint pcg(uint v) {
  uint s = v * 747796405u + 2891336453u;
  uint w = ((s >> ((s >> 28u) + 4u)) ^ s) * 277803737u;
  return (w >> 22u) ^ w;
}
float rnd() {
  rngState = pcg(rngState);
  return float(rngState) * (1.0 / 4294967296.0);
}
float hash13(vec3 p) {
  p = fract(p * 0.1031);
  p += dot(p, p.zyx + 31.32);
  return fract((p.x + p.y) * p.z);
}
vec3 randUnit() {
  float z = rnd() * 2.0 - 1.0;
  float a = rnd() * 6.2831853;
  float r = sqrt(max(0.0, 1.0 - z * z));
  return vec3(r * cos(a), r * sin(a), z);
}
vec3 cosineDir(vec3 n) {
  float u1 = rnd();
  float u2 = rnd();
  float r = sqrt(u1);
  float phi = 6.2831853 * u2;
  vec3 up = abs(n.y) < 0.99 ? vec3(0.0, 1.0, 0.0) : vec3(1.0, 0.0, 0.0);
  vec3 t = normalize(cross(n, up));
  vec3 b = cross(n, t);
  return normalize(t * (r * cos(phi)) + b * (r * sin(phi)) + n * sqrt(max(0.0, 1.0 - u1)));
}

vec3 skyCol(vec3 d, bool disk) {
  float y = d.y;
  float tH = pow(clamp(1.0 - max(y, 0.0), 0.0, 1.0), 4.0);
  vec3 c = mix(uSkyZ, uSkyH, tH);
  if (y < 0.0) c = mix(uSkyH, uSkyH * 0.35, clamp(-y * 3.0, 0.0, 1.0));
  float sd = max(dot(d, uSunDir), 0.0);
  c += uSunCol * (pow(sd, 6.0) * 0.18 + pow(sd, 64.0) * 0.35);
  if (disk) c += uSunCol * 25.0 * smoothstep(0.9993, 0.9998, sd);
  return c;
}

bool hitEllipsoid(vec3 ro, vec3 rd, vec3 c, vec3 r, out float t, out vec3 n) {
  t = 0.0;
  n = vec3(0.0, 1.0, 0.0);
  vec3 o = (ro - c) / r;
  vec3 d = rd / r;
  float a = dot(d, d);
  float b = dot(o, d);
  float cc = dot(o, o) - 1.0;
  float disc = b * b - a * cc;
  if (disc < 0.0) return false;
  float s = sqrt(disc);
  float t0 = (-b - s) / a;
  if (t0 < 1e-3) {
    t0 = (-b + s) / a;
    if (t0 < 1e-3) return false;
  }
  t = t0;
  vec3 p = o + d * t0;
  n = normalize(p / r);
  return true;
}

vec3 safeDir(vec3 d) {
  return vec3(abs(d.x) < 1e-6 ? (d.x < 0.0 ? -1e-6 : 1e-6) : d.x,
              abs(d.y) < 1e-6 ? (d.y < 0.0 ? -1e-6 : 1e-6) : d.y,
              abs(d.z) < 1e-6 ? (d.z < 0.0 ? -1e-6 : 1e-6) : d.z);
}

// Amanatides-Woo grid traversal through the voxel texture.
bool traceVox(vec3 ro, vec3 rd, int maxSteps, out float tHit, out vec3 nrm, out int id, out ivec3 cellOut) {
  tHit = 0.0;
  nrm = vec3(0.0, 1.0, 0.0);
  id = 0;
  cellOut = ivec3(0);
  vec3 d = safeDir(rd);
  vec3 inv = 1.0 / d;
  vec3 p = ro / VS;
  vec3 bmax = vec3(float(NX), float(NY), float(NZ));
  vec3 t0 = (vec3(0.0) - p) * inv;
  vec3 t1 = (bmax - p) * inv;
  vec3 tmn = min(t0, t1);
  vec3 tmx = max(t0, t1);
  float tEnter = max(max(tmn.x, tmn.y), max(tmn.z, 0.0));
  float tExit = min(min(tmx.x, tmx.y), tmx.z);
  if (tEnter >= tExit) return false;
  if (tEnter > 0.0) {
    if (tmn.x >= tmn.y && tmn.x >= tmn.z) nrm = vec3(-sign(d.x), 0.0, 0.0);
    else if (tmn.y >= tmn.z) nrm = vec3(0.0, -sign(d.y), 0.0);
    else nrm = vec3(0.0, 0.0, -sign(d.z));
  }
  vec3 pos = p + d * (tEnter + 1e-4);
  ivec3 cell = clamp(ivec3(floor(pos)), ivec3(0), ivec3(NX - 1, NY - 1, NZ - 1));
  ivec3 stp = ivec3(sign(d));
  vec3 tDelta = abs(inv);
  vec3 bnd = vec3(cell) + step(vec3(0.0), d);
  vec3 tMax = (bnd - p) * inv;
  float t = tEnter;
  for (int i = 0; i < 420; i++) {
    if (i >= maxSteps) break;
    uint v = texelFetch(uVox, cell, 0).r;
    if (v != 0u) {
      tHit = t * VS;
      id = int(v);
      cellOut = cell;
      return true;
    }
    if (tMax.x < tMax.y) {
      if (tMax.x < tMax.z) { cell.x += stp.x; t = tMax.x; tMax.x += tDelta.x; nrm = vec3(-float(stp.x), 0.0, 0.0); }
      else { cell.z += stp.z; t = tMax.z; tMax.z += tDelta.z; nrm = vec3(0.0, 0.0, -float(stp.z)); }
    } else {
      if (tMax.y < tMax.z) { cell.y += stp.y; t = tMax.y; tMax.y += tDelta.y; nrm = vec3(0.0, -float(stp.y), 0.0); }
      else { cell.z += stp.z; t = tMax.z; tMax.z += tDelta.z; nrm = vec3(0.0, 0.0, -float(stp.z)); }
    }
    if (cell.x < 0 || cell.y < 0 || cell.z < 0 || cell.x >= NX || cell.y >= NY || cell.z >= NZ) return false;
  }
  return false;
}

bool occluded(vec3 ro, vec3 rd, float maxDist, int steps) {
  float t;
  vec3 n;
  int id;
  ivec3 c;
  if (traceVox(ro, rd, steps, t, n, id, c) && t < maxDist) return true;
  for (int i = 0; i < 8; i++) {
    if (i >= uBotCount) break;
    vec4 b = uBots[i];
    if (b.w < 0.5) continue;
    float tb;
    vec3 nb;
    if (hitEllipsoid(ro, rd, b.xyz + vec3(0.0, 0.8, 0.0), vec3(0.38, 0.8, 0.38), tb, nb) && tb < maxDist) return true;
    if (hitEllipsoid(ro, rd, b.xyz + vec3(0.0, 1.78, 0.0), vec3(0.2), tb, nb) && tb < maxDist) return true;
  }
  return false;
}

bool intersectScene(vec3 ro, vec3 rd, int steps) {
  float t;
  vec3 n;
  int id;
  ivec3 c;
  bool found = false;
  hT = 1e9;
  if (traceVox(ro, rd, steps, t, n, id, c)) { hT = t; hN = n; hId = id; hCell = c; hKind = 0; found = true; }
  for (int i = 0; i < 8; i++) {
    if (i >= uBotCount) break;
    vec4 b = uBots[i];
    if (b.w < 0.5) continue;
    float tb;
    vec3 nb;
    if (hitEllipsoid(ro, rd, b.xyz + vec3(0.0, 0.8, 0.0), vec3(0.38, 0.8, 0.38), tb, nb) && tb < hT) { hT = tb; hN = nb; hKind = 1; found = true; }
    if (hitEllipsoid(ro, rd, b.xyz + vec3(0.0, 1.78, 0.0), vec3(0.2), tb, nb) && tb < hT) { hT = tb; hN = nb; hKind = 2; found = true; }
  }
  for (int i = 0; i < 12; i++) {
    if (i >= uFxCount) break;
    vec4 f = uFx[i];
    if (f.w <= 0.0) continue;
    float tb;
    vec3 nb;
    if (hitEllipsoid(ro, rd, f.xyz, vec3(f.w), tb, nb) && tb < hT) { hT = tb; hN = nb; hKind = 3; found = true; }
  }
  return found;
}

void setMaterial(vec3 p) {
  hRough = 0.8;
  hMetal = 0.0;
  hEmis = vec3(0.0);
  hAlb = vec3(0.7);
  if (hKind == 0) {
    int id = hId;
    if (id == 1) { hAlb = vec3(0.50, 0.50, 0.53); hRough = 0.9; }
    else if (id == 2) { hAlb = vec3(0.80, 0.76, 0.68); hRough = 0.95; }
    else if (id == 3) { hAlb = vec3(0.55, 0.57, 0.62); hRough = 0.28; hMetal = 1.0; }
    else if (id == 4) { hAlb = vec3(0.45, 0.30, 0.17); hRough = 0.7; }
    else if (id == 5) { hAlb = vec3(1.0); hEmis = vec3(1.0, 0.85, 0.6) * 6.0; }
    else if (id == 6) { hAlb = vec3(0.78, 0.10, 0.13); hRough = 0.6; }
    else if (id == 7) { hAlb = vec3(0.08, 0.62, 0.58); hRough = 0.6; }
    else if (id == 8) { hAlb = vec3(0.95, 0.96, 0.98); hRough = 0.02; hMetal = 1.0; }
    else if (id == 9) { hAlb = vec3(0.78, 0.80, 0.84); hRough = 0.06; }
    vec3 lp = p / VS - vec3(hCell);
    vec3 q = abs(lp - 0.5) * 2.0;
    vec3 an = abs(hN);
    float e = max(max(q.x * (1.0 - an.x), q.y * (1.0 - an.y)), q.z * (1.0 - an.z));
    float edge = smoothstep(0.86, 1.0, e);
    float h = hash13(vec3(hCell));
    float tint = 0.92 + 0.16 * h;
    if (id == 1 && hN.y > 0.5) {
      vec2 tp = fract(p.xz / 2.0);
      float line = smoothstep(0.0, 0.012, min(min(tp.x, tp.y), min(1.0 - tp.x, 1.0 - tp.y)));
      tint *= 0.78 + 0.22 * line;
    }
    hAlb *= tint * (1.0 - 0.2 * edge);
    if (uSel.w > 0.5 && gB == 0 && hCell == ivec3(uSel.xyz)) {
      float sel = smoothstep(0.80, 0.9, e);
      hEmis += vec3(1.0, 0.92, 0.55) * sel * (1.6 + 0.6 * sin(uTime * 6.0));
    }
    if (uReflect == 0) hMetal = 0.0;
  } else if (hKind == 1) {
    hAlb = vec3(0.85, 0.10, 0.12); hRough = 0.45; hEmis = vec3(0.45, 0.02, 0.04);
  } else if (hKind == 2) {
    hAlb = vec3(0.92, 0.62, 0.52); hRough = 0.5; hEmis = vec3(0.30, 0.15, 0.12);
  } else {
    hAlb = vec3(0.0); hEmis = vec3(1.0, 0.65, 0.25) * 9.0;
  }
}

vec3 sunLight(vec3 p, vec3 n) {
  vec3 L = uSunDir;
  vec3 up = abs(L.y) < 0.99 ? vec3(0.0, 1.0, 0.0) : vec3(1.0, 0.0, 0.0);
  vec3 u = normalize(cross(L, up));
  vec3 v = cross(L, u);
  float r = sqrt(rnd()) * 0.045;
  float a = rnd() * 6.2831853;
  vec3 Ls = normalize(L + (u * cos(a) + v * sin(a)) * r);
  float ndl = dot(n, Ls);
  if (ndl <= 0.0) return vec3(0.0);
  if (occluded(p + n * 0.004, Ls, 1e9, 300)) return vec3(0.0);
  return uSunCol * (6.0 * ndl / PI);
}

vec3 flashLight(vec3 p, vec3 n) {
  if (uFlash.w <= 0.001) return vec3(0.0);
  vec3 dl = uFlash.xyz - p;
  float d2 = max(dot(dl, dl), 0.04);
  float dist = sqrt(d2);
  vec3 Ld = dl / dist;
  float ndl = dot(n, Ld);
  if (ndl <= 0.0) return vec3(0.0);
  if (occluded(p + n * 0.004, Ld, dist - 0.05, 80)) return vec3(0.0);
  return vec3(1.0, 0.72, 0.38) * (uFlash.w * ndl / (d2 * PI));
}

vec3 pathTrace(vec3 ro, vec3 rd, out float primaryT) {
  vec3 Lo = vec3(0.0);
  vec3 thr = vec3(1.0);
  bool spec = true;
  int diff = 0;
  primaryT = 1000.0;
  for (int seg = 0; seg < 6; seg++) {
    gB = seg;
    int steps = (seg == 0) ? 420 : 280;
    if (!intersectScene(ro, rd, steps)) {
      Lo += thr * skyCol(rd, spec);
      break;
    }
    if (seg == 0) primaryT = hT;
    vec3 p = ro + rd * hT;
    vec3 n = hN;
    setMaterial(p);
    if (dot(n, rd) > 0.0) n = -n;
    Lo += thr * hEmis;
    vec3 alb = hAlb;
    float metal = hMetal;
    float rough = hRough;
    float cosv = clamp(dot(-rd, n), 0.0, 1.0);
    float specProb = 0.0;
    if (metal > 0.5) specProb = 1.0;
    else if (uReflect == 1 && rough < 0.5) specProb = 0.04 + 0.96 * pow(1.0 - cosv, 5.0);
    bool doSpec = rnd() < specProb;
    if (!doSpec) {
      vec3 direct = sunLight(p, n);
      if (seg == 0) direct += flashLight(p, n);
      Lo += thr * alb * direct;
      if (diff >= uBounces) {
        vec3 ad = cosineDir(n);
        vec3 amb;
        if (occluded(p + n * 0.004, ad, 1e9, 160)) amb = uSkyH * 0.07 + vec3(0.012);
        else amb = skyCol(ad, false);
        Lo += thr * alb * amb;
        break;
      }
      diff++;
      thr *= alb;
      rd = cosineDir(n);
      spec = false;
    } else {
      vec3 F0 = mix(vec3(0.04), alb, metal);
      if (metal > 0.5) thr *= F0 + (1.0 - F0) * pow(1.0 - cosv, 5.0);
      rd = reflect(rd, n);
      rd = normalize(rd + randUnit() * rough * rough);
      if (dot(rd, n) < 0.0) rd = reflect(rd, n);
      spec = true;
    }
    ro = p + n * 0.004;
  }
  return Lo;
}

void main() {
  rngState = pcg(uint(gl_FragCoord.x) + pcg(uint(gl_FragCoord.y) * 2654435761u + pcg(uFrame)));
  vec2 px = gl_FragCoord.xy + uJitter;
  vec2 ndc = px / uRes * 2.0 - 1.0;
  float asp = uRes.x / uRes.y;
  vec3 rd = normalize(uCamFwd + uCamRight * (ndc.x * uTanHalf * asp) + uCamUp * (ndc.y * uTanHalf));
  float pt;
  vec3 col = pathTrace(uCamPos, rd, pt);
  if (any(isnan(col)) || any(isinf(col))) col = vec3(0.0);
  col = min(col, vec3(24.0));
  outColor = vec4(col, pt);
}`;

/* ---------- pass 2: temporal upscale / accumulate ---------- */
const FS_RESOLVE = `#version 300 es
precision highp float;
precision highp int;
in vec2 vUv;
out vec4 outColor;

uniform sampler2D uCur;
uniform sampler2D uHist;
uniform vec2 uLowRes;
uniform vec2 uOutRes;
uniform vec2 uJitter;
uniform vec3 uCamPos;
uniform vec3 uCamRight;
uniform vec3 uCamUp;
uniform vec3 uCamFwd;
uniform float uTanHalf;
uniform vec3 uPCamPos;
uniform vec3 uPCamRight;
uniform vec3 uPCamUp;
uniform vec3 uPCamFwd;
uniform float uPTanHalf;
uniform float uHistW;

const vec3 LUMA = vec3(0.2126, 0.7152, 0.0722);
vec3 comp(vec3 c) {
  c = min(c, vec3(30.0));
  return c / (1.0 + dot(c, LUMA));
}

void main() {
  vec2 uv = gl_FragCoord.xy / uOutRes;
  vec2 suv = uv - uJitter / uLowRes;
  vec3 cur = comp(texture(uCur, suv).rgb);

  ivec2 base = ivec2(floor(suv * uLowRes));
  ivec2 lim = ivec2(uLowRes) - 1;
  vec3 m1 = vec3(0.0);
  vec3 m2 = vec3(0.0);
  for (int y = -1; y <= 1; y++) {
    for (int x = -1; x <= 1; x++) {
      ivec2 c = clamp(base + ivec2(x, y), ivec2(0), lim);
      vec3 s = comp(texelFetch(uCur, c, 0).rgb);
      m1 += s;
      m2 += s * s;
    }
  }
  vec3 mean = m1 / 9.0;
  vec3 sigma = sqrt(max(m2 / 9.0 - mean * mean, vec3(0.0)));
  vec3 lo = mean - 2.0 * sigma;
  vec3 hi = mean + 2.0 * sigma;

  float t = texelFetch(uCur, clamp(base, ivec2(0), lim), 0).a;
  float asp = uOutRes.x / uOutRes.y;
  vec2 ndc = uv * 2.0 - 1.0;
  vec3 rd = normalize(uCamFwd + uCamRight * (ndc.x * uTanHalf * asp) + uCamUp * (ndc.y * uTanHalf));
  float tt = min(t, 300.0);
  vec3 wp = uCamPos + rd * tt;
  vec3 v = wp - uPCamPos;
  float z = dot(v, uPCamFwd);
  bool valid = uHistW > 0.0 && z > 0.05;
  vec2 puv = vec2(0.5);
  if (valid) {
    puv = vec2(dot(v, uPCamRight) / (z * uPTanHalf * asp), dot(v, uPCamUp) / (z * uPTanHalf)) * 0.5 + 0.5;
    valid = puv.x > 0.0 && puv.x < 1.0 && puv.y > 0.0 && puv.y < 1.0;
  }
  vec4 h = texture(uHist, puv);
  if (valid && t < 900.0) {
    float expectD = length(v);
    if (abs(h.a - expectD) > 0.08 * expectD + 0.15) valid = false;
  }
  vec3 hist = clamp(h.rgb, lo, hi);
  float w = valid ? uHistW : 0.0;
  outColor = vec4(mix(cur, hist, w), t);
}`;

/* ---------- pass 3: tone-map, sharpen, first-person weapon ---------- */
const FS_PRESENT = `#version 300 es
precision highp float;
precision highp int;
in vec2 vUv;
out vec4 outColor;

uniform sampler2D uTex;
uniform vec2 uOutRes;
uniform float uExposure;
uniform float uSharp;
uniform float uTime;
uniform int uGunCount;
uniform vec3 uGunMin[12];
uniform vec3 uGunMax[12];
uniform vec4 uGunCol[12];
uniform vec3 uGunOff;
uniform vec3 uSunView;
uniform float uGunLight;

const vec3 LUMA = vec3(0.2126, 0.7152, 0.0722);

vec3 dec(vec3 c) {
  float l = clamp(dot(c, LUMA), 0.0, 0.995);
  return c / (1.0 - l);
}
vec3 aces(vec3 x) {
  const float a = 2.51;
  const float b = 0.03;
  const float c = 2.43;
  const float d = 0.59;
  const float e = 0.14;
  return clamp((x * (a * x + b)) / (x * (c * x + d) + e), 0.0, 1.0);
}
float hash12(vec2 p) {
  vec3 p3 = fract(vec3(p.xyx) * 0.1031);
  p3 += dot(p3, p3.yzx + 33.33);
  return fract((p3.x + p3.y) * p3.z);
}
bool rayBox(vec3 ro, vec3 rd, vec3 bmin, vec3 bmax, out float t, out vec3 n) {
  t = 0.0;
  n = vec3(0.0, 1.0, 0.0);
  vec3 inv = 1.0 / (rd + vec3(1e-7));
  vec3 t0 = (bmin - ro) * inv;
  vec3 t1 = (bmax - ro) * inv;
  vec3 tmn = min(t0, t1);
  vec3 tmx = max(t0, t1);
  float tn = max(max(tmn.x, tmn.y), tmn.z);
  float tf = min(min(tmx.x, tmx.y), tmx.z);
  if (tn > tf || tf < 0.0 || tn < 0.02) return false;
  t = tn;
  if (tmn.x >= tmn.y && tmn.x >= tmn.z) n = vec3(-sign(rd.x), 0.0, 0.0);
  else if (tmn.y >= tmn.z) n = vec3(0.0, -sign(rd.y), 0.0);
  else n = vec3(0.0, 0.0, -sign(rd.z));
  return true;
}

void main() {
  vec2 uv = gl_FragCoord.xy / uOutRes;
  vec2 px = 1.0 / uOutRes;
  vec3 c = dec(texture(uTex, uv).rgb);
  vec3 nb = dec(texture(uTex, uv + vec2(px.x, 0.0)).rgb)
          + dec(texture(uTex, uv - vec2(px.x, 0.0)).rgb)
          + dec(texture(uTex, uv + vec2(0.0, px.y)).rgb)
          + dec(texture(uTex, uv - vec2(0.0, px.y)).rgb);
  vec3 sharp = max(c + (c - nb * 0.25) * uSharp, vec3(0.0));
  vec3 col = aces(sharp * uExposure);
  vec2 q = uv * 2.0 - 1.0;
  col *= 1.0 - 0.14 * dot(q, q) * 0.5;
  col = pow(col, vec3(1.0 / 2.2));
  col += (hash12(gl_FragCoord.xy + fract(uTime) * 91.7) - 0.5) * (1.5 / 255.0);

  if (uGunCount > 0) {
    float asp = uOutRes.x / uOutRes.y;
    vec2 nd = uv * 2.0 - 1.0;
    vec3 rd = normalize(vec3(nd.x * 0.62 * asp, nd.y * 0.62, 1.0));
    vec3 ro = -uGunOff;
    float bt = 1e9;
    vec3 bn = vec3(0.0);
    vec4 bc = vec4(0.0);
    for (int i = 0; i < 12; i++) {
      if (i >= uGunCount) break;
      float t;
      vec3 n;
      if (rayBox(ro, rd, uGunMin[i], uGunMax[i], t, n) && t < bt) { bt = t; bn = n; bc = uGunCol[i]; }
    }
    if (bt < 1e8) {
      float ndl = max(dot(bn, uSunView), 0.0);
      float hemi = bn.y * 0.5 + 0.5;
      vec3 lit = bc.rgb * (0.10 + 0.16 * uGunLight + 0.22 * hemi * uGunLight + 0.85 * ndl * uGunLight);
      vec3 r = reflect(rd, bn);
      float sp = pow(max(dot(r, uSunView), 0.0), 28.0) * bc.a * uGunLight;
      float fres = pow(1.0 - max(dot(-rd, bn), 0.0), 3.0);
      lit += vec3(1.0, 0.95, 0.85) * sp * 0.9 + vec3(0.4, 0.5, 0.6) * fres * 0.22 * (0.3 + bc.a) * (0.3 + 0.7 * uGunLight);
      col = pow(clamp(lit, 0.0, 1.0), vec3(1.0 / 2.2));
    }
  }
  outColor = vec4(col, 1.0);
}`;

/* ---------- first-person weapon models (view space boxes: min, max, colour, metalness) ---------- */
const GUNS = [
  [ // sidearm
    [0.17,-0.215,0.42, 0.25,-0.145,0.80, 0.22,0.23,0.26,0.8], [0.195,-0.205,0.80, 0.225,-0.175,0.86, 0.10,0.10,0.11,0.9],
    [0.18,-0.235,0.40, 0.24,-0.215,0.72, 0.15,0.15,0.17,0.6], [0.185,-0.345,0.40, 0.235,-0.215,0.50, 0.12,0.12,0.13,0.3],
    [0.20,-0.140,0.42, 0.22,-0.125,0.45, 0.05,0.05,0.05,0.2], [0.205,-0.140,0.76, 0.215,-0.128,0.78, 0.9,0.9,0.9,0.2],
    [0.13,-0.40,0.25, 0.26,-0.28,0.48, 0.10,0.40,0.46,0.0], [0.10,-0.30,0.45, 0.20,-0.24,0.58, 0.85,0.65,0.55,0.0]
  ],
  [ // rifle
    [0.16,-0.24,0.30, 0.26,-0.14,0.85, 0.16,0.17,0.19,0.7], [0.175,-0.26,0.18, 0.245,-0.17,0.31, 0.35,0.20,0.10,0.1],
    [0.205,-0.185,0.85, 0.225,-0.165,1.15, 0.10,0.10,0.11,0.9], [0.18,-0.25,0.55, 0.25,-0.16,0.85, 0.35,0.20,0.10,0.1],
    [0.20,-0.34,0.45, 0.24,-0.22,0.55, 0.08,0.08,0.09,0.4], [0.20,-0.32,0.30, 0.235,-0.22,0.36, 0.10,0.10,0.11,0.3],
    [0.21,-0.135,0.55, 0.23,-0.115,0.62, 0.05,0.05,0.05,0.2], [0.21,-0.15,1.00, 0.22,-0.13,1.02, 0.9,0.9,0.9,0.2],
    [0.14,-0.38,0.22, 0.24,-0.28,0.40, 0.10,0.40,0.46,0.0], [0.05,-0.30,0.55, 0.19,-0.20,0.70, 0.85,0.65,0.55,0.0]
  ],
  [ // sniper
    [0.18,-0.22,0.30, 0.26,-0.14,1.00, 0.12,0.13,0.15,0.7], [0.205,-0.19,1.00, 0.235,-0.165,1.55, 0.08,0.08,0.09,0.9],
    [0.19,-0.26,0.15, 0.25,-0.16,0.30, 0.20,0.22,0.18,0.1], [0.185,-0.115,0.50, 0.255,-0.055,0.88, 0.07,0.08,0.09,0.8],
    [0.20,-0.30,0.55, 0.24,-0.22,0.65, 0.07,0.07,0.08,0.4], [0.20,-0.32,0.30, 0.235,-0.22,0.36, 0.09,0.09,0.10,0.3],
    [0.19,-0.07,0.86, 0.25,-0.05,0.92, 0.1,0.8,0.9,0.5], [0.14,-0.38,0.22, 0.24,-0.28,0.40, 0.10,0.40,0.46,0.0],
    [0.05,-0.30,0.60, 0.19,-0.20,0.75, 0.85,0.65,0.55,0.0]
  ],
  [ // build tool (a held block)
    [0.16,-0.24,0.45, 0.30,-0.10,0.59, 0.8,0.8,0.8,0.0], [0.13,-0.42,0.30, 0.26,-0.28,0.52, 0.10,0.40,0.46,0.0],
    [0.05,-0.30,0.45, 0.17,-0.22,0.58, 0.85,0.65,0.55,0.0]
  ]
];

/* ---------- GL setup ---------- */
function compile(type, src, name) {
  const s = gl.createShader(type); gl.shaderSource(s, src); gl.compileShader(s);
  if (!gl.getShaderParameter(s, gl.COMPILE_STATUS)) {
    const log = gl.getShaderInfoLog(s) || '(no log)';
    const lines = src.split('\n').map((l, i) => (i + 1) + ': ' + l).join('\n');
    throw new Error('Shader compile error in ' + name + ':\n' + log + '\n\n' + lines.slice(0, 6000));
  }
  return s;
}
function makeProgram(fsSrc, name) {
  const p = gl.createProgram();
  gl.attachShader(p, compile(gl.VERTEX_SHADER, VS_SRC, name + ' (vertex)'));
  gl.attachShader(p, compile(gl.FRAGMENT_SHADER, fsSrc, name));
  gl.linkProgram(p);
  if (!gl.getProgramParameter(p, gl.LINK_STATUS)) throw new Error('Link error in ' + name + ':\n' + gl.getProgramInfoLog(p));
  return p;
}
const locs = (p, names) => { const o = {}; for (const n of names) o[n] = gl.getUniformLocation(p, n); return o; };

function makeRT(w, h) {
  const tex = gl.createTexture(); gl.bindTexture(gl.TEXTURE_2D, tex);
  gl.texImage2D(gl.TEXTURE_2D, 0, gl.RGBA16F, w, h, 0, gl.RGBA, gl.HALF_FLOAT, null);
  gl.texParameteri(gl.TEXTURE_2D, gl.TEXTURE_MIN_FILTER, gl.LINEAR); gl.texParameteri(gl.TEXTURE_2D, gl.TEXTURE_MAG_FILTER, gl.LINEAR);
  gl.texParameteri(gl.TEXTURE_2D, gl.TEXTURE_WRAP_S, gl.CLAMP_TO_EDGE); gl.texParameteri(gl.TEXTURE_2D, gl.TEXTURE_WRAP_T, gl.CLAMP_TO_EDGE);
  const fbo = gl.createFramebuffer(); gl.bindFramebuffer(gl.FRAMEBUFFER, fbo);
  gl.framebufferTexture2D(gl.FRAMEBUFFER, gl.COLOR_ATTACHMENT0, gl.TEXTURE_2D, tex, 0);
  if (gl.checkFramebufferStatus(gl.FRAMEBUFFER) !== gl.FRAMEBUFFER_COMPLETE) throw new Error('Float render target is not supported by this GPU/browser.');
  gl.bindFramebuffer(gl.FRAMEBUFFER, null);
  return { tex, fbo, w, h };
}
function freeRT(t) { if (t) { gl.deleteTexture(t.tex); gl.deleteFramebuffer(t.fbo); } }

function allocTargets() {
  const dprCap = Math.min(window.devicePixelRatio || 1, 1.5);
  const w = Math.max(64, Math.floor(innerWidth * dprCap)), h = Math.max(64, Math.floor(innerHeight * dprCap));
  canvas.width = w; canvas.height = h;
  const rw = Math.max(64, Math.floor(w * S.scale)), rh = Math.max(64, Math.floor(h * S.scale));
  if (targets) { freeRT(targets.trace); freeRT(targets.hist[0]); freeRT(targets.hist[1]); }
  targets = { w, h, rw, rh, trace: makeRT(rw, rh), hist: [makeRT(w, h), makeRT(w, h)], cur: 0 };
  R.reset = true;
}

function initGL() {
  gl = canvas.getContext('webgl2', { antialias: false, alpha: false, depth: false, stencil: false, powerPreference: 'high-performance' });
  if (!gl) throw new Error('WebGL2 is not available in this browser.');
  if (!gl.getExtension('EXT_color_buffer_float') && !gl.getExtension('EXT_color_buffer_half_float')) throw new Error('This GPU cannot render to floating-point textures.');
  vao = gl.createVertexArray(); gl.bindVertexArray(vao);

  progTrace = makeProgram(FS_TRACE, 'path tracer');
  progResolve = makeProgram(FS_RESOLVE, 'temporal upscaler');
  progPresent = makeProgram(FS_PRESENT, 'present');
  LT = locs(progTrace, ['uVox','uRes','uCamPos','uCamRight','uCamUp','uCamFwd','uTanHalf','uJitter','uFrame','uSunDir','uSunCol','uSkyZ','uSkyH','uBounces','uReflect','uBots','uBotCount','uFx','uFxCount','uSel','uFlash','uTime']);
  LR = locs(progResolve, ['uCur','uHist','uLowRes','uOutRes','uJitter','uCamPos','uCamRight','uCamUp','uCamFwd','uTanHalf','uPCamPos','uPCamRight','uPCamUp','uPCamFwd','uPTanHalf','uHistW']);
  LP = locs(progPresent, ['uTex','uOutRes','uExposure','uSharp','uTime','uGunCount','uGunMin','uGunMax','uGunCol','uGunOff','uSunView','uGunLight']);

  voxTex = gl.createTexture(); gl.bindTexture(gl.TEXTURE_3D, voxTex);
  gl.pixelStorei(gl.UNPACK_ALIGNMENT, 1);
  gl.texImage3D(gl.TEXTURE_3D, 0, gl.R8UI, NX, NY, NZ, 0, gl.RED_INTEGER, gl.UNSIGNED_BYTE, vox);
  for (const p of [gl.TEXTURE_MIN_FILTER, gl.TEXTURE_MAG_FILTER]) gl.texParameteri(gl.TEXTURE_3D, p, gl.NEAREST);
  for (const p of [gl.TEXTURE_WRAP_S, gl.TEXTURE_WRAP_T, gl.TEXTURE_WRAP_R]) gl.texParameteri(gl.TEXTURE_3D, p, gl.CLAMP_TO_EDGE);

  allocTargets();
  const dbg = gl.getExtension('WEBGL_debug_renderer_info');
  $('gpuinfo').textContent = 'GPU: ' + (dbg ? gl.getParameter(dbg.UNMASKED_RENDERER_WEBGL) : 'unknown');
  R.ok = true;
}

/* ---------- per-frame rendering ---------- */
function halton(i, b) { let f = 1, r = 0; while (i > 0) { f /= b; r += f * (i % b); i = Math.floor(i / b); } return r; }
const botArr = new Float32Array(32), fxArr = new Float32Array(48);
const gMin = new Float32Array(36), gMax = new Float32Array(36), gCol = new Float32Array(48);

function skyParams(el) {
  const t = clamp((el - 5) / 60, 0, 1);
  const mx = (a, b) => a.map((v, i) => v + (b[i] - v) * t);
  const sun = mx([1.0, 0.45, 0.18], [1.0, 0.93, 0.80]).map(v => v * (0.55 + 0.45 * t));
  return { sun, zen: mx([0.10, 0.16, 0.34], [0.22, 0.46, 0.90]), hor: mx([0.95, 0.50, 0.30], [0.78, 0.82, 0.90]), exposure: lerp(1.5, 1.05, t) };
}

function bindTex(unit, target, tex) { gl.activeTexture(gl.TEXTURE0 + unit); gl.bindTexture(target, tex); }

function render() {
  if (!R.ok) return;
  const T = targets, w = T.w, h = T.h;
  const aspect = w / h;
  const B = basis(P.yaw, P.pitch), eye = eyePos();
  const tanV = Math.tan(S.fov * Math.PI / 180 / 2) / aspect / zoomCur;
  const fr = R.frame++;
  const jit = [halton((fr % 16) + 1, 2) - 0.5, halton((fr % 16) + 1, 3) - 0.5];

  // sun / sky
  const el = S.tod * Math.PI / 180, az = 215 * Math.PI / 180;
  R.sunDir = [Math.cos(el) * Math.sin(az), Math.sin(el), Math.cos(el) * Math.cos(az)];
  const sky = skyParams(S.tod);

  // upload voxel edits
  if (edits.length) {
    gl.bindTexture(gl.TEXTURE_3D, voxTex); gl.pixelStorei(gl.UNPACK_ALIGNMENT, 1);
    for (const [i, j, k, id] of edits) gl.texSubImage3D(gl.TEXTURE_3D, 0, i, j, k, 1, 1, 1, gl.RED_INTEGER, gl.UNSIGNED_BYTE, new Uint8Array([id]));
    edits.length = 0;
  }

  // dynamic objects
  botArr.fill(0); let nb = 0;
  for (const b of bots) { if (nb >= 8) break; botArr[nb * 4] = b.x; botArr[nb * 4 + 1] = b.y; botArr[nb * 4 + 2] = b.z; botArr[nb * 4 + 3] = b.alive ? 1 : 0; nb++; }
  fxArr.fill(0); let nf = 0;
  for (const f of fx) { fxArr[nf * 4] = f.x; fxArr[nf * 4 + 1] = f.y; fxArr[nf * 4 + 2] = f.z; fxArr[nf * 4 + 3] = f.r * clamp(f.life / f.max, 0.2, 1); nf++; }
  let sel = [0, 0, 0, 0];
  if (cur === 3 && !P.dead) { const t = targetVoxel(); if (t) sel = [t.i, t.j, t.k, 1]; }
  const fp = [eye[0] + B.f[0] * 0.7 + B.r[0] * 0.2 - B.u[0] * 0.1, eye[1] + B.f[1] * 0.7 + B.r[1] * 0.2 - B.u[1] * 0.1, eye[2] + B.f[2] * 0.7 + B.r[2] * 0.2 - B.u[2] * 0.1];

  gl.bindVertexArray(vao);
  gl.disable(gl.BLEND); gl.disable(gl.DEPTH_TEST);

  // ---- 1. trace ----
  gl.bindFramebuffer(gl.FRAMEBUFFER, T.trace.fbo); gl.viewport(0, 0, T.rw, T.rh);
  gl.useProgram(progTrace);
  bindTex(0, gl.TEXTURE_3D, voxTex); gl.uniform1i(LT.uVox, 0);
  gl.uniform2f(LT.uRes, T.rw, T.rh);
  gl.uniform3fv(LT.uCamPos, eye); gl.uniform3fv(LT.uCamRight, B.r); gl.uniform3fv(LT.uCamUp, B.u); gl.uniform3fv(LT.uCamFwd, B.f);
  gl.uniform1f(LT.uTanHalf, tanV); gl.uniform2f(LT.uJitter, jit[0], jit[1]); gl.uniform1ui(LT.uFrame, fr >>> 0);
  gl.uniform3fv(LT.uSunDir, R.sunDir); gl.uniform3fv(LT.uSunCol, sky.sun); gl.uniform3fv(LT.uSkyZ, sky.zen); gl.uniform3fv(LT.uSkyH, sky.hor);
  gl.uniform1i(LT.uBounces, S.bounces); gl.uniform1i(LT.uReflect, S.reflect ? 1 : 0);
  gl.uniform4fv(LT.uBots, botArr); gl.uniform1i(LT.uBotCount, nb);
  gl.uniform4fv(LT.uFx, fxArr); gl.uniform1i(LT.uFxCount, nf);
  gl.uniform4fv(LT.uSel, sel); gl.uniform4f(LT.uFlash, fp[0], fp[1], fp[2], flashT > 0 ? 28 * (flashT / 0.06) : 0);
  gl.uniform1f(LT.uTime, performance.now() / 1000);
  gl.drawArrays(gl.TRIANGLES, 0, 3);

  // ---- 2. temporal upscale ----
  const hc = T.cur, hp = 1 - T.cur, pv = R.prev || { pos: eye, r: B.r, u: B.u, f: B.f, tan: tanV };
  const moved = Math.abs(pv.pos[0] - eye[0]) + Math.abs(pv.pos[1] - eye[1]) + Math.abs(pv.pos[2] - eye[2]) + Math.abs(pv.f[0] - B.f[0]) + Math.abs(pv.f[1] - B.f[1]) + Math.abs(pv.f[2] - B.f[2]);
  const histW = R.reset ? 0 : (moved > 0.0005 ? 0.86 : 0.94);
  gl.bindFramebuffer(gl.FRAMEBUFFER, T.hist[hc].fbo); gl.viewport(0, 0, w, h);
  gl.useProgram(progResolve);
  bindTex(0, gl.TEXTURE_2D, T.trace.tex); bindTex(1, gl.TEXTURE_2D, T.hist[hp].tex);
  gl.uniform1i(LR.uCur, 0); gl.uniform1i(LR.uHist, 1);
  gl.uniform2f(LR.uLowRes, T.rw, T.rh); gl.uniform2f(LR.uOutRes, w, h); gl.uniform2f(LR.uJitter, jit[0], jit[1]);
  gl.uniform3fv(LR.uCamPos, eye); gl.uniform3fv(LR.uCamRight, B.r); gl.uniform3fv(LR.uCamUp, B.u); gl.uniform3fv(LR.uCamFwd, B.f); gl.uniform1f(LR.uTanHalf, tanV);
  gl.uniform3fv(LR.uPCamPos, pv.pos); gl.uniform3fv(LR.uPCamRight, pv.r); gl.uniform3fv(LR.uPCamUp, pv.u); gl.uniform3fv(LR.uPCamFwd, pv.f); gl.uniform1f(LR.uPTanHalf, pv.tan);
  gl.uniform1f(LR.uHistW, histW);
  gl.drawArrays(gl.TRIANGLES, 0, 3);

  // ---- 3. present ----
  gl.bindFramebuffer(gl.FRAMEBUFFER, null); gl.viewport(0, 0, w, h);
  gl.useProgram(progPresent);
  bindTex(0, gl.TEXTURE_2D, T.hist[hc].tex); gl.uniform1i(LP.uTex, 0);
  gl.uniform2f(LP.uOutRes, w, h); gl.uniform1f(LP.uExposure, sky.exposure); gl.uniform1f(LP.uSharp, S.scale < 0.95 ? 0.45 : 0.25);
  gl.uniform1f(LP.uTime, performance.now() / 1000);
  setGun(B);
  gl.drawArrays(gl.TRIANGLES, 0, 3);

  T.cur = hp; R.reset = false;
  R.prev = { pos: eye.slice(), r: B.r.slice(), u: B.u.slice(), f: B.f.slice(), tan: tanV };
}

function setGun(B) {
  const hide = P.dead || (cur === 2 && zoomLevel > 0 && zoomCur > 1.8);
  const boxes = GUNS[cur];
  let n = hide ? 0 : boxes.length;
  for (let i = 0; i < n; i++) {
    const b = boxes[i];
    gMin[i * 3] = b[0]; gMin[i * 3 + 1] = b[1]; gMin[i * 3 + 2] = b[2];
    gMax[i * 3] = b[3]; gMax[i * 3 + 1] = b[4]; gMax[i * 3 + 2] = b[5];
    gCol[i * 4] = b[6]; gCol[i * 4 + 1] = b[7]; gCol[i * 4 + 2] = b[8]; gCol[i * 4 + 3] = b[9];
  }
  if (cur === 3 && n) {   // tint the held block with the selected material
    const c = MATS[blockIdx].css;
    gCol[0] = parseInt(c.slice(1, 3), 16) / 255; gCol[1] = parseInt(c.slice(3, 5), 16) / 255; gCol[2] = parseInt(c.slice(5, 7), 16) / 255;
  }
  const sp = Math.hypot(P.vx, P.vz), bob = P.onGround ? Math.min(sp / RUN, 1) : 0.2;
  const tt = performance.now() / 1000;
  const ads = (cur < 3 && zoomLevel > 0 && W[cur].zoom.length) ? Math.min(1, (zoomCur - 1) / Math.max(0.2, W[cur].zoom[zoomLevel - 1] - 1)) : 0;
  let oy = Math.sin(tt * 10) * 0.006 * bob - 0.003, ox = Math.cos(tt * 5) * 0.005 * bob, oz = -kickBack;
  if (reloading) oy -= 0.22 * Math.sin(clamp(1 - reloadT / W[cur].reload, 0, 1) * Math.PI);
  if (switchT > 0) oy -= 0.35 * (switchT / 0.3);
  ox -= 0.19 * ads; oy += 0.035 * ads;
  gl.uniform1i(LP.uGunCount, n);
  gl.uniform3fv(LP.uGunMin, gMin); gl.uniform3fv(LP.uGunMax, gMax); gl.uniform4fv(LP.uGunCol, gCol);
  gl.uniform3f(LP.uGunOff, ox, oy, oz);
  const sd = R.sunDir;
  gl.uniform3f(LP.uSunView, sd[0] * B.r[0] + sd[1] * B.r[1] + sd[2] * B.r[2], sd[0] * B.u[0] + sd[1] * B.u[1] + sd[2] * B.u[2], sd[0] * B.f[0] + sd[1] * B.f[1] + sd[2] * B.f[2]);
  gl.uniform1f(LP.uGunLight, gunLight);
}

/* ===================================================================
   HUD refresh, input, menu
   =================================================================== */
const hudCache = {};
function setText(el, key, v) { if (hudCache[key] !== v) { hudCache[key] = v; el.textContent = v; } }
function updateHud(fps, ms) {
  setText(hud.hp, 'hp', String(Math.ceil(P.hp)));
  setText(hud.arv, 'ar', String(Math.ceil(P.armor)));
  setText(hud.score, 'sc', 'KILLS ' + kills);
  hud.hp.style.color = P.hp < 35 ? '#ff4655' : '';
  if (cur < 3) {
    setText(hud.wname, 'wn', W[cur].name + (reloading ? ' · RELOADING' : ''));
    const a = ammo[cur];
    const html = a.mag + ' <small>/ ' + a.res + '</small>';
    if (hudCache.ammo !== html) { hudCache.ammo = html; hud.ammo.innerHTML = html; }
  } else {
    setText(hud.wname, 'wn', 'BUILD TOOL'); const html = '∞'; if (hudCache.ammo !== html) { hudCache.ammo = html; hud.ammo.innerHTML = html; }
  }
  [...hud.slots.children].forEach((d, i) => d.classList.toggle('on', i === cur));
  hud.blockSel.classList.toggle('hidden', cur !== 3);
  if (cur === 3) { hud.swatch.style.background = MATS[blockIdx].css; setText(hud.bname, 'bn', MATS[blockIdx].name); }
  hud.abQ.lastElementChild.style.height = (P.cdQ / 8 * 100) + '%';
  hud.abE.lastElementChild.style.height = (P.cdE / 6 * 100) + '%';

  const w = cur < 3 ? W[cur] : null;
  const gap = 3 + (w ? spreadDeg(w) * 7 : 0) + kickBack * 40;
  hud.xh.style.setProperty('--gap', gap.toFixed(1) + 'px');
  hud.xh.style.display = (cur === 2 && zoomLevel > 0) ? 'none' : '';
  hud.scope.style.display = (cur === 2 && zoomLevel > 0 && zoomCur > 1.8) ? 'block' : 'none';
  if (showStats && targets) {
    hud.stats.textContent = Math.round(fps) + ' fps  ' + ms.toFixed(1) + ' ms\ntrace ' + targets.rw + '×' + targets.rh + ' → ' + targets.w + '×' + targets.h +
      ' (' + Math.round(S.scale * 100) + '%)\n' + S.bounces + ' bounce' + (S.bounces === 1 ? '' : 's') + (S.reflect ? ' + reflections' : '') + '   [V] hide';
  } else hud.stats.textContent = '';
}

const keys = new Set();
let locked = false;
addEventListener('keydown', e => {
  if (!locked) return;
  if (['Space', 'Tab', 'ArrowUp', 'ArrowDown'].includes(e.code)) e.preventDefault();
  if (e.repeat) { keys.add(e.code); return; }
  keys.add(e.code);
  switch (e.code) {
    case 'Space': P.jumpBuf = 0.12; break;
    case 'Digit1': equip(0); break; case 'Digit2': equip(1); break; case 'Digit3': equip(2); break; case 'Digit4': equip(3); break;
    case 'KeyR': startReload(); break;
    case 'KeyQ': useQ(); break; case 'KeyE': useE(); break;
    case 'KeyT': spawnBot(); break;
    case 'KeyY': bots.length = 0; toast('BOTS CLEARED'); break;
    case 'KeyU': botsAggressive = !botsAggressive; toast(botsAggressive ? 'BOTS: HOSTILE' : 'BOTS: PASSIVE'); break;
    case 'KeyF': P.fly = !P.fly; P.vy = 0; toast(P.fly ? 'FLY MODE' : 'WALK MODE'); break;
    case 'KeyV': showStats = !showStats; break;
  }
});
addEventListener('keyup', e => keys.delete(e.code));
addEventListener('blur', () => { keys.clear(); mouseL = mouseR = false; });
addEventListener('mousemove', e => {
  if (!locked) return;
  const k = 0.07 * S.sens * Math.PI / 180 / zoomCur;
  P.yaw += e.movementX * k; P.pitch = clamp(P.pitch - e.movementY * k, -1.5, 1.5);
});
addEventListener('mousedown', e => {
  if (!locked) return;
  if (e.button === 0) { mouseL = true; fireEdge = true; if (cur === 3) buildCd = 0; }
  if (e.button === 2) {
    mouseR = true;
    if (cur < 3 && !reloading) {
      const z = W[cur].zoom;
      if (z.length) zoomLevel = (cur === 2) ? (zoomLevel + 1) % (z.length + 1) : 1;   // sniper cycles 2.5x/5x/off, rifle is hold-to-aim
    }
    if (cur === 3) buildCd = 0;
  }
});
addEventListener('mouseup', e => {
  if (e.button === 0) mouseL = false;
  if (e.button === 2) { mouseR = false; if (cur === 1) zoomLevel = 0; }   // rifle zoom is hold-to-aim
});
addEventListener('contextmenu', e => e.preventDefault());
addEventListener('wheel', e => {
  if (!locked) return;
  const dir = e.deltaY > 0 ? 1 : -1;
  if (cur === 3) { blockIdx = (blockIdx + dir + MATS.length) % MATS.length; }
  else equip((cur + dir + 3) % 3);
}, { passive: true });

document.addEventListener('pointerlockchange', () => {
  locked = document.pointerLockElement === canvas;
  $('menu').classList.toggle('hidden', locked); $('hud').classList.toggle('hidden', !locked);
  if (!locked) { keys.clear(); mouseL = mouseR = false; $('play').textContent = 'RESUME'; }
});
document.addEventListener('pointerlockerror', () => showError('The browser refused pointer lock. Click the page once and try again (some embedded previews block it).'));
$('play').addEventListener('click', () => { audio(); try { canvas.requestPointerLock(); } catch (e) { showError('Pointer lock failed: ' + e.message); } });

/* ---------- settings UI ---------- */
function syncUi() {
  $('preset').value = S.preset; $('scale').value = Math.round(S.scale * 100); $('scaleV').textContent = Math.round(S.scale * 100) + '%';
  $('bounces').value = S.bounces; $('reflect').checked = S.reflect; $('auto').checked = S.auto;
  $('tod').value = S.tod; $('todV').textContent = S.tod + '°'; $('fov').value = S.fov; $('fovV').textContent = S.fov + '°';
  $('sens').value = S.sens; $('sensV').textContent = S.sens.toFixed(2);
}
function rescale() { if (R.ok) { try { allocTargets(); } catch (e) { showError(e.message); } } }
$('preset').onchange = e => { S.preset = e.target.value; Object.assign(S, PRESETS[S.preset]); syncUi(); saveSettings(); rescale(); };
$('scale').oninput = e => { S.scale = e.target.value / 100; S.preset = 'custom'; $('scaleV').textContent = e.target.value + '%'; };
$('scale').onchange = () => { saveSettings(); rescale(); };
$('bounces').onchange = e => { S.bounces = +e.target.value; S.preset = 'custom'; saveSettings(); R.reset = true; };
$('reflect').onchange = e => { S.reflect = e.target.checked; S.preset = 'custom'; saveSettings(); R.reset = true; };
$('auto').onchange = e => { S.auto = e.target.checked; saveSettings(); };
$('tod').oninput = e => { S.tod = +e.target.value; $('todV').textContent = S.tod + '°'; R.reset = true; saveSettings(); };
$('fov').oninput = e => { S.fov = +e.target.value; $('fovV').textContent = S.fov + '°'; saveSettings(); };
$('sens').oninput = e => { S.sens = +e.target.value; $('sensV').textContent = S.sens.toFixed(2); saveSettings(); };
addEventListener('resize', () => rescale());
syncUi();

/* ===================================================================
   Main loop
   =================================================================== */
for (let i = 0; i < 3; i++) spawnBot();
let last = performance.now(), accMs = 0, accN = 0, fpsShow = 60, msShow = 16, lowCount = 0, hudT = 0;
function frame(now) {
  requestAnimationFrame(frame);
  const raw = Math.min(0.1, (now - last) / 1000); last = now;
  const dt = Math.min(raw, 1 / 20);
  if (locked) updateAll(dt);
  try { render(); } catch (e) { R.ok = false; showError('Render error: ' + (e && e.message)); }

  accMs += raw * 1000; accN++;
  if (accMs > 1000) {
    msShow = accMs / accN; fpsShow = 1000 / msShow;
    if (locked && S.auto && fpsShow < 38 && S.scale > 0.3) {
      if (++lowCount >= 2) { S.scale = Math.max(0.3, Math.round((S.scale - 0.05) * 100) / 100); S.preset = 'custom'; lowCount = 0; syncUi(); rescale(); toast('Auto resolution ' + Math.round(S.scale * 100) + '%'); }
    } else lowCount = 0;
    accMs = 0; accN = 0;
  }
  hudT += raw;
  if (locked && hudT > 0.05) { hudT = 0; updateHud(fpsShow, msShow); }
}

try { initGL(); } catch (e) { showError(e.message); }
requestAnimationFrame(frame);

})();
</script>
</body>
</html>
