# NZ:P Touch — wrapper for Nazi Zombies: Portable (nzp.gay)

A mobile-friendly launcher for NZ:P, the fan-made Call of Duty: Zombies
demake (FTEQW WebGL build) hosted at https://nzp.gay/. Same game, plus
on-screen touch controls so it can be played on a phone.

## How it works

- `index.html` is the NZ:P team's own launcher page, kept intact
  (same `Module` setup, `begin()` / `showfiles()` flow), with a touch
  overlay added in plain HTML/CSS/JS.
- The engine (`ftewebgl.js` / `ftewebgl.wasm`) loads straight from
  https://nzp.gay/ too — the script tag points at the absolute URL and
  the .wasm resolves next to it. Only the manifest (`default.fmf`,
  local copy) is bundled here. This keeps the repo small and means the
  engine always matches the game data it loads.
- The big game data is **not** bundled: `nzp/game.pk3` (~94 MB) and
  `nzp/progs.pk3` (~491 KB) load at runtime straight from
  https://nzp.gay/ — the team serves `Access-Control-Allow-Origin: *`,
  so cross-origin fetch works. `Module.files` points at those absolute
  URLs; the engine fetches them with plain XHR.
- Touch controls dispatch the same DOM input a keyboard + mouse would:
  synthetic `KeyboardEvent` / `MouseEvent` on `window` / the canvas,
  which the engine's Emscripten/SDL listeners consume.

## Files (ready to push to the `nap-touch` repo, main branch, root)

| File | Size | Notes |
|---|---|---|
| `index.html` | ~24 KB | Launcher + touch overlay |
| `default.fmf` | 96 B | Unmodified copy from https://nzp.gay/default.fmf |
| `README.md` | this file | |

The engine (`ftewebgl.js` / `ftewebgl.wasm`) and the big game data
(`nzp/game.pk3`, `nzp/progs.pk3`) all load at runtime from
https://nzp.gay/ — nothing else is bundled here.

## Publish steps (main agent)

1. Jeremy creates the empty public repo `nap-touch` (GitHub App cannot
   create repos) and makes sure it is in the Muse GitHub App install.
2. Push these files to `main` via the Git Data API (same flow as the
   Hunters site). Only three small text files — no binaries in the repo.
3. Enable GitHub Pages from `main` / (root).
4. Verify https://nswa9002.github.io/nap-touch/ loads (200) and send
   Jeremy the link. `?touch=1` forces the touch UI on desktop (testing).

## Control bindings (authoritative source: the game's own
## `default.cfg`, first file inside nzp/game.pk3 — read via a targeted
## range request; the 94 MB file was NOT downloaded)

| Action | Default key | Engine command |
|---|---|---|
| Move forward / back / left / right | W / S / A / D | +forward / +back / +moveleft / +moveright |
| Fire | Mouse 1 | +attack |
| Aim down sights | Mouse 2 | +button8 |
| Jump | Space | impulse 10 |
| Sprint | Shift | impulse 23 |
| Swap weapon | Ctrl (also 1 / 2 / wheel) | +button4 |
| Reload | R | +button5 |
| Interact / use | E | +button7 |
| Grenade (primary) | G | +button3 |
| Melee / knife | V | +button6 |
| Secondary grenade | 4 | impulse 33 |
| Pause / menu | Esc | togglemenu |
| Scores | Tab | showscores |

"Change stance" (crouch/prone, impulse 30) has **no default key** in
`default.cfg`, so there is deliberately no stance button. Menu
bindings screen (source: nzp-team/quakec `source/menu/menu_bind.qc`)
confirms the same command mapping.

## Touch mapping implemented (2009 original — Call of Duty: Zombies iOS)

Researched from the CoD Wiki, Pocket Gamer / GameSpot / Gizmodo reviews,
GameFAQs player guides, and gameplay footage (see report in
~/workspace/research_notes/old-cod-zombies-ios-controls-20261116-0215/).

- **Left virtual stick** (fixed base, bottom-left, white ring with tick
  marks, visible even when idle): drag maps to W/A/S/D with a ~10 px
  dead zone; diagonals press two keys at once; releasing the stick
  releases all movement keys.
- **Right virtual stick** (fixed base, bottom-right, same style): drag
  to look. While the stick is held deflected it keeps turning the view
  (a 16 ms loop sends look movement proportional to the deflection).
- **Tap to fire — no FIRE button**: tap anywhere on screen (or tap the
  right stick) for one shot; **double-tap + hold** for a sustained
  automatic stream. In the menus the same tap is a click.
- **ADS**: circular crosshair button above the right stick (hold = mouse
  button 2, aim down sights).
- **Knife in the middle**: wide pill button at bottom-centre with a
  knife icon (V / melee).
- **Weapon icon, top-right**: tap to reload (R), double-tap to swap
  weapon (Ctrl) — exactly like the old game. No separate reload/swap
  buttons.
- **Grenade**: icon button on the right edge, below the weapon icon
  (G). The old game showed a grenade count next to it; this page can't
  read the game's ammo/grenade counts, so no count is shown.
- **Top row**: pause (Esc, top-left), OK (Enter, menu confirm), USE
  (E — the "do actions" button: buy weapons, open doors, repair
  barricades), JUMP (Space), RUN (hold = Shift). The 2009 game had no
  jump/sprint buttons; they stay tucked up top because NZ:P supports
  them and removing them would lose the ability entirely.
- **Menus**: tap to select, like the old game (a tap is a mouse click).
- **Aim assist**: the old game's "smart targeting" snapped the crosshair
  to zombies and couldn't be turned off. That lives inside the game
  itself — this page can't add it, and NZ:P's engine may or may not
  have its own. Unverified either way.
- **Buy/repair**: the old game showed a contextual prompt (e.g.
  "Repair Barricade" centre-screen). This page can't tell when the
  player is near something buyable, so USE is a plain button — tap it
  when the game's own prompt shows.
- Look movement carries BOTH `movementX/Y` (frame delta, for
  pointer-lock-style look) AND a moving `clientX/Y` virtual cursor (for
  absolute-position look) — covers both engine paths.

## Verification done (2026-11-15)

- Downloaded files match the expected sizes byte-for-byte
  (ftewebgl.js 165,608; ftewebgl.wasm 5,178,106; default.fmf 96).
- Both inline scripts pass `node --check`.
- All `getElementById` targets exist in the HTML.
- DOM-stub harness (`/tmp/nzp_touch_test.js`): 22/22 assertions pass —
  joystick directions/diagonals/release, look drag deltas + virtual
  cursor, tap-to-click, FIRE/ADS hold, all tap buttons with correct
  keyCodes (W 87, A 65, S 83, D 68, Space 32, Shift 16, Ctrl 17,
  R 82, E 69, G 71, V 86, Esc 27, Enter 13).

## Known uncertainties (iOS Safari — Jeremy is final judge)

- **Look without pointer lock is the big one.** iOS Safari has no
  pointer lock, so look relies on the engine processing absolute
  `mousemove` position deltas. Emscripten/SDL normally does, and the
  engine ships touch handlers (touch→mouse emulation), which suggests
  it does — but it is unverified on a real phone. If dragging the
  right side does not turn the view, this is the first thing to debug.
- **Synthetic `keyCode`**: the code sets `key`, `code`, `keyCode`,
  and `which`, and patches `keyCode`/`which` onto the event instance
  if the constructor dropped them. Emscripten SDL reads `keyCode`
  (older builds) or `code` (newer) — both are supplied.
- **`movementX/Y` in the MouseEvent init dict** is accepted by Chrome
  but likely ignored by Safari; the moving `clientX/Y` is the fallback
  there (see above).
- **Sprint (Shift → impulse 23)** is an edge-triggered impulse in the
  QC; whether holding Shift keeps sprinting or toggles it is not
  established from the source — the RUN button sends keydown on press
  and keyup on release, the standard hold mapping.
- **First-load download** is ~100 MB from nzp.gay (94 MB game.pk3 +
  engine); needs a decent connection and takes a moment.
- **Audio** needs a user gesture on iOS; the Begin tap should unlock
  it, but this is unverified.
- **Menus** are navigated by tap-to-click + the OK (Enter) button;
  if a menu needs arrow keys, those are not on the overlay.
- **Performance** of a full WebGL Quake-engine game on an iPhone is
  unverified from here.
