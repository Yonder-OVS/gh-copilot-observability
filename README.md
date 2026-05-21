# GitHub Copilot OTel Monitoring — Azure-Free Stack

Monitors your VS Code GitHub Copilot usage (traces, metrics, logs) with a fully
local, open-source stack. No Azure account required.

## Stack

| Role         | Service        | Port(s)                |
|--------------|---------------|------------------------|
| Fan-out      | OTel Collector | `4318` (OTLP HTTP), `4317` (OTLP gRPC), `8889` (scrape) |
| Traces       | Jaeger         | `16686` (UI)           |
| Metrics      | Prometheus     | `9090`                 |
| Logs         | Loki           | `3100`                 |
| Dashboards   | Grafana        | `4000`                 |

## Quick Start

```bash
docker compose up -d
```

Wait ~15s for all services to be healthy, then open:
- **Grafana** → http://localhost:4000  (admin / admin)
- **Jaeger**  → http://localhost:16686
- **Prometheus** → http://localhost:9090

## Configure VS Code

Add to your VS Code `settings.json`:

```json
{
  "github.copilot.chat.otel.enabled": true,
  "github.copilot.chat.otel.exporterType": "otlp-http",
  "github.copilot.chat.otel.otlpEndpoint": "http://localhost:4318"
}
```

To also capture full prompts and responses (⚠️ may contain sensitive code):
```json
{
  "github.copilot.chat.otel.captureContent": true
}
```

## What Gets Captured

- **Traces** — Full `invoke_agent → chat → execute_tool` span trees per interaction
- **Metrics** — Token usage, LLM latency, TTFT, tool call counts, edit acceptance rates, lines of code
- **Events / Logs** — Session starts, per-tool invocations, user feedback (thumbs up/down), edit survival

## Grafana Dashboard

The pre-provisioned **"GitHub Copilot — Usage & Performance"** dashboard shows:
- Input/output token totals
- Chat session counts
- Edit acceptance rate
- Token usage over time by model
- LLM latency (p50/p95/p99)
- Tool call success rates
- Time to first token
- Edit acceptance by source (inline chat, agent, etc.)
- Lines of code added/removed
- Live trace explorer (Jaeger panel)

## Data Routing

```
VS Code (otlp-http :4318)
    │
    └─► OTel Collector
            ├─► Jaeger :4317      → traces
            ├─► Prometheus :8889  → metrics (scrape)
            └─► Loki :3100        → logs/events
```

> **Note:** VS Code sends all signals (traces, metrics, logs) to the OTel
> Collector on port `4318`. The collector fans out to all three backends.

## Stop

```bash
docker compose down
# To also remove stored data:
docker compose down -v
```
