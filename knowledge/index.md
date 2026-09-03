# Knowledge Base (seed)
- 2026-09-03 REJECTED MISCONFIG @ api.vpbank.com (subdomain takeover): ellb.ch is active Swiss load balancer, not a decommissioned cloud provider tenant
- 2026-09-03 REJECTED MISCONFIG @ api.vpbank.com (API versioning/debug endpoints): All tested paths (/v1,/v2,/swagger.json,/openapi.json,/actuator/health,/beta,/internal,/.well-known/security.txt) return HTTP 500 with JSON 404 error - no exposed API contracts
- 2026-09-03 REJECTED OAUTH @ vpbank.com (OIDC discovery): .well-known/openid-configuration redirects to www.vpbank.com which returns 303 error page - no OIDC metadata exposed
- 2026-09-03 ACCEPTED MISCONFIG @ api.vpbank.com (Layer7 gateway error handling): Gateway returns structured JSON errors with request IDs - potential info leak vector
- 2026-09-03 ACCEPTED MISCONFIG @ vpbank-dev.com/vpbank-stage.com: production CSP trusts these domains; publicly reachable Apache servers behind maintenance redirect - NEW in-scope staging attack surface
- 2026-09-03 REJECTED OAUTH @ vpbank.com (OIDC discovery): .well-known/openid-configuration redirects to www.vpbank.com which returns 303/400 error page - no OIDC metadata exposed
- 2026-09-03 REJECTED MISCONFIG @ vpbank-dev.com/vpbank-stage.com: WAF intercepts ALL paths, returns maintenance page - no application content accessible
- 2026-09-03 NEW INFO: WAF version 2.3.0_20260324 exposed in maintenance page responses
