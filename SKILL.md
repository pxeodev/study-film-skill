# Study Film Generator

Generate film-style HTML storyboards from Claude Code sessions. Turn any coding session into a visual narrative with terminal animations, timed narration, and sound.

Use this skill when the user says "make a study", "generate a film", "create a storyboard from this session", "/study", or "/film".

## What This Skill Does

Turns a Claude Code session into a self-contained HTML film with:
- Auto-advancing scenes with timed durations
- Split-screen layout: terminal demo on top, narration below
- Terminal animations with line-by-line reveals
- SND sound library (kit01 sine) with calibrated volumes
- REC badge, progress bar, scene counter
- Keyboard navigation (Space=pause, arrows=prev/next)

Output is a single self-contained HTML file with no external dependencies except CDN fonts and SND.

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
| User correction quote | CAUTION | 1.0 |
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
