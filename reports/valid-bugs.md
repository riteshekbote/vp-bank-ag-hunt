# Validated findings (running count 0)

- 1 lead(s) marked VALID at 2026-09-04 04:44:56 UTC
  - | 2 | OAuth redirect_uri bypass | **HOLD** | HTTP 400 = ambiguous; needs valid client_id + bypass proof |

- 6 lead(s) marked VALID at 2026-09-04 21:52:32 UTC
  - | Q4 | Provable non-invasively? | **No** — requires valid client_id first |
  - | Q7 | Would reasonable triager accept? | **Cannot determine** — requires valid client_id not discoverable passively |
  - **Verdict: HOLD** — AUTH_HELPED testability blocks passive-only verification. /oauth/authorize exists but returns 303/400 for all tested client_id/redirect_uri combos. No valid client_id discoverable 
  - | Q7 | Would reasonable triager accept? | **Possibly** — if confirmed, valid finding |
  - **Verdict: HOLD** — PASSIVE testability but requires POST requests to create payments/consents. Documentation explicitly states last digit of X-Request-ID controls sandbox state (1=RCVD, 5=ACSC for pa
  - | **VALID** | 0 | None confirmed |

- 1 lead(s) marked VALID at 2026-09-05 07:45:18 UTC
  - **Verdict: VALID**

- 1 lead(s) marked VALID at 2026-09-07 11:39:24 UTC
  - **Verdict: HOLD** — High-severity if exploitable (CVSS ~8.8), but requires POST with valid credentials to prove. Passive proof only shows the form fields exist. Needs: POST `user[admin]=true&user[tena

- 2 lead(s) marked VALID at 2026-09-07 20:10:45 UTC
  - **Verdict: VALID**
  - | 1 | PSD2 Sandbox BOLA (developer.vpbank.com) | **VALID** | 5.3–7.5 | Report to program |

- 3 lead(s) marked VALID at 2026-09-08 05:37:40 UTC
  - **Verdict: VALID**
  - | Q7 | Reasonable triager? | **NO** — metadata exposure alone is informational, not exploitable without valid client_id + user interaction |
  - | 1 | PSD2 Sandbox BOLA (developer.vpbank.com) | **VALID** | 7.5 | Report to program |

- 3 lead(s) marked VALID at 2026-09-10 16:57:16 UTC
  - **VERDICT: VALID**
  - | Q4 | Provable non-invasively? | **No** — requires valid client_id not discoverable from JS/RAG/App Store |
  - | PSD2 Sandbox BOLA/IDOR | **VALID** | Proven end-to-end in sandbox with synthetic data; HIGH impact |

- 4 lead(s) marked VALID at 2026-09-13 14:30:15 UTC
  - **Verdict: VALID**
  - | Q4 Provable non-invasively? | **PARTIAL** — HTML form evidence confirms field existence; controller customization confirmed via code analysis; but POST with forged params **requires a valid dev cred
  - | Q2 Attacker reachable? | YES — `/.well-known/openid-configuration` returns HTTP 200; device_code grant exposed; but `/adfs` returns HTTP 503 (service degraded); `/adfs/oauth2/devicecode` returns 405
  - | PSD2 sandbox BOLA @ developer.vpbank.com | **VALID** | 7.5 High | **Report to bugs.olivermaicher.eu** with synthetic-data PoC |

- 24 lead(s) marked VALID at 2026-09-13 17:39:02 UTC
  - | **Q1 Scope** | VALID — `developer.vpbank.com` is *.vpbank.com, company-owned PSD2 developer portal under "all company-owned infrastructure" |
  - | **Q2 Reach** | VALID — publicly reachable, anonymous POST to `/psd2/berlin-group/v1/consents` returns 201; no client cert or basic auth required |
  - | **Q3 Impact** | VALID — cross-session consent/account/balance/transaction/payment-status read without identity binding; if prod mirrors sandbox authz logic, cross-TPP financial data disclosure |
  - | **Q4 Provable** | VALID — already proven end-to-end in sandbox (synthetic data): consent POST→201, fresh anonymous GET→200 across /consents/{id}, /accounts, /balances, /transactions, /payments/{id}/
  - | **Q5 Novel** | VALID — no public disclosure found; spec self-labels server "PSD2 production server" suggesting this is untested behavior |
  - | **Q6 Not always-rejected** | VALID — not on always-rejected list (not DoS, not info leak, not clickjacking, etc.) |
  - | **Q7 Triager accept** | VALID — authorization bypass on a financial API is a real finding; caveat is sandbox-only (synthetic data) reduces severity |
  - **Verdict: VALID**
  - | **Q1 Scope** | VALID — `digital-onboarding.vpbank.com` is *.vpbank.com, company-owned multi-tenant bank onboarding/back-office platform |
  - | **Q2 Reach** | VALID — publicly reachable; `/users/sign_in` serves HTTP 200 with form containing hidden fields |
  - | **Q3 Impact** | VALID — if consumed, attacker-controlled `user[admin]`, `user[tenant_id]`, `user[user_id]` could grant admin/impersonation on bank back-office |
  - | **Q5 Novel** | VALID — custom overridden SessionsController consuming client-controlled params is non-standard Devise behavior |
  - | **Q6 Not always-rejected** | VALID — not on always-rejected list |
  - | **Q1 Scope** | VALID — *.vpbank-dev.com / *.vpbank-stage.com are trusted origins in production CSP |
  - | **Q2 Reach** | VALID — DNS resolves, Apache responds with maintenance redirect |
  - | **Q1 Scope** | VALID |
  - | **Q2 Reach** | VALID |
  - | **Q1 Scope** | VALID |
  - | **Q2 Reach** | VALID |
  - | **Q3 Impact** | VALID (if proven) |

- 2 lead(s) marked VALID at 2026-09-16 18:14:17 UTC
  - **VERDICT: VALID**
  - | PSD2 BOLA (developer) | VALID | 8.6 | Report to program |

- 2 lead(s) marked VALID at 2026-09-18 21:28:18 UTC
  - | **Q4 Provable** | NO — requires valid dev/stage test credentials to POST sign-in with mass-assignment params. Cannot prove controller accepts/ignores admin param without authentication. Hidden field
  - | **Q4 Provable** | PARTIALLY — metadata endpoint visible, but device_code grant requires valid client_id (unknown) |
