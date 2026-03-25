# Study Film

A Claude Code skill that turns coding sessions into narrated videos for YouTube and social media.

<p align="center">
  <img src="https://img.shields.io/badge/Claude_Code-skill-D4A843?style=flat-square" alt="Claude Code Skill">
  <img src="https://img.shields.io/badge/output-MP4_via_Playwright-5CB87A?style=flat-square" alt="MP4 Output">
  <img src="https://img.shields.io/badge/narration-ElevenLabs-9CA3AF?style=flat-square" alt="ElevenLabs Narration">
</p>

## What it does

After a coding session, say `/study` or "make a film from this session." The skill reads your conversation history, extracts the narrative arc (setup, corrections, discoveries, ship), and generates a narrated video with:

- **Terminal animations** — line-by-line reveals matching Claude Code's tool calls
- **AI narration** — script generated from session, voiced via [ElevenLabs](https://try.elevenlabs.io/ots0dbiulx3y)
- **1080p export** — recorded headless via Playwright, muxed with ffmpeg
- **Sound design** — transition, typing, caution, and celebration sounds via [SND](https://github.com/nicklasserra/snd-lib)
- **Dual palette** — warm editorial frame, cold terminal interior
- **YouTube-ready** — 1920x1080, thumbnail generator, metadata template

Output pipeline: HTML film → Playwright recording → ffmpeg mux with narration → MP4 upload.

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
# 2. Record with Playwright at 1080p
# 3. Mux audio + video with ffmpeg
# See SKILL.md "YouTube Recording Mode" for full pipeline
```

## Pipeline Overview

```
Session → /study → HTML film → ?autoplay → Playwright 1080p → ffmpeg mux → MP4 → YouTube
                                    ↑                              ↑
                               hides web UI                  adds narration
                               auto-starts                   from ElevenLabs
```

### Voice Narration

The skill generates a narration script from the session. For the actual voice, [ElevenLabs](https://try.elevenlabs.io/ots0dbiulx3y) produces the most natural results. Their `eleven_v3` model with a narrator-style voice works well for this format. Use ellipses (`...`) for natural pauses — v3 handles them better than SSML break tags.

Any TTS service works. The narration is a separate MP3 that gets muxed into the final video with ffmpeg.

### Recording

Playwright records the film headless at 1920x1080. The `?autoplay` URL parameter:
- Hides the start screen, voice toggle, REC badge, and scene counter
- Auto-starts the film without user interaction
- Keeps animated elements (kill counters, reveals) intact

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
  await page.waitForTimeout(DURATION_MS + 5000);
  await context.close();
  await browser.close();
})();
"

# Mux with narration (trim 1s black frame from autoplay startup)
ffmpeg -y -ss 1 -i video/recording.webm -i narration.mp3 \
  -c:v libx264 -crf 16 -preset slow -c:a aac -b:a 192k \
  -map 0:v -map 1:a -shortest output.mp4
```

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

## Customization

### Colors

Two palettes. Edit CSS variables in `references/template.html`:

**Frame** (warm surround):
```css
background: #262626;    /* swap for your bg */
color: #F9FAFB;         /* primary text */
/* accent: #D4A843 (gold highlights) */
/* red: #E84749 (corrections) */
```

**Terminal** (cold interior):
```css
background: #0C0C0E;    /* terminal bg */
/* prompt: #22D3EE (cyan) */
/* path: #60A5FA (blue) */
/* caret: #FACC15 (yellow) */
/* tool: #A78BFA (purple) */
```

### Fonts

Default: [DM Sans](https://fonts.google.com/specimen/DM+Sans) + [Space Mono](https://fonts.google.com/specimen/Space+Mono) via Google Fonts CDN.

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
```

## License

MIT
