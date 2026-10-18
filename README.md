# applied-llm-systems

Coursework for COSC 650: Applied LLM Systems at Maryville University (8-week Fall 2026 term). Each week is a small Jupyter notebook experiment: build a technique, measure it, then find where it breaks. The capstone combines them into a question-answering assistant that cites its sources from eight IETF RFCs.

## What's here

| Week | Experiment | Headline result |
|---|---|---|
| [1](01-tokenization/) | English vs. Spanish tokenization | Spanish costs 1.51× the tokens (GPT-4), 1.29× (GPT-4o) |
| [2](02-architecture-inference-sampling/) | Forward pass and sampling explorer | At temperature ≤ 0.01, sampling collapses to greedy |
| [3](03-prompt-engineering/) | Versioned prompts with a test suite | Exact-match went from 80% to 90% by v3 |
| [4](04-function-and-tool-use/) | Multi-tool assistant, guarded calculator | Clear guard errors let the model fix its own calls |
| [5](05-rag-and-retrieval/) | RAG over RFCs, chunking comparison | hit@3: 0.70 for paragraph chunks, 0.10 for small fixed chunks |
| [6](06-advanced-rag-systems/) | Query expansion, re-ranking, RAGAS | Re-ranking took MRR@3 from 0.55 to 0.80; query expansion made it worse |
| [7](07-fine-tuning/) | Zero-shot vs. dynamic few-shot | 98.4% vs. 97.7%, so fine-tuning wasn't justified |
| [8](08-final/) | **Capstone: RFC Spec Assistant** | Answer match went from 93% to 100% after one prompt fix ([details](08-final/README.md)) |

## Lessons learned

- **Measure before you believe.** Two plausible ideas, query expansion and dynamic few-shot, made results worse once measured.
- **Retrieval is usually the bottleneck.** Generation was faithful (0.88), but context recall was 0.47. Heading fragments were crowding out the answers in the top-k slots.
- **Error messages are part of the prompt.** A specific rejection message was enough for the model to correct its own tool call.
- **Prompt rules collide.** The capstone's failure came from two instructions in conflict ("refuse if it's not in the passages" and "use the calculator"), not from retrieval.
- **Use a different model as the judge, and still check it.** A judge scored a refusal 0.97 for relevance.

## Setup

Python 3.13 notebooks with tiktoken, transformers, sentence-transformers, FAISS, and RAGAS, calling Claude and Gemini. Put `ANTHROPIC_API_KEY` and `GEMINI_API_KEY` in a repo-root `.env` (gitignored). The tokenization and retrieval steps run without keys.
