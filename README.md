# Study Film

A Claude Code skill that turns coding sessions into visual storyboards.

<p align="center">
  <img src="https://img.shields.io/badge/Claude_Code-skill-D4A843?style=flat-square" alt="Claude Code Skill">
  <img src="https://img.shields.io/badge/output-HTML-5CB87A?style=flat-square" alt="HTML Output">
  <img src="https://img.shields.io/badge/dependencies-none-9CA3AF?style=flat-square" alt="No Dependencies">
</p>

## What it does

After a coding session, say `/study` or "make a film from this session." The skill reads your conversation history, extracts the narrative arc (setup, corrections, discoveries, ship), and generates a self-contained HTML storyboard with:

- **Terminal animations** — line-by-line reveals matching Claude Code's tool calls
- **Timed narration** — one sentence per scene, corrections highlighted
- **Sound design** — transition, typing, caution, and celebration sounds via [SND](https://github.com/nicklasserra/snd-lib)
- **Keyboard navigation** — Space (pause/resume), arrows (prev/next)
- **Dual palette** — warm editorial frame, cold terminal interior

Output is a single HTML file. No build step. Open in any browser.

## Example

> "Three corrections. One session. It shipped."

The skill structures every session as a story: what you asked for, where the AI got it wrong, what was discovered along the way, and what shipped at the end. User corrections are the dramatic hooks — they're quoted verbatim, typos and all.

## Install

```bash
# Copy to your Claude Code skills directory
cp -r . ~/.claude/skills/study-film/
```

Then in any Claude Code session:
```
/study
```

## File structure

```
study-film/
  SKILL.md              # Skill prompt (Claude Code reads this)
  references/
    template.html       # Base HTML template with styles, sounds, and player
```

## Customization

### Colors

The template uses two palettes. Edit the CSS variables in `template.html`:

**Frame** (the warm surround):
```css
background: #262626;    /* swap for your bg */
color: #F9FAFB;         /* primary text */
/* accent: #D4A843 (gold highlights) */
/* red: #E84749 (corrections) */
```

**Terminal** (the cold interior):
```css
background: #0C0C0E;    /* terminal bg */
/* prompt: #22D3EE (cyan) */
/* path: #60A5FA (blue) */
/* caret: #FACC15 (yellow) */
/* tool: #A78BFA (purple) */
```

### Fonts

Default: [DM Sans](https://fonts.google.com/specimen/DM+Sans) + [Space Mono](https://fonts.google.com/specimen/Space+Mono) via Google Fonts CDN. Change the `<link>` tag and `font-family` values to use your own.

### Sounds

Uses [SND library](https://github.com/nicklasserra/snd-lib) kit01 (sine). Sounds are mapped to events:

| Event | Sound | When |
|-------|-------|------|
| TRANSITION_UP | Scene change | Every scene |
| TYPE | Terminal line | Each line appears |
| CAUTION | User correction | Quoted corrections |
| NOTIFICATION | Discovery | Unexpected findings |
| CELEBRATION | Ship | Final deploy/push |

### Terminal prompt

The default prompt is generic. Customize in the scene config:
```html
<span class="t-prompt">myproject</span> <span class="t-path">~/code</span> <span class="t-caret">❯</span>
```

## Privacy

The skill enforces privacy rules:
- Never includes env values, API keys, or secrets
- Redacts sensitive URLs with `████████████`
- Service IDs are truncated
- Commit hashes and branch names are fine (public in git)

## How it works

1. Reads the conversation history from the current Claude Code session
2. Identifies the narrative arc: task, corrections, discoveries, ship
3. Structures into timed scenes (2.5–7s each)
4. Generates HTML with terminal blocks, title cards, and narration
5. Outputs a single file — open it, press play

The narration style is dry and factual. No philosophy. Just what happened. Corrections are the hooks. The closing line is always a stat count: *"Three corrections. One session. It shipped."*

## License

MIT
