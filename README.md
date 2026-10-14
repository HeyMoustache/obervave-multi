<p align="center"><img src="assets/logo.png" width="800" alt="OberVave: the name in red glowing outlines, three blue bands through the lower half of the letters"></p>

# OberVave

**One 8-voice analog-style polysynth for the M-VAVE FM-1, with chorus, delay and a plate reverb.**

OberVave turns the FM-1 into a single, deep instrument in the spirit of the classic American 8-voice poly synths of
1979-84: two oscillators a voice with hard sync and cross-modulation, a 12 / 24 dB filter with an SEM-style
lowpass-to-highpass morph, two envelopes, an LFO, an arpeggiator, a two-track step sequencer, 32 programs, and
three effects after the voices. No second engine: one synth, as far as the chip goes.

<p align="center"><img src="docs/screen.png" width="240" alt="OberVave's screen: program 01 DARK ARP, the ARP page with four knobs, the scope, eight voice LEDs"></p>

> **Status: 1.2.1, runs on the FM-1.** Install it from [thefirmwarehub.com](https://thefirmwarehub.com) or with
> `tools/fm1_install.py docs/firmware/obervave-1.2.1.fwsc` (identity `FM-1_952`). 1.2.1 fixes the knobs while a
> function button is held on the GLOBAL page (they turned the GLOBAL settings, such as TUNE, instead of the sequencer). Do not
> install 1.1: its USB-MIDI out is broken, and once it runs no installer gets another firmware back on (1.2 fixed it).

**Play it in your browser:** `docs/sim/` is the firmware itself (the same C sources, compiled to WebAssembly) with
the FM-1's screen, keys, buttons and knobs, your computer keyboard or a MIDI keyboard. Build it with
`sh web/sim/build.sh`, try it with `python3 -m http.server -d docs/sim` (see `web/sim/build.sh` for the two tools).

**This branch: OberVave Multi**, six parts on MIDI channels 1-6: see [docs/MULTI.md](docs/MULTI.md).

**M-Vave SMK-37:** see [SMK-37 instructions](SMK37.md).

Hear it: [the video](video/) is nine OberVaves (the host build) playing one slow, dark track; every sound in it,
drums included, is this firmware.

## The synth

| | |
| --- | --- |
| **Voices** | 8. POLY (round-robin, steals the oldest released), DUAL (2 detuned voices a note), UNISON (2-8 voices, detune and pan spread), MONO (last note, legato). Glide in every mode. |
| **Oscillators** | 2 a voice: saw, pulse, saw+pulse, off. Pulse width and PWM. OSC 2: ±24 semitones, fine detune, hard sync to OSC 1, filter envelope to pitch (sync sweeps), X-MOD (OSC 2 → OSC 1 linear FM). Noise. PolyBLEP anti-aliasing, free-running phases. |
| **Filter** | State-variable, trapezoidal: LP 12, HP 12, BP 12, notch, LP 24 (two cascaded), SEM (LP → notch → HP by MORPH). Cutoff, resonance with a soft, bounded self-oscillation, envelope amount (bipolar), key tracking, velocity, drive before the filter. |
| **Envelopes** | Filter ADSR and amp ADSR with analog RC curves, 0.5 ms to 10 s. |
| **LFO** | Triangle, saw, ramp, square, S&H, smooth random; 0.05 to 30 Hz; delay / fade-in; phase SYNC / FREE / SPREAD (an offset per voice). To pitch (OSC 1 / 2 / both), PWM, filter, amp. The mod wheel adds vibrato and filter. |
| **Vintage** | Each of the 8 voice cards has its own small tune, cutoff and envelope offsets, plus a slow random walk on pitch, all scaled by one knob. SPREAD pans the cards like the originals' pan pots. |
| **Effects** | Chorus (two taps, Hermite-interpolated). Ping-pong delay in note values of the tempo (1/16 to 1/2, dotted and triplet), with feedback tone; the time slews like a tape when it changes. Plate reverb (Dattorro's figure-of-eight tank) at the full sample rate: SIZE, DAMP, PRE-DLY, MIX. |
| **Arpeggiator** | UP, DOWN, UP/DN, RANDOM, AS PLAYED; 1-4 octaves; 1/4 to 1/32 with triplets; gate; latch (HOLD). Sample-exact, in the audio interrupt. Follows MIDI clock. |
| **Sequencer** | Two tracks that share 256 steps (one track can have them all, or A and B split them). A step is a chord of up to 4 notes with velocity, gate or a tie into the next step, a ratchet (1-4 hits) and a chance. Each track has its own length (polymeters), note value (1/4 to 1/32 with triplets), swing and transpose, and can be muted. Step recording (chords from the keys or MIDI, rests) and live recording into the running loop, quantized to the nearest step. 8 patterns in flash, two factory ones in D Phrygian. Sample-exact in the audio interrupt, follows MIDI clock, start and stop. Both tracks play the one OberVave sound. |
| **Programs** | 32: factory set in flash, your saves in the settings sector. 01 DARK ARP, 26 UPSIDE ARP, 29 NEON PAD are the dark ones; brass, strings, leads, basses, bells, pads, drums-from-a-filter for the rest. Hold SAVE and press the SELECT knob for a new one at random, within useful ranges. |
| **MIDI** | USB-MIDI in and out. Notes, velocity, pitch bend (range 0-24), program change, sustain, mod wheel, clock / start / continue, MIDI channel or OMNI. CC 74 cutoff, 71 resonance, 73 / 72 amp attack / release, 5 glide, 1 mod depth, 7 volume, 64 sustain, 120 / 123 all off; every other parameter from CC 20 up in the order of the parameter table (see `firmware/src/obv_params.h` and CLAUDE.md). |

Integers only: the chip has no FPU. The whole engine is fixed point, the tables are generated by
`tools/gen_tables.py` and committed.

## The panel

The panel works as sloopDX's: **tap** a function button (press and let go, touching nothing else) and its pages
open; **hold** it and it is a **layer**: the white keys and the knobs change job until you let go. **HOME** tapped
while a layer is held locks it open; its button lets it go.

| Hold | Keys | KNOB 1 · 2 · 3 · 4 | Tap |
| --- | --- | --- | --- |
| **SEQ** | steps 1-16 of the page (tap: set with the last note or chord / clear; hold + a later step: one long note); black keys F#3 .. A#4: page 1-8 | NOTE · DIV · SWING · LENGTH; a step held: NOTE · LEVEL · RATCHET · GATE | the sequencer page |
| **EDIT** | erase that note (playing: as the playhead passes) | SHIFT · LENGTH x2 / 1/2 · TRANSPOSE | OSC / FILTER pages |
| **GLO** | 1 / 2 mute track A / B, 5 / 6 solo, G5 tap tempo | - | GLOBAL (TEMPO, CLICK, MIDI CH, TUNE; the CPU time in its title) / VOICE / OUTPUT |
| **SAVE** | 1-8: pattern 1-8 (with SELECT pressed: a random sound) | - | twice: save the program (on the sequencer page: the pattern) |
| **FX** | notes | CHORUS · DELAY · REVERB mix · DELAY TIME | CHORUS / DELAY / REVERB |

- **PLAY** start / stop (the light flashes on the beat). **REC** playing: record now on top (chords, held notes
  long) / stop; stopped: armed, the first note starts (CLICK on COUNT-IN: PLAY counts in a bar first). **REC held
  2 s** clears the track. The **click** (GLOBAL, KNOB 2): OFF, REC (while recording), ON, COUNT-IN; beat 1 higher, never
  recorded. **EDIT + OCT- / OCT+**
  undo / redo.
- **ALGORITHM** picks the track (A / B), **SELECT** is the tempo, **PRESETS** the program, **OCT- / OCT+** the
  octave (both: back to 0). **HOME** PERFORM (CUTOFF, RESO, ENV AMT, RELEASE). **ENV**, **LFO** their pages.
  **ARP** the arp page; on it the ARP button is HOLD (latch).
- The key LEDs show the notes that sound, the page buttons light for the page you are on, the eight round LEDs on
  the screen glow with each voice's envelope.

## Build and run

OberVave is built on [Dinghy](https://github.com/zvenson/dinghy), whose platform (boot, USB, updates, audio, settings)
it keeps unchanged; the setup is Dinghy's (Linux x86-64, Python 3 with Pillow, the JieLi toolchain, three files of
the JieLi AC79 SDK: see Dinghy's README, "Set up once").

```sh
tests/run_host.sh                 # the synth on your computer: build/host/obervave.wav and screens as PNG
tests/run_host.sh my-script.txt   # your own keys / buttons / knob turns / MIDI over time (tests/host.c explains)
sh web/sim/build.sh               # the browser simulator: docs/sim (zig cc + wasm-opt), node tests/sim_test.mjs
./build.sh                        # the device package: build/obervave.fwsc
python3 tools/fm1_install.py build/obervave.fwsc   # onto the FM-1 over USB
```

The build refuses what would brick an FM-1 (a 64-bit division, a call into the chip ROM, too much RAM); the rules
are in `docs/RULES.md`, the platform calls in `docs/API.md`.

The video: `video/song.py` writes nine scripts, runs nine host instances filmed at 30 fps and mixes the stems;
`video/make_video.py` cuts the film. Both need only Python, NumPy, Pillow and ffmpeg.

## Credits and licence

GPL-3.0-only, see `LICENSE` and `LICENSING.md` for who made what. OberVave stands on
[Dinghy](https://github.com/zvenson/dinghy) (zvenson), which carries the FM-1 platform of
[Felucca](https://github.com/hugelton/Felucca) by Leo Kuroshita (Hügelton Instruments) with the changes of
[SLOOP](https://github.com/isod89/sloop-fm1) (isod89). The filter follows Andrew Simper's trapezoidal SVF, the reverb
Jon Dattorro's 1997 plate.

OberVave is inspired by the classic 8-voice poly synths of 1979-84 (Oberheim's OB-X and OB-8 era). Those names are
trademarks of their owners; OberVave is not made by or affiliated with them, and none of the marks appear in the
firmware. "M-VAVE" and "FM-1" belong to their owners too.
