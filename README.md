# Handoff: Embedded Systems SRAM AES-256 Key-Recovery Lab

## Overview
An interactive, browser-based teaching lab for PhD-level embedded-security students. The learner is given a synthetic 8 KiB SRAM image dumped from a "seized" IoT camera (Cortex-M4) and must recover the live **AES-256** key that the firmware pins in RAM, using a toolkit of exploration scripts — then answer a design-it-out debrief on where the key *should* have lived.

The pedagogical point: entropy and statistics **locate** high-randomness regions but **cannot** tell a key apart from compressed data, a nonce, or a hash. Only reconstructing the AES-256 **key schedule** proves a candidate is a real key. The image deliberately contains several equally-random decoys so the answer is never obvious.

The lab is fully self-contained: every script runs real math against a real (deterministically generated) byte image — Shannon entropy, chi-square, and a genuine AES-256 key expansion. There is no server; the "answer" is derived from the image, not hard-coded.

## About the Design Files
The file in this bundle (`SRAM Key Recovery Lab.dc.html`) is a **design + behavior reference created in HTML** — a working prototype demonstrating the intended look, content, and interaction model. It is **not** production code to ship as-is. The task is to **recreate this lab in the target codebase's environment** (e.g. a React/Next courseware app, a Vue SPA, an LMS module, or a Python/Jupyter-backed lab runner) using that environment's established patterns, component library, and state management. If no environment exists yet, choose the most appropriate framework for an interactive terminal-style lab and implement there.

Note: the prototype is authored as a "Design Component" (a single `.dc.html` with an inline template + a `class Component` logic block). That wrapper is specific to the prototyping tool — **port the logic, not the wrapper.** All meaningful logic (image generation, the scripts, the AES key schedule, scoring) lives in the `class Component` block and is plain framework-agnostic JavaScript you can lift almost verbatim.

## Fidelity
**High-fidelity (hifi).** Final colors, typography, spacing, copy, and interactions are all intended as shown. Recreate the UI faithfully using the codebase's existing libraries. The one thing to preserve exactly is the **script logic and the crypto correctness** — the entropy formula, the chi-square test, and especially the AES-256 key expansion must behave identically or the lab breaks (candidates won't disambiguate, or a correct answer will be rejected).

## Screens / Views
There is one page presenting **two interaction models side by side** over the *same* underlying engine. In production you will likely ship **one** of these, chosen by product preference. Both must be offered as design references.

### Shared page shell
- **Header** (top, `padding:30px 44px 0`): title "Embedded Systems Lab · Live AES-256 Key Recovery" in `#b6ff8f`, 22px, weight 700. Sub-line in `#5f8a5f`, 13px, max-width 1100px: the case framing + two anchor links (`#1a`, `#1b`) in `#5fd0e0`.
- **Instructions panel** (below header): bordered card (`1px solid #1e2c1e`, radius 10, bg `#0b0f0b`, padding `20px 24px`). Heading "HOW TO RUN THIS LAB" in `#9fe6ff`, 14px, weight 700, letter-spacing .08em. Three columns via `display:flex; gap:40px; flex-wrap:wrap`:
  - **1a · SIMULATED SHELL** — how to type commands, `↑/↓` history, `help`, `cat notes.txt`, `man <tool>`, clicking mission steps.
  - **1b · GUIDED FIELD KIT** — click tools, work top row first, set off/len, submit.
  - **YOUR OBJECTIVE** — recover the 32-byte AES-256 key and submit it; ground rules that the answer is unlabelled and most random regions are decoys; read each tool's note; mission steps nudge without answering.
  Column section labels are `#e8b04b`, 12px, weight 700. Body text `#8fb98f`, 12.5px, line-height 1.8–1.85. Inline command tokens are `#b6ff8f`; offsets/keys are `#5fd0e0`; filenames are `#cfeecf`.
- **Two lab frames** in a `display:flex; gap:44px; flex-wrap:wrap` row, each 1380px wide.

### View 1a — Simulated Shell
- **Purpose**: learner types real commands into a terminal and reads real output.
- **Layout**: 1380×860 rounded card. macOS-style title bar (three traffic-light dots `#ff5f56`/`#ffbd2e`/`#27c93f`, then `analyst@lab — sram-recovery — case-8842` in `#6f9a6f` 12px). Body is a flex row:
  - **Left sidebar** 300px, border-right `1px solid #172417`, bg `#0b0f0b`, padding `18px 16px`. "MISSION STEPS" heading `#9fe6ff` 12px 700 letter-spacing .14em. Seven clickable steps; each row is `display:flex; gap:9px`, hover bg `#101610`. Step number badge: min-width 22px, centered, bg `#101610`→`#1c3a1c` when done, color `#557a55`→`#b6ff8f` when done, border `1px solid #22331f`→`#2f5a2f`, radius 5, 11px, weight 700. Label `#82a082`→`#d4f2d4` when done, 12px. Below: a brief note block, border-top `1px solid #1a271a`.
  - **Terminal pane** flex:1, bg `#080b08`, padding `16px 18px`, 13px, line-height 1.55, overflow auto. Output lines are pre-wrapped colored `<div>`s. Prompt line: `analyst@lab:~/case-8842$` in `#c9a15a`, followed by a borderless transparent `<input>` (color `#b6ff8f`, caret `#79d979`). Clicking anywhere in the pane focuses the input.

### View 1b — Guided Field Kit
- **Purpose**: same engine, no typing — click a tool, read its note, interpret output.
- **Layout**: 1380×860 rounded card, same title bar style (`field-kit — case-8842`). Body flex row:
  - **Left sidebar**: identical MISSION STEPS pattern as 1a (separate completion state).
  - **Right column** (flex:1, bg `#080b08`, flex-direction column):
    - **Tool row 1** (`padding:12px 16px`, border-bottom `1px solid #172417`, flex-wrap, gap 8): two primary buttons "► trigger crypto" and "▼ SWD halt + dump" (border `1px solid #2a3f24`, bg `#0e170e`, color `#a7e6a7`, radius 6, 12px 600, hover bg `#152015`); a vertical divider; three secondary buttons "strings", "hexdump", "entropy scan" (border `1px solid #24361f`, bg `#0d150d`, color `#9fdf9f`, hover `#132013`).
    - **Tool row 2** (`padding:11px 16px`, border-bottom): label "region →", an "off" text input (width 118) and a "len" input (width 72) — inputs styled `.fld` (bg `#0a0f0a`, border `1px solid #24361f`, color `#cfeecf`, radius 5, padding `6px 8px`, focus border `#3a5a34`); then buttons "histogram / χ²", "keyschedule" (emphasized: border `1px solid #3a5a34`, bg `#111c0e`, color `#c7f0b8`, 600), and "carve".
    - **Output log** (flex:1, overflow auto, padding `16px 18px`, 13px, line-height 1.55) — same colored pre-wrapped lines.
    - **Submit bar** (`padding:12px 16px`, border-top, bg `#0a0e0a`): label "submit key" `#c9a15a`, a flex:1 `.fld` input with placeholder "64 hex chars (32 bytes)", and a "submit ✓" button (border `1px solid #3a5a34`, bg `#16240f`, color `#b6ff8f`, 700, hover `#1d3013`).

## Interactions & Behavior

### The mission flow (both views)
1. **Trigger crypto** (`target trigger-crypto` / "► trigger crypto"): sets an internal `resident` flag. Prints a target boot log ending "AES-256 key is now LIVE in RAM." **Required before dumping** or the key won't be present.
2. **Halt & dump** (`swd-dump` / "▼ SWD halt + dump"): assigns the working image. If the crypto op ran, uses the full image (`dump`); otherwise uses `dumpNoKey` (schedule region zeroed) and prints a **warning** that the key isn't resident. This is a deliberate trap — dumping first yields nothing.
3. **Survey**: `strings` (printable ASCII runs ≥4 with offsets; reveals config/URLs/an ASCII "token" that is a decoy — a raw key never appears here) and `hexdump --start 0xOFF --len N`.
4. **Entropy** (`entropy [--window 32]`): sliding-window Shannon entropy rendered as an ASCII heat-strip (glyph ramp `" .:-=+*oO#%@"`), then a list of **saturated-entropy candidate regions**. Deliberately returns several (key, zlib chunk, nonce, hash).
5. **Disambiguate**: `histogram --start 0xOFF --len N` (distribution + chi-square; verdict text explains stats can't separate a key from compressed data) and `keyschedule --offset 0xOFF` — **the disambiguator**: only the real key verifies.
6. **Carve & submit**: `carve --start 0xOFF --len 32` prints hex; `submit <64 hex>` scores it.
7. **Debrief** (`debrief`): prints the design-it-out mitigations + a reflection prompt.

Shell also supports: `help`, `ls`, `cat <file>` (incl. `notes.txt`, `README`, `tools/*.py` — reading a tool's source explains what it does and its limits), `man <tool>`, `clear`, `whoami`, `pwd`, `openocd`, `gdb`, command history via `↑/↓`. Read-tools invoked before a dump exists return a "dump not found — halt and dump first" warning.

### Terminal behavior
- **Enter** runs the current input; the command echoes as a `#8fdb8f` prompt line, output appends below.
- **↑ / ↓** cycle command history (stored newest-first, capped 60).
- Output buffer capped at 1600 lines (`.slice(-1600)`).
- On every update, scroll the pane to bottom (`scrollTop = scrollHeight` in `componentDidUpdate`). **Do not use `scrollIntoView`.**
- Mission-step click prints a hint block for that stage (does not reveal the answer).
- Step badges light up as the learner completes stages, driven by flags: `trig, dump, survey, scan, conf, solved`.

### Submit scoring
- Normalize to lowercase hex, strip non-hex. Require exactly 64 chars (else error stating the count).
- Exact match → an "ACCESS GRANTED" banner + a "why this worked" explanation + a pointer to `debrief`.
- Otherwise → "INCORRECT — N/32 bytes match" (per-byte comparison), nudging to re-check which block passed keyschedule.

### keyschedule verdict
- Reads 32 bytes at `--offset` as a candidate master key, runs AES-256 expansion, compares the resulting 208 bytes to the following bytes in the image.
- All match → "AES-256 key schedule VERIFIED @ 0xOFF" (`confirmed:true`).
- First mismatch → "INVALID", reports the diverging round-key word index and byte offset, and suggests it's likely compressed data / nonce / hash.

## State Management
Per view (the two views keep independent state):
- `aLines` / `bLines`: array of `{k, t}` output records (`k` = semantic kind → color, `t` = text). Rendered by mapping `k` to a style.
- `aInput` / `bOffset` / `bLen` / `bSubmit`: controlled input values.
- `aFlags` / `bFlags`: `{trig, dump, survey, scan, conf, solved}` booleans for step-completion UI.
- `hist` / `hi`: command history array + cursor (shell only).
- Non-reactive engine fields: `dump` (Uint8Array 8192), `dumpNoKey`, `answerHex` (derived), `imgA`/`imgB` (the dumped working image once captured), `trigA`/`trigB` (crypto-triggered flags).

No data fetching. Everything is computed client-side. The image is generated once at construction from a **seeded PRNG** (seed `0x8842BEEF`, a mulberry32-style generator) so every learner gets the same reproducible dump and the same key.

## The engine (port this carefully)
Key pieces in the `class Component` logic block — reproduce faithfully:

- **`init()`** — builds the 8192-byte image deterministically: ARM vector table @0x00; ASCII config/`.data` strings @0x100–0x1FF (incl. a decoy `prov_api_token`); `.bss` counters @0x300; a **zlib-headed compressed OTA chunk @0x600–0x8FF** (decoy, `78 9c` header + random); heap structs @0x900; a **32-byte TLS-random nonce @0x980** (decoy); the **AES-256 key schedule @0xA00** (240 bytes — first 32 are the master key, i.e. the answer); a stack region @0xAF0–0xFFF (return addresses, stack pointers, a repeated canary `0xE2DE00A5`, low-entropy garbage); a **32-byte SHA-256-style digest @0xE40** (decoy). `answerHex` = hex of the 32 bytes at 0xA00. `dumpNoKey` = a copy with 0xA00–0xAEF zeroed.
- **`keyExpand(key)`** — real AES-256 key expansion (Nk=8, 60 words, 240 bytes) using the standard S-box + RCON (both included in the file). Used both to *build* the schedule in the image and to *validate* candidates. **These must be the same function** so a carved key re-expands to the resident schedule.
- **Script cores** — `coreStrings`, `coreHexdump`, `coreEntropy`, `coreHistogram`, `coreKS`, `coreCarve`, `coreSubmit`. All pure functions of the image + args returning line arrays.
- **Tool "source" files** — `buildFiles()` returns the text of `notes.txt`, `README`, and `tools/*.py` (educational pseudo-source explaining each tool's method and limits). Learners can `cat` these.

### Correctness notes / gotchas
- **Entropy scaling**: a window of `win` bytes can hold at most `win` distinct symbols, so max achievable Shannon entropy is `log2(win)`, not 8. The heat-map and the candidate threshold are scaled to `Hmax = log2(win)` with threshold `Hmax − 0.45`. If you naively threshold at "≥7.2 bits" over a 32-byte window, **no region ever qualifies** — this was a real bug; keep the window-relative scaling.
- **Histogram small-sample guard**: for `n < 64` bytes, print a "too small for a reliable distribution test" verdict rather than a random/structured claim; otherwise compare `H` against `log2(min(256,n)) − 0.5`.
- Strings in the logic block are plain text — **do not** put HTML entities like `&amp;` in JS string literals meant for text nodes (they'll render literally). HTML entities belong only in the markup/template.

## Design Tokens
Colors:
- Backgrounds: page `#070907`; card `#0b0f0b`; terminal `#080b08`; title bar `#0d130d`; submit bar `#0a0e0a`; sidebar note border `#1a271a`.
- Borders: frame `#1e2c1e`; inner divider `#172417`; input `#24361f` (focus `#3a5a34`); tool-primary `#2a3f24`.
- Text greens: bright `#b6ff8f`; output `#79d979`; command echo `#8fdb8f`; body `#8fb98f`; dim `#4f7a4f` / `#5f8a5f`; filename `#cfeecf`; done-label `#d4f2d4`.
- Accents: amber/prompt `#c9a15a`; warn/section `#e8b04b`; cyan links/headings `#5fd0e0` / `#9fe6ff`; error `#ff6b6b`.
- Traffic lights: `#ff5f56`, `#ffbd2e`, `#27c93f`.
- **Amber theme variant** (via `palette` tweak): cmd `#caa257`, out `#e8b04b`, ok `#ffd782`, dim `#7d6a3c`.

Typography: `'JetBrains Mono'` (Google Fonts), weights 400/500/700. Sizes: title 22, section headings 12–14, body 12.5–13, terminal 13, sub-labels 11–12. Line-heights 1.45–1.85.

Radius: cards/frames 10; buttons/inputs/badges 5–6; traffic dots 50%.

Shadows: frames `0 20px 60px rgba(0,0,0,.5)`.

Spacing: page padding `30px 44px`; frame gap 44; column gap 40; button gap 8; row padding `11–12px 16px`.

## Configurable options (exposed as props/tweaks in the prototype)
- **`palette`**: `"green"` (default) or `"amber"` — recolors terminal output kinds.
- **`hints`**: boolean (default true) — when false, strips the interpretive nudge lines from entropy/histogram output for a harder run.

## Assets
None. No images, icons, or external assets — all UI is CSS, all glyphs are Unicode/ASCII. Only external dependency is the JetBrains Mono web font (swap for the codebase's mono font if preferred; monospace is required for the hexdump and entropy strip to align).

## Files
- `SRAM Key Recovery Lab.dc.html` — the complete prototype (template + logic). The `class Component` block is the source of truth for all behavior and the engine.
