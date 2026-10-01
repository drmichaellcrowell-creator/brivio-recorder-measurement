# Silent playback on the standalone recorder measurement page

**Status:** open. One symptom observed, no cause demonstrated. Stage 2 remains on hold.

Scope of this record: the isolated measurement page only. No application change,
no deployment, no production probe.

---

## 1 · Evidence, preserved as distinct items

Each item below is one observation. They are kept separate because they were
produced by different runs, different instruments, and in one case by a person
listening rather than by a measurement.

### E1 — Profile B run (machine-reported)

| field | value |
|---|---|
| device | iPad Air, iPadOS 26.5.2, Chrome |
| profile | Live Conversation recorder settings (standalone microphone approximation) |
| wall-clock duration | 182.858 s |
| exact bytes | 4,999,701 |
| Blob type | `audio/webm; codecs=opus` |
| timeslice | `start()` — none |
| chunks delivered | 1 |
| `audioBitsPerSecond` | 192000 **(a declared setting)** |
| microphone track | 48,000 Hz **(a declared setting)** |

### E2 — Profile B listening observation (human)

The operator pressed play. **The playback timer advanced; no sound was heard.**
Output was confirmed to be routed through the iPad speaker.

E2 is a listening observation, not a measurement. It establishes *silent
playback*. It does **not** establish that the encoded file contains silence.

### E3 — Profile A run (machine-reported)

| field | value |
|---|---|
| device | same iPad Air, iPadOS 26.5.2, Chrome |
| profile | Speaking-task recorder settings |
| wall-clock duration | 15.236 s |
| exact bytes | 359,311 |
| Blob type | `audio/mp4; codecs=mp4a.40.2` |
| timeslice | `start(250)` |
| chunks delivered | 15 |
| `audioBitsPerSecond` | 192000 **(a declared setting)** |

### E4 — Profile A listening observation (human)

Playback had **audible speech**.

### E5 — Voice Memos control (human)

A Voice Memos recording on the same device recorded and played the operator's
voice audibly.

**What E5 does and does not establish.** It establishes that the device's
microphone hardware, its speaker, and the system's own record/playback path all
work. It does **not** establish that *browser* microphone capture worked during
E1: Voice Memos uses a different audio stack, a different session, and a
different time.

### E6 — Housekeeping

The second Profile B report the operator sent **corrected the device metadata
fields only. It was not another recording.** There are two recordings in this
evidence set, not three.

---

## 2 · What differs between E1 and E3

Four variables change at once. No single-variable comparison exists yet.

| | E1 (silent) | E3 (audible) |
|---|---|---|
| container / codec | WebM / Opus | MP4 / AAC |
| timeslice | none — `start()` | `start(250)` |
| duration | 182.858 s | 15.236 s |
| chunks | 1 | 15 |
| run order | first | second |

Any of the four, or an interaction, could account for the difference. **Opus is
not established as the cause**, and nothing here supports a production codec
change.

---

## 3 · Source findings

### 3.1 Exact revision of the deployed page

| | |
|---|---|
| repository | `drmichaellcrowell-creator/brivio-recorder-measurement` |
| commit | `97b205f1146b3d319fb72250590afefc03c8d09a` (2026-09-30, "Rename index (1).html to index.html") |
| blob | `998f5f05ef026208a8eb947a72a13c42ee17438a`, 30,910 bytes |
| content sha256 | `9c2dbdf066ecd5b421042a80d2975b7460624f72a44b44c5cb83a88ef60e30e1` |
| cited application source | `b5fe49566b647cd5e9d22fd164a615eefd1a534b` |

The deployed file is **byte-identical** to the copy prepared and reviewed before
hand-over. No drift. E1 and E3 were produced by exactly this code.

### 3.2 Code paths inspected, and what is sound

- **Chunk assembly** is correct. A single array, pushed only when `size > 0`,
  and the Blob built from that array. With no timeslice, one chunk is expected;
  one was delivered. E1's chunk count is consistent, not anomalous.
- **Track stopping** happens *inside* `onstop`, after the final
  `dataavailable`, so it cannot truncate the recording.
- **Blob type** is `recorder.mimeType` first, falling back to the requested
  value. E1 and E3 both show the live recorder's own value, not an assumption.
- **Format selection** matches the cited source revision for both profiles:
  candidate order, `{ mimeType }` passed unconditionally for the Live profile,
  `start()` versus `start(250)`, and the Live profile's refusal to record when
  no candidate is supported.

### 3.3 Four gaps in the deployed page — demonstrated, by reading it

These are properties of the measurement instrument, not causes of the symptom.
Each one is a reason the instrument could not localise E2.

1. **No `recorder.onerror` handler.** The page installs none. An encoder
   failure mid-recording would be swallowed, and the page would still report a
   plausible size and duration.
2. **No track-`ended` monitoring.** If the microphone track ended early — an
   interruption, the app backgrounded, the device sleeping — the page would
   report the full wall-clock duration with no indication that the source died.
   For a 3-minute recording on a tablet this is a live possibility.
3. **Duration is wall-clock, not media duration.** `Date.now()` deltas measure
   how long the button was held, never how much audio is in the file. "182.858 s"
   is a button-press interval.
4. **Encode support and decode support were never asked separately.** The page
   gated on `MediaRecorder.isTypeSupported(t)` — *can this browser encode t?* —
   and never asked `HTMLMediaElement.canPlayType(t)` — *can this browser decode
   t?* These are different questions with different answers, and the second one
   is the cheapest available discriminator for exactly the symptom in E2.

### 3.4 Comparison against the cited application source

No application file was changed; this is a read-only comparison.

The application's Live Conversation capture recorder **does** install
`recorder.onerror`, and **does** watch its source track's `ended` event in order
to record a truthful partial result. The measurement page does neither. So on
the two dimensions most likely to explain a silent recording, **the measurement
page is a weaker instrument than the application it was built to characterise.**

The third difference remains as stated from the outset: the application records
a live room's already-published call track; the page records a standalone
microphone. No standalone page can substitute for that source.

---

## 4 · Arithmetic, and what it does not settle

| run | rate | declared setting |
|---|---|---|
| E1 | 27,342 bytes/s = **218.74 kbit/s** | 192.00 kbit/s |
| E3 | 23,583 bytes/s = **188.66 kbit/s** | 192.00 kbit/s |

Both runs produced data at approximately the declared bitrate.

**This does not establish that either file contains sound.** A constant-rate
encoder emits its full bitrate for silence as readily as for speech. The
inference "a file this large cannot be silent" is only valid for a
variable-rate encoder, and which encoder this device used is unknown. Size is
not evidence of signal, and this record does not treat it as such.

---

## 5 · Hypotheses — none demonstrated

Listed to be tested, not believed. Each names the result that would support it
and the result that would kill it.

- **H1 · Rendering path.** The file carries usable audio but this device's
  playback path does not render it. *Consistent with:* decoded samples non-silent
  while the operator hears nothing. *Weakened by:* decoded samples silent.
  Note that `canPlayType` bears on this only as a **capability indication**; it
  is not a playback test, and a `""` result is not a demonstration that playback
  failed for this file.
- **H2 · No usable signal in the file.** *Consistent with:* decoded samples at
  the noise floor. **Where** the signal was lost — capture, encoding, or
  somewhere else — is a further question that decoding alone does not answer;
  the microphone meter and the track observations are what narrow it.
- **H3 · Source ended or muted early.** *Consistent with:* a track-`ended` or
  `mute` event, or a per-second decoded profile that goes quiet partway.
  *Weakened by:* an uninterrupted track with signal throughout.
- **H4 · Something associated with the timeslice.** *Consistent with:* WebM/Opus
  audible at 250 ms and not audible with no timeslice, codec held constant.
  Such a result would establish an **association with that setting in those
  runs** — not a finalisation defect, and not a cause.
- **H5 · Something associated with duration.** *Consistent with:* the same
  settings audible at 15 s and not at 3 minutes.
- **H6 · Format-related difference.** The existing pair — one audible AAC run,
  one inaudible Opus run — is *suggestive of* a format-related difference. It
  proves nothing: three other variables moved with the format.
- **H7 · Instrument or environment.** The first run failed for a reason specific
  to that moment. Addressed by repetition.

H1 and H2 are the first split worth resolving, and one test distinguishes them.
Neither outcome, on its own, localises a cause.

### Interpretation rules this investigation follows

These are constraints on what may be concluded, not findings.

1. **Non-silent decoded samples** establish that the decoder produced a signal.
   They do not establish that the file is correct: the signal could be noise,
   corruption, or incomplete audio. **Audible, intelligible speech is a separate
   operator observation** and is recorded as such, per run.
2. **Decoded silence** does not establish a capture failure. It establishes that
   the decoded samples are silent. Silence may originate in capture, in
   encoding, or elsewhere in the pipeline; the microphone meter and the track
   observations are what narrow it.
3. **A decode failure** does not establish that the device cannot decode what it
   encoded. The output may be malformed or incomplete, or the Web Audio decoding
   path may simply differ from the playback element's path. `canPlayType()` is a
   capability indication, not a playback test. These three observations are kept
   separate.
4. **A difference between runs** is an association with a setting in those runs.
   It is never, by itself, a demonstrated cause.
5. A nonzero size, an advancing timer and a successful `isTypeSupported()`
   result are not evidence of audible audio and are excluded from every verdict.

---

## 6 · What the diagnostic revision adds

- Container/codec and timeslice chosen **independently**, so one variable moves
  at a time. An unsupported combination is **refused, never substituted**.
- Microphone signal measured live from the capture stream (peak, RMS, and peak
  per second), **off by default**. The tap uses an `AudioContext`, which on a
  tablet can itself alter the audio session, so the original symptom is observed
  before anything changes that session. Whether the meter was on is recorded with
  every run, and a verdict never treats an unmeasured meter as a flat one.
- `recorder.onerror`, `onstart`/`onpause`/`onresume`/`onstop`, every
  `dataavailable` with its size and timestamp, and the source track's
  `ended`/`mute`/`unmute` events — closing gaps 1 and 2.
- Track `readyState`, `muted`, `enabled` captured at start **and** at stop.
- Playback state: `error` with its code name, `muted`, `volume`, `paused`,
  `readyState`, `networkState`, `duration`, `currentTime`, and the event sequence.
- **Encode support and decode support asked separately** — closing gap 4.
- An explicit **operator listening observation** per run — intelligible speech,
  sound but not intelligible, or no sound — recorded before decoding and labelled
  in the report as a human observation, not a measurement.
- **Decoded-sample analysis** in memory: channels, sample rate, decoded
  duration, peak, RMS, and a per-second peak profile. It separates three
  outcomes the previous page conflated, and states the limits of each:
  - decode succeeded, samples silent → *decoded samples are silent*, followed by
    the microphone and track observations that narrow where the silence arose;
  - decode succeeded, signal present → *the decoder produced a signal*, which is
    not a claim that the audio is correct or intelligible;
  - decode failed → *contents unknown*: not silence, not success, and not proof
    that the device cannot decode the format.
- A verdict computed **only** from decoded samples, capture health, the operator
  observation and playback state. Size, elapsed timer and `isTypeSupported` are
  excluded by rule, and the page says so — as it also says that comparisons
  between runs show association, never cause.
- The page's own order of work: record → **play and listen** → note what was
  heard → decode. Listening comes first because decoding cannot replace it.

Both original profiles are preserved and still labelled as before. Diagnostic
variants are labelled separately and are explicitly **not** the application's
settings.

### Local verification

Headless Chromium with a synthetic microphone: all checks pass. The listening
section precedes the decoding section; the meter is off by default and the
baseline run records capture activity as *unmeasured* rather than flat; WebM/Opus
honoured at both timeslices (1 chunk versus 11); MP4/AAC **refused explicitly**
in that browser rather than substituted; the meter measures when explicitly
enabled; the operator observation appears in both the verdict and the report,
labelled as human; playback state read back; no off-device request, no browser
storage, no download affordance, no page error.

Against a **silent** synthetic microphone the same page reported `DECODED SAMPLES
ARE SILENT`, peak `-inf dBFS`, and — checked explicitly — declined to locate the
cause, named capture, encoding and elsewhere as possibilities, asserted no
capture failure, and called capture activity unknown because the meter was off.
The discriminator is demonstrated in both directions, and so is its restraint.

---

## 7 · Not established

- No cause of E2.
- No conclusion that Opus, the timeslice, the duration or the run order is
  responsible. The existing pair of runs is suggestive of a format-related
  difference and proves nothing, because three other variables moved with the
  format.
- No recommendation of any production codec change. The evidence does not
  support one, and a measurement-page symptom would not be sufficient grounds
  for one even if it were reproduced.
- No equivalence between any measurement here and the live-room published
  track.
- Stage 2 remains deferred; the synthetic payload size is still unchosen.
