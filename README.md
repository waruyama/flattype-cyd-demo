# Flattype on a Cheap Yellow Display (ESP32-2432S028R)


[<video controls src="https://github.com/waruyama/flattype-cyd-demo/blob/main/video/flattype-cyd-demo-cHtd6wyP.mp4" title="Demo of Flattype on Cheap Yellow Display"></video>
](https://github.com/user-attachments/assets/e0aefb1e-3466-4e44-a137-e85ac51900db)


## What is this?

This demo shows rendering of TrueType vector font in a very resource constained embedded environment using Flattype. Flattype is a tiny OpenType pipeline written in pure Rust (shaper, glyph-outline extractor, and anti-aliased rasterizer) that fits inside an ESP32 with **320 KB of Ram and 4 MB of flash awith no PSRAM**. It can handle advanced typogaphy features like ligartures and complex scripts like Arabic, Indic, Khmer and Thai.

This repository is a self-contained demo for the **Cheap Yellow Display** (ESP32-2432S028R), the popular ~$10 ESP32 dev board with a 320 × 240 ILI9341 LCD and an XPT2046 resistive touchscreen.

It runs the full OpenType pipeline live on the chip:

- **GSUB** substitution: ligatures, contextual rules, alternates, reverse contextual.
- **GPOS** positioning: kerning, mark-to-base, mark-to-mark, cursive attachment.
- **Bidi** (UAX #9) for mixed LTR/RTL text.
- Script-specific shaping: Arabic joining, Devanagari and other Indic reordering, Khmer, Myanmar, Hangul, Thai, and the Universal Shaping Engine.
- Both **TrueType (glyf)** and **PostScript (CFF)** outlines.
- An anti-aliased, gamma-corrected scanline rasterizer with strip-based output to the LCD.

The whole thing is reactive to a custom three-row touch gesture model: the top of the screen navigates between demos, the middle row navigates within a demo, and the bottom row is a swipe / slider area.

## Hardware

- **Cheap Yellow Display** (ESP32-2432S028R). Variants without the resistive touch panel will most likely not work.
- 320 × 240 ILI9341 LCD over SPI2 at 80 MHz with DMA.
- XPT2046 resistive touchscreen on SPI3 at 2 MHz.
- Standard 4 MB flash, no PSRAM, **~320 KB internal SRAM**.

## What's in the binary

Total flashed image is **~2.6 MB**, of which **~2.0 MB is fonts** from Google Fonts (embedded as `include_bytes!` in the binary's read-only data) and the remaining ~645 KB is code + data. Sizes are from `xtensa-esp32-elf-size` and a per-symbol breakdown via `xtensa-esp32-elf-nm` on a release build.

![Binary percentages](img/binary-donut-chart.png)

**Fonts** (read-only data, embedded in flash):

| Font |         Size |
|---|-------------:|
| Noto Nastaliq Urdu |       517 KB |
| Hind |       285 KB |
| Noto Sans Arabic |       188 KB |
| NotoEmoji (subsetted) |       186 KB |
| Roboto Regular |       167 KB |
| Noto Sans Khmer |       102 KB |
| HennyPenny |        79 KB |
| Bonbon |        70 KB |
| SnowburstOne |        65 KB |
| Peralta |        57 KB |
| Almendra |        56 KB |
| Ultra |        50 KB |
| Assistant |        49 KB |
| Droid Serif |        43 KB |
| Architects Daughter |        37 KB |
| Orbitron |        24 KB |
| **Fonts subtotal** | **~1.93 MB** |

All are static monochrome vector fonts (not variable fonts). The fonts were downloaded from Google Fonts and are not subsetted (except for emoji); For a real product they would be slimmed down.

**Code and data** (~480 KB total):

| Component                                                          |         Size |
|--------------------------------------------------------------------|-------------:|
| **Flattype** code and data (bidi, shaper, renderer, rasterizer)    |  **~135 KB** |
| Render path (flattener, rasterizer, pushing)                       |       ~35 KB |
| **Font shaping + rendering + rasterizing subtotal**                |  **~170 KB** |
| Demo logic (layout, touch, display, intro, the 5 demos, `main`)    |       ~30 KB |
| ESP-IDF C runtime (FreeRTOS, drivers, ROM glue)                    |      ~165 KB |
| Rust (`core`, `std` + `alloc`)                                     |      ~155 KB |
| Backtrace machinery (`gimli`, `addr2line`, `rustc_demangle`, etc)  |       ~85 KB |
| `compiler_builtins`, `esp-idf-hal`, `mipidsi`, other small crates  |        ~8 KB |
| Misc rodata not attributable above (vtables, format strings, etc.) |       ~20 KB |
| **Code + data subtotal**                                           |  **~635 KB** |
| ** Fonts (16 .ttf files embedded via include_bytes!) **            | **~1.93 MB** |
| **Grand total**                                                    |  **~2.6 MB** |

The headline number worth flagging: **the entire OpenType pipeline (shaping, outline extraction, flattening, and anti-aliased rasterizing) compiles to about 76 KB**. The largest single non-font cost is the ESP-IDF C runtime at ~147 KB, which is essentially fixed overhead for any Rust ESP32 program with `std` support, not specific to this demo.

## Demos

Six demos cycle with the **top row** of the touchscreen: left to go back, right to advance. The **middle row** navigates within a demo (e.g. cycling examples). The **bottom row** is a swipe / slider area whose meaning depends on the demo.

<table>
  <colgroup>
    <col style="width: 22%">
    <col style="width: 22%">
    <col style="width: 56%">
  </colgroup>
  <thead>
    <tr>
      <th>Demo / what it does</th>
      <th>Image</th>
      <th>Why it's cool</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td rowspan="4"><strong>Typography</strong><br>Five hand-picked typographic samples: Latin with ligatures, Nastaliq, Khmer, Devanagari, mixed Hebrew + Latin. The text is drawn in a light grey "ghost"; a slider on the bottom row reveals it character by character in black as you drag. Middle row cycles examples.</td>
      <td><img src="img/demo1c.jpg" width="280" alt="Latin with ligatures: 'The first conflict'"></td>
      <td>Latin ligature reveal: <code>fi</code>, <code>ct</code>, <code>st</code> recombine as the slider crosses each glyph.</td>
    </tr>
    <tr>
      <td><img src="img/demo1b.jpg" width="280" alt="Nastaliq Urdu sample"></td>
      <td>Nastaliq's diagonal vertical stacking, one of the most demanding shapings around, live on a $10 board.</td>
    </tr>
    <tr>
      <td><img src="img/demo1a.jpg" width="280" alt="Khmer sample"></td>
      <td>Khmer subscripted consonant clusters: characters stack below the baseline in their own subjoined forms.</td>
    </tr>
    <tr>
      <td><img src="img/demo1d.jpg" width="280" alt="Devanagari sample"></td>
      <td>Devanagari conjuncts joined under the shirorekha (top bar) into a single visual unit.</td>
    </tr>
    <tr>
      <td rowspan="2"><strong>Latin text</strong><br>A long Lorem Ipsum paragraph rendered in nine display fonts at variable size. Middle row taps switch font; the bottom row swipe scales 14–120 px continuously.</td>
      <td><img src="img/demo2a.jpg" width="280" alt="Lorem Ipsum in a serif face"></td>
      <td>Word wrapping, kerning, and contextual substitutions recomputed every frame as the size changes.</td>
    </tr>
    <tr>
      <td><img src="img/demo2b.jpg" width="280" alt="Lorem Ipsum in a handwritten display face"></td>
      <td>Deliberately weird display faces (Bonbon, Peralta, Henny Penny…) show the shaper isn't only happy with system Latin.</td>
    </tr>
    <tr>
      <td rowspan="3"><strong>Exotic text</strong><br>The same Lorem in Arabic, Devanagari, and Hebrew. Middle row cycles scripts; bottom row swipe scales size.</td>
      <td><img src="img/demo3a.jpg" width="280" alt="Arabic Lorem Ipsum"></td>
      <td>Cursive joining: each letter takes initial/medial/final/isolated forms depending on its neighbours, all decided live.</td>
    </tr>
    <tr>
      <td><img src="img/demo3b.jpg" width="280" alt="Devanagari Lorem Ipsum"></td>
      <td>Devanagari conjunct reordering: vowel marks visually precede consonants they logically follow.</td>
    </tr>
    <tr>
      <td><img src="img/demo3c.jpg" width="280" alt="Hebrew Lorem Ipsum"></td>
      <td>Bidirectional layout: RTL Hebrew interleaved with LTR Latin punctuation and numerals.</td>
    </tr>
    <tr>
      <td><strong>Emoji</strong><br>A randomly-shuffled run of ~250 colour emoji. Middle row tap reshuffles; bottom row swipe scales.</td>
      <td><img src="img/demo4.jpg" width="280" alt="Grid of emoji glyphs"></td>
      <td>Emoji glyphs are very curve-heavy: a single 90 px emoji can flatten to ~900 line segments. Stresses the rasterizer's per-glyph allocation path against a tight no-PSRAM heap.</td>
    </tr>
    <tr>
      <td><strong>Random char</strong><br>One large character at 90 px, sampled from a pool of decorative Latin faces. Bottom row scrolls through letters and fonts deterministically.</td>
      <td><img src="img/demo5.jpg" width="280" alt="Large lowercase g with construction arrows"></td>
      <td>Worst-case rasterizer load at full glyph size, plus the most direct way to flip through the display fonts.</td>
    </tr>
    <tr>
      <td><strong>Colour on dark</strong><br>A short title rendered with per-glyph colour over a dark background.</td>
      <td><img src="img/demo6.jpg" width="280" alt="'The End' in colour gradients on a dark background"></td>
      <td>Rendering isn't locked to black-on-light. Each glyph can take its own colour and composite onto an arbitrary background, opening the door to theming and accents.</td>
    </tr>
  </tbody>
</table>


## Credits

- [Flattype](https://www.flattype.com): the OpenType shaper and renderer, ported to Rust from the JavaScript original.
- [mipidsi](https://crates.io/crates/mipidsi): ILI9341 driver.
- [esp-idf-hal](https://crates.io/crates/esp-idf-hal): ESP-IDF Rust bindings.
- Fonts: Roboto, Noto Sans family, Hind, Almendra, Assistant, and the display faces, all from Google Fonts under the SIL Open Font License.


