# Grounded Persona Extraction — Methodology Reference

> Shared methodology doc for POS-builder skills that build persona cards of named public figures from web sources. First consumer: `pos-advisors` (#68). Reference-only: skills link here instead of duplicating content in `SKILL.md`.
>
> Source: Codex research brief `task-mo7cozns-zp0v26` (2026-04-20, 11m 29s, gpt-5.3-codex). Full prompt and raw brief preserved in that job log. Numbers below are benchmark-grounded with source URLs; derivations are flagged.

## TL;DR

- Use an **evidence-linked fixed-schema persona card** built from primary sources, then add only light retrieval at runtime. This dominates fidelity, grounding, and cost. Retrieval-only personas lose — RoleBench SPE 19.1 vs 38.1 for structured Context-Instruct.
- **Do not fine-tune / LoRA** for public-figure advisors. Benchmark gap is usually too small to justify the author-time cost and maintenance burden of a structured-card pipeline.
- **Default search provider: Brave Search API** for discovery; add **Parallel Search** only for hard multi-hop slots. Tavily scores best on SimpleQA but Brave has the best cost/index/control balance for bilingual author research.

## Method comparison

| method | fidelity benchmark | hallucination rate | runtime tokens | author-time cost | maintenance |
|---|---:|---|---:|---|---|
| Structured extraction into fixed schema | RoleBench SPE **38.1** with Context-Instruct; retrieval-only variant **19.1** ([RoleLLM](https://aclanthology.org/2024.findings-acl.878/), Table 6) | No direct rate; much stronger role-specific knowledge than retrieval-only | 1.5k-3k (derived) | Medium | Medium |
| RAG-grounded at invocation | RoleLLaMA reaug SPE **36.7** vs **38.1** for system-instruction customization ([RoleLLM](https://aclanthology.org/2024.findings-acl.878/), Table 8) | Lower unsupported claims with clean sources; noisy retrieval harms fidelity | 2.5k-6k (derived) | Low | Medium |
| Few-shot exemplar prompting / persona cards | GPT-4 eval win rate **63.3** few-shot dialogue vs **29.8** few-shot prompt, **9.3** zero-shot ([RoleLLM](https://aclanthology.org/2024.findings-acl.878/), Table 7) | Style is strong but factual drift remains unless grounded | 3k-8k (derived) | Low-Medium | Low |
| Fine-tuning / LoRA adapters | RoleLLaMA-7B human win rate **56.1** on unseen roles ([RoleLLM](https://aclanthology.org/2024.findings-acl.878/), Tables 4-5) | Locks style but errors become harder to audit | 0.4k-1k (derived) | **High** | **High** |
| Chain-of-density summarization | Unverified on current role-play benchmarks | Good compression; omission risk is real | 1k-2k (derived) | Medium | Low-Medium |
| Principles-first / constitutional persona | Caution: GPT-4o **5.81%** in-character on RPEval ([summary](https://app.argminai.com/arxiv-dashboard/papers/2505.13157v1)) | Safer but can suppress authentic traits | 1k-2k (derived) | Low | Low |
| **Hybrid evidence card + light retrieval (recommended)** | No single end-to-end benchmark; derived from strongest published lifts above | Best practical risk profile when paired with claim-level citation gates | 1.8k-3.2k (derived) | Medium | Medium |

## Provider comparison

| provider | accuracy (source) | RU coverage | cost/1k | latency | notes |
|---|---|---|---:|---|---|
| **Brave Search API** | **76.05%** SimpleQA in [tavily-search-evals](https://github.com/tavily-ai/tavily-search-evals); [docs](https://brave.com/search/api/) | Medium (derived); `country` + `search_lang`; Brave itself recommends pairing with Yandex/Baidu for localized recall | $5 | "Low latency"; 50 QPS | Best cost/index/control; independent index; good snippets and filters |
| Parallel Search | No public head-to-head benchmark | Medium (derived, unverified) | $5 | 1-3s ([pricing](https://docs.parallel.ai/resources/pricing)) | Very good excerpt compression and objective-based search; best as second-stage |
| Exa | **71.24%** SimpleQA ([tavily-search-evals](https://github.com/tavily-ai/tavily-search-evals)); vendor 81% WebWalker ([comparison](https://exa.ai/versus/tavily)) | Medium (derived); language filtering shipped Nov 2025 ([docs](https://docs.exa.ai/changelog/language-filtering-default)) | $7 search / $12 deep ([pricing](https://exa.ai/pricing/api)) | 180ms-1s configurable | Strong semantic and people search; mixed practitioner feedback on exact lookups |
| Tavily | **93.3%** SimpleQA, 83.02% doc relevance (vendor-run public repo) | Low-medium (derived); no public RU benchmark | $8 PAYG ([credits](https://docs.tavily.com/documentation/api-credits)) | 180ms p50 vendor claim | Strong factual QA; no people/company search |
| Serper | **82.15%** SimpleQA (Google via Serper) | High (Google proxy, derived) | $1 starter ([pricing](https://serper.dev/)) | Real-time | Cheapest Google wrapper; Google-dependent |
| Google CSE | No CSE-specific score | High (derived) | $5 ([docs](https://developers.google.com/custom-search/v1/overview?hl=en_US)) | Unverified | Good global-recall fallback; setup clunky |
| Kagi API | No public benchmark | Medium-high (derived) | $25 | ~213ms example | Premium; too expensive for default batch authoring |

## Hallucination controls — ranked

1. **Author-time structured grounding beats naive retrieval stuffing.** RoleBench: Context-Instruct lifts role-specific knowledge from 19.1 (retrieval augmentation) to **38.1 SPE**. Much larger gain than simply adding runtime retrieval. [RoleLLM](https://aclanthology.org/2024.findings-acl.878/).
2. **Few-shot dialogue exemplars beat prompt-only persona prompting.** RoleGPT few-shot dialogue engineering **63.3%** GPT-4 win rate vs 29.8% prompt and 9.3% zero-shot. [RoleLLM](https://aclanthology.org/2024.findings-acl.878/).
3. **Claim-citation validation gates outperform LLM-only checking.** CoVeGAT **96.4%** citation-alignment vs top zero-shot LLMs at 82.5%. Every retained claim keeps URL + supporting excerpt before synthesis. [CoVeGAT](https://aclanthology.org/2025.r2lm-1.1/).
4. **System-instruction role customization > retrieval-demo stuffing for smaller models.** RoleLLaMA 38.1 SPE with system instructions vs 36.7 retrieval; RoleGLM 34.1 vs 25.3. [RoleLLM](https://aclanthology.org/2024.findings-acl.878/).
5. **Self-consistency is secondary, not primary.** Useful for disputed claims and error detection, but 2025 persona papers emphasize knowledge-boundary and evidence-quality first. Apply only after grounding and citation gates. [EMNLP 2025](https://aclanthology.org/2025.emnlp-main.1689/), [SemEval 2025](https://aclanthology.org/2025.semeval-1.38/).

## Recommended pipeline (pos-advisors reference)

```python
slots = [
  "bio_timeline", "core_beliefs", "decision_rules", "canonical_stories",
  "quote_bank", "style_markers", "known_reversals", "domains_of_competence",
  "blind_spots", "would_not_say",
]

for slot in slots:
    queries = decompose(slot, langs=["en", "ru"],
                        variants=["official", "interview", "lecture", "book", "primary quote"])
    hits = brave.search(queries, country="auto", search_lang=query_lang)
    if recall_is_thin or slot_is_multihop:
        hits += parallel.search(objective=slot)
    passages = keep_top_primary_or_reputable(hits, max_passages=8)
    claims = llm.extract_json(
        passages,
        schema=Claim(slot, claim, source_url, excerpt, source_type, date),
    )
    validated = [c for c in claims if c.source_type == "primary" or corroborated(c, n=2)]

persona_card = llm.summarize_to_schema(
    validated,
    schema=PersonaCard(with_negative_exemplars=True, with_citation_ids=True),
    technique="chain_of_density_at_author_time_only",
)

answer = llm.respond(
    system=compact(persona_card),
    evidence=retrieve_top_k(validated, k=2),
    rule="cite source ids inline or say unknown",
)
```

**Derived token budget per persona:** author-time ~18k-25k tokens of LLM use; runtime ~1.8k-3.2k input + 300-700 output. Far cheaper than LoRA, materially safer than pure few-shot or pure RAG.

**Bilingual note:** issue `queries` in both `en` and `ru`. Russian-source recall is under-benchmarked; pair Brave with one local engine (Yandex) when a slot comes back thin on RU.

## Open questions / risks

- Published benchmarks are mostly fictional / entertainment characters. Transfer to real named public figures is partial.
- Provider accuracy numbers are noisy because the strongest public head-to-head benchmarks are vendor-authored, even when code is public.
- Russian-source coverage is under-benchmarked. Validate on a small bilingual holdout before scaling.
- "Would-not-say" negative exemplars are good practice, but no clean 2025-2026 ablation found proving they beat the controls above.
- Practitioner feedback is workload-specific — treat provider choice as an A/B-tested ops decision, not dogma.
- Re-evaluate in 3-6 months against RPEval, RoleKE-Bench, Act-LLM, and any public bilingual search benchmark that includes Russian claim extraction.

## Sources

- https://aclanthology.org/2024.findings-acl.878/ — RoleLLM (primary benchmark source)
- https://aclanthology.org/2024.acl-long.102/
- https://aclanthology.org/2024.acl-long.638/
- https://app.argminai.com/arxiv-dashboard/papers/2505.13157v1 — RPEval summary
- https://aclanthology.org/2025.r2lm-1.1/ — CoVeGAT
- https://aclanthology.org/2025.emnlp-main.1689/
- https://aclanthology.org/2025.semeval-1.38/
- https://github.com/tavily-ai/tavily-search-evals
- https://brave.com/search/api/
- https://docs.parallel.ai/resources/pricing
- https://exa.ai/pricing/api
- https://exa.ai/versus/tavily
- https://docs.exa.ai/changelog/language-filtering-default
- https://docs.tavily.com/documentation/api-credits
- https://serper.dev/
- https://developers.google.com/custom-search/v1/overview?hl=en_US
- https://help.kagi.com/kagi/api/search.html
- https://venturebeat.com/ai/how-ai-digital-minds-startup-delphi-stopped-drowning-in-user-data-and-scaled-up-with-pinecone
- https://www.reddit.com/r/Rag/comments/1gr8jnr/which_search_api_should_i_use_between_tavilycom/
