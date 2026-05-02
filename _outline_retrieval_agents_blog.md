# Blog Outline — Retrieval Agents, RAG, and Context Engineering

**Working titles** (pick or remix):
1. *Beyond Prompts: A Decision Stack for GenAI, RAG, and Context Engineering*
2. *Three Decisions Before You Build a Retrieval Agent*
3. *From "Should We Use GenAI?" to a Working Retrieval Agent*

**Audience:** mixed technical readers — engineers who write the code and architects/leads who make the call.
**Voice:** match the existing `20260501_LLM_models_size_by_usecases.md` post — prose-first, no bullet-spam, code blocks where they pay off, occasional first-person field notes in blockquotes.
**Estimated length:** ~1,800–2,400 words.
**Series fit:** sits *upstream* of the existing LLMOps post. This one frames *what* and *why*; that one is *how to operate it*. End with a forward link.

---

## Narrative arc (one-line summary)

Three decisions, in order: **Should you use GenAI? → Which model? → How do you architect the context?** The first two are short, decisive sections anchored by your two diagrams. The third is the meat — prompt engineering's ceiling, RAG, context engineering, and token economics — drawn from the Databricks lesson.

---

## Section sketch

### Hook (≈150 words)
Open with a concrete tension: teams reach for GenAI reflexively, then hit a wall — not because the model is bad, but because they skipped the decisions *before* the model. Two failure shapes to name fast: (a) using GenAI where SQL or a classifier would have been deterministic, and (b) using GenAI without the proprietary context it actually needs. Promise the reader a three-step decision stack. End the intro with: *"The first two decisions take a paragraph each. The third is most of this post."*

---

### 1. Decision One — Is GenAI the right tool at all? (≈250 words)
**This is where Image 1 (flowchart) plugs in.** Caption it as *"A decision tree for choosing between classical ML, GenAI, and RAG+GenAI."*

Walk the diamonds in prose, not as a bulleted re-statement of the diagram:
- *Can output quality be measured clearly?* — if no, you don't have a use case yet, you have a vibe. Refine before building.
- *Deterministic accuracy required?* — taxes, payroll, regulatory checks. Don't reach for an LLM. Use rules, SQL, or a calibrated classifier.
- *Language understanding/generation needed?* — this is the GenAI gate. Without it, you're back to classical ML.
- *Synthesis or open-ended output?* — the difference between "classify this ticket" (small model, possibly classical) and "draft a reply that addresses the customer's concern."
- *Proprietary enterprise knowledge?* — this is the RAG gate. The model can write English, but not about *your* contracts, *your* runbooks, *your* SKUs.

Close the section with: the right answer is often *not* GenAI. Saying so up front earns trust for the rest of the post.

---

### 2. Decision Two — Which model? (≈250 words)
**This is where Image 2 (Model Selection Considerations) plugs in.** Caption it as *"Model size vs. job: a rough fit guide."*

Frame the six considerations from the diagram (complexity, quality, cost, latency, privacy, deployment) as a *budget*, not a checklist. Then walk the four buckets in one paragraph each:

- **Small (~7B–13B):** extraction, formatting, simple Q&A — anywhere the rubric is narrow and the answer is short. Cheap and fast, run on-prem or at the edge.
- **Medium (~30B–70B):** the workhorse for support agents and content generation. The honest sweet spot for most production retrieval agents.
- **Large (~70B+):** complex reasoning, ambiguous prompts, multi-step planning. Worth the cost when the cost of a wrong answer is high.
- **Frontier:** state-of-the-art when quality dominates cost. Reserve for high-stakes paths, not the entire product.

Add a one-line caveat the diagram can't carry: *parameter count is a proxy, not a capability.* A 2026-class small model often beats a 2024-class large one. Re-benchmark when you upgrade.

---

### 3. The wall prompt engineering hits (≈300 words)
Now you transition into the source-article material. Open with: *the first two decisions are framing — the rest of the post is what happens after you've decided "yes, GenAI" and "yes, with retrieval."*

Cover from the article's Section A:
- **A1 — what prompt engineering actually is:** instruction-layer optimization, few-shot, persona, CoT.
- **A2 — reasoning vs. non-reasoning models:** non-reasoning models need explicit "think step-by-step"; reasoning models do it themselves and can actually get *worse* if you over-prompt them. This is a common mistake worth calling out.
- **A3 — the three failure modes of prompt-only systems:** knowledge cutoff, hallucination, ambiguity. Use the article's example prompts almost verbatim — *"Who won the 2025 Nobel Prize in Physics?"*, the avocado-and-blood-sugar fabrication, and the "secure a lakehouse" ambiguity. They land because they're concrete.

End with a one-sentence pivot: *no amount of prompt cleverness fixes a model that simply doesn't know.*

---

### 4. RAG, but said properly (≈250 words)
Pull from Section B of the article.

- **B1 — definition:** retrieval, augmentation, generation. One paragraph, no need to belabor it.
- **Insert the article's RAG vs. Retrieval Agent callout** as a blockquote — it's a distinction most posts blur. RAG is the pattern; a retrieval agent is the implementation that does routing and orchestration on top.
- **B2 — context rot:** the failure that early RAG implementations all share — stuff the prompt with everything you retrieve and watch the model degrade. Name the two failure shapes: **context poisoning** (irrelevant chunks confuse the model) and **lost in the middle** (long contexts get skimmed).

Close: *the question stops being "did we retrieve?" and starts being "what did we put in front of the model, and in what order?"*

---

### 5. Context engineering — the discipline that comes after RAG (≈400 words)
The longest section. Use Section C from the article.

- **C1 — the context window as an engineered environment.** System instructions, conversation history, retrieved data, user constraints — all variables you control. Prompt engineering shapes one of these; context engineering shapes all four.
- **C2 — system prompts as behavioral programs.** Walk through the three components in prose: role definition, *negative* constraints (often more powerful than positive ones), output formatting. Note that "do not mention competitor products" is more reliable than "stay on topic."
- **C3 — grounding and metadata filtering.** This is the section to slow down on. Two ideas:
  - *Grounding instructions* — explicit phrases like *"answer using only the provided context"* meaningfully reduce hallucination, but only when paired with retrieval that actually contains the answer.
  - *Metadata filtering* — filter the corpus *before* embedding similarity, not after. "Only documents tagged `year=2024` and `region=EU`" is cheaper, faster, and more secure than retrieving everything and hoping the model picks correctly.
- **C4 — multi-turn state.** Three strategies, one sentence each: summarization, moving window, selective persistence (e.g., always keep the user ID, drop everything else past N turns).

**Optional first-person blockquote** to match the existing post's voice — something like: *"On one project, switching from 'rerank everything we retrieve' to 'filter by document owner first, rerank a small set' cut latency by 40% and improved faithfulness scores. The cheap optimization was the right one."*

---

### 6. The token economy (≈250 words)
Pull from Section D.

- **D1 — context windows as working memory.** 8k/32k/128k aren't just upper bounds; they're a *budget* split three ways: instructions, retrieved docs, conversation history. Bigger isn't free — latency rises, "lost in the middle" kicks in, cost climbs linearly with input.
- **D2 — two optimizations that actually move the needle:**
  - **Just-in-time retrieval** — give the agent a *tool* it can call to fetch a specific section, instead of front-loading the prompt with everything it might need. This is the architectural shift from "retrieval as preprocessing" to "retrieval as a tool the agent uses."
  - **Reranking** — retrieve top-50 with cheap vector search, rerank with a small dedicated model, inject only the top 3–5. This is one of the highest leverage moves in production RAG and worth its own short subsection.

Tie this section back to **Decision Two**: the model you picked sets your budget. A medium model with a 32k window and a tight reranker often outperforms a frontier model getting fed 128k of slop.

---

### 7. Putting the stack together (≈200 words)
Bring the three decisions back into one frame:

1. **Should you use GenAI?** — the flowchart from §1.
2. **Which model?** — the cost/quality/latency budget from §2.
3. **How do you architect the context?** — prompt engineering, RAG, context engineering, token economics from §3–6.

Make the point that these aren't a one-shot waterfall — they're a loop. A bad eval result in production should send you back up the stack: is it a context problem (re-rank, re-chunk), a model problem (size up), or did you misclassify the use case in the first place (the answer was actually SQL)?

---

### 8. What's next (≈100 words)
A short forward-link section. Two natural follow-ups, both worth flagging:

- **Document parsing and chunking** — the source article is the lead-in to that lesson; this is where to take it next. Chunk size, overlap, layout-aware vs. naive splitting, and how chunking strategy interacts with the embedding model.
- **Operating a retrieval agent in production** — link to your existing `LLM Models size by usecases` / LLMOps post for the deployment, eval, and observability counterpart.

End with one line: *"A working retrieval agent is the easy part. Keeping it working is the next post."*

---

## Image placement summary

| Image | Section | Caption suggestion |
|---|---|---|
| Flowchart (GenAI / RAG decision tree) | §1 — Decision One | *"A decision tree for choosing between classical ML, GenAI, and RAG+GenAI."* |
| Model Selection Considerations | §2 — Decision Two | *"Model size vs. job: a rough fit guide."* |

Both images render natively at the post's main column width — no resizing needed.

---

## Front-matter draft (Astro-compatible, matches your existing format)

```yaml
---
title: "Beyond Prompts: A Decision Stack for GenAI, RAG, and Context Engineering"
description: "Three decisions, in order: should you use GenAI? Which model? How do you architect the context? A practitioner's stack for retrieval agents."
pubDate: "May 02 2026"
heroImage: "../../assets/<add-hero>.png"
draft: true
---
```

Suggested filename: `src/content/blog/20260502_retrieval_agents_decision_stack.md`

---

## Open questions before I draft the full post

- Do you want me to include a Databricks-specific lens (Mosaic AI Vector Search, Unity Catalog) the way the source article does, or keep it vendor-neutral?
- Any field-notes blockquotes you want me to weave in (real projects, team experiences) — the existing post uses these to great effect.
- Hero image: reuse `20250430_banner.png`, or create something new?
