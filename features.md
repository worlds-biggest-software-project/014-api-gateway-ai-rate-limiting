# API Gateway with AI Rate Limiting — Feature & Functionality Survey

> Candidate #14 · Researched: 2026-04-22

---

## Solutions Analysed

| Tool | Type | Licence / Model | URL |
|------|------|-----------------|-----|
| Kong Gateway | Open source + Enterprise SaaS | Apache-2.0 (OSS); proprietary Enterprise/Konnect | https://konghq.com |
| AWS API Gateway | Managed cloud service | Proprietary / pay-per-request | https://aws.amazon.com/api-gateway |
| Azure API Management | Managed cloud service | Proprietary / subscription tiers | https://learn.microsoft.com/azure/api-management |
| Google Apigee | Managed cloud service | Proprietary / pay-as-you-go | https://cloud.google.com/apigee |
| NGINX / NGINX Gateway Fabric | Open source + Commercial Plus | BSD-2-Clause (core OSS); proprietary NGINX Plus | https://nginx.com |
| Apache APISIX | Open source (Linux Foundation) | Apache-2.0 | https://apisix.apache.org |
| Traefik / Traefik Hub | Open source + Enterprise SaaS | MIT (OSS Traefik); proprietary Traefik Hub | https://traefik.io |
| Zuplo | Commercial SaaS | Proprietary / usage-based | https://zuplo.com |
| Unkey | Open source + hosted SaaS | MIT | https://unkey.com |
| Cloudflare API Gateway / AI Gateway | Commercial edge platform | Proprietary / usage-based | https://developers.cloudflare.com/ai-gateway |

---

## Feature Analysis by Solution

### Kong Gateway

**Core features**
- Multi-protocol routing: REST, GraphQL, gRPC, TCP, WebSocket, SSE
- Plugin-based middleware: 70+ first-party plugins covering authentication, rate limiting, logging, and transformations
- Standard Rate Limiting plugin (OSS): fixed window and sliding window algorithms, Redis-backed distributed counting
- Rate Limiting Advanced plugin (Enterprise): sliding window, per-consumer/service/IP, burst control, Redis Cluster support
- Kubernetes-native deployment via Kong Ingress Controller; Konnect SaaS control plane
- Developer portal with self-service API key management (Konnect, redesigned 2025)
- Declarative configuration (deck CLI), GitOps-compatible
- RBAC, audit logging, SSO, and workspace isolation (Enterprise)

**Differentiating features**
- AI Rate Limiting Advanced plugin (Enterprise, Kong 3.12+): token-based rate limiting for LLM APIs; limits by `prompt_tokens`, `completion_tokens`, or `total_tokens` per provider or policy window; sends `X-Kong-AI-RateLimit-*` headers back to client
- Semantic caching and semantic prompt routing (Enterprise AI Gateway): directs prompts to optimal model based on content
- AI proxy plugin: provider-agnostic facade covering OpenAI, Anthropic, AWS Bedrock, Google Vertex, Azure, Mistral, Cohere
- Prompt guardrails plugin: blocks prompt injection, enforces content policy at gateway layer
- OpenMeter integration (acquired September 2025): usage-based metering and billing by token consumption, Stripe/ERP invoicing
- Model-based rate limiting added in Kong 3.14 (2026): per-model limits across multi-model deployments
- Automated RAG pipeline and PII sanitisation plugins (2025)

**UX patterns**
- Konnect SaaS: web UI for service configuration, analytics, developer portal; configuration stored as YAML/JSON (deck)
- CRD-based configuration in Kubernetes is verbose; debugging misconfiguration still requires YAML inspection
- New Konnect Dev Portal (GA 2025): redesigned self-service portal with improved developer onboarding
- Plugin configuration done per-service or globally via Admin API; no low-code visual policy builder

**Integration points**
- Kubernetes Gateway API (KIC), Helm, Terraform provider, deck CLI
- Redis / Redis Cluster for distributed rate limit counters
- Prometheus, Datadog, OpenTelemetry, Zipkin, Jaeger (observability)
- OAuth 2.0, OIDC, LDAP, SAML, mTLS (auth)
- Stripe and ERP via OpenMeter for usage billing

**Known gaps**
- AI Rate Limiting Advanced is enterprise-only; no open-source token-aware limiting
- No native ML-based behavioral abuse detection; pattern-based blocking requires custom plugins or third-party WAF
- Developer portal historically criticised for immature UX; partially addressed in 2025 redesign
- AI features require Konnect or Enterprise licence, creating a hard feature cliff between OSS and paid tiers
- Documentation gaps noted in public GitHub issues, particularly for AI plugin edge cases
- CRD verbosity in Kubernetes operator (1.5+) is a usability friction point

**Licence / IP notes**
- Kong Gateway OSS: Apache-2.0. All core plugins shipped under the same licence.
- Konnect and Enterprise plugins are proprietary. AI Rate Limiting Advanced, Semantic Caching, and Prompt Guardrails are not open source.
- No known patent encumbrances on the OSS codebase.

---

### AWS API Gateway

**Core features**
- HTTP API and REST API variants; WebSocket API for persistent bidirectional connections
- Token-bucket throttling at account (10,000 RPS default), stage, method, and per-client (usage plan + API key) levels
- Usage plans: define quota (calls per day/week/month) and rate/burst per API key
- Built-in request/response transformation, payload mapping templates (VTL)
- Lambda authorizers, Cognito user pool authorizers, IAM-based auth
- Mutual TLS, private API endpoints via VPC endpoints
- CloudWatch integration for metrics, access logs, tracing via AWS X-Ray
- OpenAPI 3.0 import/export for automated provisioning
- Developer portal added November 2025 (GA): CloudWatch RUM analytics for user engagement tracking

**Differentiating features**
- Deep AWS ecosystem lock-in is also its primary differentiator: native Lambda integration (proxy and custom integrations), EventBridge, Step Functions, SQS, Kinesis targets without extra infrastructure
- Amazon Q Developer integration for AI-assisted API analysis (2025)
- Private REST APIs served entirely within a VPC without public internet exposure

**UX patterns**
- AWS Console UI for gateway management; reasonably mature but limited compared to standalone API management platforms
- Stage-based deployment model (dev/staging/prod as separate stages); requires CloudFormation or Terraform for GitOps workflows
- Developer portal (2025) is basic relative to Kong Konnect or Azure APIM; SSO integration and multiple usage-plan support noted as limitations
- No visual policy editor; policies defined via JSON/VTL mapping templates

**Integration points**
- Native: Lambda, DynamoDB, S3, SQS, EventBridge, Step Functions, Kinesis
- AWS WAF for IP-based blocking and rate-based rules (distinct from API Gateway throttling)
- AWS Shield for DDoS mitigation
- CloudWatch, X-Ray, CloudTrail (observability and audit)
- Cognito, IAM, LDAP via Lambda authorizer

**Known gaps**
- Behavioral abuse detection not built in; requires custom Lambda authorizers or AWS WAF rule stacks
- No token-aware LLM rate limiting; AI/LLM traffic treated as standard HTTP
- Developer portal (added late 2025) still lacks CI/CD pipeline support and complex SSO configurations
- Strong AWS lock-in; migrating to a different gateway is costly
- No pub/sub or broadcast messaging primitive
- 29 MB payload limit for REST APIs; unsuitable for large streaming responses without workarounds
- Account-wide throttle quota (10,000 RPS) shared across all APIs in a region; can cause cross-API interference

**Licence / IP notes**
- Fully proprietary managed service; no source code available.
- No OSS licensing considerations. Usage subject to AWS Service Terms.

---

### Azure API Management

**Core features**
- Policy-based request/response pipeline applied at global, product, API, or operation scope
- `rate-limit` policy: fixed-window across all subscribers; `rate-limit-by-key` policy: token-bucket per arbitrary key expression (IP, JWT claim, header, query parameter)
- `quota` and `quota-by-key`: longer-period call or bandwidth quotas
- `azure-openai-token-limit` policy: token-bucket rate limiting on Azure OpenAI endpoints by prompt or total tokens per minute
- `llm-token-limit` policy: same mechanism extended to any OpenAI-compatible third-party inference provider
- `azure-openai-emit-token-metric` and `llm-emit-token-metric`: emit token usage to Azure Monitor for observability
- OAuth 2.0/OIDC, JWT validation, client certificate auth
- Built-in developer portal with OpenAPI doc rendering and interactive try-it console
- Self-hosted gateway for on-premises / hybrid deployment (container-based)

**Differentiating features**
- `azure-openai-token-limit` is among the earliest GA-production token-aware rate limiting policies in a major managed gateway (GA 2024–2025); supports both rate (tokens/minute) and quota (tokens/period) with 403 on quota breach and 429 on rate breach — two distinct response codes for two distinct enforcement types
- Policy expressions use C# expressions: very flexible, but unique to Azure APIM
- Managed identity integration: APIM can authenticate to Azure OpenAI without key management
- Native integration with Azure Monitor, Application Insights, and Event Hubs for full observability pipeline

**UX patterns**
- Azure Portal UI with policy editor (XML-based policy language); some users find the XML policy syntax unintuitive
- Developer portal is configurable but requires effort to customise; hosted or self-hosted options
- Policy scoping (global/product/API/operation) is powerful but complexity grows quickly in large deployments
- Getting simple setups done "takes too many steps" per multiple Gartner Peer Insights reviews

**Integration points**
- Azure OpenAI, Azure AI Model Inference API, third-party OpenAI-compatible endpoints
- Azure Active Directory / Entra ID for auth; Azure Monitor, App Insights, Event Hubs
- Logic Apps, Azure Functions, Service Bus as backends
- Terraform AzureRM provider for IaC; ARM templates; Bicep

**Known gaps**
- Token limit policy cannot predict completion size at pre-flight; concurrent requests can temporarily exceed limits before counters update
- No ML-based behavioral abuse detection; pattern-based detection requires Azure WAF or Defender for APIs
- Pricing not publicly transparent; Enterprise contract only for Standard/Premium tiers; Standard v2 starts ~$220/month
- Role-based access, proxy versioning, and OPDK-equivalent for non-Azure-cloud deployments are consistently noted as improvement areas in peer reviews
- Deployment of new policy revisions can be slow; minor changes may trigger full policy recompile

**Licence / IP notes**
- Fully proprietary managed service. Self-hosted gateway image is a commercial binary.
- `azure-openai-token-limit` policy implementation details are not public.

---

### Google Apigee

**Core features**
- Full API lifecycle management: design, publish, analyse, monetise, deprecate
- Spike Arrest and Quota policies (flow rate limiting): enforce smooth rate limits and period quotas
- Apigee Advanced API Security add-on: ML-powered abuse detection (Apigee Sense successor), security incident grouping, bot pattern classification
- REST, GraphQL, gRPC, SOAP proxy support
- Extensible proxy pipeline: JavaScript, Python, Java callout policies
- Built-in developer portal (Integrated Portal or Drupal-based)
- API monetisation: rate card, freemium, revenue share models
- Analytics dashboard with traffic, error, latency, and geography breakdowns

**Differentiating features**
- Apigee Advanced API Security (ML abuse detection): the most mature production-grade behavioral abuse detection in a managed gateway; ML models are trained per-organisation on that org's traffic, improving accuracy over time; detects API scraping, credential stuffing, and business-logic attacks that bypass request-count limits; generates security incidents (grouped anomaly clusters) rather than raw alerts
- IP allowlisting/denylisting actions directly from the abuse detection UI (added 2025)
- VPC-SC (VPC Service Controls) support for data-perimeter compliance (2025)
- API monetisation built-in: no third-party billing integration required for revenue share or rate card models
- CVE-2025-13292 cross-tenant data access vulnerability patched in 2025; relevant as a security provenance note

**UX patterns**
- Apigee UI in Google Cloud Console; developer portal separate (Integrated Portal or external)
- Proxy deployment model with revision-based deployments; deployment latency (even for minor policy changes) is a frequently cited complaint
- XML-based proxy bundle configuration; learning curve described as "heavy" for first-time users
- Role-based access for portal content and proxy environments needs improvement (noted in multiple Gartner reviews)

**Integration points**
- Google Cloud natively: Cloud Logging, Cloud Monitoring, Cloud Armor (WAF), Identity Platform, Secret Manager
- BigQuery export for deep traffic analytics
- Pub/Sub for async event patterns
- gRPC-JSON transcoding built-in

**Known gaps**
- Most expensive managed gateway: $30/million API calls on pay-as-you-go (8× AWS); no public pricing for subscription; extensible proxies cost 5× standard proxies
- ML abuse detection is enterprise-add-on only; not available on free or base tiers
- Slow proxy deployment for policy changes — frequently noted in G2 and Gartner reviews
- Complex for small teams; significant learning curve for proxy model and policy XML
- Vendor lock-in: Apigee proxy bundles are proprietary format with no standard migration path
- CVE-2025-13292 demonstrated potential for privilege escalation in multi-tenant environments

**Licence / IP notes**
- Fully proprietary. Apigee is a Google Cloud product following acquisition of Apigee Inc. (2016).
- No OSS components in the core gateway. Advanced API Security add-on is additional proprietary software.

---

### NGINX / NGINX Gateway Fabric

**Core features**
- High-performance HTTP/HTTPS reverse proxy and load balancer; foundational for many upstream gateways
- `ngx_http_limit_req_module`: leaky-bucket rate limiting per shared memory zone, keyed by any NGINX variable (IP, cookie, header)
- Burst parameter: allows temporary request bursts above the defined rate before dropping or delaying
- `limit_req_zone` supports multiple zones for per-endpoint rate differentiation (e.g., strict on `/auth`, lenient on `/static`)
- NGINX Gateway Fabric: official Kubernetes Gateway API implementation replacing deprecated Ingress NGINX (retirement March 2026)
- RateLimitPolicy CRD on NGINX Gateway Fabric for Kubernetes-native rate limiting
- SSL/TLS termination, HTTP/2, gRPC proxying, mTLS upstream
- NGINX Plus (commercial): Active health checks, session persistence, advanced monitoring dashboard, OIDC integration, JWT validation, key-value store for dynamic configuration

**Differentiating features**
- Raw performance: lowest latency and highest throughput of any gateway in this survey; proven at global CDN and streaming scale
- Configuration composability: any upstream gateway or full API management platform can be built on top of NGINX as the data plane
- F5 AI Gateway (2025–2026): separate product that works alongside NGINX; examines LLM prompts and responses for prompt injection and data exfiltration; fills NGINX's native AI gap
- `ngx_http_limit_req_module` is extremely battle-tested with millions of production deployments

**UX patterns**
- NGINX configuration is declarative text files (`nginx.conf`); no built-in UI in OSS; NGINX Plus has a basic dashboard
- No developer portal, no API catalogue, no self-service key management in OSS
- NGINX Gateway Fabric uses Kubernetes CRDs; GatewayClass / Gateway / HTTPRoute pattern from Gateway API spec
- Operator experience requires knowledge of NGINX configuration syntax; harder learning curve than UI-driven gateways

**Integration points**
- Prometheus metrics endpoint (`nginx-prometheus-exporter`); OpenTelemetry support
- Redis (via NGINX Plus keyval module) for distributed shared state
- Envoy, Kong, APISIX, and Traefik all offer NGINX-compatible or NGINX-replacing deployments
- Kubernetes: Gateway API (Fabric), or legacy Ingress (being retired)

**Known gaps**
- `limit_req` shared memory zone is local to one NGINX instance; distributed rate limiting across a cluster requires external coordination (Redis or NGINX Plus keyval) — not built-in OSS
- No built-in developer portal, API analytics, API key management, or OAuth 2.0 flows in OSS
- No AI-specific features in core NGINX: no token counting, no LLM cost metering, no semantic routing
- No behavioral abuse detection or ML integration; requires F5 AI Gateway or third-party WAF layer
- Ingress NGINX officially retired March 2026; migrations to NGINX Gateway Fabric required with some breaking API differences

**Licence / IP notes**
- NGINX OSS: BSD-2-Clause. Fully permissive; commercial use allowed without restriction.
- NGINX Plus: proprietary commercial binary. NGINX Gateway Fabric: Apache-2.0.
- F5 AI Gateway: proprietary commercial product.

---

### Apache APISIX

**Core features**
- High-performance cloud-native gateway written in Lua/LuaJIT on OpenResty/NGINX; sub-millisecond routing latency
- 100+ built-in plugins; extensible via Lua, WebAssembly (Rust/C++/Go), Java, Python external runners
- `limit-count`: fixed window rate limiting; `limit-req`: leaky-bucket; `limit-conn`: concurrent connection limiting
- `ai-rate-limiting` plugin (released 2025): token-based rate limiting for LLM APIs; limits by `prompt_tokens` or `total_tokens`; supports per-route, per-service, per-consumer, per-consumer-group, or custom dimension; fixed and sliding window algorithms; single-machine and Redis cluster modes
- `ai-proxy` and `ai-proxy-multi`: provider-agnostic LLM proxy for OpenAI, Anthropic, DeepSeek, Claude, Mistral, Gemini, and more
- Redis-backed distributed counting for all rate limiting plugins
- Admin API + etcd for dynamic configuration with sub-millisecond hot-reload
- `consumer` model: per-key rate limits and quotas assignable to individual API consumers

**Differentiating features**
- Only open-source (Apache-2.0) gateway with a production-ready token-aware LLM rate limiting plugin as of 2025–2026
- AI Gateway mode (announced 2025): unified interface for routing traffic to multiple LLM providers, token tracking, and cost attribution without a separate product or enterprise licence
- MCP (Model Context Protocol) support for AI agent traffic management (2026 roadmap)
- WebAssembly plugin runtime: plugins can be written in Rust, C++, or Go and run in WASM VM within APISIX — no performance penalty of external process calls
- Apache Software Foundation governance: well-defined IP policy; all contributions must pass ICLA/CCLA review; no contributor licence agreement lock-in risk

**UX patterns**
- No first-class GUI in OSS APISIX; dashboard is a separate open-source project (`apisix-dashboard`) that lags behind core feature releases
- API7.ai (commercial) provides the primary supported GUI, cloud platform, and enterprise support
- Configuration via Admin API (REST/JSON) or declarative YAML in standalone mode; etcd dependency adds operational overhead
- AI plugin documentation is English-only (as of early 2026); Chinese docs noted as missing in project issue tracker
- `ssl_trusted_certificate = system` required for external AI provider connections; not documented clearly, causes "unable to get local issuer certificate" errors

**Integration points**
- etcd (required for cluster mode), Redis / Redis Cluster (distributed rate limiting)
- Prometheus, OpenTelemetry, Zipkin, SkyWalking (observability)
- OAuth 2.0, OIDC, JWT, LDAP, Casbin RBAC (auth / authz)
- Kubernetes Gateway API, Helm, Ingress controller
- WebAssembly, external plugin runners (Java, Python, Go via unix socket)

**Known gaps**
- No built-in ML behavioral abuse detection; pattern-based blocking only
- `apisix-dashboard` OSS UI lags core; AI plugin configuration not exposed in dashboard
- Full-sync Admin API in standalone mode: on every config fetch the entire configuration is transmitted; causes excessive network load under high churn
- Frequent config changes trigger full radixtree rebuild, degrading route lookup performance temporarily
- `server-info` plugin deprecated (will be removed) due to etcd write performance issues at scale
- AI documentation gaps (English-only) for a globally used project

**Licence / IP notes**
- Apache-2.0 licence. Linux Foundation / Apache Software Foundation project.
- All contributions governed by Apache ICLA; IP clean.
- `ai-rate-limiting` plugin is open source under Apache-2.0. No enterprise-gate on token rate limiting.

---

### Traefik / Traefik Hub

**Core features**
- Cloud-native reverse proxy and ingress controller written in Go; automatic service discovery from Docker, Kubernetes, Consul, etcd
- `RateLimit` middleware: per-client token-bucket rate limiting; configurable average rate, period, and burst size
- `DistributedRateLimit` middleware (Traefik Hub Enterprise): cluster-wide rate limiting via persistent KV storage; ensures limits are enforced globally, not just per pod
- Dynamic configuration: no restart required for routing changes
- Let's Encrypt integration for automated TLS certificate provisioning
- Traefik Hub: API portal, API key management, OIDC integration, subscription management
- OpenAPI v3.1 / Swagger 2.0 spec ingestion; auto-converts Swagger 2 to OpenAPI 3.1
- Namespace isolation in Kubernetes: multiple isolated gateway instances per cluster (2025)
- `ManagedApplication` CRD: declarative application onboarding and API key pre-provisioning

**Differentiating features**
- Zero-configuration service discovery is Traefik's primary differentiator: automatically detects new services from container labels or Kubernetes annotations, no manual routing configuration required
- GitOps-native: all configuration stored as CRDs in Git; no external database or etcd required for OSS mode
- OIDC + RateLimit chaining: rate limits keyed on extracted JWT user claims (user-tier-aware limiting without custom code)
- Traefik Hub API Portal: self-service API subscription with instant access; bundle subscriptions for multiple APIs at once (2025)
- Search functionality in API Portal UI (February 2025)

**UX patterns**
- Traefik OSS: configuration via Docker labels or Kubernetes annotations; no UI; Traefik dashboard provides basic real-time metrics
- Traefik Hub: web-based API portal for API consumers; API publisher configures via CRDs
- Middleware chain model is composable and readable; simpler than Kong's plugin model for basic use cases
- Developer portal analytics: error and usage analytics available in Hub; less mature than Kong Konnect or Azure APIM

**Integration points**
- Docker, Kubernetes, Consul, etcd, Rancher (service discovery)
- Let's Encrypt (ACME), Vault (secret management)
- Prometheus, Datadog, InfluxDB, Jaeger, Zipkin (observability)
- OAuth 2.0, OIDC, JWT, ForwardAuth (external auth service delegation)
- Traefik Hub Enterprise uses persistent KV store (unspecified; Redis-compatible) for distributed rate limiting

**Known gaps**
- No AI-specific features: no token-aware LLM rate limiting, no LLM cost metering, no model routing
- No ML behavioral abuse detection
- Raw throughput lower than NGINX or APISIX under heavy middleware processing due to Go vs. LuaJIT/C performance profile
- Traefik Hub (Enterprise) required for distributed rate limiting and API management; OSS has no cluster-wide enforcement
- Plugin/middleware ecosystem significantly smaller than Kong (70+ plugins) or APISIX (100+)
- No native request transformation (body rewrite, JSON-to-XML, schema validation) without ForwardAuth delegation

**Licence / IP notes**
- Traefik OSS: MIT licence. Fully permissive.
- Traefik Hub: proprietary commercial product. Distributed RateLimit and API Portal are not open source.
- No known patent concerns.

---

### Zuplo

**Core features**
- Fully managed edge API gateway deployed across 300+ global edge data centers; zero infrastructure to manage
- Per-API, per-route, per-user, per-API-key rate limiting configurable via JSON; no DSL required
- Global rate limit state synchronised across all edge nodes by default; a user hitting from any geography draws from the same counter
- Built-in developer portal with auto-generated OpenAPI documentation and interactive try-it console
- API key management: issuance, revocation, per-key metadata, tier assignment
- Custom rate limiting logic via TypeScript policies: per-customer-tier dynamic burst allowances, business-rule-based throttling
- GitOps-native: entire gateway definition stored in text files (JSON/TypeScript) in source control
- API monetisation hooks and Stripe integration
- Unlimited preview environments per branch for testing policy changes

**Differentiating features**
- TypeScript-first extensibility: custom policies are TypeScript functions, not Lua/Go/Java; largest pool of potential contributors
- Instant global enforcement without Redis cluster management; distributed counter state is platform-managed
- GitOps + preview environments: each PR gets an isolated gateway deployment for integration testing — a DX capability no other gateway in this survey offers out of the box
- SaaS architecture eliminates cold-start latency issues (edge pre-warmed globally)

**UX patterns**
- Configuration via JSON files in a Git repository; changes deployed on push; no imperative CLI for routine changes
- Web UI for monitoring and analytics; TypeScript editor for custom policy authoring
- Developer portal is included and customisable without separate infrastructure
- Described as "rate limiting without the rage" in Zuplo's own 2026 guide — emphasises developer ergonomics over raw feature depth

**Integration points**
- OpenAPI 3.x import for auto-configuration
- Stripe (monetisation)
- AWS Marketplace and Azure Marketplace listings
- Standard HTTP webhooks for event integration
- TypeScript policy hooks for custom integration with any external system

**Known gaps**
- SaaS-only; no self-hosted deployment (primary architectural constraint)
- No ML behavioral abuse detection
- No native token-aware LLM rate limiting as of April 2026
- No service mesh integration; no gRPC support at time of writing
- Pricing wall: free tier → Builder ($25/month) → Business ($500/month); enterprise undisclosed
- Limited observability depth compared to Kong or APIM; no native APM integration

**Licence / IP notes**
- Fully proprietary SaaS. Source code not public.
- No OSS components in the runtime. Custom TypeScript policies authored by customers remain customer IP.

---

### Unkey

**Core features**
- Purpose-built API key lifecycle management: create, verify, revoke, expire keys with a single SDK call
- Per-key rate limiting: configurable per-key limits (requests per window); uses sliding window algorithm; sub-millisecond enforcement at global edge
- Standalone rate limiting (namespace-based): rate limit any identifier without requiring API key issuance
- Per-key metadata: arbitrary JSON metadata attached to each key for tier, plan, or user segmentation
- Time-based keys: automatic expiration without cron jobs
- Limited-use keys: keys that self-revoke after N verifications
- Per-key analytics: usage counts, rate limit events, verification latency
- Audit logs: full history of key creation, revocation, and verification events
- Role and permission-based access control: granular permissions propagated globally in seconds

**Differentiating features**
- Narrowly scoped, composable primitive: Unkey does one thing well — API key issuance + rate limiting — and does it with sub-millisecond global enforcement and zero operational overhead; it is not a gateway and does not attempt to be one
- Per-key dynamic overrides: rate limits and quotas can be changed per-key via API without redeployment; enables real-time tier upgrades
- Open-source and self-hostable (MIT): the only tool in this survey that offers global-grade key management and rate limiting as open source with no enterprise paywall
- SDK-first: verification is a single function call; no proxy hop required; can be used alongside any existing gateway

**UX patterns**
- Unkey Dashboard: web UI for viewing keys, usage, rate limit events; simple and focused
- SDK-first developer experience: `verifyKey()` returns a structured response including remaining quota, valid status, and rate limit state
- No policy editor, no visual routing, no developer portal (API key list is not a developer portal)
- Described consistently as "simplified" in community reviews — friction-free onboarding

**Integration points**
- REST API for all operations; SDKs for Node.js, Python, Go, Rust, and more
- Works alongside any reverse proxy or gateway (NGINX, Kong, APISIX, Traefik) at the application layer
- Webhook support for key lifecycle events

**Known gaps**
- Not a full API gateway: no routing, no load balancing, no auth beyond key verification, no request transformation
- No ML behavioral abuse detection; anomaly flagging is pattern/threshold-based only
- No token-aware LLM rate limiting (operates purely on request count)
- Analytics are per-key only; no global traffic analytics or latency histograms
- No developer portal for API consumers; no OpenAPI doc hosting
- Self-hosted deployment requires operational effort that the SaaS hosted tier abstracts away

**Licence / IP notes**
- MIT licence. Core repository at github.com/unkeyed/unkey.
- MIT is permissive but provides no patent grant (unlike Apache-2.0); this is a mild IP consideration for commercial derivative products.
- Hosted SaaS offering uses the same codebase; no open-core split noted as of April 2026.

---

### Cloudflare API Gateway / AI Gateway

**Core features**
- Edge rate limiting integrated with WAF and DDoS mitigation; enforced globally across 300+ data centers
- Rate limiting rules: per-request-parameter granularity (IP, ASN, cookie, header, query string, URI path, JSON body field, JA3 fingerprint); sliding and fixed window algorithms
- Bot detection: ML-based bot score assigned to every request; rules can block, challenge (CAPTCHA), or log based on score
- Cloudflare AI Gateway (separate product, free tier): unified proxy for LLM providers (OpenAI, Anthropic, DeepSeek, Google AI Studio, Mistral, Perplexity, Groq, Grok, Replicate, Workers AI)
- AI Gateway: exact-match response caching (identical prompts served from cache; semantic caching planned, not GA)
- AI Gateway: model fallback — automatically retries failed LLM requests against a backup provider
- AI Gateway: per-gateway rate limiting for AI traffic (requests per time window)
- AI Gateway: full request/response logging including prompt, response, token usage, cost estimate, latency, and provider
- AI Gateway: visual routing rules for A/B testing, geographic routing, and user-segment routing without code changes
- Workers AI: serverless AI inference on Cloudflare's GPU network; tight AI Gateway integration

**Differentiating features**
- Bot management is the most mature in this survey: ML-based bot score (0–99) on every request, JA3/JA4 TLS fingerprinting, behavioural analysis, and challenge orchestration — this is Cloudflare's core competence transferred to API traffic
- Zero-configuration global distribution: rate limiting state is globally consistent by default; no Redis, no etcd, no external store
- Unified billing for third-party LLM usage (2026): pay OpenAI / Anthropic costs via Cloudflare invoice; reduces procurement complexity
- AI Gateway is free for core features (caching, rate limiting, logging, fallback) with no per-call gateway fee
- Workers: TypeScript/JavaScript policy execution at edge with ~0ms cold start; enables custom rate limiting and transformation logic without a separate service

**UX patterns**
- Cloudflare Dashboard: comprehensive UI for WAF, rate limiting, bot management, and AI Gateway configuration; well-reviewed for clarity
- AI Gateway dashboard: per-request log viewer with prompt/response/cost/token; analytics for usage patterns
- Visual routing builder for AI Gateway: drag-and-drop flow without code changes
- Workers environment: write TypeScript; `wrangler` CLI for local dev; git-based deployment

**Integration points**
- WAF, DDoS protection, CDN, Workers (all native Cloudflare products)
- AI providers: OpenAI, Anthropic, Gemini, Mistral, DeepSeek, Groq, Grok, Replicate, Workers AI, Cerebras
- Vectorize vector database for semantic search and RAG context (used alongside AI Gateway)
- D1 (SQLite at edge), KV, R2 (object storage) for Workers state
- Cloudflare Logpush: log streaming to S3, Datadog, Splunk, BigQuery

**Known gaps**
- Semantic caching for AI responses: planned but not GA as of April 2026; exact-match only currently
- AI Gateway rate limiting is request-count-based; no token-aware rate limiting for LLM cost control
- No ML behavioral abuse detection specific to API business logic (Cloudflare bot score is network/TLS layer; business-logic abuse detection requires Workers custom code)
- Workers CPU limit: 50ms CPU time per request on free tier (30 seconds on paid); complex ML inference not practical in Workers
- Vendor lock-in: Cloudflare Workers and AI Gateway use Cloudflare-specific runtime APIs; migrating custom logic requires rewrite
- No developer portal for API consumers; no API catalogue or self-service subscription management

**Licence / IP notes**
- Fully proprietary managed service.
- Workers runtime is based on V8 (open source); the Cloudflare platform wrapping it is proprietary.
- No OSS licensing concerns for users; no ability to self-host.

---

## Cross-Cutting Feature Themes

### Table-Stakes Features

Any API gateway entering the market in 2026 is expected to provide all of the following without differentiation:

- **Request routing and load balancing**: HTTP/HTTPS proxying to multiple upstream backends
- **Rate limiting (request-count-based)**: per-IP, per-key, and per-consumer limits; token bucket or sliding window; 429 response with `Retry-After` header (RFC 6585)
- **Authentication and authorisation**: API key validation, JWT verification, OAuth 2.0 / OIDC integration
- **TLS termination**: HTTPS at the gateway edge; Let's Encrypt or BYO certificate
- **Request/response transformation**: header manipulation, basic body transformation
- **Observability**: request logs, error rates, latency histograms; Prometheus metrics and/or OpenTelemetry export
- **OpenAPI 3.x support**: import specs for automated route configuration; export for documentation
- **Health checks and circuit breaking**: upstream health probes; automatic failover
- **CORS policy enforcement**
- **Kubernetes deployment**: Helm chart or Gateway API CRD compatibility

### Differentiating Features

Capabilities that provide competitive advantage and are not universally implemented:

- **Token-aware LLM rate limiting**: limiting by prompt/completion/total tokens rather than request count; aligns gateway controls with LLM provider billing dimensions. Currently only Kong (enterprise), Azure APIM (`azure-openai-token-limit`), Apache APISIX (`ai-rate-limiting` — OSS), and Azure APIM (`llm-token-limit`) implement this. Cloudflare and Zuplo do not yet have it.
- **ML-based behavioral abuse detection**: pattern-aware anomaly detection that catches low-and-slow attacks, credential stuffing, API scraping, and business-logic abuse that bypasses request-count limits. Only Apigee Advanced API Security offers this in a managed gateway; no open-source gateway includes a deployable ML abuse detection layer.
- **Semantic prompt caching**: serving cached LLM responses for semantically similar (not only identical) prompts. Kong (enterprise semantic caching) partially implements this; Cloudflare has it planned; no OSS implementation exists.
- **Multi-model LLM routing and failover**: routing requests to the optimal LLM provider based on cost, latency, availability, or prompt content. Cloudflare AI Gateway and Kong AI Gateway (enterprise) offer fallback; true semantic routing is enterprise-only.
- **GitOps-native configuration**: gateway configuration stored entirely as source-controlled text files with PR-based review and deploy workflows. Zuplo, APISIX (YAML mode), and Traefik do this natively. Kong requires deck CLI as a layer on top of Admin API.
- **Edge-global distributed rate limiting without operator-managed Redis**: Zuplo and Cloudflare abstract the distributed counter store; all others require Redis cluster management.
- **Dynamic per-key limit overrides via API**: real-time tier adjustments without redeployment. Unkey is the strongest here; Kong and APISIX support it via Admin API but not as a first-class DX feature.
- **Unified LLM cost attribution and usage billing**: metering token consumption per consumer and invoicing. Kong (with OpenMeter), Azure APIM, and Apigee partially cover this; no open-source OSS-complete solution exists.

### Underserved Areas / Opportunities

Genuine gaps that no current solution fully addresses, particularly relevant for an AI-native gateway project:

1. **Open-source token-aware LLM rate limiting with cost metering**: APISIX's `ai-rate-limiting` plugin limits tokens but does not attribute cost (USD) per consumer or produce invoicing data. Kong's full cost metering is enterprise-only. An open-source gateway that limits by tokens AND produces per-consumer cost reports would be the only such tool.

2. **Deployable open-source ML behavioral abuse detection**: Research papers demonstrate 95–98% accuracy for ML-based API abuse detection (Random Forest, SVM on traffic feature vectors). Apigee Sense is the only production implementation, and it is proprietary, Google-cloud-only, and paid-add-on. No open-source gateway offers a pluggable ML abuse detection engine. This is the largest single gap in the market.

3. **Dynamic adaptive rate limiting based on backend health signals**: All gateways apply static rate limits (100 req/min regardless of backend state). No gateway dynamically reduces limits when backend error rate exceeds a threshold or latency spikes — a form of AI-assisted circuit breaking that prevents cascade failures without manual intervention.

4. **AI agent traffic classification**: Traditional rate limiting cannot distinguish between a legitimate high-volume AI agent pipeline and a botnet generating similar request patterns. AI agents make burst-heavy, semantically coherent API call sequences; botnets make burst-heavy, semantically random ones. No gateway currently classifies traffic by agent vs. human vs. bot using behavioral sequence analysis.

5. **Unified control plane for heterogeneous gateways**: 31% of organisations run multiple API gateways. No open-source tool provides a single policy management, cost attribution, and anomaly detection layer across Kong + NGINX + AWS API Gateway deployments. This is an architectural pain point with no current OSS solution.

6. **Semantic caching for LLM responses (OSS)**: Kong's semantic caching is enterprise-only; Cloudflare's is planned but not GA. No open-source implementation of vector-similarity-based prompt cache exists in a gateway context.

7. **Zero-day API abuse detection via few-shot learning**: Academic work (arXiv:2405.11247) demonstrates FastText + approximate nearest-neighbour search for detecting novel attack patterns with minimal training examples. No production gateway implements this approach.

### AI-Augmentation Candidates

Areas where replacing static rule-based logic with ML/AI models would deliver the highest value:

| Current approach | AI-augmented replacement | Expected improvement |
|---|---|---|
| Static request-count rate limits per IP | Adaptive per-consumer limits based on historical baseline, time-of-day patterns, and anomaly score | Eliminates both over-blocking legitimate users and under-blocking slow attackers |
| Fixed circuit breaker thresholds (e.g., 50% error rate) | Backend health predictor: adjusts rate limits proactively as latency trends upward, before error rate spikes | Earlier intervention; fewer cascade failures |
| Rule-based bot detection (User-Agent matching, known IP lists) | ML classifier on request feature vectors: timing jitter, request sequence entropy, header fingerprint, TLS JA3 score, geographic velocity | Higher accuracy on novel bot patterns; lower false-positive rate on legitimate automation |
| Per-endpoint quota with manual tier assignments | Usage pattern clustering: automatically segments consumers by behaviour and suggests quota tiers; flags outliers for human review | Reduces manual policy administration; surfaces abusive consumers before they breach SLA |
| Exact-match response caching for LLM | Semantic vector cache: serve cached responses for semantically equivalent prompts | 30–70% cache hit rate improvement for LLM API traffic (vs. near-zero for exact match) |
| Manual incident review for API security events | ML incident grouper: clusters related anomaly events into coherent security incidents (Apigee Sense model) | Reduces alert fatigue; surfaces actionable incidents rather than raw alert streams |

---

## Legal & IP Summary

| Tool | Licence | Key IP Notes |
|------|---------|--------------|
| Kong Gateway OSS | Apache-2.0 | Clean; AI Rate Limiting Advanced and enterprise plugins are proprietary — cannot be forked or embedded |
| AWS API Gateway | Proprietary | No reuse rights; API semantics may be imitated but not the implementation |
| Azure API Management | Proprietary | `azure-openai-token-limit` policy implementation is not public |
| Google Apigee | Proprietary | CVE-2025-13292 cross-tenant privilege escalation patched 2025; Apigee proxy bundle format is proprietary |
| NGINX OSS | BSD-2-Clause | Highly permissive; commercial derivatives permitted; NGINX Plus binary is proprietary |
| NGINX Gateway Fabric | Apache-2.0 | Clean |
| Apache APISIX | Apache-2.0 | Apache ICLA governance; `ai-rate-limiting` plugin is OSS with no enterprise gate; patent grant included in Apache-2.0 |
| Traefik OSS | MIT | Permissive; no patent grant (MIT); Traefik Hub is proprietary |
| Zuplo | Proprietary SaaS | No OSS components; customer TypeScript policies are customer IP |
| Unkey | MIT | Permissive; no patent grant; self-hostable; MIT provides no explicit patent licence (minor consideration vs. Apache-2.0) |
| Cloudflare AI Gateway | Proprietary | V8 engine is OSS; Cloudflare platform wrapping is proprietary; no self-hosting option |

**Patent risk assessment**: No known active patents covering token-bucket rate limiting, sliding window counters, or JWT-claim-based throttling. ML abuse detection methods described in academic literature are generally not patented in the API gateway domain. Apigee Sense's specific model training approach may have trade-secret protection rather than patent protection; independent implementation avoids this concern. Semantic caching via vector similarity is a well-established technique (vector databases, embedding models) with no dominant patent holder in the API gateway context.

**Safe open-source baseline**: Apache-2.0 (explicit patent grant, no trademark risk, high compatibility) is the recommended licence for any open-source derivative work in this space. MIT is acceptable but provides no patent grant. GPL/AGPL would impose copyleft requirements that limit commercial adoption.

---

## Recommended Feature Scope

### Must-Have (MVP)

- **Request routing with multi-upstream load balancing**: round-robin, least-connections, and weighted; health-check-driven failover; HTTP/HTTPS/gRPC
- **Token-aware rate limiting for LLM APIs**: limit by `prompt_tokens`, `completion_tokens`, and `total_tokens` per consumer, per model, and per time window; expose `X-RateLimit-*` headers; open source under Apache-2.0 (fills the market gap that APISIX partially addresses but without cost attribution)
- **Request-count rate limiting (table stakes)**: sliding window, per-IP/consumer/key; Redis-backed distributed counting; 429 + `Retry-After` responses
- **API key management**: issue, verify, revoke, expire; per-key metadata for tier and plan; sub-millisecond verification latency
- **JWT / OAuth 2.0 validation**: signature verification, claim extraction, claim-based rate limit keying
- **Structured observability**: request logs (prompt, response, token count, cost estimate, latency, status); Prometheus metrics; OpenTelemetry trace export; per-consumer cost dashboard

### Should-Have (v1.1)

- **ML-based behavioral anomaly detection**: pluggable scoring engine; request feature vectors (inter-arrival timing, sequence entropy, header fingerprint); produces anomaly score per consumer; configurable actions (log, throttle, block); trainable on operator's own traffic data — addresses the largest single gap in the open-source market
- **Adaptive rate limiting based on backend health**: dynamic limit reduction triggered by backend latency percentile increases or error rate spikes; removes need for manual circuit breaker tuning
- **Multi-model LLM routing and provider failover**: route to primary provider; automatically retry on secondary on 429/5xx; support OpenAI, Anthropic, Mistral, Gemini, DeepSeek with common request schema
- **GitOps-native configuration**: entire gateway configuration declarable as YAML/JSON files in source control; PR-based policy review; preview environments per branch
- **Developer portal**: OpenAPI doc hosting; self-service API key issuance; per-key usage analytics; subscription tier management

### Nice-to-Have (Backlog)

- **Semantic prompt cache**: vector-similarity-based cache for LLM responses; configurable similarity threshold; pluggable embedding model; reduces repeated LLM cost for paraphrased prompts
- **AI agent traffic classification**: behaviour-sequence classifier distinguishing legitimate AI agent pipelines from bot traffic; reduces false positives from high-volume automation
- **Unified control plane for heterogeneous gateways**: policy synchronisation layer that deploys rate limiting and abuse detection rules to Kong, NGINX, and APISIX clusters simultaneously; addresses the 31%-of-orgs-run-multiple-gateways pain point
- **Usage-based billing integration**: aggregate token consumption per consumer into billing events; Stripe webhook output; monthly invoice generation — leverages token metering already in MVP
- **Per-consumer anomaly explainability**: SHAP/LIME-style feature attribution on anomaly decisions; surfaces "why was this consumer flagged" for operator trust; addresses the XAI gap identified in ACM 2024 research
