---
name: presentation-style
description: "Apply this presentation style whenever creating slides, decks, or a PowerPoint/.pptx presentation — ESPECIALLY for technical talks, workshops, system walk-throughs, or any deck that explains how something works. Trigger this whenever the user asks to make/build/design a presentation, deck, or slides, even if they don't name a style. It layers a specific visual + writing style on top of the base pptx skill: light background, a top progress bar of chapters (current one highlighted, not-yet-reached ones faded), architecture diagrams built from official vendor icons, one-component-at-a-time spotlight reveals on a single unchanging diagram, a running analogy that carries the whole talk, plain spoken-language slide text, and a deliberate avoidance of the tells that make slides look AI-generated. Use it together with the base pptx skill, not instead of it."
---

# Presentation Style

This skill is a **style layer**. The base `pptx` skill (`/mnt/skills/public/pptx/SKILL.md`) still owns *how* a `.pptx` gets generated, rendered, and QA'd — read it and use it for all the mechanics. This skill owns *what the deck should look and read like*. When both apply, follow the base skill's technical workflow and this skill's style rules.

The style comes from studying a strong technical workshop deck. The goal is decks that a first-time listener can follow without getting lost or overwhelmed — and that don't look or read like they were auto-generated.

## The core idea

A good technical deck feels like a person walking you through a system, not a document dumped onto slides. Five habits create that feeling, and every rule below serves one of them:

1. **Anchoring** — keep one diagram on screen and move a spotlight across it, instead of flashing a new picture every slide. Listeners build one mental map instead of re-orienting constantly.
2. **Orientation** — a persistent progress bar so listeners always know where they are and how much is left.
3. **Credibility** — real official icons, not improvised shapes or generic circles.
4. **Concreteness** — a running analogy and plain language, so abstract ideas land as pictures in the listener's head.
5. **Restraint** — a clean light layout that avoids the decorations that scream "AI made this."

---

## 1. Light background, restrained accent system

Use a **light background** (white `FFFFFF` or a very light neutral). Body text is **dark blue-grey (e.g. `1F2A37`), never pure black `000000`** — pure black on white is harsh to read.

Pick **one primary accent color + one secondary/emphasis color** for the whole deck, and use them with restraint:
- Primary accent: section titles, the active progress-bar chapter, the currently-spotlighted diagram element.
- Secondary/emphasis: a second logical group in a diagram, a key arrow, one stat you want to pop.

Do **not** make the deck one flat sheet of black-on-white — that reads as lifeless. The accent color is what keeps a light deck feeling alive. But resist using more than two accents; more colors read as noise, not design.

> Why not a dark background? A dark "stage" makes bright diagrams pop dramatically, but light decks read better in bright rooms, photograph and print cleanly, and match the native look of official icons (which are designed for white backgrounds). We deliberately trade a little drama for legibility.

---

## 2. Top progress bar (chapter orientation)

Put a **horizontal chapter bar across the top** of every content slide, listing the deck's main sections in order (e.g. `1. 研究流程　2. 第一週進度　3. 結論`).

- The **current chapter is highlighted** — solid fill in the primary accent, high-contrast text.
- **Chapters not yet reached are faded** — light tint / low-contrast text.
- Already-passed chapters can be a middle state (visible but muted) or the same faded style — pick one and stay consistent.

This is the *chapter-level* version of the spotlight idea in rule 4: at any moment, exactly one thing is lit and the rest recede. Keep the bar in the same position and size on every slide so it reads as a stable frame, not new content.

---

## 3. Architecture-diagram-driven, with official icons

Explain systems **through a diagram**, not through bullet lists. When a slide is about how pieces connect, draw the pieces and the connections.

**Icons must be real, official ones** — this is general, not AWS-specific. Whatever the technology (AWS, Azure, GCP, Kubernetes, Docker, React, ChatGPT/OpenAI, Postgres, anything with an official mark), get the genuine icon:

1. Find the official source and fetch the real file (SVG preferred, PNG ok). Examples of where to look: **AWS Architecture Icons**, **Azure / GCP official icon sets**, **kubernetes.io** and the CNCF artwork repo for k8s, a project's official brand/press page or GitHub for its logo.
2. Use vendor **standard colors** — they carry meaning and recognition (e.g. AWS DynamoDB purple, Lambda orange, S3 green).
3. **Never hand-draw or "generate" a brand logo.** A model-drawn logo comes out distorted, mis-colored, or simply wrong, and instantly looks fake. If the genuine icon truly can't be fetched, fall back to a **neutral generic shape** (a plain labeled box) and note that it's a stand-in — do not fake the brand mark.
4. **Never use a generic colored circle as a substitute for a real icon.** Rows of same-colored circles with a word inside are a classic AI-deck tell (see rule 8).

If you have web access, fetch icons during the build. If you don't, tell the user which icons you couldn't retrieve and used neutral boxes for, so they can drop the real files in.

---

## 4. One-component-at-a-time spotlight reveal

This is the signature technique. When walking through a diagram with several parts, **do not swap in a new image per slide.** Instead keep the *same* diagram and move a spotlight across it, one slide per component.

Since `.pptx` has no live animation from this environment, simulate it with **a sequence of near-identical slides**:

- Slide *i* keeps the whole diagram in place.
- The **component being discussed is at full strength**: full color/opacity, plus a subtle highlight (a soft outline, a light tinted backing shape, or a gentle shadow).
- **Every other component is de-emphasized**: reduced opacity and desaturated toward light grey.
- Connecting lines/arrows into the active component can stay dark; unrelated ones fade with their components.

On a **light** background the highlight is "keep this element saturated while the rest go pale grey" — do **not** use a white outline as the highlight (it's invisible on white); use the accent color, a light tinted backing, or a soft shadow instead.

Keep positions identical across the sequence so nothing jumps — only emphasis changes. The listener's eyes stay put; only the spotlight moves.

---

## 5. Build complex diagrams up progressively

For a diagram that ends up busy, **don't reveal it fully formed.** Start from the simplest skeleton and add one piece at a time across slides, the way a person would draw it on a whiteboard while explaining.

Example rhythm for a client–server picture: two boxes → add the request arrow → add the response arrow → add the middle platform → add labels. By the last slide the full picture is built, but the listener assembled it with you and understands every part.

Use **color to separate logical groups** within a diagram (e.g. one leg of a round-trip in the accent color, the other leg in the secondary color) so a multi-step flow reads as distinct steps, not one tangled line.

---

## 6. One running analogy for the whole talk

Wrap the dry material in **one concrete, memorable analogy and carry it through the entire deck** — not a different metaphor every slide. A single sustained theme (characters, a story, a running joke) is something a human presenter does and an auto-generated deck almost never does; it's a big part of not feeling AI-made.

Anchor each abstract concept to something the audience can picture. Prefer a concrete before/after the audience can *see* (e.g. "ordinary article → LLM → better article" shown with real icons) over an abstract definition.

---

## 7. Plain, spoken slide text (this is what makes it understandable)

AI-sounding text is the #1 reason a deck is hard to follow: technically correct, professional-looking, and yet the listener can't picture what it means. Slide text must be written **for someone hearing this for the first time, not for a paper reviewer.** Understood-at-a-glance beats sounds-impressive.

Rules for on-slide wording:
- **One idea per line.** Don't stack three concepts into one long clause with 透過…以…進而… chains. People speak in short pieces; slides should too.
- **Concrete over abstract.** If you can show an example, a number, or a picture, do that instead of naming a concept. "加了這個之後，速度快一倍" beats "效能得以顯著提升".
- **Everyday verbs.** "讓它變快" not "實現效能優化"; "幫你整理" not "賦能". Plain words, real subjects, real actions.
- **Plain words first, jargon second.** Lead with the everyday phrasing and let the technical term follow as a label (as "普通的文章 / 好的文章" in plain language, with "大型語言模型" as the term, backed by an icon).
- **Fewer words per slide is better.** A slide is glanced at; put detail in what you *say*, not on the slide.

**When the user's source material is too thin to be concrete, ask for specifics** — a real example, a number, a story — rather than filling the slide with abstract, professional-sounding filler. Good input is what lets the text stay concrete; inventing abstractions to cover a gap is exactly what produces hard-to-follow AI text.

---

## 8. Stable layout frame

Give every content slide the **same skeleton** so listeners always know where to look:
- Top: the chapter progress bar (rule 2), same place every slide.
- A large slide title in a consistent position.
- Consistent footer (e.g. logo) and page number location.

A stable frame lowers cognitive load — listeners spend attention on your content, not on re-learning each slide's layout. Vary the *content area* layout between slides (see below) but keep the frame fixed.

---

## 9. Avoid the AI-deck tells

These make a deck look auto-generated. Avoid them (several are also in the base pptx skill's Avoid list):

- **No decorative line under the title.** The single most recognizable AI-slide tell.
- **No decorative color bars / stripes spanning an edge** — no full-width header/footer color band, no vertical sidebar stripe, no thin accent stripe along a card edge. To set something apart use a subtle tint, a soft shadow, or an icon.
- **No leftover template/watermark cruft** — stray faded numbers, ghost placeholder text, meaningless decorative filler shapes. Clean them out.
- **No "title + three bullets" on every slide.** That form-filling rhythm is a giveaway; vary the layout — sometimes a diagram, sometimes a big number, sometimes a two-column compare, sometimes a single sentence.
- **No abstract adjective walls** — "高效、穩定、可擴展" with no example behind them. See rule 7.
- **No generic-blue-everything, no default gradient, no everything-centered.** Center titles only; left-align body text and lists.
- **No faded light arrows on a light background** — arrows and lines must be dark enough to read on white.
- **No hand-drawn/generated brand logos and no generic circles standing in for icons** (see rule 3).

---

## Working with the base pptx skill

1. Read `/mnt/skills/public/pptx/SKILL.md` for generation mechanics (pptxgenjs to build, LibreOffice+pdftoppm to render, the visual QA loop).
2. Apply this skill's rules 1–9 for every styling and wording decision.
3. For spotlight/progressive sequences (rules 4–5), generate the near-identical slides programmatically — build the diagram once as data, then emit one slide per reveal step, changing only opacity/color/highlight of the elements. This keeps positions pixel-identical so nothing jumps.
4. Run the base skill's **visual QA loop**, and additionally check: is exactly one thing spotlighted per reveal step? is the progress bar's current/faded state correct on each slide? did any faded element or light arrow become unreadable? are all brand icons real (not drawn/circled)?
5. If icons couldn't be fetched, list which ones and where you used neutral stand-ins.

See `references/checklist.md` for a quick pre-delivery checklist.
