# Draft - Vault GCP Auth Engine Configuration Guide

 - Step 1 : Create a GCP IAM Service Account

``` bash
gcloud iam service-accounts create vault-auth \
      --display-name "Vault Authentication Service Account"
```
Output
``` bash
Created service account [vault-auth].
```

 - Step 2 : Add policies to newly created IAM Service Account

``` bash
gcloud iam service-accounts add-iam-policy-binding \
      vault-auth@allthingscloud-graham-221213.iam.gserviceaccount.com \
      --member='user:graham@hashicorp.com' --role='roles/iam.serviceAccountTokenCreator'

gcloud iam service-accounts add-iam-policy-binding \
      vault-auth@allthingscloud-graham-221213.iam.gserviceaccount.com \
      --member='user:graham@hashicorp.com' --role='roles/iam.serviceAccountKeyAdmin'
```
Output
``` bash
bindings:
- members:
  - user:graham@hashicorp.com
  role: roles/iam.serviceAccountKeyAdmin
- members:
  - user:graham@hashicorp.com
  role: roles/iam.serviceAccountTokenCreator
etag: BwV6k3gJs3s=
```

 - Step 3 : Enable GCP Auth backend

``` bash
# setup vault environment variables
VAULT_IP="192.168.2.11"
VAULT_TOKEN=`cat .vault-token`
VAULT_ADDR="http://${VAULT_IP}:8200"

# enable GCP auth backend json file
tee enable_gcp_auth.json <<EOF
{
  "type": "gcp",
  "description": "Login with Google Cloud Credentials"
}
EOF
  
# enable GCP auth backend
curl \
    --header "X-Vault-Token: ${VAULT_TOKEN}" \
    --request POST \
    --data @enable_gcp_auth.json \
    ${VAULT_ADDR}/v1/sys/auth/gcp
```
Output
``` bash

```

 - Step 4 : Set the Vault GCP Credentials

``` bash
GOOGLE_PROJECT=$(gcloud config get-value project)
SERVICE_ACCOUNT=vault-auth@${GOOGLE_PROJECT}.iam.gserviceaccount.com

# Create credentials file for gcp service account
gcloud iam service-accounts keys create ../vault-demo-creds.json --iam-account vault-auth@allthingscloud-graham-221213.iam.gserviceaccount.com
```
Output
``` bash
created key [ed48e222c25060771fc1eddf1c2be4eceefdd3dc] of type [json] as [../vault-demo-creds.json] for [vault-auth@allthingscloud-graham-221213.iam.gserviceaccount.com]
```
``` bash
sed -i '.bak' 's/\"/\\\"/g; s/\\n/\\\\n/g' ../vault-demo-creds.json
GCP_CREDENTIALS_FILE=`cat ../vault-demo-creds.json`

# configure gcp auth credentials json file
tee configure_gcp_auth_credentials.json <<EOF
{
"credentials": "${GCP_CREDENTIALS_FILE}"
}
EOF
```
Output
``` json
{
"credentials": "{
  \"type\": \"service_account\",
  \"project_id\": \"allthingscloud-graham-221213\",
  \"private_key_id\": \"ed48e222c25060771fc1eddf1c2be4eceefdd3dc\",
  \"private_key\": \"-----BEGIN PRIVATE KEY-----\\nMIIEvgIBADANBgkqhkiG9w0BAQEFAASCBKgwggSkAgEAAoIBAQDAHAtFXDsmtx/2\\nf3uo0xK83v704dCQ/7+EKDRHUaOiL1mhxDoffyfT0ng3JBZeXK3EusSe8Cl4XOz1\\nwi+VOmXP/HTPdehkIGDKEGfr/aBTOV1iAU/32NajP0UAv002rkwb0ls8sGXj5mpN\\n16oSHZecGE1MhoOsZ2h+43KjbEKLtMD4oVVi8o+TlG03VwrzgP67+iMzYcePcrcS\\ni3PLu3LmFvmXUpuZKmUHgrt3ev0NMjapm6Lrat+i2d64LHLb+dA7cfAWQn50ymVb\\neUrxWccQ4dqIaYsdsdfsfYd63gL/HTmqCnb2Sotuv1bUNdw3mRRWLZlUKJbAT7nwnuCjt0G\\nXXhFQHxbAgMBAAECggEAWnmasUrTrNMX4Y5+na7ypzLanlfvUyqvdr08ic4gI5Tr\\nQK1atlS7XB5Gcam0QzKgwAM58KSo0z/odYHDySMcqgx6su4TyXwaOW/qkZD8PdXJ\\nbguyLsbXp0B37fcqlTMXMw8p2vY0tlVhAVItjaSUL3aeiQjc4Ig/BWt3JRIqcQKZ\\n7vlXOaTcFJxwkR2uZ7hfl5vhXK3sfsdfsdfHvcUYqb9Vdffv5tJEUzZ0YZbj2wsRNaacdcxu\\no3SU8LtllD4aTcf2HNWAubsDBZim8aTUC3+0LWrb5IAlgIZ01WyFZuGRhXrmiQSK\\nH44nwILn/D59+feVAZi9yCCcu5fjksacCssoQ83DGQKBgQDfTSk8PdcOOxG5kuaO\\nBSgAmIB/3D5Yp5sh+BDTjxd3Lu5mYAk0x4DhnxZczzvwhoopsieSjavDWIWtKtCrMnKcxxTa\\nkd+0Y6KHAIXyyalVsZHWLRi0RQYh44Is+nWozxSbcCXCMH8/n+F7oLxY5WTmSYT5\\nZR6PGlhefSsxEoYujRFvBQFyrwKBgQDcPZoiyo0DccUz+UDxATw+FB5iu3rzMXzO\\nqpcqG8zzBnDy6yf++9X5iMNS6jycSFyJTBlh1u1xVGWC4t2NlVd7KGgu4SuxA8Vf\\nHofeSuKVYIGfG+zzvByZRcn6kIqVOL/3ecRhALOxYw2VSJ3cBx+TdxU1R9/py+x1\\nO3hzdVQsFQKBgQCsfsdfsfsdfsdfsdfsdfaQNszWiStF/DumKPbh4RSpQZfTO1koKNxm8ND4Zz8H9dfsQer\\nBgXp1dPE2QMiN+tnTkdJkEIFgRvbYUrukcZO/bananasdfgdfgdsmTPjXT6eoQXRkrQAOcH4IaT8m2C\\nVKHisuoxVg8/TfEZKoDAvhBd+FjzG+ZXwZqSkEhrPVwQ0hMPJTPd1T9i0QKBgEOa\\nsTaewDxfbu4uQ632+BwCJvWdoPcHqMzzdmVZlUbAImmen29YtGzdez93YVWDrMwE\\nTQJIbChOhL5xjxqHzgui8p/5RGUYyDwTbhdhGz5JGmDRvKFwi8LMtlwwhCmb+ukn\\nOo2gHoiD5EZ/vN0uXpXwhtUNFAF7NEEkGSwvxr+lAoGBAMit8gNDAEedVddlUnmT\\ne7buloUfo7kIf9eu/MJXCVL1uwP38qXT6IBo90sHWABgh2VBHCP8XZDK0lC8kmeW\\nnuf5v8t+R3hPlqCAimqsnXJ76sb65BWaykYXsFZtFlc0zEH/LAjzC9I6s8N9wGU9\\nsod+rw/XzupeAjD1uO+rXtLm\\n-----END PRIVATE KEY-----\\n\",
  \"client_email\": \"vault-auth@allthingscloud-graham-221213.iam.gserviceaccount.com\",
  \"client_id\": \"101919379348055933547\",
  \"auth_uri\": \"https://accounts.google.com/o/oauth2/auth\",
  \"token_uri\": \"https://oauth2.googleapis.com/token\",
  \"auth_provider_x509_cert_url\": \"https://www.googleapis.com/oauth2/v1/certs\",
  \"client_x509_cert_url\": \"https://www.googleapis.com/robot/v1/metadata/x509/vault-auth%40allthingscloud-graham-221213.iam.gserviceaccount.com\"
}"
}

```  

``` bash
# configure gcp auth credentials
curl \
    --header "X-Vault-Token: ${VAULT_TOKEN}" \
    --request POST \
    --data @configure_gcp_auth_credentials.json \
    ${VAULT_ADDR}/v1/auth/gcp/config
```
Output
``` bash

```

 - Step 6 : Configure a Vault GCP Role

 ``` bash
# configure gcp auth role json file
tee configure_gcp_auth_role.json <<EOF
{
  "type": "iam",
  "project_id": "${GOOGLE_PROJECT}",
  "policies": ["default"],
  "ttl": "30m",
  "max_ttl": "24h",
  "max_jwt_exp": "5m",
  "bound_service_accounts": [
    "vault-auth@allthingscloud-graham-221213.iam.gserviceaccount.com"
  ]
}
EOF
```
Output
``` json
{
  "type": "iam",
  "project_id": "allthingscloud-graham-221213",
  "policies": ["default"],
  "ttl": "30m",
  "max_ttl": "24h",
  "max_jwt_exp": "5m",
  "bound_service_accounts": [
    "vault-auth@allthingscloud-graham-221213.iam.gserviceaccount.com"
  ]
}

```
``` bash
# configure gcp auth role
curl \
    --header "X-Vault-Token: ${VAULT_TOKEN}" \
    --request POST \
    --data @configure_gcp_auth_role.json \
    ${VAULT_ADDR}/v1/auth/gcp/role/vault-gcp-auth
```
Output
``` json
{"request_id":"5e65d65e-cb7b-b252-07f8-21464decb415","lease_id":"","renewable":false,"lease_duration":0,"data":null,"wrap_info":null,"warnings":null,"auth":null}
```

 - Step 7 : Generate GCP JWT request

``` bash
cat - > login_request.json <<EOF
{
    "aud": "vault/vault-gcp-auth",
    "sub": "${SERVICE_ACCOUNT}",
    "exp": $((EXP=$(date +%s)+600))
}
EOF

JWT_TOKEN=$(gcloud beta iam service-accounts sign-jwt login_request.json \
    signed_jwt.json \
    --iam-account=${SERVICE_ACCOUNT} && cat signed_jwt.json)
```
Output
``` bash
signed jwt [login_request.json] as [signed_jwt.json] for [vault-auth@allthingscloud-graham-221213.iam.gserviceaccount.com] using key [35d57c1b59dce42495e3a8ab08f37dc1d15f2792]
```
 - Step 8 : Request Vault Token

 ``` bash
# configure gcp auth role json file
tee gcp_vault_token_request.json <<EOF
{
  "role": "vault-gcp-auth",
  "jwt": "$JWT_TOKEN"
}
EOF
```
Output
``` json
{
  "role": "vault-gcp-auth",
  "jwt": "eyJhbGciOiJSUzI1NiIsImtpZCI6IjM1ZDU3YzFiNTlkY2U0MjQ5NWUzYThhYjA4ZjM3ZGMxZDE1ZjI3OTIiLCJ0eXAiOiJKV1QifQ.ewogICAgImF1ZCI6ICJ2YXVsdC92YXVsdC1nY3AtYXV0aCIsCiAxvdWQtZ3JhaGFtLTIyMTIxMy5pYW0uZ3NlcnZpY2VhY2NvdW50LmNvbSIsCiAgICAiZXhwIjogMTU0MjE0OTU2MAp9Cg.MmlNry9fcoRA_ZdfIuhWY82UE7XAbBy6NBP-yCr31BQJ3BvWqE9gscrjDpPiGnhGp2tzdEKK9Q998-aDswf8XOfN093Qpo9x-Mr_esPSkMvqZBYoggpX7ooaD064I5_pBGBSjhyabVJgvsrEAjXBXLt0zXu-VLNy7hXgTHISHASBEENMODIFIEDS_59W6JYRfB7sqLPUzKenbpMN87O4OQ0SBEba_uosmOxKD8Rl0u2B7kM7aD6EirwXhJ4_8DAwAIzs1Yo-gTTtnzz41flLU6YVRhrySH1Tt1GW8OCiyy9Y6tKeGYcW4AN8zRBDh07BODhagddbR8KhsGvi3t58w"
}
```
``` bash  
# configure gcp auth role
curl \
    --header "X-Vault-Token: ${VAULT_TOKEN}" \
    --request POST \
    --data @gcp_vault_token_request.json \
    ${VAULT_ADDR}/v1/auth/gcp/login
```
RESULT 
``` json
{"errors":["could not find service account key or Google Oauth cert with given 'kid' id 35d57c1b59dce42495e3a8ab08f37dc1d15f2792: could not find public key with kid '35d57c1b59dce42495e3a8ab08f37dc1d15f2792'"]}
```
