# PhishNet — AI-Powered Phishing Detection Platform

> Built by TeamSynapse · Innovation Fair 2026 · 6-person team

PhishNet is a multi-engine AI platform that detects phishing threats across emails, URLs, QR codes, images, deepfake videos, file attachments, and live calls in real time. It integrates with Gmail and Outlook via OAuth and runs across Web, Desktop (Electron), and Mobile (React Native).

---

## Demo & Project Site

[![Watch the demo](https://img.youtube.com/vi/YQt3aZkwShM/maxresdefault.jpg)](https://www.youtube.com/watch?v=YQt3aZkwShM)

- **Demo video:** [youtube.com/watch?v=YQt3aZkwShM](https://www.youtube.com/watch?v=YQt3aZkwShM)
- **Project site:** [ahmeddth.github.io/phishnet-info](https://ahmeddth.github.io/phishnet-info/)

---

## My Contribution — ML Inference Orchestration & Integration Layer

I designed and built the production **ML serving infrastructure** that connects the Flask API gateway to five independently developed AI/ML detection engines — spanning NLP (TF-IDF + ensemble classifiers), classical ML (RandomForest on URLs and files), deep learning (HuggingFace deepfake CNN + BERT intent), and OCR-driven inference pipelines.

This is ML systems / MLOps work: integrating heterogeneous models behind a single inference contract, handling model-artifact serialization quirks (joblib, pickle), and normalizing outputs from models with very different score conventions into a unified schema.

### What I built

**Unified Inference Contract — `ScanResult`** (`core/orchestrator.py`)
- Designed and owned the `ScanResult` dataclass — the single output schema (scan_type, is_phishing, score, verdict, details, timestamp) that every ML engine in the system conforms to
- Normalizes outputs from five independently trained models with incompatible native formats and score conventions (0–100 vs 0–1000, probability vs raw logits, multi-key scoring) into one downstream-friendly shape
- Single `run_scan(scan_type, payload)` entry point dispatches inputs to the correct inference engine
- Fail-closed error handling — defaults to "Suspicious" on engine failure to prevent silent security bypasses
- SQLite audit logging of every inference result for traceability and debugging

**Plugin Registry** (`core/registry.py`)
- Dict-based engine registration with runtime validation; engines self-register on import
- Lets the team add new ML models with zero changes to the orchestrator or registry — `register_engine` / `get_engine` / `list_scan_types` public API

**Six ML Inference Adapters** (`core/engines/`)

Each adapter wraps a teammate's trained model behind the shared inference contract, solving real production-ML integration problems:

| Adapter | Model | ML integration challenge solved |
|---|---|---|
| `text_engine.py` | sklearn TF-IDF + LR/NB ensemble | Dynamic module loading with CWD-swap to handle relative-path model artifacts at inference time |
| `url_engine.py` | RandomForest (36 URL features) | Re-injected pickle class references into `__main__` to resolve a joblib model-deserialization failure |
| `qr_engine.py` | OpenCV decode → URL classifier | 4-stage decode fallback (OpenCV → PIL with mode coercion → pyzbar) before handing the URL to the ML classifier |
| `image_engine.py` | pytesseract OCR → downstream classifier | Auto-discovery of Tesseract binary across install paths; multi-key score extraction across model output versions |
| `video_engine.py` | HuggingFace deepfake CNN + BERT (FastAPI) | HTTP client for remote ML inference with 120s timeout and a two-tier failure model separating infra failures from model-level scan failures |
| `file_engine.py` | RandomForest + VirusTotal + entropy | Lazy-loaded singleton pattern; normalized dual-scale risk scores (0–100 and 0–1000 conventions) into the unified output |

### Cross-Team ML Collaboration

- **Defined inference-output contracts with two ML engineers.** Worked with Bilal (deepfake video pipeline) on the FastAPI response schema returned by his model service, and with Jashan (text/NLP engine) on the scan-output format the orchestrator consumes — this is what made the unified `ScanResult` work end-to-end.
- **Debugged model-output integration issues during ML adapter development** — investigated cases where engine outputs didn't match what the adapter expected (score-vs-verdict mismatches, missing or renamed fields between engine versions), and reconciled them in the adapter layer.

**Storage Layer** (`core/storage/`)
- SQLite `scan_logs` table providing an inference audit trail — parameterized inserts, idempotent schema creation, JSON-safe details serialization

**Integration Scripts**
- `smoke_check.py` — verifies all five ML engines register and run inference correctly, with health-gated upstream checks
- `diagnostics.py` — verifies Python ML packages, Tesseract binary, and the remote video-inference service reachability

---

## Architecture

```
[Web · Electron · React Native]
              |
              v
[Flask API Gateway — OAuth, email fetch, routing]
              |
              v
   [core.orchestrator.run_scan()]      <- my layer
              |
              v
        [Engine Adapters x 5]          <- my layer
              |
              v
[ML Engines — text · url · qr · image · video · file]
              |
              v
        [SQLite audit log]             <- my layer
```

---

## Team & Roles

| Member | Role |
|---|---|
| Ahmed | Flask API gateway, web/Electron/React Native frontends, OAuth, server hosting |
| Noel (me) | Backend core — orchestration, engine registry, adapters, storage |
| Jashan | Text/NLP engine — TF-IDF + Logistic Regression + Naive Bayes ensemble |
| Abdullah | URL/QR classifier (RandomForest, 200k samples), live call detection (Recall.ai + Whisper) |
| Bilal | Video pipeline — MediaPipe face detection, HuggingFace deepfake CNN, BERT intent, Whisper ASR |
| Saeed | File/malware engine — entropy analysis, VirusTotal, RandomForest classifier |

---

## Tech Stack

**My layer:** Python 3.11 · dataclasses · sqlite3 · importlib · pathlib · requests

**Platform:** Flask · FastAPI · Uvicorn · Electron · React Native · Expo

**AI/ML:** scikit-learn · HuggingFace Transformers · PyTorch · faster-whisper · MediaPipe · OpenCV · pytesseract · EasyOCR · joblib · librosa

**External APIs:** Gmail API · Microsoft Graph · Google OAuth · Microsoft OAuth (MSAL) · Recall.ai · VirusTotal

**Infrastructure:** ngrok · cloudflared · SQLite · Python venv

---

## Source Code

Source is maintained in a private collaborative repository. Available on request — contact me at **noel.roby23@gmail.com**.
