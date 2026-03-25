# Study Film Generator

Turn Claude Code sessions into narrated video content for YouTube, social media, or internal documentation.

Use this skill when the user says "make a study", "generate a film", "create a video from this session", "/study", or "/film".

## What This Skill Does

Turns a Claude Code session into a publishable video:

1. **Extract** the narrative arc from the session (task → corrections → discoveries → ship)
2. **Generate** a self-contained HTML film with terminal animations, timed scenes, and sound
3. **Narrate** with AI voice ([ElevenLabs](https://try.elevenlabs.io/ots0dbiulx3y) recommended for natural delivery)
4. **Record** at 1080p via Playwright (headless, no manual screen capture)
5. **Export** as MP4 with narration muxed in — ready to upload to YouTube

The HTML film works standalone in any browser (with keyboard navigation and sound), AND exports cleanly to video via the `?autoplay` recording mode.

## Step-by-Step Process

### 1. Extract the narrative arc

Read the conversation history from the current session. Identify:
- **The task**: what was the user trying to do?
- **Corrections**: where did the AI get it wrong and the user corrected it? (These are the most important moments — they create tension)
- **Discoveries**: unexpected findings, scans, data that appeared
- **The ship**: what was committed/pushed/deployed at the end?

Structure into acts: SETUP → INCIDENT → CORRECTIONS → RECOVERY → SHIP

### 2. Build the scene config

Each scene needs:
```javascript
{
  // Visual: either a title card OR a terminal block (never both)
  title: 'tc_id',           // title card element id
  // OR
  term: 'term_id',          // terminal element id
  tb: 'tb_id',              // terminal body id
  lines: ['...'],           // terminal lines (HTML strings)

  // Timing
  dur: 3500,                // scene duration in ms (2500-7000 range)

  // Narration
  label: 'CORRECTION 1',   // small label above narration
  narr: 'Text with <span class="hi">highlights</span>',

  // Sound (optional)
  sound: 'caution',         // caution | notification | celebration
}
```

### 3. Terminal line formatting

```html
<!-- Prompt (customize the prompt name for your project) -->
<span class="t-prompt">project</span> <span class="t-path">~/path</span> <span class="t-caret">❯</span> <span class="t-cmd">command</span>

<!-- Tool calls (Claude Code internal) -->
<span class="t-tool">  ⏵ Read</span> <span class="t-cmd">path/to/file</span>
<span class="t-tool">  ⏵ Edit</span> <span class="t-cmd">path/to/file</span>
<span class="t-tool">  ⏵ Bash</span> <span class="t-cmd">git status</span>

<!-- Output -->
<span class="t-out">  output text</span>
<span class="t-ok">  ✓ success</span>
<span class="t-red">  ✗ error</span>
<span class="t-warn">  ⚠ warning</span>
<span class="t-dim">  // comment</span>
<span class="t-comment">  # user message</span>

<!-- Redacted values -->
<span class="t-leaked">https://████████████/api/endpoint</span>

<!-- Strikethrough for corrections -->
<span class="t-red" style="text-decoration:line-through">old value</span> <span class="t-dim">→</span> <span class="t-ok">new value</span>
```

### 4. Color tokens

The film uses two palettes: a warm frame and a cold terminal interior.

**Frame (warm, editorial):**
```css
--bg:         #262626   /* neutral gray */
--text:       #F9FAFB   /* near-white */
--text-sec:   #9CA3AF   /* secondary prose */
--text-muted: #6B7280   /* labels */
--border:     #404040   /* separator */
--accent:     #D4A843   /* gold — highlights */
--red:        #E84749   /* corrections, errors */
```

**Terminal blocks (cold, precise):**
```css
--term-bg:    #0C0C0E   /* darkest surface */
--term-border:#1E1E22   /* hard cut */
--prompt:     #22D3EE   /* cyan — prompt name */
--path:       #60A5FA   /* blue — directory */
--caret:      #FACC15   /* yellow — ❯ */
--tool:       #A78BFA   /* purple — tool calls */
--ok:         #5CB87A   /* green — success */
--red:        #E84749   /* red — errors */
--warn:       #D4A843   /* gold — warnings */
--out:        #6B7280   /* gray — output */
--dim:        #333333   /* dark — comments */
```

**Fonts:**
- Frame: DM Sans (titles, narration) + Space Mono (labels, counters)
- Terminal blocks: Space Mono only
- Google Fonts URL: `DM+Sans:wght@400;500;600;700&family=Space+Mono:wght@400;700`

### 5. Sound mapping

Using SND library kit01 (sine), loaded via CDN:
```html
<script src="https://cdn.jsdelivr.net/gh/snd-lib/snd-lib@v1.2.4/dist/browser/snd.js?kit=01"></script>
```

| Event | Sound | Volume |
|-------|-------|--------|
| Scene transition | TRANSITION_UP | 0.35 |
| Terminal line appear | TYPE | 0.25 |
| Text line reveal | TAP | 0.2 |
| User correction quote | CAUTION | 0.45 |
| Discovery / scope change | NOTIFICATION | 1.0 |
| Final ship | CELEBRATION | 1.0 |

### 6. Generate the HTML

Use the template at `references/template.html` as the base. Replace:
- Title card text (study number, title, subtitle)
- All scene HTML (title cards and terminal blocks)
- The `scenes` array in the script
- Any session-specific data (repo names, branch names, commit hashes)

### 7. Output location

Write the HTML file wherever the user specifies, or default to the current working directory.

### 8. Privacy rules

- NEVER include actual env values, API keys, endpoints, or secrets
- Redact with `████████████` blocks
- Service IDs can be truncated: `srv-abc123...`
- Commit hashes are fine (public in git history)
- Branch names are fine (public on GitHub)
- Check with user if uncertain about any value

## Narration Style

- Short. One sentence per scene.
- Highlight key phrases with `<span class="hi">gold</span>` or `<span class="red">red</span>`
- User corrections are the hooks — quote them exactly as typed (typos included)
- Labels: PROMPT, CORRECTION 1-N, DISCOVERY, SHIPPED, REFRAME, etc.
- No philosophy in narration. Just what happened.
- Closing is always dry: "Four corrections. One session. It shipped."

## YouTube Recording Mode

When exporting a study film as a video for YouTube or social media, add `?autoplay` to the URL. This enables recording mode:

### 1. Hide interactive UI

Add this CSS to your template — these elements are for browser interaction, not video:

```css
html.autoplay .start { display: none !important; }
html.autoplay .audio-toggle { display: none !important; }
html.autoplay .rec-badge { display: none !important; }
html.autoplay .counter { display: none !important; }
```

### 2. Auto-start detection

Add this at the very top of `<body>`, BEFORE the start screen div:
```html
<script>if(location.search.includes('autoplay'))document.documentElement.classList.add('autoplay')</script>
```

And at the bottom of your script, after `startFilm()`:
```javascript
if (window.location.search.includes('autoplay')) {
  document.getElementById('startScreen').style.display = 'none';
  setTimeout(function() { running = true; nextScene(); }, 500);
}
```

### 3. Text sizing for 1080p video

YouTube videos are viewed at various sizes. Minimum readable sizes for 1920x1080:

```
Terminals:       width: 85vw; max-width: 1400px
Terminal body:   font-size: clamp(16px, 1.6vw, 24px)
Terminal title:  font-size: clamp(12px, 1.1vw, 15px)
Narration strip: font-size: clamp(18px, 2vw, 28px)
Badges:          font-size: clamp(14px, 1.4vw, 18px)
Labels:          font-size: clamp(14px, 1.4vw, 18px)
```

### 4. Record with Playwright

```bash
node -e "
const { chromium } = require('playwright');
(async () => {
  const browser = await chromium.launch({ headless: true });
  const context = await browser.newContext({
    viewport: { width: 1920, height: 1080 },
    recordVideo: { dir: './video/', size: { width: 1920, height: 1080 } }
  });
  const page = await context.newPage();
  await page.goto('file:///path/to/film/index.html?autoplay');
  await page.waitForTimeout(DURATION_MS + 5000);
  await context.close();
  await browser.close();
})();
"
```

### 5. Add narration audio

Use any TTS service to generate narration from your script. [ElevenLabs](https://try.elevenlabs.io/ots0dbiulx3y) produces the most natural results for this style of content.

Mux video + audio with ffmpeg:
```bash
# Trim 1s black frame from autoplay startup, add narration
ffmpeg -y -ss 1 -i video/recording.webm -i narration.mp3 \
  -c:v libx264 -crf 16 -preset slow -c:a aac -b:a 192k \
  -map 0:v -map 1:a -shortest output.mp4
```

### 6. Narration sync tips

- Estimate scene durations from word count (~2.8 words/sec)
- Verify with `ffmpeg -af silencedetect` on the narration audio
- Manual spot-check: extract 3s clips at estimated transitions
- Total scene durations should match narration ±5 seconds
- If narration strip text duplicates what's on screen, clear it

### 7. Thumbnail

Render at YouTube spec (1280x720) with Playwright:
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

## Keyboard Controls

| Key | Action |
|-----|--------|
| Space | Start / pause / resume |
| Arrow Right / Page Down | Next scene (pauses auto) |
| Arrow Left / Page Up | Previous scene |

## Installation

1. Copy this directory to `~/.claude/skills/study-film/`
2. The skill triggers on "make a study", "/study", "/film"
3. Output is a single HTML file — open in any browser
