# RUN

## Local 5 Minute Check

```bash
cd /Users/Zhuanz/Desktop/面试题目MVP/01-mining-rag-pipeline
python3 -m venv .venv
source .venv/bin/activate
pip install -r requirements.txt
make demo
```

Start the Web console:

```bash
make serve
open http://localhost:8001
```

## Docker

```bash
cd /Users/Zhuanz/Desktop/面试题目MVP/01-mining-rag-pipeline
docker compose up --build
```

Open `http://localhost:8001` after startup.

## Optional Live Model

```bash
cp .env.example .env
export MODEL_API_KEY=your_key
export MODEL_BASE_URL=https://apihub.agnes-ai.com/v1
export MODEL_NAME=agnes-2.0-flash
```

The key is read from the environment only. Do not write real keys into project files, Docker Compose, README, QA reports or release zips.

## Query Smoke Test

```bash
curl -s http://localhost:8001/query \
  -H 'content-type: application/json' \
  -d '{"question":"近 7 天澳洲锂出口价格有何变化?","top_k":5,"days":7}' | jq
```

Expected shape: Chinese `answer`, numbered `[1]` citations, citation rows with `命中段` / `概括` on the Web console, and a folded backend JSON block for audit.

## QA And Package

```bash
make test
make qa
make package
```

`make qa` writes `QA_REPORT.md` and `qa/reports/*.json`. `make package` writes `/Users/Zhuanz/Desktop/01-mining-rag-pipeline-tool.zip`.
