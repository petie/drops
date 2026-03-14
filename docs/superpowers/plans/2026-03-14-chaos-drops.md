# Chaos Drops Simulator — Implementation Plan

> **For agentic workers:** REQUIRED: Use superpowers:subagent-driven-development (if subagents available) or superpowers:executing-plans to implement this plan. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** Build a single-file static HTML Chaos Drops simulator with animated drop reveals, real Brawl Stars items, synthesized sound, and persistent history tracking.

**Architecture:** Single `index.html` file containing all HTML, CSS, and JavaScript inline. No external dependencies. Canvas overlay for particle effects. Web Audio API for sound synthesis. localStorage for persistence.

**Tech Stack:** Vanilla HTML/CSS/JS, HTML5 Canvas, Web Audio API, localStorage

**Spec:** `docs/superpowers/specs/2026-03-14-chaos-drops-design.md`

---

## File Structure

Single file: `index.html`

Logically organized as sections within that file:
- **HTML:** DOM structure (header, drop area, recent drops, modals)
- **CSS:** `<style>` block with all styles + keyframe animations
- **JS: Config** — rarity tiers, split odds, reward pool (~50 items)
- **JS: State** — localStorage read/write, schema management
- **JS: RNG** — weighted random selection for rarity, splits, rewards
- **JS: Animation** — orchestrator for the 4-phase reveal sequence
- **JS: Particles** — Canvas-based particle system (swirl, burst, flash)
- **JS: Sound** — Web Audio API synthesized sounds per event
- **JS: UI** — event handlers, stats panel, collection view, recent drops

---

## Chunk 1: Foundation — Data, Layout, Drop Mechanic

### Task 1: HTML Scaffold + CSS Layout

**Files:**
- Create: `index.html`

Build the static shell: HTML structure, all CSS, mobile viewport. No interactivity yet — just the visual layout matching the approved mockup (Layout A: centered drop).

- [ ] **Step 1: Create index.html with full HTML structure and CSS**

```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0, user-scalable=no">
  <title>Chaos Drops</title>
  <style>
    /* --- Reset & Base --- */
    * { margin: 0; padding: 0; box-sizing: border-box; }
    body {
      font-family: -apple-system, BlinkMacSystemFont, 'Segoe UI', system-ui, sans-serif;
      background: linear-gradient(135deg, #1a0a2e 0%, #2d1b4e 50%, #1a0a2e 100%);
      color: #fff;
      min-height: 100dvh;
      overflow-x: hidden;
      -webkit-tap-highlight-color: transparent;
      user-select: none;
    }

    /* --- App Container --- */
    #app {
      display: flex;
      flex-direction: column;
      align-items: center;
      min-height: 100dvh;
      padding: 12px 16px;
      max-width: 480px;
      margin: 0 auto;
      position: relative;
    }

    /* --- Header --- */
    .header {
      display: flex;
      justify-content: space-between;
      width: 100%;
      align-items: center;
      margin-bottom: 16px;
    }
    .header-btn {
      font-size: 13px;
      background: rgba(255,255,255,0.1);
      border: 1px solid rgba(255,255,255,0.15);
      color: #ccc;
      padding: 6px 14px;
      border-radius: 20px;
      cursor: pointer;
      transition: background 0.2s;
    }
    .header-btn:active { background: rgba(255,255,255,0.2); }

    /* --- Title --- */
    .title {
      font-size: 28px;
      font-weight: 900;
      text-transform: uppercase;
      letter-spacing: 3px;
      background: linear-gradient(to bottom, #ffd700, #ff8c00);
      -webkit-background-clip: text;
      -webkit-text-fill-color: transparent;
      background-clip: text;
      text-align: center;
    }
    .drop-counter {
      font-size: 12px;
      color: #888;
      text-align: center;
      margin-top: 2px;
      margin-bottom: 24px;
    }

    /* --- Drop Box --- */
    .drop-zone {
      display: flex;
      align-items: center;
      justify-content: center;
      flex: 1;
      width: 100%;
      min-height: 200px;
    }
    .drop-box {
      width: 140px;
      height: 165px;
      background: radial-gradient(ellipse at center, #6b3fa0 0%, #4a1f7a 40%, #2d1050 100%);
      border-radius: 18px;
      border: 3px solid #9b59b6;
      box-shadow: 0 0 30px rgba(155, 89, 182, 0.4), inset 0 -5px 10px rgba(0,0,0,0.3);
      display: flex;
      align-items: center;
      justify-content: center;
      cursor: pointer;
      position: relative;
      transition: transform 0.1s;
    }
    .drop-box:active { transform: scale(0.95); }
    .drop-box-icon { font-size: 56px; }
    .drop-box-label {
      position: absolute;
      bottom: -10px;
      left: 50%;
      transform: translateX(-50%);
      font-size: 10px;
      font-weight: 700;
      background: #9b59b6;
      padding: 3px 12px;
      border-radius: 12px;
      white-space: nowrap;
      text-transform: uppercase;
      letter-spacing: 1px;
    }

    /* --- Rarity Bar --- */
    .rarity-bar {
      display: flex;
      gap: 4px;
      margin: 24px 0 4px;
    }
    .rarity-bar span {
      height: 5px;
      border-radius: 3px;
      flex: 1;
    }
    .rarity-labels {
      display: flex;
      gap: 4px;
      margin-bottom: 20px;
    }
    .rarity-labels span {
      flex: 1;
      text-align: center;
      font-size: 9px;
      color: #666;
    }

    /* --- Recent Drops --- */
    .recent-section {
      width: 100%;
      border-top: 1px solid rgba(255,255,255,0.08);
      padding-top: 12px;
    }
    .recent-label {
      font-size: 10px;
      color: #666;
      text-transform: uppercase;
      letter-spacing: 1px;
      margin-bottom: 8px;
    }
    .recent-row {
      display: flex;
      gap: 8px;
      overflow-x: auto;
      padding-bottom: 8px;
      -webkit-overflow-scrolling: touch;
    }
    .recent-row::-webkit-scrollbar { display: none; }
    .recent-item {
      min-width: 56px;
      background: rgba(255,255,255,0.05);
      border: 1px solid rgba(255,255,255,0.1);
      border-radius: 10px;
      padding: 8px 6px;
      text-align: center;
      font-size: 11px;
    }
    .recent-item-icon { font-size: 20px; margin-bottom: 2px; }
    .recent-item-name { font-size: 9px; color: #aaa; white-space: nowrap; }

    /* --- Canvas Overlay (particles) --- */
    #particles {
      position: fixed;
      top: 0; left: 0;
      width: 100%; height: 100%;
      pointer-events: none;
      z-index: 100;
    }

    /* --- Modal Overlay --- */
    .modal-overlay {
      position: fixed;
      top: 0; left: 0;
      width: 100%; height: 100%;
      background: rgba(0,0,0,0.85);
      z-index: 200;
      display: none;
      justify-content: center;
      align-items: flex-start;
      padding: 20px;
      overflow-y: auto;
    }
    .modal-overlay.active { display: flex; }
    .modal {
      background: linear-gradient(135deg, #1e0f35, #2a1848);
      border: 1px solid rgba(155,89,182,0.3);
      border-radius: 16px;
      padding: 24px;
      width: 100%;
      max-width: 440px;
      margin-top: 20px;
    }
    .modal h2 {
      font-size: 20px;
      font-weight: 800;
      margin-bottom: 16px;
      text-align: center;
    }
    .modal-close {
      position: absolute;
      top: 16px;
      right: 16px;
      font-size: 24px;
      color: #888;
      cursor: pointer;
      background: none;
      border: none;
    }

    /* --- Reveal Overlay (rarity + reward) --- */
    .reveal-overlay {
      position: fixed;
      top: 0; left: 0;
      width: 100%; height: 100%;
      background: rgba(0,0,0,0.9);
      z-index: 150;
      display: none;
      justify-content: center;
      align-items: center;
      flex-direction: column;
      cursor: pointer;
    }
    .reveal-overlay.active { display: flex; }
    .reveal-rarity-text {
      font-size: 20px;
      font-weight: 900;
      text-transform: uppercase;
      letter-spacing: 4px;
      margin-bottom: 16px;
      opacity: 0;
    }
    .reveal-item-icon { font-size: 72px; opacity: 0; }
    .reveal-item-name {
      font-size: 22px;
      font-weight: 800;
      margin-top: 12px;
      opacity: 0;
    }
    .reveal-item-type {
      font-size: 13px;
      margin-top: 4px;
      opacity: 0;
    }
    .reveal-duplicate {
      font-size: 11px;
      background: rgba(255,0,0,0.3);
      padding: 3px 10px;
      border-radius: 8px;
      margin-top: 8px;
      opacity: 0;
      display: none;
    }
    .reveal-tap-hint {
      position: absolute;
      bottom: 40px;
      font-size: 12px;
      color: #555;
      opacity: 0;
    }

    /* --- Split Announcement --- */
    .split-text {
      position: fixed;
      top: 50%; left: 50%;
      transform: translate(-50%, -50%) scale(0);
      font-size: 48px;
      font-weight: 900;
      z-index: 160;
      text-shadow: 0 0 30px currentColor;
      pointer-events: none;
    }

    /* --- Animations --- */
    @keyframes shake {
      0%, 100% { transform: translateX(0); }
      25% { transform: translateX(-4px) rotate(-1deg); }
      75% { transform: translateX(4px) rotate(1deg); }
    }
    @keyframes shakeHard {
      0%, 100% { transform: translateX(0); }
      10% { transform: translateX(-6px) rotate(-2deg); }
      30% { transform: translateX(5px) rotate(1.5deg); }
      50% { transform: translateX(-7px) rotate(-2.5deg); }
      70% { transform: translateX(6px) rotate(2deg); }
      90% { transform: translateX(-4px) rotate(-1deg); }
    }
    @keyframes pulse { 0% { transform: scale(1); } 100% { transform: scale(1.06); } }
    @keyframes slamIn {
      0% { transform: scale(3); opacity: 0; }
      60% { transform: scale(0.9); opacity: 1; }
      100% { transform: scale(1); opacity: 1; }
    }
    @keyframes fadeInUp {
      from { transform: translateY(20px); opacity: 0; }
      to { transform: translateY(0); opacity: 1; }
    }
    @keyframes zoomIn {
      from { transform: scale(0); opacity: 0; }
      to { transform: scale(1); opacity: 1; }
    }
    @keyframes flash {
      0% { opacity: 1; }
      100% { opacity: 0; }
    }
    @keyframes screenShake {
      0%, 100% { transform: translate(0, 0); }
      10% { transform: translate(-8px, -5px); }
      20% { transform: translate(7px, 6px); }
      30% { transform: translate(-6px, 4px); }
      40% { transform: translate(5px, -7px); }
      50% { transform: translate(-4px, 5px); }
      60% { transform: translate(6px, -4px); }
      70% { transform: translate(-5px, 3px); }
      80% { transform: translate(4px, -6px); }
      90% { transform: translate(-3px, 5px); }
    }

    /* --- Stats Panel Styles --- */
    .stat-row {
      display: flex;
      justify-content: space-between;
      padding: 8px 0;
      border-bottom: 1px solid rgba(255,255,255,0.05);
      font-size: 14px;
    }
    .stat-row .label { color: #aaa; }
    .stat-row .value { font-weight: 700; }
    .bar-chart { margin: 12px 0; }
    .bar-row {
      display: flex;
      align-items: center;
      margin-bottom: 6px;
      font-size: 12px;
    }
    .bar-row .bar-label { width: 70px; color: #aaa; }
    .bar-row .bar-track {
      flex: 1;
      height: 14px;
      background: rgba(255,255,255,0.05);
      border-radius: 7px;
      overflow: hidden;
      margin: 0 8px;
    }
    .bar-row .bar-fill {
      height: 100%;
      border-radius: 7px;
      transition: width 0.3s;
    }
    .bar-row .bar-count { width: 35px; text-align: right; color: #ccc; }

    /* --- Collection Grid --- */
    .collection-grid {
      display: grid;
      grid-template-columns: repeat(4, 1fr);
      gap: 8px;
      margin-top: 12px;
    }
    .collection-item {
      background: rgba(255,255,255,0.05);
      border: 1px solid rgba(255,255,255,0.1);
      border-radius: 10px;
      padding: 8px;
      text-align: center;
      cursor: pointer;
      transition: background 0.2s;
    }
    .collection-item.locked {
      opacity: 0.3;
      filter: grayscale(1);
    }
    .collection-item-icon { font-size: 24px; }
    .collection-item-name { font-size: 9px; color: #aaa; margin-top: 2px; }

    /* --- Luck Meter --- */
    .luck-meter {
      margin: 12px 0;
      padding: 10px;
      background: rgba(255,255,255,0.03);
      border-radius: 10px;
    }
    .luck-meter h4 { font-size: 12px; color: #aaa; margin-bottom: 8px; }

    /* --- Buttons --- */
    .btn-danger {
      background: rgba(231, 76, 60, 0.2);
      border: 1px solid rgba(231, 76, 60, 0.4);
      color: #e74c3c;
      padding: 8px 20px;
      border-radius: 10px;
      font-size: 13px;
      cursor: pointer;
      margin-top: 16px;
      width: 100%;
    }
    .btn-danger:active { background: rgba(231, 76, 60, 0.4); }

    /* --- Tabs --- */
    .tabs {
      display: flex;
      gap: 0;
      margin-bottom: 16px;
      border-bottom: 2px solid rgba(255,255,255,0.1);
    }
    .tab {
      flex: 1;
      text-align: center;
      padding: 8px;
      font-size: 13px;
      font-weight: 600;
      color: #666;
      cursor: pointer;
      border-bottom: 2px solid transparent;
      margin-bottom: -2px;
      transition: color 0.2s;
    }
    .tab.active { color: #fff; border-bottom-color: #9b59b6; }
  </style>
</head>
<body>
  <div id="app">
    <!-- Header -->
    <div class="header">
      <button class="header-btn" id="soundToggle">🔊 Sound</button>
      <button class="header-btn" id="statsBtn">📊 Stats</button>
    </div>

    <!-- Title -->
    <div class="title">Chaos Drops</div>
    <div class="drop-counter">Drops Opened: <span id="dropCount">0</span></div>

    <!-- Drop Zone -->
    <div class="drop-zone">
      <div class="drop-box" id="dropBox">
        <div class="drop-box-icon">🎁</div>
        <div class="drop-box-label">Tap to Open</div>
      </div>
    </div>

    <!-- Rarity Bar -->
    <div class="rarity-bar">
      <span style="background: #3498db;"></span>
      <span style="background: #9b59b6;"></span>
      <span style="background: #e74c3c;"></span>
      <span style="background: #f39c12;"></span>
      <span style="background: #e91e63;"></span>
    </div>
    <div class="rarity-labels">
      <span>S.Rare</span><span>Epic</span><span>Mythic</span><span>Legend</span><span>Ultra</span>
    </div>

    <!-- Recent Drops -->
    <div class="recent-section">
      <div class="recent-label">Recent Drops</div>
      <div class="recent-row" id="recentRow">
        <!-- Populated by JS -->
      </div>
    </div>
  </div>

  <!-- Canvas for particles -->
  <canvas id="particles"></canvas>

  <!-- Reveal Overlay -->
  <div class="reveal-overlay" id="revealOverlay">
    <div class="reveal-rarity-text" id="revealRarity"></div>
    <div class="reveal-item-icon" id="revealIcon"></div>
    <div class="reveal-item-name" id="revealName"></div>
    <div class="reveal-item-type" id="revealType"></div>
    <div class="reveal-duplicate" id="revealDuplicate">DUPLICATE — Fallback Reward</div>
    <div class="reveal-tap-hint">Tap anywhere to continue</div>
  </div>

  <!-- Split Text -->
  <div class="split-text" id="splitText"></div>

  <!-- Stats Modal -->
  <div class="modal-overlay" id="statsModal">
    <div class="modal" style="position: relative;">
      <button class="modal-close" id="statsClose">✕</button>
      <h2>📊 Stats</h2>
      <div class="tabs">
        <div class="tab active" data-tab="stats">Stats</div>
        <div class="tab" data-tab="collection">Collection</div>
      </div>
      <div id="statsContent">
        <!-- Populated by JS -->
      </div>
      <div id="collectionContent" style="display: none;">
        <!-- Populated by JS -->
      </div>
      <button class="btn-danger" id="resetBtn">Reset All Data</button>
    </div>
  </div>

  <script>
  // === JS goes here in subsequent tasks ===
  </script>
</body>
</html>
```

- [ ] **Step 2: Open in browser to verify layout**

Run: `open index.html` (macOS) or just open the file in a browser.
Expected: Dark purple background, centered "CHAOS DROPS" title in gold, drop box in the center with "Tap to Open" label, rarity bar at bottom, empty recent drops section. Layout should be centered and mobile-width on desktop.

- [ ] **Step 3: Commit**

```bash
git init
git add index.html
git commit -m "feat: scaffold HTML structure and CSS layout"
```

---

### Task 2: Configuration Data — Rarity, Splits, Reward Pool

**Files:**
- Modify: `index.html` (inside `<script>` tag)

Define all static configuration: rarity tiers with probabilities and colors, split odds, and the full reward pool of ~50 real Brawl Stars items with emoji icons.

- [ ] **Step 1: Add configuration constants inside the `<script>` tag**

Replace the `// === JS goes here ===` comment with the config block. All subsequent JS tasks append after this block.

```javascript
// ============================================================
// CONFIG
// ============================================================

const RARITIES = [
  { name: 'Super Rare', chance: 0.52, color: '#3498db', glowSize: 20, shakeClass: '' },
  { name: 'Epic',       chance: 0.28, color: '#9b59b6', glowSize: 30, shakeClass: '' },
  { name: 'Mythic',     chance: 0.12, color: '#e74c3c', glowSize: 40, shakeClass: 'shake' },
  { name: 'Legendary',  chance: 0.06, color: '#f39c12', glowSize: 60, shakeClass: 'shakeHard' },
  { name: 'Ultra',      chance: 0.02, color: '#e91e63', glowSize: 80, shakeClass: 'shakeHard' },
];

const SPLITS = [
  { count: 1, chance: 0.70 },
  { count: 2, chance: 0.20 },
  { count: 4, chance: 0.08 },
  { count: 8, chance: 0.02 },
];

// Reward pool — each item has: id, name, type, icon (emoji), rarities (which rarity tiers can drop it)
const REWARDS = [
  // --- Brawlers ---
  { id: 'b1',  name: 'Darryl',     type: 'Brawler', icon: '🏴‍☠️', rarities: [0] },
  { id: 'b2',  name: 'Penny',      type: 'Brawler', icon: '💰', rarities: [0] },
  { id: 'b3',  name: 'Rico',       type: 'Brawler', icon: '🤖', rarities: [0] },
  { id: 'b4',  name: 'Piper',      type: 'Brawler', icon: '☂️', rarities: [1] },
  { id: 'b5',  name: 'Frank',      type: 'Brawler', icon: '🔨', rarities: [1] },
  { id: 'b6',  name: 'Bibi',       type: 'Brawler', icon: '⚾', rarities: [1] },
  { id: 'b7',  name: 'Mortis',     type: 'Brawler', icon: '🦇', rarities: [2] },
  { id: 'b8',  name: 'Tara',       type: 'Brawler', icon: '🔮', rarities: [2] },
  { id: 'b9',  name: 'Gene',       type: 'Brawler', icon: '🧞', rarities: [2] },
  { id: 'b10', name: 'Crow',       type: 'Brawler', icon: '🐦', rarities: [3] },
  { id: 'b11', name: 'Leon',       type: 'Brawler', icon: '🦎', rarities: [3] },
  { id: 'b12', name: 'Spike',      type: 'Brawler', icon: '🌵', rarities: [3] },
  { id: 'b13', name: 'Amber',      type: 'Brawler', icon: '🔥', rarities: [3] },
  { id: 'b14', name: 'Draco',      type: 'Brawler', icon: '🐉', rarities: [4] },
  { id: 'b15', name: 'Kenji',      type: 'Brawler', icon: '⚔️', rarities: [4] },

  // --- Gadgets (Super Rare - Mythic) ---
  { id: 'g1', name: 'Recoiling Rotator',  type: 'Gadget', icon: '🎯', rarities: [0] },
  { id: 'g2', name: 'Pocket Detonator',   type: 'Gadget', icon: '💣', rarities: [0] },
  { id: 'g3', name: 'Friendzoner',        type: 'Gadget', icon: '🛡️', rarities: [1] },
  { id: 'g4', name: 'Tweete',             type: 'Gadget', icon: '🐤', rarities: [1] },
  { id: 'g5', name: 'Survival Shovel',    type: 'Gadget', icon: '⛏️', rarities: [2] },
  { id: 'g6', name: 'Psychic Enhancer',   type: 'Gadget', icon: '🧠', rarities: [2] },

  // --- Star Powers (Epic - Legendary) ---
  { id: 'sp1', name: 'Snappy Sniping',  type: 'Star Power', icon: '⭐', rarities: [1] },
  { id: 'sp2', name: 'Sponge',          type: 'Star Power', icon: '⭐', rarities: [1] },
  { id: 'sp3', name: 'Coiled Snake',    type: 'Star Power', icon: '⭐', rarities: [2] },
  { id: 'sp4', name: 'Healing Shade',   type: 'Star Power', icon: '⭐', rarities: [2] },
  { id: 'sp5', name: 'Carrion Crow',    type: 'Star Power', icon: '⭐', rarities: [3] },
  { id: 'sp6', name: 'Invisiheal',      type: 'Star Power', icon: '⭐', rarities: [3] },

  // --- Hypercharges (Mythic - Ultra) ---
  { id: 'hc1', name: 'Mortis Hypercharge',  type: 'Hypercharge', icon: '⚡', rarities: [2] },
  { id: 'hc2', name: 'Tara Hypercharge',    type: 'Hypercharge', icon: '⚡', rarities: [2] },
  { id: 'hc3', name: 'Crow Hypercharge',    type: 'Hypercharge', icon: '⚡', rarities: [3] },
  { id: 'hc4', name: 'Leon Hypercharge',    type: 'Hypercharge', icon: '⚡', rarities: [3] },
  { id: 'hc5', name: 'Draco Hypercharge',   type: 'Hypercharge', icon: '⚡', rarities: [4] },
  { id: 'hc6', name: 'Kenji Hypercharge',   type: 'Hypercharge', icon: '⚡', rarities: [4] },

  // --- Skins (all rarities) ---
  { id: 's1',  name: 'Bandit Shelly',     type: 'Skin', icon: '👗', rarities: [0] },
  { id: 's2',  name: 'Rockstar Colt',     type: 'Skin', icon: '🎸', rarities: [0] },
  { id: 's3',  name: 'Royal Agent Colt',  type: 'Skin', icon: '🕶️', rarities: [1] },
  { id: 's4',  name: 'Phoenix Crow',      type: 'Skin', icon: '🔥', rarities: [3] },
  { id: 's5',  name: 'Werewolf Leon',     type: 'Skin', icon: '🐺', rarities: [3] },
  { id: 's6',  name: 'Mecha Crow',        type: 'Skin', icon: '🤖', rarities: [4] },
  { id: 's7',  name: 'Mecha Leon',        type: 'Skin', icon: '🦾', rarities: [4] },

  // --- Currency: Coins (all rarities, weighted heavier) ---
  { id: 'c1', name: '100 Coins',    type: 'Coins', icon: '🪙', rarities: [0] },
  { id: 'c2', name: '200 Coins',    type: 'Coins', icon: '🪙', rarities: [0, 1] },
  { id: 'c3', name: '500 Coins',    type: 'Coins', icon: '🪙', rarities: [1, 2] },
  { id: 'c4', name: '1000 Coins',   type: 'Coins', icon: '🪙', rarities: [2, 3] },
  { id: 'c5', name: '2000 Coins',   type: 'Coins', icon: '🪙', rarities: [3, 4] },

  // --- Currency: Power Points ---
  { id: 'pp1', name: '50 Power Points',   type: 'Power Points', icon: '🔋', rarities: [0] },
  { id: 'pp2', name: '100 Power Points',  type: 'Power Points', icon: '🔋', rarities: [0, 1] },
  { id: 'pp3', name: '200 Power Points',  type: 'Power Points', icon: '🔋', rarities: [1, 2] },

  // --- Currency: Bling (higher rarities) ---
  { id: 'bl1', name: '50 Bling',   type: 'Bling', icon: '💎', rarities: [1, 2] },
  { id: 'bl2', name: '100 Bling',  type: 'Bling', icon: '💎', rarities: [2, 3] },
  { id: 'bl3', name: '200 Bling',  type: 'Bling', icon: '💎', rarities: [3, 4] },

  // --- Currency: Credits ---
  { id: 'cr1', name: '100 Credits',   type: 'Credits', icon: '🎫', rarities: [0, 1] },
  { id: 'cr2', name: '500 Credits',   type: 'Credits', icon: '🎫', rarities: [2, 3] },
  { id: 'cr3', name: '1500 Credits',  type: 'Credits', icon: '🎫', rarities: [4] },

  // --- Buffie (Ultra only) ---
  { id: 'buf1', name: 'Buffie',  type: 'Buffie', icon: '🧸', rarities: [4] },
];

// Items that are "collectible" (non-currency, tracked for duplicates)
const COLLECTIBLE_TYPES = ['Brawler', 'Gadget', 'Star Power', 'Hypercharge', 'Skin', 'Buffie'];

// Fallback rewards for duplicates, by rarity index
const DUPLICATE_FALLBACKS = [
  { name: '100 Bling',    icon: '💎' },  // Super Rare
  { name: '150 Bling',    icon: '💎' },  // Epic
  { name: '250 Bling',    icon: '💎' },  // Mythic
  { name: '500 Credits',  icon: '🎫' },  // Legendary
  { name: '1500 Credits', icon: '🎫' },  // Ultra
];
```

- [ ] **Step 2: Verify config loads without errors**

Open browser console (F12), refresh page.
Expected: No errors. Type `RARITIES.length` → 5, `REWARDS.length` → ~50, `SPLITS.length` → 4.

- [ ] **Step 3: Commit**

```bash
git add index.html
git commit -m "feat: add rarity, split, and reward pool configuration data"
```

---

### Task 3: State Management — localStorage

**Files:**
- Modify: `index.html` (append to `<script>` tag after config)

Implement the state layer: loading/saving to localStorage, default state creation, and state mutation helpers.

- [ ] **Step 1: Add state management code after the config block**

```javascript
// ============================================================
// STATE
// ============================================================

const STORAGE_KEY = 'chaosDrops';

function defaultState() {
  return {
    totalDrops: 0,
    rarityCount: [0, 0, 0, 0, 0],
    splitCount: { 1: 0, 2: 0, 4: 0, 8: 0 },
    bestDrop: null, // { rarityIndex, splitCount }
    collection: {}, // id -> { dropNumber }
    recentDrops: [], // last 10: { id, name, type, icon, rarityIndex }
    soundOn: true,
  };
}

let state = loadState();

function loadState() {
  try {
    const raw = localStorage.getItem(STORAGE_KEY);
    if (raw) {
      const saved = JSON.parse(raw);
      // Merge with defaults for forward compatibility
      return { ...defaultState(), ...saved };
    }
  } catch (e) { /* corrupted data, start fresh */ }
  return defaultState();
}

function saveState() {
  try {
    localStorage.setItem(STORAGE_KEY, JSON.stringify(state));
  } catch (e) {
    console.warn('Failed to save state:', e);
  }
}

function resetState() {
  state = defaultState();
  saveState();
  updateUI();
}

function recordDrop(rarityIndex, splitCount, rewards) {
  state.totalDrops += rewards.length;
  state.rarityCount[rarityIndex] += rewards.length;
  state.splitCount[splitCount] = (state.splitCount[splitCount] || 0) + 1;

  // Best drop check
  if (!state.bestDrop ||
      rarityIndex > state.bestDrop.rarityIndex ||
      (rarityIndex === state.bestDrop.rarityIndex && splitCount > state.bestDrop.splitCount)) {
    state.bestDrop = { rarityIndex, splitCount };
  }

  // Add to recent (newest first, max 10)
  for (const r of rewards) {
    state.recentDrops.unshift(r);
  }
  state.recentDrops = state.recentDrops.slice(0, 10);

  // Track collectibles
  for (const r of rewards) {
    if (COLLECTIBLE_TYPES.includes(r.type) && !r.isDuplicate) {
      state.collection[r.id] = { dropNumber: state.totalDrops };
    }
  }

  saveState();
}
```

- [ ] **Step 2: Verify state works in console**

Refresh page. In console:
- `state` → should show default state object
- `state.totalDrops` → `0`
- `state.soundOn` → `true`

- [ ] **Step 3: Commit**

```bash
git add index.html
git commit -m "feat: add localStorage state management"
```

---

### Task 4: RNG Engine + Drop Mechanic

**Files:**
- Modify: `index.html` (append to `<script>` tag)

Implement weighted random selection and the core drop logic: roll rarity → roll split → pick rewards (with duplicate protection).

- [ ] **Step 1: Add RNG and drop mechanic**

```javascript
// ============================================================
// RNG
// ============================================================

function weightedRandom(items, getWeight) {
  const total = items.reduce((sum, item) => sum + getWeight(item), 0);
  let roll = Math.random() * total;
  for (const item of items) {
    roll -= getWeight(item);
    if (roll <= 0) return item;
  }
  return items[items.length - 1];
}

function rollRarity() {
  return RARITIES.indexOf(weightedRandom(RARITIES, r => r.chance));
}

function rollSplit() {
  return weightedRandom(SPLITS, s => s.chance).count;
}

function getRewardsForRarity(rarityIndex) {
  return REWARDS.filter(r => r.rarities.includes(rarityIndex));
}

function pickReward(rarityIndex) {
  const pool = getRewardsForRarity(rarityIndex);
  if (pool.length === 0) return null;

  const reward = pool[Math.floor(Math.random() * pool.length)];
  const isCollectible = COLLECTIBLE_TYPES.includes(reward.type);
  const isDuplicate = isCollectible && state.collection[reward.id];

  if (isDuplicate) {
    const fallback = DUPLICATE_FALLBACKS[rarityIndex];
    return {
      id: reward.id,
      name: fallback.name,
      type: 'Fallback',
      icon: fallback.icon,
      rarityIndex,
      isDuplicate: true,
      originalName: reward.name,
      originalType: reward.type,
    };
  }

  return {
    id: reward.id,
    name: reward.name,
    type: reward.type,
    icon: reward.icon,
    rarityIndex,
    isDuplicate: false,
  };
}

function performDrop() {
  const rarityIndex = rollRarity();
  const splitCount = rollSplit();
  const rewards = [];
  for (let i = 0; i < splitCount; i++) {
    rewards.push(pickReward(rarityIndex));
  }
  return { rarityIndex, splitCount, rewards };
}
```

- [ ] **Step 2: Verify in console**

Refresh page. In console:
- `rollRarity()` → returns 0-4 (run multiple times, mostly 0 and 1)
- `rollSplit()` → returns 1, 2, 4, or 8 (mostly 1)
- `performDrop()` → returns `{ rarityIndex, splitCount, rewards: [...] }`
- Run `performDrop()` 10 times, verify rewards have correct structure

- [ ] **Step 3: Commit**

```bash
git add index.html
git commit -m "feat: add RNG engine and drop mechanic with duplicate protection"
```

---

## Chunk 2: Animation System + Particles

### Task 5: Particle System (Canvas)

**Files:**
- Modify: `index.html` (append to `<script>` tag)

Build the Canvas-based particle engine that powers swirl-in, burst, and flash effects across all animation phases.

- [ ] **Step 1: Add particle system**

```javascript
// ============================================================
// PARTICLES
// ============================================================

const particleCanvas = document.getElementById('particles');
const pCtx = particleCanvas.getContext('2d');
let particles = [];
let particleAnimId = null;
const MAX_PARTICLES = 300;

function resizeCanvas() {
  particleCanvas.width = window.innerWidth;
  particleCanvas.height = window.innerHeight;
}
window.addEventListener('resize', resizeCanvas);
resizeCanvas();

class Particle {
  constructor(x, y, color, opts = {}) {
    this.x = x;
    this.y = y;
    this.color = color;
    this.size = opts.size || (Math.random() * 4 + 2);
    this.speedX = opts.speedX || (Math.random() - 0.5) * 8;
    this.speedY = opts.speedY || (Math.random() - 0.5) * 8;
    this.life = opts.life || 1;
    this.decay = opts.decay || (Math.random() * 0.02 + 0.01);
    this.gravity = opts.gravity || 0;
  }
  update() {
    this.x += this.speedX;
    this.y += this.speedY;
    this.speedY += this.gravity;
    this.life -= this.decay;
  }
  draw(ctx) {
    ctx.globalAlpha = Math.max(0, this.life);
    ctx.fillStyle = this.color;
    ctx.beginPath();
    ctx.arc(this.x, this.y, this.size, 0, Math.PI * 2);
    ctx.fill();
  }
}

function startParticleLoop() {
  if (particleAnimId) return;
  function loop() {
    pCtx.clearRect(0, 0, particleCanvas.width, particleCanvas.height);
    particles = particles.filter(p => p.life > 0);
    // Evict oldest particles if over cap
    if (particles.length > MAX_PARTICLES) {
      particles = particles.slice(particles.length - MAX_PARTICLES);
    }
    for (const p of particles) {
      p.update();
      p.draw(pCtx);
    }
    pCtx.globalAlpha = 1;
    if (particles.length > 0) {
      particleAnimId = requestAnimationFrame(loop);
    } else {
      particleAnimId = null;
      pCtx.clearRect(0, 0, particleCanvas.width, particleCanvas.height);
    }
  }
  particleAnimId = requestAnimationFrame(loop);
}

function emitBurst(x, y, color, count = 30) {
  for (let i = 0; i < count; i++) {
    const angle = (Math.PI * 2 * i) / count + Math.random() * 0.3;
    const speed = Math.random() * 6 + 2;
    particles.push(new Particle(x, y, color, {
      speedX: Math.cos(angle) * speed,
      speedY: Math.sin(angle) * speed,
      size: Math.random() * 5 + 1,
      decay: Math.random() * 0.015 + 0.008,
      gravity: 0.05,
    }));
  }
  startParticleLoop();
}

function emitSwirl(targetX, targetY, color, duration = 1000) {
  const interval = 30;
  let elapsed = 0;
  const id = setInterval(() => {
    elapsed += interval;
    if (elapsed >= duration) { clearInterval(id); return; }
    const angle = Math.random() * Math.PI * 2;
    const dist = 150 + Math.random() * 80;
    const startX = targetX + Math.cos(angle) * dist;
    const startY = targetY + Math.sin(angle) * dist;
    const dx = targetX - startX;
    const dy = targetY - startY;
    const steps = 40 + Math.random() * 20;
    particles.push(new Particle(startX, startY, color, {
      speedX: dx / steps,
      speedY: dy / steps,
      size: Math.random() * 3 + 1,
      life: 1,
      decay: 1 / steps,
    }));
    startParticleLoop();
  }, interval);
  return id;
}

function emitFlash(color) {
  const cx = particleCanvas.width / 2;
  const cy = particleCanvas.height / 2;
  for (let i = 0; i < 60; i++) {
    const angle = (Math.PI * 2 * i) / 60;
    const speed = Math.random() * 12 + 6;
    particles.push(new Particle(cx, cy, color, {
      speedX: Math.cos(angle) * speed,
      speedY: Math.sin(angle) * speed,
      size: Math.random() * 6 + 2,
      decay: Math.random() * 0.01 + 0.006,
      gravity: 0,
    }));
  }
  startParticleLoop();
}
```

- [ ] **Step 2: Test particles in console**

Refresh page. In console:
- `emitBurst(200, 400, '#f39c12', 40)` → gold particles burst from that point
- `emitSwirl(200, 400, '#9b59b6')` → purple particles swirl inward
- `emitFlash('#e91e63')` → pink particles radiate from center

- [ ] **Step 3: Commit**

```bash
git add index.html
git commit -m "feat: add Canvas particle system with burst, swirl, and flash effects"
```

---

### Task 6: Sound System (Web Audio API)

**Files:**
- Modify: `index.html` (append to `<script>` tag)

Build the synthesized sound engine: all sounds generated programmatically with oscillators, noise, and envelopes.

- [ ] **Step 1: Add sound system**

```javascript
// ============================================================
// SOUND
// ============================================================

let audioCtx = null;

function getAudioCtx() {
  if (!audioCtx) audioCtx = new (window.AudioContext || window.webkitAudioContext)();
  if (audioCtx.state === 'suspended') audioCtx.resume();
  return audioCtx;
}

function playTone(freq, duration, type = 'sine', volume = 0.3, delay = 0) {
  if (!state.soundOn) return;
  const ctx = getAudioCtx();
  const osc = ctx.createOscillator();
  const gain = ctx.createGain();
  osc.type = type;
  osc.frequency.setValueAtTime(freq, ctx.currentTime + delay);
  gain.gain.setValueAtTime(volume, ctx.currentTime + delay);
  gain.gain.exponentialRampToValueAtTime(0.001, ctx.currentTime + delay + duration);
  osc.connect(gain);
  gain.connect(ctx.destination);
  osc.start(ctx.currentTime + delay);
  osc.stop(ctx.currentTime + delay + duration);
}

function playNoise(duration, volume = 0.2, delay = 0) {
  if (!state.soundOn) return;
  const ctx = getAudioCtx();
  const bufferSize = ctx.sampleRate * duration;
  const buffer = ctx.createBuffer(1, bufferSize, ctx.sampleRate);
  const data = buffer.getChannelData(0);
  for (let i = 0; i < bufferSize; i++) data[i] = Math.random() * 2 - 1;
  const source = ctx.createBufferSource();
  source.buffer = buffer;
  const gain = ctx.createGain();
  gain.gain.setValueAtTime(volume, ctx.currentTime + delay);
  gain.gain.exponentialRampToValueAtTime(0.001, ctx.currentTime + delay + duration);
  source.connect(gain);
  gain.connect(ctx.destination);
  source.start(ctx.currentTime + delay);
}

function playSweep(startFreq, endFreq, duration, type = 'sine', volume = 0.2, delay = 0) {
  if (!state.soundOn) return;
  const ctx = getAudioCtx();
  const osc = ctx.createOscillator();
  const gain = ctx.createGain();
  osc.type = type;
  osc.frequency.setValueAtTime(startFreq, ctx.currentTime + delay);
  osc.frequency.exponentialRampToValueAtTime(endFreq, ctx.currentTime + delay + duration);
  gain.gain.setValueAtTime(0.001, ctx.currentTime);
  if (delay > 0) gain.gain.setValueAtTime(0.001, ctx.currentTime + delay - 0.001);
  gain.gain.setValueAtTime(volume, ctx.currentTime + delay);
  gain.gain.exponentialRampToValueAtTime(0.001, ctx.currentTime + delay + duration);
  osc.connect(gain);
  gain.connect(ctx.destination);
  osc.start(ctx.currentTime + delay);
  osc.stop(ctx.currentTime + delay + duration);
}

function createReverb(duration = 1.5) {
  const ctx = getAudioCtx();
  const length = ctx.sampleRate * duration;
  const impulse = ctx.createBuffer(2, length, ctx.sampleRate);
  for (let ch = 0; ch < 2; ch++) {
    const data = impulse.getChannelData(ch);
    for (let i = 0; i < length; i++) {
      data[i] = (Math.random() * 2 - 1) * Math.pow(1 - i / length, 2);
    }
  }
  const convolver = ctx.createConvolver();
  convolver.buffer = impulse;
  return convolver;
}

function playToneWithReverb(freq, duration, type = 'sine', volume = 0.3, delay = 0, reverbDuration = 1.5) {
  if (!state.soundOn) return;
  const ctx = getAudioCtx();
  const osc = ctx.createOscillator();
  const gain = ctx.createGain();
  const reverb = createReverb(reverbDuration);
  const reverbGain = ctx.createGain();
  osc.type = type;
  osc.frequency.setValueAtTime(freq, ctx.currentTime + delay);
  gain.gain.setValueAtTime(volume, ctx.currentTime + delay);
  gain.gain.exponentialRampToValueAtTime(0.001, ctx.currentTime + delay + duration);
  reverbGain.gain.setValueAtTime(0.4, ctx.currentTime);
  // Dry signal
  osc.connect(gain);
  gain.connect(ctx.destination);
  // Wet signal (reverb)
  osc.connect(reverb);
  reverb.connect(reverbGain);
  reverbGain.connect(ctx.destination);
  osc.start(ctx.currentTime + delay);
  osc.stop(ctx.currentTime + delay + duration);
}

const Sound = {
  chargeUp() {
    playSweep(200, 800, 1.0, 'sawtooth', 0.15);
    playSweep(80, 150, 1.0, 'sine', 0.2); // rumble
  },
  split(count) {
    playNoise(0.15, 0.3); // shatter
    for (let i = 0; i < count; i++) {
      playTone(400 + i * 100, 0.1, 'sine', 0.2, 0.15 + i * 0.08);
    }
  },
  rarityReveal(rarityIndex) {
    const reveals = [
      () => { // Super Rare — two-note chime
        playTone(523, 0.3, 'sine', 0.25);
        playTone(659, 0.4, 'sine', 0.25, 0.15);
      },
      () => { // Epic — three-note ascending
        playTone(523, 0.25, 'sine', 0.25);
        playTone(659, 0.25, 'sine', 0.25, 0.12);
        playTone(784, 0.35, 'sine', 0.25, 0.24);
      },
      () => { // Mythic — dramatic stab
        playTone(220, 0.4, 'sawtooth', 0.2);
        playTone(440, 0.4, 'sawtooth', 0.15, 0.05);
        playTone(880, 0.5, 'square', 0.1, 0.1);
      },
      () => { // Legendary — fanfare sweep with reverb
        playSweep(300, 600, 0.3, 'sawtooth', 0.2);
        playToneWithReverb(784, 0.3, 'sine', 0.2, 0.2, 1.5);
        playToneWithReverb(988, 0.3, 'sine', 0.2, 0.35, 1.5);
        playToneWithReverb(1175, 0.5, 'sine', 0.25, 0.5, 2.0);
      },
      () => { // Ultra — bass drop + shimmer + long reverb tail
        playSweep(600, 60, 0.5, 'sawtooth', 0.3);
        playNoise(0.3, 0.15, 0.1);
        playSweep(2000, 4000, 0.8, 'sine', 0.1, 0.2);
        playToneWithReverb(1175, 0.8, 'sine', 0.15, 0.3, 3.0);
      },
    ];
    reveals[rarityIndex]();
  },
  rewardReveal() {
    playTone(880, 0.15, 'sine', 0.2);
    playTone(1320, 0.2, 'sine', 0.15, 0.08);
    playTone(1760, 0.3, 'sine', 0.1, 0.16);
  },
  crack() {
    playNoise(0.08, 0.15);
  },
  uiClick() {
    playTone(600, 0.05, 'sine', 0.1);
  },
};
```

- [ ] **Step 2: Test sounds in console**

Refresh page. In console (make sure volume is up):
- `Sound.chargeUp()` → rising sweep with rumble
- `Sound.split(4)` → crack then 4 pops
- `Sound.rarityReveal(3)` → legendary fanfare
- `Sound.rewardReveal()` → sparkle ding
- `Sound.uiClick()` → subtle click

- [ ] **Step 3: Commit**

```bash
git add index.html
git commit -m "feat: add Web Audio API sound synthesis engine"
```

---

### Task 7: Animation Orchestrator — The 4-Phase Reveal

**Files:**
- Modify: `index.html` (append to `<script>` tag)

Build the animation controller that sequences all 4 phases: charge up → split → rarity reveal → reward reveal. This is the core experience.

- [ ] **Step 1: Add animation orchestrator**

```javascript
// ============================================================
// ANIMATION ORCHESTRATOR
// ============================================================

let isAnimating = false;

const dropBox = document.getElementById('dropBox');
const revealOverlay = document.getElementById('revealOverlay');
const revealRarity = document.getElementById('revealRarity');
const revealIcon = document.getElementById('revealIcon');
const revealName = document.getElementById('revealName');
const revealType = document.getElementById('revealType');
const revealDuplicate = document.getElementById('revealDuplicate');
const splitTextEl = document.getElementById('splitText');

function sleep(ms) { return new Promise(r => setTimeout(r, ms)); }

function getDropBoxCenter() {
  const rect = dropBox.getBoundingClientRect();
  return { x: rect.left + rect.width / 2, y: rect.top + rect.height / 2 };
}

async function animateChargeUp(rarityIndex) {
  const color = RARITIES[rarityIndex].color;
  const center = getDropBoxCenter();

  // Start swirl particles
  const swirlId = emitSwirl(center.x, center.y, color, 900);

  // Start sound
  Sound.chargeUp();

  // Animate box: increasing glow + shake
  dropBox.style.transition = 'box-shadow 1s, transform 0.05s';
  dropBox.style.boxShadow = `0 0 ${RARITIES[rarityIndex].glowSize}px ${color}, inset 0 -5px 10px rgba(0,0,0,0.3)`;
  dropBox.style.animation = 'shake 0.1s infinite';

  await sleep(500);
  dropBox.style.animation = 'shakeHard 0.08s infinite';

  await sleep(500);
  clearInterval(swirlId);
}

async function animateSplitCheck(splitCount, rarityIndex) {
  const color = RARITIES[rarityIndex].color;
  const center = getDropBoxCenter();

  if (splitCount > 1) {
    // Shatter burst
    Sound.split(splitCount);
    emitBurst(center.x, center.y, color, 40);
    emitBurst(center.x, center.y, '#fff', 15);

    // Flash the split text
    splitTextEl.textContent = `×${splitCount} SPLIT!`;
    splitTextEl.style.color = color;
    splitTextEl.style.transform = 'translate(-50%, -50%) scale(0)';
    splitTextEl.style.display = 'block';
    splitTextEl.style.opacity = '1';

    // Animate slam in
    await sleep(50);
    splitTextEl.style.transition = 'transform 0.3s cubic-bezier(0.17, 0.67, 0.21, 1.3)';
    splitTextEl.style.transform = 'translate(-50%, -50%) scale(1)';

    await sleep(600);

    // Fade out
    splitTextEl.style.transition = 'opacity 0.3s';
    splitTextEl.style.opacity = '0';
    await sleep(300);
    splitTextEl.style.display = 'none';
  } else {
    // Single crack effect
    Sound.crack();
    emitBurst(center.x, center.y, color, 10);
    await sleep(300);
  }
}

async function animateRarityReveal(rarityIndex) {
  const rarity = RARITIES[rarityIndex];
  const center = getDropBoxCenter();

  // Color shift on drop box
  dropBox.style.animation = 'none';
  dropBox.style.transition = 'all 0.4s';
  dropBox.style.background = `radial-gradient(ellipse at center, ${rarity.color}, ${rarity.color}88)`;
  dropBox.style.borderColor = rarity.color;
  dropBox.style.boxShadow = `0 0 ${rarity.glowSize}px ${rarity.color}`;

  Sound.rarityReveal(rarityIndex);

  // Higher rarities get screen effects
  if (rarityIndex >= 3) {
    // Screen flash
    emitFlash(rarity.color);
    document.getElementById('app').style.animation = 'screenShake 0.4s';
    await sleep(400);
    document.getElementById('app').style.animation = '';
  } else if (rarityIndex >= 2) {
    emitBurst(center.x, center.y, rarity.color, 25);
    document.getElementById('app').style.animation = 'screenShake 0.2s';
    await sleep(200);
    document.getElementById('app').style.animation = '';
  }

  await sleep(400);
}

async function animateRewardReveal(reward, rarityIndex) {
  const rarity = RARITIES[rarityIndex];

  // Show reveal overlay
  revealOverlay.classList.add('active');

  // Rarity text slam in
  revealRarity.textContent = `★ ${rarity.name.toUpperCase()} ★`;
  revealRarity.style.color = rarity.color;
  revealRarity.style.textShadow = `0 0 20px ${rarity.color}`;
  revealRarity.style.animation = 'slamIn 0.4s forwards';

  await sleep(300);

  // Icon zoom in
  revealIcon.textContent = reward.icon;
  revealIcon.style.animation = 'zoomIn 0.3s forwards';

  Sound.rewardReveal();
  emitBurst(window.innerWidth / 2, window.innerHeight / 2, rarity.color, 25);

  await sleep(200);

  // Name + type fade in
  revealName.textContent = reward.name;
  revealName.style.animation = 'fadeInUp 0.3s forwards';

  await sleep(100);

  const typeLabel = reward.isDuplicate ? reward.originalType : reward.type;
  revealType.textContent = typeLabel;
  revealType.style.color = rarity.color;
  revealType.style.animation = 'fadeInUp 0.3s forwards';

  // Show duplicate badge if applicable
  if (reward.isDuplicate) {
    revealDuplicate.style.display = 'block';
    revealDuplicate.style.animation = 'fadeInUp 0.3s forwards';
  }

  await sleep(200);

  // Tap hint
  const tapHint = revealOverlay.querySelector('.reveal-tap-hint');
  tapHint.style.animation = 'fadeInUp 0.5s forwards';

  // Wait for tap
  await new Promise(resolve => {
    revealOverlay.addEventListener('click', function handler() {
      revealOverlay.removeEventListener('click', handler);
      resolve();
    });
  });

  // Clean up overlay
  revealOverlay.classList.remove('active');
  revealRarity.style.animation = '';
  revealRarity.style.opacity = '0';
  revealIcon.style.animation = '';
  revealIcon.style.opacity = '0';
  revealName.style.animation = '';
  revealName.style.opacity = '0';
  revealType.style.animation = '';
  revealType.style.opacity = '0';
  revealDuplicate.style.display = 'none';
  revealDuplicate.style.animation = '';
  revealDuplicate.style.opacity = '0';
  tapHint.style.animation = '';
  tapHint.style.opacity = '0';
}

function resetDropBox() {
  dropBox.style.transition = 'all 0.3s';
  dropBox.style.background = 'radial-gradient(ellipse at center, #6b3fa0 0%, #4a1f7a 40%, #2d1050 100%)';
  dropBox.style.borderColor = '#9b59b6';
  dropBox.style.boxShadow = '0 0 30px rgba(155, 89, 182, 0.4), inset 0 -5px 10px rgba(0,0,0,0.3)';
  dropBox.style.animation = '';
}

async function openDrop() {
  if (isAnimating) return;
  isAnimating = true;

  // Roll the drop
  const drop = performDrop();
  const { rarityIndex, splitCount, rewards } = drop;

  // Phase 1: Charge Up
  await animateChargeUp(rarityIndex);

  // Phase 2: Split Check
  await animateSplitCheck(splitCount, rarityIndex);

  // Phase 3 + 4: For each reward in the split, reveal rarity then reward
  for (let i = 0; i < rewards.length; i++) {
    if (i > 0) {
      // Brief pause between split reveals
      resetDropBox();
      await sleep(300);
      // Quick charge for subsequent drops
      dropBox.style.boxShadow = `0 0 ${RARITIES[rarityIndex].glowSize}px ${RARITIES[rarityIndex].color}`;
      dropBox.style.borderColor = RARITIES[rarityIndex].color;
      await sleep(200);
    }
    await animateRarityReveal(rarityIndex);
    await animateRewardReveal(rewards[i], rarityIndex);
  }

  // Record and update
  recordDrop(rarityIndex, splitCount, rewards);
  resetDropBox();
  updateUI();

  isAnimating = false;
}
```

- [ ] **Step 2: Verify animation sequence**

This can't be fully tested yet (needs `updateUI()` from next task), but verify the function exists:
- In console: `typeof openDrop` → `"function"`

- [ ] **Step 3: Commit**

```bash
git add index.html
git commit -m "feat: add 4-phase animation orchestrator for drop opening"
```

---

## Chunk 3: UI, Stats, Polish

### Task 8: UI Wiring — Event Handlers + Dynamic Updates

**Files:**
- Modify: `index.html` (append to `<script>` tag)

Wire up all event handlers (drop tap, sound toggle, stats button) and the `updateUI()` function that refreshes the DOM from state.

- [ ] **Step 1: Add UI code**

```javascript
// ============================================================
// UI
// ============================================================

function updateUI() {
  // Drop counter
  document.getElementById('dropCount').textContent = state.totalDrops;

  // Sound toggle
  document.getElementById('soundToggle').textContent = state.soundOn ? '🔊 Sound' : '🔇 Muted';

  // Recent drops
  const recentRow = document.getElementById('recentRow');
  if (state.recentDrops.length === 0) {
    recentRow.innerHTML = '<div style="color:#555; font-size:12px; padding:8px;">No drops yet — tap the box!</div>';
  } else {
    recentRow.innerHTML = state.recentDrops.map(r => {
      const color = RARITIES[r.rarityIndex].color;
      return `<div class="recent-item" style="border-color: ${color}33; background: ${color}15;">
        <div class="recent-item-icon">${r.icon}</div>
        <div class="recent-item-name" style="color: ${color};">${r.name}</div>
      </div>`;
    }).join('');
  }
}

// --- Stats Panel ---

function renderStats() {
  const content = document.getElementById('statsContent');
  const total = Math.max(state.totalDrops, 1); // avoid division by zero

  const bestDropText = state.bestDrop
    ? `${RARITIES[state.bestDrop.rarityIndex].name} ×${state.bestDrop.splitCount}`
    : 'None yet';

  const collectionCount = Object.keys(state.collection).length;
  const totalCollectibles = REWARDS.filter(r => COLLECTIBLE_TYPES.includes(r.type)).length;

  content.innerHTML = `
    <div class="stat-row"><span class="label">Total Drops</span><span class="value">${state.totalDrops}</span></div>
    <div class="stat-row"><span class="label">Best Drop</span><span class="value">${bestDropText}</span></div>
    <div class="stat-row"><span class="label">Collection</span><span class="value">${collectionCount}/${totalCollectibles}</span></div>

    <h3 style="font-size:14px; margin:16px 0 8px; color:#aaa;">Rarity Distribution</h3>
    <div class="bar-chart">
      ${RARITIES.map((r, i) => {
        const count = state.rarityCount[i];
        const pct = (count / total * 100);
        const expected = r.chance * 100;
        return `<div class="bar-row">
          <span class="bar-label">${r.name}</span>
          <div class="bar-track">
            <div class="bar-fill" style="width:${pct}%; background:${r.color};"></div>
          </div>
          <span class="bar-count">${count}</span>
        </div>`;
      }).join('')}
    </div>

    <h3 style="font-size:14px; margin:16px 0 8px; color:#aaa;">Luck Meter</h3>
    <div class="luck-meter">
      ${RARITIES.map((r, i) => {
        const actualNum = state.totalDrops > 0 ? (state.rarityCount[i] / state.totalDrops * 100) : 0;
        const expectedNum = r.chance * 100;
        const diffNum = actualNum - expectedNum;
        const diffColor = diffNum > 0.05 ? '#2ecc71' : diffNum < -0.05 ? '#e74c3c' : '#888';
        return `<div style="display:flex; justify-content:space-between; font-size:12px; padding:3px 0;">
          <span style="color:${r.color};">${r.name}</span>
          <span>${actualNum.toFixed(1)}% <span style="color:${diffColor};">(${diffNum > 0 ? '+' : ''}${diffNum.toFixed(1)}%)</span> vs ${expectedNum.toFixed(1)}%</span>
        </div>`;
      }).join('')}
    </div>

    <h3 style="font-size:14px; margin:16px 0 8px; color:#aaa;">Split Distribution</h3>
    <div class="bar-chart">
      ${SPLITS.map(s => {
        const count = state.splitCount[s.count] || 0;
        const totalSplits = Object.values(state.splitCount).reduce((a, b) => a + b, 0) || 1;
        const pct = (count / totalSplits * 100);
        return `<div class="bar-row">
          <span class="bar-label">×${s.count}</span>
          <div class="bar-track">
            <div class="bar-fill" style="width:${pct}%; background:#9b59b6;"></div>
          </div>
          <span class="bar-count">${count}</span>
        </div>`;
      }).join('')}
    </div>
  `;
}

function renderCollection() {
  const content = document.getElementById('collectionContent');
  const collectibles = REWARDS.filter(r => COLLECTIBLE_TYPES.includes(r.type));

  content.innerHTML = `
    <div class="collection-grid">
      ${collectibles.map(item => {
        const owned = state.collection[item.id];
        const rarity = RARITIES[item.rarities[0]];
        return `<div class="collection-item ${owned ? '' : 'locked'}"
          style="border-color: ${owned ? rarity.color + '44' : 'rgba(255,255,255,0.1)'};"
          ${owned ? `onclick="alert('${item.name}\\n${item.type}\\n${rarity.name}\\nDrop #${owned.dropNumber}')"` : ''}>
          <div class="collection-item-icon">${owned ? item.icon : '❓'}</div>
          <div class="collection-item-name">${owned ? item.name : '???'}</div>
        </div>`;
      }).join('')}
    </div>
  `;
}

// --- Event Handlers ---

dropBox.addEventListener('click', () => {
  Sound.uiClick();
  openDrop();
});

document.getElementById('soundToggle').addEventListener('click', () => {
  state.soundOn = !state.soundOn;
  saveState();
  updateUI();
});

document.getElementById('statsBtn').addEventListener('click', () => {
  Sound.uiClick();
  renderStats();
  renderCollection();
  document.getElementById('statsModal').classList.add('active');
});

document.getElementById('statsClose').addEventListener('click', () => {
  document.getElementById('statsModal').classList.remove('active');
});

// Tab switching
document.querySelectorAll('.tab').forEach(tab => {
  tab.addEventListener('click', () => {
    document.querySelectorAll('.tab').forEach(t => t.classList.remove('active'));
    tab.classList.add('active');
    const target = tab.dataset.tab;
    document.getElementById('statsContent').style.display = target === 'stats' ? 'block' : 'none';
    document.getElementById('collectionContent').style.display = target === 'collection' ? 'block' : 'none';
  });
});

// Reset button
document.getElementById('resetBtn').addEventListener('click', () => {
  if (confirm('Reset all data? This cannot be undone.')) {
    resetState();
    renderStats();
    renderCollection();
  }
});

// Close modal on overlay click
document.getElementById('statsModal').addEventListener('click', (e) => {
  if (e.target === document.getElementById('statsModal')) {
    document.getElementById('statsModal').classList.remove('active');
  }
});

// --- Init ---
updateUI();
```

- [ ] **Step 2: Full end-to-end test in browser**

Open `index.html` in a mobile-sized browser window (Chrome DevTools → toggle device toolbar → select a phone).

Verify:
1. Page loads with centered drop box, "Drops Opened: 0", empty recent drops
2. Tap drop box → charge-up animation with particles + sound → split check → rarity reveal with color + slam text → reward reveal with item → tap to dismiss
3. After dismiss: drop counter increments, recent drops shows the item
4. Tap sound toggle → shows "Muted", sounds stop
5. Tap stats → modal opens with rarity distribution, luck meter, collection tab
6. Collection tab shows locked items as "???"
7. Open several more drops → stats update, collection items unlock
8. Reset button → confirmation dialog → all data cleared

- [ ] **Step 3: Commit**

```bash
git add index.html
git commit -m "feat: add UI wiring, stats panel, collection view, and event handlers"
```

---

### Task 9: Polish — Edge Cases, Mobile UX, Final Touches

**Files:**
- Modify: `index.html`

Final polish pass: disable drop box during animation (visual feedback), add `.gitignore`, ensure smooth mobile experience.

- [ ] **Step 1: Add animation-lock visual feedback to CSS**

In the `<style>` block, add after the `.drop-box:active` rule:

```css
.drop-box.disabled {
  pointer-events: none;
  opacity: 0.7;
}
```

- [ ] **Step 2: Update the openDrop function to toggle disabled state**

At the start of `openDrop()` after `isAnimating = true;`, add:
```javascript
dropBox.classList.add('disabled');
```

At the end of `openDrop()` before `isAnimating = false;`, add:
```javascript
dropBox.classList.remove('disabled');
```

- [ ] **Step 3: Create .gitignore**

Create `.gitignore` in the project root:

```
.superpowers/
.DS_Store
```

- [ ] **Step 4: Final browser test**

Test the complete experience on mobile viewport:
1. Rapid tapping during animation → drop box is grayed out, no double-opens
2. Open 20+ drops → recent row scrolls, stats accumulate correctly
3. Get a duplicate → shows "DUPLICATE — Fallback Reward" badge
4. Sound toggle persists across page reload
5. Stats persist across page reload
6. Reset clears everything
7. Page works when opened directly as a file (no server needed)

- [ ] **Step 5: Commit everything**

```bash
git add index.html .gitignore
git commit -m "feat: add polish — animation lock, gitignore, final mobile UX"
```

---

## Execution Handoff

All 9 tasks build the complete Chaos Drops simulator incrementally. Each task produces a testable state. The final result is a single `index.html` that can be opened directly in any browser or deployed to any static host.
