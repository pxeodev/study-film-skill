# Production Checklist

Use this after the first HTML draft exists and before you call the film
finished.

## 1. Lock narration first

- Write the narration as its own script
- Generate the actual audio take
- Measure the runtime
- Treat the audio as the fixed clock for the final cut

Do not spend hours tuning visual `dur` values against imagined delivery.

## 2. Review mode and capture mode are different

- Review pacing on the normal page with narration enabled
- Capture on `?autoplay`
- Keep `?autoplay` silent so the final mux always uses the clean narration track

If the film feels wrong with audio, fix the storyboard or the scene timings
before you record anything.

## 3. Contradiction check

Scan the full film for:

- term drift: signal vs filter vs carry vs trade
- number drift: percentages, profitable counts, dates, hold windows
- policy drift: stop, cooldown, and max-hold values
- provenance drift: statements stronger than the evidence supports
- packaging drift: title/thumbnail selling a side point instead of the core conflict

If a claim is directionally true but not system-of-record verified, soften it.

## 4. Timing check

- Compare total scene duration against the narration runtime
- Watch long scenes for internal drift, not just start/end sync
- If needed, transcribe the narration and place cut points on real spoken cues
- Remove narration-strip text that only repeats what the frame already says

## 5. Capture fallback

Use headless Playwright first when it keeps time correctly.

If headless timing drifts because:
- long animations miss beats
- timers fire late
- a heavy scene stalls under headless load

then record the real browser in the foreground and capture video only. Mux the
locked narration afterward.

For one-shot final masters, real-browser capture is acceptable.

## 6. Thumbnail rules

- One dominant contrast
- No wall of text
- No cropped headline words
- Verify at `320x180`, not just full size
- Comparison scenes should read in under a second

Good default:
- left pain or mistake
- right filter / skip / method
- one small badge for context

## 7. Ship pack

Minimum useful deliverables:

- `index.html`
- final video (`.mp4`)
- `thumbnail.png`
- `captions.en.srt`
- optional `captions.en.vtt`
- upload title
- short description
- pinned comment

If the workflow is the product, use the pinned comment to invite viewers to
make their own study film from a Claude or Codex coding session.
