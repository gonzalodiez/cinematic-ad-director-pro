# Cinematic Ad Director Pro — Best Practices

> **Read this file at the start of every project, before Phase 0.** This is the parameters file. Update it once, propagate everywhere. SKILL.md handles orchestration; this file handles taste.

Last updated: 2026-05-18 — Pro adds credit discipline (§10), prompt cleaner reference (§5b), and live model routing notes (§11).

---

## 0. Credit Discipline (Pro)

This skill never spends Higgsfield credits silently. Every paid call is preceded by:

1. **Prompt cleaner** — SKILL.md Phase 2.5. No buzzwords reach the model.
2. **Live model recommendation** — `models_explore(action: recommend)` per shot. No hardcoded picks.
3. **Draft + Full `get_cost: true` preflights** — two cost calls per shot, user picks A or B.

These three gates are non-negotiable. They exist to keep spend honest and to give the user a real choice between a cheap validation pass and a committed render.

**Honest limits (disclose at session open):**
- No seed lock — anchor identity via reference images or `soul_2` trained Souls
- No batch video — Seedance renders one at a time
- No mid-session credit tracking — `balance` is checked at pre-flight only

---

## 1. Duration Doctrine

| Duration | Beats | Shot budget | Voiceover | Use case |
|---|---|---|---|---|
| 8s | Hook → Reveal → Payoff | 1–2 shots | None | Scroll-stopper, A/B hook tests |
| 10s | Hook → Build → Payoff | 2–3 shots | ≤8 words | Paid social hero |
| **15s (default)** | Hook → Setup → Demo → Payoff+CTA | 3–5 shots | ~30 words | Most paid social, bumpers |
| 30s | Three-act | 5–8 shots | ~75 words | Launch ads, YouTube |
| 60s | Story spot | 8–14 shots | Full script | Only with a real narrative reason |

**Shot budget rule:** if you need more shots than the budget, cut a shot — do not shrink durations under 1s in non-music-video edits.

**VO pace:** ~2.5 words/second. Always read the script aloud at target speed before approval.

---

## 2. Production Mode Parameters

| Param | Premium | Optimized |
|---|---|---|
| Render resolution | 1920×1080 (or 1080×1920 vertical) | 1280×720 (or 720×1280 vertical) |
| Higgsfield `count` per shot | 3 variants | 2 variants |
| Seedance render multiplier (vs final) | 2.0× | 1.5× |
| Animation duration per shot | up to model max | clamp at 5s |
| FFmpeg `-crf` | 18 | 23 |
| FFmpeg `-preset` | slow | medium |
| Audio bitrate | 192k AAC | 128k AAC |
| Audio sample rate | 48 kHz | 48 kHz |
| Color pass | `lut3d` + `eq` | `eq` only |

Always preflight Higgsfield cost with `get_cost: true` before submitting batch generations.

---

## 3. Cinematography Defaults

### Camera moves (one dominant per shot)
- **Push-in** — emotional escalation, product reveal
- **Pull-out** — context reveal, scale shift
- **Orbit** — sensory shots, premium objects
- **Dolly** — character walk-ins, environment reveals
- **Crane** — establishing or finishing shot
- **Locked-off** — testimonial, packshot, hold-on-CTA
- **Handheld** — UGC angle, urgency

### Lens defaults
- Product hero: 50mm or 85mm equivalent, shallow depth
- Demo: 35mm, mid depth
- Environment: 24mm wide
- Anamorphic flares only when brand is genuinely cinematic-coded

### Light defaults by mood
- **Premium / hero** — single key + practical, hard shadows, color contrast
- **Sensory** — soft wraparound + rim, neutral palette
- **UGC** — flat daylight, no rim, low contrast
- **Story / cinematic** — motivated practicals, warm/cool split

---

## 4. Audio Targets

### Loudness
- Social feed (Reels, TikTok, Shorts): integrated **−16 LUFS**, true peak ≤ −1 dBTP
- YouTube long-form: integrated **−14 LUFS**, true peak ≤ −1 dBTP
- Broadcast: integrated **−24 LUFS**, true peak ≤ −2 dBTP

### Voice direction
- Generate **3 takes per line** minimum
- Direct each: pace, emphasis word, attitude, breath placement, ending energy
- Pick on emphasis landing, not on "best read overall"
- Save all 3 takes; selected take goes to `voice/selected/line-{n}.mp3`

### Music
- Generate **at least 2 candidates** against locked picture
- Brief format: `genre / BPM range / instrumentation / energy curve / lift timecode / reference track`
- Lift must land within **±4 frames** of the visual payoff (Phase 6.5 will reject misalignment)
- Prefer stems over single-mix bounces — gives FFmpeg ducking room

### Sound design
- One **signature SFX** per ad, tied to the hook moment
- Diegetic SFX on the product (the can opens, the fabric brushes, the engine starts) — sells reality more than any visual polish

### Defaults
- Default music style: cinematic minimal with a clean lift; swap if Brief libre demands otherwise
- Default VO style: confident, mid-pace, low-affect; swap per brand

---

## 5. Anti-Generic Patterns

### Banned, unless explicitly demanded
- Slow-mo water splash on every product
- Drone reveal over neutral desert / city
- Hands-of-faceless-model holding the product
- Glass smash transitions
- Lens flares as transitions
- "Cinematic" color-grade preset with crushed blacks + teal-orange
- Unmotivated camera shake
- Lower-thirds with brand-color gradient bars
- AI-default backgrounds: foggy forest, neon Tokyo alley, abstract liquid

### Required positives
- Concrete location, surfaces, textures named in the prompt
- 2–4 named colors per shot, not "moody tones"
- Lighting direction named (key from camera-left, etc.)
- Real reference: director, photographer, brand, film, frame
- Specific lens character (focal length + depth of field)

---

## 6. Character Consistency

When the Concept Board contains people, the Character Bible (see SKILL.md Phase 1) is mandatory.

### Lock priority (high → low)
1. **Trained Soul** via Higgsfield `soul_2` + `soul_id` — strongest lock, requires 5–20 reference photos and ~10 min training
2. **Reference image + verbatim Character Bible** injected into every prompt
3. **Verbatim Character Bible only** — weakest, drift expected over many shots

### Banned mid-spot variations (default)
- Hair color, length, or style change
- Wardrobe swap
- Accessory removal/addition
- Apparent age change

Allow variation only if the script explicitly requires it (e.g. a transformation moment).

---

## 7. FFmpeg Recipes

### `build_cuts.sh` — picture-lock concat

```bash
#!/usr/bin/env bash
set -euo pipefail
source .env

# Build a concat list from videos/shot-*.mp4 in order
printf "file 'videos/shot-1.mp4'\nfile 'videos/shot-2.mp4'\n" > /tmp/concat.txt

ffmpeg -y -f concat -safe 0 -i /tmp/concat.txt \
  -c:v libx264 -preset "${FFMPEG_PRESET:-slow}" -crf "${FFMPEG_CRF:-18}" \
  -pix_fmt yuv420p \
  output/${PROJECT_SLUG}_picture-lock.mp4
```

### `build_voice_paced.sh` — picture + VO only

```bash
#!/usr/bin/env bash
set -euo pipefail
source .env

ffmpeg -y \
  -i output/${PROJECT_SLUG}_picture-lock.mp4 \
  -i voice/selected/master_vo.wav \
  -filter_complex "[1:a]aresample=48000,loudnorm=I=-16:TP=-1.5:LRA=11[vo]" \
  -map 0:v -map "[vo]" \
  -c:v copy -c:a aac -b:a "${AUDIO_BITRATE:-192k}" \
  output/${PROJECT_SLUG}_vo.mp4
```

### `build_voice_v2.sh` — full mix with sidechain duck

```bash
#!/usr/bin/env bash
set -euo pipefail
source .env

ffmpeg -y \
  -i output/${PROJECT_SLUG}_picture-lock.mp4 \
  -i voice/selected/master_vo.wav \
  -i music/selected.mp3 \
  -filter_complex "
    [1:a]aresample=48000,loudnorm=I=-16:TP=-1.5:LRA=11[vo];
    [2:a][vo]sidechaincompress=threshold=0.05:ratio=8:attack=5:release=200[duck];
    [vo][duck]amix=inputs=2:duration=longest[mix]
  " \
  -map 0:v -map "[mix]" \
  -c:v libx264 -preset "${FFMPEG_PRESET:-slow}" -crf "${FFMPEG_CRF:-18}" \
  -c:a aac -b:a "${AUDIO_BITRATE:-192k}" -ar 48000 \
  output/${PROJECT_SLUG}_${ASPECT}_${DURATION}s.mp4
```

### Aspect ratio masters

For each delivery aspect, re-run `build_voice_v2.sh` with `ASPECT` set, and add the appropriate `crop`/`scale` filter:
- `9x16` → `crop=ih*9/16:ih,scale=1080:1920`
- `16x9` → `scale=1920:1080`
- `1x1` → `crop=ih:ih,scale=1080:1080`

---

## 8. Defaults to override per brand

These ship as starting points. Override in the project's `creative/project.json` to pin per-brand.

```json
{
  "preferred_music_style": "cinematic minimal, 80-100 BPM, one lift at 70% mark",
  "preferred_vo_style": "confident, mid-pace, low-affect",
  "banned_aesthetics": ["teal-orange grade", "drone over desert", "slow-mo splash"],
  "preferred_references": ["Apple product films", "Patagonia documentary tone"],
  "default_aspect_ratios": ["9x16", "1x1"],
  "default_duration_s": 15
}
```

---

## 9. Iteration Diagnostic Table

When iterating after launch metrics:

| Symptom | Likely phase | What to change |
|---|---|---|
| 3s hook retention low | Phase 3 keyframe, Phase 4 first-second motion | New hook angle, sharper first frame |
| Thumb-stop rate low | Phase 3 first frame | Test 3 alternative opening frames |
| View-through drops mid | Phase 2 pacing, Phase 5 music energy | Cut a shot, raise music energy |
| CTR low | Phase 1 angle, Phase 5 VO emphasis | Reframe angle, re-read CTA line |
| CVR low after click | Brief promise vs landing page | Out of scope — flag for marketing |

Iterate **one phase at a time**. Log results in Airtable `iterations` table.

---

## 10. Model Misfit Detection (Phase 3 cost saver)

Some Higgsfield models behave unexpectedly with reference images and waste credits before the user notices. Detect misfits at low cost **before** committing to a Full Premium generation.

### Known misfits (capture these in your scouting pass)

| Symptom | Model that does this | Why | Mitigation |
|---|---|---|---|
| Output is a multi-panel collage matching the reference layout | `soul_2` with multi-image reference sheet | `enhance_prompt: true` is hardcoded; treats the reference as a layout spec | Crop reference to a single subject; or skip reference and use prompt-only |
| Output has Reels/TikTok chrome (gibberish text overlays, side captions) | `soul_2` with `style_id: General` (default) and no reference | The "General" Soul style is UGC-biased and bakes social UI | Switch to `nano_banana_pro` for premium commercial; or try a different `style_id` |
| Identity drifts across consecutive generations | Any character model without `soul_id` | Reference image is a weak identity anchor; multiple calls drift independently | Train a Soul Character with `show_characters(action='train')` and reuse the returned `soul_id` |
| `get_cost` preflight reports way less than actual spend | `nano_banana_pro` at 4k with `count > 1` | Preflight returns base cost, actual scales per-variant | Budget 2-3× the preflight when `count > 1` at 4k; or generate variants sequentially |
| UI text in generated phone screens is garbled/hallucinated | Any model except `nano_banana_pro` | Most image models can't render legible small text | Use NBP for any shot with on-screen text; or composite real UI in post (Phase 6 overlay) |

### Scouting protocol (run before Phase 3 Step 3.3 real generation)

For the **first shot only** of any new pipeline:

1. Take the top 2 recommended models from `models_explore`
2. Generate **1 variant each** at the lowest aspect/quality the model supports (~1-3 cr each)
3. Compare against the brief intent (composition match, no chrome bake, no collage, no UI hallucination)
4. Discard the misfit. Use the survivor for the full keyframe set.

Total scouting cost: ~3-5 cr. Saves the ~30-100 cr that would otherwise be burned discovering misfits at full quality.

For shots 2 and 3 of the same pipeline, reuse the surviving model — scouting once is enough.

---

## 11. User Orientation & Decision Ledger

This section parameterizes the Response Format Protocol from `SKILL.md`. Edit values here to tune the user-facing rhythm without touching orchestration logic.

### Header format defaults

```
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
📍 Phase {N} — {Phase Name}  ·  {sub-step}
{progress bar} {pct}% pipeline complete
💰 {used} cr used  ·  {remaining} cr left  ·  next ~{est} cr
✓ Locked: {ledger}
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
```

- Progress bar width: 12 characters
- Filled char: `█`  ·  Empty char: `░`
- Phases counted (11 total): 0, 1, 2, 2.5, 3, 3.5, 4, 5, 6, 6.5, 7
- Unknown field: `—` (em-dash). Never guess.

### Lockable decisions (defaults — extendable per project)

| Decision | Default lock behavior | Override syntax |
|---|---|---|
| Production mode (Premium / Optimized) | Locked at Phase 0 for whole pipeline | "switch to {mode}" |
| Cost strategy (Always-Full / Always-Draft / Mixed) | Locked after 2 consecutive same answers in cost gates | "draft for {phase/shot}" |
| Variant selection bias | Default `auto-recommended` after first variant pick; user can request `show-all` per shot | "show all variants for {shot}" |
| UI language strategy | Locked once user picks ES-as-is / PT-screenshots / composite | "swap UI to {lang}" |
| VO voice + model | Locked at Phase 5 first take if user accepts | "regenerate VO with voice {id}" |
| Character Bible attributes | Locked at Phase 1 | "edit character bible" |
| Aspect ratio set | Locked at Phase 0 from `aspect_ratios` field | "add {aspect} master" |
| Save path | Locked at Phase 0.1 | "move project to {path}" |

### Breakpoint conditions (auto-unlock and re-ask)

The skill must break a lock and re-confirm with the user when:

- A single generation costs > 2× the running per-generation average
- Total spent exceeds 75% of starting balance
- A model returns a misfit signal (collage / chrome / drift / garbled text)
- An API limit error fires (rate limit, credit cap, workspace constraint)
- The user explicitly types an override command (§G recovery)

When breaking a lock, show the user **why** in the breakpoint banner — not just "I need to ask again."

### Recovery commands (slash syntax)

| Command | Action |
|---|---|
| `/back-to phase {N}` | Re-open phase N decisions, rewind progress bar, mark downstream artifacts as stale |
| `/redo {artifact}` | Regenerate exactly one artifact (character-bible, brief, shot-list, keyframe-{n}, clip-{n}, vo, music) |
| `/show ledger` | Dump full decision ledger (every locked decision + when it was locked) |
| `/show budget` | Dump per-phase cost breakdown (estimated vs actual) |
| `/skip {phase}` | Mark a phase as user-opted-out, log in Airtable, advance to next |

Recovery commands always work. The skill cannot block them.

---

## 12. Multi-Service Cost Ledger

The skill stack consumes 4 paid services. Only Higgsfield exposes a runtime balance API; the other 3 require capture + tally. This section specifies the data model + query helpers.

### 12.1 — user-plans.json schema

Captured once per machine via the A.0 setup wizard. Stored at `~/.claude/skills/cinematic-ad-director-pro/user-plans.json`.

```jsonc
{
  "captured_at": "ISO-8601",
  "higgsfield": {
    "plan": "free | plus | ultra",
    "starting_balance_cr": 1010      // captured at first run; refresh manually if topped up
  },
  "elevenlabs": {
    "plan": "free | starter | creator | pro | scale",
    "monthly_quota_cr": 30000,        // from plan tier
    "accumulated_free_cr": 10000,     // any non-expiring credits the user has
    "current_remaining_cr": 39283,    // last queried value (if API-queryable)
    "api_queryable": true,            // false if user did not provide ELEVENLABS_API_KEY
    "last_query_at": "ISO-8601"
  },
  "airtable": {
    "plan": "free | plus | pro | team | enterprise",
    "records_limit": 1000,             // per base, per plan
    "attachments_gb_limit": 1          // per base, per plan
  },
  "claude_code": {
    "plan": "pro | max | api"
  },
  "other": []                          // free-form: anything else the user pays per-use for
}
```

### 12.2 — Per-service query helpers

**Higgsfield** (runtime balance):
```js
balance()  // MCP tool — returns { credits, subscription_plan_type }
```
Cost: 0 cr. Use at session open + after every paid call to keep header accurate.

**ElevenLabs** (subscription endpoint):
```bash
curl -sS -H "xi-api-key: $ELEVENLABS_API_KEY" \
  "https://api.elevenlabs.io/v1/user/subscription"
# Returns { tier, character_count, character_limit, next_character_count_reset_unix, ... }
```
Run at session open + after every TTS/music/SFX generation. Cache result in user-plans.json `current_remaining_cr` field.

**Airtable** (record count + storage):
```js
list_records_for_table({ baseId, tableId, returnFieldsByFieldId: false })
// Sum lengths across all tables in the active project's base
```
Sum attachment sizes from local `{project-slug}/output/` folder via Bash `du -sh`. Update header at phase boundaries (don't poll mid-phase).

**Claude Code** (not queryable runtime):
Track a rough proxy: count `tool_calls` made in the session. Display as `~{N} tool calls (plan: {tier})`. No marginal $ cost shown.

### 12.3 — Measured cost rates (calibration data)

Empirical rates from real ECC runs. Use these to estimate when API queries are not available.

**Higgsfield** — exact via preflight:
- Soul 2 image, 2k, 1 variant: ~0.12 cr
- NBP image, 2k, 1 variant: ~2 cr
- NBP image, 4k, 3 variants: ~12 cr actual (preflight under-reports as 4 cr — budget 3× the preflight when count>1 at 4k)
- Seedance 2.0 video, 1080p std, 4s, 1 clip: ~36 cr

**ElevenLabs** — calibrated 2026-05-21 against Starter plan:
- TTS: 1 cr per character (PT-BR multilingual_v2). Same rate for eleven_v3.
- Music: **~46 cr per second** generated (10s music gen = ~460 cr). Earlier estimates of 150 cr/sec were 3× too high — do not use those.
- SFX: ~10 cr per second (untested, this is the published default).

**Airtable** — no per-call cost; only quota:
- Each record creation: 0 cr, counts against base's record limit
- Each attachment upload: 0 cr, counts against base's GB limit

**Claude Code** — no per-call cost (sub-based); approximate:
- A typical cinematic-ad-director-pro full pipeline consumes ~150-300 tool calls (~500k-1M tokens)
- On Pro plan, this is a meaningful slice of weekly quota
- On Max plan, headroom is wide

### 12.4 — Quota warnings (when to nag the user)

The skill must surface a breakpoint banner when:

- Higgsfield: balance drops below 25% of starting OR next-phase estimate exceeds remaining
- ElevenLabs: monthly remaining drops below 10% of quota
- Airtable: records used > 80% of plan limit, OR attachment storage > 80% of plan limit
- Claude Code: tool-call count exceeds 400 in a single session (typical pipeline limit, plus warns of approaching plan caps)

Banner format: `⚠️ {service}: {metric} at {N}% — {action recommended}`.

### 12.5 — Sharing the skill across teammates

The skill itself (SKILL.md, best_practices.md) is portable and identical across users. What differs per user:

- `~/.env` — their API keys (each user provides own)
- `~/.claude/skills/cinematic-ad-director-pro/user-plans.json` — their subscription tiers (each user runs A.0 wizard once)

Neither file should be checked into git if the skill is shared via repo. Add both to `.gitignore`. On first run on a new machine, the wizard auto-triggers because `user-plans.json` is absent.
