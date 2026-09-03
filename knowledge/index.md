# Knowledge Base (seed)
- 2026-09-03 REJECTED MISCONFIG @ api.vpbank.com (subdomain takeover): ellb.ch is active Swiss load balancer, not a decommissioned cloud provider tenant
- 2026-09-03 REJECTED MISCONFIG @ api.vpbank.com (API versioning/debug endpoints): All tested paths (/v1,/v2,/swagger.json,/openapi.json,/actuator/health,/beta,/internal,/.well-known/security.txt) return HTTP 500 with JSON 404 error - no exposed API contracts
- 2026-09-03 REJECTED OAUTH @ vpbank.com (OIDC discovery): .well-known/openid-configuration redirects to www.vpbank.com which returns 303 error page - no OIDC metadata exposed
- 2026-09-03 ACCEPTED MISCONFIG @ api.vpbank.com (Layer7 gateway error handling): Gateway returns structured JSON errors with request IDs - potential info leak vector
