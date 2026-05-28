# Flattype on a Cheap Yellow Display (ESP32-2432S028R)


[<video controls src="https://github.com/waruyama/flattype-cyd-demo/blob/main/video/flattype-cyd-demo-cHtd6wyP.mp4" title="Demo of Flattype on Cheap Yellow Display"></video>
](https://github.com/user-attachments/assets/e0aefb1e-3466-4e44-a137-e85ac51900db)


## What is this?

**All the text you see in this demo is rendered live on the chip from TrueType fonts. There are no prerendered bitmaps, no glyph atlases, nothing pre-baked. All the text goes through the full OpenType shaper and rasterizer.**

This repository contains a ready-to-flash installable version of a demo that showcases TrueType vector font rendering in a very resource-constrained embedded environment using Flattype. Flattype is a tiny OpenType pipeline written in pure Rust (shaper, glyph-outline extractor, and anti-aliased rasterizer) that fits inside an ESP32 with **320 KB of RAM, 4 MB of flash, and no PSRAM**. It can handle advanced typography features like ligatures and complex scripts like Arabic, Indic, Khmer, and Thai.

The demo is designed for the **Cheap Yellow Display** (ESP32-2432S028R), the popular ~$10 ESP32 dev board with a 320 × 240 ILI9341 LCD and an XPT2046 resistive touchscreen.

It runs the full OpenType pipeline live on the chip:

- Bidirectional text ordering.
- Substitution: ligatures, contextual rules, alternates, reverse contextual.
- Positioning: kerning, mark-to-base, mark-to-mark, cursive attachment.
- Script-specific shaping: Arabic joining, Devanagari and other Indic reordering, Khmer, Myanmar, Hangul, Thai, and universal shaping for less common scripts.
- Both **TrueType (glyf)** and **PostScript (CFF)** outlines.
- Anti-aliased, gamma-corrected rendering, drawn to the LCD in horizontal strips.

Navigation works via a three-row touch gesture model: the top of the screen cycles between demos, the middle row navigates within a demo, and the bottom row is a swipe / slider area.


## Why this matters

This appears to be the first demonstration of full OpenType shaping with complex script support on hardware this constrained. Typically, embedded displays meant pre-rendering bitmaps (inflexible, storage-heavy) or commissioning special fonts. Here, any text is shaped and rasterized live: ligatures, kerning, complex scripts (Arabic, Indic, Thai, Khmer, etc.), and bidirectional text reordering. Put any standard OpenType or TrueType font on flash, and the pipeline handles everything at render time on a ~$10 microcontroller with 320 KB of RAM. All in 135 KB of code.


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
      <td rowspan="4"><strong>Typography</strong><br>Four hand-picked typographic samples: Latin with ligatures, Nastaliq, Khmer, Devanagari. The text is drawn in a light grey "ghost"; a slider on the bottom row reveals it character by character in black as you drag. Middle row cycles examples.</td>
      <td><img src="img/demo1c.jpg" width="280" alt="Latin with ligatures: 'The first conflict'"></td>
      <td>Latin ligature reveal: <code>Th</code>, <code>fi</code>, <code>st</code>, <code>fl</code>, and <code>ct</code> combine into ligatures as the slider crosses each glyph.</td>
    </tr>
    <tr>
      <td><img src="img/demo1b.jpg" width="280" alt="Nastaliq Urdu sample"></td>
      <td>Nastaliq's diagonal vertical stacking, considered one of the most demanding scripts for shaping.</td>
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
      <td>Word wrapping, kerning, and contextual substitutions.</td>
    </tr>
    <tr>
      <td><img src="img/demo2b.jpg" width="280" alt="Lorem Ipsum in a handwritten display face"></td>
      <td>Deliberately weird display faces (Bonbon, Peralta, Henny Penny…) show the shaper isn't only happy with standard serif and sans-serif fonts.</td>
    </tr>
    <tr>
      <td rowspan="3"><strong>Complex scripts</strong><br>The same Lorem in Arabic, Devanagari, and Hebrew. Middle row cycles scripts; bottom row swipe scales size.</td>
      <td><img src="img/demo3a.jpg" width="280" alt="Arabic Lorem Ipsum"></td>
      <td>Cursive joining: each letter takes initial/medial/final/isolated forms depending on its neighbours, all decided live.</td>
    </tr>
    <tr>
      <td><img src="img/demo3b.jpg" width="280" alt="Devanagari Lorem Ipsum"></td>
      <td>Devanagari conjunct reordering: vowel marks visually precede consonants they logically follow.</td>
    </tr>
    <tr>
      <td><img src="img/demo3c.jpg" width="280" alt="Hebrew Lorem Ipsum"></td>
      <td>Right-to-left layout, with Latin punctuation and digits embedded inline (live bidi).</td>
    </tr>
    <tr>
      <td><strong>Emoji</strong><br>A randomly shuffled run of ~250 colour emoji. Middle row tap reshuffles; bottom row swipe scales.</td>
      <td><img src="img/demo4.jpg" width="280" alt="Grid of emoji glyphs"></td>
      <td>Emoji glyphs are very curve-heavy: a single 90 px emoji can flatten to ~900 line segments. This stresses the rasterizer's per-glyph allocation.</td>
    </tr>
    <tr>
      <td><strong>Random char</strong><br>One large character at 90 px, sampled from a pool of decorative Latin faces. Bottom row scrolls through letters and fonts deterministically.</td>
      <td><img src="img/demo5.jpg" width="280" alt="Large lowercase g with construction arrows"></td>
      <td>Swapping fonts is essentially free. Fonts take almost no memory in RAM, so multiple fonts can be held in memory at the same time. This makes switching between fonts as fast as rendering different text from the same font.</td>
    </tr>
    <tr>
      <td><strong>Colour on dark</strong><br>A short title rendered with per-glyph colour over a dark background.</td>
      <td><img src="img/demo6.jpg" width="280" alt="'The End' in colour gradients on a dark background"></td>
      <td>Rendering isn't locked to black-on-light. Each glyph can take its own colour and composite onto an arbitrary background, opening the door to theming and accents.</td>
    </tr>
  </tbody>
</table>


## Hardware

- **Cheap Yellow Display** (ESP32-2432S028R). Variants without the resistive touch panel will most likely not work.
- 320 × 240 ILI9341 LCD over SPI2 at 80 MHz with DMA.
- XPT2046 resistive touchscreen on SPI3 at 2 MHz.
- Standard 4 MB flash, no PSRAM, **~320 KB internal SRAM**.


## Performance

Stage-by-stage timings measured on the CYD. **Shaping** happens once when starting the demo or when changing the text; all other steps run on every font-size change.

All measurements are with smallest font size and maximum number of visible glyphs. 13 lines of text with approx. 200-800 glyphs (depending on font and script).

- **Refresh subtotal** = rendering + rasterizing + pushing
- **Total** = refresh + shaping (the cost of the first frame after a text change)

| Stage                | Single glyph |          Latin |     Arabic | Devanagari |     Hebrew |      Emoji |
|----------------------|-------------:|---------------:|-----------:|-----------:|-----------:|-----------:|
| Number of glyphs     |            1 |        360-548 |        809 |        618 |        623 |        208 |
| **Shaping**          |         0 ms |       12-46 ms |      79 ms |     156 ms |      34 ms |       9 ms |
| Line-breaking        |         1 ms |        7-10 ms |      14 ms |      14 ms |      13 ms |       0 ms |
| Rendering            |         0 ms |       25-97 ms |      47 ms |      30 ms |      18 ms |      95 ms |
| Flattening           |         0 ms |       13-65 ms |      35 ms |      21 ms |      14 ms |      56 ms |
| Rasterizing          |       1-2 ms |      64-187 ms |     121 ms |     111 ms |      77 ms |     199 ms |
| Pushing              |        18 ms |           5 ms |       5 ms |       5 ms |       4 ms |       5 ms |
| Other                |         0 ms |           1 ms |       1 ms |       1 ms |       1 ms |       1 ms |
| **Refresh subtotal** |    **21 ms** | **114-314 ms** | **224 ms** | **183 ms** | **130 ms** | **360 ms** |
| **Total**            |    **21 ms** | **135-350 ms** | **303 ms** | **339 ms** | **164 ms** | **369 ms** |

## What's in the binary

Total flashed image is **~2.6 MB**, of which **~1.9 MB is fonts** from Google Fonts (embedded as `include_bytes!` in the binary's read-only data) and the remaining ~480 KB is code + data. Sizes are from `xtensa-esp32-elf-size` and a per-symbol breakdown via `xtensa-esp32-elf-nm` on a release build.

![Binary percentages](img/binary-donut-chart.png)

**Code and data** (~480 KB total):

| Component                                                          |         Size |
|--------------------------------------------------------------------|-------------:|
| **Flattype** code and data (bidi, shaper, renderer, rasterizer)    |  **~135 KB** |
| Render path (flattener, rasterizer, pushing)                       |       ~35 KB |
| **Text shaping + rendering + rasterizing subtotal**                |  **~170 KB** |
| Demo logic (layout, touch, display, intro, the 6 demos, `main`)    |       ~30 KB |
| ESP-IDF C runtime (FreeRTOS, drivers, ROM glue)                    |      ~145 KB |
| Rust (`core`, `std` + `alloc`)                                     |      ~155 KB |
| Backtrace machinery (`gimli`, `addr2line`, `rustc_demangle`, etc)  |       ~85 KB |
| `compiler_builtins`, `esp-idf-hal`, `mipidsi`, other small crates  |        ~8 KB |
| Misc rodata not attributable above (vtables, format strings, etc.) |       ~20 KB |
| **Code + data subtotal**                                           |  **~480 KB** |
| **Fonts (16 .ttf files embedded via include_bytes!)**              | **~1.93 MB** |
| **Grand total**                                                    |  **~2.6 MB** |

The headline number worth flagging: **the entire OpenType pipeline (shaping, outline extraction, flattening, and anti-aliased rasterizing) compiles to about 170 KB**. The largest single non-font cost is the ESP-IDF C runtime at ~145 KB, which is essentially fixed overhead for any Rust ESP32 program with `std` support, not specific to this demo.

**Fonts** (read-only data, embedded in flash): Noto Nastaliq Urdu (517 KB), Hind (285 KB), Noto Sans Arabic (188 KB), NotoEmoji subsetted (186 KB), Roboto Regular (167 KB), Noto Sans Khmer (102 KB), Henny Penny (79 KB), Bonbon (70 KB), SnowburstOne (65 KB), Peralta (57 KB), Almendra (56 KB), Ultra (50 KB), Assistant (49 KB), Droid Serif (43 KB), Architects Daughter (37 KB), and Orbitron (24 KB). **Fonts subtotal: ~1.93 MB.**

All are static monochrome vector fonts (not variable fonts). The fonts were downloaded from Google Fonts and are not subsetted (except for emoji); for a real product they would be slimmed down.


## Install the demo

### Web flasher (recommended)

Open **<https://waruyama.github.io/flattype-cyd-demo-install/>** in Chrome or Edge, plug your CYD into USB, and click **Connect**. Flashing takes about 30 seconds.

### Manual install

If you can't use the web flasher (Firefox, Safari, Linux without WebSerial permissions, etc.), grab `dist/firmware.bin` and flash it with [`esptool.py`](https://github.com/espressif/esptool):

```bash
esptool.py --chip esp32 --port /dev/ttyUSB0 write_flash 0x0 firmware.bin
```

Or with [`espflash`](https://github.com/esp-rs/espflash):

```bash
espflash write-bin 0x0 firmware.bin
```

The image is a merged ESP32 build (bootloader + partition table + app, 4 MB) that flashes at offset 0x0.


## Credits

- [Flattype](https://www.flattype.com): the OpenType shaper and renderer, ported to Rust from the JavaScript original.
- [mipidsi](https://crates.io/crates/mipidsi): ILI9341 driver.
- [esp-idf-hal](https://crates.io/crates/esp-idf-hal): ESP-IDF Rust bindings.
- Fonts: Roboto, Noto Sans family, Hind, Almendra, Assistant, and the display faces, all from Google Fonts under the SIL Open Font License.


