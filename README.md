# UnitySVC ntfy

Service data for [ntfy.sh](https://ntfy.sh) push notifications on the [UnitySVC](https://unitysvc.com) platform.

## Overview

This repo contains the provider, offering, and listing data for the **ntfy Push Notifications** service. Each enrollment gets a unique private topic — publish messages via simple HTTP POST requests and receive them instantly on mobile (iOS/Android), desktop, or via streaming API.

**Key features:**

- HTTP POST/PUT publishing (no client library required)
- Real-time subscriptions via WebSocket and JSON stream
- Mobile apps for iOS and Android
- Message priorities (1–5), emoji tags, click actions, and attachments
- Scheduled/delayed delivery
- Free — no signup or cost to use

## Data Structure

```
data/
└── ntfy-sh/
    ├── provider.json                  # Provider: ntfy.sh
    ├── docs/
    │   ├── connectivity.sh.j2         # Connectivity test (private)
    │   ├── code_example.sh.j2         # cURL publish example (public)
    │   ├── code_example.py.j2         # Python publish example (public)
    │   └── user-guide.md              # Tutorial: setup, publish, subscribe (public)
    └── services/
        └── ntfy-notifications/
            ├── offering.json          # Offering: capabilities, upstream config
            └── listing-default.json   # Listing: documents, user access interface
```

### Documents

Documents are defined in `listing-default.json` and referenced via relative `file_path`:

| Document | Category | Public | Description |
|----------|----------|--------|-------------|
| cURL code example | `code_example` | Yes | Publish a notification with cURL |
| Python code example | `code_example` | Yes | Publish a notification with Python |
| User guide | `tutorial` | Yes | Tutorial covering publishing, subscribing, and mobile app setup |
| Connectivity test | `connectivity_test` | No | Automated connectivity validation |

Code examples (`.sh.j2`, `.py.j2`) are Jinja2 templates that use environment variables `SERVICE_BASE_URL` and `UNITYSVC_API_KEY` at runtime.

## Setup

```bash
python -m venv .venv
source .venv/bin/activate
pip install -r requirements.txt
```

Requires [`unitysvc-services`](https://pypi.org/project/unitysvc-services/) (installed via requirements.txt).

## Commands

```bash
# Validate data files against schemas
usvc data validate

# Format/prettify data files
usvc data format

# List local services
usvc data list

# Run code examples locally (requires upstream credentials)
usvc data run-tests

# Upload to UnitySVC (requires seller credentials)
export SERVICE_BASE_URL="https://backend.unitysvc.com/v1"
export UNITYSVC_API_KEY="<api-key>"
usvc data upload

# Submit for review
usvc services submit <service_id>
```

## Architecture

```
Publisher (cURL/Python/etc.)
    │
    ▼
UnitySVC Gateway  ──────►  ntfy.svcpass.com/{topic}
    (auth + routing)              │
                                  ├──► Mobile app (iOS/Android)
                                  ├──► WebSocket clients
                                  └──► JSON stream subscribers
```

- **Gateway** handles authentication (API key) and routes requests to the upstream ntfy instance
- **Upstream** (`ntfy.svcpass.com`) is a private ntfy server — each enrollment gets a unique 6-character topic code generated via `{{ enrollment_code(6) }}`
- **Mobile apps** connect directly to the ntfy server (no API key needed for receiving)
