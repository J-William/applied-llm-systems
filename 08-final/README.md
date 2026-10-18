# RFC Spec Assistant (Week 8 Capstone)

Answers developer questions about eight IETF RFCs (JSON, RFC 3339 timestamps, Base64, JWT, OAuth bearer tokens, cookies, WebSocket, and UUIDs). Answers cite their passages, numbers come from a calculator tool, and questions the corpus doesn't cover get a refusal. Everything is in [`capstone.ipynb`](capstone.ipynb).

## Results (15 held-out cases)

| Metric | v1 (held out) | v2 (after fix) |
|---|---|---|
| Tool selection (expected tools called) | 93% | 100% |
| Retrieval hit (needed facts reached the model) | 100% | 100% |
| Answer match (key value in answer) | 93% | 100% |
| Refusal on out-of-scope questions | 3/3 | 3/3 |
| LLM judge: correctness / grounding / citation (1–5) | 4.73 / 4.73 / 4.73 | 5.00 / 5.00 / 5.00 |

The 15 cases are 8 single-fact lookups, 4 that need a retrieved value plus arithmetic, and 3 out of scope. The judge is Gemini `gemini-3.1-flash-lite`, a different model family from the generator, and it scored three named dimensions. The v2 column includes the case used to design the fix, so v1 is the held-out number.

## Architecture

```
question ─► Claude + prompts/system_v2.txt ─┬─► search_rfcs(query) ─► FAISS top-20 (section chunks) ─► cross-encoder ─► top-3 [RFC n] passages
                                            └─► run_python(expr)   ─► 2-stage AST guard ─► number
            ◄──────────── tool results (logged per call) ────────────┘
answer ─► model-free metrics + Gemini judge (prompts/judge_v1.txt)
```

## Techniques, where they came from, and why

| Technique | Week | Why it's here |
|---|---|---|
| Section-aware chunking + FAISS retrieval | 5 | Prefixing each paragraph with its heading fixed Week 5's failure, where bare headings were retrieved as empty chunks, and it gives every passage a citation anchor. |
| Cross-encoder re-ranking (20 → 3) | 6 | The only Week 6 transform that improved ranking without regressing any query (MRR@3 0.48 → 0.80 here). PRF was dropped because it regressed. |
| Tool use: schemas, guarded calculator, validate/dispatch loop | 4 | The model routes between retrieval and arithmetic itself, replacing a brittle keyword router. The guard means numbers are computed, not recalled. |
| Versioned prompt files | 3 | The Part 3 fix is a new prompt file, `system_v2.txt`, measured against v1 on the same cases. |
| Throttled, cached judge calls; cross-family judge | 6, 7 | Re-runs cost nothing, and the generator never grades itself. |

## The failure we found

- **Symptom:** asked for the minimum per-domain cookie storage (50 cookies × 4096 bytes = 204,800), v1 searched three times and then refused, even though both limits were in its passages.
- **Cause:** the prompt, not retrieval. v1's rule "refuse if the passages do not contain the answer" collides with "use the calculator" when an answer must be *derived*. Over 5 runs, v1 refused once, stopped short of computing twice, and was correct twice.
- **Fix:** v2 rewords only that rule. It went 5/5 correct with the calculator, with no regressions on the other 14 cases, and out-of-scope refusals still held. The evidence is in the call logs and two ablations in Part 3.

## Run it

1. Put `ANTHROPIC_API_KEY` and `GEMINI_API_KEY` in the repo-root `.env`.
2. Run `capstone.ipynb` top to bottom from `08-final/` (dependencies: `sentence-transformers`, `faiss-cpu`, `anthropic`, `openai`, `python-dotenv`).

The corpus is read from `../06-advanced-rag-systems/rfcs/`. Generator runs are cached in `eval_runs.json` and judge calls in `gemini_cache.json`, so a re-run makes no new API calls.
