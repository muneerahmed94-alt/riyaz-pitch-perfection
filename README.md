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
  - 🩵 **Teal** — good (within ±20 cents)
  - 🟡 **Yellow** — close (within ±35 cents)
  - 🔴 **Red** — off pitch
- **Note readout** — the current swara, cents deviation, and detected frequency
  (Hz). A blinking red dot indicates the mic is listening.
- **MIDI keyboard (optional)** — connect an external MIDI keyboard (Web MIDI
  API) via the footer **🎹 MIDI** button to play a harmonium reference along
  with your practice. The adjacent **⚙** opens a small config popover:
  - **Device** — pick which connected MIDI input to listen to
  - **Brand** — harmonium voice: Paul & Co / Pakrashi / DSR / MKS
  - **Reed** — reed-bank stop (Bass-Male, Bass-Male-Female, … Bass-Bass-Male-Female)
  - **Coupler** — add an octave below (Left) or above (Right)
  - **Velocity sensitivity** — off by default (every note plays at a fixed
    moderate level, like a steady drone); enable to let key velocity scale loudness
  - **Octave** — shift the keyboard ±3 octaves
  - **Transpose** — shift ±12 semitones (align the keyboard to your Sa)

  Notes are voiced by the same physically-modelled harmonium synth used in the
  `riyaz-palta-alankar` sibling app — a source-filter model (reed
  `PeriodicWave` → per-brand wooden-cabinet biquad chain → small-room
  convolution reverb) with a multi-reed bank, bellows LFO, and attack chiff.
  It runs on its own AudioContext, independent of the microphone/pitch engine.
  (Requires a browser with Web MIDI support — Chrome/Edge; Safari/Firefox may
  need it enabled.)
- **Metronome (optional)** — a footer **🎵 Metronome** button starts/stops a
  click track; the adjacent **⚙** opens a config popover:
  - **Tempo** — 30–250 BPM via slider, ± buttons, or **Tap tempo**
  - **Beats per bar** — 2–7, with the first beat accented (higher click) and a
    live row of beat dots that light up in time
  - **Play _N_ bars and mute _M_ bars** — optional practice mode that silences
    the click for a stretch so you can keep time on your own, then brings it back

  Clicks are scheduled with a look-ahead scheduler on their own `AudioContext`,
  so timing stays solid independent of frame rate.

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
| Pitch detection | McLeod Pitch Method (NSDF) over a bounded lag window (~70–1100 Hz). Picking the first NSDF peak above 90% of the maximum locks onto the true fundamental instead of its octave-down sub-harmonic (which plain autocorrelation reports on higher notes). Parabolic interpolation refines the estimate. |
| Mapping | Frequency → semitones above Sa (`12 · log2(f / Sa)`), snapped to the nearest swara; the remaining error in cents drives the colour. |
| Stability | Light exponential smoothing plus an octave-jump guard reduce flicker and harmonic errors. |
| Rendering | A `<canvas>` draws the swara grid and a scrolling coloured pitch trail, tiered per frame (green ±10¢ / teal ±20¢ / yellow ±35¢ / red beyond). |
| MIDI (optional) | `navigator.requestMIDIAccess()` listens to the chosen input; note-on/off drive a physically-modelled harmonium synth (reed `PeriodicWave` → per-brand cabinet biquads → shared small-room convolution reverb, with a multi-reed bank, bellows LFO and attack chiff — ported from `riyaz-palta-alankar`) on its own `AudioContext`, with octave/transpose offsets applied before frequency conversion. |
| Metronome (optional) | A look-ahead scheduler (`setInterval` tick + `AudioContext` sample-clock scheduling) queues short sine clicks — accented first beat, plain remaining beats — with optional play/mute bar cycling and tap-tempo averaging. |

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
