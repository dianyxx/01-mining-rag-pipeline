# Mining RAG Pipeline Console

Industry-facing RAG question-answering tool for mining news, critical-minerals policy and price evidence. The app ingests public/fixture sources, deduplicates and chunks documents, builds a local retrieval index, then answers Chinese industry questions with numbered evidence citations.

This repository is project 01 from the mining interview MVP set. It is intentionally standalone: no shared backend, no shared database and no dependency on the other interview projects.

## What It Does

- Collects three evidence types: mining news, policy/regulatory notes and commodity price fixtures.
- Falls back to `data/fixtures` when public sources are unavailable, with `source_mode` and `warnings` exposed in every response.
- Parses common mining questions by commodity, region, intent and time window.
- Retrieves and filters evidence by source priority, so price questions prefer price evidence and policy questions prefer policy evidence.
- Uses a model, when configured, to synthesize the final Chinese answer from retrieved evidence and to write citation-specific Chinese summaries.
- Falls back to deterministic Chinese output when no model key is provided, so the demo still runs offline.
- Provides a FastAPI API, Web console, pytest suite, QA suite and Docker Compose entrypoint.

## Answer Format

The business-facing answer is designed for auditability:

```text
结论：... [1][2]
关键依据：... [1]
风险/限制：... [3]
下一步建议：...
```

Each citation is rendered as:

```text
1 - 原文标题
命中段：English source sentence or paragraph
概括：中文概括，由模型基于该条命中段和问题生成
链接：https://...
```

Debug terms such as keyword hits and relevance scores are kept in the folded Raw JSON output only.

## Quick Start

```bash
python3 -m venv .venv
source .venv/bin/activate
pip install -r requirements.txt
make demo
make serve
```

Open `http://localhost:8001`.

Docker:

```bash
docker compose up --build
```

## Model Configuration

The app reads model settings from environment variables only. Do not commit real keys.

```bash
export MODEL_API_KEY=your_key
export MODEL_BASE_URL=https://apihub.agnes-ai.com/v1
export MODEL_NAME=agnes-2.0-flash
```

When `MODEL_API_KEY` is present, `/query` uses the configured OpenAI-compatible chat endpoint to generate the Chinese answer and citation summaries. Without a key, the deterministic fallback remains runnable for interviews and CI.

## API Example

```bash
curl -s http://localhost:8001/query \
  -H 'content-type: application/json' \
  -d '{"question":"近 7 天澳洲锂出口价格有何变化?","top_k":5,"days":7}' | jq
```

Stable response fields include `status`, `warnings`, `source_mode`, `elapsed_ms`, `data_quality`, `intent`, `answer`, `answer_points`, `citations` and `hits`.

## QA And Packaging

```bash
make test
make qa
make package
```

`make qa` runs industry generalization cases for lithium, copper, nickel, zinc, iron ore, rare earth and cobalt across common regions. `make package` creates `/Users/Zhuanz/Desktop/01-mining-rag-pipeline-tool.zip`.

## Boundaries

This is a complete MVP, not a production market-data platform. It does not bypass login walls or paid data sources. Evidence gaps are returned as `limited` or `abstain` with explicit warnings instead of fabricated answers.
