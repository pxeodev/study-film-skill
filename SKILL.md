# Study Film Generator

Turn Claude Code sessions into narrated video content for YouTube, social media, or internal documentation.

Use this skill when the user says "make a study", "generate a film", "create a video from this session", "/study", or "/film".

## What This Skill Does

Turns a Claude Code session into a publishable video:

1. **Extract** the narrative arc and the factual claims from the session
2. **Generate** a self-contained HTML film with terminal animations, timed scenes, and sound
3. **Lock** a narration script and audio take before final timing
4. **Pace** the visual cut to the actual narration, not word-count estimates
5. **Review** on the normal page with audio, then capture the silent `?autoplay` cut
6. **Export** as MP4, captions, and thumbnail assets for upload

The HTML film works standalone in any browser (with keyboard navigation and sound), AND exports cleanly to video via the `?autoplay` recording mode.

## Step-by-Step Process

### 1. Extract the narrative arc

Read the conversation history from the current session. Identify:
- **The task**: what was the user trying to do?
- **Corrections**: where did the AI get it wrong and the user corrected it? (These are the most important moments — they create tension)
- **Discoveries**: unexpected findings, scans, data that appeared
- **The ship**: what was committed/pushed/deployed at the end?
- **Claims**: dates, percentages, counts, policy values, and terminology that must stay internally consistent

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

### 2b. Storyboard discipline

- One idea per scene. If a viewer needs more than 1-2 seconds to parse a line, split it.
- Avoid metadata kickers that read like logs. Top lines should frame, not explain.
- Comparison scenes should have one dominant contrast (`-127%` vs `SKIPPED`, `intuition` vs `method`) and minimal supporting copy.
- If the narration already says the sentence, reduce or remove duplicate on-screen text.

### 2c. Lock narration before timing

Write the narration as a separate script, generate the audio, and treat that audio as the fixed clock.

- Do not finalize scene `dur` values from rough estimates alone.
- Review on the normal page with narration on.
- Use `?autoplay` only for silent capture after the pacing already feels right.
- If a long section drifts, cut the scene on the spoken cue, not on a guessed midpoint.

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

The film uses a single study-film canon. There is no parallel "cold
terminal" palette — terminals live on the same surface ladder and use
the same canonical tokens throughout the piece.

**Palette (canon — locked):**
```css
--terracotta:  #D4574A   /* primary editorial accent — prompts, carets, tool calls, hi */
--failure:     #EF4444   /* failure / error / correction red */
--gold:        #D4A843   /* warning / open-issue / progress accent */
--success:     #4ADE80   /* success / pass-state */
--axis:        #999999   /* axes, captions, metadata, paths, output, comments */
--text:        #E0E0E0   /* body text, command output */
--surface:     #0A0A0A   /* page background */
--logo:        #E74C3C   /* bloom + wordmark only — never UI */
```

**Surface ladder (allowed by exception, three steps only):**
```css
--surface-0: #0A0A0A   /* page */
--surface-1: #141414   /* panel, terminal body, narration area */
--surface-2: #1E1E22   /* nested element, terminal title bar */
```

**Terminal-block token mapping (no parallel palette):**
```css
.t-prompt  { color: #D4574A; }   /* terracotta — prompt name */
.t-path    { color: #999999; }   /* axis gray — directory */
.t-caret   { color: #D4574A; }   /* terracotta — ❯ */
.t-tool    { color: #D4574A; }   /* terracotta — tool calls */
.t-cmd     { color: #E0E0E0; }   /* text — command body */
.t-out     { color: #999999; }   /* axis — stdout */
.t-comment { color: #999999; }   /* axis — italics */
.t-ok      { color: #4ADE80; }   /* success */
.t-red     { color: #EF4444; }   /* failure */
.t-warn    { color: #D4A843; }   /* warning */
.t-dim     { color: #999999; }   /* dim */
```

**Hard rules:**
- Logo red `#E74C3C` and failure red `#EF4444` never share a frame.
  Logo red is allowed only on the end card (physarum bloom + wordmark).
- No `#E84749`, `#5CB87A`, `#22D3EE`, `#60A5FA`, `#FACC15`, `#A78BFA` —
  these were drift in earlier films and are now scan-flagged.
- No borders or shadows around content panels. Use surface steps
  (`#0A0A0A` → `#141414` → `#1E1E22`) for hierarchy.
- Glow on small marks (signal dots, gate cells) is allowed at
  `0 0 12px` ≤ 0.3 opacity. Glow on panels is forbidden.
- Background gradients on the surface itself are allowed at ≤ 0.15
  opacity. They're surface variation, not depth.

**Fonts (canon — locked):**
- DM Sans — editorial text, headers, narration, panel titles, body copy.
- Geist Mono — all monospace: terminals, kickers, axes, code, labels,
  counters, timestamps.
- Google Fonts URL: `DM+Sans:wght@400;500;600;700&family=Geist+Mono:wght@400;500;700`
- No Space Mono. No system mono fallbacks. No JetBrains.

**Motion — character-stepped reveals:**
- Allowed only as terminal output: Geist Mono on dark surface,
  terracotta cursor, no UI chrome. Step cadence 30–80ms per character.
- Not allowed for narration overlays, panel titles, or quoted text
  in DM Sans. Quoted text uses full-quote fade-in.

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

### 7. Contradiction pass

Before capture, review the whole film for internal contradictions.

- Terms: roles must stay distinct (signal vs filter vs action, or any equivalent domain language)
- Numbers: dates, percentages, counts, durations, thresholds, and rule values must match across scenes
- Policy: rule wording must not change scene-to-scene
- Provenance: if an entry date or source is not system-of-record verified, soften the claim
- Packaging: thumbnail/title should emphasize the clearest conflict, not a side detail

### 8. Output location

Write the HTML file wherever the user specifies, or default to the current working directory.

Optional YouTube deliverables:
- `thumbnail.html`
- `thumbnail.png`
- `captions.en.srt`
- `captions.en.vtt`
- short description + pinned comment draft

For the production checklist that captures the current workflow, see
`references/production-checklist.md`.

### 9. Privacy rules

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
html.autoplay .start-screen { display: none !important; }
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
  setTimeout(function() { running = true; nextScene(); }, 500);
}
```

### 3. Review mode vs capture mode

- `index.html` (normal page): use this for the human pacing pass with narration on
- `index.html?autoplay`: use this for the silent capture pass only
- Do not judge timing from the silent capture URL alone

### 4. Text sizing for 1080p video

YouTube videos are viewed at various sizes. Minimum readable sizes for 1920x1080:

```
Terminals:       width: 85vw; max-width: 1400px
Terminal body:   font-size: clamp(16px, 1.6vw, 24px)
Terminal title:  font-size: clamp(12px, 1.1vw, 15px)
Narration strip: font-size: clamp(18px, 2vw, 28px)
Badges:          font-size: clamp(14px, 1.4vw, 18px)
Labels:          font-size: clamp(14px, 1.4vw, 18px)
```

### 5. Capture strategy

Start with Playwright when the page keeps time correctly in headless mode.

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
  // DURATION_MS = ffprobe narration.mp3; narration is the fixed clock.
  await page.waitForTimeout(DURATION_MS + 5000);
  await context.close();
  await browser.close();
})();
"
```

If headless capture drifts because timers miss beats, long animations stall,
or the browser deprioritizes work, switch to a real foreground browser capture
(QuickTime or the OS screen recorder) and record video only. Then mux the
locked narration in afterward.

### 6. Add narration audio

Use any TTS service to generate narration from your script. [ElevenLabs](https://try.elevenlabs.io/ots0dbiulx3y) produces the most natural results for this style of content.

Mux video + audio with ffmpeg:
```bash
# If the capture has a leading delay, trim or set -t explicitly.
ffmpeg -y -i video/recording.webm -i narration.mp3 \
  -map 0:v -map 1:a \
  -c:v libx264 -crf 16 -preset slow -c:a aac -b:a 192k \
  -shortest output.mp4
```

### 7. Narration sync tips

- Lock the narration file before final timing
- Measure the actual narration duration with `ffprobe` or equivalent
- Use `ffmpeg -af silencedetect` only as a rough aid, not the source of truth
- If a long section drifts, transcribe the audio and move internal cuts to the spoken words
- Total scene durations should match the narration closely enough that the last visual beat lands without dead air
- If narration strip text duplicates what's on screen, clear or shorten it

### 8. Packaging

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

Rules:
- Verify legibility at both `1280x720` and `320x180`
- Prefer one dominant contrast over lots of explanatory text
- Do not crop headline words at the export edge
- If the workflow is the real hook, package the structure, not the niche detail

Also ship:
- captions (`.srt` and optionally `.vtt`)
- a concise title
- a short description
- a pinned comment

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
