# mindlayer

[![PyPI version](https://img.shields.io/pypi/v/mindlayer.svg)](https://pypi.org/project/mindlayer/)
[![CI](https://github.com/Genious07/mindlayer/actions/workflows/ci.yml/badge.svg)](https://github.com/Genious07/mindlayer/actions/workflows/ci.yml)
[![License: MIT](https://img.shields.io/badge/License-MIT-green.svg)](LICENSE)
[![Python 3.9+](https://img.shields.io/badge/python-3.9+-blue.svg)](https://pypi.org/project/mindlayer/)

Open-source, model-agnostic memory layer for LLMs.

Give any LLM application persistent, structured memory with a single `pip install`. No API key required. No infrastructure. No vendor lock-in.

```python
import mindlayer

with mindlayer.MemCore() as mem:
    mem.add("My name is Alice. I prefer dark mode and I work in Python.")
    results = mem.search("programming preferences")
    for r in results:
        print(r.content)
```

---

## Why mindlayer?

Most LLM apps lose context between sessions. Vector databases are heavy to set up. Existing memory libraries tie you to a specific LLM or cloud service.

mindlayer is different:

| | mindlayer |
|---|---|
| Setup | `pip install mindlayer`, nothing else |
| Storage | SQLite, embedded, zero config |
| LLM | Bring your own, or use built-in Gemma |
| Offline | Fully offline capable |
| License | MIT |

---

## Installation

```bash
pip install mindlayer
```

With semantic vector search (downloads ~130MB embedding model on first use):

```bash
pip install "mindlayer[vector]"
```

With LLM-powered extraction (downloads Gemma ~800MB on first use):

```bash
pip install "mindlayer[llm]"
```

---

## Architecture

### 3-Layer Memory Model

Inspired by how human memory works: short-term facts get promoted to long-term knowledge based on how often they are accessed.

```mermaid
flowchart TD
    Input(["Raw Text"]) --> Ingestion["Ingestion"]
    Ingestion --> W["Working Memory\nshort-term facts"]
    W -->|"accessed 3+ times"| E["Episodic Memory\nmid-term facts"]
    E -->|"accessed 10+ times"| S["Semantic Memory\nlong-term knowledge"]

    Decay(["Decay"]) -.->|"prunes idle entries"| W
    Decay -.->|"prunes idle entries"| E

    style W fill:#dbeafe,stroke:#3b82f6
    style E fill:#fef9c3,stroke:#eab308
    style S fill:#dcfce7,stroke:#22c55e
    style Input fill:#f3f4f6,stroke:#6b7280
    style Decay fill:#fee2e2,stroke:#ef4444
```

### Data Flow

```mermaid
flowchart LR
    T(["Text Input"]) --> X["Extractor\nRules / Gemma"]
    X --> F["Discrete Facts"]
    F --> EM["Embedder\nFastEmbed / bge-small"]
    EM --> DB[("SQLite\n+ vec0")]

    Q(["Search Query"]) --> EM2["Embedder"]
    EM2 -->|"vector similarity"| DB
    DB --> R(["Ranked Results"])

    style DB fill:#f3f4f6,stroke:#6b7280
    style R fill:#dcfce7,stroke:#22c55e
    style T fill:#dbeafe,stroke:#3b82f6
    style Q fill:#dbeafe,stroke:#3b82f6
```

### 5 Core Primitives

```mermaid
flowchart LR
    A["Ingestion"] --> B["Conflict\nResolution"]
    B --> C["Consolidation"]
    C --> D["Retrieval"]
    D --> E["Decay"]

    style A fill:#dbeafe,stroke:#3b82f6
    style B fill:#fce7f3,stroke:#ec4899
    style C fill:#fef9c3,stroke:#eab308
    style D fill:#dcfce7,stroke:#22c55e
    style E fill:#fee2e2,stroke:#ef4444
```

---

## Usage

### Default: rule-based extractor, no LLM needed

```python
import mindlayer

mem = mindlayer.MemCore()
mem.add("I am a Python developer. I love open source.")
results = mem.search("developer")
```

### Semantic vector search: best recall

```python
# pip install "mindlayer[vector]"
mem = mindlayer.MemCore(use_vector=True)
mem.add("I prefer concise explanations and dislike verbose output.")
results = mem.search("communication style")
```

### Gemma LLM extractor: best extraction quality

```python
# pip install "mindlayer[llm]"
mem = mindlayer.MemCore(use_llm=True)
mem.add("Long conversation text with lots of context...")
```

### Bring your own extractor

```python
from mindlayer.extractors.base import BaseExtractor

class MyExtractor(BaseExtractor):
    def extract(self, text: str) -> list[str]:
        # call OpenAI, Anthropic, Ollama, anything
        return ["fact 1", "fact 2"]

mem = mindlayer.MemCore(extractor=MyExtractor())
```

### Bring your own storage backend

```python
from mindlayer.storage.base import BaseStorage

class PostgresStorage(BaseStorage):
    # implement the interface
    ...

mem = mindlayer.MemCore(storage=PostgresStorage())
```

### Memory maintenance

```python
mem.consolidate()  # promote memories across layers
mem.decay()        # decay and prune stale memories
```

---

## Roadmap

- [x] SQLite storage with vector search (sqlite-vec)
- [x] Rule-based extractor
- [x] Gemma LLM extractor (auto-download)
- [x] 3-layer memory model
- [ ] Async support
- [ ] PostgreSQL storage backend
- [ ] LLM-based conflict resolution
- [ ] REST API server mode
- [ ] JavaScript / TypeScript port

---

## Contributing

Contributions are welcome. Please open an issue before submitting large PRs.

```bash
git clone https://github.com/Genious07/mindlayer
cd mindlayer
pip install -e ".[dev,vector]"
pytest
```

---

## License

MIT
