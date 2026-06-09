# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# zero2-keymap — project context

A published guide + interactive config generator that turns an **8BitDo Zero 2** into a
one-handed **Wispr Flow + Claude Code** controller on macOS, using **Karabiner-Elements**.

## Development workflow
- **No build, lint, or test step** — plain static files. Preview the app with
  `open zero2-keymapper.html` (file:// works; nothing requires a server).
- **Deploy = push to `main`** — GitHub Pages (repo `R005ter/zero2-keymap`) serves the branch
  directly. Note the local directory name (`Zero2_Config`) differs from the repo name.
- To verify a generated config, import it via Karabiner-Elements → Complex Modifications;
  rules only fire for the Zero 2 (`device_if`), so a real keyboard is never affected.

## Status: shipped & live
- Site:        https://r005ter.github.io/zero2-keymap/
- App:         https://r005ter.github.io/zero2-keymap/zero2-keymapper.html
- Raw config:  https://raw.githubusercontent.com/r005ter/zero2-keymap/main/8bitdo-zero2.json
- One-click import (paste in browser address bar — custom URI scheme, can't be a clickable MD link):
  `karabiner://karabiner/assets/complex_modifications/import?url=https%3A%2F%2Fraw.githubusercontent.com%2Fr005ter%2Fzero2-keymap%2Fmain%2F8bitdo-zero2.json`

## Files
- `README.md` — the guide (Pages home + repo readme)
- `zero2-keymapper.html` — interactive config generator (self-contained, runs on Pages)
- `cheatsheet.html` — printable one-page desk reference (US Letter landscape; static, hand-maintained — does NOT read from the generator). Update it by hand if the default layout changes.
- `8bitdo-zero2.json` — default Karabiner complex_modifications config
- `zero2-keymap.png` — keymap image embedded in README
- `zero2-keymap.svg` — source for the PNG
- `CLAUDE.md` — this file

## How it works
Zero 2 runs in **keyboard mode** (pair: power off → hold **Start+R** → hold **Select ~3s** →
connect "8BitDo Zero 2 gamepad"). Each button emits a letter; Karabiner remaps those letters —
**scoped to the device via `device_if`** — into the keys/shortcuts Wispr and the terminal expect.
macOS pops the Keyboard Setup Assistant on pair; dismiss it (it's a real keyboard).

## Hardware facts
- Device IDs (keyboard mode, **model-level → shared by all Zero 2 units, not sensitive**):
  `vendor_id 11720` (0x2DC8 = 8BitDo), `product_id 36888`.
- The per-unit **serial is intentionally never used or committed** — rules key only off vendor/product.
- Button → emitted key:
  - D-pad: up=`e`, down=`f`, left=`d`, right=`c`
  - Face:  top=`i`, left=`j`, right=`h`, bottom=`g`
  - Side:  upper/index=`k`, lower/ring=`m`
  - Start/Select: spare/unused

## Default layout (what the shipped JSON encodes)
| key | function |
|-----|----------|
| e/f/d/c | arrows up/down/left/right |
| k (index, tap) | Fn+Space → Wispr hands-free toggle (while m held: Return → ⌘Enter send) |
| m (ring) | tap = Return · hold = Cmd layer (Cmd held; D-pad ⇄ apps, k = ⌘Enter) |
| j (left face, hold) | Ctrl+Opt → Wispr push-to-talk |
| g | `open -a Terminal` |
| h | Cmd+\` → cycle terminal windows |
| i | Cmd+Tab → cycle apps (two-app toggle; true cycling = hold m) |

Wispr Flow shortcuts must match: push-to-talk = **Ctrl+Opt**, hands-free = **Fn+Space**.

## App architecture (`zero2-keymapper.html`)
- Vanilla HTML/CSS/JS, **single file, no framework, no build, no CDN except Google Fonts**.
- **No localStorage/sessionStorage** (must also work inside sandboxed iframes).
- State object `state` = {vendor, product, terminal, launchCmd, map}. Plus `BUTTONS`, `FUNCS`, `DEFAULTS`.
- `buildConfig()` emits one Karabiner rule. `toEvents(fn)` maps a function id → `to` events.
  `launchShell()` builds the terminal-launch command (`open -a`, or an osascript do-script /
  iTerm variant when `launchCmd` is set).
- **App-switcher layer**: two layer functions — `app_layer` (hold only) and `enter_app_layer`
  (default on m; adds `to_if_alone` = Return, so tap = Enter, hold = layer). Both hold Cmd lazily +
  set variable `zero2_app_switcher`; the D-pad left/right buttons get variable-gated Shift+Tab/Tab
  manipulators emitted BEFORE the normal arrow ones (Karabiner = first-match-wins). The **hands-free
  button gets the same treatment** — a variable-gated `return_or_enter` manipulator emitted before its
  plain Fn+Space one, so while the layer holds Cmd lazily, tapping it lands as **⌘Enter** (send). A
  tap-to-latch switcher is impossible: macOS commits the instant Cmd releases, so the layer must sit on
  a finger-held button (m/ring) while the thumb works the D-pad.
- **Every `from` carries `modifiers:{optional:['any']}`** (`fromKey()`): Karabiner counts its own
  rule-held output modifiers (lazy Cmd, PTT's Ctrl+Opt) toward from-matching, and a `from` without
  `modifiers` only matches when none are active — omitting it breaks every button mid-layer/mid-PTT.
- Live SVG diagram updates label text nodes by id `t-<buttonid>`.
- `openImport()` navigates to the `karabiner://` link; `copyImport()` copies it.

## Conventions / constraints
- Keep everything **dependency-free and self-contained** — it has to run from static GitHub Pages.
- README embeds **PNG**, not SVG (GitHub renders SVG inconsistently). Regenerate the PNG from the SVG:
  `/opt/homebrew/bin/python3.13 -c "import cairosvg;cairosvg.svg2png(url='zero2-keymap.svg',write_to='zero2-keymap.png',output_width=960,output_height=1120)"`
  (must be **homebrew** python — system python can't load homebrew's libcairo under SIP). The SVG
  font stacks list **Menlo/Verdana first**: cairosvg doesn't fall through a missing first family,
  and DejaVu isn't installed on this machine — DejaVu-first renders tofu for ↑↓←→▸.
- The import-link URL param is **percent-encoded**.
- License: MIT (stated in README; no `LICENSE` file yet).

## Open items / likely next steps
- Add `LICENSE` (MIT) and a `.gitignore` (`.DS_Store`).
- Verify the one-click import end-to-end on a clean machine (granting Input Monitoring + pairing).
- Possible enhancements: "reset to defaults" button, dark mode for the app, more assignable
  functions, additional one-press macros.
- If a unit's firmware reports different IDs, README already points users to EventViewer.
