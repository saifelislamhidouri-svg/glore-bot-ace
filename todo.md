# GLORE ACE ᵛˣ — Boost In-Game Activity + Glory Farm (Phase E)

## Phase E1 — More movement / less AFK (Clash Squad Ranked)
- [x] Lower idle chance + raise walk/run/sprint chances (constant tuning)
- [x] Faster movement tick + higher speed multiplier + wider radius
- [x] Add path jitter so movement isn't a perfect loop
- [x] More aggressive anti-AFK fire cadence
- [x] Add periodic combat "engagement burst" worker (rapid fire spray)
- [x] Start/stop all workers reliably in play_game; count packets

## Phase E2 — Farm a LOT more Glory
- [x] Raise base Glory per CS match + ranked multiplier
- [x] Add kill/performance bonus + win-streak bonus in add_glory
- [x] Persist streak + best streak; expose to dashboard

## Phase E3 — Dashboard & branding
- [x] Expose movement/fire/glory-streak stats in /api/stats + /health
- [x] index.html: live Activity stat box + Glory streak sub
- [x] Keep mode badge + Glory card in sync

## Phase E4 — Verify & deliver
- [x] Compile-check all Python modules
- [x] Boot bot + dashboard, hit every endpoint
- [x] DOM checks + screenshot
- [x] Update README + package deliverables
