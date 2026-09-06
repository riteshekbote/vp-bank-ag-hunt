
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

## 2026-09-03 21:57:41 UTC
https://api.vpbank.com/ -> HTTP 500
https://api.vpbank.com/v1 -> HTTP 500
https://www.vpbank.com/.well-known/openid-configuration -> HTTP 400
https://www.vpbank.com.evil.com -> ERR <urlopen error [Errno -2] Name or service not know
https://www.vpbank.com/portal/api/language/en -> HTTP 403

## 2026-09-03 23:51:18 UTC
https://api.vpbank.com/ -> HTTP 500
https://www.vpbank.com/portal/api/language/en -> HTTP 403

## 2026-09-04 02:38:11 UTC
https://www.vpbank.com/.well-known/openid-configuration -> HTTP 400
https://www.vpbank.com/oauth/authorize?client_id=<valid>&redirect_uri=https://www.vpbank.com.evil.com&response_type=code&state=x -> HTTP 400
https://api.vpbank.com/ -> HTTP 500
https://api.vpbank.com/v1 -> HTTP 500

## 2026-09-04 07:28:50 UTC
https://www.vpbank.com/oauth/authorize?client_id=<valid>&redirect_uri=https://www.vpbank.com.evil.com&response_type=code&state=x -> HTTP 400
https://api.vpbank.com/ -> HTTP 500
https://www.vpbank.com/en -> HTTP 400

## 2026-09-04 12:21:00 UTC
https://api.vpbank.com/ -> HTTP 500
https://www.vpbank.com/oauth/authorize?client_id=<valid>&redirect_uri=https://www.vpbank.com.evil.com&response_type=code&state=test -> HTTP 400
https://api.vpbank.com/nonexistent -> HTTP 500

## 2026-09-04 16:37:47 UTC
https://www.vpbank.com/oauth/authorize?client_id=<valid>&redirect_uri=https://www.vpbank.com.evil.com&response_type=code&state=test -> HTTP 400
https://api.vpbank.com/ -> HTTP 500
https://api.vpbank.com/nonexistent -> HTTP 500
https://www.vpbank.com/portal/api/ -> HTTP 403
https://www.vpbank.com/portal/api/CSRFT759.js -> HTTP 403
https://www.vpbank.com/portal/api/graphql -> HTTP 403
https://www.vpbank.com/portal/api/graphql?query=__schema -> HTTP 403
https://api.vpbank.com/debug -> HTTP 500
https://api.vpbank.com/admin -> HTTP 500
https://api.vpbank.com/gateway -> HTTP 500
https://api.vpbank.com/config -> HTTP 500
https://api.vpbank.com/management -> HTTP 500

## 2026-09-04 19:13:41 UTC
https://www.vpbank.com/oauth/authorize?client_id=<valid>&redirect_uri=https://www.vpbank.com.evil.com&response_type=code&state=test -> HTTP 400
https://api.vpbank.com/ -> HTTP 500
https://www.vpbank.com/portal/api/ -> HTTP 403
https://www.vpbank.com/portal/api/CSRFT759.js -> HTTP 403
https://www.vpbank.com/portal/api/graphql -> HTTP 403
https://www.vpbank.com/portal/api/graphql?query=__schema -> HTTP 403
https://api.vpbank.com/debug -> HTTP 500
https://api.vpbank.com/admin -> HTTP 500
https://api.vpbank.com/gateway -> HTTP 500
https://api.vpbank.com/config -> HTTP 500
https://api.vpbank.com/management -> HTTP 500
https://api.vpbank.com/_internal -> HTTP 500

## 2026-09-04 21:38:40 UTC
https://www.vpbank.com/oauth/authorize?client_id=<valid>&redirect_uri=https://www.vpbank.com.evil.com&response_type=code&state=test -> HTTP 400
https://openbanking.vpbank.com -> ERR [SSL: TLSV13_ALERT_CERTIFICATE_REQUIRED] tlsv13 al
https://www.vpbank.com/oauth/authorize?client_id=<id>&response_type=code&redirect_uri= -> HTTP 400

## 2026-09-04 23:23:08 UTC
https://www.vpbank.com/oauth/authorize?client_id=<valid>&redirect_uri=https://www.vpbank.com.evil.com&response_type=code&state=test -> HTTP 400
https://www.vpbank.com/oauth/authorize?client_id=<id>&response_type=code&redirect_uri=<test>&state=x -> HTTP 400

## 2026-09-05 01:11:31 UTC
https://www.vpbank.com/oauth/authorize?client_id=<id>&response_type=code&redirect_uri=<test>&state=x -> HTTP 400
https://sts.vpbank.com/adfs -> HTTP 503
https://digital-onboarding-dev.vpbank.com/users/sign_in -> 200 len=0
https://digital-onboarding-dev.vpbank.com/admin/api/v1/users -> HTTP 401
https://digital-onboarding-dev.vpbank.com/api/v1/sessions/idp_login -> HTTP 404

## 2026-09-05 05:51:02 UTC
https://sts.vpbank.com/adfs -> HTTP 503
https://digital-onboarding.vpbank.com/users/sign_in -> 200 len=0
https://sts.vpbank.com/adfs/oauth2/token/devicecode -> HTTP 405
https://www.vpbank.com/portal/api/graphql -> HTTP 403
https://www.vpbank.com/portal/api/graphql?query=__schema -> HTTP 403
https://api.vpbank.com/debug -> HTTP 500
https://api.vpbank.com/admin -> HTTP 500
https://api.vpbank.com/gateway -> HTTP 500
https://api.vpbank.com/config -> HTTP 500
https://api.vpbank.com/management -> HTTP 500
https://api.vpbank.com/_internal -> HTTP 500
https://www.vpbank.com/oauth/authorize?client_id=<valid>&redirect_uri=https://www.vpbank.com.evil.com&response_type=code&state=test -> HTTP 400

## 2026-09-05 09:58:24 UTC
https://digital-onboarding.vpbank.com/users/sign_in -> 200 len=0
https://sts.vpbank.com/adfs -> HTTP 503
https://sts.vpbank.com/adfs/oauth2/token/devicecode -> HTTP 405
https://digital-onboarding.vpbank.com/api/v1/brand -> 200 len=0
https://digital-onboarding.vpbank.com/api/v1/users -> HTTP 404
https://digital-onboarding.vpbank.com/control-center/ -> 200 len=3167

## 2026-09-05 13:21:12 UTC
https://digital-onboarding.vpbank.com/users/sign_in -> 200 len=0
https://sts.vpbank.com/adfs -> HTTP 503
https://sts.vpbank.com/adfs/oauth2/token/devicecode -> HTTP 405
https://digital-onboarding.vpbank.com/api/v1/brand -> 200 len=0
https://digital-onboarding.vpbank.com/api/v1/users -> HTTP 404
https://digital-onboarding.vpbank.com/control-center/ -> 200 len=3167
https://digital-onboarding-dev.vpbank.com/api/v1/brand?force_tenant=vpbank -> 200 len=0

## 2026-09-05 16:14:06 UTC
https://digital-onboarding.vpbank.com/users/sign_in -> 200 len=0
https://sts.vpbank.com/adfs -> HTTP 503
https://sts.vpbank.com/adfs/oauth2/token/devicecode -> HTTP 405

## 2026-09-05 18:36:44 UTC
https://digital-onboarding.vpbank.com/users/sign_in -> 200 len=0
https://sts.vpbank.com/adfs -> HTTP 503
https://sts.vpbank.com/adfs/oauth2/token/devicecode -> HTTP 405
https://digital-onboarding.vpbank.com/api/v1/brand?force_tenant=vpbank -> 200 len=0
https://sts.vpbank.com/adfs/oauth2/devicecode -> HTTP 405

## 2026-09-05 20:50:30 UTC
https://digital-onboarding.vpbank.com/users/sign_in -> 200 len=0
https://sts.vpbank.com/adfs -> HTTP 503
https://sts.vpbank.com/adfs/oauth2/devicecode -> HTTP 405
https://digital-onboarding.vpbank.com/api/v1/onboarding_cases?force_tenant=vpbank -> HTTP 404
https://digital-onboarding.vpbank.com/api/v1/bankingtransactions?force_tenant=vpbank -> HTTP 404
https://digital-onboarding.vpbank.com/api/v1/incomingwire?force_tenant=vpbank -> HTTP 404
https://digital-onboarding.vpbank.com/api/v1/ident_documents?force_tenant=vpbank -> HTTP 404

## 2026-09-05 22:41:14 UTC
https://digital-onboarding.vpbank.com/users/sign_in -> 200 len=0
https://digital-onboarding.vpbank.com/api/v1/onboarding_cases?force_tenant=vpbank -> HTTP 404
https://digital-onboarding.vpbank.com/api/v1/bankingtransactions?force_tenant=vpbank -> HTTP 404
https://digital-onboarding.vpbank.com/api/v1/incomingwire?force_tenant=vpbank -> HTTP 404
https://digital-onboarding.vpbank.com/api/v1/ident_documents?force_tenant=vpbank -> HTTP 404
https://sts.vpbank.com/adfs -> HTTP 503
https://sts.vpbank.com/adfs/oauth2/devicecode -> HTTP 405
https://digital-onboarding-dev.vpbank.com/api/v1/brand?force_tenant=vpbank -> 200 len=0
https://sts.vpbank.com/adfs/oauth2/token/devicecode -> HTTP 405
https://digital-onboarding.vpbank.com/api/v1/brand?force_tenant=vpbank -> 200 len=0
https://digital-onboarding-dev.vpbank.com/users/sign_in -> 200 len=0

## 2026-09-06 00:26:17 UTC
https://digital-onboarding.vpbank.com/users/sign_in -> 200 len=0
https://digital-onboarding.vpbank.com/api/v1/qr_codes/generate?force_tenant=vpbank -> HTTP 401
https://digital-onboarding.vpbank.com/api/v1/tenants?force_tenant=vpbank -> HTTP 403
https://sts.vpbank.com/adfs -> HTTP 503
https://sts.vpbank.com/adfs/oauth2/devicecode -> HTTP 405
https://digital-onboarding-dev.vpbank.com/users/sign_in -> 200 len=0
https://digital-onboarding-dev.vpbank.com/admin/api/v1/bankingtransactions?force_tenant=vpbank -> HTTP 404

## 2026-09-06 04:47:23 UTC
https://digital-onboarding.vpbank.com/users/sign_in -> 200 len=0
https://digital-onboarding.vpbank.com/api/v1/qr_codes/generate?force_tenant=vpbank -> HTTP 401
https://digital-onboarding.vpbank.com/api/v1/tenants?force_tenant=vpbank -> HTTP 403
https://sts.vpbank.com/adfs -> HTTP 503
https://sts.vpbank.com/adfs/oauth2/devicecode -> HTTP 405
https://digital-onboarding-dev.vpbank.com/admin/api/v1/bankingtransactions?force_tenant=<other -> HTTP 400

## 2026-09-06 09:14:50 UTC
https://digital-onboarding.vpbank.com/users/sign_in -> 200 len=0
https://digital-onboarding.vpbank.com/api/v1/qr_codes/generate?force_tenant=vpbank -> HTTP 401
https://digital-onboarding.vpbank.com/api/v1/tenants?force_tenant=vpbank -> HTTP 403
https://sts.vpbank.com/adfs -> HTTP 503
https://sts.vpbank.com/adfs/oauth2/devicecode -> HTTP 405
https://digital-onboarding-dev.vpbank.com/admin/api/v1/bankingtransactions?force_tenant=<other -> HTTP 400
https://digital-onboarding-dev.vpbank.com/users/sign_in -> 200 len=0
https://digital-onboarding-dev.vpbank.com/api/v1/current_user_details -> HTTP 404

## 2026-09-06 13:00:15 UTC
https://digital-onboarding.vpbank.com/users/sign_in -> 200 len=0
https://digital-onboarding.vpbank.com/api/v1/qr_codes/generate?force_tenant=vpbank -> HTTP 401
https://digital-onboarding.vpbank.com/api/v1/tenants?force_tenant=vpbank -> HTTP 403
https://sts.vpbank.com/adfs -> HTTP 503
https://sts.vpbank.com/adfs/oauth2/devicecode -> HTTP 405

## 2026-09-06 16:03:38 UTC
https://digital-onboarding.vpbank.com/users/sign_in -> 200 len=0
https://digital-onboarding.vpbank.com/api/v1/current_user_details?force_tenant=vpbank -> HTTP 404
https://sts.vpbank.com/adfs -> HTTP 503
https://sts.vpbank.com/adfs/oauth2/devicecode -> HTTP 405
https://digital-onboarding-dev.vpbank.com/admin/api/v1/bankingtransactions?force_tenant=<other -> HTTP 400
https://digital-onboarding.vpbank.com/rails/active_storage/direct_uploads -> HTTP 404
https://digital-onboarding-dev.vpbank.com/users/sign_in -> 200 len=0
https://digital-onboarding-dev.vpbank.com/api/v1/current_user_details -> HTTP 404
https://digital-onboarding-dev.vpbank.com/admin/api/v1/users -> HTTP 401

## 2026-09-06 18:10:35 UTC
https://digital-onboarding.vpbank.com/users/sign_in -> 200 len=0
https://sts.vpbank.com/adfs -> HTTP 503
https://sts.vpbank.com/adfs/oauth2/devicecode -> HTTP 405

## 2026-09-06 20:35:07 UTC
https://digital-onboarding.vpbank.com/users/sign_in -> 200 len=0
https://digital-onboarding-dev.vpbank.com/users/sign_in -> 200 len=0
https://sts.vpbank.com/adfs -> HTTP 503
https://sts.vpbank.com/adfs/oauth2/devicecode -> HTTP 405
