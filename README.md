# Beast Observatory

> Real-time observability and monitoring platform for Beast Mode multi-agent ecosystem

[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](https://opensource.org/licenses/MIT)
[![Powered by Directus](https://img.shields.io/badge/Powered%20by-Directus-6644FF)](https://directus.io/)
[![Redis](https://img.shields.io/badge/Redis-Pub%2FSub-DC382D)](https://redis.io/)
[![Prometheus](https://img.shields.io/badge/Prometheus-Metrics-E6522C)](https://prometheus.io/)

**Public URL**: https://observatory.nkllon.com  
**Infrastructure**: Dockerized stack on vonnegut.local  
**Purpose**: Centralized real-time monitoring for CI/CD, agents, and quality metrics

## Features

- 🔴 **Real-Time Updates**: WebSocket-powered live dashboard
- 📊 **CI/CD Visibility**: Track builds, tests, deployments across all projects
- 🤖 **Agent Monitoring**: Beast Mode agent activity and collaboration patterns
- 📈 **Quality Metrics**: Code quality trends and quality gate results
- 🔔 **Alerting**: Automatic notifications for failures and critical events
- 🗄️ **Historical Data**: Query and analyze past events
- 🔒 **Secure**: API token authentication and Cloudflare protection
- 🚀 **Scalable**: Redis pub/sub + Prometheus for high-volume events

## Quick Start

### Prerequisites

- Docker and Docker Compose
- Cloudflare tunnel configured
- vonnegut.local server access

### Installation

```bash
# Clone repository
git clone https://github.com/nkllon/beast-observatory.git
cd beast-observatory

# Configure environment
cp .env.example .env
# Edit .env with your tokens and secrets

# Start services
docker-compose up -d

# Check health
curl http://localhost:8000/health
curl http://localhost:8055  # Directus admin
curl http://localhost:9090  # Prometheus
```

### First Event

```bash
# Send test event
curl -X POST http://localhost:8000/api/events \
  -H "Authorization: Bearer YOUR_TOKEN" \
  -H "Content-Type: application/json" \
  -d '{
    "event_type": "test_event",
    "source": "manual_test",
    "data": {"message": "Hello Observatory!"}
  }'
```

## Architecture

```
Internet → Cloudflare → vonnegut.local
                             ↓
                    ┌────────────────────┐
                    │  Nginx (80/443)    │
                    └────────┬───────────┘
                             ↓
        ┌────────────────────┼────────────────────┐
        ↓                    ↓                    ↓
   FastAPI API          Directus CMS        Prometheus
   (port 8000)          (port 8055)         (port 9090)
        ↓                    ↓                    ↓
        └────────────────────┼────────────────────┘
                             ↓
                         Redis (6379)
```

## Services

### 1. FastAPI Event Ingestion API
- **Port**: 8000
- **Purpose**: Receive and process events
- **Endpoints**:
  - `POST /api/events` - Submit single event
  - `POST /api/events/batch` - Submit multiple events
  - `GET /health` - Health check
  - `GET /metrics` - Prometheus metrics

### 2. Directus CMS
- **Port**: 8055
- **Purpose**: Data persistence + admin UI
- **Features**:
  - Event storage in collections
  - Dashboard configuration
  - User management
  - REST + GraphQL APIs

### 3. Redis
- **Port**: 6379
- **Purpose**: Pub/sub + caching
- **Configuration**:
  - Max memory: 512MB
  - Policy: allkeys-lru
  - Persistence: AOF enabled

### 4. Prometheus
- **Port**: 9090
- **Purpose**: Metrics collection
- **Retention**: 90 days
- **Scrape Targets**:
  - FastAPI /metrics
  - OpenFlow agents
  - Quality system

### 5. Nginx
- **Ports**: 80, 443
- **Purpose**: Reverse proxy
- **Routes**:
  - `/api/*` → FastAPI
  - `/admin/*` → Directus
  - `/metrics` → Prometheus

## Event Types

### CI/CD Events
- `ci_build_started` - Build initiated
- `ci_build_completed` - Build finished
- `ci_tests_completed` - Test results
- `quality_check_completed` - Quality gate results
- `deployment_started` - Deployment initiated
- `deployment_completed` - Deployment finished

### Agent Events
- `agent_registered` - Agent came online
- `agent_request_started` - Request received
- `agent_request_completed` - Request processed
- `agent_collaboration` - Multi-agent interaction
- `agent_error` - Agent failure

### Quality Events
- `quality_score_updated` - Quality metrics changed
- `quality_gate_passed` - Gate succeeded
- `quality_gate_failed` - Gate failed
- `coverage_updated` - Test coverage changed

## Configuration

### Environment Variables

```bash
# Directus
DIRECTUS_KEY=<random-key>
DIRECTUS_SECRET=<random-secret>
DIRECTUS_ADMIN_EMAIL=admin@nkllon.com
DIRECTUS_ADMIN_PASSWORD=<secure-password>
DIRECTUS_TOKEN=<api-token>

# Observatory API
OBSERVATORY_SECRET=<api-secret>
OBSERVATORY_TOKEN=<client-token>

# Redis (optional custom config)
REDIS_PASSWORD=<optional-password>

# Cloudflare
CLOUDFLARE_TUNNEL_TOKEN=<tunnel-token>
```

### Cloudflare Tunnel Setup

```bash
# Install cloudflared
brew install cloudflare/cloudflare/cloudflared

# Login and create tunnel
cloudflared tunnel login
cloudflared tunnel create observatory

# Configure tunnel (see cloudflare/tunnel-config.yml)
cloudflared tunnel route dns observatory observatory.nkllon.com

# Run tunnel
cloudflared tunnel run observatory
```

## Usage

### From OpenFlow Playground

```bash
# Set token in GitHub Secrets
gh secret set OBSERVATORY_TOKEN --body "your-token-here"

# Events automatically sent from CI workflows
# (see integration in .github/workflows/*.yml)
```

### From Python Code

```python
from observability.observatory_client import ObservatoryClient

client = ObservatoryClient(
    api_url="https://observatory.nkllon.com",
    api_token=os.getenv("OBSERVATORY_TOKEN")
)

await client.send_event("custom_event", {
    "key": "value",
    "metric": 123
})
```

### View Dashboard

```
https://observatory.nkllon.com
```

## Development

```bash
# Run locally
docker-compose up

# View logs
docker-compose logs -f api

# Rebuild API
docker-compose build api
docker-compose up -d api

# Access Directus admin
open http://localhost:8055
```

## Monitoring

### Health Checks

```bash
# API health
curl http://localhost:8000/health

# Directus health
curl http://localhost:8055/server/health

# Redis health
redis-cli ping

# Prometheus health
curl http://localhost:9090/-/healthy
```

### Metrics

```bash
# API metrics
curl http://localhost:8000/metrics

# Prometheus query
curl 'http://localhost:9090/api/v1/query?query=up'
```

## Deployment (vonnegut.local)

```bash
# SSH to vonnegut.local
ssh vonnegut.local

# Clone repository
git clone https://github.com/nkllon/beast-observatory.git
cd beast-observatory

# Configure
cp .env.example .env
vim .env

# Start services
docker-compose up -d

# Verify
docker-compose ps
curl http://localhost:8000/health
```

## Documentation

- [Requirements](.kiro/specs/beast-observatory/requirements.md) - System requirements
- [API Documentation](docs/api.md) - API endpoints and schemas
- [Integration Guide](docs/integration.md) - How to integrate projects
- [Deployment Guide](docs/deployment.md) - vonnegut.local deployment

## License

MIT License - see [LICENSE](LICENSE) for details

## Links

- **Observatory**: https://observatory.nkllon.com
- **GitHub**: https://github.com/nkllon/beast-observatory
- **OpenFlow Playground**: https://github.com/louspringer/OpenFlow-Playground
- **Directus**: https://directus.io
- **Prometheus**: https://prometheus.io

---

**Status**: In Development  
**Infrastructure**: vonnegut.local + Cloudflare  
**Services**: Redis, Prometheus, Directus (running)  
**Next**: Implement FastAPI event ingestion

