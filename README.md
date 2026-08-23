# TalentBliss Job Ingestion

Validated, resumable ingestion pipeline that prepares job feeds and publishes them to the TalentBliss API in safe, idempotent batches.

## Overview

This repository is the ingestion stage between the job-scraping workflow and the TalentBliss platform. It downloads the latest feed from Google Drive or accepts a local JSON file, validates and deduplicates records, and publishes the complete dataset through the TalentBliss import API.

The pipeline is intentionally fail-safe: incomplete uploads cannot finalize and therefore cannot remove the existing production job set.

## Pipeline

```text
job sources
   ↓
job-scraping-pipeline
   ↓
validated JSON feed / Google Drive
   ↓
talentbliss-job-ingestion
   ↓
TalentBliss import API
   ↓
PostgreSQL
```

## Key features

- Google Drive feed discovery and integrity checks
- Local-file dry-run support
- Validation and duplicate-URL removal
- Stable run IDs for resumable/idempotent imports
- Producer-consumer batch publishing
- Configurable byte and record limits per request
- Independent retry handling for transient failures
- Finalization only after all expected batches succeed
- Unit tests for download, validation, and import behavior

## Tech stack

- Python
- Google Drive API
- Requests
- GitHub Actions
- TalentBliss HTTP import API

## Repository structure

```text
.
├── main.py                 # CLI entry point and pipeline orchestration
├── filedownload.py         # Google Drive discovery/download logic
├── oracle_import.py        # Validation, batching, and API publishing
├── tests/                  # Unit tests
├── .github/workflows/      # Scheduled/manual automation
├── requirements.txt
└── README.md
```

## Local setup

```bash
python -m venv .venv
source .venv/bin/activate
pip install -r requirements.txt
python -m unittest discover -s tests -v
python main.py --help
```

Validate a local feed without publishing it:

```bash
JOB_DATA_FILE=/path/to/job_summary.json DRY_RUN=true python main.py
```

## Configuration

For Google Drive ingestion, configure the relevant Drive credentials/IDs. Publishing requires a TalentBliss API URL and import token. Oracle SSH-tunnel variables are only needed when the staging API is reached through the private server path.

Never commit secrets. Keep production import enablement disabled until a manual run has been verified successfully.

## Safety model

The loader aborts before finalization when the feed cannot be downloaded or validated, the API is unhealthy, credentials are missing, a batch fails permanently, response counts are inconsistent, or finalization does not acknowledge the complete job count.

Jobs absent from a new feed are deleted only by the server-side finalize operation after all expected batches have completed successfully.

## Verification

```bash
python -m pip check
python -m unittest discover -s tests -v
python -m compileall -q .
python main.py --help
```
