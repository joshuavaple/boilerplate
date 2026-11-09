For a mono-repo with frontend and backend folder, reference structure for backend agent:
```
backend/
├── src/my_agent/
│   ├── core/
│   │   ├── graph.py        # agent/graph construction (build_agent(llm, tools, checkpointer))
│   │   ├── state.py        # state schema (TypedDict / pydantic)
│   │   ├── ports.py        # Protocols: LLMClient, MemoryStore, Retriever
│   │   └── prompts/        # versioned prompt templates (as data, not f-strings in code)
│   ├── tools/              # pure, deterministic capabilities (mortgage math) — no LLM imports
│   ├── adapters/
│   │   ├── llm.py          # model client(s)
│   │   ├── memory.py       # checkpointer / Lakebase
│   │   └── tracing.py      # MLflow / observability hooks
│   ├── utilities/          # formatting, parsing
│   └── settings.py         # pydantic-settings, reads env
├── app/
│   ├── main.py             # app factory = composition root (wires adapters → core)
│   ├── routes.py           # /invoke, /stream (SSE), /health
│   └── schemas.py          # request/response models = frontend contract
├── tests/
│   ├── unit/               # tools, utilities, graph with fake LLM
│   └── integration/        # real adapters, gated by env/markers
├── evals/                  # LLM-quality evaluation datasets + scorers
├── scripts/                # notebooks, backfills, one-offs
├── config/
├── .env.example
└── pyproject.toml
```