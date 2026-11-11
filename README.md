# Nail the Pitch

A real-time vocal pitch monitor. Sing, and the note you are actually on is
drawn on a scrolling piano roll against the note you are aiming for.

**[Open the app](https://kushawng.github.io/nail-the-pitch/)**

Unlike most pitch monitors, the tuning reference is yours to choose:
**A = 440 Hz**, **A = 432 Hz**, or anything from 380 to 500 Hz.

## Using it on a phone

Open the link in Safari or Chrome and tap **Start listening**; the browser
will ask for microphone access. To keep it one tap away, use
Share → **Add to Home Screen** — it then launches full-screen with its own
icon.

A microphone needs a secure page, which is why this is served over HTTPS.

## What it does

- Live pitch tracking on a piano roll, coloured by how far off the note you are
- The view follows your voice: it centres on you when a phrase starts and keeps
  you on screen through leaps, so you never scroll to find yourself. Drag up or
  down to look around and it stands down for a moment, then resumes; drag
  sideways to scroll back through what you sang, and **Back to live** returns
- Note name, cents sharp or flat, measured frequency and target frequency
- A reference keyboard that sounds notes **at your chosen tuning**
- Record a take, play it back, export the melody as MIDI or the notes as CSV
- Takes are saved in the browser, each remembering the tuning it was sung at

## About the tuning reference

The detector reports hertz and knows nothing about tuning. Every conversion
from hertz to a note name is anchored on one number — the frequency of A4:

```
midi = 69 + 12 * log2(frequency / referenceA4)
```

Two consequences worth knowing:

**Takes are stored as frequencies, not note numbers.** Changing the reference
re-reads a recording rather than rewriting it. A take sung against A = 440
reads as consistently about 32 cents sharp once you switch to A = 432 — which
is exactly what it was.

**MIDI export compensates.** MIDI note numbers carry no absolute pitch, and
nearly every instrument reading them assumes A = 440, so a melody sung at 432
and exported by note name would play back 32 cents sharp of what was sung.
The exported file carries a pitch bend that offsets the whole track. At
A = 440 no bend is written.

## How the pitch detection works

Each 2048-sample window (46 ms at 44.1 kHz) is run through a normalised
square-difference function — the McLeod Pitch Method — with the
autocorrelation term computed via FFT.

The accuracy comes from the peak picking. Taking the tallest NSDF peak
reports the octave above on harmonically rich voices, so the detector takes
the *earliest* peak within 85% of the tallest. That search starts at lag 1
rather than at the shortest in-range lag: for a high note the shortest
in-range lag already sits past the fundamental's own peak, and starting there
finds its second multiple and reports the note an octave low. Parabolic
interpolation around the chosen peak recovers sub-sample resolution, which is
what keeps vibrato visible rather than quantised away.

Against synthetic tones the worst error across D2–C7 is 0.04 cents.

## Limits

- Monophonic: one voice at a time, chords are not tracked.
- Tuned for voice, roughly 62 Hz to 2300 Hz.
- Analysis runs at the browser's animation frame rate, so the trace is a
  little coarser through fast vibrato than a native build would be.

## Layout

One self-contained HTML file with no build step and no dependencies.

```
index.html        the whole app
assets/icon.png   home-screen icon
```
