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
