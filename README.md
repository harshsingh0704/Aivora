# Aivora — Intelligent Voice Receptionist & Business Automation Platform

**A conversational AI receptionist for service businesses, combining phone calls, retrieval-augmented answers, appointment scheduling, and CRM workflows.**

> **Project provenance:** This README is a rewritten presentation of the supplied Cleo.ai codebase, originally associated with the CleoVoice project. Retain original copyright notices and comply with the repository's license before redistributing. Do not present upstream code as independently authored.

## Overview

Aivora is designed for HVAC, plumbing, electrical, and other home-service businesses. It receives inbound calls, transcribes speech, uses an LLM to decide what to say or which tool to call, and synthesizes spoken responses. It also provides a dashboard for reviewing calls and configuring business information.

### Core capabilities

- **AI phone receptionist:** Twilio call handling, Deepgram speech recognition, Claude-based reasoning, and ElevenLabs speech output.
- **Knowledge-base Q&A (RAG):** Upload text/Markdown documents, embed them with Voyage AI, and retrieve relevant passages from Qdrant.
- **Appointment workflows:** Google Calendar availability checks, booking, and conflict prevention logic.
- **Customer context:** Phone-number-based customer lookup and conversation history.
- **CRM and field-service integration:** Modules for HubSpot, Jobber, and Housecall Pro; external credentials and service configuration are required.
- **Follow-up automation:** SMS booking confirmations/reminders and post-call email summaries.
- **Admin experience:** Call transcripts, business configuration, analytics, and knowledge-document management.

> Capabilities above describe the supplied project implementation and documentation. Actual end-to-end behavior depends on external API credentials, account configuration, and testing. Live-call latency and production reliability have not been independently benchmarked here.

## System architecture

```text
Inbound caller
     |
     v
Twilio Voice / webhooks
     |
     v
FastAPI application <----> PostgreSQL (customers, calls, appointments)
     |
     +--> Deepgram (speech-to-text)
     |
     +--> Claude (reasoning and structured tool use)
     |      |--> Qdrant + Voyage AI (knowledge retrieval)
     |      |--> Google Calendar (availability / bookings)
     |      +--> Customer and business services
     |
     +--> ElevenLabs (text-to-speech) --> Twilio --> caller
     |
     +--> HubSpot / Jobber / Housecall Pro integrations
     +--> Twilio SMS / email follow-ups
     +--> Admin dashboard and analytics
```

## Technology stack

| Layer | Technologies |
| --- | --- |
| Backend | Python, FastAPI |
| LLM | Anthropic Claude |
| Speech recognition | Deepgram |
| Speech generation | ElevenLabs |
| Telephony / SMS | Twilio |
| RAG | Voyage AI embeddings, Qdrant |
| Database | PostgreSQL |
| Scheduling | Google Calendar, APScheduler |
| Business integrations | HubSpot, Jobber, Housecall Pro |
| Dashboard | HTML, Tailwind CSS, Chart.js |
| Infrastructure | Docker, Docker Compose; production configuration included |

## Repository layout

```text
backend/
  app/
    api/          # Call webhooks, streaming, documents, customers, analytics, auth
    core/         # Configuration, database, vector store, runtime state
    models/       # Database models
    services/     # Conversation, calendar, speech, documents, CRM integrations
    main.py       # FastAPI application setup
  static/         # Admin dashboard
  requirements.txt
  Dockerfile
docker-compose.yml
.env.example
apex_home_services.txt  # Example business knowledge
```

## Getting started

### Prerequisites

- Docker and Docker Compose
- API credentials for Twilio, Deepgram, Anthropic, Voyage AI, and ElevenLabs
- Optional credentials for Google Calendar, CRM systems, and email
- A publicly reachable HTTPS URL for telephony webhooks (for example, an ngrok tunnel during development)

### 1. Configure

```bash
cp .env.example .env
```

Fill in the required keys and database/vector-store settings. **Never commit `.env` or live credentials.**

### 2. Start services

```bash
docker compose up --build
```

### 3. Add knowledge

```bash
curl -X POST http://localhost:8000/documents/upload \
  -F "file=@apex_home_services.txt"
```

### 4. Connect phone calls

```bash
ngrok http 8000
```

Configure the Twilio voice webhook to send HTTP POST requests to:

```text
https://YOUR-PUBLIC-HOST/call/incoming
```

Set the status callback to `/call/status`. Check the installed Twilio configuration and webhook security before exposing the service publicly.

### 5. Inspect the dashboard

Visit `http://localhost:8000/dashboard/` to review calls, documents, and business configuration, subject to your configured authentication.

## Selected API endpoints

| Method | Endpoint | Purpose |
| --- | --- | --- |
| GET | `/health` | Health status |
| POST | `/call/incoming` | Twilio inbound webhook |
| POST | `/call/status` | Call status callback |
| POST | `/documents/upload` | Add a knowledge document |
| GET | `/documents/search?q=...` | Semantic document search |
| GET | `/calls/` | Call records |
| GET | `/analytics/overview` | Call analytics |
| GET | `/business/config` | Business configuration |

Refer to the FastAPI-generated `/docs` endpoint for the routes exposed by your running version.

## Engineering concepts demonstrated

- Coordinating speech recognition, LLM inference, and voice synthesis across external services.
- Using structured tool calls for actions rather than relying on free-form model responses.
- Grounding answers in business-specific documents with retrieval-augmented generation.
- Separating API routes, service logic, data models, and third-party integrations.
- Persisting operational data and exposing it through an admin interface.
- Managing external credentials and running dependent services with containers.

## Limitations and next improvements

- **Live validation:** Test real phone calls, webhook event handling, and booking/CRM workflows with configured provider accounts.
- **Latency:** Add measured time-to-first-audio and p50/p95 dashboards; avoid claiming sub-second performance without measurement.
- **Interruptions:** Add and test explicit barge-in cancellation and telephony playback clearing if not already supported in the running call path.
- **Multi-tenancy:** Strengthen per-tenant data isolation, authorization, and integration-secret storage for a shared SaaS deployment.
- **Reliability:** Add automated integration tests, idempotency for external actions, retries, and observability.
- **Security:** Review webhook signature validation, credential handling, PII retention, and consent requirements for call recordings/transcripts.


