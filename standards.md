# Standards & API Reference

> Project: API Gateway with AI Rate Limiting · Generated: 2026-05-03

## Industry Standards & Specifications

### HTTP Status Codes & Rate Limiting

- **RFC 6585 (Additional HTTP Status Codes)** — Defines HTTP 429 "Too Many Requests" status code for rate limiting responses. Includes optional Retry-After header for client backoff guidance. Enables clients to distinguish rate limiting from other errors (403 Forbidden, 503 Service Unavailable). [https://datatracker.ietf.org/doc/html/rfc6585](https://datatracker.ietf.org/doc/html/rfc6585)

### API Authentication & Authorization Standards

- **OAuth 2.0 (RFC 6749)** — IETF standard authorization framework enabling identity-aware rate limiting per user, tenant, or API consumer. Supports delegation without exposing credentials. Critical for multi-tenant rate limit enforcement and cost attribution per principal. [https://datatracker.ietf.org/doc/html/rfc6749](https://datatracker.ietf.org/doc/html/rfc6749)

- **OAuth 2.0 Bearer Token Usage (RFC 6750)** — Defines how to transmit bearer tokens in HTTP requests, enabling token-based rate limit tracking and cost attribution per access token.

- **OpenID Connect 1.0** — Identity layer on top of OAuth 2.0, enabling federated authentication for API gateway access control and tenant isolation in multi-tenant deployments.

### API Specification & Security

- **OpenAPI 3.1.0** — RESTful API specification defining security schemes (API Key, HTTP Auth, OAuth 2.0, OpenID Connect, mTLS) and rate limiting headers. Essential for machine-readable API contracts describing authentication and resource consumption constraints. [https://spec.openapis.org/oas/v3.1.0.html](https://spec.openapis.org/oas/v3.1.0.html)

- **Security Schemes (OpenAPI 3.1)** — Specification for defining API authentication methods (HTTP Basic, Bearer tokens, API keys, OAuth 2.0) and their applicability to endpoints. Enables gateway to enforce authentication-based rate limit policies. [https://learn.openapis.org/specification/security.html](https://learn.openapis.org/specification/security.html)

### Error Handling Standards

- **RFC 9457 (Problem Details for HTTP APIs)** — IETF standard for structured error responses including rate limit details, retry guidance, and trace information. Provides machine-readable error format (application/problem+json) with extensible fields for custom rate limit metadata. Successor to RFC 7807. [https://www.rfc-editor.org/rfc/rfc9457.html](https://www.rfc-editor.org/rfc/rfc9457.html)

### Security Frameworks

- **OWASP API Security Top 10 (2023) – API4: Unrestricted Resource Consumption** — Identifies resource consumption limits and rate limiting as critical defenses against DoS attacks and abuse. Categorizes threats including high-volume request flooding, parameter manipulation for resource exhaustion, and cost-draining attacks. Directly relevant for behavioral abuse detection and adaptive limiting strategies. [https://owasp.org/API-Security/editions/2023/en/0xa4-unrestricted-resource-consumption/](https://owasp.org/API-Security/editions/2023/en/0xa4-unrestricted-resource-consumption/)

## Similar Products — Developer Documentation & APIs

### Apache APISIX

- **Description:** Top-level Apache Foundation open-source API gateway built on OpenResty with native plugin ecosystem. Achieves 23,000 QPS per core with 0.2ms average latency. Includes ai-rate-limiting plugin with token-aware support (no cost attribution). 100% Apache 2.0 licensed with no open-core model.
- **API Documentation:** [https://apisix.apache.org/docs/](https://apisix.apache.org/docs/)
- **SDKs/Libraries:** Go, Lua, JavaScript plugins; CLI tools; Docker/Kubernetes support
- **Developer Guide:** [https://apisix.apache.org/learning-center/](https://apisix.apache.org/learning-center/)
- **Standards:** REST API with etcd sync, OpenAPI compatibility, OAuth 2.0 plugins, native Kubernetes CRD support
- **Authentication:** API Key, OAuth 2.0, JWT, HTTP Basic
- **Key Differentiators:** Best-in-class performance (23k QPS); full open-source; token-aware rate limiting plugin; no cost tracking; no behavioral abuse detection; no unified control plane for multi-gateway deployments

### Kong (Kong Gateway)

- **Description:** Open-source (AGPL) + commercial API gateway with extensive plugin ecosystem. Core gateway is free; OIDC, canary releases, and admin dashboard require commercial license. Built on NGINX, widely deployed in enterprises.
- **API Documentation:** [https://docs.konghq.com/](https://docs.konghq.com/)
- **SDKs/Libraries:** Go, Python, Node.js plugins; extensive plugin marketplace; Docker/Kubernetes support
- **Developer Guide:** [https://docs.konghq.com/gateway/latest/](https://docs.konghq.com/gateway/latest/)
- **Standards:** REST Admin API, OpenAPI compatible, OAuth 2.0, JWT, mTLS support
- **Authentication:** API Key, OAuth 2.0, JWT, LDAP, OIDC (enterprise)
- **Key Differentiators:** Largest enterprise adoption; extensive plugin ecosystem; open-source core with proprietary enterprise features; no token-aware rate limiting; no ML-based abuse detection; requires commercial license for advanced features

### AWS API Gateway

- **Description:** Fully managed, serverless API gateway integrated with AWS ecosystem. Native support for REST, WebSocket, and HTTP APIs. 10,000 RPS default limit per region (increasable via support). Tight IAM integration for authentication.
- **API Documentation:** [https://docs.aws.amazon.com/apigateway/](https://docs.aws.amazon.com/apigateway/)
- **SDKs/Libraries:** AWS SDKs (Python boto3, JavaScript, Go, Java); CloudFormation/CDK for IaC; Lambda integration
- **Developer Guide:** [https://docs.aws.amazon.com/apigateway/latest/developerguide/](https://docs.aws.amazon.com/apigateway/latest/developerguide/)
- **Standards:** AWS-native REST/HTTP APIs, JWT validation, mutual TLS (mTLS)
- **Authentication:** IAM roles, API keys, Lambda authorizers, Cognito, OIDC
- **Key Differentiators:** Fully managed (no ops overhead); serverless scaling; tight AWS integration (but vendor lock-in); no multi-cloud support; limited rate limiting options; no behavioral abuse detection; no unified cross-gateway control plane

### Cloudflare API Gateway

- **Description:** Cloud-based API gateway emphasizing edge computing, DDoS protection, and bot detection. Includes bot score for abuse detection (not ML-based behavioral analysis). Request-based rate limiting only (no token awareness).
- **API Documentation:** [https://developers.cloudflare.com/api-shield/](https://developers.cloudflare.com/api-shield/)
- **SDKs/Libraries:** Cloudflare Workers for custom logic; REST API for management
- **Developer Guide:** [https://developers.cloudflare.com/api-shield/getting-started/](https://developers.cloudflare.com/api-shield/getting-started/)
- **Standards:** REST API, JWT validation, mTLS, OpenAPI import (beta)
- **Authentication:** mTLS, API key, OpenID Connect
- **Key Differentiators:** Cloudflare-centric (vendor lock-in); bot score-based abuse detection (not behavioral analysis); edge computing integration; no token-based rate limiting; no multi-gateway control plane

### Apigee (Google Cloud)

- **Description:** Enterprise API management platform (SaaS only, Google-cloud-only). Includes Advanced API Security with ML-powered abuse detection (proprietary). Request-based and quota-based rate limiting. Industry leader in abuse detection but not open-source.
- **API Documentation:** [https://cloud.google.com/apigee/docs/](https://cloud.google.com/apigee/docs/)
- **SDKs/Libraries:** Java, JavaScript, Node.js; Google Cloud SDK integration
- **Developer Guide:** [https://cloud.google.com/apigee/docs/api-platform](https://cloud.google.com/apigee/docs/api-platform)
- **Standards:** REST API, OAuth 2.0, JWT, mTLS, OpenID Connect
- **Authentication:** OAuth 2.0, JWT, API keys, mTLS, SAML
- **Key Differentiators:** Industry-leading ML abuse detection (but proprietary); SaaS only (no self-hosted); Google Cloud lock-in; most expensive option; high barrier to entry; no open-source alternative; no token-aware rate limiting

### Traefik

- **Description:** Modern, lightweight reverse proxy and API gateway emphasizing Kubernetes-native architecture. Built in Go. Smaller footprint than Kong/APISIX but less feature-rich. MIT licensed.
- **API Documentation:** [https://doc.traefik.io/traefik/](https://doc.traefik.io/traefik/)
- **SDKs/Libraries:** Go plugins, Docker/Kubernetes native; HTTP middleware ecosystem
- **Developer Guide:** [https://doc.traefik.io/traefik/getting-started/](https://doc.traefik.io/traefik/getting-started/)
- **Standards:** REST API, Kubernetes CRD, OpenAPI compatible
- **Authentication:** Basic auth, forward auth, JWT middleware available
- **Key Differentiators:** Lightweight, Kubernetes-first design; simpler than Kong/APISIX but less enterprise-feature-rich; no rate limiting out-of-box (via plugins); no abuse detection; no unified control plane

### Tyk (Open Source + Commercial)

- **Description:** Open-source (MPL) API gateway with commercial cloud offering. Golang-based, lightweight. Includes rate limiting, JWT, and OAuth 2.0 support. Smaller community than Kong/APISIX.
- **API Documentation:** [https://tyk.io/docs/](https://tyk.io/docs/)
- **SDKs/Libraries:** Go plugins; JavaScript middleware; multiple language support
- **Developer Guide:** [https://tyk.io/docs/](https://tyk.io/docs/)
- **Standards:** REST API, OpenAPI import, OAuth 2.0, JWT
- **Authentication:** API Key, OAuth 2.0, JWT, LDAP
- **Key Differentiators:** Open-source alternative with commercial SaaS option; built-in rate limiting; simpler than Kong; smaller ecosystem; no ML-based abuse detection; no unified multi-gateway control plane

### AWS WAF + API Gateway

- **Description:** AWS Web Application Firewall combined with API Gateway for DDoS protection and bot detection. Managed service with pay-as-you-go pricing. AWS-native only.
- **API Documentation:** [https://docs.aws.amazon.com/waf/](https://docs.aws.amazon.com/waf/)
- **SDKs/Libraries:** AWS SDKs; CloudFormation/CDK integration
- **Developer Guide:** [https://docs.aws.amazon.com/waf/latest/developerguide/](https://docs.aws.amazon.com/waf/latest/developerguide/)
- **Standards:** AWS-native, IP reputation, bot detection rules
- **Authentication:** IAM-based rule management
- **Key Differentiators:** AWS-native DDoS and bot protection; bot score-based abuse detection; request-based rate limiting only; no token awareness; vendor lock-in; no unified cross-cloud control plane

### Envoy Proxy + AI Gateway Extensions

- **Description:** High-performance reverse proxy (CNCF/open-source) with emerging AI Gateway extensions for token-based rate limiting and LLM traffic management. Built in C++, low latency.
- **API Documentation:** [https://www.envoyproxy.io/docs/envoy/latest/](https://www.envoyproxy.io/docs/envoy/latest/)
- **SDKs/Libraries:** C++ extensions, WebAssembly (WASM) filters, Go integration
- **Developer Guide:** [https://www.envoyproxy.io/docs/envoy/latest/intro/getting_help](https://www.envoyproxy.io/docs/envoy/latest/intro/getting_help)
- **Standards:** REST control plane API, gRPC, WASM extension support, Kubernetes CRD
- **Authentication:** Custom WASM filters for auth; mTLS support
- **Key Differentiators:** Highest performance (ultra-low latency); emerging AI Gateway extensions for token-based rate limiting; WASM for custom logic; smaller API gateway ecosystem than Kong; no built-in ML abuse detection; requires sophisticated configuration

## Notes

### Standards Evolution for AI-Era APIs

1. **Token-Based Metrics Over Request Counts**: RFC 6585 (HTTP 429) was designed for request-per-second rate limiting. LLM APIs require token-based tracking (prompt + completion tokens) for cost-aligned limits. No standard yet exists; this is a critical gap.

2. **OAuth 2.0 for Multi-Tenant Cost Attribution**: RFC 6749 enables per-principal rate limits, but traditional implementations assume uniform request cost. Token-weighted rate limits require gateway-specific extensions.

3. **ML-Based Abuse Detection Standardization Needed**: Apigee Sense is proprietary and Google-cloud-only. No IETF/W3C standard exists for ML-powered behavioral abuse detection. This platform's pluggable ML engine could establish an open standard.

4. **RFC 9457 for Rate Limit Responses**: Structured error format enables clients to programmatically understand rate limit status, retry guidance, and cost impact. Critical for AI APIs where cost budgets matter.

5. **Unified Multi-Gateway Control Plane Gap**: 31% of organizations run multiple API gateways (Kong + NGINX + APISIX). No open standard exists for unified policy synchronization, cost aggregation, or anomaly detection across heterogeneous deployments. This is a significant gap.

### Competitive Positioning

- **vs. APISIX**: APISIX has token-aware plugin but no cost tracking, behavioral abuse detection, or multi-gateway control plane. This platform adds all three.
- **vs. Kong**: Kong dominates enterprise; lacks token-aware limiting, ML abuse detection, and unified control. This platform targets the AI/multi-gateway use case Kong doesn't serve.
- **vs. AWS API Gateway**: Proprietary and AWS-locked. This platform provides open-source, multi-cloud alternative with better LLM support.
- **vs. Apigee**: Apigee leads on ML abuse detection (proprietary, Google-cloud-only, expensive). This platform provides open-source alternative with pluggable ML engine.
- **vs. Cloudflare AI Gateway**: Cloudflare is fast but request-based only. This platform adds token-aware limiting and behavioral (not just bot score) abuse detection.

### Architecture Alignment Points

1. **Rate Limiting**: Implement token-aware limits using OpenAPI provider schemas (OpenAI, Anthropic, Mistral unification)
2. **Abuse Detection**: Pluggable ML scoring engine trainable on operator's traffic; Random Forest / SVM for 98% accuracy
3. **Adaptive Limiting**: Backend health signals (latency, error rate, queue depth) trigger dynamic limit reduction
4. **Multi-Gateway Control**: Unified policy sync (etcd or Consul) for Kong + NGINX + APISIX deployments
5. **Cost Attribution**: Per-token pricing integration with billing systems; RFC 9457 structured responses for cost feedback
