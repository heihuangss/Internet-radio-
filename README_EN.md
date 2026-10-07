# PANDA 6519 · Retro Online Radio

![License: MIT](https://img.shields.io/badge/License-MIT-green.svg)
![Single File](https://img.shields.io/badge/Single%20File-No%20Build-blue.svg)
![HLS](https://img.shields.io/badge/HLS-hls.js%201.5.17%20inlined-orange.svg)
![No Backend](https://img.shields.io/badge/Backend-None-lightgrey.svg)

> A retro-styled online radio built with plain HTML / CSS / JavaScript.
> **Single file, zero build step, zero backend** — just double-click and listen.
> Ships with a VU meter, live spectrum analyzer, cassette deck, FM tuning pointer,
> and a plain-text station book called the *Spectrum Sheet*.

[简体中文](./README.md) | English

---

## ✨ Features

### 🎛 The Cabinet
- Wood-grain body with a top carry handle, rounded corners, inner shadows and ambient glow
- **VU meter**: vector SVG scale (-20 ‖ -10 · -7 · -5 · -3 -2 -1 0 +1 +2 +3 · +5); the needle is driven by
  level computed from **time-domain** audio samples, with a PEAK hold indicator
- **Live spectrum**: canvas-drawn frequency bars, rebuilt at the current `devicePixelRatio` so it stays crisp on HiDPI
- **Cassette deck**: reels spin and the tape counter runs (100 → 0) while playing; click the door to play / pause
- **E-ink clock** and a **tuning dial window** (FM 88 – 108 MHz pointer plus station / status readout)
- TUNING knob (opens the station list), VOL knob (click to mute) and a volume slider

### 📻 Playback Core
- **HLS streams**: inlined hls.js 1.5.17 (MSE); Safari / iOS uses native HLS
- **Direct streams**: `.mp3` / `.aac` and other native formats go straight to `<audio>`
- **Automatic source fallback**: one station can carry N candidate URLs; failures roll to the next one,
  and only a total failure shows `NO SIGNAL`
- **Health watchdog**: stalls and errors during playback trigger an automatic source switch
- **Connection memory**: the last working URL is promoted to the front of the queue and persisted

### 📋 Spectrum Sheet v1 (Local Station Book)
- Station sources are maintained as **plain text**: one station per line, candidate URLs indented two spaces
- Supports groups, `频率：` (frequency), `别名：` (aliases), the one-liner form `Name|url1|url2`,
  and shorthand `@freq`, `=alias`, `#group`, `//comment`
- Built-in editor with **Apply / Format / Silent Probe / Import .txt / Export .txt / Clear**, plus a persistent syntax cheat-sheet
- `Tab` indents, `Esc` closes the editor

### 🔎 Auto Scan
- 6 parallel probes, 5 s per-URL timeout, 25 s overall deadline — it never spins forever
- Two-stage verdict: a `fetch` pre-check first (HLS is validated against `#EXTM3U`), then a
  **silent playback** fallback (fully `muted`, not a single sound leaks out)
- Scan order: last working station → local stations → national networks → the rest;
  dead URLs are blocked for the session and unblocked on reload
- Result cached for 7 days, so the next visit connects instantly

### 📡 Automatic FM Frequency Tracking
Flip the toggle at the top-right of the VU meter and the tuning pointer snaps to the station's real FM frequency.
Resolution order:

| # | Source | Accuracy |
| --- | --- | --- |
| 0 | Hand-written `频率：` in the Spectrum Sheet | exact |
| 1 | Digits inside the station name (103.9 / 971 → 97.1) | exact |
| 2 | Built-in city frequency table (incl. a nationwide MusicRadio city table) | exact |
| 3 | Radio Browser online lookup | exact |
| 4 | Stable hash estimate from the station name | estimated (marked `≈`) |

The city comes from browser Geolocation + OpenStreetMap Nominatim reverse geocoding, with a default fallback.
Frequency range is 87.0 – 108.0 MHz; the pointer stays locked inside the 20% – 80% band of the dial.

### 🎨 Five Skins
`Classic Wood` · `Midnight Vinyl` · `Mint Cream` · `Rosewood` · `Neon` — the choice is stored locally and survives reloads.

### 🖥 Fullscreen & HiDPI
- **FLIP + Web Animations API** zoom transition: it animates the real DOM, so text renders at vector clarity
  instead of "blurry first, sharp later"
- Fills a 16:9 viewport, scaling the cabinet proportionally and centering it
- Tiered zoom for 2K / 4K (breakpoints at 1440 / 1800 / 2200 / 2800 / 3400px → 1.06 – 1.58),
  disabled on short viewports (≤760px tall)
- Watches `devicePixelRatio` changes and rebuilds the spectrum bitmap when the window moves to a 4K screen
- The toolbar auto-hides in fullscreen and fades in when the pointer nears the top edge

### ♿ Motion & Accessibility
- One shared motion token set (`--dur-1…5`, `--ease-std/dec/acc/expo`), all decelerate-family, restrained and bounce-free
- Honors the system "reduce motion" setting (`prefers-reduced-motion`)
- Uniform stroke-width SVG icons, `:focus-visible` focus rings and `aria-label` on controls

---

## 🚀 Quick Start

**Option 1 — open it directly**

```bash
git clone https://github.com/<your-username>/panda-6519-web-radio.git
cd panda-6519-web-radio
# double-click Internet radio.html
```


**Option 3 — GitHub Pages**

Repo Settings → Pages → pick the branch root. No build step required.

---

## 📻 Listen in Three Steps

1. Click **📋 Spectrum Sheet** in the top-right corner
2. Paste your station sources in the format below and click **Apply**
3. Hit ▶️, or click **Auto Scan** to let it find the first source that actually plays

> A fresh install ships with no stations — deliberately. Stream URLs belong to the broadcasters,
> and public URLs break without notice, so the project ships the **format**, not the **content**.

---

## 📋 Spectrum Sheet v1 Syntax

```
// Spectrum Sheet v1 · station sources

Group: CNR
CNR Voice of China
  https://example.com/live/zgzs.m3u8
  https://example.com/live/zgzs.mp3
  Freq: 106.1
  Alias: Voice of China, CNR-1

Shanghai Traffic Radio
  https://example.com/live/jtgb.m3u8
  Freq: 105.7
```

A single line works too:

```
CNR Voice of China|https://example.com/live/zgzs.m3u8|https://example.com/live/zgzs.mp3
```

| Syntax | Meaning | Shorthand |
| --- | --- | --- |
| `Group: xxx` | Group heading (unindented) | `# xxx` |
| Unindented line | Station name; every indented line below belongs to it | — |
| Two-space indent + `http(s)://` | Candidate source, several allowed, tried in order | — |
| `Freq: 103.9` | Pin the tuning pointer to 103.9 | `@103.9` |
| `Alias: a, b` | Short names become searchable | `=a, b` |
| `Note: xxx` | Comment, ignored | `// xxx` |

A protocol-less URL (`live.xxx.com/xxx.m3u8`) is auto-prefixed with `http://`.
The search box matches station names, aliases and frequency numbers.

---

## ⌨️ Keyboard Shortcuts

| Key | Action |
| --- | --- |
| `F11` | Enter / exit fullscreen |
| `Ctrl + Shift + F` (macOS `⌘ + Shift + F`) | Enter / exit fullscreen |
| `Esc` | Close the side panel / confirm dialog; exits fullscreen via the browser |
| `Enter` | Run search while focused in the search box |

---

## 💾 Local Storage

Everything stays in your own browser — nothing is ever uploaded:

| Key | Contents |
| --- | --- |
| `panda6519_stationpack_v1` | Spectrum Sheet text (station book) |
| `panda6519_besturl_v1` | Last working URL per station |
| `panda6519_laststation_v1` | Last listened station (cached 7 days) |
| `panda_skin` | Active skin |

---

## 🌐 Browser Support

| Browser | Version | Notes |
| --- | --- | --- |
| Chrome / Edge | 90+ | Everything works |
| Firefox | 90+ | Everything works |
| Safari | 15+ | HLS plays natively, rest identical |
| Mobile browsers | Modern | Responsive layout, touch and haptic feedback |

Requires `MediaSource Extensions` (HLS), `Web Audio API` (meter / spectrum), `Fetch` and `localStorage`.

---

## ❓ FAQ

**Why is it empty on first open?**
No station sources are bundled. Open **📋 Spectrum Sheet**, add your own sources, then click **Apply**.

**A source plays elsewhere but reports CORS / network failure here?**
The `<audio>` element carries `crossorigin="anonymous"`, so sources without permissive CORS cannot load.
That is exactly why Auto Scan has a **silent playback** fallback — or just use a CORS-friendly URL.

**Why does the FM readout show `≈`?**
None of the four exact sources matched, so a stable hash estimate is used. Add `Freq:` to the Spectrum Sheet to pin it.

**Does fullscreen get blurry?**
It shouldn't: the zoom animates the real DOM via FLIP. When the window is dragged onto a 4K screen the page
listens for DPR changes and rebuilds the spectrum bitmap; reload if anything still looks off.

**The toolbar vanished in fullscreen.**
By design. Move the pointer toward the top edge (or tap) to bring it back, along with an exit hint.

---

## 🤝 Contributing

Issues and PRs are welcome. Easy ways to start:

- **Add station sources**: submit a `.txt` of URLs you have verified, formatted per the Spectrum Sheet
- **Extend the FM table**: add city frequencies to `CNR_FM_MAP`
- **Design a skin**: add a CSS variable block under `body[data-skin="xxx"]` and one entry to the `SKINS` array
- **Refine interaction**: reuse the existing duration / easing tokens to keep the motion restrained

Before submitting: the single file must still run standalone, no external CDN dependencies,
and no third-party stream URLs bundled in.

---

## ⚖️ Disclaimer

- The "PANDA ELECTRIC CO. / MODEL 6519" nameplate is a tribute to vintage radio design and is
  not associated with any real manufacturer, brand or product.
- Stream URLs remain the property of their broadcasters. This project bundles, hosts and relays nothing;
  it is intended for personal listening and technical demonstration only.
- FM data comes from public sources and the community Radio Browser database; city lookup uses
  OpenStreetMap Nominatim. Always trust your local broadcast for the actual frequency.

---

## 📄 License

[MIT License](./LICENSE) © 2026 PANDA 6519 Web Radio Contributors
