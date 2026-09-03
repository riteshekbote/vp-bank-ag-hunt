
## 2026-09-02 21:53:27 UTC


## 2026-09-02 23:50:48 UTC


## 2026-09-03 02:36:22 UTC


## 2026-09-03 07:29:02 UTC


## 2026-09-03 12:18:41 UTC


## 2026-09-03 16:34:13 UTC
https://api.vpbank.com/v1 -> HTTP 500
https://api.vpbank.com/v2 -> HTTP 500
https://api.vpbank.com/swagger.json -> HTTP 500
https://api.vpbank.com/openapi.json -> HTTP 500
https://api.vpbank.com/actuator/health -> HTTP 500
https://api.vpbank.com/.well-known/security.txt -> HTTP 500
https://vpbank.com/.well-known/openid-configuration -> HTTP 404
https://vpbank.com/oauth/authorize?client_id=test&redirect_uri=https://evil.com&response_type=code&state=x -> HTTP 404

## 2026-09-03 19:30:25 UTC
https://api.vpbank.com/ -> HTTP 500
https://api.vpbank.com/v1 -> HTTP 500
https://www.vpbank.com/.well-known/openid-configuration -> HTTP 400
https://www.vpbank.com.evil.com -> ERR <urlopen error [Errno -2] Name or service not know
https://www.vpbank.com/en -> HTTP 400
