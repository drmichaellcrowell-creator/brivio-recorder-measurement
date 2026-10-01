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

---

# Round 2 — three controlled runs, and the application paths

## 8 · Evidence, preserved as separate observations

iPad Air, iPadOS 26.5.2, Chrome. **Device identification comes from the
operator's confirmation: the device fields in these three reports were left
blank.** In every run the operator played and listened **before** decoding.

| id | container / codec | timeslice | bytes | wall clock | chunks | decoded | heard |
|---|---|---|---|---|---|---|---|
| **E7** | WebM / Opus | none | 451,399 | 16.655 s | — | signal present | **no sound** |
| **E8** | MP4 / AAC | none | 383,403 | 16.354 s | — | signal present | **intelligible speech** |
| **E9** | WebM / Opus | 250 ms | 489,566 | 18.027 s | 18 | signal present | **no sound** |

Meter off in all three, so no `AudioContext` touched the audio session before
the symptom was observed. Each is a separate observation; none is a repeat of
another.

### 8.1 What these three runs do establish

Comparisons, each with one variable moving:

- **E7 vs E9** — container/codec held, timeslice varied (`start()` versus
  `start(250)`, 1 implicit chunk versus 18). **Both inaudible.** Changing the
  timeslice did not resolve the symptom. **[CORRECTION]** An earlier draft said
  this "retires" the finalisation hypothesis. It does not. It rules out *this
  one* timeslice change as a remedy; other finalisation-related mechanisms —
  how the container is muxed or finalised, what the last chunk carries, how the
  stream is terminated — remain untested and are not excluded.
- **E7 vs E8** — timeslice held at none, duration within 0.3 s, container/codec
  varied. **WebM inaudible, MP4 audible.**
- **Run order** was E7, E8, E9. The audible run sits *between* two inaudible
  ones. **[CORRECTION]** An earlier draft said order "does not account for" the
  difference. More precisely: the alternation makes a simple monotonic order
  effect — a first-run failure, or progressive degradation — an unlikely single
  explanation, and so **strengthens the association with format**. It does not
  eliminate every order effect, nor every audio-session effect: a per-format
  session interaction, or an effect that resets between runs, would produce the
  same alternation.
- **Duration** is now controlled: 16.4–18.0 s across all three, against the
  original 182 s. Duration is not required to produce the symptom.

So, on this device, with timeslice, duration and order controlled: **the WebM
output is decodable by the Web Audio path but inaudible through the media
element, while the MP4 output is both decodable and audible.** The decoded
signal in E7 and E9 means the inaudibility is not explained by an absence of
signal in the file.

### 8.2 What they do not establish

- **Not a universal Opus defect.** One device, one OS version, one browser,
  three runs. Nothing here speaks to other devices, other iPadOS versions, or
  other browsers.
- **Not a cause.** The association is with the container/codec in these runs.
  The mechanism — container muxing, the Opus decoder in the element's path,
  something else entirely — is not identified.
- **Not equivalence to the live-room track.** Every run used a standalone
  microphone. The Live Conversation recorder wraps a room's already-published
  call track, which no standalone page reproduces.
- **Not a size argument either way.** Measured rates: E7 216.8 kbit/s,
  E8 187.6 kbit/s, E9 217.3 kbit/s. MP4/AAC was the *smaller* of the two
  formats here, so preferring it would not be a size regression on this device.

### 8.3 Playback state — supplied for all three runs

| field | E7 (WebM, silent) | E8 (MP4, audible) | E9 (WebM, silent) |
|---|---|---|---|
| `element.error` | none | none | none |
| `muted` | false | false | false |
| `volume` | 1 | 1 | 1 |
| `readyState` | 4 | 4 | 4 |
| `canPlayType` | probably | probably | probably |

Identical across audible and inaudible runs. The element **did not error**, had
loaded enough data to play (`readyState` 4), was neither muted nor at zero
volume, and reported `probably` for the very files that produced no sound.

**[CORRECTION]** An earlier section of this record said that making the player
use the stored MIME type "would turn a silent failure into an honest message".
**That was wrong, and these fields show why:** `canPlayType` returned `probably`
for the inaudible files. A capability check gated on it would have passed and
displayed nothing. P1 and P3 below remain worth doing for other reasons, but
neither would have surfaced *this* failure.

So the failure is silent at **every** programmatic surface the application can
read. There is no error, no state, and no capability answer that distinguishes
an inaudible recording from a working one. No further information is needed from
the operator on this point.

## 9 · Application paths — source findings

Read-only inspection of `main`. No application file was modified.

### 9.1 Two capture paths, opposite preferences

| | speaking-task recorder | Live Conversation capture |
|---|---|---|
| first candidate | `audio/mp4;codecs=mp4a.40.2` | **`audio/webm;codecs=opus`** |
| second | `audio/mp4` | `audio/webm` |
| then | `audio/webm;codecs=opus`, … | `audio/mp4`, `audio/ogg;codecs=opus` |

`CAPTURE_MIME_CANDIDATES` is defined at `src/lib/liveVoiceCaptureState.js:173`.

On the measured device WebM/Opus is encodable, so the Live Conversation path
would select it. E7 and E9 are the measurements of that selection's output on
that device. The speaking-task path selects MP4/AAC on the same device — which
is E8, the audible one.

### 9.2 The application never asks whether it can play what it records

`canPlayType` appears **nowhere** in `src/`. Capture-format selection is gated
solely on `MediaRecorder.isTypeSupported` — *can this browser encode it?* —
which the three runs show is a different question from whether the result is
audible through a media element.

### 9.3 The player receives the recorded MIME type and discards it

`src/components/audio/PrivateAudioPlayer.jsx` destructures a `mimeType` prop at
line 19 and **never references it again** — one occurrence in the whole file.
The element is rendered with `src={signedUrl}` only: no `<source type=…>`, no
capability check. Ten call sites pass it, including the teacher review panel
(`StudentEvidencePanel`), student work display, and the room transcript modal.
The information needed for a check is already stored on the recording and
already threaded into the player. It is simply unused.

### 9.4 The error path cannot distinguish undecodable from expired

`handleAudioError` responds to *any* element error by minting a fresh signed URL
once and retrying, then falling back to the message "Recording could not be
loaded." An unsupported-source error would be retried as though the URL had
expired, and then reported as a generic load failure.

**And the more likely case here is worse.** In E7 and E9 the playback timer
advanced, which is consistent with the element not firing `error` at all. If it
does not error, `onError` never runs, no retry happens, no message appears — a
teacher sees a normal-looking player producing silence, with nothing anywhere
indicating a problem. §8.3's unread fields would settle which of the two is
happening.

## 10 · Candidate mitigation — AAC first

**This is a candidate mitigation, not a fix with a known mechanism.** It does
**not** guarantee audible recordings, on the measured device or any other, and it
does **not** establish broader review-device compatibility — no evidence here
speaks to any device other than the one measured. The mechanism of the WebM
inaudibility is still unidentified, so this changes which format is attempted
first without knowing why the other one failed.

**Proposed:** reorder `CAPTURE_MIME_CANDIDATES` at
`src/lib/liveVoiceCaptureState.js:173` so MP4/AAC is preferred, matching the
order the speaking-task recorder already uses in production:

```js
export const CAPTURE_MIME_CANDIDATES = Object.freeze([
  'audio/mp4;codecs=mp4a.40.2',
  'audio/mp4',
  'audio/webm;codecs=opus',
  'audio/webm',
  'audio/ogg;codecs=opus',
]);
```

One frozen array. No change to selection logic, to the refusal behaviour when
nothing is supported, to the recorder's options, to `start()`, to finalisation,
or to any entity.

### Why this and not something larger

- The fallback chain is preserved, so a device that cannot encode AAC still
  records: capture availability does not narrow.
- The order is not novel. It is the order the speaking-task recorder already
  ships, and E8 is that order measured as audible on the affected device.
- It is the order the speaking-task recorder already ships, so it narrows the
  divergence between the two capture paths rather than introducing a third
  behaviour. No claim is made that MP4/AAC is more compatible for review on
  devices that were not measured.
- It was not a size regression in these measurements — AAC came out smaller.

### What this fix is NOT justified by, stated plainly

It is **not** justified by a demonstrated universal Opus defect, and it does not
claim one. It is justified by: a reproducible inaudibility on a classroom device
with the timeslice, duration and run order controlled; the fact that nothing the
application can read distinguishes the inaudible files from working ones (§8.3);
and the availability of an order already in production that was audible on that
same device.

It also carries an unresolved gap: **every one of these recordings used a
standalone microphone.** None of them establishes what the Live Conversation
recorder's actual source — a room's already-published call track — produces, in
either format. The mitigation is aimed at a symptom measured on a different
source, and only a real Live Conversation capture can test it.

If a second device, or that real capture, contradicts the picture, this should be
re-opened rather than defended.

### Deliberately excluded from "smallest"

Three real defects found above are **not** in this proposal, and each should be
decided separately:

- **P1 — the player discards `mimeType`** (§9.3). Worth correcting on its own
  terms, but per §8.3 a `canPlayType` gate would **not** have caught this
  symptom: it answered `probably` for the inaudible files.
- **P2 — the error path conflates undecodable with expired** (§9.4).
- **P3 — capture selection ignores playability** (§9.2). Gating on `canPlayType`
  at capture time is tempting but is *not* obviously correct: the capture device
  is not necessarily the review device, and in this case `canPlayType` would have
  endorsed the inaudible format anyway.

P1–P3 are worth doing, but §8.3 shows they would not have made *this* failure
visible: there was no error to report and the capability answer was wrong. Only
the ordering change bears on the recordings being inaudible, and it bears on it
as a mitigation rather than a understood fix. They are complementary, not
alternatives.

## 11 · Validation plan for the proposed fix

Nothing below has been executed, and no application change has been made.

**Offline, before any deployment:**

1. Extend `test/liveVoiceCaptureModel.test.js` to assert the candidate order
   explicitly, so a future reorder cannot pass silently: MP4/AAC first, and the
   full list in the stated sequence.
2. A selection test per device profile: a fake `isTypeSupported` that accepts
   only AAC, only WebM, both, and neither. Assert AAC chosen when available,
   WebM chosen when AAC is not, and the existing **refusal** preserved when
   nothing is supported — that last one is the regression most worth guarding.
3. Confirm in `test/liveVoiceCaptureMounted.test.jsx` that the recorder still
   constructs with `{ mimeType }` passed unconditionally and still calls
   `start()` with no timeslice. The fix must not disturb either.
4. Confirm the stored `mime_type` follows the selection, so review metadata
   stays truthful.

**On device, after the patch is reviewed and with explicit approval.** No
further standalone-microphone recording: E8 already gives an audible MP4/AAC
result with no timeslice, and repeating it would add nothing.

5. **A real Live Conversation capture on the affected iPad, played back through
   the application's own review path**, listened to by the operator. This is the
   first and only test of the actual source — the room's published track — and
   of the real playback surface. Confirm the stored `audio_mime_type` is the AAC
   type and that the recording is audible.
6. One non-iPad device doing the same, so the change is not validated solely
   where the symptom appeared.

**Stopping rule:** if step 5 is inaudible, the ordering change was not the
mitigation and the investigation re-opens at §9.3/§9.4 rather than proceeding.
If step 5 is audible, that establishes the mitigation worked **on that device
for that capture** — not that the mechanism is understood.

## 12 · Still not established

- No mechanism for the WebM inaudibility.
- No universal Opus defect; no claim about any device other than the one measured.
- No equivalence between any of these measurements and the live-room published
  track.
- Stage 2 remains on hold. The measured rates imply roughly 4.0–4.7 MiB for
  three minutes on this device, which is recorded here as an observation and is
  **not** a chosen payload size.
