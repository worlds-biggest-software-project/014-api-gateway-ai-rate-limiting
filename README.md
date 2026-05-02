# API Gateway with AI Rate Limiting

Cloud-native API gateway with token-aware LLM rate limiting, behavioral abuse detection, and adaptive traffic management — purpose-built for the AI API economy.

## The Problem

API gateways exist. But the AI era broke them:

- **LLM APIs are priced by tokens, not requests** — traditional request-count rate limiting doesn't align with cost
- **Behavioral abuse detection is production-proven but not open-source** — Apigee Sense is the only implementation (proprietary, Google-cloud-only)
- **31% of organizations run multiple API gateways** — Kong, NGINX, AWS API Gateway each siloed with their own policies
- **No unified control plane** for heterogeneous gateway deployments

Current solutions optimized for HTTP APIs. The market gap: an open-source gateway that:

1. Limits by token count (not request count) for LLM providers
2. Includes ML behavioral abuse detection (research shows 95–98% accuracy)
3. Adapts limits dynamically based on backend health
4. Provides unified policy management across Kong + NGINX + APISIX

## What This Does

### Token-Aware LLM Rate Limiting
- **Limit by `prompt_tokens`, `completion_tokens`, or `total_tokens`** per consumer, model, and time window
- **Multi-provider support** — OpenAI, Anthropic, Mistral, Gemini, DeepSeek unified under single schema
- **Cost attribution per consumer** — track USD spend per API key, tenant, or team
- **Exposure tracking** — `X-RateLimit-*` headers inform clients of remaining token budget
- **Apache APISIX's `ai-rate-limiting` plugin as the foundation** (Apache-2.0, token-aware but without cost attribution)

### Behavioral Abuse Detection (ML/AI-Native)
- **Request feature vectors** — inter-arrival timing, sequence entropy, header fingerprints, TLS JA3
- **Pluggable scoring engine** — trainable on operator's own traffic data
- **Distinguishes bots from legitimate high-volume automation** — semantic coherence analysis
- **Research validated**: Random Forest + SVM achieve ~98% accuracy (Springer Nature 2023–2025)
- **Zero production implementations in open source** — Apigee Sense is the only tool; this is the largest single gap

### Adaptive Rate Limiting
- **Dynamic limit reduction** triggered by backend latency or error rate spikes
- **Removes manual circuit breaker configuration** — AI-assisted feedback loops
- **Predicts SLO breaches** 5–30 minutes ahead from early signals (memory growth, queue depth)
- **Proactive mitigation** — rotate pods, shed load, page on-call before user impact

### Unified Control Plane
- **Policy synchronization** across Kong, NGINX, APISIX, AWS API Gateway
- **Cost attribution aggregated** from all gateways
- **Anomaly detection** across heterogeneous deployments
- **Single source of truth** for rate limiting and abuse detection policies

## Key Differentiators

| Feature | This Platform | Kong | NGINX | APISIX | Azure APIM | Cloudflare AI GW |
|---------|---|---|---|---|---|---|
| **Token-aware LLM limiting** | ✓ (cost tracking) | Enterprise only | — | ✓ (plugin, no cost) | ✓ (token-limit policy) | (Request-based) |
| **ML behavioral detection** | ✓ (Pluggable) | — | (via F5 AI GW) | — | — | (Bot score only) |
| **Adaptive limiting** | ✓ (AI-assisted) | — | — | — | — | — |
| **Unified control plane** | ✓ | — | — | — | — | — |
| **Open source** | ✓ | ✓ (OSS core) | ✓ | ✓ | — | — |
| **Multi-cloud routing** | ✓ | ✓ | ✓ | ✓ | ✓ (Azure-native) | (Cloudflare-only) |

## Market & Opportunity

- **API Management market**: $6–9B (2025) → $16.93B by 2029
- **Largest gap**: Behavioral abuse detection is 95–98% accurate in research; zero OSS implementations exist
- **LLM pricing alignment**: No major open-source gateway has unified token-based metering with cost attribution
- **Buyers**: Platform teams running heterogeneous gateways (31% of organizations), AI/ML companies, startups

## Research Foundation

- **ML-based API abuse detection achieves 95–98% accuracy** on traffic feature vectors (IEEE, Springer 2023–2025)
- **Apigee Sense is the only production implementation** — proprietary, Google-cloud-only, expensive
- **Traditional rate limiting insufficient against low-and-slow attacks** — behavioral analysis required (Softup.io 2026)
- **Few-shot learning** can detect zero-day attack patterns with minimal training examples (arXiv:2405.11247, 2024)

## Quick Start

```bash
# Configure token-aware rate limiting for OpenAI
gateway.yaml:
  routes:
    - path: /v1/chat/completions
      rate_limit:
        provider: openai
        limits:
          - window: 1m
            tokens: 90000  # 90k tokens per minute
            cost_budget: $50

# Enable behavioral abuse detection
      abuse_detection:
        enabled: true
        feature_vectors: [inter_arrival_time, sequence_entropy, tls_ja3]
        model: random_forest  # trainable on your traffic
        threshold: 0.75

# Adaptive limiting triggered by backend signals
      adaptive:
        enabled: true
        breach_prediction: true  # predict SLO breaches 5-30min ahead
```

## Target Users

1. **Platform Engineering Teams** — managing Kong + NGINX + APISIX heterogeneous deployments
2. **AI/ML Companies** — need token-based limiting and cost tracking for LLM APIs
3. **Startups** — zero licensing cost, cloud-efficient scaling
4. **Enterprises** — behavioral abuse detection for business logic attack prevention
5. **FinOps Teams** — cloud database cost attribution and optimization

## Related Standards

- OpenAPI / Swagger 3.1.x — REST API description standard
- OAuth 2.0 (RFC 6749) + OpenID Connect 1.0 — identity-aware rate limiting per user/tenant
- RFC 6585 — HTTP 429 Too Many Requests status code
- RFC 9457 — Problem Details for HTTP APIs (structured error responses)
- OWASP API Security Top 10 (2023) — API4: Unrestricted Resource Consumption

---

Built on academic research in ML-based API abuse detection and production learnings from Apigee, Kong, and APISIX. [Read the full research](./research.md) | [Feature roadmap](./features.md)
