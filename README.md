# PhishGuard AI
### AI-Powered Phishing Website and Scam Email Detection

PhishGuard AI is an academic cybersecurity project that helps users assess suspicious website URLs and email messages. It combines machine-learning predictions with understandable rule-based checks to highlight potential warning signs.

> **Important:** This project is a prototype for learning and experimentation. Its results are estimates, not a guarantee that a website or email is safe or malicious.

[![Python](https://img.shields.io/badge/Python-3.x-blue?logo=python)](https://www.python.org/)
[![FastAPI](https://img.shields.io/badge/API-FastAPI-009688?logo=fastapi)](https://fastapi.tiangolo.com/)
[![Scikit-learn](https://img.shields.io/badge/ML-scikit--learn-orange?logo=scikitlearn)](https://scikit-learn.org/)
[![Deploy to Render](https://render.com/images/deploy-to-render-button.svg)](https://render.com/deploy?repo=https://github.com/anshuljangra23/ai-phishing-scam-detection)

---

## Contents

- [Deploy online](#deploy-online)
- [What it does](#what-it-does)
- [How it works](#how-it-works)
- [Technology stack](#technology-stack)
- [Project structure](#project-structure)
- [Getting started](#getting-started)
- [Using the API](#using-the-api)
- [Machine-learning models](#machine-learning-models)
- [Datasets and model files](#datasets-and-model-files)
- [Limitations](#limitations)
- [Future improvements](#future-improvements)
- [Disclaimer](#disclaimer)

## Deploy online

A Render Blueprint is included in `render.yaml`. To publish a live demo:

1. Click the **Deploy to Render** button above and sign in to Render.
2. Review the `phishguard-ai` web service configuration and authorize Render to access this public GitHub repository.
3. Click **Apply** or **Deploy Blueprint** when Render shows the proposed service.
4. Wait for the build and deployment to complete, then open the public `onrender.com` URL shown by Render.

The configuration uses the free web-service plan for demonstration. Free services can spin down after inactivity, so the first request after a quiet period may take around a minute to respond. Treat the site as an academic demo, not a production security service.

## What it does

### Website and URL analysis
- Accepts a website URL or domain for analysis.
- Normalizes the URL and removes selected tracking parameters before prediction.
- Uses a trained text-based machine-learning model to estimate phishing likelihood.
- Checks URL characteristics such as IP-address hosts, missing HTTPS, unusually long URLs, suspicious symbols, many subdomains, and unusual ports.
- Returns a classification, estimated probabilities, structural risk points, and reasons for the result.

### Email analysis
- Checks email subject and body for common scam signals, such as urgency, requests for passwords or OTPs, payment pressure, and suspicious links.
- Uses a separate trained text-classification model to estimate whether an email may be a scam.
- Provides API endpoints for rule-based analysis, ML-based analysis, and a combined response that compares the two detectors.

### Results that are easier to understand
The website scanner presents the verdict, phishing score, structural risk points, detected warning signs, URL details, and practical safety advice.

## How it works

For a website URL, the main flow is:

1. **Submit a URL** through the web interface.
2. **Validate and normalize** the input.
3. **Run the ML model** to estimate phishing and legitimate probabilities.
4. **Run structural checks** to identify URL warning signs.
5. **Combine the signals** using the configured thresholds.
6. **Display a report** with the classification and reasons.

Email analysis follows a similar idea, but uses message-text rules and a separate email ML model.

## Technology stack

| Area | Technologies |
|---|---|
| Language | Python |
| Data processing | Pandas, NumPy |
| Machine learning | Scikit-learn, TF-IDF, Logistic Regression |
| Additional ML dependency | XGBoost |
| API server | FastAPI, Uvicorn |
| Frontend | HTML, CSS, JavaScript |
| Model storage | Joblib |

## Project structure

```text
ai-phishing-scam-detection/
├── backend/
│   └── main.py                     # FastAPI application and endpoints
├── email_detection/
│   ├── email_detector.py           # Rule-based email analysis
│   ├── email_ml_detector.py        # Trained email model inference
│   ├── train_email_model.py        # Train and evaluate the email model
│   ├── predict_email_ml.py         # Command-line email model test
│   └── models/
│       └── email_text_model.joblib # Saved email classifier
├── frontend/
│   ├── index.html                  # Website scanner page
│   ├── style.css                   # Interface styling
│   └── script.js                   # Calls the URL scanning API
├── results/
│   ├── raw_url_phishing_model.pkl  # Saved URL classifier
│   └── website_feature_names.pkl   # Saved feature-name artifact
├── website_detection/
│   ├── final_detector.py           # Main website detection pipeline
│   ├── risk_engine.py              # URL structural risk checks
│   ├── url_features.py             # URL feature extraction utilities
│   ├── train_url_model.py          # Train and evaluate URL model
│   ├── train_improved_model.py     # Experimental model training
│   ├── train_normalized_model.py   # Experimental normalized-URL model
│   └── evaluate_unseen_domains.py  # Evaluation on held-out hostnames
├── requirements.txt
├── .gitignore
└── README.md
```

Some directories also contain diagnostic scripts and README placeholders. The browser-extension directory currently does not contain an implemented extension.

## Getting started

### Requirements

- Python 3.10 or later is a reasonable starting point. The project does not currently specify a Python version, so compatibility with your environment may vary.
- Git
- pip
- The saved model files included in the repository

### 1. Clone the repository

```bash
git clone https://github.com/anshuljangra23/ai-phishing-scam-detection.git
cd ai-phishing-scam-detection
```

### 2. Create and activate a virtual environment

**Windows (PowerShell):**

```powershell
python -m venv .venv
.\.venv\Scripts\Activate.ps1
```

If PowerShell blocks activation, you can use Command Prompt instead:

```bat
.venv\Scripts\activate.bat
```

**macOS / Linux:**

```bash
python3 -m venv .venv
source .venv/bin/activate
```

### 3. Install dependencies

```bash
python -m pip install --upgrade pip
pip install -r requirements.txt
```

### 4. Start the application

From the repository root, run:

```bash
uvicorn backend.main:app --reload
```

When the server starts successfully, open:

- **Web interface:** http://127.0.0.1:8000/
- **Interactive API documentation:** http://127.0.0.1:8000/docs

Keep the terminal running while you use the app. If startup fails, check the error message, dependency compatibility, and whether the saved model files are present at the expected paths.

## Using the API

FastAPI exposes these endpoints:

| Method | Endpoint | Purpose |
|---|---|---|
| `GET` | `/` | Serves the web interface |
| `POST` | `/check-url` | Analyzes a website URL |
| `POST` | `/check-email-ml` | Runs email ML classification |
| `POST` | `/check-email-combined` | Returns rule-based and ML email results together |

### Example: scan a URL

Send a `POST` request to `http://127.0.0.1:8000/check-url` with JSON:

```json
{
  "url": "https://example.com"
}
```

The response includes the submitted URL, hostname, normalized URL, prediction, phishing and legitimate probabilities, structural risk points, and reasons.

### Example: analyze an email

Send a `POST` request to `http://127.0.0.1:8000/check-email-combined` with JSON:

```json
{
  "sender_email": "sender@example.com",
  "subject": "Please verify your account",
  "body": "Review this message carefully before opening any links."
}
```

The response contains separate rule-based and machine-learning results, plus an indicator of whether their classifications agree. The example is for demonstrating the request format only; it does not imply a particular result.

You can also explore request schemas and try endpoints through the interactive documentation at `/docs`.

## Machine-learning models

### URL model
The main URL model is loaded from `results/raw_url_phishing_model.pkl`. The training script `website_detection/train_url_model.py` builds a character n-gram TF-IDF and Logistic Regression pipeline. The configured label mapping in the training code is:

- `0` = phishing
- `1` = legitimate

The production detector reads the class labels from the saved model rather than assuming their order.

### Email model
The email classifier is loaded from `email_detection/models/email_text_model.joblib`. Its training script uses TF-IDF text features and Logistic Regression. The configured label mapping is:

- `0` = safe
- `1` = phishing/scam

### Rule-based checks
The URL risk engine assigns points to configured URL characteristics. The rule-based email detector looks for common scam indicators in the sender address, subject, body, and links. These checks are useful signals, but they can produce false positives and false negatives.

## Datasets and model files

The repository includes saved model artifacts, but the datasets referenced by the training scripts are not included in the repository because data files are excluded by `.gitignore`.

To retrain the models, obtain the appropriate datasets separately and place them at the paths expected by the relevant training scripts. For example, the URL training scripts expect `data/PhiUSIIL_Phishing_URL_Dataset.csv`, and the email training script expects `email_detection/data/email_dataset.csv`.

Follow each dataset's licence and usage conditions. Do not assume that running a training script will work without first supplying the required data. Some diagnostic scripts reference other files or model names, so check their paths before running them.

## Limitations

- A low phishing score does **not** prove that a URL or email is safe.
- A high score is a warning, not independent proof of malicious intent.
- The URL checks are based on configured patterns and model predictions, not a comprehensive live security audit.
- The rule-based email detector checks message content and link structure; it does not verify the real identity of the sender.
- Model probabilities depend on the training data and may not reflect performance on new or previously unseen attacks.
- The browser extension and a dedicated email-scanning web interface are not currently implemented in the checked-in frontend.
- The project has evaluation scripts, but this README does not claim a verified accuracy percentage.

## Future improvements

Possible next steps for the project include:

- Add a dedicated email-scanning interface.
- Implement and document the browser extension.
- Add repeatable tests and continuous integration.
- Document dataset sources, licences, model evaluation results, and known failure cases.
- Evaluate performance on previously unseen domains and recent phishing examples.
- Improve deployment configuration, logging, and input validation before any public deployment.

These are possible enhancements, not claims that the features already exist.

## Disclaimer

PhishGuard AI is an educational project intended for cybersecurity learning and experimentation. Use it as one source of information alongside independent verification and established security tools. Do not rely on it alone to make decisions involving passwords, payments, personal information, or sensitive systems.

---

**Repository:** [anshuljangra23/ai-phishing-scam-detection](https://github.com/anshuljangra23/ai-phishing-scam-detection)
