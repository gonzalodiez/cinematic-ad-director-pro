# cinematic-ad-director-pro

A [Claude Code skill](https://www.claude.com/product/claude-code) that turns a single product photo into a finished cinematic ad — and acts as a credit guardian along the way. Combines creative direction (story, references, style) with a fixed production stack: Higgsfield for image and animation (Seedance 2.0), ElevenLabs for voice and music, FFmpeg for assembly, Airtable as the central project database.

## What's "Pro" about it

The Pro version layers four hard guarantees on top of the base skill:

- **Multi-service cost ledger** — every response shows live spend across Higgsfield, ElevenLabs, Airtable, and Claude Code so you never lose track of what you're burning.
- **Mandatory cost gates** — `get_cost: true` preflights for Draft + Full on every paid call. The user decides A or B before credits move.
- **Live model routing** — no hardcoded model picks. Higgsfield's `models_explore` recommends, the user chooses, the skill explains the trade-offs.
- **Prompt cleaner** — strips buzzwords (`photorealistic`, `cinematic`, `epic`, etc.) and replaces them with camera-measurable language before any paid call.
- **Orientation contract** — persistent state header + phase entry/exit ceremonies + decision ledger so you always know where you are, what's locked, and what's coming next.

## Pipeline (11 phases)

| Phase | What happens |
|---|---|
| Pre-flight | MCP wizard + .env validation + plan capture (first run only) |
| 0 | Project setup: folders, Airtable row, project.json |
| 1 | Lock 3 inputs: Money Shot, Concept Board, Brief libre + Character Bible |
| 2 | Brief + shot list (one Airtable row per shot) |
| 2.5 | Prompt cleaning (anti-buzzword pass) |
| 3 | Keyframes (Higgsfield image gen with cost gate per shot) |
| 3.5 | Keyframe approval gate (contact sheet) |
| 4 | Animation (Seedance 2.0 per clip) — preceded by 4.0 VO scratch |
| 5 | Audio (ElevenLabs VO + music) |
| 6 | Assembly (FFmpeg picture-lock + voice mix + final master) |
| 6.5 | Art Direction critique |
| 7 | QA + Airtable upload |

## Stack (required)

| Tool | How it's used | How to connect |
|---|---|---|
| **Higgsfield** | Image generation (NBP, Soul 2.0), animation (Seedance 2.0) | MCP via Claude Code Connectors |
| **ElevenLabs** | Voice synthesis, music, SFX | MCP via Claude Code Connectors |
| **Airtable** | Project DB, asset lineage, decision audit | MCP via Claude Code Connectors |
| **FFmpeg** | Final assembly, color, audio mixing | Local — `brew install ffmpeg` on macOS |
| **PIL (Python)** | Text overlay PNG generation | Local — installed by default with Python 3 |

## Installation

```bash
# 1. Clone into your Claude Code skills folder
cd ~/.claude/skills
git clone https://github.com/gonzalodiez/cinematic-ad-director-pro.git

# 2. Make sure the 3 MCPs are connected in Claude Code Connectors UI
#    https://claude.ai/connections

# 3. Make sure FFmpeg is installed locally
brew install ffmpeg   # or your package manager

# 4. Add API keys to ~/.env (or {project}/.env)
#    HIGGSFIELD_API_KEY_ID=...
#    HIGGSFIELD_API_KEY_SECRET=...
#    ELEVENLABS_API_KEY=...
#    AIRTABLE_API_KEY=...
#    AIRTABLE_BASE_ID=...        # populated by the skill on first project
```

## First-time use

In Claude Code, invoke the skill by dropping a product photo and saying something like:

> "convertí esta foto en un ad cinematic de 15 segundos para Reels"

On your **first session ever**, the skill runs a one-time setup wizard (A.0) that captures your subscription tiers across the 4 paid services. This populates `user-plans.json` so the cost ledger in every response reflects YOUR quotas, not generic defaults.

The wizard captures:

1. Higgsfield plan (Free / Plus / Ultra)
2. ElevenLabs plan (Free / Starter / Creator / Pro / Scale) + accumulated free credits
3. Airtable plan (Free / Plus / Pro / Team / Enterprise)
4. Claude Code plan (Pro / Max / API-pay-as-you-go)

After that, every response starts with an 8-line state header:

```
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
📍 Phase 4 — Animation  ·  Shot 2 of 3 rendering
█████░░░░░░░ 45% pipeline complete
💰 Higgsfield: 33 cr used  ·  977 left  ·  next ~36 cr
🎙 ElevenLabs: ~261 cr · 39022 cr left (starter+free)
📊 Airtable:   12 records · 15 MB attachments (free)
🤖 Claude:     ~62 tool calls this session (pro)
✓ Locked: Premium · Mechanism · 3 shots · UI=ES · VO=PT-BR Adam
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
```

## Recovery commands

Use these any time during a session to reclaim control:

- `/back-to phase {N}` — re-open a phase's decisions for editing
- `/redo {artifact}` — regenerate one artifact (character-bible, brief, shot-list, keyframe-N, clip-N, vo, music)
- `/show ledger` — dump the full decision ledger
- `/show budget` — dump per-phase cost breakdown
- `/skip {phase}` — skip a phase (mark as user-opted-out)
- `/setup-wizard` — re-run the A.0 plan capture (e.g. when you upgrade a sub)

## Customization

Two layers:

- **`SKILL.md`** — orchestration logic (the "what to do"). Edit only if you're changing the pipeline structure.
- **`best_practices.md`** — taste + parameters (the "how it should feel"). Edit freely to tune: cinematography defaults, audio LUFS targets, anti-generic banlists, color grades, BPM ranges, FFmpeg recipes, model misfit detection rules, decision lockability table.

Per-project overrides live in `creative/project.json` inside each project folder.

## Cost calibration (measured)

Empirical rates from real runs — use these for budgeting:

- **Higgsfield Seedance 2.0** — 1080p std, 4s clip: ~36 cr
- **Higgsfield Nano Banana Pro** — 4k image, count=3: ~12 cr actual (preflight reports 4 cr — multiply preflight by ~3 at high counts/resolutions)
- **Higgsfield Soul 2.0** — 2k image: ~0.12 cr per variant
- **ElevenLabs TTS** — 1 cr per character
- **ElevenLabs Music** — ~46 cr per second of generated audio

A typical 10s vertical Reels ad runs ~140 Higgsfield credits + ~1700 ElevenLabs credits + ~15 Airtable records + ~300 Claude tool calls.

## Known limits (disclosed up front)

- **No seed lock** — Higgsfield MCP does not expose seeds. Identity across shots is anchored via reference images or trained Soul Characters, not random seeds.
- **No batch video** — Seedance renders one clip at a time.
- **No runtime Claude usage API** — token counter is a rough proxy via tool-call count.
- **NBP preflight under-reports** — multiply by ~3 when `count > 1` at 4k.
- **Soul 2 with multi-image reference can flip to image-to-image mode** — crop reference to a single subject, or skip the reference. See `best_practices.md` §10.

## Sharing with teammates

This skill is portable. Each teammate clones the repo and runs the A.0 wizard on first use.

- `user-plans.json` is gitignored — your personal subscriptions don't leak
- Each teammate's `~/.env` is their own (with their own API keys)
- The skill itself (SKILL.md, best_practices.md) is identical across the team

If you fork this repo and customize `best_practices.md`, that's the right place — taste decisions stay in the fork, orchestration stays portable.

## Origin

This is the Pro fork of `cinematic-ad-director`, built by [gonzalodiezmac](https://github.com/gonzalodiez). Inspired by these vetted skills:

- `coreyhaines31/marketingskills@ad-creative` — angle-based structured generation, performance iteration
- `dylanfeltus/skills@creative-direction` — anti-generic philosophy, "every image has a job", specificity over adjectives
- `higgsfield-token-helper` — live model routing, draft-vs-full cost gate, anti-buzzword prompt cleaner

## License

MIT. Use, fork, share. If you build on top of this, drop a star or open a PR.
