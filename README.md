# Tokaphone

Your phone is the instrument. Make some music.

Tokaphone turns phones into instruments so people who can't play music can jam and make songs. It is one file, `index.html`, hosted on Netlify.

- Live: https://tokaphone.netlify.app
- Netlify project: `tokaphone`

## Working on it

There is no build step. Edit `index.html`, open it in a browser, and it runs. Camera and mic only work on the hosted https site, not on a local file or a preview, so test Air, Whistle and Voice on the deployed link.

Deploy: the Netlify site publishes this folder (`netlify.toml` sets `publish = "."`). Either link the Netlify site to this repo so every push to `main` deploys, or deploy the folder manually with the Netlify CLI.

## Design rules

- Minimal and elite: thin black lines, no helper copy, no em dashes.
- Everything locks to the drum machine clock and to A minor (Am F C G).
- Camera and mic only work on the hosted https site.

## Code map (all in index.html)

- Audio core: `audio()`, `bus` (melodic, echo + reverb), `drumBus`, `mfilt` (master lowpass), `fx.dg` / `fx.rg` sends, `VIBES`.
- Drum machine: `PATTERNS`, `beat`, `beatTick()`, `setBeat()`. Clock is epoch based: `epochAtCtx0` converts between Date.now() and audio context time.
- Event loop recorder: `trig(ev, when)` records and plays, `run(ev, when)` plays, `loops`, `loopTick()`. Pluck events take an optional velocity `v`.
- Screens: `openInst(kind)`, `INSTS`, `octwrap` (top controls), `stage` (play area). The stage swallows clicks, so in-stage controls must use `zone(el, {down, move, up})`. `stageStop` is the cleanup hook.
- Round 3 module (last `<script>` block, "BODY + VOICE"):
  - Shared detectors: `peaker()` finds the bottom and top of a repeating motion. `swinger()` fires the moment a fast move starts braking, before it reverses, one hit per swing. `hitTime()` plays now, or on the next 16th if it is under 50 ms away. Nothing is pushed into a later beat.
  - **Air** (`buildAir`): MediaPipe tasks-vision 0.10.14 from jsDelivr. `AIR_NEEDS` picks the model per mode: pose for Dance and Drums, HandLandmarker for Guitar and Sax. Selfie segmenter draws the lime (#C2F000) screen with a black silhouette every third frame. `AIR_INTRO` holds the per-mode intro card (tap to play, `?` brings it back).
    - Dance: a hand above the chest plays a note on every 8th, height is pitch, left hand an octave down. Bounce plays bass, lean picks root, third or fifth. Nod is a clap, jump is a crash with a swell, arms wide adds echo, freezing for over a second cuts the drums. Motion energy sweeps `mfilt` so the mix opens as you move.
    - Drums: `DZ` is a kit in body units anchored to the shoulders, so it follows you. A strike is the braking of a downswing, the nearest zone sounds. A knee lift is the kick.
    - Guitar: the faster hand strums, one swing is one strum, velocity scaled. The other hand picks the chord by extended fingers (1 to 4 is Am F C G), a fist mutes.
    - Sax: hands above the gate line play, curled fingers walk down `SAXN`, drop below the line to stop. Octave buttons. Optional Breath mode reads mic level.
  - **Whistle** (`buildWhistle`): hold to record, YIN pitch analysis, `segment()` into notes, grid quantize, loop as a note layer. Instrument picker (`NOTE_INSTS`), octave, Glide, Grid. Drag notes in `takeEditor`. Keep = new layer.
  - **Voice** (`buildVoice`): hold to sing (headphones), YIN at 16k, PSOLA re-pitch (`pitchMarks`, `psola`), Hard or Natural tune, sounds Clean / Robot (offline vocoder) / Choir / Deep / High. Drag notes to retune. Keep = new layer.
  - Layers: `bodyLayers`, `noteSched()`, `startBufLayer()`, `takeWindow()`; the picker's loop count and clear button include them.

## Status

- Round 3 rebuilt Air after phone testing of round 2: hits were quantized up to a beat late, dance doubled sounds the beat already had, drums were fixed screen spots, guitar strums clipped, sax droned without a gate.
- Round 3 is tested with simulated bodies and hands only. Gesture thresholds (`swinger` minimum speeds, knee lift, sax gate) need tuning from real phone tests.
