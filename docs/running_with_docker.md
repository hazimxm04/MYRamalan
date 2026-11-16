# Run with Docker

_Moved out of the main [README](../README.md) to keep it short._

The dashboard (including the analyst agent page) can run in a container, so nothing needs installing except Docker.

```bash
docker build -t myramalan .
docker run --rm -p 8501:8501 -e GROQ_API_KEY=your_key_here myramalan
```

Then open http://localhost:8501. The first build takes a few minutes (it downloads the Python packages); later builds
reuse the cached layers and only repeat the steps whose files changed. Stop the container with Ctrl+C.

| What | How it is set up |
|---|---|
| Base image | `python:3.11-slim` |
| Packages | `requirements.txt` only. The heavy optional packages (torch, chromadb, sentence-transformers, yfinance, apscheduler) are not installed |
| XGBoost | Installed as `xgboost-cpu` (same 2.1.1 release, without the large GPU library); the app only predicts on CPU |
| User | Runs as a normal user (`appuser`), not root |
| Port and health | Serves on 8501; a health check calls Streamlit's `/_stcore/health` |
| Secrets | The Groq key is passed when the container starts (`-e GROQ_API_KEY=...`). It is never copied into the image: `.env` files are excluded by `.dockerignore` |
| Contents | The code, the trained models, the frozen forecasts and the processed data, as they are in the repo at build time. Rebuild the image after retraining or refreshing data |

What the container does **not** do:

- It runs the dashboard only. The FastAPI service (`uvicorn backend.app.main:app`) is not started in it.
- The chatbot page needs `GROQ_API_KEY`. Without it, the pages that do not call the model still work.
- The older ChromaDB chatbot and the LSTM economic score need the optional packages, so they are unavailable in the container.
- It is not a hosted demo. It runs on your machine.

Built and run locally; all pages load. In PowerShell, run the two commands exactly as written above (one line each).
