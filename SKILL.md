---
name: find-broll
description: Source b-roll for a video edit — classify each moment, scope the search, return vetted candidates, place on the word. Use when the user says "find b-roll", "/find-broll", "source clips for this edit", or wants footage/memes/screenshots to lay over a talking-head video.
---

# B-Roll Finder — skill methodology

> A reusable methodology for sourcing and placing b-roll on talking-head video.
> Genericized from a working agent skill. Adapt paths and brand tokens to your own setup.

## ⛔ USER OVERRIDES ARE LAW — and they persist

When the user states a preference or ban mid-run ("no memes", "no text cards", "nothing political"), it applies IMMEDIATELY and for the rest of the session, AND gets written into the profile's Guardrails right then — not at the end. A banned category is never sourced again, never proposed again, never "just one candidate to check". Violating a stated ban is the worst failure this skill can make: it tells the user the agent doesn't listen.

## ⛔ STEP 0 — ONBOARDING GATE (run this check before ANYTHING else, every session)

Open the profile (TASTE.md or the user's fork). **The ONLY thing that skips onboarding is a literal `Confirmed-by: <name> (<date>)` line at the top of the profile.** No line → ask the four onboarding questions (below) out loud, write the answers into the profile, add the line. This is NOT optional and NOT inferable:

- Do **NOT** infer the answers from CLAUDE.md, memory, prior conversations, or this profile's prose — even if you are confident you know them. Asking IS the feature.
- The shipped TASTE.md is an EXAMPLE, **even if the current user is its original author**. Author ≠ confirmed.
- A CLAUDE.md pointer that says "load my taste profile" does not count as confirmation either — check for the line.
- "redo my profile" re-triggers onboarding at any time.

## The one rule that governs everything

**The agent NEVER picks the final b-roll. The user does.** The right clip is often a taste call. This skill's job is to **narrow the funnel** — classify each moment, scope the search to trusted/authoritative sources, score candidates, and hand back a tight contact sheet. The user makes the final pick.

## The taste profile — load it before sourcing anything

Curation is half the skill. Every run starts by loading a **taste profile**: the b-roll fingerprint (which types, how fast to cut), the trusted-source list tagged by topic, and the guardrails. This repo ships with a working default — **[TASTE.md](TASTE.md)**, revealed from real published videos — so the skill has good taste out of the box. Use it as-is until the user builds their own (TASTE.md's "Make it yours" section covers the import-then-prune mechanism: pull their YouTube subscriptions via `yt-dlp --cookies-from-browser`, prune together, then reveal their fingerprint from their own published videos).

## Onboarding preferences (ask once, store in the profile)

**When onboarding fires:** the shipped [TASTE.md](TASTE.md) is an EXAMPLE profile (the author's), not the user's. On first run, check the profile for a `Confirmed-by:` line naming the current user. Absent → run onboarding: ask the questions below, write the answers into the user's profile (their fork of TASTE.md), and add `Confirmed-by: <user> (<date>)` at the top. Present → skip onboarding and just load it. The user can re-trigger anytime with "redo my profile".

Ask these on first run and write the answers into the taste profile (TASTE.md ships with one set of answers; confirm they fit):

1. **Audio** — does the user talk over b-roll (strip ALL audio: `ffmpeg -an`) or want the clip's sound?
2. **Stills motion** — static images get a **very subtle Ken Burns zoom-in by default** (~1.5%/sec, centered, capped ~2%/sec); offer opt-out to fully static. **Method matters more than the setting** — see "Stills motion" below.
3. **Source attribution** — credit each b-roll's source in small text bottom-right? Options: `off` · `white` · `black` · `auto` (contrast-pick per clip). The credit is the source's canonical name + platform ("Vinexposium / YouTube", "Decanter") — short, never a URL. If the footage already carries its own badge bottom-right, move the credit bottom-left for that clip. Note: an on-screen credit is etiquette, not a license — official/authoritative sources remain the real copyright posture.
4. **AI-generated b-roll** — off by default; opt-in only for beats with no real footage, via a quality model the user names. Never silently substitute cheap AI stock.

## 🔓 Standing authorization — invoking the skill IS the permission

Do NOT ask conversational permission for access or searches while sourcing — running /find-broll already authorizes the skill to act on the user's behalf. Never ask "want me to search/look up/access X?" for:

- Web searches, news/headline lookups, oEmbed/embed endpoints
- `yt-dlp` metadata searches and downloads of public videos/channels
- Headless-browser captures of public pages (incl. clicking through cookie/consent walls)
- The user's logged-in browser or browser cookies **for sourcing/reading** (profiles, posts, walled pages)
- Reading local files, transcripts, prior project folders, the user's asset library

Just do it and show the result. The user's taste checkpoints stay: the **plan approval** before sourcing and the **final pick** on taste-route beats. The ONLY things that still warrant a question: publishing/posting anything outward, paid actions, deleting user files, and installing heavy dependencies. Ask-permission friction everywhere else is a bug, not politeness.

## ⚡ LEAN PATH IS THE DEFAULT — the funnel is for taste calls, not everything

The skill earned trust through restraint; heavyweight process makes output WORSE, not better (tested 2026-06-11: a full-funnel run took ~10× longer and shipped worse composition than the lean run). Defaults:

- **Plan approval IS the pick.** When the user approves the beat plan ("go for it"), source **ONE best candidate per beat** for objective routes (Entity/Receipts/Product) and place it. NO multi-candidate sourcing, NO contact sheets. (Contact sheets exist only when the user EXPLICITLY asks for options on a beat — and meme/cultural beats are never auto-sourced at all; see the routing table.)
- **Research depth is bounded at plan time.** Verify referents enough to source accurately; do NOT exhaust every sourcing route before the plan. The escalation ladder fires only AFTER the user has agreed a beat and the easy routes failed — never speculatively.
- **Per-beat time box:** if a single beat's sourcing exceeds ~5 minutes, place the best-available candidate or drop the beat and flag it. One stubborn artifact must not stall the edit.
- **One verify pass + targeted fixes** beats endless polishing. Render → grid → fix the failures → re-verify only the fixed beats.
- **NO RETRIES — first failure switches, second failure drops.** A method that fails once is NOT retried (not with tweaked flags, not "one more time") — switch to a different method immediately. If the second method also fails, DROP the beat, flag it in the plan, move on. A beat is never worth a third attempt; the edit ships without it. This applies to everything: captures, downloads, renders, browser actions. (Louise: "stop doing things that keep failing, just skip to save time.")
- **Never auto-install heavy dependencies mid-run** (conda, aligners, etc.) — use the fallback, note it, offer the install after delivery.

### Source by SOURCE, not by beat (the big sourcing speed-up)
After the plan is approved, **cluster beats by where their asset lives, then fetch each source ONCE**:
- One full-page capture of the subject's website often covers 4–6 beats (history, team photo, map, packshots) — crop per beat from the same capture.
- One official channel/film often covers several beats (an aerial, a close-up, a process shot) — download once, cut multiple windows.
- Run all downloads as ONE batched background job (metadata-only search first, download only the chosen items); run captures as a second batch in parallel. Never fetch serially beat-by-beat.
- Cache raw downloads next to the project (`assets/raw/`) so re-renders and v2s never re-fetch.

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

**⛔ Cards never replace real footage of a literal thing.** If the beat names a concrete entity (grape varieties, a product, a place) and real footage exists, a generated text card is a FAIL — even if the user's fingerprint shows they like cards. Cards are for ideas with NO literal footage. (2026-06-11: a "three grapes" beat got a navy text card while the official film had the actual clusters — wrong.)
| **Cultural / Meme** | A creator clip or joke where *taste* decides | **NEVER searched or fetched by the agent.** Propose the MOMENT + register ("punchline at 2:31, deadpan") in the plan; the user supplies the meme from their library — or strikes the beat. Agent-found memes have failed every time they've been tried. |

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

- **Faces (video)** — for a named person, a live clip of them talking (never a frozen headshot). Partial-frame / split-screen subjects → blurred-fill, never hard cover-crop. For a podcast/panel edit, the guests are ON CAMERA in the source recording — crop their tile as live video instead of hunting externally. **⛔ But NEVER cut to a speaker's own tile as b-roll inside the same video they're speaking in** — the audience is already looking at them; it reads as a glitch. Tile-cropping is for using their face in OTHER videos.
- **Product / UI** — the actual app UI or a real screen-recording (prefer own-recording > official channel > nothing; reject random third-party tutorials). **Demos must be the MOST RECENT available** — product UIs change fast; check upload dates, present them next to candidates.
- **Receipts** — tweets, headlines, reviews, search results. For a named company, prefer a **news headline about a real event** (IPO, funding, milestone) over the homepage.
- **Reference screenshots** — the real post/essay/page cited (an authentic screenshot beats a synthetic quote-card). **The subject's own website, captured full-page and cropped per viewport, is a goldmine** — history pages, team photos, maps, product pages.
- **Concept motion-graphics** — for ideas, charts, stats. Build on-brand; never synthetic-looking stock.
- **Real / evocative footage** — stock that conveys a story/action/mood. Eyeball every frame for watermarks, burned-in captions, and AI-slop.
- **Memes / reactions** — from the user's curated library only; the agent proposes the moment, never the meme. If the profile or the user says no memes: the category does not exist.

If a plan is >60% website screenshots, it's wrong.

## Cadence guardrails — restraint reads as taste

- **Beats run ~2–4s.** Anything under ~1.2s is unreadable (no sub-second shots except inside ONE deliberate burst montage).
- **Max ONE burst montage per intro** (3–4 quick stills on a list-beat). Two+ bursts = chaos.
- **Max ONE long hold (>5s) per intro**, and only on a genuine explainer beat. An 8-second static map is dead air.
- When in doubt: fewer, real, obvious. The strongest edits use one clearly-right asset per beat, not the most assets.

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
- **Timing source: Whisper word-level timestamps** (`--word-timestamps True`, one pass during transcription — no extra tooling). Find the keyword's word time and anchor +0.2–0.5s after it. Whisper word-ends run slightly early and embed pauses inside word durations, so **bias LATER when unsure** — late reads as intentional, early reads as a mistake.
- _(MFA forced alignment was tried for ~10–20ms precision and REMOVED 2026-06-11 — the runtime cost wasn't worth it; whisper + later-bias is accurate enough in practice. Don't reintroduce it.)_
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

After every render, extract a frame at **every beat's midpoint AND every joint** (b-roll↔b-roll, b-roll↔face), tile them into a grid, and LOOK at it. Fix and re-verify. A render isn't done until the grid is clean.

**Explicit AUTO-REJECT checklist — a frame containing ANY of these fails, no judgment call:**
1. Burned-in captions/subtitles from the source clip
2. Name-tags / lower-thirds identifying strangers
3. Watermarks or channel logo bugs (corner badges from official bodies are a flagged exception, not a free pass)
4. The speaker's own face/tile appearing as b-roll in their own video
5. A generated card where the beat names a literal, filmable thing
6. Template-looking composites (two portraits side by side, collage cards)
7. Letterboxing, tiny floating content, or visible blur-edge framing errors
8. A split-second sliver of talking head (<1s) between two cutaways — connect them: extend the first beat to the second's start (never start the second early)

Grade the grid against this list line by line — "looks fine" without the list is how watermarked clips ship.

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

1. **Load the taste profile / set topic** — [TASTE.md](TASTE.md) (or the user's fork): fingerprint, preferences, and trusted sources filtered to this video's topic tag.
2. **Ask style + cadence** — format (podcast / tutorial / fast-cut / heavy-intro) and density. These override genre defaults.
3. **Get the transcript** — paste, pull from the editor, or transcribe (GPU Whisper, word-level). Long-form (>~10 min): score segments for b-roll value and select the high-value ones first — don't uniformly b-roll an hour.
4. **Classify + propose (no fetching yet)** — annotate each beat with its interpretation, route, the reference sweep result, and the palette mix. **Present the plan and wait for the user to react** before sourcing.
5. **Constrained search** — scoped to trusted/official sources; score candidates; verify person clips against mention-context; drop the weak ones.
6. **Contact sheet → user picks.**
7. **(Optional) place & render** — cut full-bleed + silent, anchored on the word, adjacent beats connected, manifest updated, **self-verification grid before delivery**.

## Tools

- **Transcription:** GPU Whisper for transcript text (large model — the transcript drives *understanding*, so text accuracy matters).
- **Anchor timing:** Whisper word-level timestamps + later-bias (+0.2–0.5s past the keyword).
- **Search / download:** `yt-dlp` (no API key); headless browser + CDP for public-page screenshots (consent walls: click accept in every frame context, verify visually).
- **Motion-graphics:** Remotion (or similar), rendered full-bleed + silent.
- **Stills zoom:** PIL float-box resize piped to x264 (sub-pixel; never `zoompan`).
- **Compositing:** `ffmpeg`; `ImageMagick` for contact sheets.
