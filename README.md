# 🎵 Riyaz — Pitch Perfection

A lightweight, single-file web app for practicing the pitch of **Hindustani
classical swaras**. Pick your Sa, hit **Practice**, and sing — the app listens
through your microphone and draws your live pitch on a swara grid, colouring it
by how close you are to the perfect note.

There is no build step and there are no dependencies. The entire app is one
`index.html` file that runs in any modern browser.

## Features

- **Shruti (Sa) selector** — choose your tonic as an explicit note + octave
  (e.g. `B2`, `C3`). The range covers common male and female Sa positions
  (A2 → A3). All swara positions are computed relative to this Sa.
- **Range selector** — choose which saptaks (octaves) to display:
  - Mandra Sa → Sa
  - Madhya Sa → Sa
  - Taar Sa → Sa
  - Mandra Pa → Taar Pa (default, widest)
- **Live pitch graph** — horizontal lines mark each swara
  (`S r R g G m M P d D n N`, with `.` for mandra and `'` for taar). Your sung
  pitch scrolls across as a coloured trail.
- **Colour feedback**
  - 🟢 **Green** — perfect (within ±10 cents)
  - 🟡 **Yellow** — close (within ±35 cents)
  - 🔴 **Red** — off pitch
- **Note readout** — the current swara, cents deviation, and detected frequency
  (Hz). A blinking red dot indicates the mic is listening.

## Usage

1. Open `index.html` in a browser (Chrome/Edge/Firefox/Safari).
   - For microphone access, most browsers require a **secure context**. Chrome
     allows it for local files; if your browser blocks the mic, serve the file
     locally instead:
     ```bash
     python3 -m http.server 8000
     # then open http://localhost:8000
     ```
2. Select your **Shruti (Sa)** and a **Range**.
3. Click **Practice** and allow microphone access when prompted.
4. Sing — match the trail to a swara line and aim for green.
5. Click **Stop** when done.

## How it works

| Step | What happens |
| ---- | ------------ |
| Sa frequency | Derived from the selected MIDI note: `440 · 2^((midi − 69) / 12)`. |
| Capture | `getUserMedia` → Web Audio `AnalyserNode` (FFT size 2048). |
| Pitch detection | Autocorrelation over a bounded lag window (~70–1100 Hz) so it locks onto the vocal fundamental instead of a harmonic. Parabolic interpolation refines the estimate. |
| Mapping | Frequency → semitones above Sa (`12 · log2(f / Sa)`), snapped to the nearest swara; the remaining error in cents drives the colour. |
| Stability | Light exponential smoothing plus an octave-jump guard reduce flicker and harmonic errors. |
| Rendering | A `<canvas>` draws the swara grid and a scrolling coloured pitch trail (green/yellow/red per frame). |

### A note on tuning

The pitch model uses **12-tone equal temperament** — each swara is exactly 100
cents apart. Authentic Hindustani intonation is shruti/just-intonation based,
so a classically "perfect" Ga or Ni may read slightly off here. For the goal of
training precise, consistent pitch this is a reasonable and demanding target
(±10 cents is near the limit of human pitch discrimination).

## Tech

- Plain HTML + CSS + vanilla JavaScript
- Web Audio API (`AudioContext`, `AnalyserNode`)
- Canvas 2D for rendering
- No frameworks, no build tooling, no external assets

## License

MIT
