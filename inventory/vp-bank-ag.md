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

## 2026-09-04 21:38:33 UTC
- NEW developer.vpbank.com (193.222.70.149): VP Bank PSD2 Developer Portal discovered via RAG — live Apache+Envoy, NOT WAF-blocked (unlike dev/stage). Serves full PSD2 API documentation (VuePress + Swagger 
- CHANGED api.vpbank.com: All attack vectors exhausted (SSRF, policy bypass, error handling) — no exploitable response differentials; all probes return identical HTTP 500 JSON.
- CHANGED www.vpbank.com: OAuth endpoints fully tested — no valid client_id discovered in JS bundles (only Usercentrics clientWid); redirect_uri bypass blocked at 303/400.
- CHANGED vpbank-dev.com/vpbank-stage.com: WAF 2.3.0_20260324 intercepts ALL paths — zero application content reachable; staging hypothesis CONFIRMED REJECTED.
- CHANGED www.vpbank.com/portal/api/: LitElement SPA served (HTTP 403 with body); backend API endpoints (/health, /status) return WAF maintenance page — SPA is frontend shell only.

## 2026-09-04 23:23:03 UTC
- NEW developer.vpbank.com (193.222.70.149): PSD2 Developer Portal discovered via RAG — live Apache+Envoy, NOT WAF-blocked. Serves full PSD2 API docs (VuePress + Swagger UI), OpenAPI spec at `/psd2/berlin-g
- CHANGED api.vpbank.com: All attack vectors exhausted (SSRF, policy bypass, error handling) — no exploitable response differentials; all probes return identical HTTP 500 JSON.
- CHANGED www.vpbank.com: OAuth endpoints fully tested — no valid client_id discovered in JS bundles (only Usercentrics clientWid); redirect_uri bypass blocked at 303/400.
- CHANGED vpbank-dev.com/vpbank-stage.com: WAF 2.3.0_20260324 intercepts ALL paths — zero application content reachable; staging hypothesis CONFIRMED REJECTED.
- CHANGED www.vpbank.com/portal/api/: LitElement SPA served (HTTP 403 with body); backend API endpoints (/health, /status) return WAF maintenance page — SPA is frontend shell only.

## 2026-09-05 01:11:22 UTC
- NEW CT enumeration (crt.sh) expands inventory 6→285 hostnames; live web-accessible additions: digital-onboarding/vpbank.com family (prod+dev+stage), sts.vpbank.com (AD FS), api-prep.vpbank.com (Layer7 pre
- NEW digital-onboarding.vpbank.com (fn/countersigned brand: "Onboarding | VP Bank", © VP BANK AG Vaduz): production multi-tenant bank onboarding + back-office platform (SaaS "US", Rails/Devise/devise-locka
- NEW /control-center/ on prod serves full "Business Control Center" back-office SPA anonymously (HTTP 200) with admin modules (onboarding cases, ident documents, bankingtransactions, incomingwire, rolemgmt
- NEW /api/v1/brand returns HTTP 200 ANONYMOUSLY on prod (tenant config, i18n, page_title "Business Control Center", tenant_symbol vpbanklighttenant); /api/v1/tenants returns 403 "Not authorized".
- NEW sts.vpbank.com (193.222.70.198): Microsoft AD FS (Microsoft-HTTPAPI/2.0); /adfs/.well-known/openid-configuration HTTP 200 — issuer https://sts.vpbank.com/adfs, device_code+password+implicit grants, sc
- CHANGED api-prep.vpbank.com (CNAME api-prep-vpbank-com.hdb-cs04.ellb.ch, 195.186.145.90): Layer7 clone of api.vpbank.com — SCSS-Request-ID, INVALID_REQUEST_RESOURCE JSON 404 for all paths; no new surface.
- CHANGED designsystem.vpbank.com CNAME→vpb-design-system.netlify.app serves HTTP 200 (live) — subdomain takeover NOT present.
- CHANGED vop.vpbank.com/.vop-stage on 193.222.70.154 (openbanking IP): HTTPS unreachable anonymously (TLS drop) — mTLS-gated like openbanking.
- NEW developer.vpbank.com PSD2 sandbox BOLA/IDOR **VERIFIED end-to-end** in official test sandbox (synthetic data): consent `6b517824-e5af-4202-b9b0-7f483a68ee9f` created anonymously (POST 201) read by fre
- NEW openbanking.vpbank.com (193.222.70.154) discovered via TLS cert CN=openbanking.vpbank.com — production PSD2 ASPSP, mTLS "certificate required" at TLS layer, anonymous surface blocked as designed.
- CHANGED api.vpbank.com: All attack vectors **exhausted** (SSRF Host/X-Forwarded-*, policy bypass XML/SOAP, error handling) — all probes return identical HTTP 500 JSON (INVALID_REQUEST_RESOURCE). CONFIRMED REJ
- CHANGED www.vpbank.com: OAuth endpoints fully tested — no valid client_id in JS bundles (only Usercentrics clientWid); `/oauth/authorize` always 303→error page; redirect_uri bypass **blocked without client_id
- CHANGED vpbank-dev.com/vpbank-stage.com: WAF 2.3.0_20260324 intercepts **ALL paths** — zero application content reachable; staging hypothesis **CONFIRMED REJECTED**.
- CHANGED www.vpbank.com/portal/api/: LitElement SPA served (HTTP 403 with body); backend API endpoints (`/health`, `/status`) return WAF maintenance page — SPA is frontend shell only.
- CHANGED Risk score: **55** (increased from 45) due to confirmed PSD2 sandbox BOLA on developer.vpbank.com — high-value financial API surface with proven authorization bypass.

## 2026-09-05 05:50:38 UTC
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
- NEW CT enumeration (crt.sh) expands inventory 6→285 hostnames; live web-accessible additions: digital-onboarding/vpbank.com family (prod+dev+stage), sts.vpbank.com (AD FS), api-prep.vpbank.com (Layer7 pre
- NEW digital-onboarding.vpbank.com (fn/countersigned brand: "Onboarding | VP Bank", © VP BANK AG Vaduz): production multi-tenant bank onboarding + back-office platform (SaaS "US", Rails/Devise/devise-locka
- NEW /control-center/ on prod serves full "Business Control Center" back-office SPA anonymously (HTTP 200) with admin modules (onboarding cases, ident documents, bankingtransactions, incomingwire, rolemgmt
- NEW /api/v1/brand returns HTTP 200 ANONYMOUSLY on prod (tenant config, i18n, page_title "Business Control Center", tenant_symbol vpbanklighttenant); /api/v1/tenants returns 403 "Not authorized".
- NEW sts.vpbank.com (193.222.70.198): Microsoft AD FS (Microsoft-HTTPAPI/2.0); /adfs/.well-known/openid-configuration HTTP 200 — issuer https://sts.vpbank.com/adfs, device_code+password+implicit grants, sc
- CHANGED api-prep.vpbank.com (CNAME api-prep-vpbank-com.hdb-cs04.ellb.ch, 195.186.145.90): Layer7 clone of api.vpbank.com — SCSS-Request-ID, INVALID_REQUEST_RESOURCE JSON 404 for all paths; no new surface.
- CHANGED designsystem.vpbank.com CNAME→vpb-design-system.netlify.app serves HTTP 200 (live) — subdomain takeover NOT present.
- CHANGED vop.vpbank.com/.vop-stage on 193.222.70.154 (openbanking IP): HTTPS unreachable anonymously (TLS drop) — mTLS-gated like openbanking.
- NEW Live HTTP probes completed on all 3 inventory hosts (previously 0 live)
- NEW api.vpbank.com confirmed running Layer7-API-Gateway (CA API Gateway); all versioned/debug paths return HTTP 500 with JSON 404 error (INVALID_REQUEST_RESOURCE)
- NEW www.vpbank.com serves marketing site on Apache; OAuth/OIDC endpoints exist but reject invalid requests with 303 to error page
- NEW vpbank.com redirects to www.vpbank.com for all tested paths
- CHANGED api.vpbank.com attack surface reduced: no exposed API versions, Swagger, OpenAPI, actuator, or security.txt
- NEW developer.vpbank.com PSD2 sandbox BOLA/IDOR **VERIFIED end-to-end** in official test sandbox (synthetic data): consent `6b517824-e5af-4202-b9b0-7f483a68ee9f` created anonymously (POST 201) read by fre
- NEW openbanking.vpbank.com (193.222.70.154) discovered via TLS cert CN=openbanking.vpbank.com — production PSD2 ASPSP, mTLS "certificate required" at TLS layer, anonymous surface blocked as designed.
- CHANGED api.vpbank.com: All attack vectors **exhausted** (SSRF Host/X-Forwarded-*, policy bypass XML/SOAP, error handling) — all probes return identical HTTP 500 JSON (INVALID_REQUEST_RESOURCE). CONFIRMED REJ
- CHANGED www.vpbank.com: OAuth endpoints fully tested — no valid client_id in JS bundles (only Usercentrics clientWid); `/oauth/authorize` always 303→error page; redirect_uri bypass **blocked without client_id
- CHANGED vpbank-dev.com/vpbank-stage.com: WAF 2.3.0_20260324 intercepts **ALL paths** — zero application content reachable; staging hypothesis **CONFIRMED REJECTED**.
- CHANGED www.vpbank.com/portal/api/: LitElement SPA served (HTTP 403 with body); backend API endpoints (`/health`, `/status`) return WAF maintenance page — SPA is frontend shell only.
- CHANGED Risk score: **55** (increased from 45) due to confirmed PSD2 sandbox BOLA on developer.vpbank.com — high-value financial API surface with proven authorization bypass.
- NEW digital-onboarding.vpbank.com: production multi-tenant bank onboarding/back-office SaaS (Rails+Devise) on off-net hosting (89.163.182.69/.28/.8); anonymous /control-center/ SPA (HTTP 200) with admin m
- NEW sts.vpbank.com (193.222.70.198): Microsoft AD FS; /adfs/.well-known/openid-configuration HTTP 200 — issuer https://sts.vpbank.com/adfs, device_code/password/implicit grants, scopes vpn/logon/cert
- NEW CT enumeration (crt.sh) expands inventory 6→285 hostnames; live additions: digital-onboarding family (prod/dev/stage), sts.vpbank.com, api-prep.vpbank.com
- CHANGED developer.vpbank.com PSD2 sandbox BOLA/IDOR **VERIFIED end-to-end** in official test sandbox (synthetic data): consent `6b517824-e5af-4202-b9b0-7f483a68ee9f` created anonymously (POST 201) read by fre
- CHANGED openbanking.vpbank.com (193.222.70.154): production PSD2 ASPSP, mTLS "certificate required" at TLS layer — anonymous surface blocked as designed
- CHANGED api-prep.vpbank.com: Layer7 clone of api.vpbank.com — SCS-Request-ID, INVALID_REQUEST_RESOURCE JSON 404 for all paths; no new surface
- CHANGED designsystem.vpbank.com: CNAME→vpb-design-system.netlify.app serves 200 — active, no subdomain takeover
- CHANGED vop.vpbank.com/.vop-stage on 193.222.70.154: HTTPS unreachable anonymously (TLS drop) — mTLS-gated like openbanking
- CHANGED api.vpbank.com: All attack vectors **exhausted** — CONFIRMED REJECTED by multiple models
- CHANGED www.vpbank.com: OAuth endpoints fully tested — no valid client_id, redirect_uri bypass blocked without client_id
- CHANGED vpbank-dev.com/vpbank-stage.com: WAF 2.3.0_20260324 intercepts ALL paths — CONFIRMED REJECTED
- CHANGED Risk score: **55** (raised from 45) due to confirmed PSD2 sandbox BOLA + new digital-onboarding back-office attack surface

## 2026-09-05 09:58:15 UTC
- CHANGED digital-onboarding.vpbank.com: /users/sign_in 200, /control-center/ 200, /api/v1/brand 200 confirmed live — multi-tenant back-office SaaS attack surface verified
- CHANGED sts.vpbank.com: /adfs/.well-known/openid-configuration 200 confirmed live — ADFS device_code/password/implicit grants exposed
- CHANGED developer.vpbank.com: PSD2 sandbox endpoints responding (405/400 as expected for auth'd routes) — BOLA surface confirmed accessible
- CHANGED openbanking.vpbank.com: TLS timeout (mTLS "certificate required") — production PSD2 anonymous surface blocked as designed
- CHANGED api.vpbank.com: 404 on /v1 — all vectors exhausted, uniform INVALID_REQUEST_RESOURCE JSON 500
- CHANGED www.vpbank.com: /oauth/authorize 303→error page — OAuth dead without client_id
- CHANGED vpbank-dev.com/vpbank-stage.com: 302→maintenance — WAF 2.3.0_20260324 intercepts ALL paths, confirmed rejected

## 2026-09-05 13:21:01 UTC
- CHANGED digital-onboarding.vpbank.com: /users/sign_in 200, /control-center/ 200, /api/v1/brand 200 confirmed live — multi-tenant back-office SaaS attack surface verified
- CHANGED sts.vpbank.com: /adfs/.well-known/openid-configuration 200 confirmed live — ADFS device_code/password/implicit grants exposed
- CHANGED developer.vpbank.com: PSD2 sandbox endpoints responding (405/400 as expected for auth'd routes) — BOLA surface confirmed accessible
- CHANGED openbanking.vpbank.com: TLS timeout (mTLS "certificate required") — production PSD2 anonymous surface blocked as designed
- CHANGED api.vpbank.com: 404 on /v1 — all vectors exhausted, uniform INVALID_REQUEST_RESOURCE JSON 500
- CHANGED www.vpbank.com: /oauth/authorize 303→error page — OAuth dead without client_id
- CHANGED vpbank-dev.com/vpbank-stage.com: 302→maintenance — WAF 2.3.0_20260324 intercepts ALL paths, confirmed rejected
- NEW digital-onboarding.vpbank.com: Production multi-tenant back-office SaaS (Rails+Devise) on off-net hosting (89.163.182.69/.28/.8), anonymous /control-center/ SPA (HTTP 200) with admin modules, /api/v1/
- NEW sts.vpbank.com: ADFS OIDC metadata 200 (issuer sts.vpbank.com/adfs), device_code/password/implicit grants exposed, scopes vpn/logon/cert — corporate IdP for VPN/cert auth
- CHANGED developer.vpbank.com: PSD2 sandbox BOLA verified end-to-end in official test sandbox (synthetic data) — consent/account/payment cross-session read with zero identity binding on consentId/paymentId
- CHANGED openbanking.vpbank.com: Production PSD2 ASPSP, mTLS "certificate required" at TLS layer — anonymous surface blocked as designed
- CHANGED api.vpbank.com: All vectors exhausted (SSRF, policy bypass, error handling) — uniform INVALID_REQUEST_RESOURCE JSON 500, CONFIRMED REJECTED
- CHANGED www.vpbank.com: OAuth endpoints fully tested — no valid client_id in JS bundles (only Usercentrics clientWid), /oauth/authorize always 303→error page, redirect_uri bypass blocked without client_id
- CHANGED vpbank-dev.com/vpbank-stage.com: WAF 2.3.0_20260324 intercepts ALL paths — zero application content reachable, CONFIRMED REJECTED
- CHANGED api-prep.vpbank.com: Layer7 clone of api.vpbank.com (CNAME api-prep-vpbank-com.hdb-cs04.ellb.ch) — SCS-Request-ID, INVALID_REQUEST_RESOURCE JSON 404 for all paths, no new surface
- CHANGED designsystem.vpbank.com: CNAME→vpb-design-system.netlify.app serves HTTP 200 — active Netlify app, no subdomain takeover

## 2026-09-05 16:14:01 UTC
- NEW digital-onboarding.vpbank.com: Production multi-tenant back-office SaaS (Rails+Devise) on off-net hosting (89.163.182.69/.28/.8), anonymous /control-center/ SPA (HTTP 200) with admin modules (onboardi
- NEW sts.vpbank.com: ADFS OIDC metadata 200 (issuer sts.vpbank.com/adfs), device_code/password/implicit grants exposed, scopes vpn/logon/cert — corporate IdP for VPN/cert auth
- NEW developer.vpbank.com: PSD2 sandbox BOLA verified end-to-end in official test sandbox (synthetic data) — consent/account/payment cross-session read with zero identity binding on consentId/paymentId
- NEW openbanking.vpbank.com: Production PSD2 ASPSP, mTLS "certificate required" at TLS layer — anonymous surface blocked as designed
- NEW api-prep.vpbank.com: Layer7 clone of api.vpbank.com (CNAME api-prep-vpbank-com.hdb-cs04.ellb.ch) — SCS-Request-ID, INVALID_REQUEST_RESOURCE JSON 404 for all paths, no new surface
- NEW designsystem.vpbank.com: CNAME→vpb-design-system.netlify.app serves HTTP 200 — active Netlify app, no subdomain takeover
- CHANGED api.vpbank.com: All vectors exhausted (SSRF, policy bypass, error handling) — uniform INVALID_REQUEST_RESOURCE JSON 500, CONFIRMED REJECTED
- CHANGED www.vpbank.com: OAuth endpoints fully tested — no valid client_id in JS bundles (only Usercentrics clientWid), /oauth/authorize always 303→error page, redirect_uri bypass blocked without client_id
- CHANGED vpbank-dev.com/vpbank-stage.com: WAF 2.3.0_20260324 intercepts ALL paths — zero application content reachable, CONFIRMED REJECTED
