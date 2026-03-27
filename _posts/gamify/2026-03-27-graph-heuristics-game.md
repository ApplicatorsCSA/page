---
toc: true
layout: post
title: Graph Heuristics Academy (Pathfinder Game)
description: Interactive Greedy Best-First, Dijkstra, and A* on a small weighted graph — NPC guides, step replay, and synth “AI” audio. Tied to the Graph Heuristics lesson.
courses: { csa: {week: 25} }
categories: [Collaboration]
type: tangibles
permalink: /gamify/fortuneFinders/graph-heuristics
---

<link rel="preconnect" href="https://fonts.googleapis.com">
<link rel="preconnect" href="https://fonts.gstatic.com" crossorigin>
<link href="https://fonts.googleapis.com/css2?family=JetBrains+Mono:wght@500;700&family=Space+Grotesk:wght@500;600;700&display=swap" rel="stylesheet">

<div id="graphQuestApp">
  <header class="gq-header">
    <div class="gq-title-block">
      <h1 class="gq-title">Graph Heuristics Academy</h1>
      <p class="gq-sub">Learn <strong>h(n)</strong>, <strong>g(n)</strong>, and <strong>f(n)=g+h</strong> by watching the frontier move — with NPC coaches and synth audio cues.</p>
      <p class="gq-note">Based on the lesson <a href="{{ site.baseurl }}/graph">Graph Heuristics</a> (<code>_notebooks/CSA/data_structures/2025-05-22-graph-heuristics.ipynb</code>).</p>
    </div>
    <div class="gq-header-actions">
      <button type="button" class="gq-btn gq-btn-ghost" id="gqMuteBtn" title="Mute / unmute synth audio">🔊 Sound on</button>
      <button type="button" class="gq-btn gq-btn-primary" id="gqTutorialBtn">How to play</button>
    </div>
  </header>

  <!-- NPC strip -->
  <section class="gq-npcs" aria-label="NPC guides">
    <article class="gq-npc gq-npc-active" data-mode="greedy" tabindex="0">
      <div class="gq-npc-avatar" style="--accent:#f59e0b">G</div>
      <div class="gq-npc-text">
        <div class="gq-npc-name">Gwen the Greedy</div>
        <div class="gq-npc-role">Greedy Best-First</div>
        <p class="gq-npc-blurb">“I only trust <strong>h(n)</strong> — whichever node <em>looks</em> closest to the goal!”</p>
      </div>
    </article>
    <article class="gq-npc" data-mode="dijkstra" tabindex="0">
      <div class="gq-npc-avatar" style="--accent:#38bdf8">D</div>
      <div class="gq-npc-text">
        <div class="gq-npc-name">Dee Uniform</div>
        <div class="gq-npc-role">Dijkstra / UCS</div>
        <p class="gq-npc-blurb">“I ignore guesses. I expand the smallest known <strong>g(n)</strong> — true cost from start.”</p>
      </div>
    </article>
    <article class="gq-npc" data-mode="astar" tabindex="0">
      <div class="gq-npc-avatar" style="--accent:#a78bfa">A</div>
      <div class="gq-npc-text">
        <div class="gq-npc-name">Aria ★</div>
        <div class="gq-npc-role">A*</div>
        <p class="gq-npc-blurb">“Best of both: <strong>f(n)=g(n)+h(n)</strong>. Admissible <strong>h</strong> keeps us honest.”</p>
      </div>
    </article>
  </section>

  <div class="gq-layout">
    <div class="gq-main">
      <div class="gq-canvas-wrap">
        <canvas id="gqCanvas" width="720" height="420" aria-label="Graph visualization"></canvas>
        <div class="gq-legend">
          <span><i class="gq-dot" style="background:#22c55e"></i>Start</span>
          <span><i class="gq-dot" style="background:#ef4444"></i>Goal</span>
          <span><i class="gq-dot" style="background:#38bdf8"></i>Frontier</span>
          <span><i class="gq-dot" style="background:#64748b"></i>Visited</span>
          <span><i class="gq-line"></i>Path</span>
        </div>
      </div>

      <div class="gq-controls">
        <label class="gq-field">
          <span>Algorithm</span>
          <select id="gqAlgo" class="gq-select">
            <option value="greedy">Greedy Best-First (order by h)</option>
            <option value="dijkstra">Dijkstra (order by g)</option>
            <option value="astar" selected>A* (order by f = g + h)</option>
          </select>
        </label>
        <div class="gq-btn-row">
          <button type="button" class="gq-btn gq-btn-primary" id="gqStepBtn">Step</button>
          <button type="button" class="gq-btn" id="gqAutoBtn">Auto play</button>
          <button type="button" class="gq-btn" id="gqResetBtn">Reset</button>
        </div>
      </div>
      <div id="gqStatus" class="gq-status" role="status">Ready — pick an algorithm and press Step.</div>
      <div class="gq-metrics" id="gqMetrics"></div>
    </div>

    <aside class="gq-side" id="gqDialogue" aria-live="polite">
      <div class="gq-side-header">
        <span class="gq-badge" id="gqSpeakerBadge">Aria ★</span>
        <span class="gq-badge gq-badge-soft">NPC dialogue</span>
      </div>
      <div class="gq-dialogue" id="gqDialogueText">
        Welcome to the Academy. I’m <strong>Aria</strong>. Try <strong>Step</strong> to expand one node at a time, or <strong>Auto play</strong> to finish. Sound uses Web Audio (synth), not recorded files — click anywhere once if your browser blocks audio.
      </div>
    </aside>
  </div>

  <!-- Lightweight tutorial panel -->
  <div id="gqTutorial" class="gq-tutorial" aria-hidden="true">
    <div class="gq-tutorial-panel" role="dialog" aria-labelledby="gqTutorialTitle">
      <div class="gq-tutorial-top">
        <h2 id="gqTutorialTitle">How to play</h2>
        <button type="button" class="gq-btn gq-btn-ghost" id="gqTutorialClose" aria-label="Close">×</button>
      </div>
      <ol class="gq-tutorial-list">
        <li>Choose <strong>Greedy</strong>, <strong>Dijkstra</strong>, or <strong>A*</strong>. Each NPC explains the rule they follow.</li>
        <li>Press <strong>Step</strong> to expand the next node from the priority queue (open set).</li>
        <li>Watch colors: <strong>frontier</strong> (open), <strong>visited</strong> (closed), and the <strong>path</strong> when the goal is reached.</li>
        <li>Audio: soft blips for expand, a brighter ping for dequeue, a chord when the goal is found — all generated in-browser (Web Audio API).</li>
        <li>This graph matches the lesson ideas: <strong>g</strong> is real edge cost; <strong>h</strong> is an estimate to goal (here, admissible for teaching).</li>
      </ol>
    </div>
  </div>
</div>

<style>
  #graphQuestApp {
    --bg: #0b1220;
    --panel: #111a2e;
    --border: #2a3f5f;
    --text: #e5e7eb;
    --muted: #94a3b8;
    --accent: #38bdf8;
    font-family: "Space Grotesk", system-ui, sans-serif;
    color: var(--text);
    background: radial-gradient(1200px 600px at 20% 0%, #162042 0%, var(--bg) 55%);
    border-radius: 16px;
    padding: 24px;
    margin: 0 auto;
    max-width: 1180px;
  }
  #graphQuestApp * { box-sizing: border-box; }
  .gq-header { display: flex; flex-wrap: wrap; gap: 16px; justify-content: space-between; align-items: flex-start; margin-bottom: 20px; }
  .gq-title { margin: 0 0 8px; font-size: 1.75rem; letter-spacing: -0.02em; }
  .gq-sub { margin: 0; color: var(--muted); max-width: 720px; line-height: 1.55; }
  .gq-note { margin: 10px 0 0; font-size: 0.85rem; color: var(--muted); }
  .gq-note a { color: var(--accent); }
  .gq-header-actions { display: flex; gap: 10px; flex-wrap: wrap; }
  .gq-btn {
    border: 1px solid var(--border);
    background: var(--panel);
    color: var(--text);
    padding: 10px 16px;
    border-radius: 10px;
    cursor: pointer;
    font-weight: 600;
    font-size: 0.95rem;
  }
  .gq-btn:hover { border-color: rgba(56,189,248,0.5); }
  .gq-btn-primary {
    background: linear-gradient(135deg, #38bdf8, #2563eb);
    border: none;
    color: #fff;
  }
  .gq-btn-ghost { background: transparent; }

  .gq-npcs {
    display: grid;
    grid-template-columns: repeat(3, 1fr);
    gap: 12px;
    margin-bottom: 20px;
  }
  @media (max-width: 900px) { .gq-npcs { grid-template-columns: 1fr; } }
  .gq-npc {
    display: flex;
    gap: 12px;
    padding: 14px;
    border-radius: 14px;
    border: 1px solid var(--border);
    background: rgba(17, 26, 46, 0.85);
    cursor: pointer;
    transition: border-color 0.15s, box-shadow 0.15s;
  }
  .gq-npc:hover, .gq-npc:focus { outline: none; border-color: rgba(56,189,248,0.45); box-shadow: 0 0 0 3px rgba(56,189,248,0.12); }
  .gq-npc-active {
    border-color: rgba(167,139,250,0.7);
    box-shadow: 0 0 0 1px rgba(167,139,250,0.35);
  }
  .gq-npc-avatar {
    width: 48px; height: 48px; border-radius: 12px;
    display: flex; align-items: center; justify-content: center;
    font-weight: 800; font-size: 1.1rem;
    background: linear-gradient(145deg, rgba(255,255,255,0.08), rgba(0,0,0,0.2));
    border: 2px solid var(--accent, #64748b);
    color: #fff;
  }
  .gq-npc-name { font-weight: 700; }
  .gq-npc-role { font-size: 0.8rem; color: var(--muted); margin-bottom: 4px; }
  .gq-npc-blurb { margin: 0; font-size: 0.88rem; color: var(--muted); line-height: 1.45; }

  .gq-layout { display: grid; grid-template-columns: 1fr 300px; gap: 16px; align-items: start; }
  @media (max-width: 900px) { .gq-layout { grid-template-columns: 1fr; } }

  .gq-canvas-wrap {
    border: 1px solid var(--border);
    border-radius: 14px;
    overflow: hidden;
    background: linear-gradient(180deg, #0d1528 0%, #0a1020 100%);
  }
  #gqCanvas { display: block; width: 100%; height: auto; }
  .gq-legend {
    display: flex; flex-wrap: wrap; gap: 12px 18px;
    padding: 10px 14px; font-size: 0.8rem; color: var(--muted);
    border-top: 1px solid var(--border);
    background: rgba(0,0,0,0.2);
  }
  .gq-dot { display: inline-block; width: 10px; height: 10px; border-radius: 50%; margin-right: 6px; vertical-align: middle; }
  .gq-line { display: inline-block; width: 18px; height: 3px; background: #fbbf24; margin-right: 6px; vertical-align: middle; border-radius: 2px; }

  .gq-controls { margin-top: 14px; display: flex; flex-wrap: wrap; gap: 12px; align-items: flex-end; }
  .gq-field { display: flex; flex-direction: column; gap: 6px; font-size: 0.8rem; color: var(--muted); }
  .gq-select {
    min-width: 260px; padding: 10px 12px; border-radius: 10px;
    border: 1px solid var(--border); background: var(--panel); color: var(--text);
    font-family: inherit; font-size: 0.95rem;
  }
  .gq-btn-row { display: flex; gap: 8px; flex-wrap: wrap; }

  .gq-status { margin-top: 12px; padding: 12px 14px; border-radius: 10px; border: 1px solid var(--border); background: rgba(17,26,46,0.6); font-size: 0.92rem; }
  .gq-metrics { margin-top: 10px; font-family: "JetBrains Mono", ui-monospace, monospace; font-size: 0.78rem; color: var(--muted); line-height: 1.5; }

  .gq-side {
    border: 1px solid var(--border);
    border-radius: 14px;
    padding: 14px;
    background: rgba(17, 26, 46, 0.75);
    min-height: 200px;
  }
  .gq-side-header { display: flex; flex-wrap: wrap; gap: 8px; align-items: center; margin-bottom: 10px; }
  .gq-badge { font-size: 0.75rem; font-weight: 700; padding: 4px 10px; border-radius: 999px; background: rgba(167,139,250,0.2); color: #e9d5ff; border: 1px solid rgba(167,139,250,0.35); }
  .gq-badge-soft { background: rgba(56,189,248,0.12); color: #bae6fd; border-color: rgba(56,189,248,0.3); }
  .gq-dialogue { font-size: 0.92rem; line-height: 1.55; color: var(--text); }
  .gq-dialogue strong { color: #fff; }

  .gq-tutorial {
    position: fixed; inset: 0; background: rgba(0,0,0,0.5); display: none; align-items: center; justify-content: center; z-index: 10000; padding: 20px;
  }
  .gq-tutorial.is-open { display: flex; }
  .gq-tutorial-panel {
    max-width: 520px; width: 100%; background: var(--panel); border: 1px solid var(--border); border-radius: 16px; padding: 18px 20px;
    box-shadow: 0 20px 50px rgba(0,0,0,0.45);
  }
  .gq-tutorial-top { display: flex; justify-content: space-between; align-items: center; gap: 12px; margin-bottom: 12px; }
  .gq-tutorial-top h2 { margin: 0; font-size: 1.2rem; }
  .gq-tutorial-list { margin: 0; padding-left: 18px; color: var(--muted); line-height: 1.6; font-size: 0.95rem; }
  .gq-tutorial-list li { margin: 8px 0; }
</style>

<script>
(function () {
  const GRAPH = {
    nodes: [
      { id: "S", x: 80, y: 220, h: 7 },
      { id: "A", x: 260, y: 120, h: 5 },
      { id: "B", x: 260, y: 320, h: 4 },
      { id: "C", x: 460, y: 220, h: 2 },
      { id: "G", x: 640, y: 220, h: 0 }
    ],
    edges: [
      { from: "S", to: "A", w: 4 },
      { from: "S", to: "B", w: 3 },
      { from: "A", to: "C", w: 2 },
      { from: "B", to: "C", w: 2 },
      { from: "C", to: "G", w: 3 }
    ],
    start: "S",
    goal: "G"
  };

  const NPC = {
    greedy: { name: "Gwen the Greedy", badge: "Gwen · Greedy", tip: "Greedy expands the node with the smallest <strong>h(n)</strong> — it can be fast but may not find the cheapest path." },
    dijkstra: { name: "Dee Uniform", badge: "Dee · Dijkstra", tip: "Dijkstra always expands the node with the smallest <strong>g(n)</strong> — guaranteed shortest path when edge weights are non‑negative." },
    astar: { name: "Aria ★", badge: "Aria · A*", tip: "A* uses <strong>f(n)=g(n)+h(n)</strong>. With an admissible heuristic, A* stays optimal while exploring fewer nodes than Dijkstra." }
  };

  /** Web Audio “AI synth” cues (no external mp3 files) */
  const AudioAI = (() => {
    let ctx = null;
    let muted = false;

    function ensureCtx() {
      if (!ctx) ctx = new (window.AudioContext || window.webkitAudioContext)();
      return ctx;
    }

    function playTone(freq, duration, type, gain, when) {
      if (muted) return;
      const c = ensureCtx();
      const t0 = when ?? c.currentTime;
      const osc = c.createOscillator();
      const g = c.createGain();
      osc.type = type;
      osc.frequency.setValueAtTime(freq, t0);
      g.gain.setValueAtTime(0.0001, t0);
      g.gain.exponentialRampToValueAtTime(gain, t0 + 0.02);
      g.gain.exponentialRampToValueAtTime(0.0001, t0 + duration);
      osc.connect(g);
      g.connect(c.destination);
      osc.start(t0);
      osc.stop(t0 + duration + 0.02);
    }

    function chord(freqs, duration, gain) {
      if (muted) return;
      const c = ensureCtx();
      const t0 = c.currentTime;
      freqs.forEach((f, i) => playTone(f, duration, "sine", gain * (0.55 - i * 0.08), t0 + i * 0.04));
    }

    return {
      unlock: async () => { const c = ensureCtx(); if (c.state === "suspended") await c.resume(); },
      setMuted: (v) => { muted = !!v; },
      isMuted: () => muted,
      npcBlip: () => playTone(880, 0.06, "square", 0.06),
      expand: () => playTone(420, 0.08, "triangle", 0.07),
      dequeue: () => playTone(660, 0.09, "sine", 0.08),
      goal: () => chord([523.25, 659.25, 783.99], 0.35, 0.12),
      wrong: () => playTone(120, 0.15, "sawtooth", 0.05)
    };
  })();

  function neighborsOf(id) {
    const out = [];
    for (const e of GRAPH.edges) {
      if (e.from === id) out.push({ to: e.to, w: e.w });
      if (e.to === id) out.push({ to: e.from, w: e.w });
    }
    return out;
  }

  function makeEngine(mode) {
    const start = GRAPH.start;
    const goal = GRAPH.goal;
    const h = Object.fromEntries(GRAPH.nodes.map(n => [n.id, n.h]));
    const bestG = {};
    const parent = {};
    const visited = new Set();
    let open = [];

    function pushOpen(nodeId, g) {
      const hh = h[nodeId] ?? 0;
      let priority;
      let tie;
      if (mode === "greedy") {
        priority = hh;
        tie = g;
      } else if (mode === "dijkstra") {
        priority = g;
        tie = hh;
      } else {
        priority = g + hh;
        tie = g;
      }
      open.push({ id: nodeId, g, f: g + hh, h: hh, priority, tie });
      open.sort((a, b) => {
        if (a.priority !== b.priority) return a.priority - b.priority;
        return a.tie - b.tie;
      });
    }

    bestG[start] = 0;
    parent[start] = null;
    pushOpen(start, 0);

    function step() {
      if (open.length === 0) return { done: true, reason: "empty", current: null, visited: new Set(visited), parent: { ...parent } };
      const cur = open.shift();
      if (visited.has(cur.id)) return step();
      visited.add(cur.id);
      if (cur.id === goal) {
        return { done: true, reason: "goal", current: cur.id, visited: new Set(visited), parent: { ...parent } };
      }
      for (const nb of neighborsOf(cur.id)) {
        if (visited.has(nb.to)) continue;
        const ng = cur.g + nb.w;
        if (bestG[nb.to] === undefined || ng < bestG[nb.to]) {
          bestG[nb.to] = ng;
          parent[nb.to] = cur.id;
          pushOpen(nb.to, ng);
        }
      }
      return { done: false, reason: "expand", current: cur.id, visited: new Set(visited), parent: { ...parent }, frontier: open.map(o => o.id) };
    }

    function snapshot() {
      return { open: open.map(o => ({ ...o })), visited: new Set(visited), parent: { ...parent }, bestG: { ...bestG } };
    }

    return { step, snapshot };
  }

  let engine = null;
  let mode = "astar";
  let path = [];
  let autoTimer = null;

  const canvas = document.getElementById("gqCanvas");
  const ctx2 = canvas.getContext("2d");
  const statusEl = document.getElementById("gqStatus");
  const metricsEl = document.getElementById("gqMetrics");
  const dialogueEl = document.getElementById("gqDialogueText");
  const speakerBadge = document.getElementById("gqSpeakerBadge");
  const algoSelect = document.getElementById("gqAlgo");
  const npcEls = document.querySelectorAll(".gq-npc");

  function nodePos(id) { return GRAPH.nodes.find(n => n.id === id); }

  function rebuildPath(parentMap) {
    const goal = GRAPH.goal;
    if (parentMap[goal] === undefined && goal !== GRAPH.start) return [];
    const out = [];
    let x = goal;
    while (x) { out.push(x); x = parentMap[x]; }
    return out.reverse();
  }

  function draw(state) {
    const w = canvas.width, h = canvas.height;
    ctx2.clearRect(0, 0, w, h);
    ctx2.fillStyle = "#0a1020";
    ctx2.fillRect(0, 0, w, h);

    const pos = new Map(GRAPH.nodes.map(n => [n.id, n]));

    ctx2.strokeStyle = "rgba(148,163,184,0.35)";
    ctx2.lineWidth = 2;
    for (const e of GRAPH.edges) {
      const a = pos.get(e.from), b = pos.get(e.to);
      ctx2.beginPath();
      ctx2.moveTo(a.x, a.y);
      ctx2.lineTo(b.x, b.y);
      ctx2.stroke();
      const mx = (a.x + b.x) / 2, my = (a.y + b.y) / 2;
      ctx2.fillStyle = "rgba(148,163,184,0.9)";
      ctx2.font = "12px JetBrains Mono, monospace";
      ctx2.fillText(String(e.w), mx + 4, my - 4);
    }

    const visited = state.visited || new Set();
    const frontier = new Set(state.frontier || []);
    const pathSet = new Set(state.path || []);

    for (const n of GRAPH.nodes) {
      const isStart = n.id === GRAPH.start;
      const isGoal = n.id === GRAPH.goal;
      let fill = "#1e293b";
      if (pathSet.has(n.id)) fill = "#fbbf24";
      else if (visited.has(n.id)) fill = "#64748b";
      else if (frontier.has(n.id)) fill = "#38bdf8";
      if (isStart) fill = "#22c55e";
      if (isGoal) fill = "#ef4444";

      ctx2.beginPath();
      ctx2.arc(n.x, n.y, 22, 0, Math.PI * 2);
      ctx2.fillStyle = fill;
      ctx2.fill();
      ctx2.strokeStyle = "rgba(255,255,255,0.25)";
      ctx2.lineWidth = 2;
      ctx2.stroke();

      ctx2.fillStyle = "#0f172a";
      ctx2.font = "700 14px Space Grotesk, sans-serif";
      ctx2.textAlign = "center";
      ctx2.textBaseline = "middle";
      ctx2.fillText(n.id, n.x, n.y);

      ctx2.font = "11px JetBrains Mono, monospace";
      ctx2.fillStyle = "rgba(226,232,240,0.85)";
      ctx2.fillText("h=" + n.h, n.x, n.y + 36);
    }

    ctx2.fillStyle = "rgba(148,163,184,0.9)";
    ctx2.font = "12px Space Grotesk, sans-serif";
    ctx2.textAlign = "left";
    ctx2.fillText("Start = " + GRAPH.start + "   Goal = " + GRAPH.goal, 16, 24);
  }

  function setDialogue(html, badgeText) {
    dialogueEl.innerHTML = html;
    if (badgeText) speakerBadge.textContent = badgeText;
  }

  function syncNpcUI() {
    npcEls.forEach(el => el.classList.toggle("gq-npc-active", el.getAttribute("data-mode") === mode));
    const info = NPC[mode];
    if (info) setDialogue(info.tip, info.badge);
  }

  function setStatus(msg) { statusEl.textContent = msg; }

  function updateMetrics(snap) {
    if (!snap) { metricsEl.textContent = ""; return; }
    const open = snap.open || [];
    const line = open.slice(0, 6).map(o =>
      o.id + ": g=" + (o.g?.toFixed?.(1) ?? o.g) + " h=" + (o.h?.toFixed?.(1) ?? o.h) + " f=" + (o.f?.toFixed?.(1) ?? o.f)
    ).join("  |  ");
    metricsEl.textContent = "Open (priority queue, first 6): " + (line || "—");
  }

  function reset() {
    if (autoTimer) { clearInterval(autoTimer); autoTimer = null; }
    mode = algoSelect.value;
    engine = makeEngine(mode);
    path = [];
    syncNpcUI();
    const snap = engine.snapshot();
    draw({ visited: snap.visited, frontier: snap.open.map(o => o.id), path: [] });
    setStatus("Ready — " + NPC[mode].name + " is coaching. Press Step.");
    updateMetrics(snap);
  }

  function stepOnce() {
    if (!engine) reset();
    const r = engine.step();
    AudioAI.expand();
    const snap = engine.snapshot();
    draw({ visited: r.visited, frontier: r.frontier || snap.open.map(o => o.id), path: [] });
    updateMetrics(snap);

    if (r.reason === "expand") {
      AudioAI.dequeue();
      setStatus("Expanded node " + r.current + " — watch the frontier reorder.");
    }
    if (r.done && r.reason === "goal") {
      path = rebuildPath(r.parent);
      AudioAI.goal();
      setStatus("Goal reached! Path: " + path.join(" → "));
      draw({ visited: r.visited, frontier: [], path });
      if (autoTimer) { clearInterval(autoTimer); autoTimer = null; }
      return true;
    }
    if (r.done && r.reason === "empty") {
      AudioAI.wrong();
      setStatus("Open set empty — no path to goal.");
      return true;
    }
    return false;
  }

  function wire() {
    document.getElementById("gqStepBtn").addEventListener("click", async () => {
      await AudioAI.unlock();
      stepOnce();
    });
    document.getElementById("gqAutoBtn").addEventListener("click", async () => {
      await AudioAI.unlock();
      if (autoTimer) { clearInterval(autoTimer); autoTimer = null; return; }
      autoTimer = setInterval(() => {
        const done = stepOnce();
        if (done) { clearInterval(autoTimer); autoTimer = null; }
      }, 520);
    });
    document.getElementById("gqResetBtn").addEventListener("click", () => { reset(); AudioAI.npcBlip(); });

    algoSelect.addEventListener("change", () => { reset(); AudioAI.npcBlip(); });

    npcEls.forEach(el => {
      el.addEventListener("click", () => {
        const m = el.getAttribute("data-mode");
        if (m && ["greedy", "dijkstra", "astar"].includes(m)) {
          algoSelect.value = m;
          reset();
          AudioAI.npcBlip();
        }
      });
      el.addEventListener("keydown", (e) => {
        if (e.key === "Enter" || e.key === " ") { e.preventDefault(); el.click(); }
      });
    });

    const tut = document.getElementById("gqTutorial");
    document.getElementById("gqTutorialBtn").addEventListener("click", () => { tut.classList.add("is-open"); tut.setAttribute("aria-hidden", "false"); });
    document.getElementById("gqTutorialClose").addEventListener("click", () => { tut.classList.remove("is-open"); tut.setAttribute("aria-hidden", "true"); });
    tut.addEventListener("click", (e) => { if (e.target === tut) { tut.classList.remove("is-open"); tut.setAttribute("aria-hidden", "true"); } });

    const muteBtn = document.getElementById("gqMuteBtn");
    muteBtn.addEventListener("click", async () => {
      await AudioAI.unlock();
      AudioAI.setMuted(!AudioAI.isMuted());
      muteBtn.textContent = AudioAI.isMuted() ? "🔇 Muted" : "🔊 Sound on";
    });

    document.getElementById("graphQuestApp").addEventListener("click", () => { AudioAI.unlock(); }, { once: true });
  }

  wire();
  reset();
})();
</script>
