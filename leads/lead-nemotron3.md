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
