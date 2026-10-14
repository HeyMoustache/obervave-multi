# OberVave Multi: six parts on six MIDI channels

This branch makes OberVave multitimbral. There are **six parts**, each a complete OberVave sound with its own
program. They share the **8 voices**: a part takes voices when it plays and gives them back when its notes end.

## MIDI

| GLOBAL page, MIDI knob | What it does |
| --- | --- |
| **MULTI** (the default) | Channels 1-6 play parts 1-6. Channels 7-16 play the part the panel is on. |
| **1 .. 16** | Only that channel, and it plays the panel's part (as OberVave 1.2). |
| **OMNI** | Every channel plays the panel's part (as OberVave 1.2's OMNI). |

Each part listens on its own channel for notes, pitch bend, the sustain pedal (CC 64), volume (CC 7), program
change and every sound CC (the same numbers as OberVave: CC 74 cutoff, 71 resonance and so on).

## The panel

The keys, the knobs, the arpeggiator and HOLD work on **one part at a time**, the *panel's part*. The screen's
second line starts with it: `P1`, `P2` ...

- **Pick a part:** hold **GLO** and press one of the white keys **9 to 14** (part 1 to 6).
- **Or:** tap **GLO** until the **PARTS** page shows (GLOBAL, VOICE, OUTPUT, PARTS):
  - KNOB 1 **PART**: the panel's part
  - KNOB 2 **TRACK A**, KNOB 3 **TRACK B**: which part each sequencer track plays
  - KNOB 4 **VOLUME**: the panel part's volume (a quick mixer)
- **PRESETS** changes the panel part's program. Every page edits the panel part's sound.
- **SAVE** saves the panel part's sound as a program, as before.

The settings remember each part's program, the tracks' parts and the panel's part. Programs and patterns saved
with OberVave 1.2 are kept.

## Effects

There is one chorus, one delay and one reverb, shared by all parts, like on a mixing desk:

- Each part's **CHORUS MIX**, **DELAY MIX** and **REVERB MIX** are its *sends*: how much of that part goes in.
- How the effects sound (chorus rate and depth, delay time, feedback and tone, reverb size, damping and
  pre-delay) comes from **part 1's** program.

With one part playing, a sound without effects matches OberVave 1.2 (the samples differ by 4 at most, out of 32767). With effects
it is very close; one difference: the chorus no longer feeds the delay and the reverb.

The reverb runs at half the sample rate (a slightly darker tail, about 13% less work in all), and the very last
stage is a limiter instead of a hard clip: when the mix would go over, it turns the whole output down for a moment
(and back up over about 50 ms), so six loud parts with every effect up do not crackle. Below that it changes nothing.

## What a part does in each voice mode

- **POLY:** one voice a note.
- **DUAL:** two voices a note.
- **UNISON:** the VOICES setting's number of voices for one note (up to 8: that leaves none for the other parts).
- **MONO:** one voice, legato. Going back to a key still held works for the panel's part.

When all 8 voices are busy, a new note takes the oldest released voice, then the part's own oldest, then the
oldest of all.

## Cost

Measured on the computer build (instructions per sample, valgrind): the 8-voice worst case is about 17% more
than OberVave 1.2, with all six parts or with one. Silent parts cost almost nothing. Check the CPU time on the
GLOBAL page on your FM-1 with a big patch and all effects on: OberVave aims for under 2900 us.

## Identity

`FM-1_96N` (OberVave is `FM-1_95N`), so installers and the hub can tell them apart.
