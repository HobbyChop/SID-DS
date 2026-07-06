# SID-DS Owner's Manual

**C64 SID synthesizer & groovebox for the Nintendo DS**
Firmware v1.0.1B (beta), July 2026

---

## Table of contents

1. [Introduction](#1-introduction)
2. [Quick start](#2-quick-start)
3. [The instrument at a glance](#3-the-instrument-at-a-glance)
4. [Architecture & signal flow](#4-architecture--signal-flow)
5. [The three voice tracks](#5-the-three-voice-tracks)
6. [Page reference](#6-page-reference)
7. [The step sequencer (SEQ)](#7-the-step-sequencer-seq)
8. [The drum machine (DRM)](#8-the-drum-machine-drm)
9. [Recording: REC, LOC, motion & parameter locks](#9-recording-rec-loc-motion--parameter-locks)
10. [Song mode (SONG)](#10-song-mode-song)
11. [Mixing (MIX)](#11-mixing-mix)
12. [Effects (FX)](#12-effects-fx)
13. [Saving & presets (SAVE)](#13-saving--presets-save)
14. [Settings & MIDI (SET)](#14-settings--midi-set)
15. [MIDI implementation](#15-midi-implementation)
16. [The demo song: "LOW RES"](#16-the-demo-song-low-res)
17. [Specifications](#17-specifications)
18. [Troubleshooting](#18-troubleshooting)

---

## 1. Introduction

SID-DS turns a Nintendo DS into a three-voice **MOS 6581/8580 (SID)** synthesizer
with a full groovebox workflow: a 3-track by 32-step sequencer, a SID-flavoured
drum machine, motion recording, parameter locks, a 64-slot song arranger, a
9-channel mixer, delay & chorus, and 3-part multitimbral MIDI.

The sound engine is a register-level SID emulation running on the DS's second
CPU (the ARM7), leaving the main CPU free for the touch interface. Everything
you hear, including the drums, is produced by SID synthesis recipes:
combinable waveforms, ring modulation, hard sync, the one shared analog-style
filter, and the 6581's famous distortion.

**ROM variants**

| File | Contents |
|---|---|
| `sid-ds-1.0.1b.nds` | Full firmware **with** the built-in demo song |
| `sid-ds-1.0.1b-clean.nds` | Identical firmware, **empty** pattern bank |

---

## 2. Quick start

1. Copy the `.nds` ROM to your flashcart (or open it in an emulator such as
   melonDS) and boot it. Enjoy the C64 boot sequence.
2. **Hear the demo:** press **START** and the arrangement plays from the top.
   Watch the top screen: filter curve, voice activity, and the 3-track step
   strip all animate live. Press **START** again to stop.
3. **Play the keys:** the piano strip at the bottom of the touch screen is
   always live. The two thin sliders at its left are the **pitch wheel**
   (springs back to centre) and the **mod wheel** (vibrato depth); `+`/`-`
   shift the octave.
4. **Tweak the sound:** open **FLT** and drag on the XY pad. X is cutoff,
   Y is resonance. Try the other pages; every control is live.
5. **Make a beat:** open **DRM**, tap cells into the four lanes, press PLAY.
6. **Save your work:** open **SAVE**, tap a slot, tap SAVE. (The working state
   also autosaves to SD a few seconds after you stop editing.)

---

## 3. The instrument at a glance

### Top screen: the dashboard (display only)

- **Header:** the parameter you touched last, with its real value
  (e.g. `CUTOFF 5813 Hz, Q 2.5, LP`).
- **Filter curve:** the actual filter response, redrawn live, including while
  motion and parameter locks play back.
- **SID panel:** the selected track's waveform (TRI/SAW/PUL/NOI), pulse width,
  RING/SYNC state, chip model, LFO rate/depth bars, master volume, and the
  per-voice filter-routing flags (1 2 3). The WAV label wears the selected
  track's colour.
- **Step strip:** all three tracks of the sounding pattern, stacked and
  colour-coded (T1 **cyan** bottom, T2 **yellow**, T3 **green** top), with a
  white playhead column sweeping in time.
- **Status line:** playing pattern, transport state (`P3 SONG`, `P1 PATTERN
  LOOP`, `STOPPED`) and BPM.
- **Scope:** the actual audio output.

### Bottom screen: the touch interface

- **Tab row:** `FLT ENV MOD OSC SEQ DRM SONG FX MIX SAVE SET`
- **Page area:** the selected page's controls (all draggable).
- **Performance strip** (always present): pitch wheel, mod wheel, octave
  `+`/`-`, and the piano keyboard.

### Hardware buttons

| Button | Function |
|---|---|
| **START** | Transport: starts/stops the **song** from any page, except on the SEQ and DRM pages, where it starts/stops the **pattern** you're editing. |

Everything else is stylus-driven.

---

## 4. Architecture & signal flow

The signal path, from oscillators to output:

1. The **three SID voices** each generate a waveform through their own
   envelope and level.
2. Voices selected on the FLT page's routing buttons pass through the **one
   shared SID filter**; the others bypass it and stay bright.
3. The filtered and bypassed voices meet the **drum kit** at the mix bus.
4. The mix passes through the **6581 output drive** (when that model is
   selected), then **chorus**, then **delay**, and out to the speakers.

Voice details:

- Each voice: 24-bit phase accumulator oscillator with **combinable**
  waveforms (selecting TRI+SAW etc. ANDs them, like the real chip), pulse
  width, ring modulation (from the previous voice), hard sync, and a
  SID-accurate ADSR (the real chip's rate tables, including its famously
  snappy attack curve).
- **One filter for the whole chip**, just like the SID. The FLT page's
  `1 2 3` buttons choose which voices pass through it; the rest bypass.
  A classic SID arrangement filters the bass and leaves lead/harmony bright.
- **6581 vs 8580** (SET page): the 6581 model adds the non-linear cutoff
  curve and output-stage grit; the 8580 is linear and clean.
- Drums are **SID recipes**: LFSR noise bursts, triangle-sweep kicks, pulse
  ticks, not samples of a drum machine.

---

## 5. The three voice tracks

SID-DS is a **3-track mono-voice** machine, the classic SID tracker model:

| Track | Voice | Colour | Factory timbre |
|---|---|---|---|
| TR1 | SID voice 1 | **Cyan** | Saw bass |
| TR2 | SID voice 2 | **Yellow** | Pulse lead |
| TR3 | SID voice 3 | **Green** | Triangle harmony |

- Each voice has **its own timbre**: waveform, pulse width, ring, sync, and
  ADSR (edit them per voice with the V1/V2/V3 tabs on OSC and ENV).
- The **selected track** (the TRK box on SEQ, or the V-tabs) scopes note
  editing, the keyboard, the arpeggiator, and what the top-screen SID panel
  shows.
- Track colours follow you everywhere: the SEQ grid, the TRK box, the V-tabs,
  the dashboard strip, and the mixer's V1/V2/V3 faders.

---

## 6. Page reference

### FLT: filter
- **XY pad:** X = cutoff, Y = resonance. The dot and the top-screen curve track
  every move (including automation).
- **FILTER ROUTE 1 2 3:** which voices pass through the shared filter.
- **LP / HP / BP / NOTCH / OFF:** filter mode.

### ENV: envelope (per voice)
- **V1 V2 V3 tabs** select which voice you're editing.
- **ATK DEC SUS REL** sliders; the resolved SID rate step (0-15) is shown per
  slider. You are programming real SID envelope registers.

### MOD: modulation
- **RATE / DPTH:** the LFO (DPTH is also the mod wheel's target).
- **DEST:** LFO destination: cutoff, pulse width, or pitch.
- **GLID:** glide (portamento) time for slides.
- **ARP / ARPR:** reserved for chip-style arp modes (the playable arpeggiator
  lives on the SEQ transport).

### OSC: oscillator (per voice)
- **V1 V2 V3 tabs** select the voice.
- **WAVE:** TRI / SAW / PUL / NOI. Tap to toggle; multiple waves combine
  (AND), exactly like the SID.
- **PW:** pulse width. **RING:** ring modulation. **SYNC:** hard sync.

### SEQ / DRM / SONG / FX / MIX / SAVE / SET
Covered in their own chapters below.

---

## 7. The step sequencer (SEQ)

**16 patterns (P1-P16), 32 steps each, 3 tracks**, plus 4 drum lanes and 4
motion lanes per pattern.

### Layout
- **Pattern column** (left): `+`/`-` cycle the edit pattern P1-P16. While a
  pattern plays, selecting another **queues** it (shown in yellow); it lands
  when the bar wraps.
- **Step grid:** 16 steps per page; the **A/B** box switches between steps
  1-16 and 17-32. Step cells draw the note as a pitch bar in the selected
  track's colour; the playing step lights white.
- **Tapping a step** cycles its articulation: NOTE, SLIDE, TIE, REST.
  - **SLIDE** glides into the note (GLID time, MOD page).
  - **TIE** holds the previous note through the step (no retrigger).
- **Editor row** (edits the *selected* step):
  - **NOTE box:** the note name; tap its halves to nudge by semitone (works on
    empty steps too, creating the note).
  - **V:** velocity (tap to cycle levels).
  - **G:** gate length: `G25 / G50 / G75 / GFL` (25/50/75/full %).
  - **R:** ratchet: `R:1` to `R:4` retriggers inside the step.
  - **P:** probability: `P100 / P75 / P50 / P25` chance the step fires.
  - **T- / T+:** transpose the whole pattern by a semitone.
  - **TRK:** cycles TR1, TR2, TR3 (colour-coded); selects the track that
    the grid, keyboard and editor address.
- **Transport row:** `PLAY, BPM (40-240), LEN (1-32 steps), SWG (swing),
  ARP, REC, CLR`.
  - **ARP:** hold keys on the piano strip and they arpeggiate at step rate on
    the selected track.
  - **CLR:** clears the selected track's note lane in this pattern.

### Entering notes
- **REC off:** the keyboard just plays (jam along freely).
- **REC on (first tap):** keyboard taps **write** into the selected step and
  advance the cursor. If the pattern is *playing*, keys **live-record**
  quantized to the step.
- **REC again, now LOC:** parameter-lock mode (see chapter 9). Keys set the
  selected step's note *without* advancing.

### Follow mode
While playing, the SEQ and DRM grids follow the **sounding** pattern (song
mode advances it) and flip A/B pages with the playhead automatically.

### Copy / paste
`CPY` / `PST` on the **SAVE** page move the whole edit pattern (all tracks,
drums, and motion lanes) through the clipboard.

---

## 8. The drum machine (DRM)

Four TR-style lanes, 32 steps, per pattern:

| Lane | Sound | MIDI note (ch10) |
|---|---|---|
| KCK | SID kick (triangle sweep) | 36 |
| SNR | SID snare (noise + body) | 38 |
| HTC | Closed hat (LFSR noise tick) | 42 |
| HTO | Open hat (longer noise) | 46 |

- **Tap a cell:** off, hit, **ACCENT**, off.
- **Lane label:** mutes the lane. **S:** solos it.
- **Editor row** (for the selected cell/lane):
  - **A:** accent toggle. **R:** ratchet `R:1` to `R:4`. **P:** probability.
  - **T / D:** the selected lane's drum **TUNE** and **DECAY**. Drag to
    sweep. These are live synthesis parameters (sculpt the kit while it
    plays) and can be motion-recorded and p-locked.
- Same transport as SEQ; **CLR** clears the drum lanes.
- Per-drum **LEVEL** trims and the drum **master** live on the MIX page.

---

## 9. Recording: REC, LOC, motion & parameter locks

The REC button is three-state: **OFF, REC, LOC**, then back to OFF.

### Motion recording (REC, volca-style)
While the pattern **plays** with REC lit, *move any sound control* (the
filter pad, a slider, a drum T/D box, a mixer fader) and the movement is
recorded into one of the pattern's **4 motion lanes**, step by step, and
replays in a loop from then on. Steps that hold automation show a **yellow
dot**. Gaps hold the previous value (sample-and-hold, like a volca).

### Parameter locks (LOC, Elektron-style)
With **LOC** lit (and the transport stopped or running):
1. Tap a step to select it.
2. Move a control; its value is **locked to that step only**.
3. Keys set the selected step's note without advancing the cursor.

Locks live in the same 4 motion lanes. When the pattern plays, each step
re-applies its locked values; the on-screen controls (and the top-screen
dashboard) ride the automation so you can *see* the sound move.

**What can be motion-recorded / p-locked:** every live sound control:
cutoff, resonance, envelope, LFO, pulse width, wave*, glide, FX sends, SID
master volume, drum master, per-drum TUNE / DECAY / LEVEL.

> \* A wave lock changes **all three voices** (waveform motion is
> voice-agnostic); per-voice timbres are the OSC/ENV V-tabs' job.

---

## 10. Song mode (SONG)

A **64-slot chain** (4 pages of 16) arranges patterns into a song.

- **Tap a slot** to select it; **re-tap** to cycle its pattern.
- **Editor row** for the selected slot:
  - `-P#+` direct pattern pick.
  - `-X#+` repeat count x1 to x8.
  - **SYN / DRM** per-slot lane mutes (e.g. drumless intro, synthless break).
  - `-#/4+` chain page.
- **Transport:** `PLAY`, **`LOOP`**, `CLR`.
  - **LOOP lit:** the arrangement wraps forever.
  - **LOOP dim:** playback **stops by itself** after the last filled slot.
- **Live readout:** `NOW P4 2/4, NEXT P6`: current pattern, pass count,
  and what's coming. The playing slot is highlighted and the page follows it.
- **START** anywhere outside SEQ/DRM drives song transport.

Song playback plays both lanes of each pattern (synth + drums) with all
motion locks, per-slot mutes and repeats. The demo song is built entirely
from these tools.

---

## 11. Mixing (MIX)

Nine faders, grouped:

| Fader | Controls |
|---|---|
| **SYN** | SID master volume (the chip's volume register), motion-recordable |
| **V1 V2 V3** | **Per-voice levels** (a SID-DS extra; the real chip had no per-voice volume) |
| **DRM** | Drum kit master, motion-recordable and p-lockable |
| **KCK SNR HTC HTO** | Per-lane drum trims, p-lockable |

Faders track the live engine, so automation moves them on screen.

---

## 12. Effects (FX)

Post-mix, in series: **chorus into delay**.

- **DELAY:** `TIME, FBK, MIX`: tempo-flavoured echo, feedback, wet level.
- **CHORUS:** `RATE, DEP, MIX`: a subtle stereo-widening ensemble.

All six are live controls: automatable via motion recording and p-locks, and
addressable over MIDI (see the CC chart).

---

## 13. Saving & presets (SAVE)

A saved slot is a **complete machine state**: every parameter, the three
voices' timbres and levels, all 16 patterns (notes, flags, drums, motion
lanes), the song chain with repeats and mutes, kit tuning, mixer, BPM, swing,
octave.

- **16 slots.** Tap to select; `SAVE` / `LOAD` (confirmed); `NAME` opens the
  naming keyboard; `CLR` clears the slot (confirmed).
- **CPY / PST:** pattern clipboard (see chapter 7).
- **Autosave:** the working state persists to SD a few seconds after your
  last edit and returns at boot. Presets, slot names and MIDI settings live
  in their own files:

| File | Contents |
|---|---|
| `/sid-ds.dat` | working state (autosave) |
| `/sid-ds-pre.dat` | the 16 preset slots |
| `/sid-ds-nm.dat` | slot names |
| `/sid-ds-cfg.dat` | global MIDI settings |

---

## 14. Settings & MIDI (SET)

| Row | Function |
|---|---|
| **MIDI IN** | Base channel. Shows the 3-channel window (e.g. `CH 1-3`), or OMNI. |
| **MIDI THRU** | Echo incoming MIDI back out. |
| **CLOCK IN** | Follow external MIDI clock & transport. |
| **CLOCK OUT** | Send MIDI clock while playing. |
| **SYNC** | External-clock ratio: HALF / 1:1 / DBL. |
| **DRUM VOL** | Drum master (same as the MIX DRM fader). |
| **SID MODEL** | **6581** (gritty, non-linear) / **8580** (clean). |

MIDI I/O uses a slot-2 MIDI cartridge. `CART OK` in the header confirms
detection; `SD OK` confirms persistence.

---

## 15. MIDI implementation

### Channels (3-part multitimbral)

| Channel | Part |
|---|---|
| base (default 1) | SID voice 1 (mono) |
| base + 1 | SID voice 2 (mono) |
| base + 2 | SID voice 3 (mono) |
| 10 | Drum kit (GM notes 36/38/42/46) |
| OMNI mode | Everything plays the **selected track's** voice |

Notes, velocity and pitch bend are honoured per voice. With the arpeggiator
on, held notes on the selected track's channel feed the arp.

### Control changes

| CC | Parameter | | CC | Parameter |
|---|---|---|---|---|
| 74 | Filter cutoff | | 90 | Waveform mask (low nibble) |
| 71 | Resonance | | 102 | Filter routing (bits = voices) |
| 73 | Attack | | 103 | LFO destination |
| 75 | Decay | | 106 | Filter mode |
| 70 | Sustain | | 107 | Delay time |
| 72 | Release | | 108 | Delay feedback |
| 76 | LFO rate | | 109 | Delay mix |
| 77 | LFO depth | | 110 | Chorus rate |
| 79 | Glide | | 111 | Chorus depth |
| 85 | Ring mod | | 112 | Chorus mix |
| 86 | Hard sync | | 80 | Load preset (value = slot) |
| 87 | Pulse width | | 120/123 | All sound / notes off |
| 88 | Master volume | | 121 | Reset controllers |
| 89 | SID model | | | |

CCs address **all voices**; per-voice timbre editing is an on-screen
(or preset) affair. Performance CCs that MIDI files spam (mod wheel CC1,
portamento CC5/84, GM effect sends 91/93) are deliberately unmapped so a
random MIDI file can't detune your patch.

### Clock
With **CLOCK IN** on, the transport follows external start/stop and tempo (at
the SYNC ratio). **CLOCK OUT** makes SID-DS the master.

---

## 16. The demo song: "LOW RES"

The factory ROM boots with a complete bass-music arrangement (A minor,
128 BPM) that exercises the whole machine. Study it, then wreck it:

- **P1** sub intro: T1 bass alone, T3 pad breathing in, baked filter sweep.
- **P2** groove: wah bass (cutoff motion), T2 offbeat octave stabs.
- **P3** acid line: slides + resonance locks, T3 drone underneath.
- **P4** stomp: PWM hook on T2 (pulse-width motion).
- **P5** build: contrary motion, bass walks down while the lead climbs.
- **P6** the drop: all three voices, doubled walk, harmony thirds.
- **P7** drum break: a tuned-kick tom run, ghost snares, breathing hats and
  a two-stage snare roll, built **entirely from drum p-locks**.
- The SONG chain arranges these across 14 slots with repeats and per-slot
  mutes (drumless intro, synthless break), then wraps.

Open SEQ/DRM while it plays and watch follow mode page through it; open FLT
and watch the locks drive the filter.

---

## 17. Specifications

| | |
|---|---|
| Synthesis | 3-voice SID (6581/8580 models), register-level emulation on the ARM7 |
| Waveforms | Triangle, saw, pulse (PWM), noise; combinable per voice |
| Modulators | Ring mod, hard sync, 1 LFO (cutoff/PW/pitch), glide |
| Filter | 1 shared multimode (LP/HP/BP/NOTCH), per-voice routing, model-correct drive |
| Envelopes | Per-voice ADSR using the SID's rate tables |
| Drums | 4-lane SID-recipe kit, per-drum tune/decay/level |
| Sequencer | 16 patterns, 32 steps, 3 synth + 4 drum lanes |
| Per step | Velocity, gate (25-100%), ratchet (1-4), probability (25-100%), slide, tie, accent (drums) |
| Automation | 4 motion lanes per pattern; motion recording + parameter locks |
| Song | 64 slots (pattern, x1-8 repeats, lane mutes), loop on/off |
| Mixer | SID master + 3 voice levels + drum master + 4 lane trims |
| FX | Chorus + delay (post-mix) |
| MIDI | 3-part multitimbral in + ch10 drums, clock in/out, CC map, THRU |
| Storage | 16 full-state preset slots + autosave (SD via flashcart) |
| Tempo | 40-240 BPM, swing |

---

## 18. Troubleshooting

- **No sound?** Check the MIX faders and the SID master (SYN), and that the
  track you're playing isn't routed into a closed filter (FLT page, mode OFF
  or cutoff at zero with that voice routed).
- **Keyboard writes notes when I just want to jam:** REC is on; tap it until
  it's dark.
- **A pattern changed sound mid-bar:** that's a motion lane or p-lock
  (yellow dots on the steps). Clear the pattern (or copy the notes to a fresh
  pattern) to shed automation.
- **MIDI plays only one voice:** set MIDI IN to a base channel (`CH 1-3`) and
  send your parts on base/base+1/base+2. OMNI funnels everything into the
  selected track on purpose.
- **`SD NO` in SET:** your flashcart's DLDI doesn't expose writable SD;
  presets will live for the session only.
- **Stuck note from external gear:** press STOP; the transport stop doubles
  as an all-notes-off panic.

---

*SID-DS is a fan-made homebrew instrument. MOS 6581/8580, Commodore 64 and
their look-and-feel are the property of their respective rights holders; this
project is an affectionate tribute, not an official product.*
