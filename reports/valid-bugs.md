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
