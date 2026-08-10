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
specs/
└── labs/
    └── ntfy-sh/                       # the folder name IS the service name
        ├── provider.json              # Provider: UnitySVC Labs (`labs`)
        ├── offering.json              # Offering: capabilities, upstream config
        ├── listing.json               # Listing: documents, price, user access interface
        ├── service.json               # Identity sidecar (backend service_id)
        ├── connectivity.sh.j2         # Connectivity test (private)
        ├── code_example.sh.j2         # cURL publish example (public)
        ├── code_example.py.j2         # Python publish example (public)
        ├── user-guide.md              # Tutorial: setup, publish, subscribe (public)
        ├── tutorials/
        │   └── getting-started.md     # Step-by-step onboarding guide (public)
        └── images/                    # Screenshots used by the guides
```

The service is published as **`labs/ntfy-sh`** — `listing.name` is the folder's
path under `specs/`, and `offering.name` is the bare service name (`ntfy-sh`).

### Documents

Documents are defined in `listing.json` and referenced via relative `file_path`:

| Document | Category | Public | Description |
|----------|----------|--------|-------------|
| cURL code example | `code_example` | Yes | Publish a notification with cURL |
| Python code example | `code_example` | Yes | Publish a notification with Python |
| Getting started tutorial | `tutorial` | Yes | Enrollment, publishing, mobile app, WebSocket subscriptions |
| User guide | `tutorial` | Yes | Tutorial covering publishing, subscribing, and mobile app setup |
| Connectivity test | `connectivity_test` | No | Automated connectivity validation |

Code examples (`.sh.j2`, `.py.j2`) are Jinja2 templates that use environment variables `SERVICE_BASE_URL` and `UNITYSVC_API_KEY` at runtime.

## Setup

Load the committed secrets manifest, then supply your seller credentials:

```bash
set -a; . ./seller.secrets.txt; set +a     # .env.example is a symlink to it

export UNITYSVC_SELLER_API_KEY="<seller-key>"
export UNITYSVC_SELLER_API_URL="https://seller.staging.unitysvc.com/v1/"
export UNITYSVC_API_KEY="<svcpass-key>"    # used as a gateway customer
```

The CLI ships with [`unitysvc-sellers`](https://github.com/unitysvc/unitysvc-sellers).
Invoke it as `usvc_seller …` if it's on PATH, else
`uvx --from unitysvc-sellers usvc_seller …`.

## Commands

```bash
# Schema + cross-file validation (no network)
usvc_seller specs validate

# Canonical formatting; commit the result
usvc_seller specs format

# Upstream-side tests — renders the docs against ntfy directly
usvc_seller specs run-tests labs/ntfy-sh

# Push to staging (re-upload creates a revision)
usvc_seller specs upload labs/ntfy-sh

# Gateway-side tests — route + svcpass key + upstream
usvc_seller services run-tests labs/ntfy-sh --force
```

`specs run-tests` skips cases that previously passed; add `--force` to re-run them.

## Architecture

```
Publisher (cURL/Python/etc.)
    │
    ▼
UnitySVC Gateway  ──────►  ntfy.unitysvc.dev/{topic}
    (auth + routing)              │
                                  ├──► Mobile app (iOS/Android)
                                  ├──► WebSocket clients
                                  └──► JSON stream subscribers
```

- **Gateway** handles authentication (API key) and routes requests to the upstream ntfy instance
- **Upstream** is a private ntfy server — each enrollment gets a unique 6-character topic code, referenced in the specs as `{{ enrollment.code }}`
- The upstream host is the seller secret **`NTFY_SERVER_URL`**, declared in `seller.secrets.txt` and defaulting to `https://ntfy.unitysvc.dev`
- **Mobile apps** connect directly to the ntfy server (no API key needed for receiving)
