# B-Roll Finder — skill methodology

> A reusable methodology for sourcing and placing b-roll on talking-head video.
> Genericized from a working agent skill. Adapt paths and brand tokens to your own setup.

## The one rule that governs everything

**The agent NEVER picks the final b-roll. The user does.** The right clip is often a taste call. This skill's job is to **narrow the funnel** — classify each moment, scope the search to trusted/authoritative sources, score candidates, and hand back a tight contact sheet. The user makes the final pick.

## Accuracy over volume

Fewer, perfectly-accurate beats beat lots of mediocre ones. The habit to kill is keyword-matching to hit a quota.

- **Every beat passes a per-citation interpretation** — write *"what is this line actually about?"* first, then source THAT. If you can't source something that accurately illustrates the real point, **drop the beat — don't pad.** No b-roll is better than wrong b-roll.
- **Default cadence = front-loaded:** dense, punchy b-roll in the intro/hook, then sparse and precise through the body.

## Understand the TOPIC first, then source the MEANING — not the keywords

1. Read the **whole transcript** and understand the thesis BEFORE sourcing anything. Context decides what illustrates each line.
2. **B-roll illustrates the POINT, not the words.** The proper noun in a sentence is often NOT the referent:
   - *"X from Company"* → the person and their work, not the company logo.
   - *"like Company did"* → the *thing* Company did.
   - *"what Person said"* → the said-thing (their quote/post/essay), not their face.
3. **Literal vs evocative:** concrete nouns (a product, a named person, an event) → show the actual thing. Actions & abstractions → show something that *conveys* it; don't reenact literally.

## Routing — three+ kinds of b-roll go to three+ sources

Classify EVERY moment before searching:

| Route | Trigger | Source |
|---|---|---|
| **Receipts** | Time-sensitive — drama, news, complaints, a current claim/stat | Tweets / article headlines / reviews, recency-sorted, captured as clean screenshots |
| **Entity** | A **person**, a **physical product**, or a **historical moment** | The official / authoritative channel — the *canonical* clip, not a random upload |
| **Concept** | An abstract idea you'd have to *draw* (a process, a mental model, a stat) | Custom motion-graphics (e.g. Remotion) in your brand style — or real footage from the authoritative source |
| **Cultural / Meme** | A creator clip or joke where *taste* decides | The user's own library / exemplars; surfaced for the user to pick, never blind-picked |

**Litmus (in order):** *Happening now?* → Receipts. *A person / product / event?* → Entity (official source). *An abstract idea?* → Concept (motion-graphics). *A reaction beat?* → Meme (user's library).

YouTube's sweet spot is the Entity route (people, products, historical moments). Don't force it onto abstract ideas — those are Concept jobs.

## The palette — MIX it, never default to website screenshots

- **Faces (video)** — for a named person, a live clip of them talking (never a frozen headshot). Partial-frame / split-screen subjects → blurred-fill, never hard cover-crop.
- **Product / UI** — the actual app UI or a real screen-recording (prefer own-recording > official channel > nothing; reject random third-party tutorials).
- **Receipts** — tweets, headlines, reviews, search results. For a named company, prefer a **news headline about a real event** (IPO, funding, milestone) over the homepage.
- **Reference screenshots** — the real post/essay/page cited (an authentic screenshot beats a synthetic quote-card).
- **Concept motion-graphics** — for ideas, charts, stats. Build on-brand; never synthetic-looking stock.
- **Real / evocative footage** — stock that conveys a story/action/mood. Eyeball every frame for watermarks and AI-slop.
- **Memes / reactions** — from the user's curated library only.

If a plan is >60% website screenshots, it's wrong.

## Eval rubric — score every candidate before showing it

Score 1–5 and drop anything below the bar:

1. **Recency fit** — is the beat time-sensitive or evergreen? Time-sensitive + old clip = FAIL.
2. **Source authority** — primary/official/reputable vs random creator.
3. **Relevance** — depicts the exact named thing, not a loose association.
4. **Recognizability / impact** — reads instantly, screenshots clean.
5. **Format fit** — silent-able, ~2–6s, full-bleed-able, ≥720p.

Check **time-sensitivity first** — a dated tweet from this month beats a years-old YouTube clip for a current story.

## Placement timing — land ON or just AFTER the word, never before

- Anchor to when the keyword is **spoken**, then add a small lead (~+0.2–0.5s) so the cut lands as/just after it. B-roll *before* the word reads as a mistake.
- Use **word-level timestamps** (e.g. Whisper `--word-timestamps`) for precise anchoring — the transcript cue start is usually a beat too early.
- For punchlines, land on the beat *after* the punchline.

## Full-bleed rule — b-roll always fills the frame

- **Cover-crop, never letterbox:** `scale=W:H:force_original_aspect_ratio=increase,crop=W:H`.
- **Partial-source / split-screen subjects:** blurred-fill, never cover-crop (cover-cropping a half-frame zooms hard into a face). Crop the region, fit at natural scale, fill the rest with a blurred background.
- Don't upscale a tiny source to full-bleed — find a higher-res source.

## Transitions — soft only where b-roll meets the face

- Keep cuts **clean between back-to-back b-roll** (no gap, or the bare talking-head flashes through for a few frames).
- Use a short **cross-dissolve (~0.1–0.15s) only at b-roll↔face boundaries** to soften the snap — never a slow Ken Burns zoom (sub-pixel `zoompan` jitter looks shaky).

## Fetching & formatting (editor-friendly, silent, full-bleed)

- Don't grab low-res pre-merged streams — select a real stream (`-f "bv*[height<=1080]+ba/b"`).
- Don't let `--download-sections` be the final cut (variable framerate stutters) — download the short clip, then trim with a re-encode.
- Standard format: constant fps, cover-crop full-bleed, audio stripped:
  ```
  ffmpeg -ss <in> -t <dur> -i full.mp4 \
    -vf "fps=30,scale=1920:1080:force_original_aspect_ratio=increase,crop=1920:1080,setsar=1" \
    -an -c:v libx264 -crf 18 -preset slow -pix_fmt yuv420p -movflags +faststart out.mp4
  ```

## Workflow summary

1. **Load taste profile / set topic** — scope every search to trusted sources tagged for this video's topic.
2. **Ask style + cadence** — format (podcast / tutorial / fast-cut / heavy-intro) and density. These override genre defaults.
3. **Get the transcript** — paste, pull from the editor, or transcribe (GPU Whisper, word-level).
4. **Classify + propose (no fetching yet)** — annotate each beat with its interpretation, route, and the palette mix. **Present the plan and wait for the user to react** before sourcing.
5. **Constrained search** — scoped to trusted/official sources; score candidates; drop the weak ones.
6. **Contact sheet → user picks.**
7. **(Optional) place** — cut full-bleed + silent, anchored on the word, soft dissolves at face boundaries.

## Tools

- **Transcription:** GPU Whisper (word-level timestamps).
- **Search / download:** `yt-dlp` (no API key); headless browser for public-page screenshots (handle consent walls via CDP).
- **Motion-graphics:** Remotion (or similar), rendered full-bleed + silent.
- **Compositing:** `ffmpeg`; `ImageMagick` for contact sheets.
