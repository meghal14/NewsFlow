# NewsFlow 

An AI-powered global news pipeline that automatically discovers trending topics from around the world, scrapes article content, fetches images, rewrites everything into polished journalism, and renders a beautiful newspaper-style HTML digest — with zero human input per run.

---

## How it works

NewsFlow runs as a 5-stage automated pipeline:

| Stage | What it does | LLM? |
|---|---|---|
| 1 — Trend discovery | Fetches trending topics from Google Trends across 8 countries, picks the 5 most newsworthy | 1 call |
| 2 — News scraping | Searches Google News and scrapes full article text for each topic | No |
| 3 — Image fetching | Fetches up to 3 images per topic from Unsplash and Serper Images | No |
| 4 — Content editing | Rewrites raw scraped content into a 150-200 word AP-style article | 1 call per topic |
| 5 — HTML render | Builds a complete newspaper-style HTML page from all content | No |

**Total LLM calls per run: 6** (1 for topic selection + 1 per article × 5 topics)

---

## Output

Each run produces `output/digest.html` — a self-contained newspaper page covering 5 global stories with images, articles, and a topic navigation bar. Open it in any browser or serve it via the built-in FastAPI server.

---

## Tech stack

- **Pipeline** — [CrewAI Flow](https://docs.crewai.com)
- **LLM** — [Mistral API](https://console.mistral.ai) via [LiteLLM](https://litellm.ai)
- **Search** — [Serper API](https://serper.dev)
- **Images** — [Unsplash API](https://unsplash.com/developers)
- **Trends** — Google Trends RSS (no key needed)
- **Server** — [FastAPI](https://fastapi.tiangolo.com) + [Uvicorn](https://www.uvicorn.org)
- **Scheduler** — [APScheduler](https://apscheduler.readthedocs.io)

---

## Project structure

```
newsapp/
├── server.py                          # FastAPI server
├── requirements.txt                   # Python dependencies
├── output/
│   └── digest.html                    # Generated digest (created on each run)
└── newsflow/
    └── src/newsflow/
        ├── main.py                    # Pipeline — all 5 stages
        └── tools/
            └── custom_tool.py         # GoogleTrendsTool, UnsplashSearchTool, HTMLRenderTool
```

---

## Setup

### 1. Clone the repo

```bash
git clone https://github.com/yourusername/newsflow.git
cd newsflow
```

### 2. Install dependencies

```bash
pip install -r requirements.txt
```

### 3. Set up environment variables

Create a `.env` file in the `newsapp/` root:

```env
MISTRAL_API_KEY=your_mistral_key
SERPER_API_KEY=your_serper_key
UNSPLASH_ACCESS_KEY=your_unsplash_key
```

| Key | Where to get it | Required? |
|---|---|---|
| `MISTRAL_API_KEY` | [console.mistral.ai](https://console.mistral.ai) | Yes |
| `SERPER_API_KEY` | [serper.dev](https://serper.dev) | Yes |
| `UNSPLASH_ACCESS_KEY` | [unsplash.com/developers](https://unsplash.com/developers) | No (Serper images used as fallback) |

---

## Running locally

### Run the pipeline once

```bash
python newsflow/src/newsflow/main.py
```

Takes ~4-5 minutes. Output saved to `output/digest.html`.

### Start the web server

```bash
uvicorn server:app --reload --host 0.0.0.0 --port 8000
```

Then open [http://localhost:8000](http://localhost:8000) in your browser.

### Trigger a pipeline run via the API

```bash
curl -X POST http://localhost:8000/run
```

Or open [http://localhost:8000/docs](http://localhost:8000/docs) and use the built-in Swagger UI.

---

## API endpoints

| Method | Endpoint | Description |
|---|---|---|
| `GET` | `/` | Serves the latest `digest.html` |
| `POST` | `/run` | Triggers a new pipeline run |

---

## Scheduling

The pipeline runs automatically every day at **7:00 AM** via APScheduler — no manual trigger needed as long as the server is running. To change the time, edit `server.py`:

```python
scheduler.add_job(run_pipeline_job, "cron", hour=7, minute=0)
```

---

## Free tier limits

| Service | Free tier |
|---|---|
| Mistral API | Rate limited — pipeline uses only 6 calls per run with 30s gaps |
| Serper | 2,500 searches/month (~250 runs/month) |
| Unsplash | 50 requests/hour |

---

## Configuration

To change the number of topics per run, edit `main.py`:

```python
self.state.topics = topics[:5]  # change 5 to any number
```

Note: more topics = more LLM calls = longer run time. 5 is recommended for free tier.

---

