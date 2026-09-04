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
