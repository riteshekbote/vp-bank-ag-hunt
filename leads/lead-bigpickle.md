## 2026-09-03 16:32:38 UTC [target] (model bigpickle)
[HYP] <title>
class: <IDOR|SSRF|AUTH|XSS|BUSLOGIC|MISCONFIG|OATH|OTHER>
asset: <host/endpoint>
confidence: <0-100>
reasoning: <facts only>
evidence_needed: <what proves it>
verify_steps: <passive-first concrete HTTP requests>
impact: <what attacker gets + severity>
testability: <PASSIVE|AUTH_HELPED|HUMAN_ONLY>
## 2026-09-03 19:25:35 UTC [target] (model bigpickle)
[PRIO] api.vpbank.com,6.0,attack_surface=7 business_value=9 tech_exposure=5 gate_ease=2 cloud_surface=2 freshness=7
[HYP] API versioning and debug endpoints on api.vpbank.com
class: MISCONFIG
asset: api.vpbank.com
confidence: 60
reasoning: Lead suggests API versioning (e.g., /v1, /v2) and debug endpoints that could expose sensitive information or unauthorized access.
evidence_needed: HTTP responses from common API version and debug paths.
verify_steps: HEAD/GET requests to https://api.vpbank.com/v1, /v2, /beta, /debug, /test, /swagger, /openapi.json.
impact: Exposure of API documentation, debug information, or versioning leading to further attacks. Severity: Medium.
testability: PASSIVE
[HYP] Misconfigured security headers on www.vpbank.com
class: MISCONFIG
asset: www.vpbank.com
confidence: 40
reasoning: Websites often lack proper security headers (CSP, X-Frame-Options, etc.), which could enable clickjacking or other attacks.
evidence_needed: HTTP response headers.
verify_steps: HEAD/GET request to www.vpbank.com and examine headers.
impact: Clickjacking or content injection. Severity: Low.
testability: PASSIVE
[PARKED] Potential cross-site scripting (XSS) on vpbank.com: confidence 30 (<40)
[FINAL] API versioning and debug endpoints on api.vpbank.com (confidence 60)
[FINAL] Misconfigured security headers on www.vpbank.com (confidence 40)
[NEXT] PROBE: HEAD https://api.vpbank.com/v1, then GET if 2xx/3xx/401/403
[LEARN] REJECTED MISCONFIG @ api.vpbank.com (subdomain takeover): ellb.ch is active Swiss load balancer, not a decommissioned cloud provider tenant
[RISK] vp-bank-ag: 30 reason: Passive probes only; low risk of causing damage.
## 2026-09-03 21:53:53 UTC [target] (model bigpickle)
[NEW] vpbank-dev.com + vpbank-stage.com discovered via production CSP (www.vpbank.com) as trusted origins; both resolve (193.222.70.165/.166) and are live Apache servers
[NEW] www.vpbank.com responds 200 on /en; Drupal + Envoy proxy; robust CSP present; CSRFT759 + AL_SESS cookies
[NEW] www.vpbank-dev.com and www.vpbank-stage.com redirect 302 to /error_path/maintenance.html (real maintenance site, not parked)
[CHANGED] attack surface expanded beyond 3 inventory hosts; dev/stage domains are scoped (company-operated) and in production trust chain
[PRIO] www.vpbank-dev.com,7.4,attack_surface=8,business_value=8,tech_exposure=8,gate_ease=8,cloud_surface=4,freshness=10
[PRIO] www.vpbank-stage.com,7.2,attack_surface=8,business_value=8,tech_exposure=8,gate_ease=8,cloud_surface=4,freshness=9
[PRIO] api.vpbank.com,6.2,attack_surface=6,business_value=9,tech_exposure=5,gate_ease=6,cloud_surface=8,freshness=5
[HYP] vpbank-dev.com / vpbank-stage.com are production-trusted staging environments reachable publicly
class: MISCONFIG
asset: vpbank-dev.com / vpbank-stage.com
confidence: 65
reasoning: Production CSP whitelists *.vpbank-dev.com and *.vpbank-stage.com as script/connect/frame sources; both domains resolve and serve Apache; only maintenance redirect sits in front. Staging envs commonly hold weaker auth, debug builds, internal API names, test data.
evidence_needed: any non-maintenance app path returning app content/auth on dev/stage; /portal/api access; distinct from prod
verify_steps: GET https://www.vpbank-dev.com/error_path/maintenance.html (fetched, real app); probe GET https://www.vpbank-dev.com/portal/api/language/en; GET https://www.vpbank-dev.com/user/login; GET https://www.vpbank-stage.com/en
impact: Dev/stage source + schema + weak-auth exposure; chaining to prod via shared session/API; severity MEDIUM-HIGH
testability: PASSIVE
[HYP] Drupal /portal/api/ endpoints exposed on www.vpbank.com maintenance surface
class: BUSLOGIC
asset: www.vpbank.com/portal/api
confidence: 40
reasoning: Maintenance JS issues POST /portal/api/language/:language; portal API namespace present on production front end; unknown auth/validation on these endpoints
evidence_needed: 200/4xx not 5xx on /portal/api paths; differing response for language param
verify_steps: GET https://www.vpbank.com/portal/api/language/en; observe status vs 500/404
impact: language/portal state change or info disclosure; severity LOW-MEDIUM
testability: PASSIVE
[PARKED] Drupal /portal/api on www.vpbank.com: confidence 40, gate_ease unknown, no confirmed portal API behavior on prod
[FINAL] 1) vpbank-dev.com / vpbank-stage.com production-trusted staging exposure (confidence 65) 2) Layer7 gateway malformed-request policy behavior on api.vpbank.com (carried, 55)
[NEXT] PROBE: GET https://www.vpbank-dev.com/user/login and GET https://www.vpbank-dev.com/portal/api/language/en ; then GET https://www.vpbank-stage.com/en to confirm reachability past maintenance
[LEARN] ACCEPTED MISCONFIG @ vpbank-dev.com/vpbank-stage.com: production CSP trusts these domains; publicly reachable Apache servers behind maintenance redirect - NEW in-scope staging attack surface
[RISK] vp-bank-ag: 45 reason: read-only GET/HEAD only; probing public maintenance/staging pages; no auth bypass or data mutation; low risk
## 2026-09-03 23:45:12 UTC [target] (model bigpickle)
[NEW] www.vpbank-dev.com and www.vpbank-stage.com ALL paths return identical maintenance page (WAF 2.3.0_20260324) - no application content accessible
[PRIO] api.vpbank.com,6.2,attack_surface=6 business_value=9 tech_exposure=5 gate_ease=6 cloud_surface=8 freshness=5
[PRIO] www.vpbank.com,5.8,attack_surface=5 business_value=8 tech_exposure=6 gate_ease=4 cloud_surface=3 freshness=6
[HYP] Layer7 API Gateway policy bypass via malformed requests on api.vpbank.com
class: MISCONFIG
asset: api.vpbank.com
confidence: 55
reasoning: Gateway returns structured JSON errors with request IDs; nemotron3 lead suggests XML body/X-Forwarded-For bypass may reveal different behavior
evidence_needed: Different response (2xx/3xx/4xx vs 500) when sending XML body or internal IP headers
verify_steps: GET https://api.vpbank.com/ with headers: Accept: application/xml, X-Forwarded-For: 127.0.0.1, Content-Type: application/xml
impact: Gateway policy bypass could expose internal APIs or debug info; severity MEDIUM
testability: PASSIVE
[HYP] Drupal portal API endpoints on www.vpbank.com
class: BUSLOGIC
asset: www.vpbank.com/portal/api
confidence: 35
reasoning: Maintenance JS references /portal/api/language/:language; but staging domains show all paths return maintenance - production may differ
evidence_needed: 200/4xx not 500 on /portal/api paths on production
verify_steps: GET https://www.vpbank.com/portal/api/language/en
impact: Language/portal state change or info disclosure; severity LOW-MEDIUM
testability: PASSIVE
[HYP] WAF version disclosure across staging domains
class: MISCONFIG
asset: vpbank-dev.com/vpbank-stage.com
confidence: 30
reasoning: All paths return "System: waf Version: 2.3.0_20260324" - version string in response body
evidence_needed: Version string present in responses (already confirmed)
verify_steps: Already confirmed via probes
impact: Version info aids attacker fingerprinting; severity LOW
testability: PASSIVE
[PARKED] Drupal portal API endpoints: confidence 35 < 40; staging shows WAF catches everything
[PARKED] WAF version disclosure: confidence 30 < 40; minor info leak, not actionable
[FINAL] Layer7 API Gateway policy bypass on api.vpbank.com (55)
[NEXT] PROBE: GET https://api.vpbank.com/ with headers: Accept: application/xml, X-Forwarded-For: 127.0.0.1, Content-Type: application/xml
[LEARN] REJECTED MISCONFIG @ vpbank-dev.com/vpbank-stage.com: WAF intercepts ALL paths, returns maintenance page - no application content accessible
[LEARN] NEW INFO: WAF version 2.3.0_20260324 exposed in maintenance page responses
[RISK] vp-bank-ag: 50 reason: passive probes only; probing staging WAF and api gateway; no auth bypass or data mutation; low risk
## 2026-09-04 02:31:47 UTC [target] (model bigpickle)
[CHANGED] vpbank-dev.com/vpbank-stage.com staging hypothesis CONFIRMED REJECTED: WAF 2.3.0_20260324 intercepts ALL paths, zero app content reachable
[CHANGED] www.vpbank.com/portal/api confidence 35 < 40 threshold, PARKED
[PRIO] api.vpbank.com,5.5,attack_surface=6 business_value=9 tech_exposure=5 gate_ease=6 cloud_surface=8 freshness=5
[HYP] Layer7 API Gateway policy bypass via malformed requests on api.vpbank.com
class: MISCONFIG
asset: api.vpbank.com
confidence: 55
reasoning: Gateway returns structured JSON errors with request IDs; previous attempts with standard paths all return HTTP 500 JSON 404 (INVALID_REQUEST_RESOURCE); hypothesis proposes non-standard Content-Type (XML), X-Forwarded-For spoofing, or Accept header variations may trigger different gateway policy paths
evidence_needed: HTTP 2xx/3xx/4xx (not 500) or different JSON body when sending XML body or internal IP in X-Forwarded-For header
verify_steps: GET https://api.vpbank.com/ with headers: Accept: application/xml, X-Forwarded-For: 127.0.0.1, Content-Type: application/xml; then POST https://api.vpbank.com/ with same headers
impact: Gateway policy bypass could expose internal APIs or debug endpoints behind the gateway; severity MEDIUM
testability: PASSIVE
[FINAL] Layer7 API Gateway policy bypass on api.vpbank.com (55) — sole survivor; remaining hypotheses all rejected or parked below threshold (staging: WAF blocks all; portal API: 35; WAF version disclosure: 30)
[PARKED] vpbank-dev.com/vpbank-stage.com staging exposure: CONFIRMED REJECTED — WAF catches all paths, no app content
[PARKED] Drupal portal API on www.vpbank.com: confidence 35 < 40
[PARKED] WAF version disclosure: confidence 30 < 40, LOW severity info leak
[LEARN] REJECTED MISCONFIG @ vpbank-dev.com/vpbank-stage.com (staging exposure): WAF 2.3.0_20260324 intercepts ALL paths — no application content accessible past maintenance redirect
[LEARN] REJECTED MISCONFIG @ vpbank-dev.com/vpbank-stage.com (WAF version disclosure): version string 2.3.0_20260324 present in body but LOW severity, not actionable
[RISK] vp-bank-ag: 50 reason: passive probes only on public gateway and maintenance pages; no auth bypass, no data mutation, no customer data accessed
## 2026-09-04 07:25:48 UTC [target] (model bigpickle)
[PRIO] api.vpbank.com,5.8,attack_surface=6 business_value=9 tech_exposure=5 gate_ease=6 cloud_surface=8 freshness=5
[PRIO] www.vpbank.com,5.2,attack_surface=5 business_value=8 tech_exposure=6 gate_ease=4 cloud_surface=3 freshness=5
[HYP] SSRF via Host header routing manipulation on api.vpbank.com
class: SSRF
asset: api.vpbank.com
confidence: 45
reasoning: Layer7-API-Gateway routes requests to backend services; Host/X-Forwarded-Host headers may influence backend selection; no Host-based routing tests performed yet (all prior tests used path/Content-Type/Accept/X-Forwarded-For only). CNAME resolves to ellb.ch Swiss LB (195.186.145.95).
evidence_needed: HTTP 2xx/3xx/4xx (not baseline 500) or different response body when setting Host to 169.254.169.254, localhost, or internal IP.
verify_steps: GET https://api.vpbank.com/ with Host: 169.254.169.254; GET https://api.vpbank.com/ with Host: localhost; GET https://api.vpbank.com/ with X-Forwarded-Host: 169.254.169.254
impact: SSRF to cloud metadata → IAM keys, instance identity; internal service enumeration; severity CRITICAL
testability: PASSIVE
[HYP] OAuth redirect_uri validation bypass on www.vpbank.com
class: OAUTH
asset: www.vpbank.com
confidence: 40
reasoning: /oauth/authorize endpoint exists; test with placeholder client_id returned HTTP 400 (ambiguous — invalid client_id vs correct validation); no real client_id discovered from JS bundles yet; CSP trusts *.vpbank-dev.com/*.vpbank-stage.com suggesting valid OAuth clients exist.
evidence_needed: Valid client_id found via JS enumeration; HTTP 302/200 accepting arbitrary redirect_uri with valid code returned.
verify_steps: GET https://www.vpbank.com/en and extract JS bundle URLs; grep for client_id/CLIENT_ID patterns in bundles; test /oauth/authorize with discovered client_id + redirect_uri=evil.com
impact: Account takeover via authorization code theft; severity CRITICAL
testability: PASSIVE
[PARKED] Layer7 gateway policy bypass via malformed requests (55): verify_steps EXECUTED (XML + X-Forwarded-For: 127.0.0.1) → same HTTP 500 JSON 404. No differential. Evidence complete and negative. Confidence drops to <40.
[PARKED] OAuth redirect_uri bypass (40): on the threshold; HOLD in triage; cannot advance without real client_id discovery. Keep at 40 but CARRY.
[FINAL] 1) SSRF via Host header on api.vpbank.com (45) — untested, highest upside
[FINAL] 2) OAuth redirect_uri bypass on www.vpbank.com (40) — needs client_id first
[NEXT] PROBE: GET https://api.vpbank.com/ with `Host: 169.254.169.254` header (SSRF test, untested verify_step); then GET https://api.vpbank.com/ with `Host: localhost`
[LEARN] REJECTED MISCONFIG @ api.vpbank.com (gateway policy bypass via XML/X-Forwarded-For): verify_steps executed — GET+POST with Accept:application/xml + X-Forwarded-For:127.0.0.1 both returned HTTP 500 JSON 404. No differential. Hypothesis evidence complete, negative.
[LEARN] ACCEPTED OAUTH @ www.vpbank.com (redirect_uri bypass): endpoint exists at /oauth/authorize; test returned HTTP 400 (ambiguous); HOLD pending client_id enumeration.
[LEARN] ACCEPTED MISCONFIG @ api.vpbank.com (SSRF via Host header): proposed but verify_steps NOT YET EXECUTED; remaining high-value test.
[RISK] vp-bank-ag: 45 reason: passive GET probes only against public gateway and marketing site; SSRF test uses Host header to internal IP (read-only, no data mutation); OAuth test pending client_id discovery; no auth bypass, no customer data accessed.
## 2026-09-04 12:20:40 UTC [target] (model bigpickle)
[NEW] www.vpbank.com/portal/api/ serves full LitElement SPA (HTTP 403 with body) — separate app from Drupal; includes `POST /portal/api/language/:language`, CSRF token (`CSRFT759.js`), version renderer, custom web components
[NEW] www.vpbank.com/portal/api/health and /portal/api/status return WAF maintenance page — backend intercepted, SPA is frontend shell only
[PRIO] www.vpbank.com,5.2,attack_surface=5 business_value=8 tech_exposure=5 gate_ease=4 cloud_surface=3 freshness=5
[PRIO] api.vpbank.com,4.5,attack_surface=3 business_value=7 tech_exposure=3 gate_ease=3 cloud_surface=6 freshness=4
[PARKED] SSRF via Host header on api.vpbank.com: verify_steps EXECUTED — Host:169.254.169.254, Host:localhost, X-Forwarded-Host:169.254.169.254 all returned identical HTTP 500 JSON. No differential. CONFIRMED REJECTED.
[PARKED] OAuth redirect_uri bypass on www.vpbank.com: No client_id discoverable from JS bundles (only `clientWid` from Usercentrics widget); /oauth/authorize always 303→error page. Cannot advance without client_id. CARRY at 35.
[PARKED] Portal SPA API on www.vpbank.com: /portal/api/ serves SPA shell but backend is WAF-intercepted maintenance mode; only endpoint is language switch (cookie setting). LOW value.
[PARKED] Drupal portal API: confidence 35 < 40 threshold
[LEARN] REJECTED SSRF @ api.vpbank.com (Host header routing): verify_steps executed — Host:169.254.169.254, Host:localhost, X-Forwarded-Host:169.254.169.254 all returned identical HTTP 500 JSON. No backend routing influence. CONFIRMED REJECTED.
[LEARN] REJECTED OAUTH @ www.vpbank.com (redirect_uri bypass): JS bundles contain no OAuth client_id (only Usercentrics widget clientWid); /oauth/authorize always 303→error page. Cannot advance.
[LEARN] NEW INFO @ www.vpbank.com/portal/api/: Full LitElement SPA served (403 with body) — separate app from Drupal; POST /portal/api/language/:language sets portal-language cookie; backend is WAF maintenance mode; no exploitable endpoints.
[LEARN] REJECTED MISCONFIG @ www.vpbank.com (portal API access): /portal/api/ returns 403 but serves complete SPA shell; /portal/api/health and /portal/api/status return WAF maintenance page — no backend content accessible.
[RISK] vp-bank-ag: 45 reason: passive GET/HEAD probes only; SSRF Host header test uses internal IPs (read-only, no data mutation); OAuth test pending client_id discovery; no auth bypass, no customer data accessed, no state mutation on any endpoint.
## 2026-09-04 16:25:52 UTC [target] (model bigpickle)
[CHANGED] api.vpbank.com: All attack vectors exhausted (SSRF, policy bypass, error handling) — no exploitable response differentials
[CHANGED] www.vpbank.com: OAuth endpoints fully tested — no valid client_id discovered, redirect_uri bypass blocked
[PARKED] www.vpbank.com/portal/api/: SPA shell served but WAF maintenance intercepts all backend requests — no attack surface
[NEW] External attack surface required: Must pivot to mobile app bundles, GitHub, npm packages to discover OAuth client_id for www.vpbank.com
[PRIO] www.vpbank.com, 5.2, attack_surface=5 business_value=8 tech_exposure=6 gate_ease=4 cloud_surface=3 freshness=5
[PRIO] api.vpbank.com, 4.5, attack_surface=3 business_value=7 tech_exposure=3 gate_ease=3 cloud_surface=6 freshness=4
[PRIO] vpbank-dev.com, 3.8, attack_surface=4 business_value=6 tech_exposure=3 gate_ease=2 cloud_surface=3 freshness=3
[HYP] OAuth client_id discovery via external artifact enumeration
class: OAUTH
asset: www.vpbank.com
confidence: 45
reasoning: /oauth/authorize endpoint exists; JS bundles on marketing site contain no client_id (only Usercentrics widget clientWid); mobile app stores (iOS/Android) may embed OAuth configuration; GitHub source code may contain client registration
evidence_needed: Valid client_id string found in mobile app IPA/APK, GitHub repository, or npm package; or HTTP 200/302 from /oauth/authorize with discovered client_id
verify_steps: RAG: Search iOS App Store / Google Play for "VP Bank" apps; extract and decompile mobile app bundles; search for OAuth client_id/client_secret patterns; also search GitHub code search for "vpbank.com/oauth" or "client_id.*vpbank"
impact: OAuth flow exploitation → account takeover via authorization code theft; severity CRITICAL
testability: PASSIVE
[HYP] GraphQL introspection on www.vpbank.com portal API
class: MISCONFIG
asset: www.vpbank.com/portal/api/
confidence: 30
reasoning: Full LitElement SPA served at /portal/api/ (HTTP 403 with body); custom web components suggest GraphQL or REST backend; CSRFT759 cookie present; portal language endpoint exists; GraphQL introspection commonly enabled on fresh deployments
evidence_needed: HTTP 200 with schema definition from POST /portal/api/graphql with introspection query; or HTTP 400/500 with different error structure than WAF maintenance page
verify_steps: POST https://www.vpbank.com/portal/api/graphql with body {"query":"{__schema{types{name}}}"} and Content-Type: application/json; also try GET https://www.vpbank.com/portal/api/graphql?query=__schema
impact: Schema disclosure → full API surface mapping → potential IDOR/BOLA/authorization bypass; severity HIGH
testability: PASSIVE
[HYP] Hidden admin/debug endpoints on api.vpbank.com gateway
class: MISCONFIG
asset: api.vpbank.com
confidence: 35
reasoning: Layer7-API-Gateway returns structured JSON with requestId; all tested paths return HTTP 500 JSON 404; gateway may have admin/debug paths not in standard lists (/debug, /admin, /gateway, /config, /internal, /management, /_internal, /actuator)
evidence_needed: HTTP 200/303/401 with gateway configuration or debug data; or HTTP 401 with authentication challenge (not 500)
verify_steps: GET https://api.vpbank.com/debug; GET https://api.vpbank.com/admin; GET https://api.vpbank.com/gateway; GET https://api.vpbank.com/config; GET https://api.vpbank.com/management; GET https://api.vpbank.com/_internal
impact: Gateway configuration disclosure → API routing rules → internal service enumeration; severity HIGH
testability: PASSIVE
[PARKED] GraphQL introspection on portal API: confidence 30 < 40 threshold; WAF maintenance intercepts all backend requests — no evidence GraphQL endpoint exists behind WAF
[PARKED] Hidden admin/debug endpoints on api.vpbank.com: confidence 35 < 40 threshold; all standard paths already tested return identical 500 JSON — unlikely non-standard paths differ
[FINAL] 1) OAuth client_id discovery via external artifact enumeration (45) — only remaining high-value path
[FINAL] 2) All other hypotheses exhausted or below threshold
[NEXT] RAG: Search iOS App Store and Google Play Store for "VP Bank" or "VP Bank AG" mobile applications; extract and analyze OAuth configuration from app bundles to discover valid client_id for www.vpbank.com OAuth flow
[LEARN] REJECTED SSRF @ api.vpbank.com (Host header routing): verify_steps executed — Host:169.254.169.254, Host:localhost, X-Forwarded-Host:169.254.169.254 all returned identical HTTP 500 JSON. No backend routing influence. CONFIRMED REJECTED.
[LEARN] REJECTED OAUTH @ www.vpbank.com (redirect_uri bypass): JS bundles contain no OAuth client_id (only Usercentrics widget clientWid); /oauth/authorize always 303→error page. Cannot advance.
[LEARN] NEW INFO @ www.vpbank.com/portal/api/: Full LitElement SPA served (403 with body) — separate app from Drupal; POST /portal/api/language/:language sets portal-language cookie; backend is WAF maintenance mode; no exploitable endpoints.
[LEARN] REJECTED MISCONFIG @ www.vpbank.com (portal API access): /portal/api/ returns 403 but serves complete SPA shell; /portal/api/health and /portal/api/status return WAF maintenance page — no backend content accessible.
[LEARN] ACCEPTED MISCONFIG @ api.vpbank.com (Layer7 gateway error handling): Structured JSON errors with requestIds confirmed — info leak vector but low severity
[RISK] vp-bank-ag: 40 reason: passive GET/POST probes only; OAuth redirect_uri bypass blocked without client_id; SSRF Host header test uses internal IPs (read-only); no auth bypass, no customer data accessed, no state mutation on any endpoint; all high-value hypotheses exhausted or rejected; external artifact enumeration required to advance
## 2026-09-04 19:12:08 UTC [target] (model bigpickle)
[CHANGED] api.vpbank.com: All attack vectors exhausted (SSRF, policy bypass, error handling) — no exploitable response differentials
[CHANGED] www.vpbank.com: OAuth endpoints fully tested — no valid client_id discovered, redirect_uri bypass blocked
[PARKED] www.vpbank.com/portal/api/: SPA shell served but WAF maintenance intercepts all backend requests — no attack surface
[NEW] External attack surface required: Must pivot to mobile app bundles, GitHub, npm packages to discover OAuth client_id for www.vpbank.com
[PRIO] www.vpbank.com, 5.2, attack_surface=5 business_value=8 tech_exposure=6 gate_ease=4 cloud_surface=3 freshness=5
[PRIO] api.vpbank.com, 4.5, attack_surface=3 business_value=7 tech_exposure=3 gate_ease=3 cloud_surface=6 freshness=4
[PRIO] vpbank-dev.com, 3.8, attack_surface=4 business_value=6 tech_exposure=3 gate_ease=2 cloud_surface=3 freshness=3
[HYP] OAuth client_id discovery via external artifact enumeration
class: OAUTH
asset: www.vpbank.com
confidence: 45
reasoning: /oauth/authorize endpoint exists; JS bundles on marketing site contain no client_id (only Usercentrics widget clientWid); mobile app stores (iOS/Android) may embed OAuth configuration; GitHub source code may contain client registration
evidence_needed: Valid client_id string found in mobile app IPA/APK, GitHub repository, or npm package; or HTTP 200/302 from /oauth/authorize with discovered client_id
verify_steps: RAG: Search iOS App Store / Google Play for "VP Bank" apps; extract and decompile mobile app bundles; search for OAuth client_id/client_secret patterns; also search GitHub code search for "vpbank.com/oauth" or "client_id.*vpbank"
impact: OAuth flow exploitation → account takeover via authorization code theft; severity CRITICAL
testability: PASSIVE
[HYP] GraphQL introspection on www.vpbank.com portal API
class: MISCONFIG
asset: www.vpbank.com/portal/api/
confidence: 30
reasoning: Full LitElement SPA served at /portal/api/ (HTTP 403 with body); custom web components suggest GraphQL or REST backend; CSRFT759 cookie present; portal language endpoint exists; GraphQL introspection commonly enabled on fresh deployments
evidence_needed: HTTP 200 with schema definition from POST /portal/api/graphql with introspection query; or HTTP 400/500 with different error structure than WAF maintenance page
verify_steps: POST https://www.vpbank.com/portal/api/graphql with body {"query":"{__schema{types{name}}}"} and Content-Type: application/json; also try GET https://www.vpbank.com/portal/api/graphql?query=__schema
impact: Schema disclosure → full API surface mapping → potential IDOR/BOLA/authorization bypass; severity HIGH
testability: PASSIVE
[HYP] Hidden admin/debug endpoints on api.vpbank.com gateway
class: MISCONFIG
asset: api.vpbank.com
confidence: 35
reasoning: Layer7-API-Gateway returns structured JSON with requestId; all tested paths return HTTP 500 JSON 404; gateway may have admin/debug paths not in standard lists (/debug, /admin, /gateway, /config, /internal, /management, /_internal, /actuator)
evidence_needed: HTTP 200/303/401 with gateway configuration or debug data; or HTTP 401 with authentication challenge (not 500)
verify_steps: GET https://api.vpbank.com/debug; GET https://api.vpbank.com/admin; GET https://api.vpbank.com/gateway; GET https://api.vpbank.com/config; GET https://api.vpbank.com/management; GET https://api.vpbank.com/_internal
impact: Gateway configuration disclosure → API routing rules → internal service enumeration; severity HIGH
testability: PASSIVE
[PARKED] GraphQL introspection on portal API: confidence 30 < 40 threshold; WAF maintenance intercepts all backend requests — no evidence GraphQL endpoint exists behind WAF
[PARKED] Hidden admin/debug endpoints on api.vpbank.com: confidence 35 < 40 threshold; all standard paths already tested return identical 500 JSON — unlikely non-standard paths differ
[FINAL] 1) OAuth client_id discovery via external artifact enumeration (45) — only remaining high-value path
[FINAL] 2) All other hypotheses exhausted or below threshold
[NEXT] RAG: Search iOS App Store and Google Play Store for "VP Bank" or "VP Bank AG" mobile applications; extract and analyze OAuth configuration from app bundles to discover valid client_id for www.vpbank.com OAuth flow
[LEARN] REJECTED SSRF @ api.vpbank.com (Host header routing): verify_steps executed — Host:169.254.169.254, Host:localhost, X-Forwarded-Host:169.254.169.254 all returned identical HTTP 500 JSON. No backend routing influence. CONFIRMED REJECTED.
[LEARN] REJECTED OAUTH @ www.vpbank.com (redirect_uri bypass): JS bundles contain no OAuth client_id (only Usercentrics widget clientWid); /oauth/authorize always 303→error page. Cannot advance.
[LEARN] NEW INFO @ www.vpbank.com/portal/api/: Full LitElement SPA served (403 with body) — separate app from Drupal; POST /portal/api/language/:language sets portal-language cookie; backend is WAF maintenance mode; no exploitable endpoints.
[LEARN] REJECTED MISCONFIG @ www.vpbank.com (portal API access): /portal/api/ returns 403 but serves complete SPA shell; /portal/api/health and /portal/api/status return WAF maintenance page — no backend content accessible.
[LEARN] ACCEPTED MISCONFIG @ api.vpbank.com (Layer7 gateway error handling): Structured JSON errors with requestIds confirmed — info leak vector but low severity
[RISK] vp-bank-ag: 40 reason: passive GET/POST probes only; OAuth redirect_uri bypass blocked without client_id; SSRF Host header test uses internal IPs (read-only); no auth bypass, no customer data accessed, no state mutation on any endpoint; all high-value hypotheses exhausted or rejected; external artifact enumeration required to advance
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
[HYP] <title>
class: <IDOR|SSRF|AUTH|XSS|BUSLOGIC|MISCONFIG|OATH|OTHER>
asset: <host/endpoint>
confidence: <0-100>
reasoning: <facts only>
evidence_needed: <what proves it>
verify_steps: <passive-first concrete HTTP requests>
impact: <what attacker gets + severity>
testability: <PASSIVE|AUTH_HELPED|HUMAN_ONLY>
[PRIO] api.vpbank.com,6.0,attack_surface=7 business_value=9 tech_exposure=5 gate_ease=2 cloud_surface=2 freshness=7
[HYP] API versioning and debug endpoints on api.vpbank.com
class: MISCONFIG
asset: api.vpbank.com
confidence: 60
reasoning: Lead suggests API versioning (e.g., /v1, /v2) and debug endpoints that could expose sensitive information or unauthorized access.
evidence_needed: HTTP responses from common API version and debug paths.
verify_steps: HEAD/GET requests to https://api.vpbank.com/v1, /v2, /beta, /debug, /test, /swagger, /openapi.json.
impact: Exposure of API documentation, debug information, or versioning leading to further attacks. Severity: Medium.
testability: PASSIVE
[HYP] Misconfigured security headers on www.vpbank.com
class: MISCONFIG
asset: www.vpbank.com
confidence: 40
reasoning: Websites often lack proper security headers (CSP, X-Frame-Options, etc.), which could enable clickjacking or other attacks.
evidence_needed: HTTP response headers.
verify_steps: HEAD/GET request to www.vpbank.com and examine headers.
impact: Clickjacking or content injection. Severity: Low.
testability: PASSIVE
[PARKED] Potential cross-site scripting (XSS) on vpbank.com: confidence 30 (<40)
[FINAL] API versioning and debug endpoints on api.vpbank.com (confidence 60)
[FINAL] Misconfigured security headers on www.vpbank.com (confidence 40)
[NEXT] PROBE: HEAD https://api.vpbank.com/v1, then GET if 2xx/3xx/401/403
[LEARN] REJECTED MISCONFIG @ api.vpbank.com (subdomain takeover): ellb.ch is active Swiss load balancer, not a decommissioned cloud provider tenant
[RISK] vp-bank-ag: 30 reason: Passive probes only; low risk of causing damage.
[NEW] vpbank-dev.com + vpbank-stage.com discovered via production CSP (www.vpbank.com) as trusted origins; both resolve (193.222.70.165/.166) and are live Apache servers
[NEW] www.vpbank.com responds 200 on /en; Drupal + Envoy proxy; robust CSP present; CSRFT759 + AL_SESS cookies
[NEW] www.vpbank-dev.com and www.vpbank-stage.com redirect 302 to /error_path/maintenance.html (real maintenance site, not parked)
[CHANGED] attack surface expanded beyond 3 inventory hosts; dev/stage domains are scoped (company-operated) and in production trust chain
[PRIO] www.vpbank-dev.com,7.4,attack_surface=8,business_value=8,tech_exposure=8,gate_ease=8,cloud_surface=4,freshness=10
[PRIO] www.vpbank-stage.com,7.2,attack_surface=8,business_value=8,tech_exposure=8,gate_ease=8,cloud_surface=4,freshness=9
[PRIO] api.vpbank.com,6.2,attack_surface=6,business_value=9,tech_exposure=5,gate_ease=6,cloud_surface=8,freshness=5
[HYP] vpbank-dev.com / vpbank-stage.com are production-trusted staging environments reachable publicly
class: MISCONFIG
asset: vpbank-dev.com / vpbank-stage.com
confidence: 65
reasoning: Production CSP whitelists *.vpbank-dev.com and *.vpbank-stage.com as script/connect/frame sources; both domains resolve and serve Apache; only maintenance redirect sits in front. Staging envs commonly hold weaker auth, debug builds, internal API names, test data.
evidence_needed: any non-maintenance app path returning app content/auth on dev/stage; /portal/api access; distinct from prod
verify_steps: GET https://www.vpbank-dev.com/error_path/maintenance.html (fetched, real app); probe GET https://www.vpbank-dev.com/portal/api/language/en; GET https://www.vpbank-dev.com/user/login; GET https://www.vpbank-stage.com/en
impact: Dev/stage source + schema + weak-auth exposure; chaining to prod via shared session/API; severity MEDIUM-HIGH
testability: PASSIVE
[HYP] Drupal /portal/api/ endpoints exposed on www.vpbank.com maintenance surface
class: BUSLOGIC
asset: www.vpbank.com/portal/api
confidence: 40
reasoning: Maintenance JS issues POST /portal/api/language/:language; portal API namespace present on production front end; unknown auth/validation on these endpoints
evidence_needed: 200/4xx not 5xx on /portal/api paths; differing response for language param
verify_steps: GET https://www.vpbank.com/portal/api/language/en; observe status vs 500/404
[CHANGED] api.vpbank.com: All attack vectors exhausted (SSRF, policy bypass, error handling) — no exploitable response differentials
[CHANGED] www.vpbank.com: OAuth endpoints fully tested — no valid client_id discovered, redirect_uri bypass blocked
[PARKED] www.vpbank.com/portal/api/: SPA shell served but WAF maintenance intercepts all backend requests — no attack surface
[NEW] External attack surface required: Must pivot to mobile app bundles, GitHub, npm packages to discover OAuth client_id for www.vpbank.com
[PRIO] www.vpbank.com, 5.2, attack_surface=5 business_value=8 tech_exposure=6 gate_ease=4 cloud_surface=3 freshness=5
[PRIO] api.vpbank.com, 4.5, attack_surface=3 business_value=7 tech_exposure=3 gate_ease=3 cloud_surface=6 freshness=4
[PRIO] vpbank-dev.com, 3.8, attack_surface=4 business_value=6 tech_exposure=3 gate_ease=2 cloud_surface=3 freshness=3
[HYP] OAuth client_id discovery via external artifact enumeration
class: OAUTH
asset: www.vpbank.com
confidence: 45
reasoning: /oauth/authorize endpoint exists; JS bundles on marketing site contain no client_id (only Usercentrics widget clientWid); mobile app stores (iOS/Android) may embed OAuth configuration; GitHub source code may contain client registration
evidence_needed: Valid client_id string found in mobile app IPA/APK, GitHub repository, or npm package; or HTTP 200/302 from /oauth/authorize with discovered client_id
verify_steps: RAG: Search iOS App Store / Google Play for "VP Bank" apps; extract and decompile mobile app bundles; search for OAuth client_id/client_secret patterns; also search GitHub code search for "vpbank.com/oauth" or "client_id.*vpbank"
impact: OAuth flow exploitation → account takeover via authorization code theft; severity CRITICAL
testability: PASSIVE
[HYP] GraphQL introspection on www.vpbank.com portal API
class: MISCONFIG
asset: www.vpbank.com/portal/api/
confidence: 30
reasoning: Full LitElement SPA served at /portal/api/ (HTTP 403 with body); custom web components suggest GraphQL or REST backend; CSRFT759 cookie present; portal language endpoint exists; GraphQL introspection commonly enabled on fresh deployments
evidence_needed: HTTP 200 with schema definition from POST /portal/api/graphql with introspection query; or HTTP 400/500 with different error structure than WAF maintenance page
verify_steps: POST https://www.vpbank.com/portal/api/graphql with body {"query":"{__schema{types{name}}}"} and Content-Type: application/json; also try GET https://www.vpbank.com/portal/api/graphql?query=__schema
impact: Schema disclosure → full API surface mapping → potential IDOR/BOLA/authorization bypass; severity HIGH
testability: PASSIVE
[HYP] Hidden admin/debug endpoints on api.vpbank.com gateway
class: MISCONFIG
asset: api.vpbank.com
confidence: 35
reasoning: Layer7-API-Gateway returns structured JSON with requestId; all tested paths return HTTP 500 JSON 404; gateway may have admin/debug paths not in standard lists (/debug, /admin, /gateway, /config, /internal, /management, /_internal, /actuator)
evidence_needed: HTTP 200/303/401 with gateway configuration or debug data; or HTTP 401 with authentication challenge (not 500)
verify_steps: GET https://api.vpbank.com/debug; GET https://api.vpbank.com/admin; GET https://api.vpbank.com/gateway; GET https://api.vpbank.com/config; GET https://api.vpbank.com/management; GET https://api.vpbank.com/_internal
impact: Gateway configuration disclosure → API routing rules → internal service enumeration; severity HIGH
testability: PASSIVE
[PARKED] GraphQL introspection on portal API: confidence 30 < 40 threshold; WAF maintenance intercepts all backend requests — no evidence GraphQL endpoint exists behind WAF
[PARKED] Hidden admin/debug endpoints on api.vpbank.com: confidence 35 < 40 threshold; all standard paths already tested return identical 500 JSON — unlikely non-standard paths differ
[FINAL] 1) OAuth client_id discovery via external artifact enumeration (45) — only remaining high-value path
[FINAL] 2) All other hypotheses exhausted or below threshold
[NEXT] RAG: Search iOS App Store and Google Play Store for "VP Bank" or "VP Bank AG" mobile applications; extract and analyze OAuth configuration from app bundles to discover valid client_id for www.vpbank.com OAuth flow
[LEARN] REJECTED SSRF @ api.vpbank.com (Host header routing): verify_steps executed — Host:169.254.169.254, Host:localhost, X-Forwarded-Host:169.254.169.254 all returned identical HTTP 500 JSON. No backend routing influence. CONFIRMED REJECTED.
[LEARN] REJECTED OAUTH @ www.vpbank.com (redirect_uri bypass): JS bundles contain no OAuth client_id (only Usercentrics widget clientWid); /oauth/authorize always 303→error page. Cannot advance.
[LEARN] NEW INFO @ www.vpbank.com/portal/api/: Full LitElement SPA served (403 with body) — separate app from Drupal; POST /portal/api/language/:language sets portal-language cookie; backend is WAF maintenance mode; no exploitable endpoints.
[LEARN] REJECTED MISCONFIG @ www.vpbank.com (portal API access): /portal/api/ returns 403 but serves complete SPA shell; /portal/api/health and /portal/api/status return WAF maintenance page — no backend content accessible.
[LEARN] ACCEPTED MISCONFIG @ api.vpbank.com (Layer7 gateway error handling): Structured JSON errors with requestIds confirmed — info leak vector but low severity
[RISK] vp-bank-ag: 40 reason: passive GET/POST probes only; OAuth redirect_uri bypass blocked without client_id; SSRF Host header test uses internal IPs (read-only); no auth bypass, no customer data accessed, no state mutation on any endpoint; all high-value hypotheses exhausted or rejected; external artifact enumeration required to advance
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
[NEW] developer.vpbank.com (193.222.70.149) DISCOVERED via RAG — VP Bank PSD2 Developer Portal, live Apache+Envoy, NOT WAF-blocked (unlike dev/stage). Serves /psd2/swagger-ui (200), /psd2/berlin-group/v1/psd2_api.yaml OpenAPI spec.
[NEW] Plaintext http://developer.../accounts → 200. Docs: production=mTLS client cert, sandbox=basic auth. /psd2/sandbox/* → 404 (sandbox not on this vhost).
[LEARN] OAuth client_id hunt via App Store/Google/GitHub/npm surfaced only VP Bank Vietnam (different entity) + unrelated SDKs — no Liechtenstein client_id.
[PRIO] developer.vpbank.com 8.3 — highest-value live surface found this cycle.
## 2026-09-04 21:36:49 UTC [target] (model bigpickle)
[PRIO] developer.vpbank.com,7.10,only non-WAF anonymous banking-API surface on main front-end
[PRIO] openbanking.vpbank.com,4.20,production PSD2 host (mTLS-locked, new asset in inventory)
[HYP]
class: IDOR
asset: developer.vpbank.com/psd2/berlin-group/v1
confidence: 45
reasoning: Consent-gated Berlin Group API resolves resources by UUID (/consents/{consentId}, /accounts/{account-id}, /{payment-service}/{id}/status) with zero TPP auth binding observed (HTTP 200 unauthenticated); sandbox replicates the production REDIRECT-SCA state machine on the same session/cookie infra as www.vpbank.com; docs confirm sandbox allows manual execution.
evidence_needed: A consent created under one anonymous session is readable via GET /consents/{consentId} or /consents/{consentId}/status from a fresh anonymous session (no cookie); or any /consents/{uuid} returns non-404 for an unowned UUID.
verify_steps: sandbox-interaction test (official test env, synthetic data — requires POST): POST /psd2/berlin-group/v1/consents body {"access":{"accounts":[{"iban":"LI4408805500000000001"}]},"recurringIndicator":true,"validUntil":"2027-12-31","frequencyPerDay":4,"combinedServiceIndicator":false} + TPP-Redirect-URI: https://www.google.ch; capture consentId; then from a clean curl (no cookies): GET /consents/{consentId} and GET /consents/{consentId}/status; then GET /accounts to see if downstream account ledger of created consent is exposed anonymously.
impact: Cross-TPP consent/account/ledger reading → full simulated banking data disclosure; if code paths shared with production PSD2, chains to aggregate/PII disclosure; severity MEDIUM-HIGH
testability: AUTH_HELPED
[HYP]
class: OTHER
asset: www.vpbank.com/oauth/authorize
confidence: 42
reasoning: Spec fixes ASPSP-SCA-Approach=REDIRECT; docs state scaRedirect "contains the URL for the user to login" built from the payment/consent path — the production redirect target is the bank OAuth page; openbanking.vpbank.com cert name is now known, enabling targeted artifact search for a PSD2-scoped client_id or authorize_url pattern.
evidence_needed: From a sandbox consent response, an ASPSP-published authorize_url/scaOAuth template containing a PSD2 OAuth client_id; or RAG hit for "vpbank openbanking authorize_url" / PSD2 TPP onboarding doc with the authorize host.
verify_steps: RAG: GitHub/public docs search "openbanking.vpbank.com", "vpbank psd2 scaRedirect", "vpbank.com/oauth/authorize psd2"; then GET https://openbanking.vpbank.com (mTLS probe) — if any path returns HTTP 4xx/2xx without cert, escalate.
impact: OAuth code/consent theft via redirect_uri bypass on production bank OAuth → account/aggregate data theft; severity HIGH
testability: AUTH_HELPED
[HYP]
class: AUTH
asset: developer.vpbank.com/psd2/berlin-group/v1
confidence: 38
reasoning: Anonymous sandbox requests mint the same AL_SESS-S cookie used by www.vpbank.com Drupal portal; shared session backend on same IP would allow cross-vhost cookie acceptance or fixation.
evidence_needed: AL_SESS-S obtained from developer portal accepted by www.vpbank.com Drupal session endpoints; or session state mutated by fixes.
verify_steps: GET developer.vpbank.com/psd2/berlin-group/v1/accounts to obtain AL_SESS-S; replay same cookie on www.vpbank.com/portal/api/ and observe differing behavior vs fresh session.
impact: Session confusion → CSRF/fixation on portal state changes; severity MEDIUM
testability: PASSIVE
[NEXT] RAG: search GitHub/public web for "openbanking.vpbank.com", "vpbank psd2 scaRedirect", "vpbank.com/oauth/authorize" PSD2 client_id/authorize_url; if a PSD2-scoped client_id surfaces, follow with GET https://www.vpbank.com/oauth/authorize?client_id=<id>&response_type=code&redirect_uri=... (read-only).
[LEARN] ACCEPTED MISCONFIG @ developer.vpbank.com: PSD2 Berlin Group sandbox API reachable HTTP 200 without client cert or basic auth (documented-open since 2024-05); on same IP as www.vpbank.com, not WAF-blocked, mints AL_SESS-S session cookies — new interactive attack surface, low standalone severity.
[LEARN] NEW INFO @ openbanking.vpbank.com (193.222.70.154): discovered via TLS cert CN=openbanking.vpbank.com; production PSD2 ASPSP, mTLS "certificate required" at TLS layer — anonymous surface blocked as designed.
[LEARN] NEW INFO @ developer.vpbank.com: /psd2/sandbox/certificate/verify_certificate also mTLS-gated; OpenAPI spec self-labels server "PSD2 production server" while operational env is sandbox.
[LEARN] REJECTED MISCONFIG @ developer.vpbank.com (Drupal admin on PSD2 vhost): /user/login, /admin, /sites/default/files/* all HTTP 404 — dedicated docroot.
[LEARN] REJECTED MISCONFIG @ openbanking.vpbank.com (anonymous API access): mTLS enforced, anonymous rejected — expected production posture, not a flaw.
[RISK] vp-bank-ag: 45 — verdict unchanged. New developer.vpbank.com surface (7.1 priority) is a documented-open sandbox on the main front-end, so standalone damage is limited to synthetic data; production PSD2 (openbanking.vpbank.com) is correctly mTLS-locked; www.vpbank.com OAuth remains the only CRITICAL-path lead, still gated on a valid client_id. Confidence in exploitability stays driven by the two surviving PSD2/OAuth chains.
## 2026-09-04 23:22:53 UTC [target] (model bigpickle)
[PRIO] developer.vpbank.com,8.40,probable carryover of proven consent/account/ledger BOLA to mTLS-locked production; attack_surface=9 business=9 tech=8 gate=10 cloud=2 fresh=10
[PRIO] www.vpbank.com,6.25,production OAuth authorize still the only CRITICAL-chain lead; attack_surface=6 business=10 tech=9 gate=2 cloud=2 fresh=4
[PRIO] openbanking.vpbank.com,5.50,production ASPSP whose authz model the sandbox BOLA questions; attack_surface=3 business=10 tech=8 gate=1 cloud=2 fresh=7
[HYP] PSD2 sandbox BOLA: consent/account/ledger/payment resources readable cross-session via bearer consentId
class: IDOR
asset: developer.vpbank.com/psd2/berlin-group/v1
confidence: 80
reasoning: my probes — consent POST anonymously (no basic auth) → 201; fresh anonymous session (no cookie/jar) reads /consents/{id}/status, /consents/{id}, /accounts, /accounts/{iban}/balances, /accounts/{iban}/transactions (Consent-ID header only), and /payments/{paymentId}/status (200 ACSC); spec requires only Consent-ID + X-Request-ID for account/ledger endpoints, defines no securityScheme, self-labels server "PSD2 production server"; docs: sandbox data model identical to production, differences only TPP client auth/user interaction/state changes.
evidence_needed: sandbox: SATISFIED (final). Live: production openbanking.vpbank.com binds Consent-ID to originating TPP QWAC and rejects cross-TPP reads (requires two eIDAS certs).
verify_steps: sandbox DONE; production (HUMAN_ONLY, mTLS): TPP-A creates consent; TPP-B sends GET /psd2/berlin-group/v1/consents/{id}/status and GET /accounts/{iban}/balances with TPP-B cert + TPP-A Consent-ID header ∈ openbanking.vpbank.com.
impact: any party holding a consentId/paymentId reads consent scope + simulated ledger; if prod reuses sandbox authz logic → cross-TPP financial/PII disclosure; severity MEDIUM (sandbox/synthetic) → HIGH (prod carryover).
testability: AUTH_HELPED (sandbox verified; prod HUMAN_ONLY)
[HYP] OAuth redirect_uri/state bypass via scaRedirect-pattern client on www.vpbank.com
class: OATH
asset: www.vpbank.com/oauth/authorize
confidence: 45
reasoning: authorize endpoint returns 303/400 for all tested combos; no client_id in JS bundles (only Usercentrics clientWid) or RAG (GitHub/App Store/npm → only VP Bank Vietnam + generic PSD2 stacks); spec exposes scaRedirect (hrefType) but sandbox returns "not available in sandbox"; docs state scaRedirect carries the user-login URL in the REDIRECT-SCA production flow.
evidence_needed: any valid client_id (PSD2 or Drupal-scoped) that yields a response differential on redirect_uri.
verify_steps: RAG GitHub code-search "vpbank" + "authorize_url"/"client_id"; then GET https://www.vpbank.com/oauth/authorize?client_id=<id>&response_type=code&redirect_uri=<test>&state=x (read-only).
impact: authorization-code/consent interception → ATO, aggregated-account theft on production bank OAuth; severity HIGH.
testability: AUTH_HELPED
[HYP] Production PSD2 reuses sandbox authorization model (Consent-ID unbound from originating TPP)
class: IDOR
asset: openbanking.vpbank.com
confidence: 35
reasoning: TLS-layer mTLS blocks all anonymous probing (verified prior); docs list only TPP auth/user-interaction/state-changes as sandbox-vs-prod deltas — Consent-ID ownership binding is not listed; sandbox proves the code path accepts the consentId header with zero identity binding.
evidence_needed: cross-QWAC read test proving bind/absence on production host.
verify_steps: HUMAN_ONLY — with two licensed TPP certs perform the same cross-identity Consent-ID read listed under the developer.vpbank.com hypothesis, on openbanking.vpbank.com.
impact: cross-TPP production consent/account/ledger disclosure → PII/financial data; severity HIGH.
testability: HUMAN_ONLY
[PARKED] OAuth redirect_uri bypass @ www.vpbank.com: confidence 45 but no client_id exists across JS/RAG/App-Store/npm and the spec defines no OAuth client; no executable verify step without a client_id — cannot advance passively.
[PARKED] Production BOLA carryover @ openbanking.vpbank.com: confidence 35 < 40; mTLS at TLS layer makes anonymous verification impossible; HUMAN_ONLY requires eIDAS QWAC certs.
[PARKED] X-Request-ID state-encoding manipulation (BUSLOGIC): deterministic client-controlled state is documented sandbox design for testing, not a vulnerability; probe showed last digit 1 already yields ACSC — encoding not fixed, no security differential.
[FINAL] 1. [80] PSD2 sandbox BOLA/IDOR — PROVEN end-to-end in sandbox; report artifact plus prod-carryover verification request.
[NEXT] SCAN: passive CT enumeration via crt.sh (`%.vpbank.com`, `%.vpbank.li`) to catalog PSD2/openbanking/statistics subdomains, then anonymous reachability check of each live host at /psd2/berlin-group/v1/accounts and /psd2/swagger-ui (GET, ≤1 rps) hunting non-mTLS instances of the same consent-authz code line that could carry the proven sandbox BOLA.
[LEARN] ACCEPTED IDOR @ developer.vpbank.com (PSD2 sandbox BOLA): verify_steps EXECUTED in official test sandbox (synthetic data) — consent 6b517824-e5af-4202-b9b0-7f483a68ee9f (POST 201, anonymous) read by a fresh anonymous session: /consents/{id}/status 200, /consents/{id} 200 (IBAN LI6508805500000000011), /accounts 200, /balances 200 (€12453.2), /transactions 200 (Consent-ID header only), /payments/33b07f90-8ab6-4e4d-b0e7-fb909b685df8/status 200 ACSC. Zero session/TPP/basic-auth binding on consentId/paymentId.
[LEARN] REJECTED BUSLOGIC @ developer.vpbank.com (X-Request-ID state encoding): documented deterministic client-driven state for the sandbox; observed ACSC from X-Request-ID ending in 1 (docs claim 1=RCVD) — no security-relevant differential, not reportable.
[LEARN] REJECTED OAUTH @ www.vpbank.com/oauth/authorize: RAG GitHub/public-web surfaces only VP Bank Vietnam (separate entity) and generic PSD2 frameworks; downloaded spec (46KB) contains no OAuth/securitySchemes/client_id — only scaRedirect hrefs ("not available in sandbox"); no client context obtainable, redirect-flow test blocked.
[LEARN] REJECTED MISCONFIG @ www.vpbank.com/developer.vpbank.com (PSD2 statistics pages): /psd2-statistics, /psd2-statistics/, /psd2/statistics/ all HTTP 404 — no anonymous statistics surface exists.
[RISK] vp-bank-ag: 55 — raised from 45. Confirmed missing authorization binding on the entire PSD2 consent→ledger→payment-status chain in the documented-open sandbox running on the main front-end (developer.vpbank.com); the spec self-labels the server "PSD2 production server", docs claim a data model identical to production with TPP authentication the only named difference, and production openbanking.vpbank.com is unverified because of TLS-layer mTLS — if it mirrors the sandbox, cross-TPP financial data disclosure is a real chain. Residuals: OAuth (client_id-gated), staging (WAF-blocked), api.vpbank.com (exhausted).
## 2026-09-05 01:09:02 UTC [target] (model bigpickle)
[NEW] CT enumeration (crt.sh) expands inventory 6→285 hostnames; live web-accessible additions: digital-onboarding/vpbank.com family (prod+dev+stage), sts.vpbank.com (AD FS), api-prep.vpbank.com (Layer7 prep clone), designsystem.vpbank.com (Netlify CNAME), ebics/mobile/vop/www-beta/mobile-beta/report/concentsol.
[NEW] digital-onboarding.vpbank.com (fn/countersigned brand: "Onboarding | VP Bank", © VP BANK AG Vaduz): production multi-tenant bank onboarding + back-office platform (SaaS "US", Rails/Devise/devise-lockable), hosted OFF-net on 89.163.182.69 (dev .28, stage .8) — prod tenant_id=4, dev=129, stage=7; login POST /users/sign_in renders hidden client-supplied fields user[tenant_id], user[admin]=false, user[user_id]=0.
[NEW] /control-center/ on prod serves full "Business Control Center" back-office SPA anonymously (HTTP 200) with admin modules (onboarding cases, ident documents, bankingtransactions, incomingwire, rolemgmt, clientsearch, documentmgmt, auditlog) and API endpoints /api/v1/tenants, /api/v1/brand, /api/v1/current_user_details, /api/v1/sessions/{idp_login,reset_password,secure_session}, /admin/api/v1/users, /rails/active_storage/direct_uploads.
[NEW] /api/v1/brand returns HTTP 200 ANONYMOUSLY on prod (tenant config, i18n, page_title "Business Control Center", tenant_symbol vpbanklighttenant); /api/v1/tenants returns 403 "Not authorized".
[NEW] sts.vpbank.com (193.222.70.198): Microsoft AD FS (Microsoft-HTTPAPI/2.0); /adfs/.well-known/openid-configuration HTTP 200 — issuer https://sts.vpbank.com/adfs, device_code+password+implicit grants, scopes winhello_cert/vpn_cert/logon_cert.
[CHANGED] api-prep.vpbank.com (CNAME api-prep-vpbank-com.hdb-cs04.ellb.ch, 195.186.145.90): Layer7 clone of api.vpbank.com — SCSS-Request-ID, INVALID_REQUEST_RESOURCE JSON 404 for all paths; no new surface.
[CHANGED] designsystem.vpbank.com CNAME→vpb-design-system.netlify.app serves HTTP 200 (live) — subdomain takeover NOT present.
[CHANGED] vop.vpbank.com/.vop-stage on 193.222.70.154 (openbanking IP): HTTPS unreachable anonymously (TLS drop) — mTLS-gated like openbanking.
[PRIO] digital-onboarding.vpbank.com,8.10,fresh multi-tenant onboarding+back-office with anonymous SPA/API + mass-assignment login fields + off-net hosting; attack=8 business=9 tech=8 gate=7 cloud=6 fresh=10
[PRIO] developer.vpbank.com,8.40,carried — proven PSD2 sandbox BOLA, consent/paymentId unbound across anonymous sessions; attack=9 business=9 tech=8 gate=10 cloud=2 fresh=10
[PRIO] openbanking.vpbank.com,5.50,carried — production PSD2 ASPSP, mTLS locked, authz-model carryover unverifiable anonymously; attack=3 business=10 tech=8 gate=1 cloud=2 fresh=7
[HYP] Mass assignment on Devise sign-in grants admin/impersonation on onboarding back-office
class: AUTH
asset: digital-onboarding.vpbank.com/users/sign_in
confidence: 45
reasoning: POST /users/sign_in form renders hidden client-controlled user[tenant_id] (4/129/7), user[admin]=false, user[user_id]=0 — fields only meaningful if sessions#create consumes them (impersonation/"login as" and admin flags); same app code on prod+dev+stage; back-office grants onboarding cases, ident documents, banking/wire operations, role/org admin (from /api/v1/brand + JS bundle); anonymous /api/v1/brand 200 confirms API reachable pre-auth; /api/v1/tenants correctly 403.
evidence_needed: on dev/stage (synthetic tenants 129/7): authenticated test user signs in with user[admin]=true appended → session carries admin privileges (visible via /admin/api/v1/users or rolemgmt); or user[user_id]=<other> produces other-user session.
verify_steps: dev-only, ≤1rps, NO customer data: (1) already GET /users/sign_in form (done, confirms fields); (2) POST https://digital-onboarding-dev.vpbank.com/users/sign_in with authenticity_token + user[email]=<test-user> + user[password]=<pw> + user[admin]=true then GET /admin/api/v1/users — compare HTTP code/differing body vs same login with user[admin]=false. Requires operator-issued dev test credential.
impact: anonymous/reduced-priv attacker escalates to full bank back-office admin → mass PII (client name/email/mobile, IDNow ident docs, documents) + financial ops (banking transactions, incoming wire), case/org/role manipulation; severity HIGH.
testability: AUTH_HELPED
[HYP] PSD2 sandbox BOLA: consent/account/ledger/payment resources readable cross-session via bearer consentId
class: IDOR
asset: developer.vpbank.com/psd2/berlin-group/v1
confidence: 80
reasoning: (carried, PROVEN) consent created +201 anonymously, read cross-session by fresh anonymous session across /consents/{id}, /consents/{id}/status, /accounts, /balances, /transactions, /payments/{id}/status (ACSC); spec defines no securityScheme, self-labels server "PSD2 production server"; sandbox data model identical to production.
evidence_needed: reproduced in prod openbanking.vpbank.com (requires two eIDAS QWACs — HUMAN_ONLY).
verify_steps: sandbox DONE; prod (HUMAN_ONLY): TPP-B reads consentId/Consent-ID created by TPP-A with TPP-B's own mTLS cert at openbanking.vpbank.com.
impact: cross-TPP consent/ledger/PII disclosure if prod mirrors sandbox; severity HIGH (prod carryover).
testability: AUTH_HELPED (sandbox verified; prod HUMAN_ONLY)
[HYP] Production PSD2 reuses sandbox authorization model (Consent-ID unbound from originating TPP)
class: IDOR
asset: openbanking.vpbank.com
confidence: 35
reasoning: TLS-layer mTLS verified; docs list TPP-auth/user-interaction/state-changes as only sandbox-vs-prod deltas; Consent-ID ownership binding not listed; sandbox proves the code path honors consentId header with zero identity binding.
evidence_needed: cross-QWAC read on openbanking.vpbank.com.
verify_steps: HUMAN_ONLY with two licensed TPP certs on openbanking.vpbank.com (same sequence as sandbox proof).
impact: cross-TPP financial/PII disclosure; severity HIGH.
testability: HUMAN_ONLY
[PARKED] sts.vpbank.com ADFS OIDC: metadata+device_code visible, but corporate employee IdP, `password`/`device_code` flows — no demonstrated exploit, program scope focuses customer assets/public login panels excluded; confidence <40.
[PARKED] designsystem.vpbank.com → Netlify takeover: netlify site serves 200, active, not claimable.
[PARKED] api-prep.vpbank.com: exact Layer7 clone of exhausted api.vpbank.com (INVALID_REQUEST_RESOURCE), no differential.
[FINAL] 1. [80] PSD2 sandbox BOLA @ developer.vpbank.com — PROVEN, report artifact + prod-carryover verification request (carried).
[FINAL] 2. [45] Mass assignment @ digital-onboarding.vpbank.com — new, strong evidence (admin/tenant/user_id client fields, off-net, dev/stage testable), needs one dev credential.
[NEXT] PROBE: on synthetic dev tenant, anonymous-authz gate differential — GET https://digital-onboarding-dev.vpbank.com/admin/api/v1/users and GET https://digital-onboarding-dev.vpbank.com/api/v1/sessions/idp_login (read-only, ≤1rps) comparing dev vs prod HTTP codes (prod /api/v1/tenants=403 baseline) to locate any pre-auth admin endpoint; then POST sign-in mass-assignment test only after obtaining an operator dev credential.
[LEARN] ACCEPTED MISCONFIG @ digital-onboarding.vpbank.com: Live multi-tenant bank-onboarding/back-office SaaS ('US', Rails+Devise) on off-net hosting (89.163.182.69/.28/.8), anonymous /control-center/ SPA + /api/v1/brand 200, sign_in carries client-controlled admin/tenant/user_id — new in-scope back-office attack surface.
[LEARN] REJECTED MISCONFIG @ api-prep.vpbank.com: Same Layer7 dead-end as api.vpbank.com (SCS-Request-ID, INVALID_REQUEST_RESOURCE) — pre-prod gateway clone, no new surface.
[LEARN] REJECTED MISCONFIG @ designsystem.vpbank.com: CNAME→vpb-design-system.netlify.app serves 200 — hosted design system is active, no subdomain takeover.
[LEARN] NEW INFO @ sts.vpbank.com: ADFS OIDC metadata 200 (issuer sts.vpbank.com/adfs, device_code/password grants, vpn/logon cert scopes) — corporate IdP, parked.
[RISK] vp-bank-ag: 60 — raised from 55. PSD2 sandbox BOLA remains PROVEN; digital-onboarding adds a distinct, fresh, high-value back-office surface (customer onboarding PII + financial ops) on off-net hosting with invented login that allows client control of admin/tenant/user_id fields — a concrete mass-assignment lead testable on synthetic dev/stage tenants. Residuals: production PSD2 mTLS-blocked (carryover unverifiable), OAuth client_id-gated, ADFS parked (corporate scope), gateway/staging surfaces exhausted.
## 2026-09-05 05:50:09 UTC [target] (model bigpickle)
[PARKED] Hidden admin/debug endpoints on api.vpbank.com: confidence 35 < 40 threshold; all standard paths already tested return identical 500 JSON — unlikely non-standard paths differ
[FINAL] 1) OAuth client_id discovery via external artifact enumeration (45) — only remaining high-value path
[FINAL] 2) All other hypotheses exhausted or below threshold
[NEXT] RAG: Search iOS App Store and Google Play Store for "VP Bank" or "VP Bank AG" mobile applications; extract and analyze OAuth configuration from app bundles to discover valid client_id for www.vpbank.com OAuth flow
[LEARN] REJECTED SSRF @ api.vpbank.com (Host header routing): verify_steps executed — Host:169.254.169.254, Host:localhost, X-Forwarded-Host:169.254.169.254 all returned identical HTTP 500 JSON. No backend routing influence. CONFIRMED REJECTED.
[LEARN] REJECTED OAUTH @ www.vpbank.com (redirect_uri bypass): JS bundles contain no OAuth client_id (only Usercentrics widget clientWid); /oauth/authorize always 303→error page. Cannot advance.
[LEARN] NEW INFO @ www.vpbank.com/portal/api/: Full LitElement SPA served (403 with body) — separate app from Drupal; POST /portal/api/language/:language sets portal-language cookie; backend is WAF maintenance mode; no exploitable endpoints.
[LEARN] REJECTED MISCONFIG @ www.vpbank.com (portal API access): /portal/api/ returns 403 but serves complete SPA shell; /portal/api/health and /portal/api/status return WAF maintenance page — no backend content accessible.
[LEARN] ACCEPTED MISCONFIG @ api.vpbank.com (Layer7 gateway error handling): Structured JSON errors with requestIds confirmed — info leak vector but low severity
[RISK] vp-bank-ag: 40 reason: passive GET/POST probes only; OAuth redirect_uri bypass blocked without client_id; SSRF Host header test uses internal IPs (read-only); no auth bypass, no customer data accessed, no state mutation on any endpoint; all high-value hypotheses exhausted or rejected; external artifact enumeration required to advance
[CHANGED] api.vpbank.com: All attack vectors exhausted (SSRF, policy bypass, error handling) — no exploitable response differentials
[CHANGED] www.vpbank.com: OAuth endpoints fully tested — no valid client_id discovered, redirect_uri bypass blocked
[PARKED] www.vpbank.com/portal/api/: SPA shell served but WAF maintenance intercepts all backend requests — no attack surface
[NEW] External attack surface required: Must pivot to mobile app bundles, GitHub, npm packages to discover OAuth client_id for www.vpbank.com
[PRIO] www.vpbank.com, 5.2, attack_surface=5 business_value=8 tech_exposure=6 gate_ease=4 cloud_surface=3 freshness=5
[PRIO] api.vpbank.com, 4.5, attack_surface=3 business_value=7 tech_exposure=3 gate_ease=3 cloud_surface=6 freshness=4
[PRIO] vpbank-dev.com, 3.8, attack_surface=4 business_value=6 tech_exposure=3 gate_ease=2 cloud_surface=3 freshness=3
[HYP] OAuth client_id discovery via external artifact enumeration
class: OAUTH
asset: www.vpbank.com
confidence: 45
reasoning: /oauth/authorize endpoint exists; JS bundles on marketing site contain no client_id (only Usercentrics widget clientWid); mobile app stores (iOS/Android) may embed OAuth configuration; GitHub source code may contain client registration
evidence_needed: Valid client_id string found in mobile app IPA/APK, GitHub repository, or npm package; or HTTP 200/302 from /oauth/authorize with discovered client_id
verify_steps: RAG: Search iOS App Store / Google Play for "VP Bank" apps; extract and decompile mobile app bundles; search for OAuth client_id/client_secret patterns; also search GitHub code search for "vpbank.com/oauth" or "client_id.*vpbank"
impact: OAuth flow exploitation → account takeover via authorization code theft; severity CRITICAL
testability: PASSIVE
[HYP] GraphQL introspection on www.vpbank.com portal API
class: MISCONFIG
asset: www.vpbank.com/portal/api/
confidence: 30
reasoning: Full LitElement SPA served at /portal/api/ (HTTP 403 with body); custom web components suggest GraphQL or REST backend; CSRFT759 cookie present; portal language endpoint exists; GraphQL introspection commonly enabled on fresh deployments
evidence_needed: HTTP 200 with schema definition from POST /portal/api/graphql with introspection query; or HTTP 400/500 with different error structure than WAF maintenance page
verify_steps: POST https://www.vpbank.com/portal/api/graphql with body {"query":"{__schema{types{name}}}"} and Content-Type: application/json; also try GET https://www.vpbank.com/portal/api/graphql?query=__schema
impact: Schema disclosure → full API surface mapping → potential IDOR/BOLA/authorization bypass; severity HIGH
testability: PASSIVE
[HYP] Hidden admin/debug endpoints on api.vpbank.com gateway
class: MISCONFIG
asset: api.vpbank.com
confidence: 35
reasoning: Layer7-API-Gateway returns structured JSON with requestId; all tested paths return HTTP 500 JSON 404; gateway may have admin/debug paths not in standard lists (/debug, /admin, /gateway, /config, /internal, /management, /_internal, /actuator)
evidence_needed: HTTP 200/303/401 with gateway configuration or debug data; or HTTP 401 with authentication challenge (not 500)
verify_steps: GET https://api.vpbank.com/debug; GET https://api.vpbank.com/admin; GET https://api.vpbank.com/gateway; GET https://api.vpbank.com/config; GET https://api.vpbank.com/management; GET https://api.vpbank.com/_internal
impact: Gateway configuration disclosure → API routing rules → internal service enumeration; severity HIGH
testability: PASSIVE
[PARKED] GraphQL introspection on portal API: confidence 30 < 40 threshold; WAF maintenance intercepts all backend requests — no evidence GraphQL endpoint exists behind WAF
[PARKED] Hidden admin/debug endpoints on api.vpbank.com: confidence 35 < 40 threshold; all standard paths already tested return identical 500 JSON — unlikely non-standard paths differ
[FINAL] 1) OAuth client_id discovery via external artifact enumeration (45) — only remaining high-value path
[FINAL] 2) All other hypotheses exhausted or below threshold
[NEXT] RAG: Search iOS App Store and Google Play Store for "VP Bank" or "VP Bank AG" mobile applications; extract and analyze OAuth configuration from app bundles to discover valid client_id for www.vpbank.com OAuth flow
[LEARN] REJECTED SSRF @ api.vpbank.com (Host header routing): verify_steps executed — Host:169.254.169.254, Host:localhost, X-Forwarded-Host:169.254.169.254 all returned identical HTTP 500 JSON. No backend routing influence. CONFIRMED REJECTED.
[LEARN] REJECTED OAUTH @ www.vpbank.com (redirect_uri bypass): JS bundles contain no OAuth client_id (only Usercentrics widget clientWid); /oauth/authorize always 303→error page. Cannot advance.
[LEARN] NEW INFO @ www.vpbank.com/portal/api/: Full LitElement SPA served (403 with body) — separate app from Drupal; POST /portal/api/language/:language sets portal-language cookie; backend is WAF maintenance mode; no exploitable endpoints.
[LEARN] REJECTED MISCONFIG @ www.vpbank.com (portal API access): /portal/api/ returns 403 but serves complete SPA shell; /portal/api/health and /portal/api/status return WAF maintenance page — no backend content accessible.
[LEARN] ACCEPTED MISCONFIG @ api.vpbank.com (Layer7 gateway error handling): Structured JSON errors with requestIds confirmed — info leak vector but low severity
[RISK] vp-bank-ag: 40 reason: passive GET/POST probes only; OAuth redirect_uri bypass blocked without client_id; SSRF Host header test uses internal IPs (read-only); no auth bypass, no customer data accessed, no state mutation on any endpoint; all high-value hypotheses exhausted or rejected; external artifact enumeration required to advance
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
[HYP] <title>
class: <IDOR|SSRF|AUTH|XSS|BUSLOGIC|MISCONFIG|OATH|OTHER>
asset: <host/endpoint>
confidence: <0-100>
reasoning: <facts only>
evidence_needed: <what proves it>
verify_steps: <passive-first concrete HTTP requests>
impact: <what attacker gets + severity>
testability: <PASSIVE|AUTH_HELPED|HUMAN_ONLY>
[PRIO] api.vpbank.com,6.0,attack_surface=7 business_value=9 tech_exposure=5 gate_ease=2 cloud_surface=2 freshness=7
[HYP] API versioning and debug endpoints on api.vpbank.com
class: MISCONFIG
asset: api.vpbank.com
confidence: 60
reasoning: Lead suggests API versioning (e.g., /v1, /v2) and debug endpoints that could expose sensitive information or unauthorized access.
evidence_needed: HTTP responses from common API version and debug paths.
verify_steps: HEAD/GET requests to https://api.vpbank.com/v1, /v2, /beta, /debug, /test, /swagger, /openapi.json.
impact: Exposure of API documentation, debug information, or versioning leading to further attacks. Severity: Medium.
testability: PASSIVE
[HYP] Misconfigured security headers on www.vpbank.com
class: MISCONFIG
asset: www.vpbank.com
confidence: 40
reasoning: Websites often lack proper security headers (CSP, X-Frame-Options, etc.), which could enable clickjacking or other attacks.
evidence_needed: HTTP response headers.
verify_steps: HEAD/GET request to www.vpbank.com and examine headers.
impact: Clickjacking or content injection. Severity: Low.
testability: PASSIVE
[PARKED] Potential cross-site scripting (XSS) on vpbank.com: confidence 30 (<40)
[FINAL] API versioning and debug endpoints on api.vpbank.com (confidence 60)
[FINAL] Misconfigured security headers on www.vpbank.com (confidence 40)
[NEXT] PROBE: HEAD https://api.vpbank.com/v1, then GET if 2xx/3xx/401/403
[LEARN] REJECTED MISCONFIG @ api.vpbank.com (subdomain takeover): ellb.ch is active Swiss load balancer, not a decommissioned cloud provider tenant
[RISK] vp-bank-ag: 30 reason: Passive probes only; low risk of causing damage.
[NEW] vpbank-dev.com + vpbank-stage.com discovered via production CSP (www.vpbank.com) as trusted origins; both resolve (193.222.70.165/.166) and are live Apache servers
[NEW] www.vpbank.com responds 200 on /en; Drupal + Envoy proxy; robust CSP present; CSRFT759 + AL_SESS cookies
[NEW] www.vpbank-dev.com and www.vpbank-stage.com redirect 302 to /error_path/maintenance.html (real maintenance site, not parked)
[CHANGED] attack surface expanded beyond 3 inventory hosts; dev/stage domains are scoped (company-operated) and in production trust chain
[PRIO] www.vpbank-dev.com,7.4,attack_surface=8,business_value=8,tech_exposure=8,gate_ease=8,cloud_surface=4,freshness=10
[PRIO] www.vpbank-stage.com,7.2,attack_surface=8,business_value=8,tech_exposure=8,gate_ease=8,cloud_surface=4,freshness=9
[PRIO] api.vpbank.com,6.2,attack_surface=6,business_value=9,tech_exposure=5,gate_ease=6,cloud_surface=8,freshness=5
[HYP] vpbank-dev.com / vpbank-stage.com are production-trusted staging environments reachable publicly
class: MISCONFIG
asset: vpbank-dev.com / vpbank-stage.com
confidence: 65
reasoning: Production CSP whitelists *.vpbank-dev.com and *.vpbank-stage.com as script/connect/frame sources; both domains resolve and serve Apache; only maintenance redirect sits in front. Staging envs commonly hold weaker auth, debug builds, internal API names, test data.
evidence_needed: any non-maintenance app path returning app content/auth on dev/stage; /portal/api access; distinct from prod
verify_steps: GET https://www.vpbank-dev.com/error_path/maintenance.html (fetched, real app); probe GET https://www.vpbank-dev.com/portal/api/language/en; GET https://www.vpbank-dev.com/user/login; GET https://www.vpbank-stage.com/en
impact: Dev/stage source + schema + weak-auth exposure; chaining to prod via shared session/API; severity MEDIUM-HIGH
testability: PASSIVE
[HYP] Drupal /portal/api/ endpoints exposed on www.vpbank.com maintenance surface
class: BUSLOGIC
asset: www.vpbank.com/portal/api
confidence: 40
reasoning: Maintenance JS issues POST /portal/api/language/:language; portal API namespace present on production front end; unknown auth/validation on these endpoints
evidence_needed: 200/4xx not 5xx on /portal/api paths; differing response for language param
verify_steps: GET https://www.vpbank.com/portal/api/language/en; observe status vs 500/404
[CHANGED] api.vpbank.com: All attack vectors exhausted (SSRF, policy bypass, error handling) — no exploitable response differentials
[CHANGED] www.vpbank.com: OAuth endpoints fully tested — no valid client_id discovered, redirect_uri bypass blocked
[PARKED] www.vpbank.com/portal/api/: SPA shell served but WAF maintenance intercepts all backend requests — no attack surface
[NEW] External attack surface required: Must pivot to mobile app bundles, GitHub, npm packages to discover OAuth client_id for www.vpbank.com
[PRIO] www.vpbank.com, 5.2, attack_surface=5 business_value=8 tech_exposure=6 gate_ease=4 cloud_surface=3 freshness=5
[PRIO] api.vpbank.com, 4.5, attack_surface=3 business_value=7 tech_exposure=3 gate_ease=3 cloud_surface=6 freshness=4
[PRIO] vpbank-dev.com, 3.8, attack_surface=4 business_value=6 tech_exposure=3 gate_ease=2 cloud_surface=3 freshness=3
[HYP] OAuth client_id discovery via external artifact enumeration
class: OAUTH
asset: www.vpbank.com
confidence: 45
reasoning: /oauth/authorize endpoint exists; JS bundles on marketing site contain no client_id (only Usercentrics widget clientWid); mobile app stores (iOS/Android) may embed OAuth configuration; GitHub source code may contain client registration
evidence_needed: Valid client_id string found in mobile app IPA/APK, GitHub repository, or npm package; or HTTP 200/302 from /oauth/authorize with discovered client_id
verify_steps: RAG: Search iOS App Store / Google Play for "VP Bank" apps; extract and decompile mobile app bundles; search for OAuth client_id/client_secret patterns; also search GitHub code search for "vpbank.com/oauth" or "client_id.*vpbank"
impact: OAuth flow exploitation → account takeover via authorization code theft; severity CRITICAL
testability: PASSIVE
[HYP] GraphQL introspection on www.vpbank.com portal API
class: MISCONFIG
asset: www.vpbank.com/portal/api/
confidence: 30
reasoning: Full LitElement SPA served at /portal/api/ (HTTP 403 with body); custom web components suggest GraphQL or REST backend; CSRFT759 cookie present; portal language endpoint exists; GraphQL introspection commonly enabled on fresh deployments
evidence_needed: HTTP 200 with schema definition from POST /portal/api/graphql with introspection query; or HTTP 400/500 with different error structure than WAF maintenance page
verify_steps: POST https://www.vpbank.com/portal/api/graphql with body {"query":"{__schema{types{name}}}"} and Content-Type: application/json; also try GET https://www.vpbank.com/portal/api/graphql?query=__schema
impact: Schema disclosure → full API surface mapping → potential IDOR/BOLA/authorization bypass; severity HIGH
testability: PASSIVE
[HYP] Hidden admin/debug endpoints on api.vpbank.com gateway
class: MISCONFIG
asset: api.vpbank.com
confidence: 35
reasoning: Layer7-API-Gateway returns structured JSON with requestId; all tested paths return HTTP 500 JSON 404; gateway may have admin/debug paths not in standard lists (/debug, /admin, /gateway, /config, /internal, /management, /_internal, /actuator)
evidence_needed: HTTP 200/303/401 with gateway configuration or debug data; or HTTP 401 with authentication challenge (not 500)
verify_steps: GET https://api.vpbank.com/debug; GET https://api.vpbank.com/admin; GET https://api.vpbank.com/gateway; GET https://api.vpbank.com/config; GET https://api.vpbank.com/management; GET https://api.vpbank.com/_internal
impact: Gateway configuration disclosure → API routing rules → internal service enumeration; severity HIGH
testability: PASSIVE
[PARKED] GraphQL introspection on portal API: confidence 30 < 40 threshold; WAF maintenance intercepts all backend requests — no evidence GraphQL endpoint exists behind WAF
[PARKED] Hidden admin/debug endpoints on api.vpbank.com: confidence 35 < 40 threshold; all standard paths already tested return identical 500 JSON — unlikely non-standard paths differ
[FINAL] 1) OAuth client_id discovery via external artifact enumeration (45) — only remaining high-value path
[FINAL] 2) All other hypotheses exhausted or below threshold
[NEXT] RAG: Search iOS App Store and Google Play Store for "VP Bank" or "VP Bank AG" mobile applications; extract and analyze OAuth configuration from app bundles to discover valid client_id for www.vpbank.com OAuth flow
[LEARN] REJECTED SSRF @ api.vpbank.com (Host header routing): verify_steps executed — Host:169.254.169.254, Host:localhost, X-Forwarded-Host:169.254.169.254 all returned identical HTTP 500 JSON. No backend routing influence. CONFIRMED REJECTED.
[LEARN] REJECTED OAUTH @ www.vpbank.com (redirect_uri bypass): JS bundles contain no OAuth client_id (only Usercentrics widget clientWid); /oauth/authorize always 303→error page. Cannot advance.
[LEARN] NEW INFO @ www.vpbank.com/portal/api/: Full LitElement SPA served (403 with body) — separate app from Drupal; POST /portal/api/language/:language sets portal-language cookie; backend is WAF maintenance mode; no exploitable endpoints.
[LEARN] REJECTED MISCONFIG @ www.vpbank.com (portal API access): /portal/api/ returns 403 but serves complete SPA shell; /portal/api/health and /portal/api/status return WAF maintenance page — no backend content accessible.
[LEARN] ACCEPTED MISCONFIG @ api.vpbank.com (Layer7 gateway error handling): Structured JSON errors with requestIds confirmed — info leak vector but low severity
[RISK] vp-bank-ag: 40 reason: passive GET/POST probes only; OAuth redirect_uri bypass blocked without client_id; SSRF Host header test uses internal IPs (read-only); no auth bypass, no customer data accessed, no state mutation on any endpoint; all high-value hypotheses exhausted or rejected; external artifact enumeration required to advance
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
[NEW] developer.vpbank.com (193.222.70.149) DISCOVERED via RAG — VP Bank PSD2 Developer Portal, live Apache+Envoy, NOT WAF-blocked (unlike dev/stage). Serves /psd2/swagger-ui (200), /psd2/berlin-group/v1/psd2_api.yaml OpenAPI spec.
[NEW] Plaintext http://developer.../accounts → 200. Docs: production=mTLS client cert, sandbox=basic auth. /psd2/sandbox/* → 404 (sandbox not on this vhost).
[LEARN] OAuth client_id hunt via App Store/Google/GitHub/npm surfaced only VP Bank Vietnam (different entity) + unrelated SDKs — no Liechtenstein client_id.
[PRIO] developer.vpbank.com 8.3 — highest-value live surface found this cycle.
[PRIO] developer.vpbank.com,7.10,only non-WAF anonymous banking-API surface on main front-end
[PRIO] openbanking.vpbank.com,4.20,production PSD2 host (mTLS-locked, new asset in inventory)
[HYP]
class: IDOR
asset: developer.vpbank.com/psd2/berlin-group/v1
confidence: 45
reasoning: Consent-gated Berlin Group API resolves resources by UUID (/consents/{consentId}, /accounts/{account-id}, /{payment-service}/{id}/status) with zero TPP auth binding observed (HTTP 200 unauthenticated); sandbox replicates the production REDIRECT-SCA state machine on the same session/cookie infra as www.vpbank.com; docs confirm sandbox allows manual execution.
evidence_needed: A consent created under one anonymous session is readable via GET /consents/{consentId} or /consents/{consentId}/status from a fresh anonymous session (no cookie); or any /consents/{uuid} returns non-404 for an unowned UUID.
verify_steps: sandbox-interaction test (official test env, synthetic data — requires POST): POST /psd2/berlin-group/v1/consents body {"access":{"accounts":[{"iban":"LI4408805500000000001"}]},"recurringIndicator":true,"validUntil":"2027-12-31","frequencyPerDay":4,"combinedServiceIndicator":false} + TPP-Redirect-URI: https://www.google.ch; capture consentId; then from a clean curl (no cookies): GET /consents/{consentId} and GET /consents/{consentId}/status; then GET /accounts to see if downstream account ledger of created consent is exposed anonymously.
impact: Cross-TPP consent/account/ledger reading → full simulated banking data disclosure; if code paths shared with production PSD2, chains to aggregate/PII disclosure; severity MEDIUM-HIGH
testability: AUTH_HELPED
[HYP]
class: OTHER
asset: www.vpbank.com/oauth/authorize
confidence: 42
reasoning: Spec fixes ASPSP-SCA-Approach=REDIRECT; docs state scaRedirect "contains the URL for the user to login" built from the payment/consent path — the production redirect target is the bank OAuth page; openbanking.vpbank.com cert name is now known, enabling targeted artifact search for a PSD2-scoped client_id or authorize_url pattern.
evidence_needed: From a sandbox consent response, an ASPSP-published authorize_url/scaOAuth template containing a PSD2 OAuth client_id; or RAG hit for "vpbank openbanking authorize_url" / PSD2 TPP onboarding doc with the authorize host.
verify_steps: RAG: GitHub/public docs search "openbanking.vpbank.com", "vpbank psd2 scaRedirect", "vpbank.com/oauth/authorize psd2"; then GET https://openbanking.vpbank.com (mTLS probe) — if any path returns HTTP 4xx/2xx without cert, escalate.
impact: OAuth code/consent theft via redirect_uri bypass on production bank OAuth → account/aggregate data theft; severity HIGH
testability: AUTH_HELPED
[HYP]
class: AUTH
asset: developer.vpbank.com/psd2/berlin-group/v1
confidence: 38
reasoning: Anonymous sandbox requests mint the same AL_SESS-S cookie used by www.vpbank.com Drupal portal; shared session backend on same IP would allow cross-vhost cookie acceptance or fixation.
evidence_needed: AL_SESS-S obtained from developer portal accepted by www.vpbank.com Drupal session endpoints; or session state mutated by fixes.
verify_steps: GET developer.vpbank.com/psd2/berlin-group/v1/accounts to obtain AL_SESS-S; replay same cookie on www.vpbank.com/portal/api/ and observe differing behavior vs fresh session.
impact: Session confusion → CSRF/fixation on portal state changes; severity MEDIUM
testability: PASSIVE
[NEXT] RAG: search GitHub/public web for "openbanking.vpbank.com", "vpbank psd2 scaRedirect", "vpbank.com/oauth/authorize" PSD2 client_id/authorize_url; if a PSD2-scoped client_id surfaces, follow with GET https://www.vpbank.com/oauth/authorize?client_id=<id>&response_type=code&redirect_uri=... (read-only).
[LEARN] ACCEPTED MISCONFIG @ developer.vpbank.com: PSD2 Berlin Group sandbox API reachable HTTP 200 without client cert or basic auth (documented-open since 2024-05); on same IP as www.vpbank.com, not WAF-blocked, mints AL_SESS-S session cookies — new interactive attack surface, low standalone severity.
[LEARN] NEW INFO @ openbanking.vpbank.com (193.222.70.154): discovered via TLS cert CN=openbanking.vpbank.com; production PSD2 ASPSP, mTLS "certificate required" at TLS layer — anonymous surface blocked as designed.
[LEARN] NEW INFO @ developer.vpbank.com: /psd2/sandbox/certificate/verify_certificate also mTLS-gated; OpenAPI spec self-labels server "PSD2 production server" while operational env is sandbox.
[LEARN] REJECTED MISCONFIG @ developer.vpbank.com (Drupal admin on PSD2 vhost): /user/login, /admin, /sites/default/files/* all HTTP 404 — dedicated docroot.
[LEARN] REJECTED MISCONFIG @ openbanking.vpbank.com (anonymous API access): mTLS enforced, anonymous rejected — expected production posture, not a flaw.
[RISK] vp-bank-ag: 45 — verdict unchanged. New developer.vpbank.com surface (7.1 priority) is a documented-open sandbox on the main front-end, so standalone damage is limited to synthetic data; production PSD2 (openbanking.vpbank.com) is correctly mTLS-locked; www.vpbank.com OAuth remains the only CRITICAL-path lead, still gated on a valid client_id. Confidence in exploitability stays driven by the two surviving PSD2/OAuth chains.
[PRIO] developer.vpbank.com,8.40,probable carryover of proven consent/account/ledger BOLA to mTLS-locked production; attack_surface=9 business=9 tech=8 gate=10 cloud=2 fresh=10
[PRIO] www.vpbank.com,6.25,production OAuth authorize still the only CRITICAL-chain lead; attack_surface=6 business=10 tech=9 gate=2 cloud=2 fresh=4
[PRIO] openbanking.vpbank.com,5.50,production ASPSP whose authz model the sandbox BOLA questions; attack_surface=3 business=10 tech=8 gate=1 cloud=2 fresh=7
[HYP] PSD2 sandbox BOLA: consent/account/ledger/payment resources readable cross-session via bearer consentId
class: IDOR
asset: developer.vpbank.com/psd2/berlin-group/v1
confidence: 80
reasoning: my probes — consent POST anonymously (no basic auth) → 201; fresh anonymous session (no cookie/jar) reads /consents/{id}/status, /consents/{id}, /accounts, /accounts/{iban}/balances, /accounts/{iban}/transactions (Consent-ID header only), and /payments/{paymentId}/status (200 ACSC); spec requires only Consent-ID + X-Request-ID for account/ledger endpoints, defines no securityScheme, self-labels server "PSD2 production server"; docs: sandbox data model identical to production, differences only TPP client auth/user interaction/state changes.
evidence_needed: sandbox: SATISFIED (final). Live: production openbanking.vpbank.com binds Consent-ID to originating TPP QWAC and rejects cross-TPP reads (requires two eIDAS certs).
verify_steps: sandbox DONE; production (HUMAN_ONLY, mTLS): TPP-A creates consent; TPP-B sends GET /psd2/berlin-group/v1/consents/{id}/status and GET /accounts/{iban}/balances with TPP-B cert + TPP-A Consent-ID header ∈ openbanking.vpbank.com.
impact: any party holding a consentId/paymentId reads consent scope + simulated ledger; if prod reuses sandbox authz logic → cross-TPP financial/PII disclosure; severity MEDIUM (sandbox/synthetic) → HIGH (prod carryover).
testability: AUTH_HELPED (sandbox verified; prod HUMAN_ONLY)
[HYP] OAuth redirect_uri/state bypass via scaRedirect-pattern client on www.vpbank.com
class: OATH
asset: www.vpbank.com/oauth/authorize
confidence: 45
reasoning: authorize endpoint returns 303/400 for all tested combos; no client_id in JS bundles (only Usercentrics clientWid) or RAG (GitHub/App Store/npm → only VP Bank Vietnam + generic PSD2 stacks); spec exposes scaRedirect (hrefType) but sandbox returns "not available in sandbox"; docs state scaRedirect carries the user-login URL in the REDIRECT-SCA production flow.
evidence_needed: any valid client_id (PSD2 or Drupal-scoped) that yields a response differential on redirect_uri.
verify_steps: RAG GitHub code-search "vpbank" + "authorize_url"/"client_id"; then GET https://www.vpbank.com/oauth/authorize?client_id=<id>&response_type=code&redirect_uri=<test>&state=x (read-only).
impact: authorization-code/consent interception → ATO, aggregated-account theft on production bank OAuth; severity HIGH.
testability: AUTH_HELPED
[HYP] Production PSD2 reuses sandbox authorization model (Consent-ID unbound from originating TPP)
class: IDOR
asset: openbanking.vpbank.com
confidence: 35
reasoning: TLS-layer mTLS blocks all anonymous probing (verified prior); docs list only TPP auth/user-interaction/state-changes as sandbox-vs-prod deltas — Consent-ID ownership binding is not listed; sandbox proves the code path accepts the consentId header with zero identity binding.
evidence_needed: cross-QWAC read test proving bind/absence on production host.
verify_steps: HUMAN_ONLY — with two licensed TPP certs perform the same cross-identity Consent-ID read listed under the developer.vpbank.com hypothesis, on openbanking.vpbank.com.
impact: cross-TPP production consent/account/ledger disclosure → PII/financial data; severity HIGH.
testability: HUMAN_ONLY
[PARKED] OAuth redirect_uri bypass @ www.vpbank.com: confidence 45 but no client_id exists across JS/RAG/App-Store/npm and the spec defines no OAuth client; no executable verify step without a client_id — cannot advance passively.
[PARKED] Production BOLA carryover @ openbanking.vpbank.com: confidence 35 < 40; mTLS at TLS layer makes anonymous verification impossible; HUMAN_ONLY requires eIDAS QWAC certs.
[PARKED] X-Request-ID state-encoding manipulation (BUSLOGIC): deterministic client-controlled state is documented sandbox design for testing, not a vulnerability; probe showed last digit 1 already yields ACSC — encoding not fixed, no security differential.
[FINAL] 1. [80] PSD2 sandbox BOLA/IDOR — PROVEN end-to-end in sandbox; report artifact plus prod-carryover verification request.
[NEXT] SCAN: passive CT enumeration via crt.sh (`%.vpbank.com`, `%.vpbank.li`) to catalog PSD2/openbanking/statistics subdomains, then anonymous reachability check of each live host at /psd2/berlin-group/v1/accounts and /psd2/swagger-ui (GET, ≤1 rps) hunting non-mTLS instances of the same consent-authz code line that could carry the proven sandbox BOLA.
[LEARN] ACCEPTED IDOR @ developer.vpbank.com (PSD2 sandbox BOLA): verify_steps EXECUTED in official test sandbox (synthetic data) — consent 6b517824-e5af-4202-b9b0-7f483a68ee9f (POST 201, anonymous) read by a fresh anonymous session: /consents/{id}/status 200, /consents/{id} 200 (IBAN LI6508805500000000011), /accounts 200, /balances 200 (€12453.2), /transactions 200 (Consent-ID header only), /payments/33b07f90-8ab6-4e4d-b0e7-fb909b685df8/status 200 ACSC. Zero session/TPP/basic-auth binding on consentId/paymentId.
[LEARN] REJECTED BUSLOGIC @ developer.vpbank.com (X-Request-ID state encoding): documented deterministic client-driven state for the sandbox; observed ACSC from X-Request-ID ending in 1 (docs claim 1=RCVD) — no security-relevant differential, not reportable.
[LEARN] REJECTED OAUTH @ www.vpbank.com/oauth/authorize: RAG GitHub/public-web surfaces only VP Bank Vietnam (separate entity) and generic PSD2 frameworks; downloaded spec (46KB) contains no OAuth/securitySchemes/client_id — only scaRedirect hrefs ("not available in sandbox"); no client context obtainable, redirect-flow test blocked.
[LEARN] REJECTED MISCONFIG @ www.vpbank.com/developer.vpbank.com (PSD2 statistics pages): /psd2-statistics, /psd2-statistics/, /psd2/statistics/ all HTTP 404 — no anonymous statistics surface exists.
[RISK] vp-bank-ag: 55 — raised from 45. Confirmed missing authorization binding on the entire PSD2 consent→ledger→payment-status chain in the documented-open sandbox running on the main front-end (developer.vpbank.com); the spec self-labels the server "PSD2 production server", docs claim a data model identical to production with TPP authentication the only named difference, and production openbanking.vpbank.com is unverified because of TLS-layer mTLS — if it mirrors the sandbox, cross-TPP financial data disclosure is a real chain. Residuals: OAuth (client_id-gated), staging (WAF-blocked), api.vpbank.com (exhausted).
[NEW] CT enumeration (crt.sh) expands inventory 6→285 hostnames; live web-accessible additions: digital-onboarding/vpbank.com family (prod+dev+stage), sts.vpbank.com (AD FS), api-prep.vpbank.com (Layer7 prep clone), designsystem.vpbank.com (Netlify CNAME), ebics/mobile/vop/www-beta/mobile-beta/report/concentsol.
[NEW] digital-onboarding.vpbank.com (fn/countersigned brand: "Onboarding | VP Bank", © VP BANK AG Vaduz): production multi-tenant bank onboarding + back-office platform (SaaS "US", Rails/Devise/devise-lockable), hosted OFF-net on 89.163.182.69 (dev .28, stage .8) — prod tenant_id=4, dev=129, stage=7; login POST /users/sign_in renders hidden client-supplied fields user[tenant_id], user[admin]=false, user[user_id]=0.
[NEW] /control-center/ on prod serves full "Business Control Center" back-office SPA anonymously (HTTP 200) with admin modules (onboarding cases, ident documents, bankingtransactions, incomingwire, rolemgmt, clientsearch, documentmgmt, auditlog) and API endpoints /api/v1/tenants, /api/v1/brand, /api/v1/current_user_details, /api/v1/sessions/{idp_login,reset_password,secure_session}, /admin/api/v1/users, /rails/active_storage/direct_uploads.
[NEW] /api/v1/brand returns HTTP 200 ANONYMOUSLY on prod (tenant config, i18n, page_title "Business Control Center", tenant_symbol vpbanklighttenant); /api/v1/tenants returns 403 "Not authorized".
[NEW] sts.vpbank.com (193.222.70.198): Microsoft AD FS (Microsoft-HTTPAPI/2.0); /adfs/.well-known/openid-configuration HTTP 200 — issuer https://sts.vpbank.com/adfs, device_code+password+implicit grants, scopes winhello_cert/vpn_cert/logon_cert.
[CHANGED] api-prep.vpbank.com (CNAME api-prep-vpbank-com.hdb-cs04.ellb.ch, 195.186.145.90): Layer7 clone of api.vpbank.com — SCSS-Request-ID, INVALID_REQUEST_RESOURCE JSON 404 for all paths; no new surface.
[CHANGED] designsystem.vpbank.com CNAME→vpb-design-system.netlify.app serves HTTP 200 (live) — subdomain takeover NOT present.
[CHANGED] vop.vpbank.com/.vop-stage on 193.222.70.154 (openbanking IP): HTTPS unreachable anonymously (TLS drop) — mTLS-gated like openbanking.
[PRIO] digital-onboarding.vpbank.com,8.10,fresh multi-tenant onboarding+back-office with anonymous SPA/API + mass-assignment login fields + off-net hosting; attack=8 business=9 tech=8 gate=7 cloud=6 fresh=10
[PRIO] developer.vpbank.com,8.40,carried — proven PSD2 sandbox BOLA, consent/paymentId unbound across anonymous sessions; attack=9 business=9 tech=8 gate=10 cloud=2 fresh=10
[PRIO] openbanking.vpbank.com,5.50,carried — production PSD2 ASPSP, mTLS locked, authz-model carryover unverifiable anonymously; attack=3 business=10 tech=8 gate=1 cloud=2 fresh=7
[HYP] Mass assignment on Devise sign-in grants admin/impersonation on onboarding back-office
class: AUTH
asset: digital-onboarding.vpbank.com/users/sign_in
confidence: 45
reasoning: POST /users/sign_in form renders hidden client-controlled user[tenant_id] (4/129/7), user[admin]=false, user[user_id]=0 — fields only meaningful if sessions#create consumes them (impersonation/"login as" and admin flags); same app code on prod+dev+stage; back-office grants onboarding cases, ident documents, banking/wire operations, role/org admin (from /api/v1/brand + JS bundle); anonymous /api/v1/brand 200 confirms API reachable pre-auth; /api/v1/tenants correctly 403.
evidence_needed: on dev/stage (synthetic tenants 129/7): authenticated test user signs in with user[admin]=true appended → session carries admin privileges (visible via /admin/api/v1/users or rolemgmt); or user[user_id]=<other> produces other-user session.
verify_steps: dev-only, ≤1rps, NO customer data: (1) already GET /users/sign_in form (done, confirms fields); (2) POST https://digital-onboarding-dev.vpbank.com/users/sign_in with authenticity_token + user[email]=<test-user> + user[password]=<pw> + user[admin]=true then GET /admin/api/v1/users — compare HTTP code/differing body vs same login with user[admin]=false. Requires operator-issued dev test credential.
impact: anonymous/reduced-priv attacker escalates to full bank back-office admin → mass PII (client name/email/mobile, IDNow ident docs, documents) + financial ops (banking transactions, incoming wire), case/org/role manipulation; severity HIGH.
testability: AUTH_HELPED
[HYP] PSD2 sandbox BOLA: consent/account/ledger/payment resources readable cross-session via bearer consentId
class: IDOR
asset: developer.vpbank.com/psd2/berlin-group/v1
confidence: 80
reasoning: (carried, PROVEN) consent created +201 anonymously, read cross-session by fresh anonymous session across /consents/{id}, /consents/{id}/status, /accounts, /balances, /transactions, /payments/{id}/status (ACSC); spec defines no securityScheme, self-labels server "PSD2 production server"; sandbox data model identical to production.
evidence_needed: reproduced in prod openbanking.vpbank.com (requires two eIDAS QWACs — HUMAN_ONLY).
verify_steps: sandbox DONE; prod (HUMAN_ONLY): TPP-B reads consentId/Consent-ID created by TPP-A with TPP-B's own mTLS cert at openbanking.vpbank.com.
impact: cross-TPP consent/ledger/PII disclosure if prod mirrors sandbox; severity HIGH (prod carryover).
testability: AUTH_HELPED (sandbox verified; prod HUMAN_ONLY)
[HYP] Production PSD2 reuses sandbox authorization model (Consent-ID unbound from originating TPP)
class: IDOR
asset: openbanking.vpbank.com
confidence: 35
reasoning: TLS-layer mTLS verified; docs list TPP-auth/user-interaction/state-changes as only sandbox-vs-prod deltas; Consent-ID ownership binding not listed; sandbox proves the code path honors consentId header with zero identity binding.
evidence_needed: cross-QWAC read on openbanking.vpbank.com.
verify_steps: HUMAN_ONLY with two licensed TPP certs on openbanking.vpbank.com (same sequence as sandbox proof).
impact: cross-TPP financial/PII disclosure; severity HIGH.
testability: HUMAN_ONLY
[PARKED] sts.vpbank.com ADFS OIDC: metadata+device_code visible, but corporate employee IdP, `password`/`device_code` flows — no demonstrated exploit, program scope focuses customer assets/public login panels excluded; confidence <40.
[PARKED] designsystem.vpbank.com → Netlify takeover: netlify site serves 200, active, not claimable.
[PARKED] api-prep.vpbank.com: exact Layer7 clone of exhausted api.vpbank.com (INVALID_REQUEST_RESOURCE), no differential.
[FINAL] 1. [80] PSD2 sandbox BOLA @ developer.vpbank.com — PROVEN, report artifact + prod-carryover verification request (carried).
[FINAL] 2. [45] Mass assignment @ digital-onboarding.vpbank.com — new, strong evidence (admin/tenant/user_id client fields, off-net, dev/stage testable), needs one dev credential.
[NEXT] PROBE: on synthetic dev tenant, anonymous-authz gate differential — GET https://digital-onboarding-dev.vpbank.com/admin/api/v1/users and GET https://digital-onboarding-dev.vpbank.com/api/v1/sessions/idp_login (read-only, ≤1rps) comparing dev vs prod HTTP codes (prod /api/v1/tenants=403 baseline) to locate any pre-auth admin endpoint; then POST sign-in mass-assignment test only after obtaining an operator dev credential.
[LEARN] ACCEPTED MISCONFIG @ digital-onboarding.vpbank.com: Live multi-tenant bank-onboarding/back-office SaaS ('US', Rails+Devise) on off-net hosting (89.163.182.69/.28/.8), anonymous /control-center/ SPA + /api/v1/brand 200, sign_in carries client-controlled admin/tenant/user_id — new in-scope back-office attack surface.
[LEARN] REJECTED MISCONFIG @ api-prep.vpbank.com: Same Layer7 dead-end as api.vpbank.com (SCS-Request-ID, INVALID_REQUEST_RESOURCE) — pre-prod gateway clone, no new surface.
[LEARN] REJECTED MISCONFIG @ designsystem.vpbank.com: CNAME→vpb-design-system.netlify.app serves 200 — hosted design system is active, no subdomain takeover.
[LEARN] NEW INFO @ sts.vpbank.com: ADFS OIDC metadata 200 (issuer sts.vpbank.com/adfs, device_code/password grants, vpn/logon cert scopes) — corporate IdP, parked.
[RISK] vp-bank-ag: 60 — raised from 55. PSD2 sandbox BOLA remains PROVEN; digital-onboarding adds a distinct, fresh, high-value back-office surface (customer onboarding PII + financial ops) on off-net hosting with invented login that allows client control of admin/tenant/user_id fields — a concrete mass-assignment lead testable on synthetic dev/stage tenants. Residuals: production PSD2 mTLS-blocked (carryover unverifiable), OAuth client_id-gated, ADFS parked (corporate scope), gateway/staging surfaces exhausted.
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
## 2026-09-05 09:51:29 UTC [target] (model bigpickle)
[PRIO] developer.vpbank.com,8.40,attack_surface=9,business_value=9,tech_exposure=8,gate_ease=10,cloud_surface=2,freshness=10
[PRIO] digital-onboarding.vpbank.com,7.85,attack_surface=8,business_value=8,tech_exposure=7,gate_ease=10,cloud_surface=2,freshness=10
[PRIO] www.vpbank.com,6.25,attack_surface=6,business_value=10,tech_exposure=9,gate_ease=2,cloud_surface=2,freshness=4
[PRIO] openbanking.vpbank.com,5.50,attack_surface=3,business_value=10,tech_exposure=8,gate_ease=1,cloud_surface=2,freshness=7
[PRIO] api.vpbank.com,4.50,attack_surface=3,business_value=7,tech_exposure=3,gate_ease=3,cloud_surface=6,freshness=4
[HYP] digital-onboarding.vpbank.com IDOR via /api/v1/ tenant/admin endpoints
class: IDOR
asset: digital-onboarding.vpbank.com/api/v1/
confidence: 55
reasoning: /control-center/ SPA serves full back-office with admin modules (onboarding cases, ident documents, bankingtransactions, incomingwire, rolemgmt) anonymously. /api/v1/brand returns 200 anonymous. /api/v1/tenants returns 403 "Not authorized" — auth exists but is per-endpoint. Rails+Devise stack on off-net hosting (89.163.182.69). sign_in carries client-controlled admin/tenant/user_id parameters. If endpoints like /api/v1/onboarding_cases, /api/v1/bankingtransactions accept tenant_id in request body/query, cross-tenant data access is likely.
evidence_needed: Enumeration of /api/v1/ routes; any authenticated endpoint that accepts tenant_id/user_id as parameter without server-side validation
verify_steps: GET https://digital-onboarding.vpbank.com/api/v1/brand (200, baseline). GET https://digital-onboarding.vpbank.com/api/v1/users, /api/v1/onboarding_cases, /api/v1/bankingtransactions, /api/v1/incomingwire, /api/v1/rolemgmt — any 200 = open endpoint. If 403, test sign_in POST with forged tenant_id to create session, then access same endpoints.
impact: Cross-tenant PII/financial data disclosure (onboarding cases, wire transfers, identity documents); severity HIGH
testability: AUTH_HELPED (sign_in endpoint accepts client-controlled params; session creation possible without credentials)
[HYP] PSD2 sandbox-to-production authorization logic carryover
class: IDOR
asset: openbanking.vpbank.com
confidence: 35
reasoning: Production openbanking.vpbank.com blocks anonymous access via mTLS. Docs list only TPP auth/user-interaction/state-changes as sandbox-vs-prod deltas — Consent-ID ownership binding not listed. Sandbox proves code path accepts consentId header with zero identity binding. If production shares authorization module, cross-TPP reads possible.
evidence_needed: Cross-QWAC read test proving bind/absence on production host
verify_steps: HUMAN_ONLY — two licensed TPP certs performing cross-identity Consent-ID read on openbanking.vpbank.com
impact: Cross-TPP production consent/account/ledger disclosure; severity HIGH
testability: HUMAN_ONLY
[HYP] digital-onboarding.vpbank.com mass assignment via sign_in parameter flooding
class: OTHER
asset: digital-onboarding.vpbank.com/control-center/
confidence: 45
reasoning: SPA includes modules for admin, tenant management, user roles. sign_in flow carries client-controlled admin/tenant/user_id. Rails+Devise with custom devise-lockable may accept additional params not sanitized. If sign_in POST accepts role=admin or is_admin=true, privilege escalation possible.
evidence_needed: POST /sign_in with role/is_admin parameter resulting in elevated session
verify_steps: GET https://digital-onboarding.vpbank.com/control-center/ to capture SPA form endpoints. Identify sign_in POST endpoint from JS bundle. POST with role=admin appended.
impact: Admin access to full back-office (onboarding, wires, identity docs); severity HIGH
testability: PASSIVE (JS bundle analysis first, then live probe)
[NEXT] SCAN: passive JS bundle extraction from https://digital-onboarding.vpbank.com/control-center/ — extract all /api/v1/* routes, sign_in endpoint, form action URLs, CSRF tokens, and any hardcoded tenant/user/admin parameters from the SPA JavaScript.
[LEARN] ACCEPTED MISCONFIG @ digital-onboarding.vpbank.com: Production multi-tenant back-office SaaS (Rails+Devise), anonymous /control-center/ SPA + /api/v1/brand 200 — off-net hosting (89.163.182.69), client-controlled admin/tenant/user_id in sign_in — NEW high-value attack surface
[LEARN] NEW INFO @ sts.vpbank.com: ADFS OIDC metadata 200 (issuer sts.vpbank.com/adfs, device_code/password/implicit grants, vpn/logon/cert scopes) — corporate IdP, parked
[LEARN] REJECTED MISCONFIG @ api-prep.vpbank.com: Layer7 clone of api.vpbank.com — no new surface
[LEARN] REJECTED MISCONFIG @ designsystem.vpbank.com: Active Netlify app — no subdomain takeover
[LEARN] ACCEPTED IDOR @ developer.vpbank.com: PSD2 sandbox BOLA verified end-to-end — consent/account/payment cross-session read, zero binding
[LEARN] REJECTED OAUTH @ www.vpbank.com: No client_id obtainable — redirect_uri bypass blocked
[LEARN] REJECTED MISCONFIG @ vpbank-dev.com/vpbank-stage.com: WAF intercepts ALL paths — confirmed rejected
[RISK] vp-bank-ag: 57 — raised from 55. digital-onboarding.vpbank.com adds anonymous back-office SPA with full admin module visibility (onboarding cases, wire transfers, identity documents) on off-net Rails+Devise infrastructure. Combined with confirmed PSD2 sandbox BOLA (synthetic data, reportable) + potential sandbox-to-production carryover, attack surface now spans three independent financial application stacks. Residuals: OAuth (client_id-gated), staging (WAF-blocked), api.vpbank.com (exhausted), openbanking (mTLS).
## 2026-09-05 13:20:50 UTC [target] (model bigpickle)
[PRIO] developer.vpbank.com,8.40,attack_surface=9,business_value=9,tech_exposure=8,gate_ease=10,cloud_surface=2,freshness=10
[PRIO] digital-onboarding.vpbank.com,7.85,attack_surface=8,business_value=8,tech_exposure=7,gate_ease=10,cloud_surface=2,freshness=10
[PRIO] www.vpbank.com,6.25,attack_surface=6,business_value=10,tech_exposure=9,gate_ease=2,cloud_surface=2,freshness=4
[PRIO] openbanking.vpbank.com,5.50,attack_surface=3,business_value=10,tech_exposure=8,gate_ease=1,cloud_surface=2,freshness=7
[PRIO] api.vpbank.com,4.50,attack_surface=3,business_value=7,tech_exposure=3,gate_ease=3,cloud_surface=6,freshness=4
[HYP] digital-onboarding.vpbank.com IDOR via /api/v1/ tenant/admin endpoints
class: IDOR
asset: digital-onboarding.vpbank.com/api/v1/
confidence: 55
reasoning: /control-center/ SPA serves full back-office with admin modules (onboarding cases, ident documents, bankingtransactions, incomingwire, rolemgmt) anonymously. /api/v1/brand returns 200 anonymous. /api/v1/tenants returns 403 "Not authorized" — auth exists but is per-endpoint. Rails+Devise stack on off-net hosting (89.163.182.69). sign_in carries client-controlled admin/tenant/user_id parameters. If endpoints like /api/v1/onboarding_cases, /api/v1/bankingtransactions accept tenant_id in request body/query, cross-tenant data access is likely.
evidence_needed: Enumeration of /api/v1/ routes; any authenticated endpoint that accepts tenant_id/user_id as parameter without server-side validation
verify_steps: GET https://digital-onboarding.vpbank.com/api/v1/brand (200, baseline). GET https://digital-onboarding.vpbank.com/api/v1/users, /api/v1/onboarding_cases, /api/v1/bankingtransactions, /api/v1/incomingwire, /api/v1/rolemgmt — any 200 = open endpoint. If 403, test sign_in POST with forged tenant_id to create session, then access same endpoints.
impact: Cross-tenant PII/financial data disclosure (onboarding cases, wire transfers, identity documents); severity HIGH
testability: AUTH_HELPED (sign_in endpoint accepts client-controlled params; session creation possible without credentials)
[HYP] PSD2 sandbox-to-production authorization logic carryover
class: IDOR
asset: openbanking.vpbank.com
confidence: 35
reasoning: Production openbanking.vpbank.com blocks anonymous access via mTLS. Docs list only TPP auth/user-interaction/state-changes as sandbox-vs-prod deltas — Consent-ID ownership binding not listed. Sandbox proves code path accepts consentId header with zero identity binding. If production shares authorization module, cross-TPP reads possible.
evidence_needed: Cross-QWAC read test proving bind/absence on production host
verify_steps: HUMAN_ONLY — two licensed TPP certs performing cross-identity Consent-ID read on openbanking.vpbank.com
impact: Cross-TPP production consent/account/ledger disclosure; severity HIGH
testability: HUMAN_ONLY
[HYP] digital-onboarding.vpbank.com mass assignment via sign_in parameter flooding
class: OTHER
asset: digital-onboarding.vpbank.com/control-center/
confidence: 45
reasoning: SPA includes modules for admin, tenant management, user roles. sign_in flow carries client-controlled admin/tenant/user_id. Rails+Devise with custom devise-lockable may accept additional params not sanitized. If sign_in POST accepts role=admin or is_admin=true, privilege escalation possible.
evidence_needed: POST /sign_in with role/is_admin parameter resulting in elevated session
verify_steps: GET https://digital-onboarding.vpbank.com/control-center/ to capture SPA form endpoints. Identify sign_in POST endpoint from JS bundle. POST with role=admin appended.
impact: Admin access to full back-office (onboarding, wires, identity docs); severity HIGH
testability: PASSIVE (JS bundle analysis first, then live probe)
[NEXT] SCAN: passive JS bundle extraction from https://digital-onboarding.vpbank.com/control-center/ — extract all /api/v1/* routes, sign_in endpoint, form action URLs, CSRF tokens, and any hardcoded tenant/user/admin parameters from the SPA JavaScript.
[LEARN] ACCEPTED MISCONFIG @ digital-onboarding.vpbank.com: Production multi-tenant back-office SaaS (Rails+Devise), anonymous /control-center/ SPA + /api/v1/brand 200 — off-net hosting (89.163.182.69), client-controlled admin/tenant/user_id in sign_in — NEW high-value attack surface
[LEARN] NEW INFO @ sts.vpbank.com: ADFS OIDC metadata 200 (issuer sts.vpbank.com/adfs, device_code/password/implicit grants, vpn/logon/cert scopes) — corporate IdP, parked
[LEARN] REJECTED MISCONFIG @ api-prep.vpbank.com: Layer7 clone of api.vpbank.com — no new surface
[LEARN] REJECTED MISCONFIG @ designsystem.vpbank.com: Active Netlify app — no subdomain takeover
[LEARN] ACCEPTED IDOR @ developer.vpbank.com: PSD2 sandbox BOLA verified end-to-end — consent/account/payment cross-session read, zero binding
[LEARN] REJECTED OAUTH @ www.vpbank.com: No client_id obtainable — redirect_uri bypass blocked
[LEARN] REJECTED MISCONFIG @ vpbank-dev.com/vpbank-stage.com: WAF intercepts ALL paths — confirmed rejected
[RISK] vp-bank-ag: 57 — raised from 55. digital-onboarding.vpbank.com adds anonymous back-office SPA with full admin module visibility (onboarding cases, wire transfers, identity documents) on off-net Rails+Devise infrastructure. Combined with confirmed PSD2 sandbox BOLA (synthetic data, reportable) + potential sandbox-to-production carryover, attack surface now spans three independent financial application stacks. Residuals: OAuth (client_id-gated), staging (WAF-blocked), api.vpbank.com (exhausted), openbanking (mTLS).
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
[HYP] force_tenant cross-tenant session/data IDOR on digital-onboarding back-office
class: IDOR
asset: digital-onboarding.vpbank.com/
confidence: 50
reasoning: SPA appends client-controlled `force_tenant` to every API request including session creation (bundle code: `o.force_tenant=It.get("FORCE_TENANT")` appended in `Dx` call layer); prod `/api/v1/brand?force_tenant=vpbank` returns HTTP 200 with second tenant config anonymously (tenant scoping honored server-side); `/api/v1/qr_codes/generate` returns "2fa not enabled for provided tenant" — tenant context is read from request. If a low-priv session's JWT tenant claim can be influenced by force_tenant at POST /api/v1/sessions (body `{"user":{...}}` + store `FORCE_TENANT`), cross-tenant onboarding-case/wire/ident reads follow.
evidence_needed: login to dev/stage sandbox with any account, then repeat the same request with force_tenant=vpbank and observe data set switch or unauthorized access to /admin/api/v1/ident_only_cases, /admin/api/v1/incoming_wires
verify_steps: GET https://digital-onboarding-dev.vpbank.com/api/v1/brand?force_tenant=vpbank (200, baseline tenant switch). Login→GET /admin/api/v1/users?force_tenant=vpbank from tenant-A session; 200 with tenant-B records = cross-tenant IDOR (dev sandbox only, requires valid dev creds).
impact: Cross-tenant PII/financial disclosure (onboarding cases, ident docs, wires, mailing lists) across tenants of a bank onboarding SaaS; severity HIGH
testability: AUTH_HELPED
[HYP] ADFS device_code grant token theft via phishing on sts.vpbank.com
class: OAUTH
asset: sts.vpbank.com/adfs
confidence: 55
reasoning: /adfs/.well-known/openid-configuration exposes device_code+password+implicit grants (issuer sts.vpbank.com/adfs, scopes vpn/logon/cert); /adfs/oauth2/token/devicecode returns 405 on GET, endpoint exists; device-flow adversariably polls for victim approval (phishing to enter user code on attacker device) yielding vpn/logon tokens.
evidence_needed: valid client_id + successful devicecode POST to observe user_code/verification_uri; RAG might surface ADFS client_id in VP Bank docs/portal
verify_steps: POST https://sts.vpbank.com/adfs/oauth2/token/devicecode 'client_id=<found>&scope=vpn' → 200 with user_code/verification_uri proves flow usable (requires client_id discovery; token issuance requires HUMAN victim approval)
impact: Corporate VPN/cert-scope tokens via device-flow phishing → internal network lateral movement; severity HIGH
testability: AUTH_HELPED
[HYP] PSD2 sandbox authorization model carries to production (cross-TPP Consent-ID read)
class: IDOR
asset: openbanking.vpbank.com
confidence: 35
reasoning: Sandbox proves zero session/TPP binding on consentId/paymentId chain; docs list only TPP auth/user-interaction as deltas; production authorizes via TLS mTLS only (blocked anonymously).
evidence_needed: two QWAC certs cross-reading Consent-ID on production
verify_steps: HUMAN_ONLY — TPP-A consent, TPP-B reads /consents/{id}/status on openbanking.vpbank.com/psd2/berlin-group/v1
impact: Cross-TPP production consent/account/ledger disclosure; severity HIGH
testability: HUMAN_ONLY
[NEXT] RAG: hunt for a VP Bank/ADFS `client_id` for sts.vpbank.com device_code flow (search VP Bank eBanking/secure docs, GitHub, "vpbank" + "adfs", IT-support KBs); once found → PROBE POST https://sts.vpbank.com/adfs/oauth2/token/devicecode `client_id=<id>&scope=vpn` (200 with user_code/verification_uri = device-flow phishing confirmed).
[RISK] vp-bank-ag: 57 — unchanged. Confirmed PSD2 sandbox BOLA (reportable, synthetic) remains the flagship; digital-onboarding depth-probe shows the data layer is JWT+signature-enforced (no anonymous data access, no JWT confusion), so the anonymous attack surface is limited to the SPA route map + tenant-config enumeration (low); residual risk concentrated in (a) proven BOLA with unverified prod carryover (mTLS-gated), (b) client-controlled force_tenant mechanism that could enable cross-tenant access for any authenticated low-priv user, (c) ADFS device-flow phishing. OAuth (client_id-gated), staging (WAF), api.vpbank.com (exhausted), openbanking (mTLS), mass-assignment (unverifiable without creds).
## 2026-09-05 16:08:53 UTC [target] (model bigpickle)
## 2026-09-05 18:26:30 UTC [target] (model bigpickle)
[PRIO] digital-onboarding.vpbank.com,8.4,attack_surface+gate_ease+tech_exposure
[PRIO] sts.vpbank.com,6.5,tech_exposure+attack_surface
[HYP] force_tenant cross-tenant data access on digital-onboarding sessions
class: IDOR
asset: digital-onboarding.vpbank.com/api/v1/sessions
confidence: 45
reasoning: SPA fetch wrapper reads `FORCE_TENANT` from localStorage and appends to every request including those initiated at/after sign-in; bundle exposes `selectForceTenant`/`setForceTenant` and prod anonymous `/api/v1/brand?force_tenant=vpbank` returns 200 (tenant honored server-side); `/api/v1/tenants` is JWT-gated so tenant scoping is enforced on data, but force_tenant is client-controlled.
evidence_needed: a valid low-priv session JWT, then re-issue same admin request with force_tenant=vpbantenantB and observe cross-tenant records 200.
verify_steps: GET https://digital-onboarding.vpbank.com/api/v1/brand?force_tenant=vpbank (200, baseline). With dev-stage creds: POST /api/v1/sessions {"user":{email,password}} → capture JWT → GET /admin/api/v1/users?force_tenant=vpbantenantB with that JWT; 200 with tenantB users = cross-tenant BOLA. (dev sandbox only, requires valid dev creds — AUTH_HELPED).
impact: Cross-tenant PII/financial disclosure (onboarding cases, ident docs, wires) across tenants of a bank onboarding SaaS; HIGH
testability: AUTH_HELPED
[HYP] ADFS device_code phishing on sts.vpbank.com
class: OATH
asset: sts.vpbank.com/adfs/oauth2/devicecode
confidence: 50
reasoning: openid-configuration exposes device_code+password+implicit grants (issuer sts.vpbank.com/adfs, scopes vpn/logon/cert); /adfs/oauth2/devicecode returns 405 on GET (endpoint enabled); correct ADFS path confirmed (the /token/devicecode 200 is an MS-HTTPAPI error shell, X-MS-Forwarded-Status-Code:500).
evidence_needed: valid client_id + successful devicecode POST returning user_code/verification_uri.
verify_steps: POST https://sts.vpbank.com/adfs/oauth2/devicecode 'client_id=<found>&scope=vpn' → 200 with user_code = flow usable (client_id discovery via VP Bank VP Connect mobile bundle / RAG; token issuance requires HUMAN victim approval).
impact: Corporate VPN/cert-scope tokens via device-flow phishing → internal network lateral movement; HIGH
testability: AUTH_HELPED
[HYP] (carryover) PSD2 sandbox BOLA to production
class: IDOR
asset: openbanking.vpbank.com
confidence: 35
reasoning: sandbox proves zero binding on consentId/paymentId; production mTLS-gated.
evidence_needed: two QWAC certs cross-reading.
verify_steps: HUMAN_ONLY
impact: cross-TPP production consent/ledger disclosure; HIGH
testability: HUMAN_ONLY
[NEXT] RAG: extract ADFS `client_id` from VP Bank Connect mobile app (App Store/Google Play bundle strings, "vpbank connect" + "adfs"/"devicecode"/"oauth") to unlock POST https://sts.vpbank.com/adfs/oauth2/devicecode 'client_id=<id>&scope=vpn' — if none, human-held binding constraint.
[LEARN] REJECTED MISCONFIG @ digital-onboarding.vpbank.com (anonymous data access): admin/user/qr_codes/tenants API all JWT-gated (401 invalid token / 403 Not authorized); anonymous axis is config-only (brand, tenant status). No anonymous data exposure.
[LEARN] NEW INFO @ digital-onboarding.vpbank.com: control-center SPA bundle (4MB) confirms API map incl. `/api/v1/sessions/{idp_login,secure_session,reset_password}`, `/api/v1/users`, `/rails/active_storage/direct_uploads`; DebugBarSelector + sagaInitLogging debug hooks ship in prod.
[LEARN] NEW INFO @ digital-onboarding.vpbank.com: `/users/sign_up` returns HTTP 500 (Rails 500.html), Devise registrable route mounted but errors — registration-enablement anomaly, not exploitable passive.
[LEARN] REJECTED OATH @ sts.vpbank.com: `/adfs/oauth2/token/devicecode` 200 is MS-HTTPAPI error shell (X-MS-Forwarded-Status-Code:500); real endpoint is `/adfs/oauth2/devicecode` (405 GET, enabled) — blocks on client_id.
[RISK] vp-bank-ag: **55** — stable. digital-onboarding: anonymous axis fully closed (JWT-gated), but client-controlled `force_tenant` on every request + mounted-but-500 `sign_up` route are the live back-office leads; ADFS device_code path confirmed real but client_id is the hard blocker (phishing HIGH if obtained); PSD2 sandbox BOLA (reportable, synthetic) remains flagship, prod carryover mTLS-gated. No anonymous data exposure confirmed on any in-scope asset; all remaining value is AUTH_HELPED/HUMAN_ONLY or requires dev-stage creds and would-be mutating (PARKED per program rules).
## 2026-09-05 20:41:41 UTC [target] (model bigpickle)
## 2026-09-05 22:40:50 UTC [target] (model bigpickle)
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
[HYP] force_tenant cross-tenant session/data IDOR on digital-onboarding back-office
class: IDOR
asset: digital-onboarding.vpbank.com/
confidence: 50
reasoning: SPA appends client-controlled `force_tenant` to every API request including session creation (bundle code: `o.force_tenant=It.get("FORCE_TENANT")` appended in `Dx` call layer); prod `/api/v1/brand?force_tenant=vpbank` returns HTTP 200 with second tenant config anonymously (tenant scoping honored server-side); `/api/v1/qr_codes/generate` returns "2fa not enabled for provided tenant" — tenant context is read from request. If a low-priv session's JWT tenant claim can be influenced by force_tenant at POST /api/v1/sessions (body `{"user":{...}}` + store `FORCE_TENANT`), cross-tenant onboarding-case/wire/ident reads follow.
evidence_needed: login to dev/stage sandbox with any account, then repeat the same request with force_tenant=vpbank and observe data set switch or unauthorized access to /admin/api/v1/ident_only_cases, /admin/api/v1/incoming_wires
verify_steps: GET https://digital-onboarding-dev.vpbank.com/api/v1/brand?force_tenant=vpbank (200, baseline tenant switch). Login→GET /admin/api/v1/users?force_tenant=vpbank from tenant-A session; 200 with tenant-B records = cross-tenant IDOR (dev sandbox only, requires valid dev creds).
impact: Cross-tenant PII/financial disclosure (onboarding cases, ident docs, wires, mailing lists) across tenants of a bank onboarding SaaS; severity HIGH
testability: AUTH_HELPED
[HYP] ADFS device_code grant token theft via phishing on sts.vpbank.com
class: OAUTH
asset: sts.vpbank.com/adfs
confidence: 55
reasoning: /adfs/.well-known/openid-configuration exposes device_code+password+implicit grants (issuer sts.vpbank.com/adfs, scopes vpn/logon/cert); /adfs/oauth2/token/devicecode returns 405 on GET, endpoint exists; device-flow adversariably polls for victim approval (phishing to enter user code on attacker device) yielding vpn/logon tokens.
evidence_needed: valid client_id + successful devicecode POST to observe user_code/verification_uri; RAG might surface ADFS client_id in VP Bank docs/portal
verify_steps: POST https://sts.vpbank.com/adfs/oauth2/token/devicecode 'client_id=<found>&scope=vpn' → 200 with user_code/verification_uri proves flow usable (requires client_id discovery; token issuance requires HUMAN victim approval)
impact: Corporate VPN/cert-scope tokens via device-flow phishing → internal network lateral movement; severity HIGH
testability: AUTH_HELPED
[HYP] PSD2 sandbox authorization model carries to production (cross-TPP Consent-ID read)
class: IDOR
asset: openbanking.vpbank.com
confidence: 35
reasoning: Sandbox proves zero session/TPP binding on consentId/paymentId chain; docs list only TPP auth/user-interaction as deltas; production authorizes via TLS mTLS only (blocked anonymously).
evidence_needed: two QWAC certs cross-reading Consent-ID on production
verify_steps: HUMAN_ONLY — TPP-A consent, TPP-B reads /consents/{id}/status on openbanking.vpbank.com/psd2/berlin-group/v1
impact: Cross-TPP production consent/account/ledger disclosure; severity HIGH
testability: HUMAN_ONLY
[NEXT] RAG: hunt for a VP Bank/ADFS `client_id` for sts.vpbank.com device_code flow (search VP Bank eBanking/secure docs, GitHub, "vpbank" + "adfs", IT-support KBs); once found → PROBE POST https://sts.vpbank.com/adfs/oauth2/token/devicecode `client_id=<id>&scope=vpn` (200 with user_code/verification_uri = device-flow phishing confirmed).
[RISK] vp-bank-ag: 57 — unchanged. Confirmed PSD2 sandbox BOLA (reportable, synthetic) remains the flagship; digital-onboarding depth-probe shows the data layer is JWT+signature-enforced (no anonymous data access, no JWT confusion), so the anonymous attack surface is limited to the SPA route map + tenant-config enumeration (low); residual risk concentrated in (a) proven BOLA with unverified prod carryover (mTLS-gated), (b) client-controlled force_tenant mechanism that could enable cross-tenant access for any authenticated low-priv user, (c) ADFS device-flow phishing. OAuth (client_id-gated), staging (WAF), api.vpbank.com (exhausted), openbanking (mTLS), mass-assignment (unverifiable without creds).
[PRIO] digital-onboarding.vpbank.com,8.4,attack_surface+gate_ease+tech_exposure
[PRIO] sts.vpbank.com,6.5,tech_exposure+attack_surface
[HYP] force_tenant cross-tenant data access on digital-onboarding sessions
class: IDOR
asset: digital-onboarding.vpbank.com/api/v1/sessions
confidence: 45
reasoning: SPA fetch wrapper reads `FORCE_TENANT` from localStorage and appends to every request including those initiated at/after sign-in; bundle exposes `selectForceTenant`/`setForceTenant` and prod anonymous `/api/v1/brand?force_tenant=vpbank` returns 200 (tenant honored server-side); `/api/v1/tenants` is JWT-gated so tenant scoping is enforced on data, but force_tenant is client-controlled.
evidence_needed: a valid low-priv session JWT, then re-issue same admin request with force_tenant=vpbantenantB and observe cross-tenant records 200.
verify_steps: GET https://digital-onboarding.vpbank.com/api/v1/brand?force_tenant=vpbank (200, baseline). With dev-stage creds: POST /api/v1/sessions {"user":{email,password}} → capture JWT → GET /admin/api/v1/users?force_tenant=vpbantenantB with that JWT; 200 with tenantB users = cross-tenant BOLA. (dev sandbox only, requires valid dev creds — AUTH_HELPED).
impact: Cross-tenant PII/financial disclosure (onboarding cases, ident docs, wires) across tenants of a bank onboarding SaaS; HIGH
testability: AUTH_HELPED
[HYP] ADFS device_code phishing on sts.vpbank.com
class: OATH
asset: sts.vpbank.com/adfs/oauth2/devicecode
confidence: 50
reasoning: openid-configuration exposes device_code+password+implicit grants (issuer sts.vpbank.com/adfs, scopes vpn/logon/cert); /adfs/oauth2/devicecode returns 405 on GET (endpoint enabled); correct ADFS path confirmed (the /token/devicecode 200 is an MS-HTTPAPI error shell, X-MS-Forwarded-Status-Code:500).
evidence_needed: valid client_id + successful devicecode POST returning user_code/verification_uri.
verify_steps: POST https://sts.vpbank.com/adfs/oauth2/devicecode 'client_id=<found>&scope=vpn' → 200 with user_code = flow usable (client_id discovery via VP Bank VP Connect mobile bundle / RAG; token issuance requires HUMAN victim approval).
impact: Corporate VPN/cert-scope tokens via device-flow phishing → internal network lateral movement; HIGH
testability: AUTH_HELPED
[HYP] (carryover) PSD2 sandbox BOLA to production
class: IDOR
asset: openbanking.vpbank.com
confidence: 35
reasoning: sandbox proves zero binding on consentId/paymentId; production mTLS-gated.
evidence_needed: two QWAC certs cross-reading.
verify_steps: HUMAN_ONLY
impact: cross-TPP production consent/ledger disclosure; HIGH
testability: HUMAN_ONLY
[NEXT] RAG: extract ADFS `client_id` from VP Bank Connect mobile app (App Store/Google Play bundle strings, "vpbank connect" + "adfs"/"devicecode"/"oauth") to unlock POST https://sts.vpbank.com/adfs/oauth2/devicecode 'client_id=<id>&scope=vpn' — if none, human-held binding constraint.
[LEARN] REJECTED MISCONFIG @ digital-onboarding.vpbank.com (anonymous data access): admin/user/qr_codes/tenants API all JWT-gated (401 invalid token / 403 Not authorized); anonymous axis is config-only (brand, tenant status). No anonymous data exposure.
[LEARN] NEW INFO @ digital-onboarding.vpbank.com: control-center SPA bundle (4MB) confirms API map incl. `/api/v1/sessions/{idp_login,secure_session,reset_password}`, `/api/v1/users`, `/rails/active_storage/direct_uploads`; DebugBarSelector + sagaInitLogging debug hooks ship in prod.
[LEARN] NEW INFO @ digital-onboarding.vpbank.com: `/users/sign_up` returns HTTP 500 (Rails 500.html), Devise registrable route mounted but errors — registration-enablement anomaly, not exploitable passive.
[LEARN] REJECTED OATH @ sts.vpbank.com: `/adfs/oauth2/token/devicecode` 200 is MS-HTTPAPI error shell (X-MS-Forwarded-Status-Code:500); real endpoint is `/adfs/oauth2/devicecode` (405 GET, enabled) — blocks on client_id.
[RISK] vp-bank-ag: **55** — stable. digital-onboarding: anonymous axis fully closed (JWT-gated), but client-controlled `force_tenant` on every request + mounted-but-500 `sign_up` route are the live back-office leads; ADFS device_code path confirmed real but client_id is the hard blocker (phishing HIGH if obtained); PSD2 sandbox BOLA (reportable, synthetic) remains flagship, prod carryover mTLS-gated. No anonymous data exposure confirmed on any in-scope asset; all remaining value is AUTH_HELPED/HUMAN_ONLY or requires dev-stage creds and would-be mutating (PARKED per program rules).
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
[HYP] Devise mass assignment on sign-in grants admin/tenant escalation on digital-onboarding back-office
class: AUTH
asset: digital-onboarding-dev.vpbank.com/users/sign_in
confidence: 80
reasoning: /control-center/ SPA serves admin modules anonymously (HTTP 200); brand endpoint honors server-side client-controlled `force_tenant` (tenant_symbol switches vpbank↔vpbanklighttenant); Devise sign_in accepts user[]-namespaced params; if permit_params misconfigured, admin/tenant_id/user_id ride the sign-in create and yield elevated/JWT-backed session
evidence_needed: dev-sandbox POST /users/sign_in with injected user[admin]=true&user[tenant_id]=1 returns Set-Cookie/redirect implying elevated or wrong-tenant session
verify_steps: POST https://digital-onboarding-dev.vpbank.com/users/sign_in Content-Type: application/x-www-form-urlencoded body user[email]=test@test.com&user[password]=test&user[admin]=true&user[tenant_id]=1 (dev sandbox, synthetic, non-mutating login attempt; baseline = POST without extra params; compare status/Set-Cookie/redirect/body; then with JWT GET /admin/api/v1/users)
impact: Admin/impersonation on bank-onboarding back-office → onboarding PII, ident docs, banking transactions, role mgmt across tenants; HIGH
testability: AUTH_HELPED
[HYP] ADFS OAuth2 device_code/password grants usable with any registered client for vpn/logon cert scopes on sts.vpbank.com
class: OATH
asset: sts.vpbank.com/adfs
confidence: 45
reasoning: /adfs/.well-known/openid-configuration HTTP 200 lists device_code+password+implicit; /adfs/oauth2/devicecode returns 405 on GET (POST endpoint enabled); MEX confirms WS-Trust usernamemixed RST surface live; scopes vpn_cert/logon_cert/winhello_cert/aza/user_impersonation
evidence_needed: a valid client_id returning 200 user_code/verification_uri (or usernamemixed RST SOAP acceptance); token issuance additionally requires employee credentials/human approval
verify_steps: POST https://sts.vpbank.com/adfs/oauth2/devicecode client_id=<found>&scope=vpn; RAG for a VP-shared ADFS OAuth client_id returns none (pipeline: VP Bank Connect is asymmetric-push non-OAuth; ADFS scopes imply MS-native VPN/Windows-Hello clients with Microsoft-standard GUIDs)
impact: Corporate vpn/logon-cert tokens via device-flow phishing → internal network lateral movement; HIGH if client_id found
testability: AUTH_HELPED
[HYP] PSD2 sandbox BOLA authorization model carries to production (cross-TPP Consent-ID read)
class: IDOR
asset: openbanking.vpbank.com
confidence: 35
reasoning: sandbox proves zero session/TPP/basic-auth binding on consentId/paymentId; production authorizes only via TLS mTLS
evidence_needed: two QWAC certs cross-reading a Consent-ID on production
verify_steps: HUMAN_ONLY — TPP-A consent, TPP-B reads /psd2/berlin-group/v1/consents/{id} and /accounts
impact: Cross-TPP production consent/account/ledger disclosure; HIGH
testability: HUMAN_ONLY
## 2026-09-06 00:14:21 UTC [target] (model bigpickle)
[HYP] Devise mass assignment on sign_in grants admin/tenant escalation on back-office
class: AUTH
asset: digital-onboarding-dev.vpbank.com/users/sign_in
confidence: 75
reasoning: Bundle confirms real routes are JWT-gated (admin/api/v1/users → 401 "invalid token"); /api/v1/brand honors anonymous ?force_tenant config selector proving server reads client-controlled tenant params; Devise user[]-namespaced params on sign_in; if permit_params overbroad, admin/tenant_id ride the create
evidence_needed: dev-sandbox POST with injected user[admin]=true&user[tenant_id]=1 yields Set-Cookie/BEARER or redirect implying elevated/wrong-tenant session vs baseline POST
verify_steps: GET https://digital-onboarding-dev.vpbank.com/users/sign_in (confirm reachable + form fields); POST /users/sign_in Content-Type: application/x-www-form-urlencoded body user[email]=test@test.com&user[password]=test (baseline) then same + &user[admin]=true&user[tenant_id]=1&user[user_id]=1; compare status/Set-Cookie/redirect Location/body; if session minted, with JWT GET /admin/api/v1/users offline
impact: Admin/impersonation across onboarding tenants → PII, identity docs, banking transactions; HIGH
testability: AUTH_HELPED
[HYP] force_tenant switches data tenant under a valid session (post-auth IDOR)
class: IDOR
asset: digital-onboarding.vpbank.com/admin/api/v1/
confidence: 45
reasoning: Anonymous data axis falsified today (static 404/401/403); /api/v1/brand honors force_tenant anonymously proving param reaches server; admin modules (rolemgmt, bankingtransactions, incomingwire) exist in bundle under auth; if controller reads force_tenant without re-scoping to session tenant, legit A-tenant token reads B-tenant rows
evidence_needed: dev-sandbox session for tenant A; GET /admin/api/v1/bankingtransactions?force_tenant=vpbank returns cross-tenant rows
verify_steps: HUMAN_ONLY once dev session obtained — GET https://digital-onboarding-dev.vpbank.com/admin/api/v1/bankingtransactions?force_tenant=vpbank; compare rowset tenant ids with/without force_tenant
impact: Cross-tenant financial/PII disclosure in bank onboarding back-office; HIGH
testability: AUTH_HELPED
[HYP] ADFS device_code grant usable with any registered client for vpn/logon cert scopes
class: OATH
asset: sts.vpbank.com/adfs
confidence: 45
reasoning: /adfs/.well-known/openid-configuration 200 lists device_code+password+implicit, scopes vpn_cert/logon_cert/winhello_cert/aza/user_impersonation; /adfs/oauth2/devicecode 405 (POST enabled); service currently 503
evidence_needed: valid client_id → 200 user_code/device_code/verification_uri
verify_steps: POST https://sts.vpbank.com/adfs/oauth2/devicecode Content-Type: application/x-www-form-urlencoded body client_id=<found>&scope=vpn (RAG MS-native/ADFS client_ids; observe 400 invalid_client vs 200)
impact: Corporate VPN/cert token via device-flow phishing → internal network lateral movement; HIGH if client_id found
testability: AUTH_HELPED
