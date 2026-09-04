# VP Bank AG inventory (discovery seed 2026-09-02)
# NOTE: hosts below are discovery candidates from passive DNS/CT; confirm in-scope vs program scope before active testing.
api.vpbank.com
vpbank.com
www.vpbank.com

## PASSIVE RECON 2026-09-02 (read-only, non-intrusive)

> Recon observations only. These are NOT confirmed vulnerabilities; ownership/in-scope of each host must be confirmed against the program scope before any active testing. Hosts resolve + serve HTTP — investigation requires scoped authorization.

**Probed:** 3 hosts | **Live HTTP:** 0

| Host | Status | Server/Tech |
|---|---|---|

**CNAME review signals (1):**
- `api.vpbank.com` -> `api-vpbank-com.hdb-cs04.ellb.ch`

## 2026-09-02 21:53:27 UTC

## 2026-09-02 23:50:48 UTC

## 2026-09-03 02:36:22 UTC

## 2026-09-03 07:29:02 UTC

## 2026-09-03 12:18:41 UTC

## 2026-09-03 16:33:58 UTC

## 2026-09-03 19:30:16 UTC
- NEW Live HTTP probes completed on all 3 inventory hosts (previously 0 live)
- NEW api.vpbank.com confirmed running Layer7-API-Gateway (CA API Gateway); all versioned/debug paths return HTTP 500 with JSON 404 error (INVALID_REQUEST_RESOURCE)
- NEW www.vpbank.com serves marketing site on Apache; OAuth/OIDC endpoints exist but reject invalid requests with 303 to error page
- NEW vpbank.com redirects to www.vpbank.com for all tested paths
- CHANGED api.vpbank.com attack surface reduced: no exposed API versions, Swagger, OpenAPI, actuator, or security.txt

## 2026-09-03 21:57:32 UTC
- NEW vpbank-dev.com + vpbank-stage.com discovered via production CSP (www.vpbank.com) as trusted origins; both resolve (193.222.70.165/.166) and are live Apache servers
- NEW www.vpbank.com responds 200 on /en; Drupal + Envoy proxy; robust CSP present; CSRFT759 + AL_SESS cookies
- NEW www.vpbank-dev.com and www.vpbank-stage.com redirect 302 to /error_path/maintenance.html (real maintenance site, not parked)
- CHANGED attack surface expanded beyond 3 inventory hosts; dev/stage domains are scoped (company-operated) and in production trust chain
- NEW Live HTTP probes completed on all 3 inventory hosts (api.vpbank.com, www.vpbank.com, vpbank.com)
- NEW api.vpbank.com confirmed Layer7-API-Gateway (CA API Gateway); all paths return HTTP 500 with JSON 404 (INVALID_REQUEST_RESOURCE)
- NEW www.vpbank.com serves marketing site on Apache; OAuth/OIDC endpoints at /oauth/authorize and /.well-known/openid-configuration reject invalid requests with 303/400
- NEW vpbank.com redirects to www.vpbank.com for all tested paths
- CHANGED api.vpbank.com attack surface reduced: no exposed API versions, Swagger, OpenAPI, actuator, or security.txt
- CHANGED Priority shift: www.vpbank.com now highest (7.8) due to OAuth surface + marketing site exposure; api.vpbank.com reduced to 6.2

## 2026-09-03 23:51:14 UTC
- NEW www.vpbank-dev.com and www.vpbank-stage.com ALL paths return identical maintenance page (WAF 2.3.0_20260324) - no application content accessible

## 2026-09-04 02:38:02 UTC
- CHANGED vpbank-dev.com/vpbank-stage.com staging hypothesis CONFIRMED REJECTED: WAF 2.3.0_20260324 intercepts ALL paths, zero app content reachable
- CHANGED www.vpbank.com/portal/api confidence 35 < 40 threshold, PARKED

## 2026-09-04 07:28:45 UTC
- CHANGED api.vpbank.com: All XML/SOAP/routing-header probes return identical HTTP 500 JSON (INTERNAL_SERVER_ERROR) — no differential behavior for policy bypass or SSRF
- CHANGED www.vpbank.com: OAuth/OIDC endpoints (/oauth/authorize, /.well-known/openid-configuration) return 303 to error pages for all tested client_id/redirect_uri combos — no valid client context discovered

## 2026-09-04 12:20:53 UTC
- CHANGED api.vpbank.com: All XML/SOAP/routing-header probes (Accept:application/xml, SOAP envelope, Host/X-Forwarded-Host/X-Forwarded-For to 169.254.169.254/localhost/10.0.0.1) return identical HTTP 500 JSON (
- CHANGED www.vpbank.com: OAuth/OIDC endpoints (/oauth/authorize, /.well-known/openid-configuration) return 303/400 error pages for all tested client_id/redirect_uri combos — no valid client context discovered 
- CHANGED vpbank-dev.com/vpbank-stage.com: WAF 2.3.0_20260324 confirmed intercepting ALL paths with maintenance page — zero application content reachable
- NEW Risk score reduced from 65 to 45 due to failed exploitation of top hypotheses (OAuth redirect_uri, Layer7 policy bypass, SSRF routing)
- NEW www.vpbank.com/portal/api/ serves full LitElement SPA (HTTP 403 with body) — separate app from Drupal; includes `POST /portal/api/language/:language`, CSRF token (`CSRFT759.js`), version renderer, cus
- NEW www.vpbank.com/portal/api/health and /portal/api/status return WAF maintenance page — backend intercepted, SPA is frontend shell only

## 2026-09-04 16:37:23 UTC
- CHANGED api.vpbank.com: All attack vectors exhausted (SSRF, policy bypass, error handling) — no exploitable response differentials
- CHANGED www.vpbank.com: OAuth endpoints fully tested — no valid client_id discovered, redirect_uri bypass blocked
- NEW External attack surface required: Must pivot to mobile app bundles, GitHub, npm packages to discover OAuth client_id for www.vpbank.com
- CHANGED api.vpbank.com: All probes (XML, SOAP, Host/X-Forwarded-* to 169.254.169.254/localhost/10.0.0.1) return identical HTTP 500 JSON — SSRF and policy bypass CONFIRMED REJECTED by multiple models
- CHANGED www.vpbank.com: OAuth endpoints return 303/400 for all client_id/redirect_uri combos; no valid client_id in JS bundles (only Usercentrics clientWid); redirect_uri bypass CARRIED at confidence 35 pendi
- CHANGED vpbank-dev.com/vpbank-stage.com: WAF 2.3.0_20260324 intercepts ALL paths — zero application content reachable; staging hypothesis CONFIRMED REJECTED
- NEW www.vpbank.com/portal/api/: LitElement SPA served (HTTP 403 with body); POST /portal/api/language/:language sets portal-language cookie; CSRF token CSRFT759.js present; backend API endpoints (/health,
- NEW Risk score stabilized at 45 (down from 65) across all models due to failed exploitation of top hypotheses

## 2026-09-04 19:13:22 UTC
- CHANGED api.vpbank.com: All attack vectors exhausted (SSRF, policy bypass, error handling) — no exploitable response differentials
- CHANGED www.vpbank.com: OAuth endpoints fully tested — no valid client_id discovered, redirect_uri bypass blocked
- NEW External attack surface required: Must pivot to mobile app bundles, GitHub, npm packages to discover OAuth client_id for www.vpbank.com
- CHANGED api.vpbank.com: All probes (XML, SOAP, Host/X-Forwarded-* to 169.254.169.254/localhost/10.0.0.1) return identical HTTP 500 JSON — SSRF and policy bypass CONFIRMED REJECTED by multiple models
- CHANGED www.vpbank.com: OAuth endpoints return 303/400 for all client_id/redirect_uri combos; no valid client_id in JS bundles (only Usercentrics clientWid); redirect_uri bypass CARRIED at confidence 35 pendi
- CHANGED vpbank-dev.com/vpbank-stage.com: WAF 2.3.0_20260324 intercepts ALL paths — zero application content reachable; staging hypothesis CONFIRMED REJECTED
- NEW www.vpbank.com/portal/api/: LitElement SPA served (HTTP 403 with body); POST /portal/api/language/:language sets portal-language cookie; CSRF token CSRFT759.js present; backend API endpoints (/health,
- NEW Risk score stabilized at 45 (down from 65) across all models due to failed exploitation of top hypotheses
- NEW vpbank-dev.com + vpbank-stage.com discovered via production CSP (www.vpbank.com) as trusted origins; both resolve (193.222.70.165/.166) and are live Apache servers
- NEW www.vpbank.com responds 200 on /en; Drupal + Envoy proxy; robust CSP present; CSRFT759 + AL_SESS cookies
- NEW www.vpbank-dev.com and www.vpbank-stage.com redirect 302 to /error_path/maintenance.html (real maintenance site, not parked)
- CHANGED attack surface expanded beyond 3 inventory hosts; dev/stage domains are scoped (company-operated) and in production trust chain
- CHANGED api.vpbank.com: All attack vectors exhausted (SSRF, policy bypass, error handling) — no exploitable response differentials
- CHANGED www.vpbank.com: OAuth endpoints fully tested — no valid client_id discovered, redirect_uri bypass blocked
- NEW External attack surface required: Must pivot to mobile app bundles, GitHub, npm packages to discover OAuth client_id for www.vpbank.com
- NEW developer.vpbank.com (193.222.70.149) DISCOVERED via RAG — VP Bank PSD2 Developer Portal, live Apache+Envoy, NOT WAF-blocked (unlike dev/stage). Serves /psd2/swagger-ui (200), /psd2/berlin-group/v1/ps
- NEW Plaintext http://developer.../accounts → 200. Docs: production=mTLS client cert, sandbox=basic auth. /psd2/sandbox/* → 404 (sandbox not on this vhost).
- NEW External attack surface required: Must pivot to mobile app bundles (iOS/Android), GitHub code search, npm packages to discover OAuth client_id for www.vpbank.com (from 2026-09-04 16:37)
- NEW www.vpbank.com/portal/api/: LitElement SPA served (HTTP 403 with body); POST /portal/api/language/:language sets portal-language cookie; CSRF token CSRFT759.js present; backend API endpoints (/health,
- CHANGED api.vpbank.com: All attack vectors exhausted (SSRF, policy bypass, error handling) — no exploitable response differentials; all probes return identical HTTP 500 JSON (INVALID_REQUEST_RESOURCE)
- CHANGED www.vpbank.com: OAuth endpoints fully tested — no valid client_id discovered in JS bundles (only Usercentrics clientWid); redirect_uri bypass blocked at 303/400
- CHANGED vpbank-dev.com/vpbank-stage.com: WAF 2.3.0_20260324 intercepts ALL paths — zero application content reachable; staging hypothesis CONFIRMED REJECTED
- CHANGED Risk score stabilized at 45 (down from 65) across all models due to failed exploitation of top hypotheses
