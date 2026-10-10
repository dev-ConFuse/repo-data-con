# Repository Data Connector (repo-data-con)

**Port**: 8081 · **Role**: Repository source integration and intelligent file
routing (repository flavour of the data connector)

> This is the canonical README for the service; **all documentation lives in
> [`docs/`](./)** (this folder). See [index.md](index.md) for the docs index.

## Overview

`repo-data-con` is the entry point for **repository** sources in the ConFuse
platform. It handles:

- **Source Management**: Connect Git repositories (GitHub, GitLab, Bitbucket)
- **File Type Detection**: Analyze files to determine if they're code or documents
- **Intelligent Routing**: Route files to the appropriate processors via Kafka
- **Event Production**: Publish file ingestion events (Confluent Cloud ready)
- **Webhook Handling**: Receive triggers from external systems

> The document flavour lives in `doc-data-con` (Notion/Figma/URLs + HTTP handoff).

## Architecture

```
┌─────────────────────────────────────────────────────────────┐
│                 Repository Data Connector (:8081)            │
├─────────────────────────────────────────────────────────────┤
│  Source Management │ File Classification │ Events           │
└────────────────────┴─────────────────────┴──────────────────┘
                             │
                             ▼
                   ┌──────────────────┐
                   │  Kafka /         │
                   │  Confluent Cloud │
                   └──────────────────┘
```

## Supported Sources

- **Code repositories**: GitHub, GitLab, Bitbucket
- **File types**: Python, JavaScript, TypeScript, Java, Go, Rust, C/C++;
  Markdown/Text documents; YAML/JSON/TOML configuration

## Key API surface

| Endpoint | Method | Description |
|----------|--------|-------------|
| `/health`, `/api/v1/health` | GET | Health checks |
| `/api/repositories` | GET/POST | List / connect repositories |
| `/api/repositories/{id}` | GET/DELETE | Repository details / disconnect |
| `/api/repositories/{id}/sync` (also `/clone`) | POST | Trigger sync/streaming |
| `/api/repositories/{id}/clone-status` | GET | Sync status |
| `/api/repositories/{id}/branches` | GET | Branches for a repository |
| `/api/repositories/oauth/check` | POST | Validate repo access via OAuth |
| `/api/repositories/fetch-branches` | POST | Fetch branches & file types |
| `/api/sources*` | — | Source CRUD + `/sync` |

Full schemas: [api-reference.md](api-reference.md).

## Event Flow

```
Repository Sync → File Discovery → Classification → Event Production
```

Events produced: `code.ingested`, `docs.ingested`, `source.sync.completed`,
`source.sync.failed` (failed messages go to `KAFKA_DLQ_TOPIC`).

## Configuration

```bash
# Downstream processing via Kafka
KAFKA_BOOTSTRAP_SERVERS=localhost:9092
KAFKA_DLQ_TOPIC=confuse-dlq

# Confluent Cloud (optional)
CONFLUENT_BOOTSTRAP_SERVERS=pkc-7prvp.centralindia.azure.confluent.cloud:9092
CONFLUENT_API_KEY=your_api_key
CONFLUENT_API_SECRET=your_api_secret

# Service
PORT=8081
ENVIRONMENT=development
```

## How to run the microservice

```bash
# Install dependencies
pip install -e .

# Set environment
export ENVIRONMENT=development
export KAFKA_BOOTSTRAP_SERVERS=localhost:9092

# Run service
python -m app.main
```

## Testing

Tests for this service live in the central test module (this service contains
no tests):

```bash
cd ../ConFuse-test-module
pip install -r requirements.txt
pytest data_connector -v
```

## Deployment

```bash
# Docker
docker build -t confuse/repo-data-con .
docker run -p 8081:8081 confuse/repo-data-con
```

## Documentation

| Document | Contents |
|----------|----------|
| [index.md](index.md) | Overview & docs index |
| [api-reference.md](api-reference.md) | Full endpoint reference |
| [architecture.md](architecture.md) | Internal architecture |
| [connectors.md](connectors.md) | Provider connector details |
| [setup.md](setup.md) | Local development setup |
