# OpenTelemetry-AWS — Browse Slice Seed

Browse-path slice of the OpenTelemetry Astronomy Shop demo, seeded as the
starting point for the AWS-native rebuild. Sources are copied byte-identical
from the vendor demo clone; no code was modified in this seed commit.
Stripping/AWS-porting happens in later todos.

## Provenance

- Cloned from: `E:\OmniWatch-Telementry\OpenTelemetry-Demo(Cloned Repo)`
  (upstream `open-telemetry/opentelemetry-demo`)
- Source commit (full SHA): `898ede2f314b2c1e92a2d9a6ebd7efc188709290`
- Source commit (short SHA): `898ede2`
- Seed commit message: `chore(aws-slice): seed browse slice from demo 898ede2`
- Vendor references kept intact: `compose.vendor.yaml` (= upstream
  `compose.yaml`), `.env.vendor` (= upstream `.env`)

## Slice rationale

Keep only the user-facing browse path (frontend → catalog/recommendation →
cart → currency) plus its direct dependencies (feature flags, product-data
seed). Everything checkout/payment/shipping and Lane-A/C, observability, and
demo-only stays out so the AWS port starts from a minimal runnable surface.

## Slice members (why each)

| Path | Why kept |
|------|----------|
| `src/frontend` | Browse UI entry point (browse path starts here) |
| `src/product-catalog` | Product data + search backing browse pages |
| `src/recommendation` | Recommendations shown on browse/product pages |
| `src/currency` | Price conversion for displayed products |
| `src/cart` | Cart reached directly from browse (add-to-cart) |
| `src/flagd` (incl. `demo.flagd.json`) | Feature flags gating browse-path behavior |
| `src/postgresql/init.sql` | Product/catalog seed data for browse |
| `compose.vendor.yaml` | Vendor compose kept as reference (service wiring) |
| `.env.vendor` | Vendor env defaults kept as reference (ports/images) |

Note: `valkey-cart` is a `compose.yaml` **service** but has **no `src/`
directory** in the clone — it runs the stock image
`ghcr.io/valkey-io/valkey:9.1.2-alpine3.24` with no vendored source, so there
was nothing to copy (cart's Valkey dependency is satisfied by the image at
runtime). `src/postgresql` is copied whole (includes `init.sql` + Dockerfile).

## EXCLUDED (why not)

| Excluded | Why not |
|----------|---------|
| `src/checkout`, `src/payment`, `src/shipping` | Post-cart purchase path, not browse |
| `src/ad`, `src/quote`, `src/email`, `src/accounting` | Lane-A/C + auxiliary services, out of browse scope |
| `src/fraud-detection`, `src/image-provider` | Auxiliary/demo services, not browse dependencies |
| `src/otel-collector`, `src/opensearch`, `src/prometheus`, `src/grafana`, `src/jaeger` | Observability stack — OmniWatch provides its own ingestion |
| `src/kafka` | Demo event bus — replaced by AWS-native messaging later |
| `src/load-generator`, `src/flagd-ui`, `src/frontend-proxy` | Demo-only traffic/UI tooling, not shipped |
| `src/agent`, `src/mcp`, `src/chatbot`, `src/opamp-server`, `src/telemetry-docs` | Agent/telemetry demo extras, not browse |
| `src/react-native-app`, `src/checkout` extras | Alternate clients / out-of-scope |
| `compose.*.yaml` overlays, `Makefile`, CI, docs | Upstream build/test scaffolding, not needed for slice |

## Secrets note

`.env.vendor` was scanned before commit: it holds only vendor demo defaults
(`POSTGRES_PASSWORD=changeit`, `astronomy_password`, `monitoring_password`,
empty `API_KEY=`) — no real secrets. Real credentials come later via
Railway/env, never committed.

## Observer layer (box-preserved configs)

The running EC2 observer configs are preserved here: `compose.agent.yaml`
(additive overlay — never merged into `compose.aws.yaml`) mounts
`config/agent.vm.yaml` (agent VM-mode: no kubelet, emptied k8s lists, OTLP
export to `host.docker.internal:4317`) into `omniwatch-agent:aws-vm`, and
`config/otelcol-aws.yaml` is the box collector config (OTLP gRPC on 4317 →
batch → debug; on the box it runs as `otel/opentelemetry-collector-contrib:0.130.0`
with `/tmp/otelcol-aws.yaml` bound to `/etc/otelcol.yaml`). Bring the
observer layer up with `docker compose -f compose.aws.yaml -f
compose.agent.yaml up -d omniwatch-agent` (agent health on `:8081`). The box
`.env.aws` was reviewed and deliberately excluded: it equals
`.env.aws.example` except for the live Railway `DB_CONNECTION_STRING`, which
is never committed — copy `.env.aws.example` to `.env.aws` and inject the
real URL at deploy time.

## Plan pointer

Build plan: `E:\Project-OmniWatch\.omo\plans\opentelemetry-aws.md`
