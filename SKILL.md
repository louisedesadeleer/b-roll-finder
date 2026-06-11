# B-Roll Finder — skill methodology

> A reusable methodology for sourcing and placing b-roll on talking-head video.
> Genericized from a working agent skill. Adapt paths and brand tokens to your own setup.

## The one rule that governs everything

**The agent NEVER picks the final b-roll. The user does.** The right clip is often a taste call. This skill's job is to **narrow the funnel** — classify each moment, scope the search to trusted/authoritative sources, score candidates, and hand back a tight contact sheet. The user makes the final pick.

## Onboarding preferences (ask once, store in a profile)

Ask these on first run and write the answers to a profile file the skill loads every run:

1. **Audio** — does the user talk over b-roll (strip ALL audio: `ffmpeg -an`) or want the clip's sound?
2. **Stills motion** — static images get a **very subtle Ken Burns zoom-in by default** (~1.5%/sec, centered, capped ~2%/sec); offer opt-out to fully static. **Method matters more than the setting** — see "Stills motion" below.
3. **Source attribution** — credit each b-roll's source in small text bottom-right? Options: `off` · `white` · `black` · `auto` (contrast-pick per clip). The credit is the source's canonical name + platform ("Vinexposium / YouTube", "Decanter") — short, never a URL. If the footage already carries its own badge bottom-right, move the credit bottom-left for that clip. Note: an on-screen credit is etiquette, not a license — official/authoritative sources remain the real copyright posture.
4. **AI-generated b-roll** — off by default; opt-in only for beats with no real footage, via a quality model the user names. Never silently substitute cheap AI stock.

## Accuracy over volume

Fewer, perfectly-accurate beats beat lots of mediocre ones. The habit to kill is keyword-matching to hit a quota.

- **Every beat passes a per-citation interpretation** — write *"what is this line actually about?"* first, then source THAT. If you can't source something that accurately illustrates the real point, **drop the beat — don't pad.** No b-roll is better than wrong b-roll.
- **Default cadence = front-loaded:** dense, punchy b-roll in the intro/hook, then sparse and precise through the body.
- **But "skip" is the LAST resort, not the default.** Before proposing skip for a beat, walk the full palette out loud — receipt? headline? face? product UI? concept graphic? — and only skip when every lane genuinely fails. A rhetorical line with no proper noun is exactly when a *sentiment receipt* or a *concept graphic* shines.

## Understand the TOPIC first, then source the MEANING — not the keywords

1. Read the **whole transcript** and understand the thesis BEFORE sourcing anything. Context decides what illustrates each line.
2. **B-roll illustrates the POINT, not the words.** The proper noun in a sentence is often NOT the referent:
   - *"X from Company"* → the person and their work, not the company logo.
   - *"like Company did"* → the *thing* Company did.
   - *"what Person said"* → the said-thing (their quote/post/essay), not their face.
3. **Literal vs evocative:** concrete nouns (a product, a named person, an event) → show the actual thing. Actions & abstractions → show something that *conveys* it; don't reenact literally.
4. **Mandatory reference sweep — every beat:** *is there a REAL ARTIFACT behind this line?* A quote, a stat, "people are saying", a named company event, "I saw/read" — almost always means a tweet, headline, review, or post exists. That artifact is a default candidate; sourcing it is the skill's job, not an extra the user must request.

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

## Person clips — source the MOMENT, not the NAME

A clip merely *containing* the person is NOT relevant b-roll (the #1 person-clip failure):

1. Write the **mention-context** first: what is the line saying ABOUT the person — the trait, the action, the claim, the event?
2. **Query = person + mention-context, never the bare name.** "Steve Jobs iPhone keynote 2007", not "Steve Jobs". Bare-name searches return generic press-junket clips.
3. **Verify before presenting:** for each candidate, fill in *"in this clip, X is <doing/saying what, where, when>"* from the title/date/auto-subs. Can't fill it in → not a candidate.
4. No context match found → say so; offer the canonical clip explicitly flagged "generic", or switch palette to their work/quote/headline (often better anyway).
5. **"The show/podcast" ≠ "this episode"** — a reference to a show sources the show's persistent branding, not one episode's guest art.

## The palette — MIX it, never default to website screenshots

- **Faces (video)** — for a named person, a live clip of them talking (never a frozen headshot). Partial-frame / split-screen subjects → blurred-fill, never hard cover-crop. For a podcast/panel edit, the guests are ON CAMERA in the source recording — crop their tile as live video instead of hunting externally.
- **Product / UI** — the actual app UI or a real screen-recording (prefer own-recording > official channel > nothing; reject random third-party tutorials). **Demos must be the MOST RECENT available** — product UIs change fast; check upload dates, present them next to candidates.
- **Receipts** — tweets, headlines, reviews, search results. For a named company, prefer a **news headline about a real event** (IPO, funding, milestone) over the homepage.
- **Reference screenshots** — the real post/essay/page cited (an authentic screenshot beats a synthetic quote-card). **The subject's own website, captured full-page and cropped per viewport, is a goldmine** — history pages, team photos, maps, product pages.
- **Concept motion-graphics** — for ideas, charts, stats. Build on-brand; never synthetic-looking stock.
- **Real / evocative footage** — stock that conveys a story/action/mood. Eyeball every frame for watermarks, burned-in captions, and AI-slop.
- **Memes / reactions** — from the user's curated library only.

If a plan is >60% website screenshots, it's wrong.

## Motion-first — video beats a static page

When BOTH a moving and a static version of a source exist, take the moving one: the product's own demo video over its homepage; a real screen-recording of scrolling over a static capture; a live excerpt over a headshot. Stills stay right where READING is the point (a tweet, a headline, a review). Rubric tie-breaker: equal relevance → motion wins.

## Stills motion — sub-pixel subtle zoom only; ffmpeg `zoompan` is banned

Hard-won distinction — **the ban is on the METHOD, not the effect**:

- **ffmpeg `zoompan` / crop-pans / scroll-pans are BANNED, at any speed, with any supersampling.** They sample on integer pixel steps, so even a ~1%/sec zoom stutters. Tested exhaustively (4× lanczos supersample → zoompan → downscale): still shaky. Don't burn time re-attempting.
- **Sub-pixel rendering is the legitimate path** and produces a smooth subtle zoom:
  - **PIL recipe (proven):** per-frame `img.resize((W,H), Image.LANCZOS, box=<float coords>)` — float box coordinates = sub-pixel sampling — piped as rawvideo into x264. ~30 lines of Python.
  - **Remotion** (CSS transform scale) is equally valid when a project is scaffolded.
- Defaults: **zoom-IN only, centered, ~1.5%/sec, cap ~2%/sec, stills only** — no pans, no zoom-outs, never synthetic motion on top of real video. Per-profile opt-out → fully static.

## Eval rubric — score every candidate before showing it

Score 1–5 and drop anything below the bar:

1. **Recency fit** — is the beat time-sensitive or evergreen? Time-sensitive + old clip = FAIL.
2. **Source authority** — primary/official/reputable vs random creator.
3. **Relevance** — depicts the exact named thing, not a loose association. Person clips: relevance = context-match.
4. **Recognizability / impact** — reads instantly, screenshots clean.
5. **Format fit** — silent-able, ~2–6s, full-bleed-able, ≥720p.

Check **time-sensitivity first** — a dated tweet from this month beats a years-old YouTube clip for a current story.

## Once a beat is agreed, sourcing it is the AGENT's job — the escalation ladder

"Login-walled" is a claim you prove by attempting, not a label for punting. Exhaust ALL of these before handing a beat back to the user:

1. **Local artifacts** — grep the user's notes/downloads/prior sessions for the exact link or handle.
2. **Identity hunt** — web-search via mention-context (distinctive phrases beat bare names); resolve shortlinks; oEmbed endpoints identify authors without auth.
3. **Plain `yt-dlp`** — public profiles/videos usually need NO login.
4. **`yt-dlp --cookies-from-browser`** — the user's logged-in cookies beat most walls.
5. **Headless Chrome + CDP** — for captures behind consent walls: click "accept" in EVERY frame context (CMPs render in iframes), with overlay-removal as fallback; then VERIFY the capture by looking at it. Never deliver a screenshot you haven't visually inspected.
6. **The user's logged-in browser** (browser-automation MCP) when available.
7. Only if ALL genuinely fail → a precise you-source list, with a note of what was tried.

## Placement timing — land ON or just AFTER the word, never before

- Anchor to when the keyword is **spoken**, then add a small lead (~+0.2s) so the cut lands as/just after it. B-roll *before* the word reads as a mistake.
- **Timing engine: forced alignment, not Whisper timestamps.** Whisper word times are ±100–300ms with pauses embedded inside word durations — fine for reading, not for anchoring. Use **MFA (Montreal Forced Aligner)**: Whisper produces the transcript *text*, MFA aligns it to the audio and returns ~10–20ms word boundaries (with true silences as explicit intervals). The keyword's MFA end time + ~0.2s is the anchor. (Same architecture as the [cut-video](https://github.com/louisedesadeleer/cut-video) skill — see its README for the full MFA recipe.)
- Whisper `--word-timestamps` is the fallback when MFA isn't installed — bias later (+0.3–0.5s) since whisper word-ends run early, and say so in the plan.
- For punchlines, land on the beat *after* the punchline.
- **Connect adjacent b-rolls:** if two cutaways sit closer than ~a half-sentence apart, extend the first to the second's start — a <2s flash of the speaker's face between them reads as an error. (Extend the earlier clip; never start the next one before its keyword.)

## Composition — full-bleed, no composites

- **Cover-crop, never letterbox:** `scale=W:H:force_original_aspect_ratio=increase,crop=W:H`.
- **No agent-built composites:** no split-screens, no 2-ups, no clever framing. Two referenced people → sequential full-bleed singles. The ONE allowed treatment beyond cover-crop is **blurred-fill** (enlarged blurred copy behind a fitted clip).
- **Partial-source / split-screen subjects:** blurred-fill, never cover-crop (cover-cropping a half-frame zooms hard into a face).
- Don't upscale a tiny source to full-bleed — find a higher-res source.

## Iteration discipline — the b-roll manifest

Across re-renders the #1 failure is silently DROPPING beats the user already approved. Keep a **`BROLL-MANIFEST.md` next to the deliverable**: one row per beat (in/out · beat · asset · status incl. which version approved it) plus a "Removed (do not re-add)" list. Before EVERY re-render: read the manifest, verify every approved beat is in the new cut, add the new ones, update statuses. Approved b-roll never disappears without the user explicitly cutting it.

## Self-verification — look at every cut before the user does

After every render, extract a frame at **every beat's midpoint AND every joint** (b-roll↔b-roll, b-roll↔face), tile them into a grid, and LOOK at it. This catches: shot-cuts inside a window flipping to wrong content, burned-in captions/watermarks, stray overlays, wrong crops, face flashes. Fix and re-verify. A render isn't done until the grid is clean.

## Fetching & formatting (editor-friendly, silent, full-bleed)

- Don't grab low-res pre-merged streams — select a real stream (`-f "bv*[height<=1080]+ba/b"`).
- Don't let `--download-sections` be the final cut (variable framerate stutters) — download the short clip, then trim with a re-encode.
- Standard format: constant fps, cover-crop full-bleed, audio stripped:
  ```
  ffmpeg -ss <in> -t <dur> -i full.mp4 \
    -vf "fps=30,scale=1920:1080:force_original_aspect_ratio=increase,crop=1920:1080,setsar=1" \
    -an -c:v libx264 -crf 18 -preset slow -pix_fmt yuv420p -movflags +faststart out.mp4
  ```
- Attribution on → composite the small credit label (PIL-rendered PNG, ~70% opacity) via `overlay` during formatting (some ffmpeg builds lack `drawtext`).

## Workflow summary

1. **Load profile / set topic** — preferences + trusted sources tagged for this video's topic.
2. **Ask style + cadence** — format (podcast / tutorial / fast-cut / heavy-intro) and density. These override genre defaults.
3. **Get the transcript** — paste, pull from the editor, or transcribe (GPU Whisper, word-level). Long-form (>~10 min): score segments for b-roll value and select the high-value ones first — don't uniformly b-roll an hour.
4. **Classify + propose (no fetching yet)** — annotate each beat with its interpretation, route, the reference sweep result, and the palette mix. **Present the plan and wait for the user to react** before sourcing.
5. **Constrained search** — scoped to trusted/official sources; score candidates; verify person clips against mention-context; drop the weak ones.
6. **Contact sheet → user picks.**
7. **(Optional) place & render** — cut full-bleed + silent, anchored on the word, adjacent beats connected, manifest updated, **self-verification grid before delivery**.

## Tools

- **Transcription:** GPU Whisper for transcript text (large model — the transcript drives *understanding*, so text accuracy matters).
- **Anchor timing:** MFA forced alignment (~10–20ms word boundaries); Whisper word-timestamps as fallback.
- **Search / download:** `yt-dlp` (no API key); headless browser + CDP for public-page screenshots (consent walls: click accept in every frame context, verify visually).
- **Motion-graphics:** Remotion (or similar), rendered full-bleed + silent.
- **Stills zoom:** PIL float-box resize piped to x264 (sub-pixel; never `zoompan`).
- **Compositing:** `ffmpeg`; `ImageMagick` for contact sheets.
