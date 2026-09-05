# Hypotheses (ranked)

## RANKED HYPOTHESES 2026-09-02 21:53:27 UTC

## RANKED HYPOTHESES 2026-09-02 23:50:48 UTC

## RANKED HYPOTHESES 2026-09-03 02:36:22 UTC

## RANKED HYPOTHESES 2026-09-03 07:29:02 UTC

## RANKED HYPOTHESES 2026-09-03 12:18:41 UTC

## RANKED HYPOTHESES 2026-09-03 16:33:58 UTC
- [70] api.vpbank.com: API versioning & debug endpoints on api.vpbank.com (from art/lead_nemotron3.txt)
- [0] <host/endpoint>: <title> (from art/lead_bigpickle.txt)
- NEXT(hypotheses-nemotron3.txt): PROBE: GET https://api.vpbank.com/v1 (HEAD first, then GET if 2xx/3xx/401/403)
- LEARN: REJECTED MISCONFIG @ api.vpbank.com (subdomain takeover): ellb.ch is active Swiss load balancer, not a decommissioned cloud provider tenant

## RANKED HYPOTHESES 2026-09-03 19:30:16 UTC
- [60] api.vpbank.com: API versioning and debug endpoints on api.vpbank.com (from art/lead_bigpickle.txt)
- [55] api.vpbank.com: Layer7 API Gateway policy bypass via malformed requests (from art/lead_nemotron3.txt)
- NEXT(hypotheses-bigpickle.txt): PROBE: HEAD https://api.vpbank.com/v1, then GET if 2xx/3xx/401/403
- NEXT(hypotheses-nemotron3.txt): PROBE: GET https://api.vpbank.com/ with headers: Accept: application/xml, X-Forwarded-For: 127.0.0.1, Content-Type: application/xml; then POST https://api.vpban
- LEARN: REJECTED MISCONFIG @ api.vpbank.com (subdomain takeover): ellb.ch is active Swiss load balancer, not a decommissioned cloud provider tenant
- LEARN: REJECTED MISCONFIG @ api.vpbank.com (API versioning/debug endpoints): All tested paths (/v1,/v2,/swagger.json,/openapi.json,/actuator/health,/beta,/internal,/.w
- LEARN: REJECTED OAUTH @ vpbank.com (OIDC discovery): .well-known/openid-configuration redirects to www.vpbank.com which returns 303 error page - no OIDC metadata expos
- LEARN: ACCEPTED MISCONFIG @ api.vpbank.com (Layer7 gateway error handling): Gateway returns structured JSON errors with request IDs - potential info leak vector

## RANKED HYPOTHESES 2026-09-03 21:57:32 UTC
- [65] vpbank-dev.com: vpbank-dev.com / vpbank-stage.com are production-trusted staging environments reachable publicly (from art/lead_bigpickle.txt)
- [55] api.vpbank.com: Layer7 API Gateway policy bypass via malformed requests (from art/lead_nemotron3.txt)
- NEXT(hypotheses-bigpickle.txt): PROBE: GET https://www.vpbank-dev.com/user/login and GET https://www.vpbank-dev.com/portal/api/language/en ; then GET https://www.vpbank-stage.com/en to confirm
- NEXT(hypotheses-nemotron3.txt): PROBE: GET https://api.vpbank.com/ with headers: Accept: application/xml, X-Forwarded-For: 127.0.0.1, Content-Type: application/xml; then POST https://api.vpban
- LEARN: ACCEPTED MISCONFIG @ vpbank-dev.com/vpbank-stage.com: production CSP trusts these domains; publicly reachable Apache servers behind maintenance redirect - NEW i
- LEARN: REJECTED MISCONFIG @ api.vpbank.com (API versioning/debug endpoints): All tested paths (/v1,/v2,/swagger.json,/openapi.json,/actuator/health,/beta,/internal,/.w
- LEARN: REJECTED OAUTH @ vpbank.com (OIDC discovery): .well-known/openid-configuration redirects to www.vpbank.com which returns 303/400 error page - no OIDC metadata e
- LEARN: REJECTED MISCONFIG @ api.vpbank.com (subdomain takeover): ellb.ch is active Swiss load balancer, not a decommissioned cloud provider tenant
- LEARN: ACCEPTED MISCONFIG @ api.vpbank.com (Layer7 gateway error handling): Gateway returns structured JSON errors with request IDs - potential info leak vector

## RANKED HYPOTHESES 2026-09-03 23:51:14 UTC
- [55] api.vpbank.com: Layer7 API Gateway policy bypass via malformed requests on api.vpbank.com (from art/lead_bigpickle.txt)
- NEXT(hypotheses-bigpickle.txt): PROBE: GET https://api.vpbank.com/ with headers: Accept: application/xml, X-Forwarded-For: 127.0.0.1, Content-Type: application/xml
- LEARN: REJECTED MISCONFIG @ vpbank-dev.com/vpbank-stage.com: WAF intercepts ALL paths, returns maintenance page - no application content accessible
- LEARN: NEW INFO: WAF version 2.3.0_20260324 exposed in maintenance page responses

## RANKED HYPOTHESES 2026-09-04 02:38:02 UTC
- [55] api.vpbank.com: Layer7 API Gateway policy bypass via malformed requests on api.vpbank.com (from art/lead_bigpickle.txt)
- [50] www.vpbank.com: OAuth/OIDC redirect_uri validation bypass on www.vpbank.com (from art/lead_nemotron3.txt)
- NEXT(hypotheses-nemotron3.txt): PROBE: GET https://api.vpbank.com/ with headers: Accept: application/xml, X-Forwarded-For: 127.0.0.1, Content-Type: application/xml; then POST https://api.vpban
- LEARN: REJECTED MISCONFIG @ vpbank-dev.com/vpbank-stage.com (staging exposure): WAF 2.3.0_20260324 intercepts ALL paths — no application content accessible past mainte
- LEARN: REJECTED MISCONFIG @ vpbank-dev.com/vpbank-stage.com (WAF version disclosure): version string 2.3.0_20260324 present in body but LOW severity, not actionable
- LEARN: REJECTED MISCONFIG @ api.vpbank.com (subdomain takeover): ellb.ch is active Swiss load balancer, not a decommissioned cloud provider tenant
- LEARN: REJECTED MISCONFIG @ api.vpbank.com (API versioning/debug endpoints): All tested paths (/v1,/v2,/swagger.json,/openapi.json,/actuator/health,/beta,/internal,/.w
- LEARN: REJECTED OAUTH @ vpbank.com (OIDC discovery): .well-known/openid-configuration redirects to www.vpbank.com which returns 303/400 error page - no OIDC metadata e
- LEARN: REJECTED MISCONFIG @ vpbank-dev.com/vpbank-stage.com: WAF intercepts ALL paths, returns maintenance page - no application content accessible
- LEARN: ACCEPTED MISCONFIG @ api.vpbank.com (Layer7 gateway error handling): Gateway returns structured JSON errors with request IDs - potential info leak vector
- LEARN: ACCEPTED MISCONFIG @ vpbank-dev.com/vpbank-stage.com: production CSP trusts these domains; publicly reachable Apache servers behind maintenance redirect - NEW i
- LEARN: NEW INFO: WAF version 2.3.0_20260324 exposed in maintenance page responses

## RANKED HYPOTHESES 2026-09-04 07:28:45 UTC
- [45] api.vpbank.com: SSRF via Host header routing manipulation on api.vpbank.com (from art/lead_bigpickle.txt)
- [35] www.vpbank.com: OAuth/OIDC redirect_uri validation bypass on www.vpbank.com (from art/lead_nemotron3.txt)
- NEXT(hypotheses-bigpickle.txt): PROBE: GET https://api.vpbank.com/ with `Host: 169.254.169.254` header (SSRF test, untested verify_step); then GET https://api.vpbank.com/ with `Host: localhost
- NEXT(hypotheses-nemotron3.txt): RAG: Search for VP Bank AG OAuth client_id references in public GitHub, npm, mobile app bundles, or JS source maps to obtain valid client_id for redirect_uri te
- LEARN: REJECTED MISCONFIG @ api.vpbank.com (gateway policy bypass via XML/X-Forwarded-For): verify_steps executed — GET+POST with Accept:application/xml + X-Forwarded-
- LEARN: ACCEPTED OAUTH @ www.vpbank.com (redirect_uri bypass): endpoint exists at /oauth/authorize; test returned HTTP 400 (ambiguous); HOLD pending client_id enumerati
- LEARN: ACCEPTED MISCONFIG @ api.vpbank.com (SSRF via Host header): proposed but verify_steps NOT YET EXECUTED; remaining high-value test.
- LEARN: REJECTED MISCONFIG @ api.vpbank.com (Layer7 policy bypass): All malformed request probes (XML, SOAP, routing headers) return identical HTTP 500 JSON — no policy
- LEARN: REJECTED SSRF @ api.vpbank.com (routing header manipulation): Host/X-Forwarded-Host/X-Forwarded-For to 169.254.169.254, localhost, 10.0.0.1 all return identical
- LEARN: REJECTED OAUTH @ www.vpbank.com (OIDC discovery): .well-known/openid-configuration returns 303 to error page — no metadata exposed
- LEARN: ACCEPTED MISCONFIG @ api.vpbank.com (Layer7 gateway error handling): Structured JSON errors with request IDs confirmed — info leak vector but low severity

## RANKED HYPOTHESES 2026-09-04 12:20:53 UTC
- [45] api.vpbank.com: SSRF via Host header backend routing on api.vpbank.com (from art/lead_nemotron3.txt)
- NEXT(hypotheses-nemotron3.txt): RAG: Search GitHub (code search), npm packages, mobile app bundles (iOS/Android), and JS source maps for "vpbank", "VP Bank AG", "vpbank.com" OAuth client_id re
- LEARN: REJECTED MISCONFIG @ api.vpbank.com (Layer7 policy bypass): All malformed request probes (XML, SOAP, routing headers) return identical HTTP 500 JSON — no policy
- LEARN: REJECTED SSRF @ api.vpbank.com (routing header manipulation): Host/X-Forwarded-Host/X-Forwarded-For to 169.254.169.254, localhost, 10.0.0.1 all return identical
- LEARN: REJECTED OAUTH @ www.vpbank.com (OIDC discovery): .well-known/openid-configuration returns 303 to error page — no metadata exposed
- LEARN: REJECTED MISCONFIG @ vpbank-dev.com/vpbank-stage.com (staging exposure): WAF 2.3.0_20260324 intercepts ALL paths — no application content accessible
- LEARN: ACCEPTED MISCONFIG @ api.vpbank.com (Layer7 gateway error handling): Structured JSON errors with requestIds confirmed — info leak vector but low severity
- LEARN: ACCEPTED MISCONFIG @ vpbank-dev.com/vpbank-stage.com: Production CSP trusts these domains; publicly reachable Apache servers behind maintenance redirect — in-sc
- LEARN: REJECTED SSRF @ api.vpbank.com (Host header routing): verify_steps executed — Host:169.254.169.254, Host:localhost, X-Forwarded-Host:169.254.169.254 all returne
- LEARN: REJECTED OAUTH @ www.vpbank.com (redirect_uri bypass): JS bundles contain no OAuth client_id (only Usercentrics widget clientWid); /oauth/authorize always 303→e
- LEARN: NEW INFO @ www.vpbank.com/portal/api/: Full LitElement SPA served (403 with body) — separate app from Drupal; POST /portal/api/language/:language sets portal-la
- LEARN: REJECTED MISCONFIG @ www.vpbank.com (portal API access): /portal/api/ returns 403 but serves complete SPA shell; /portal/api/health and /portal/api/status retur

## RANKED HYPOTHESES 2026-09-04 16:37:23 UTC
- [45] www.vpbank.com: OAuth client_id discovery via external artifact enumeration (from art/lead_bigpickle.txt)
- [40] www.vpbank.com: OAuth redirect_uri validation bypass via subdomain confusion on www.vpbank.com (from art/lead_nemotron3.txt)
- NEXT(hypotheses-bigpickle.txt): RAG: Search iOS App Store and Google Play Store for "VP Bank" or "VP Bank AG" mobile applications; extract and analyze OAuth configuration from app bundles to d
- NEXT(hypotheses-nemotron3.txt): RAG: Search GitHub (code search), npm packages, mobile app bundles (iOS/Android), and JS source maps for "vpbank", "VP Bank AG", "vpbank.com" OAuth client_id re
- LEARN: REJECTED SSRF @ api.vpbank.com (Host header routing): verify_steps executed — Host:169.254.169.254, Host:localhost, X-Forwarded-Host:169.254.169.254 all returne
- LEARN: REJECTED OAUTH @ www.vpbank.com (redirect_uri bypass): JS bundles contain no OAuth client_id (only Usercentrics widget clientWid); /oauth/authorize always 303→e
- LEARN: NEW INFO @ www.vpbank.com/portal/api/: Full LitElement SPA served (403 with body) — separate app from Drupal; POST /portal/api/language/:language sets portal-la
- LEARN: REJECTED MISCONFIG @ www.vpbank.com (portal API access): /portal/api/ returns 403 but serves complete SPA shell; /portal/api/health and /portal/api/status retur
- LEARN: ACCEPTED MISCONFIG @ api.vpbank.com (Layer7 gateway error handling): Structured JSON errors with requestIds confirmed — info leak vector but low severity
- LEARN: REJECTED SSRF @ api.vpbank.com (Host header routing): verify_steps executed — Host:169.254.169.254, Host:localhost, X-Forwarded-Host:169.254.169.254 all returne
- LEARN: REJECTED MISCONFIG @ api.vpbank.com (Layer7 policy bypass): All malformed request probes (XML, SOAP, routing headers) return identical HTTP 500 JSON — no policy
- LEARN: REJECTED OAUTH @ www.vpbank.com (OIDC discovery): .well-known/openid-configuration returns 303 to error page — no metadata exposed
- LEARN: REJECTED OAUTH @ www.vpbank.com (redirect_uri bypass): JS bundles contain no OAuth client_id (only Usercentrics widget clientWid); /oauth/authorize always 303→e
- LEARN: REJECTED MISCONFIG @ vpbank-dev.com/vpbank-stage.com (staging exposure): WAF 2.3.0_20260324 intercepts ALL paths — no application content accessible
- LEARN: REJECTED MISCONFIG @ www.vpbank.com (portal API access): /portal/api/ returns 403 but serves complete SPA shell; /portal/api/health and /portal/api/status retur
- LEARN: ACCEPTED MISCONFIG @ api.vpbank.com (Layer7 gateway error handling): Structured JSON errors with requestIds confirmed — info leak vector but low severity
- LEARN: ACCEPTED MISCONFIG @ vpbank-dev.com/vpbank-stage.com: Production CSP trusts these domains; publicly reachable Apache servers behind maintenance redirect — in-sc
- LEARN: NEW INFO @ www.vpbank.com/portal/api/: Full LitElement SPA served (403 with body) — separate app from Drupal; POST /portal/api/language/:language sets portal-la

## RANKED HYPOTHESES 2026-09-04 19:13:22 UTC
- [45] www.vpbank.com: OAuth client_id enumeration via external artifact mining for redirect_uri bypass (from art/lead_nemotron3.txt)
- [45] www.vpbank.com: OAuth client_id discovery via external artifact enumeration (from art/lead_bigpickle.txt)
- NEXT(hypotheses-bigpickle.txt): RAG: Search iOS App Store and Google Play Store for "VP Bank" or "VP Bank AG" mobile applications; extract and analyze OAuth configuration from app bundles to d
- NEXT(hypotheses-nemotron3.txt): RAG: Search GitHub (code search), npm packages, mobile app bundles (iOS/Android), and JS source maps for "vpbank", "VP Bank AG", "vpbank.com" OAuth client_id re
- LEARN: REJECTED SSRF @ api.vpbank.com (Host header routing): verify_steps executed — Host:169.254.169.254, Host:localhost, X-Forwarded-Host:169.254.169.254 all returne
- LEARN: REJECTED OAUTH @ www.vpbank.com (redirect_uri bypass): JS bundles contain no OAuth client_id (only Usercentrics widget clientWid); /oauth/authorize always 303→e
- LEARN: NEW INFO @ www.vpbank.com/portal/api/: Full LitElement SPA served (403 with body) — separate app from Drupal; POST /portal/api/language/:language sets portal-la
- LEARN: REJECTED MISCONFIG @ www.vpbank.com (portal API access): /portal/api/ returns 403 but serves complete SPA shell; /portal/api/health and /portal/api/status retur
- LEARN: ACCEPTED MISCONFIG @ api.vpbank.com (Layer7 gateway error handling): Structured JSON errors with requestIds confirmed — info leak vector but low severity
- LEARN: REJECTED SSRF @ api.vpbank.com (Host header routing): verify_steps executed — Host:169.254.169.254, Host:localhost, X-Forwarded-Host:169.254.169.254 all returne
- LEARN: REJECTED MISCONFIG @ api.vpbank.com (Layer7 policy bypass): All malformed request probes (XML, SOAP, routing headers) return identical HTTP 500 JSON — no policy
- LEARN: REJECTED OAUTH @ www.vpbank.com (OIDC discovery): .well-known/openid-configuration returns 303 to error page — no metadata exposed
- LEARN: REJECTED OAUTH @ www.vpbank.com (redirect_uri bypass): JS bundles contain no OAuth client_id (only Usercentrics widget clientWid); /oauth/authorize always 303→e
- LEARN: REJECTED MISCONFIG @ vpbank-dev.com/vpbank-stage.com (staging exposure): WAF 2.3.0_20260324 intercepts ALL paths — no application content accessible
- LEARN: REJECTED MISCONFIG @ www.vpbank.com (portal API access): /portal/api/ returns 403 but serves complete SPA shell; /portal/api/health and /portal/api/status retur
- LEARN: ACCEPTED MISCONFIG @ api.vpbank.com (Layer7 gateway error handling): Structured JSON errors with requestIds confirmed — info leak vector but low severity
- LEARN: ACCEPTED MISCONFIG @ vpbank-dev.com/vpbank-stage.com: Production CSP trusts these domains; publicly reachable Apache servers behind maintenance redirect — in-sc
- LEARN: NEW INFO @ www.vpbank.com/portal/api/: Full LitElement SPA served (403 with body) — separate app from Drupal; POST /portal/api/language/:language sets portal-la
- LEARN: REJECTED MISCONFIG @ api.vpbank.com (subdomain takeover): ellb.ch is active Swiss load balancer, not a decommissioned cloud provider tenant
- LEARN: REJECTED SSRF @ api.vpbank.com (Host header routing): verify_steps executed — Host:169.254.169.254, Host:localhost, X-Forwarded-Host:169.254.169.254 all returne
- LEARN: REJECTED OAUTH @ www.vpbank.com (redirect_uri bypass): JS bundles contain no OAuth client_id (only Usercentrics widget clientWid); /oauth/authorize always 303→e
- LEARN: NEW INFO @ www.vpbank.com/portal/api/: Full LitElement SPA served (403 with body) — separate app from Drupal; POST /portal/api/language/:language sets portal-la
- LEARN: REJECTED MISCONFIG @ www.vpbank.com (portal API access): /portal/api/ returns 403 but serves complete SPA shell; /portal/api/health and /portal/api/status retur
- LEARN: ACCEPTED MISCONFIG @ api.vpbank.com (Layer7 gateway error handling): Structured JSON errors with requestIds confirmed — info leak vector but low severity
- LEARN: REJECTED SSRF @ api.vpbank.com (Host header routing): verify_steps executed — Host:169.254.169.254, Host:localhost, X-Forwarded-Host:169.254.169.254 all returne
- LEARN: REJECTED MISCONFIG @ api.vpbank.com (Layer7 policy bypass): All malformed request probes (XML, SOAP, routing headers) return identical HTTP 500 JSON — no policy
- LEARN: REJECTED OAUTH @ www.vpbank.com (OIDC discovery): .well-known/openid-configuration returns 303 to error page — no metadata exposed
- LEARN: REJECTED OAUTH @ www.vpbank.com (redirect_uri bypass): JS bundles contain no OAuth client_id (only Usercentrics widget clientWid); /oauth/authorize always 303→e
- LEARN: REJECTED MISCONFIG @ vpbank-dev.com/vpbank-stage.com (staging exposure): WAF 2.3.0_20260324 intercepts ALL paths — no application content accessible
- LEARN: REJECTED MISCONFIG @ www.vpbank.com (portal API access): /portal/api/ returns 403 but serves complete SPA shell; /portal/api/health and /portal/api/status retur
- LEARN: ACCEPTED MISCONFIG @ api.vpbank.com (Layer7 gateway error handling): Structured JSON errors with requestIds confirmed — info leak vector but low severity
- LEARN: ACCEPTED MISCONFIG @ vpbank-dev.com/vpbank-stage.com: Production CSP trusts these domains; publicly reachable Apache servers behind maintenance redirect — in-sc
- LEARN: NEW INFO @ www.vpbank.com/portal/api/: Full LitElement SPA served (403 with body) — separate app from Drupal; POST /portal/api/language/:language sets portal-la
- LEARN: OAuth client_id hunt via App Store/Google/GitHub/npm surfaced only VP Bank Vietnam (different entity) + unrelated SDKs — no Liechtenstein client_id.
- LEARN: REJECTED SSRF @ api.vpbank.com (Host header routing): verify_steps executed — Host:169.254.169.254, Host:localhost, X-Forwarded-Host:169.254.169.254 all returne
- LEARN: REJECTED MISCONFIG @ api.vpbank.com (Layer7 policy bypass): All malformed request probes (XML, SOAP, routing headers) return identical HTTP 500 JSON — no policy
- LEARN: REJECTED OAUTH @ www.vpbank.com (OIDC discovery): .well-known/openid-configuration returns 303 to error page — no metadata exposed
- LEARN: REJECTED OAUTH @ www.vpbank.com (redirect_uri bypass): JS bundles contain no OAuth client_id (only Usercentrics widget clientWid); /oauth/authorize always 303→e
- LEARN: REJECTED MISCONFIG @ vpbank-dev.com/vpbank-stage.com (staging exposure): WAF 2.3.0_20260324 intercepts ALL paths — no application content accessible
- LEARN: REJECTED MISCONFIG @ www.vpbank.com (portal API access): /portal/api/ returns 403 but serves complete SPA shell; /portal/api/health and /portal/api/status retur
- LEARN: ACCEPTED MISCONFIG @ api.vpbank.com (Layer7 gateway error handling): Structured JSON errors with requestIds confirmed — info leak vector but low severity
- LEARN: ACCEPTED MISCONFIG @ vpbank-dev.com/vpbank-stage.com: Production CSP trusts these domains; publicly reachable Apache servers behind maintenance redirect — in-sc
- LEARN: NEW INFO @ www.vpbank.com/portal/api/: Full LitElement SPA served (403 with body) — separate app from Drupal; POST /portal/api/language/:language sets portal-la

## RANKED HYPOTHESES 2026-09-04 21:38:33 UTC
- [75] developer.vpbank.com: PSD2 Sandbox BOLA/IDOR via consentId/paymentId manipulation (from art/lead_nemotron3.txt)
- NEXT(hypotheses-bigpickle.txt): RAG: search GitHub/public web for "openbanking.vpbank.com", "vpbank psd2 scaRedirect", "vpbank.com/oauth/authorize" PSD2 client_id/authorize_url; if a PSD2-scop
- NEXT(hypotheses-nemotron3.txt): PROBE: Test BOLA/IDOR on developer.vpbank.com PSD2 sandbox — create two basic auth contexts (userA:test, userB:test), generate consentId_A and consentId_B, then
- LEARN: ACCEPTED MISCONFIG @ developer.vpbank.com: PSD2 Berlin Group sandbox API reachable HTTP 200 without client cert or basic auth (documented-open since 2024-05); o
- LEARN: NEW INFO @ openbanking.vpbank.com (193.222.70.154): discovered via TLS cert CN=openbanking.vpbank.com; production PSD2 ASPSP, mTLS "certificate required" at TLS
- LEARN: NEW INFO @ developer.vpbank.com: /psd2/sandbox/certificate/verify_certificate also mTLS-gated; OpenAPI spec self-labels server "PSD2 production server" while op
- LEARN: REJECTED MISCONFIG @ developer.vpbank.com (Drupal admin on PSD2 vhost): /user/login, /admin, /sites/default/files/* all HTTP 404 — dedicated docroot.
- LEARN: REJECTED MISCONFIG @ openbanking.vpbank.com (anonymous API access): mTLS enforced, anonymous rejected — expected production posture, not a flaw.
- LEARN: ACCEPTED MISCONFIG @ developer.vpbank.com (PSD2 Developer Portal exposure): Live PSD2 sandbox API with full OpenAPI spec, functional endpoints (consents, accoun
- LEARN: REJECTED MISCONFIG @ vpbank-dev.com/vpbank-stage.com (staging exposure): WAF 2.3.0_20260324 intercepts ALL paths — no application content accessible past mainte
- LEARN: REJECTED SSRF @ api.vpbank.com (Host header routing): verify_steps executed — Host:169.254.169.254, Host:localhost, X-Forwarded-Host:169.254.169.254 all returne
- LEARN: REJECTED OAUTH @ www.vpbank.com (redirect_uri bypass): JS bundles contain no OAuth client_id (only Usercentrics widget clientWid); /oauth/authorize always 303→e
- LEARN: REJECTED MISCONFIG @ www.vpbank.com (portal API access): /portal/api/ returns 403 but serves complete SPA shell; /portal/api/health and /portal/api/status retur
- LEARN: ACCEPTED MISCONFIG @ api.vpbank.com (Layer7 gateway error handling): Structured JSON errors with requestIds confirmed — info leak vector but low severity.
- LEARN: NEW INFO @ developer.vpbank.com: PSD2 sandbox uses basic auth (test:test works), X-Request-ID last digit controls state encoding (1=RCVD, 5=ACSC for payments), 

## RANKED HYPOTHESES 2026-09-04 23:23:03 UTC
- [80] developer.vpbank.com/psd2/berlin-group/v1: PSD2 sandbox BOLA: consent/account/ledger/payment resources readable cross-session via bearer consentId (from art/lead_bigpickle.txt)
- [75] developer.vpbank.com: PSD2 Sandbox BOLA/IDOR via consentId/paymentId manipulation (from art/lead_nemotron3.txt)
- NEXT(hypotheses-bigpickle.txt): SCAN: passive CT enumeration via crt.sh (`%.vpbank.com`, `%.vpbank.li`) to catalog PSD2/openbanking/statistics subdomains, then anonymous reachability check of 
- NEXT(hypotheses-nemotron3.txt): PROBE: Test BOLA/IDOR on developer.vpbank.com PSD2 sandbox — create two basic auth contexts (userA:test, userB:test), generate consentId_A and consentId_B, then
- LEARN: ACCEPTED IDOR @ developer.vpbank.com (PSD2 sandbox BOLA): verify_steps EXECUTED in official test sandbox (synthetic data) — consent 6b517824-e5af-4202-b9b0-7f48
- LEARN: REJECTED BUSLOGIC @ developer.vpbank.com (X-Request-ID state encoding): documented deterministic client-driven state for the sandbox; observed ACSC from X-Reque
- LEARN: REJECTED OAUTH @ www.vpbank.com/oauth/authorize: RAG GitHub/public-web surfaces only VP Bank Vietnam (separate entity) and generic PSD2 frameworks; downloaded s
- LEARN: REJECTED MISCONFIG @ www.vpbank.com/developer.vpbank.com (PSD2 statistics pages): /psd2-statistics, /psd2-statistics/, /psd2/statistics/ all HTTP 404 — no anony
- LEARN: ACCEPTED MISCONFIG @ developer.vpbank.com (PSD2 Developer Portal exposure): Live PSD2 sandbox API with full OpenAPI spec, functional endpoints (consents, accoun
- LEARN: REJECTED MISCONFIG @ vpbank-dev.com/vpbank-stage.com (staging exposure): WAF 2.3.0_20260324 intercepts ALL paths — no application content accessible past mainte
- LEARN: REJECTED SSRF @ api.vpbank.com (Host header routing): verify_steps executed — Host:169.254.169.254, Host:localhost, X-Forwarded-Host:169.254.169.254 all returne
- LEARN: REJECTED OAUTH @ www.vpbank.com (redirect_uri bypass): JS bundles contain no OAuth client_id (only Usercentrics widget clientWid); /oauth/authorize always 303→e
- LEARN: REJECTED MISCONFIG @ www.vpbank.com (portal API access): /portal/api/ returns 403 but serves complete SPA shell; /portal/api/health and /portal/api/status retur
- LEARN: ACCEPTED MISCONFIG @ api.vpbank.com (Layer7 gateway error handling): Structured JSON errors with requestIds confirmed — info leak vector but low severity.
- LEARN: NEW INFO @ developer.vpbank.com: PSD2 sandbox uses basic auth (test:test works), X-Request-ID last digit controls state encoding (1=RCVD, 5=ACSC for payments), 

## RANKED HYPOTHESES 2026-09-05 01:11:22 UTC
- [80] developer.vpbank.com/psd2/berlin-group/v1: PSD2 sandbox BOLA: consent/account/ledger/payment resources readable cross-session via bearer consentId (from art/lead_nemotron3.txt)
- [45] digital-onboarding.vpbank.com/users/sign_in: Mass assignment on Devise sign-in grants admin/impersonation on onboarding back-office (from art/lead_bigpickle.txt)
- NEXT(hypotheses-bigpickle.txt): PROBE: on synthetic dev tenant, anonymous-authz gate differential — GET https://digital-onboarding-dev.vpbank.com/admin/api/v1/users and GET https://digital-onb
- NEXT(hypotheses-nemotron3.txt): SCAN: passive CT enumeration via crt.sh (`%.vpbank.com`, `%.vpbank.li`) to catalog PSD2/openbanking/statistics subdomains, then anonymous reachability check of 
- LEARN: ACCEPTED MISCONFIG @ digital-onboarding.vpbank.com: Live multi-tenant bank-onboarding/back-office SaaS ('US', Rails+Devise) on off-net hosting (89.163.182.69/.2
- LEARN: REJECTED MISCONFIG @ api-prep.vpbank.com: Same Layer7 dead-end as api.vpbank.com (SCS-Request-ID, INVALID_REQUEST_RESOURCE) — pre-prod gateway clone, no new sur
- LEARN: REJECTED MISCONFIG @ designsystem.vpbank.com: CNAME→vpb-design-system.netlify.app serves 200 — hosted design system is active, no subdomain takeover.
- LEARN: NEW INFO @ sts.vpbank.com: ADFS OIDC metadata 200 (issuer sts.vpbank.com/adfs, device_code/password grants, vpn/logon cert scopes) — corporate IdP, parked.
- LEARN: ACCEPTED IDOR @ developer.vpbank.com (PSD2 sandbox BOLA): verify_steps EXECUTED in official test sandbox (synthetic data) — consent 6b517824-e5af-4202-b9b0-7f48
- LEARN: REJECTED BUSLOGIC @ developer.vpbank.com (X-Request-ID state encoding): documented deterministic client-driven state for the sandbox; observed ACSC from X-Reque
- LEARN: REJECTED OAUTH @ www.vpbank.com/oauth/authorize: RAG GitHub/public-web surfaces only VP Bank Vietnam (separate entity) and generic PSD2 frameworks; downloaded s
- LEARN: REJECTED MISCONFIG @ www.vpbank.com/developer.vpbank.com (PSD2 statistics pages): /psd2-statistics, /psd2-statistics/, /psd2/statistics/ all HTTP 404 — no anony
- LEARN: REJECTED SSRF @ api.vpbank.com (Host header routing): verify_steps executed — Host:169.254.169.254, Host:localhost, X-Forwarded-Host:169.254.169.254 all returne
- LEARN: REJECTED MISCONFIG @ api.vpbank.com (Layer7 policy bypass): All malformed request probes (XML, SOAP, routing headers) return identical HTTP 500 JSON — no policy
- LEARN: REJECTED MISCONFIG @ vpbank-dev.com/vpbank-stage.com (staging exposure): WAF 2.3.0_20260324 intercepts ALL paths — no application content accessible past mainte
- LEARN: REJECTED MISCONFIG @ www.vpbank.com (portal API access): /portal/api/ returns 403 but serves complete SPA shell; /portal/api/health and /portal/api/status retur
- LEARN: ACCEPTED MISCONFIG @ api.vpbank.com (Layer7 gateway error handling): Structured JSON errors with requestIds confirmed — info leak vector but low severity.
- LEARN: ACCEPTED MISCONFIG @ developer.vpbank.com (PSD2 Developer Portal exposure): Live PSD2 sandbox API with full OpenAPI spec, functional endpoints (consents, accoun
- LEARN: NEW INFO @ openbanking.vpbank.com (193.222.70.154): discovered via TLS cert CN=openbanking.vpbank.com; production PSD2 ASPSP, mTLS "certificate required" at TLS

## RANKED HYPOTHESES 2026-09-05 05:50:38 UTC
- [45] www.vpbank.com: OAuth client_id discovery via external artifact enumeration (from art/lead_bigpickle.txt)
- [40] openbanking.vpbank.com: PSD2 sandbox BOLA carries to production via shared consent-authorization code (from art/lead_nemotron3.txt)
- NEXT(hypotheses-bigpickle.txt): RAG: Search iOS App Store and Google Play Store for "VP Bank" or "VP Bank AG" mobile applications; extract and analyze OAuth configuration from app bundles to d
- NEXT(hypotheses-nemotron3.txt): PROBE: POST https://digital-onboarding.vpbank.com/users/sign_in Content-Type: application/x-www-form-urlencoded body: user[email]=test@test.com&user[password]=t
- LEARN: REJECTED SSRF @ api.vpbank.com (Host header routing): verify_steps executed — Host:169.254.169.254, Host:localhost, X-Forwarded-Host:169.254.169.254 all returne
- LEARN: REJECTED OAUTH @ www.vpbank.com (redirect_uri bypass): JS bundles contain no OAuth client_id (only Usercentrics widget clientWid); /oauth/authorize always 303→e
- LEARN: NEW INFO @ www.vpbank.com/portal/api/: Full LitElement SPA served (403 with body) — separate app from Drupal; POST /portal/api/language/:language sets portal-la
- LEARN: REJECTED MISCONFIG @ www.vpbank.com (portal API access): /portal/api/ returns 403 but serves complete SPA shell; /portal/api/health and /portal/api/status retur
- LEARN: ACCEPTED MISCONFIG @ api.vpbank.com (Layer7 gateway error handling): Structured JSON errors with requestIds confirmed — info leak vector but low severity
- LEARN: REJECTED SSRF @ api.vpbank.com (Host header routing): verify_steps executed — Host:169.254.169.254, Host:localhost, X-Forwarded-Host:169.254.169.254 all returne
- LEARN: REJECTED OAUTH @ www.vpbank.com (redirect_uri bypass): JS bundles contain no OAuth client_id (only Usercentrics widget clientWid); /oauth/authorize always 303→e
- LEARN: NEW INFO @ www.vpbank.com/portal/api/: Full LitElement SPA served (403 with body) — separate app from Drupal; POST /portal/api/language/:language sets portal-la
- LEARN: REJECTED MISCONFIG @ www.vpbank.com (portal API access): /portal/api/ returns 403 but serves complete SPA shell; /portal/api/health and /portal/api/status retur
- LEARN: ACCEPTED MISCONFIG @ api.vpbank.com (Layer7 gateway error handling): Structured JSON errors with requestIds confirmed — info leak vector but low severity
- LEARN: REJECTED SSRF @ api.vpbank.com (Host header routing): verify_steps executed — Host:169.254.169.254, Host:localhost, X-Forwarded-Host:169.254.169.254 all returne
- LEARN: REJECTED MISCONFIG @ api.vpbank.com (Layer7 policy bypass): All malformed request probes (XML, SOAP, routing headers) return identical HTTP 500 JSON — no policy
- LEARN: REJECTED OAUTH @ www.vpbank.com (OIDC discovery): .well-known/openid-configuration returns 303 to error page — no metadata exposed
- LEARN: REJECTED OAUTH @ www.vpbank.com (redirect_uri bypass): JS bundles contain no OAuth client_id (only Usercentrics widget clientWid); /oauth/authorize always 303→e
- LEARN: REJECTED MISCONFIG @ vpbank-dev.com/vpbank-stage.com (staging exposure): WAF 2.3.0_20260324 intercepts ALL paths — no application content accessible
- LEARN: REJECTED MISCONFIG @ www.vpbank.com (portal API access): /portal/api/ returns 403 but serves complete SPA shell; /portal/api/health and /portal/api/status retur
- LEARN: ACCEPTED MISCONFIG @ api.vpbank.com (Layer7 gateway error handling): Structured JSON errors with requestIds confirmed — info leak vector but low severity
- LEARN: ACCEPTED MISCONFIG @ vpbank-dev.com/vpbank-stage.com: Production CSP trusts these domains; publicly reachable Apache servers behind maintenance redirect — in-sc
- LEARN: NEW INFO @ www.vpbank.com/portal/api/: Full LitElement SPA served (403 with body) — separate app from Drupal; POST /portal/api/language/:language sets portal-la
- LEARN: REJECTED MISCONFIG @ api.vpbank.com (subdomain takeover): ellb.ch is active Swiss load balancer, not a decommissioned cloud provider tenant
- LEARN: REJECTED SSRF @ api.vpbank.com (Host header routing): verify_steps executed — Host:169.254.169.254, Host:localhost, X-Forwarded-Host:169.254.169.254 all returne
- LEARN: REJECTED OAUTH @ www.vpbank.com (redirect_uri bypass): JS bundles contain no OAuth client_id (only Usercentrics widget clientWid); /oauth/authorize always 303→e
- LEARN: NEW INFO @ www.vpbank.com/portal/api/: Full LitElement SPA served (403 with body) — separate app from Drupal; POST /portal/api/language/:language sets portal-la
- LEARN: REJECTED MISCONFIG @ www.vpbank.com (portal API access): /portal/api/ returns 403 but serves complete SPA shell; /portal/api/health and /portal/api/status retur
- LEARN: ACCEPTED MISCONFIG @ api.vpbank.com (Layer7 gateway error handling): Structured JSON errors with requestIds confirmed — info leak vector but low severity
- LEARN: REJECTED SSRF @ api.vpbank.com (Host header routing): verify_steps executed — Host:169.254.169.254, Host:localhost, X-Forwarded-Host:169.254.169.254 all returne
- LEARN: REJECTED MISCONFIG @ api.vpbank.com (Layer7 policy bypass): All malformed request probes (XML, SOAP, routing headers) return identical HTTP 500 JSON — no policy
- LEARN: REJECTED OAUTH @ www.vpbank.com (OIDC discovery): .well-known/openid-configuration returns 303 to error page — no metadata exposed
- LEARN: REJECTED OAUTH @ www.vpbank.com (redirect_uri bypass): JS bundles contain no OAuth client_id (only Usercentrics widget clientWid); /oauth/authorize always 303→e
- LEARN: REJECTED MISCONFIG @ vpbank-dev.com/vpbank-stage.com (staging exposure): WAF 2.3.0_20260324 intercepts ALL paths — no application content accessible
- LEARN: REJECTED MISCONFIG @ www.vpbank.com (portal API access): /portal/api/ returns 403 but serves complete SPA shell; /portal/api/health and /portal/api/status retur
- LEARN: ACCEPTED MISCONFIG @ api.vpbank.com (Layer7 gateway error handling): Structured JSON errors with requestIds confirmed — info leak vector but low severity
- LEARN: ACCEPTED MISCONFIG @ vpbank-dev.com/vpbank-stage.com: Production CSP trusts these domains; publicly reachable Apache servers behind maintenance redirect — in-sc
- LEARN: NEW INFO @ www.vpbank.com/portal/api/: Full LitElement SPA served (403 with body) — separate app from Drupal; POST /portal/api/language/:language sets portal-la
- LEARN: OAuth client_id hunt via App Store/Google/GitHub/npm surfaced only VP Bank Vietnam (different entity) + unrelated SDKs — no Liechtenstein client_id.
- LEARN: ACCEPTED MISCONFIG @ developer.vpbank.com: PSD2 Berlin Group sandbox API reachable HTTP 200 without client cert or basic auth (documented-open since 2024-05); o
- LEARN: NEW INFO @ openbanking.vpbank.com (193.222.70.154): discovered via TLS cert CN=openbanking.vpbank.com; production PSD2 ASPSP, mTLS "certificate required" at TLS
- LEARN: NEW INFO @ developer.vpbank.com: /psd2/sandbox/certificate/verify_certificate also mTLS-gated; OpenAPI spec self-labels server "PSD2 production server" while op
- LEARN: REJECTED MISCONFIG @ developer.vpbank.com (Drupal admin on PSD2 vhost): /user/login, /admin, /sites/default/files/* all HTTP 404 — dedicated docroot.
- LEARN: REJECTED MISCONFIG @ openbanking.vpbank.com (anonymous API access): mTLS enforced, anonymous rejected — expected production posture, not a flaw.
- LEARN: ACCEPTED IDOR @ developer.vpbank.com (PSD2 sandbox BOLA): verify_steps EXECUTED in official test sandbox (synthetic data) — consent 6b517824-e5af-4202-b9b0-7f48
- LEARN: REJECTED BUSLOGIC @ developer.vpbank.com (X-Request-ID state encoding): documented deterministic client-driven state for the sandbox; observed ACSC from X-Reque
- LEARN: REJECTED OAUTH @ www.vpbank.com/oauth/authorize: RAG GitHub/public-web surfaces only VP Bank Vietnam (separate entity) and generic PSD2 frameworks; downloaded s
- LEARN: REJECTED MISCONFIG @ www.vpbank.com/developer.vpbank.com (PSD2 statistics pages): /psd2-statistics, /psd2-statistics/, /psd2/statistics/ all HTTP 404 — no anony
- LEARN: ACCEPTED MISCONFIG @ digital-onboarding.vpbank.com: Live multi-tenant bank-onboarding/back-office SaaS ('US', Rails+Devise) on off-net hosting (89.163.182.69/.2
- LEARN: REJECTED MISCONFIG @ api-prep.vpbank.com: Same Layer7 dead-end as api.vpbank.com (SCS-Request-ID, INVALID_REQUEST_RESOURCE) — pre-prod gateway clone, no new sur
- LEARN: REJECTED MISCONFIG @ designsystem.vpbank.com: CNAME→vpb-design-system.netlify.app serves 200 — hosted design system is active, no subdomain takeover.
- LEARN: NEW INFO @ sts.vpbank.com: ADFS OIDC metadata 200 (issuer sts.vpbank.com/adfs, device_code/password grants, vpn/logon cert scopes) — corporate IdP, parked.
- LEARN: REJECTED MISCONFIG @ api.vpbank.com (subdomain takeover): ellb.ch is active Swiss load balancer, not a decommissioned cloud provider tenant
- LEARN: ACCEPTED MISCONFIG @ developer.vpbank.com (PSD2 Developer Portal exposure): Live PSD2 sandbox API with full OpenAPI spec, functional endpoints (consents, accoun
- LEARN: REJECTED MISCONFIG @ vpbank-dev.com/vpbank-stage.com (staging exposure): WAF 2.3.0_20260324 intercepts ALL paths — no application content accessible past mainte
- LEARN: REJECTED SSRF @ api.vpbank.com (Host header routing): verify_steps executed — Host:169.254.169.254, Host:localhost, X-Forwarded-Host:169.254.169.254 all returne
- LEARN: REJECTED OAUTH @ www.vpbank.com (redirect_uri bypass): JS bundles contain no OAuth client_id (only Usercentrics widget clientWid); /oauth/authorize always 303→e
- LEARN: REJECTED MISCONFIG @ www.vpbank.com (portal API access): /portal/api/ returns 403 but serves complete SPA shell; /portal/api/health and /portal/api/status retur
- LEARN: ACCEPTED MISCONFIG @ api.vpbank.com (Layer7 gateway error handling): Structured JSON errors with requestIds confirmed — info leak vector but low severity.
- LEARN: NEW INFO @ developer.vpbank.com: PSD2 sandbox uses basic auth (test:test works), X-Request-ID last digit controls state encoding (1=RCVD, 5=ACSC for payments), 
- LEARN: ACCEPTED IDOR @ developer.vpbank.com (PSD2 sandbox BOLA): verify_steps EXECUTED in official test sandbox (synthetic data) — consent 6b517824-e5af-4202-b9b0-7f48
- LEARN: REJECTED BUSLOGIC @ developer.vpbank.com (X-Request-ID state encoding): documented deterministic client-driven state for the sandbox; observed ACSC from X-Reque
- LEARN: REJECTED OAUTH @ www.vpbank.com/oauth/authorize: RAG GitHub/public-web surfaces only VP Bank Vietnam (separate entity) and generic PSD2 frameworks; downloaded s
- LEARN: REJECTED MISCONFIG @ www.vpbank.com/developer.vpbank.com (PSD2 statistics pages): /psd2-statistics, /psd2-statistics/, /psd2/statistics/ all HTTP 404 — no anony
- LEARN: REJECTED SSRF @ api.vpbank.com (Host header routing): verify_steps executed — Host:169.254.169.254, Host:localhost, X-Forwarded-Host:169.254.169.254 all returne
- LEARN: REJECTED MISCONFIG @ api.vpbank.com (Layer7 policy bypass): All malformed request probes (XML, SOAP, routing headers) return identical HTTP 500 JSON — no policy
- LEARN: REJECTED MISCONFIG @ vpbank-dev.com/vpbank-stage.com (staging exposure): WAF 2.3.0_20260324 intercepts ALL paths — no application content accessible past mainte
- LEARN: REJECTED MISCONFIG @ www.vpbank.com (portal API access): /portal/api/ returns 403 but serves complete SPA shell; /portal/api/health and /portal/api/status retur
- LEARN: ACCEPTED MISCONFIG @ api.vpbank.com (Layer7 gateway error handling): Structured JSON errors with requestIds confirmed — info leak vector but low severity.
- LEARN: ACCEPTED MISCONFIG @ developer.vpbank.com (PSD2 Developer Portal exposure): Live PSD2 sandbox API with full OpenAPI spec, functional endpoints (consents, accoun
- LEARN: NEW INFO @ openbanking.vpbank.com (193.222.70.154): discovered via TLS cert CN=openbanking.vpbank.com; production PSD2 ASPSP, mTLS "certificate required" at TLS
- LEARN: ACCEPTED MISCONFIG @ digital-onboarding.vpbank.com: Live multi-tenant bank-onboarding/back-office SaaS ('US', Rails+Devise) on off-net hosting (89.163.182.69/.2
- LEARN: ACCEPTED IDOR @ developer.vpbank.com (PSD2 sandbox BOLA): verify_steps EXECUTED in official test sandbox (synthetic data) — consent 6b517824-e5af-4202-b9b0-7f48
- LEARN: REJECTED BUSLOGIC @ developer.vpbank.com (X-Request-ID state encoding): documented deterministic client-driven state for sandbox; observed ACSC from X-Request-I
- LEARN: REJECTED OAUTH @ www.vpbank.com/oauth/authorize: RAG GitHub/public-web surfaces only VP Bank Vietnam (separate entity) + generic PSD2 frameworks; spec contains 
- LEARN: REJECTED MISCONFIG @ www.vpbank.com/developer.vpbank.com (PSD2 statistics pages): /psd2-statistics, /psd2-statistics/, /psd2/statistics/ all HTTP 404
- LEARN: REJECTED SSRF @ api.vpbank.com (Host header routing): verify_steps executed — Host:169.254.169.254, Host:localhost, X-Forwarded-Host:169.254.169.254 all returne
- LEARN: REJECTED MISCONFIG @ api.vpbank.com (Layer7 policy bypass): All malformed probes (XML, SOAP, routing headers) return identical HTTP 500 JSON — no policy differe
- LEARN: REJECTED MISCONFIG @ vpbank-dev.com/vpbank-stage.com (staging exposure): WAF 2.3.0_20260324 intercepts ALL paths — CONFIRMED REJECTED
- LEARN: REJECTED MISCONFIG @ www.vpbank.com (portal API access): /portal/api/ returns 403 with SPA shell; /portal/api/health and /portal/api/status return WAF maintenan
- LEARN: ACCEPTED MISCONFIG @ api.vpbank.com (Layer7 gateway error handling): Structured JSON errors with requestIds confirmed — info leak vector, low severity
- LEARN: ACCEPTED MISCONFIG @ developer.vpbank.com (PSD2 Developer Portal exposure): Live PSD2 sandbox API with full OpenAPI spec, functional endpoints (consents, accoun
- LEARN: NEW INFO @ openbanking.vpbank.com (193.222.70.154): production PSD2 ASPSP, mTLS "certificate required" at TLS layer — anonymous blocked as designed
- LEARN: NEW INFO @ sts.vpbank.com: ADFS OIDC metadata 200 (issuer sts.vpbank.com/adfs, device_code/password/implicit grants, vpn/logon/cert scopes) — corporate IdP

## RANKED HYPOTHESES 2026-09-05 09:58:15 UTC
- [70] digital-onboarding.vpbank.com/users/sign_in: Mass assignment on Devise sign-in grants admin/impersonation on digital-onboarding back-office (from art/lead_nemotron3.txt)
- [55] digital-onboarding.vpbank.com/api/v1/: digital-onboarding.vpbank.com IDOR via /api/v1/ tenant/admin endpoints (from art/lead_bigpickle.txt)
- NEXT(hypotheses-bigpickle.txt): SCAN: passive JS bundle extraction from https://digital-onboarding.vpbank.com/control-center/ — extract all /api/v1/* routes, sign_in endpoint, form action URLs
- NEXT(hypotheses-nemotron3.txt): PROBE: POST https://digital-onboarding.vpbank.com/users/sign_in Content-Type: application/x-www-form-urlencoded body: user[email]=test@test.com&user[password]=t
- LEARN: ACCEPTED MISCONFIG @ digital-onboarding.vpbank.com: Production multi-tenant back-office SaaS (Rails+Devise), anonymous /control-center/ SPA + /api/v1/brand 200 
- LEARN: NEW INFO @ sts.vpbank.com: ADFS OIDC metadata 200 (issuer sts.vpbank.com/adfs, device_code/password/implicit grants, vpn/logon/cert scopes) — corporate IdP, par
- LEARN: REJECTED MISCONFIG @ api-prep.vpbank.com: Layer7 clone of api.vpbank.com — no new surface
- LEARN: REJECTED MISCONFIG @ designsystem.vpbank.com: Active Netlify app — no subdomain takeover
- LEARN: ACCEPTED IDOR @ developer.vpbank.com: PSD2 sandbox BOLA verified end-to-end — consent/account/payment cross-session read, zero binding
- LEARN: REJECTED OAUTH @ www.vpbank.com: No client_id obtainable — redirect_uri bypass blocked
- LEARN: REJECTED MISCONFIG @ vpbank-dev.com/vpbank-stage.com: WAF intercepts ALL paths — confirmed rejected
- LEARN: ACCEPTED MISCONFIG @ digital-onboarding.vpbank.com: Live multi-tenant bank-onboarding/back-office SaaS ('US', Rails+Devise) on off-net hosting (89.163.182.69/.2
- LEARN: ACCEPTED IDOR @ developer.vpbank.com (PSD2 sandbox BOLA): verify_steps EXECUTED in official test sandbox (synthetic data) — consent 6b517824-e5af-4202-b9b0-7f48
- LEARN: REJECTED BUSLOGIC @ developer.vpbank.com (X-Request-ID state encoding): documented deterministic client-driven state for sandbox; observed ACSC from X-Request-I
- LEARN: REJECTED OAUTH @ www.vpbank.com/oauth/authorize: RAG GitHub/public-web surfaces only VP Bank Vietnam (separate entity) + generic PSD2 frameworks; spec contains 
- LEARN: REJECTED MISCONFIG @ www.vpbank.com/developer.vpbank.com (PSD2 statistics pages): /psd2-statistics, /psd2-statistics/, /psd2/statistics/ all HTTP 404
- LEARN: REJECTED SSRF @ api.vpbank.com (Host header routing): verify_steps executed — Host:169.254.169.254, Host:localhost, X-Forwarded-Host:169.254.169.254 all returne
- LEARN: REJECTED MISCONFIG @ api.vpbank.com (Layer7 policy bypass): All malformed probes (XML, SOAP, routing headers) return identical HTTP 500 JSON — no policy differe
- LEARN: REJECTED MISCONFIG @ vpbank-dev.com/vpbank-stage.com (staging exposure): WAF 2.3.0_20260324 intercepts ALL paths — CONFIRMED REJECTED
- LEARN: REJECTED MISCONFIG @ www.vpbank.com (portal API access): /portal/api/ returns 403 with SPA shell; /portal/api/health and /portal/api/status return WAF maintenan
- LEARN: ACCEPTED MISCONFIG @ api.vpbank.com (Layer7 gateway error handling): Structured JSON errors with requestIds confirmed — info leak vector, low severity
- LEARN: ACCEPTED MISCONFIG @ developer.vpbank.com (PSD2 Developer Portal exposure): Live PSD2 sandbox API with full OpenAPI spec, functional endpoints (consents, accoun
- LEARN: NEW INFO @ openbanking.vpbank.com (193.222.70.154): production PSD2 ASPSP, mTLS "certificate required" at TLS layer — anonymous blocked as designed
- LEARN: NEW INFO @ sts.vpbank.com: ADFS OIDC metadata 200 (issuer sts.vpbank.com/adfs, device_code/password/implicit grants, vpn/logon/cert scopes) — corporate IdP

## RANKED HYPOTHESES 2026-09-05 13:21:01 UTC
- [75] digital-onboarding.vpbank.com/users/sign_in: Mass assignment on Devise sign-in grants admin/impersonation on digital-onboarding back-office (from art/lead_nemotron3.txt)
- [55] digital-onboarding.vpbank.com/api/v1/: digital-onboarding.vpbank.com IDOR via /api/v1/ tenant/admin endpoints (from art/lead_bigpickle.txt)
- NEXT(hypotheses-bigpickle.txt): SCAN: passive JS bundle extraction from https://digital-onboarding.vpbank.com/control-center/ — extract all /api/v1/* routes, sign_in endpoint, form action URLs
- NEXT(hypotheses-nemotron3.txt): PROBE: POST https://digital-onboarding.vpbank.com/users/sign_in Content-Type: application/x-www-form-urlencoded body: user[email]=test@test.com&user[password]=t
- LEARN: ACCEPTED MISCONFIG @ digital-onboarding.vpbank.com: Production multi-tenant back-office SaaS (Rails+Devise), anonymous /control-center/ SPA + /api/v1/brand 200 
- LEARN: NEW INFO @ sts.vpbank.com: ADFS OIDC metadata 200 (issuer sts.vpbank.com/adfs, device_code/password/implicit grants, vpn/logon/cert scopes) — corporate IdP, par
- LEARN: REJECTED MISCONFIG @ api-prep.vpbank.com: Layer7 clone of api.vpbank.com — no new surface
- LEARN: REJECTED MISCONFIG @ designsystem.vpbank.com: Active Netlify app — no subdomain takeover
- LEARN: ACCEPTED IDOR @ developer.vpbank.com: PSD2 sandbox BOLA verified end-to-end — consent/account/payment cross-session read, zero binding
- LEARN: REJECTED OAUTH @ www.vpbank.com: No client_id obtainable — redirect_uri bypass blocked
- LEARN: REJECTED MISCONFIG @ vpbank-dev.com/vpbank-stage.com: WAF intercepts ALL paths — confirmed rejected
- LEARN: ACCEPTED MISCONFIG @ digital-onboarding.vpbank.com: Live multi-tenant bank-onboarding/back-office SaaS ('US', Rails+Devise) on off-net hosting (89.163.182.69/.2
- LEARN: ACCEPTED IDOR @ developer.vpbank.com (PSD2 sandbox BOLA): verify_steps EXECUTED in official test sandbox (synthetic data) — consent 6b517824-e5af-4202-b9b0-7f48
- LEARN: REJECTED BUSLOGIC @ developer.vpbank.com (X-Request-ID state encoding): documented deterministic client-driven state for sandbox; observed ACSC from X-Request-I
- LEARN: REJECTED OAUTH @ www.vpbank.com/oauth/authorize: RAG GitHub/public-web surfaces only VP Bank Vietnam (separate entity) + generic PSD2 frameworks; spec contains 
- LEARN: REJECTED MISCONFIG @ www.vpbank.com/developer.vpbank.com (PSD2 statistics pages): /psd2-statistics, /psd2-statistics/, /psd2/statistics/ all HTTP 404
- LEARN: REJECTED SSRF @ api.vpbank.com (Host header routing): verify_steps executed — Host:169.254.169.254, Host:localhost, X-Forwarded-Host:169.254.169.254 all returne
- LEARN: REJECTED MISCONFIG @ api.vpbank.com (Layer7 policy bypass): All malformed probes (XML, SOAP, routing headers) return identical HTTP 500 JSON — no policy differe
- LEARN: REJECTED MISCONFIG @ vpbank-dev.com/vpbank-stage.com (staging exposure): WAF 2.3.0_20260324 intercepts ALL paths — CONFIRMED REJECTED
- LEARN: REJECTED MISCONFIG @ www.vpbank.com (portal API access): /portal/api/ returns 403 with SPA shell; /portal/api/health and /portal/api/status return WAF maintenan
- LEARN: ACCEPTED MISCONFIG @ api.vpbank.com (Layer7 gateway error handling): Structured JSON errors with requestIds confirmed — info leak vector, low severity
- LEARN: ACCEPTED MISCONFIG @ developer.vpbank.com (PSD2 Developer Portal exposure): Live PSD2 sandbox API with full OpenAPI spec, functional endpoints (consents, accoun
- LEARN: NEW INFO @ openbanking.vpbank.com (193.222.70.154): production PSD2 ASPSP, mTLS "certificate required" at TLS layer — anonymous blocked as designed
- LEARN: NEW INFO @ sts.vpbank.com: ADFS OIDC metadata 200 (issuer sts.vpbank.com/adfs, device_code/password/implicit grants, vpn/logon/cert scopes) — corporate IdP
- LEARN: ACCEPTED MISCONFIG @ digital-onboarding.vpbank.com: Live multi-tenant bank-onboarding/back-office SaaS ('US', Rails+Devise) on off-net hosting (89.163.182.69/.2
- LEARN: ACCEPTED IDOR @ developer.vpbank.com (PSD2 sandbox BOLA): verify_steps EXECUTED in official test sandbox (synthetic data) — consent 6b517824-e5af-4202-b9b0-7f48
- LEARN: REJECTED BUSLOGIC @ developer.vpbank.com (X-Request-ID state encoding): documented deterministic client-driven state for sandbox; observed ACSC from X-Request-I
- LEARN: REJECTED OAUTH @ www.vpbank.com/oauth/authorize: RAG GitHub/public-web surfaces only VP Bank Vietnam (separate entity) + generic PSD2 frameworks; spec contains 
- LEARN: REJECTED MISCONFIG @ www.vpbank.com/developer.vpbank.com (PSD2 statistics pages): /psd2-statistics, /psd2-statistics/, /psd2/statistics/ all HTTP 404
- LEARN: REJECTED SSRF @ api.vpbank.com (Host header routing): verify_steps executed — Host:169.254.169.254, Host:localhost, X-Forwarded-Host:169.254.169.254 all returne
- LEARN: REJECTED MISCONFIG @ api.vpbank.com (Layer7 policy bypass): All malformed probes (XML, SOAP, routing headers) return identical HTTP 500 JSON — no policy differe
- LEARN: REJECTED MISCONFIG @ vpbank-dev.com/vpbank-stage.com (staging exposure): WAF 2.3.0_20260324 intercepts ALL paths — CONFIRMED REJECTED
- LEARN: REJECTED MISCONFIG @ www.vpbank.com (portal API access): /portal/api/ returns 403 with SPA shell; /portal/api/health and /portal/api/status return WAF maintenan
- LEARN: ACCEPTED MISCONFIG @ api.vpbank.com (Layer7 gateway error handling): Structured JSON errors with requestIds confirmed — info leak vector, low severity
- LEARN: ACCEPTED MISCONFIG @ developer.vpbank.com (PSD2 Developer Portal exposure): Live PSD2 sandbox API with full OpenAPI spec, functional endpoints (consents, accoun
- LEARN: NEW INFO @ openbanking.vpbank.com (193.222.70.154): production PSD2 ASPSP, mTLS "certificate required" at TLS layer — anonymous blocked as designed
- LEARN: NEW INFO @ sts.vpbank.com: ADFS OIDC metadata 200 (issuer sts.vpbank.com/adfs, device_code/password/implicit grants, vpn/logon/cert scopes) — corporate IdP
- LEARN: REJECTED MISCONFIG @ api-prep.vpbank.com: Layer7 clone of api.vpbank.com — no new surface
- LEARN: REJECTED MISCONFIG @ designsystem.vpbank.com: Active Netlify app — no subdomain takeover

## RANKED HYPOTHESES 2026-09-05 16:14:01 UTC
- [75] digital-onboarding.vpbank.com/users/sign_in: Mass assignment on Devise sign-in grants admin/impersonation on digital-onboarding back-office (from art/lead_nemotron3.txt)
- NEXT(hypotheses-nemotron3.txt): PROBE: POST https://digital-onboarding.vpbank.com/users/sign_in Content-Type: application/x-www-form-urlencoded body: user[email]=test@test.com&user[password]=t
- LEARN: ACCEPTED MISCONFIG @ digital-onboarding.vpbank.com: Live multi-tenant bank-onboarding/back-office SaaS ('US', Rails+Devise) on off-net hosting (89.163.182.69/.2
- LEARN: ACCEPTED IDOR @ developer.vpbank.com (PSD2 sandbox BOLA): verify_steps EXECUTED in official test sandbox (synthetic data) — consent 6b517824-e5af-4202-b9b0-7f48
- LEARN: REJECTED BUSLOGIC @ developer.vpbank.com (X-Request-ID state encoding): documented deterministic client-driven state for sandbox; observed ACSC from X-Request-I
- LEARN: REJECTED OAUTH @ www.vpbank.com/oauth/authorize: RAG GitHub/public-web surfaces only VP Bank Vietnam (separate entity) + generic PSD2 frameworks; spec contains 
- LEARN: REJECTED MISCONFIG @ www.vpbank.com/developer.vpbank.com (PSD2 statistics pages): /psd2-statistics, /psd2-statistics/, /psd2/statistics/ all HTTP 404
- LEARN: REJECTED SSRF @ api.vpbank.com (Host header routing): verify_steps executed — Host:169.254.169.254, Host:localhost, X-Forwarded-Host:169.254.169.254 all returne
- LEARN: REJECTED MISCONFIG @ api.vpbank.com (Layer7 policy bypass): All malformed probes (XML, SOAP, routing headers) return identical HTTP 500 JSON — no policy differe
- LEARN: REJECTED MISCONFIG @ vpbank-dev.com/vpbank-stage.com (staging exposure): WAF 2.3.0_20260324 intercepts ALL paths — CONFIRMED REJECTED
- LEARN: REJECTED MISCONFIG @ www.vpbank.com (portal API access): /portal/api/ returns 403 with SPA shell; /portal/api/health and /portal/api/status return WAF maintenan
- LEARN: ACCEPTED MISCONFIG @ api.vpbank.com (Layer7 gateway error handling): Structured JSON errors with requestIds confirmed — info leak vector, low severity
- LEARN: ACCEPTED MISCONFIG @ developer.vpbank.com (PSD2 Developer Portal exposure): Live PSD2 sandbox API with full OpenAPI spec, functional endpoints (consents, accoun
- LEARN: NEW INFO @ openbanking.vpbank.com (193.222.70.154): production PSD2 ASPSP, mTLS "certificate required" at TLS layer — anonymous blocked as designed
- LEARN: NEW INFO @ sts.vpbank.com: ADFS OIDC metadata 200 (issuer sts.vpbank.com/adfs, device_code/password/implicit grants, vpn/logon/cert scopes) — corporate IdP
- LEARN: REJECTED MISCONFIG @ api-prep.vpbank.com: Layer7 clone of api.vpbank.com — no new surface
- LEARN: REJECTED MISCONFIG @ designsystem.vpbank.com: Active Netlify app — no subdomain takeover

## RANKED HYPOTHESES 2026-09-05 18:36:35 UTC
- [80] digital-onboarding.vpbank.com/users/sign_in: Mass assignment on Devise sign-in grants admin/impersonation on digital-onboarding back-office (from art/lead_nemotron3.txt)
- [45] digital-onboarding.vpbank.com/api/v1/sessions: force_tenant cross-tenant data access on digital-onboarding sessions (from art/lead_bigpickle.txt)
- NEXT(hypotheses-bigpickle.txt): RAG: extract ADFS `client_id` from VP Bank Connect mobile app (App Store/Google Play bundle strings, "vpbank connect" + "adfs"/"devicecode"/"oauth") to unlock P
- NEXT(hypotheses-nemotron3.txt): PROBE: POST https://digital-onboarding.vpbank.com/users/sign_in Content-Type: application/x-www-form-urlencoded body: user[email]=test@test.com&user[password]=t
- LEARN: REJECTED MISCONFIG @ digital-onboarding.vpbank.com (anonymous data access): admin/user/qr_codes/tenants API all JWT-gated (401 invalid token / 403 Not authorize
- LEARN: NEW INFO @ digital-onboarding.vpbank.com: control-center SPA bundle (4MB) confirms API map incl. `/api/v1/sessions/{idp_login,secure_session,reset_password}`, `
- LEARN: NEW INFO @ digital-onboarding.vpbank.com: `/users/sign_up` returns HTTP 500 (Rails 500.html), Devise registrable route mounted but errors — registration-enablem
- LEARN: REJECTED OATH @ sts.vpbank.com: `/adfs/oauth2/token/devicecode` 200 is MS-HTTPAPI error shell (X-MS-Forwarded-Status-Code:500); real endpoint is `/adfs/oauth2/d
- LEARN: ACCEPTED MISCONFIG @ digital-onboarding.vpbank.com: Live multi-tenant bank-onboarding/back-office SaaS ('US', Rails+Devise) on off-net hosting (89.163.182.69/.2
- LEARN: ACCEPTED MISCONFIG @ sts.vpbank.com: ADFS OIDC metadata 200 exposes device_code/password/implicit grants + vpn/logon/cert scopes — corporate IdP for VPN/cert au
- LEARN: ACCEPTED IDOR @ developer.vpbank.com (PSD2 sandbox BOLA): verify_steps EXECUTED in official test sandbox (synthetic data) — consent 6b517824-e5af-4202-b9b0-7f48
- LEARN: REJECTED MISCONFIG @ api.vpbank.com: All vectors exhausted — uniform INVALID_REQUEST_RESOURCE JSON 500, CONFIRMED REJECTED
- LEARN: REJECTED OAUTH @ www.vpbank.com: No valid client_id in JS bundles (only Usercentrics clientWid); /oauth/authorize always 303→error page — redirect_uri bypass bl
- LEARN: REJECTED MISCONFIG @ vpbank-dev.com/vpbank-stage.com: WAF 2.3.0_20260324 intercepts ALL paths — CONFIRMED REJECTED
- LEARN: REJECTED MISCONFIG @ api-prep.vpbank.com: Layer7 clone — no new surface
- LEARN: REJECTED MISCONFIG @ designsystem.vpbank.com: Active Netlify app — no subdomain takeover
- LEARN: NEW INFO @ openbanking.vpbank.com: Production PSD2 ASPSP, mTLS "certificate required" at TLS layer — anonymous blocked as designed
