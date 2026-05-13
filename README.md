# Study Film

A Claude Code skill that turns coding sessions into narrated videos for YouTube and social media.

<p align="center">
  <img src="https://img.shields.io/badge/Claude_Code-skill-D4A843?style=flat-square" alt="Claude Code Skill">
  <img src="https://img.shields.io/badge/output-HTML_film_%2B_MP4_export-4ADE80?style=flat-square" alt="HTML Film and MP4 Export">
  <img src="https://img.shields.io/badge/narration-ElevenLabs-9CA3AF?style=flat-square" alt="ElevenLabs Narration">
</p>

## What it does

After a coding session, say `/study` or "make a film from this session." The skill reads your conversation history, extracts the narrative arc (setup, corrections, discoveries, ship), and generates a narrated video with:

- **Terminal animations** — line-by-line reveals matching Claude Code's tool calls
- **AI narration** — script generated from session, voiced via [ElevenLabs](https://try.elevenlabs.io/ots0dbiulx3y)
- **Audio-locked pacing** — scene timing is tuned to the actual narration, not rough word counts
- **1080p export** — recorded headless via Playwright when stable, or via real-browser fallback
- **Sound design** — transition, typing, caution, and celebration sounds via [SND](https://github.com/nicklasserra/snd-lib)
- **Canon palette** — single editorial palette on a three-step surface ladder
- **YouTube-ready** — 1920x1080, thumbnail generator, captions, and metadata template

Output pipeline: HTML film → narration lock → browser review → silent capture → ffmpeg mux → captions + thumbnail → upload.

## Example

> "Three corrections. One session. It shipped."

The skill structures every session as a story: what you asked for, where the AI got it wrong, what was discovered along the way, and what shipped at the end. User corrections are the dramatic hooks — they're quoted verbatim, typos and all.

## Quick Start

```bash
# Install
cp -r . ~/.claude/skills/study-film/

# In any Claude Code session
/study

# Export to video
# 1. Generate narration (ElevenLabs or any TTS)
# 2. Pace the HTML to the actual audio
# 3. Review in browser with narration on
# 4. Capture the silent ?autoplay version
# 5. Mux audio + video with ffmpeg
# 6. Ship thumbnail + captions
# See SKILL.md and references/production-checklist.md
```

## Pipeline Overview

```
Session → /study → HTML film + narration script → narration audio lock → review page
                                          ↑                              ↓
                                   retime scenes if needed      silent capture / fallback capture
                                                                     ↓
YouTube ← title/thumbnail/captions ← MP4 mux ← silent `?autoplay`
```

### Voice Narration

The skill generates a narration script from the session. For the actual voice, [ElevenLabs](https://try.elevenlabs.io/ots0dbiulx3y) produces the most natural results. Their `eleven_v3` model with a narrator-style voice works well for this format. Use ellipses (`...`) for natural pauses — v3 handles them better than SSML break tags.

Any TTS service works. The narration is a separate MP3 that gets muxed into the final video with ffmpeg.
Keep the narration file next to the exported film as `narration.mp3` unless you intentionally change the template path.

### Recording

Playwright records the film headless at 1920x1080 when the page keeps time correctly. The `?autoplay` URL parameter:
- Hides the start screen, voice toggle, REC badge, and scene counter
- Auto-starts the film without user interaction
- Keeps animated elements (kill counters, reveals) intact
- Stays silent so the clean narration track is added in post

```bash
# Record
node -e "
const { chromium } = require('playwright');
(async () => {
  const browser = await chromium.launch({ headless: true });
  const context = await browser.newContext({
    viewport: { width: 1920, height: 1080 },
    recordVideo: { dir: './video/', size: { width: 1920, height: 1080 } }
  });
  const page = await context.newPage();
  await page.goto('file:///path/to/index.html?autoplay');
  // DURATION_MS = ffprobe narration.mp3; narration is the fixed clock.
  await page.waitForTimeout(DURATION_MS + 5000);
  await context.close();
  await browser.close();
})();
"

# Mux with narration
ffmpeg -y -i video/recording.webm -i narration.mp3 \
  -map 0:v -map 1:a \
  -c:v libx264 -crf 16 -preset slow -c:a aac -b:a 192k \
  -shortest output.mp4
```

If headless playback drifts or drops timing-sensitive beats, record the real
browser in the foreground and capture video only. Then mux the locked narration
track afterward.

### Thumbnails

Generate at YouTube spec (1280x720):
```bash
node -e "
const { chromium } = require('playwright');
(async () => {
  const browser = await chromium.launch();
  const page = await browser.newPage({ viewport: { width: 1280, height: 720 } });
  await page.goto('file:///path/to/thumbnail.html');
  await page.waitForTimeout(1000);
  await page.screenshot({ path: 'thumbnail.png' });
  await browser.close();
})();
"
```

Check the export at both `1280x720` and `320x180`. If the headline gets
cropped or the frame reads like a wall of text, simplify it.

## Production Checklist

For the current production checklist, see
`references/production-checklist.md`.

## Customization

### Colors

The film uses a single study-film canon (single locked palette, no
parallel "cold terminal" sub-palette). To customize for a different
brand, edit the canon tokens in `references/template.html` and
`SKILL.md` §4 together — they must stay in sync.

```css
/* Canon palette */
background: #0A0A0A;    /* surface */
color: #E0E0E0;         /* text on dark */
/* terracotta: #D4574A — primary editorial accent */
/* failure:    #EF4444 — failure / error / correction */
/* gold:       #D4A843 — warning / progress */
/* success:    #4ADE80 — pass-state */
/* axis:       #999999 — paths, captions, output */

/* Surface ladder (three steps only) */
/* #0A0A0A page → #141414 panel → #1E1E22 nested */
```

Hard rules: no borders or shadows around content panels, no parallel
terminal palette, glow only on small marks at ≤ 0.3 opacity.

### Fonts

Default: [DM Sans](https://fonts.google.com/specimen/DM+Sans) + [Geist Mono](https://fonts.google.com/specimen/Geist+Mono) via Google Fonts CDN.

### Sounds

[SND library](https://github.com/nicklasserra/snd-lib) kit01 (sine):

| Event | Sound | Volume |
|-------|-------|--------|
| Scene change | TRANSITION_UP | 0.35 |
| Terminal line | TYPE | 0.25 |
| Text reveal | TAP | 0.2 |
| Correction | CAUTION | 0.45 |
| Discovery | NOTIFICATION | 1.0 |
| Ship | CELEBRATION | 1.0 |

### Text Sizing for Video

When exporting to YouTube, text must be readable at 1080p. Minimum sizes:

```
Terminals:       width: 85vw; max-width: 1400px
Terminal body:   clamp(16px, 1.6vw, 24px)
Narration strip: clamp(18px, 2vw, 28px)
Badges/labels:   clamp(14px, 1.4vw, 18px)
```

## Privacy

- Never includes env values, API keys, or secrets
- Redacts sensitive URLs with `████████████`
- Service IDs truncated
- Commit hashes and branch names are fine (public in git)

## File Structure

```
study-film/
  SKILL.md              # Skill prompt (Claude Code reads this)
  references/
    template.html       # Base HTML template with styles, sounds, and player
    production-checklist.md
```

## License

MIT
