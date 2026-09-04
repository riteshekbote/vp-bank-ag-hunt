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
