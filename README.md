# 🎬 B-Roll Finder

**An agent skill that sources accurate, on-brand b-roll for talking-head video — and places each cutaway on the exact word.**

It reads your transcript, figures out what each line is *actually* about, routes every moment to the right kind of footage, and hands back a tight set of vetted candidates (you make the final pick). No keyword-matching, no random stock, no padding.

---

## ✅ What to use it for

It's built for **reference-rich talking-head video**, where the speaker names concrete things or makes verifiable claims. That's where b-roll has a *correct* answer the agent can nail.

**Best use cases:**

- **🎙️ Podcast & interview intros** — "In today's episode I'm joined by *[guest]*, founder of *[company]*…" Every name, company, role, and credential becomes a precise cutaway: the guest's face, their product UI, a receipt proving the claim.
- **📚 Tutorial & explainer intros** — "Today we're using *[tool]* to do *[thing]*." Product UIs, concept motion-graphics, and stat cards that make the setup land.
- **Competitor / drama videos** — receipt-heavy (tweets, headlines, reviews).
- **Storytelling / listicles** — people, products, historical moments, text cards.

**Where it shines:** the more proper nouns and provable claims a script has, the better. *Name it or prove it → great. Feel it → it defers to you.*

**Not for:** pure vibe/mood montages or non-verbal (music-only) videos — there b-roll is a taste call, and this skill hands the wheel back to you.

---

## ⚙️ How it works

```
transcript ─► interpret each beat ─► classify & route ─► source candidates ─► you pick ─► place on the word
```

1. **Understand the topic first.** It reads the *whole* transcript and states the thesis before sourcing anything. B-roll illustrates the **point**, not the nearest keyword. (A line like *"Zaria from Duolingo made their social pop off"* is about *the person and the account she built* — not the Duolingo logo.)
2. **Per-beat interpretation.** For every candidate moment it writes one sentence — *"what is this line actually about?"* — then sources *that*. If nothing accurately illustrates the real point, it **drops the beat**. No b-roll beats wrong b-roll.
3. **Classify & route.** Each moment goes to the source that has the *correct* answer for it (see the routing table below).
4. **Scope the search.** YouTube searches stay inside trusted/official sources — official channels for entities, authoritative sources for concepts — never random open-search.
5. **Score every candidate** on recency-fit, source authority, relevance, recognizability, and format-fit. Stale or off-topic candidates are dropped before you ever see them.
6. **Contact sheet → you pick.** It narrows the funnel and presents vetted candidates. **The agent never makes the final taste call — you do.**
7. **Place it precisely.** Picked clips are cut full-bleed and silent, then dropped on the exact spoken word (anchored with word-level timestamps, landing just *after* the word — never before), with soft dissolves where b-roll meets the talking head and clean cuts between back-to-back b-roll.

**Cadence:** front-loaded by default — a dense, punchy hook, then sparse and precise through the body. Accuracy over volume, always.

---

## 🎨 What kind of b-roll it sources

It deliberately **mixes the palette** — never leans on website screenshots for everything:

| Type | When | Example |
|---|---|---|
| **Faces (video)** | A named person is introduced | A live interview clip of the guest talking (never a frozen headshot) |
| **Product / UI** | A tool or company is named | The actual app UI / a real screen-recording of the product |
| **Receipts** | Something happening *now* — news, claims, drama, "people are saying" | Tweets, article headlines, reviews, search results — captured as clean screenshots |
| **Reference screenshots** | A specific thing is cited | The real post, essay, or page being referenced (not a synthetic card) |
| **Concept motion-graphics** | An abstract idea, process, or stat with no literal footage | Custom on-brand animations (e.g. via Remotion) — voice-agent waveforms, growth charts, "$300M+" stat cards |
| **Real / evocative footage** | A story, action, place, or mood | Stock that *conveys* the point (bakery, paperwork, an airport) — checked for watermarks and AI-slop |
| **Historical / products** | A historical moment or physical product | Canonical archival footage; clean product shots |
| **Memes / reactions** | A punchline or reaction beat | Curated from your own library (never auto-picked) |

---

## 🧭 The routing model

Every moment is split by **what decides the right clip**:

| Route | Trigger | Source |
|---|---|---|
| **Receipts** | Time-sensitive — drama, news, complaints, a current claim | Tweets / headlines / reviews, recency-sorted |
| **Entity** | A person, a physical product, or a historical moment | Official / authoritative channel (the canonical clip) |
| **Concept** | An abstract idea you'd have to *draw* | Custom motion-graphics — or real footage from the authoritative source |
| **Cultural / Meme** | A creator clip or joke where *taste* decides | Your own library / exemplars — surfaced for you to pick, never blind-picked |

**Litmus:** *Happening now?* → Receipts. *A person / product / event?* → Entity (official source). *An abstract idea?* → Concept (motion-graphics). *A reaction beat?* → Meme (your library).

---

## 🛠️ Under the hood

- **Transcription:** GPU Whisper (word-level timestamps) for precise anchoring.
- **Search & download:** `yt-dlp` (no API key) for YouTube; headless browser for public-page screenshots (with consent-wall handling via CDP).
- **Motion-graphics:** Remotion, rendered full-bleed and silent in your brand style.
- **Compositing:** `ffmpeg` — full-bleed cover-crop (never letterboxed), blurred-fill for partial-frame subjects, audio stripped from every clip.

---

## 📐 Principles it follows

- **The agent never picks the final b-roll — you do.** Its job is to narrow the funnel to vetted candidates.
- **Accuracy over volume.** Fewer, perfectly-accurate beats beat lots of mediocre ones. Drop a beat before padding it.
- **Full-bleed, always.** Every clip fills the frame edge-to-edge — never tiny, never letterboxed.
- **Land on the word, never before.** B-roll that appears before the word is said reads as a mistake.
- **Mix the palette.** If a plan is >60% website screenshots, it's wrong.

---

## License

MIT — see [LICENSE](LICENSE).
