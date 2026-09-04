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
