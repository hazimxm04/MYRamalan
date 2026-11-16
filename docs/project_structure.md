# Project structure

_Moved out of the main [README](../README.md) to keep it short._

```
scripts/
  training_config.py     -- transitions, shared model settings, feature lists
  train_models.py         -- unified training pipeline (all 5 states), held-out score
  validation.py            -- scores saved models on real results (johor, neg_sembilan)
  evaluate_holdout.py      -- time-ordered test vs simple baselines, all states
  evaluate_pooled.py       -- pooled vs per-state experiment
  state_data.py            -- one shared data loader
  check_state_years.py     -- prints years / seat counts per state
  tune_hyperparams.py      -- time-ordered settings comparison (current settings kept: gaps were within noise)
  freeze_forecasts.py      -- save a dated, checksummed forecast for states that have not voted; --verify checks it
  score_forecast.py        -- score a frozen forecast against the real result, next to the simple rules
  download_data.py         -- election data, driven by the state registry
  download_ethnicity.py    -- voter-roll ethnicity files, driven by the registry
  add_ethnicity_features.py -- demographic-weighted narrative features

tests/                     -- all tests (leakage, data quality, training, predictor, API, agent), run via CI

backend/
  core/features.py                 -- the single feature definition (no leakage)
  core/states.py                   -- registry of states and elections
  core/models/state_predictor.py   -- live prediction, OOD fallback
  models/{state}/                   -- trained models, versioned history, metadata

sentiment/
  scrapers/news_scraper.py         -- RSS scraping, 5 sources, per-source lean disclosed
  scoring/sentiment_scorer.py      -- Groq scoring, national narrative themes
  scoring/groq_settings.py         -- which Groq model to call and how (token limit for reasoning models)

forecasts/<state>/<date>/          -- frozen forecasts: forecast.csv + manifest.json (never edited)

finetune/                          -- headline classifier: build_pool.py, score_pool_groq.py, cross_validate.py, ...
eval/                              -- hand-labelled headlines, pooled set, cross-validation results

economic/
  models/lstm_model.py              -- PyTorch LSTM, economic pressure score

chatbot/
  agent.py                          -- tool-choosing loop (max 6 model calls), trace, what_if_seat
  tools.py                          -- read-only data tools over the frozen and validated files
  planner.py                        -- optional plan-first step
  guardrails.py                     -- input, tool-result and output checks (code, not prompt)
  verifier.py                       -- every number in an answer must appear in a tool result
  docs_search.py                    -- search_docs tool over knowledge_base/docs/*.md (no figures)
  chain/, knowledge_base/builder.py -- older keyword chatbot (ChromaDB), kept for comparison

eval/chatbot_questions.csv          -- 27 fixed questions for the agent
scripts/eval_chatbot.py             -- runs them (old, router, agent, agent_verified, agent_planned)

mlops/
  model_versioning.py    -- held-out promotion gate, archiving
  alerts.py                -- event logging + optional email/Slack
  scheduler.py             -- APScheduler entry point
  dags/
    dag_sentiment.py       -- daily: scrape -> score -> update -> rebuild RAG
    dag_economic.py        -- weekly: fetch -> LSTM -> pressure score
    dag_drift.py            -- manual: post-election accuracy + drift check
    dag_training.py         -- wraps train_models.py for scheduled/manual runs

.github/workflows/
  ci.yml                    -- leakage + training tests on push / pull request
  scheduled-training.yml    -- monthly: refresh data, test, retrain, validate, open a PR

docs/
  leakage_fix.md            -- what leaked, how it was found, how the fix is tested
  classifier_comparison.md  -- DistilBERT vs TF-IDF vs Groq zero-shot on 352 labelled headlines
  multiclass_analysis.md    -- full investigation log: binary vs. 3-class target,
                                sentiment-feature leakage analysis, BN-PN coalition
                                finding, known issues and future work

data/
  raw/                       -- ballots, stats, ethnicity, news (parquet/CSV)
  processed/                 -- sentiment scores, narrative scores, economic pressure
```
