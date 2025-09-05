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

HOST=localhost
PORT=3080

DOMAIN_CLIENT=http://localhost:3080
DOMAIN_SERVER=http://localhost:3080

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

### Run the server
```bash
docker compose up -d
```
