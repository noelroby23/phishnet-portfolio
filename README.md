# PhishNet — AI-Powered Phishing Detection Platform

> Built by TeamSynapse · Innovation Fair 2026 · 6-person team

PhishNet is a multi-engine AI platform that detects phishing threats across emails, URLs, QR codes, images, deepfake videos, file attachments, and live calls in real time. It integrates with Gmail and Outlook via OAuth and runs across Web, Desktop (Electron), and Mobile (React Native).

---

## My Contribution — Backend Core & Integration Layer

I designed and built the backend core: the Python orchestration layer that connects the Flask API gateway to five independently developed AI detection engines across the team.

### What I built

**Engine Orchestrator** (`core/orchestrator.py`)
- Single `run_scan(scan_type, payload)` entry point for all scan types
- Fail-closed error handling — defaults to "Suspicious" on any engine failure, preventing silent security bypasses
- Normalises heterogeneous engine outputs into a unified typed `ScanResult` dataclass (scan_type, is_phishing, score, verdict, details, timestamp)
- SQLite audit logging of every scan result

**Plugin Registry** (`core/registry.py`)
- Dict-based engine registration system with runtime validation
- Engines self-register on import — zero changes needed to the orchestrator or registry to add a new engine
- `register_engine` / `get_engine` / `list_scan_types` public API

**Six Engine Adapters** (`core/engines/`)

Each adapter integrates a teammate's ML engine behind the shared contract, solving real interoperability problems:

| Adapter | Engine | Integration challenge solved |
|---|---|---|
| `text_engine.py` | sklearn TF-IDF + LR/NB ensemble | Dynamic module loading with CWD-swap to handle relative-path model files |
| `url_engine.py` | RandomForest (36 URL features) | Re-injected pickle class references into `__main__` to resolve joblib deserialisation failure |
| `qr_engine.py` | OpenCV + URL classifier | 4-stage decode fallback (OpenCV → PIL with mode coercion → pyzbar) for exotic image formats |
| `image_engine.py` | pytesseract OCR pipeline | Auto-discovery of Tesseract binary across Windows install paths; multi-key score extraction |
| `video_engine.py` | HuggingFace deepfake + BERT (FastAPI) | HTTP client with 120s timeout and two-tier failure model (infra failure vs scan failure) |
| `file_engine.py` | RandomForest + VirusTotal + entropy | Lazy-loaded singleton; normalised dual-scale risk scores (0–100 and 0–1000 conventions) |

**Storage Layer** (`core/storage/`)
- SQLite `scan_logs` table with parameterised inserts, idempotent schema creation, and JSON-safe details serialisation

**Integration Scripts**
- `smoke_check.py` — verifies all 5 engines register, runs text/image/video scans with health-gated upstream checks
- `diagnostics.py` — checks Python packages, Tesseract binary, and video service reachability

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

---

## Screenshots / Demo

*Coming soon — dashboard screenshots and a scan walkthrough GIF will be added.*
