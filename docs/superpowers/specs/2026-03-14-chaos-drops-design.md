# Chaos Drops Simulator — Design Spec

A static HTML site that simulates the Chaos Drops mechanic from Brawl Stars. Users tap to open drops, watch animated reveals, collect real Brawl Stars items, and track their luck over time.

## Architecture

Single self-contained `index.html` file. No build tools, no external dependencies, no framework. All CSS inline, all JS inline, all sounds synthesized via Web Audio API. Deploys anywhere that serves static files (or just open the file directly).

## Visual Style

Brawl Stars-inspired: bold colors, dark purple/navy background, cartoony energy, gold accents. Mobile-first responsive layout that also works on desktop.

## Core Layout

Centered drop design:
- **Header:** Sound toggle (left), Stats button (right)
- **Title:** "CHAOS DROPS" in gold gradient, drop counter below
- **Hero:** Large drop box in center — the primary interaction target
- **Rarity bar:** Five color segments showing the rarity tiers
- **Recent drops:** Horizontally scrollable row of last 10 drops at the bottom

## Rarity System

| Rarity     | Chance | Color               | Animation Intensity                         |
|------------|--------|---------------------|---------------------------------------------|
| Super Rare | 52%    | Blue (#3498db)      | Subtle glow                                 |
| Epic       | 28%    | Purple (#9b59b6)    | Moderate glow                               |
| Mythic     | 12%    | Red (#e74c3c)       | Strong glow + shake                         |
| Legendary  | 6%     | Gold (#f39c12)      | Full screen flash + shake                   |
| Ultra      | 2%     | Hot Pink (#e91e63)  | Max everything — particles, flash, shake, special sound |

## Split Mechanic

After rarity is determined, the drop can split into multiple drops of the same rarity:

| Split | Chance |
|-------|--------|
| x1    | 70%    |
| x2    | 20%    |
| x4    | 8%     |
| x8    | 2%     |

All split drops share the same rarity. An Ultra x8 split (0.04% chance) is the jackpot.

## Opening Animation — 4 Phases

### Phase 1: Charge Up (~1s)
Drop box shakes with increasing intensity. Particles swirl inward toward the box. Glow intensifies. Rising audio tone builds anticipation.

### Phase 2: Split Check (~0.5s)
Drop cracks open. If splitting: box shatters outward, reforms as multiple smaller boxes. "x4 SPLIT!" text flashes on screen with impact sound. If no split: brief crack effect, single box remains.

### Phase 3: Rarity Reveal (~0.8s)
Box color shifts to the rarity color (blue → purple → red → gold → pink). Rarity name slams onto screen with bold text. Higher rarities get progressively more dramatic effects. Legendary and Ultra get full-screen flash + screen shake.

### Phase 4: Reward Reveal (~1s)
Box opens to show the actual reward — brawler icon, gadget, skin, coins, etc. Reward card zooms in with particle burst. Item name and type displayed below the icon. Tap anywhere to dismiss.

For splits, phases 3-4 repeat sequentially for each drop so the user experiences each reveal individually.

## Reward Pool

~50 real Brawl Stars items distributed across rarity tiers:

### Reward Types
- **Coins** — currency (all rarities, more common)
- **Power Points** — currency (all rarities, more common)
- **Credits** — currency (all rarities)
- **Bling** — premium currency (higher rarities)
- **Brawlers** — named characters (rarity-gated: e.g., Legendary brawlers only from Legendary+ drops)
- **Gadgets** — ability unlocks (Super Rare through Mythic)
- **Star Powers** — ability unlocks (Epic through Legendary)
- **Hypercharges** — ability unlocks (Mythic through Ultra)
- **Skins** — cosmetics (all rarities)
- **Buffie** — special collectible (Ultra only)

### Duplicate Protection
Items tracked in localStorage. If you pull an item you already have, you receive a fallback currency reward instead (Bling or Credits, scaled by rarity). The fallback is still displayed with a "DUPLICATE" label so you know what happened.

## Stats & History Tracking

All data persisted in localStorage.

### Data Tracked
- Total drops opened (counting each split individually)
- Rarity distribution — count per tier
- Split history — count of x1, x2, x4, x8 splits
- Best drop — highest rarity + best split combo
- Collection — which unique items have been unlocked
- Recent drops — last 10 items for the main screen display

### Stats Panel
Modal overlay accessed via header button:
- Rarity distribution bar chart (color-coded)
- Collection progress (e.g., "12/50 items unlocked")
- Luck meter — actual rates vs expected rates
- Split distribution
- Reset button with confirmation dialog

### Collection View
Grid of all possible items. Unlocked items shown in full color. Locked items grayed out with "?" silhouette. Tapping an unlocked item shows its name, rarity, and which drop number it was received on.

## Sound Design

All sounds synthesized via Web Audio API (no external audio files):

- **Charge Up** — Rising frequency sweep (200Hz → 800Hz) with rumble bass undertone
- **Split** — White noise burst (shatter), then ascending "pop" sounds for each new box
- **Rarity Reveal** — Varies by tier:
  - Super Rare: Simple two-note chime
  - Epic: Three-note ascending chord
  - Mythic: Dramatic brass-like stab
  - Legendary: Full fanfare sweep with reverb
  - Ultra: Distorted bass drop + high shimmer + longest reverb tail
- **Reward Reveal** — Satisfying "ding" with sparkle overtones
- **UI Taps** — Subtle click for buttons/navigation
- **Split Announcement** — Quick ascending blips matching the split count

Sound is on by default. Toggle in header. State persisted in localStorage.

## Technical Implementation Notes

- **RNG:** `Math.random()` with weighted selection for rarity and split rolls
- **Animations:** CSS keyframes + JS-driven Canvas overlay for particles
- **Particles:** HTML5 Canvas layer on top of the DOM for burst/swirl effects
- **Screen Shake:** CSS transform on the body/container element
- **localStorage Schema:** Single JSON object keyed as `chaosDrops` containing stats, collection, settings
- **Mobile Touch:** All interactions via `click` events (works for both tap and click)
- **Viewport:** `<meta name="viewport" content="width=device-width, initial-scale=1">` for proper mobile scaling
