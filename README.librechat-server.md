# Librechat server with vLLM 

## Overview
This is a setup for a Librechat server with using [vLLM server](https://github.com/SecureDevice-DevOps/vllm-server).

## Prerequisites
- Docker
- Docker Compose
- OpenSSL

## Setup

### Create certificates 
Make certs directory.
```bash
# Create directories for certs
mkdir -p certs
```

Generate the certificate and key.
```bash
# Generate the certificate and key
openssl req -x509 -nodes -days 365 -newkey rsa:2048 -keyout certs/privkey.pem -out certs/fullchain.pem -config certs/req.cnf
# Generate the dhparam file
openssl dhparam -out certs/dhparam.pem 2048
# Copy the fullchain.pem to ca.crt
cp certs/fullchain.pem certs/ca.crt
```

### Create a .env file
Create a .env file in the root of the project.
```bash
# Create a .env file
cp .env.example .env
```
Update the following keys in the .env file:
```bash
# vLLM Configuration
VLLM_API_KEY=api-key
VLLM_BASE_URL=https://vllm.securedevice.local/v1
VLLM_HOST_DNS_MAPPING=vllm.securedevice.local:192.168.44.3 # For some reason docker cannot resolve the dns

RAG_API_URL=https://vllm.securedevice.local/v1
RAG_OPENAI_API_KEY=api-key

HOST=chad.securedevice.local
PORT=3080

DOMAIN_CLIENT=https://chad.securedevice.local
DOMAIN_SERVER=https://chad.securedevice.local

```



### Create a librechat.yaml file
Create a librechat.yaml file in the root of the project.
```bash
# Create a librechat.yaml file
cp librechat.yaml.example librechat.yaml
```

Now add vLLM settings to the librechat.yaml file.
```yaml
endpoints:
  custom:
      - name: "vLLM"
      apiKey: "api-key"
      baseURL: "https://vllm.securedevice.local/v1"  # Make sure this is HTTPS
      models:
        default: ['Qwen/Qwen3-0.6B']
        fetch: true
      titleConvo: true
      titleModel: "current_model"
      modelDisplayLabel: "vLLM"
      titleMessageRole: "user"
      summarize: false
      summaryModel: "current_model"
      forcePrompt: false
```


Also add domains to the .env file 


### Generate new CREDS_KEY, CREDS_IV, JWT_SECRET, and JWT_REFRESH_SECRET
```bash
# Generate new CREDS_KEY, CREDS_IV, JWT_SECRET, and JWT_REFRESH_SECRET
    openssl rand -hex 32 # CREDS_KEY
    openssl rand -hex 16 # CREDS_IV
    openssl rand -hex 32 # JWT_SECRET
    openssl rand -hex 32 # JWT_REFRESH_SECRET
```

### Enable Azure OpenID login
ALLOW_SOCIAL_LOGIN=true
 
OPENID_CLIENT_ID=Your Application (client) ID
OPENID_CLIENT_SECRET=Your client secret
OPENID_ISSUER=https://login.microsoftonline.com/Your Directory (tenant ID)/v2.0/
OPENID_SESSION_SECRET=Any random string
OPENID_SCOPE=openid profile email #DO NOT CHANGE THIS
OPENID_CALLBACK_URL=/oauth/openid/callback # this should be the same for everyone
 
OPENID_REQUIRED_ROLE_TOKEN_KIND=id
 
# If you want to restrict access by groups
OPENID_REQUIRED_ROLE_PARAMETER_PATH="roles"
OPENID_REQUIRED_ROLE="Your Group Name"
 
# Optional: redirects the user to the end session endpoint after logging out
OPENID_USE_END_SESSION_ENDPOINT=true 

### Run the server
```bash
docker compose -f deploy-compose.yml up -d
```


