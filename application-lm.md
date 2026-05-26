# 🌐 GCP Load Balancer Self-Service Automation

Welcome to the Self-Service Load Balancer provisioning tool! This pipeline allows you to deploy enterprise-grade Google Cloud Load Balancers in minutes using GitHub Actions and Terraform.

To deploy a Load Balancer, you will fill out a few high-level parameters in the GitHub Actions UI and paste a YAML configuration block to define your exact backend and routing logic. :contentReference[oaicite:0]{index=0}

---

# 🏗️ How It Works: UI vs. YAML

To bypass GitHub Actions' input limits and provide maximum flexibility, the configuration is split into two parts:

1. **GitHub UI Inputs (The Architecture)**  
   You select:
   - Name
   - Exposure (External/Internal)
   - Scope (Global/Regional)
   - Region

2. **The YAML Payload (The Brains)**  
   You paste a YAML block defining:
   - Health checks
   - SSL certificates
   - CDN toggles
   - Routing rules

> Do not put Exposure, Scope, or Region in the YAML.

---

# 📝 The YAML Schema Reference

Your YAML file must contain three root-level blocks:

- `config`
- `backends`
- `routing`

---


# 1. `config` Block (Frontend & Network Configuration)

This block defines how the Load Balancer receives traffic from the internet or your internal network.

| Field | Type | Required? | Description |
|---|---|---|---|
| `frontend_ip_type` | String | Optional | `ephemeral` (default) or `static`. |
| `frontend_ip_name` | String | If static | The exact GCP resource name of your pre-reserved static IP address. |
| `enable_https` | Boolean | Optional | Set to `true` to deploy an HTTPS proxy. Defaults to `false` (HTTP). |
| `ssl_certificate` | String | If HTTPS is true | The full GCP self-link to your SSL cert. Example: `projects/.../global/sslCertificates/my-cert` |
| `enable_http_redirect` | Boolean | Optional | If `true` and HTTPS is enabled, forces all HTTP traffic to redirect to HTTPS. Defaults to `false`. |
| `network` | String | If Internal | The VPC network name. Mandatory for Internal Load Balancers. |
| `subnetwork` | String | If Internal | The subnet name. Mandatory for Internal Load Balancers. |

---

# 2. `backends` Block (Compute & Storage Attachments)

This block defines your backend services (the actual servers/buckets).

You can define multiple backends by creating new keys like:
- `api-backend`
- `web-backend`

| Field | Type | Required? | Description |
|---|---|---|---|
| `[backend-name]` | Dictionary | Yes | The custom name for your backend. Example: `api-backend` |
| `backend_type` | String | Yes | `MIG` (Managed Instance Group), `NEG` (Network Endpoint Group), or `BUCKET` |
| `enable_cdn` | Boolean | Optional | Set to `true` to enable Cloud CDN caching. Defaults to `false`. |
| `enable_logging` | Boolean | Optional | Set to `true` to enable Cloud Logging for this specific backend's traffic. |
| `health_check` | Dictionary | Optional | Define if this backend requires a health check. Not used for `BUCKET` types. |
| `health_check.path` | String | If health_check | The HTTP path the prober will hit. Example: `/healthz` |
| `health_check.port` | Integer | If health_check | The port to check. Example: `80` or `8080` |
---

# 3. `routing` Block

This block defines your URL Map and how traffic routes to backends.

| Field | Type | Required? | Description |
|---|---|---|---|
| `host_rules` | List | Yes | List mapping domains (hosts) to backends. |
| `host_rules.host` | String | Yes | Domain name. Example: `api.example.com` or `*` |
| `host_rules.backend` | String | Yes | Must exactly match a backend name from the `backends` block. |
| `path_rules` | List | Optional | Advanced routing based on URL path. |
| `path_rules.path` | String | Yes | URL path. Example: `/images/*` |
| `path_rules.backend` | String | Yes | Backend to route this path to. |

---

# 🚦 Architectural Guardrails & Edge Cases

Google Cloud has strict rules about how Load Balancers can be configured. Our automation enforces the following logic:

## Internal Load Balancers
- Cannot be Global.
- Must be:
  - Regional
  - Cross-Regional
- `network` and `subnetwork` are mandatory in YAML.

## External Load Balancers
- Generally should be Global.
- If Regional External LB is selected, a region must be provided.

## Cross-Regional Internal
This is a hybrid architecture:
- Uses global backends
- Uses a global URL map
- Requires a regional frontend forwarding rule

UI Selection:
- Internal
- Cross-Regional
- Region

---

# 📖 Example Configurations

Copy and paste these templates to get started.

---

# Example 1: Standard Global External HTTPS with CDN

### UI Choices
- External
- Global

```yaml
config:
  enable_https: true
  ssl_certificate: "projects/my-project/global/sslCertificates/prod-wildcard-cert"

backends:
  frontend-web:
    enable_cdn: true
    health_check:
      path: /
      port: 80

  api-service:
    enable_cdn: false
    health_check:
      path: /api/health
      port: 8080

routing:
  host_rules:
    - host: www.example.com
      backend: frontend-web

    - host: api.example.com
      backend: api-service

  path_rules:
    - path: /static/*
      backend: frontend-web
```

---

# Example 2: Internal Regional API Load Balancer

### UI Choices
- Internal
- Regional
- `us-central1`

```yaml
config:
  enable_https: false
  network: "projects/my-project/global/networks/my-vpc"
  subnetwork: "projects/my-project/regions/us-central1/subnetworks/my-proxy-subnet"

backends:
  internal-api:
    enable_cdn: false
    health_check:
      path: /healthz
      port: 80

routing:
  host_rules:
    - host: "*"
      backend: internal-api
```

---

# Example 3: Simple Global External HTTP (No SSL, No CDN)

### UI Choices
- External
- Global

```yaml
config:
  enable_https: false

backends:
  default-backend:
    enable_cdn: false
    health_check:
      path: /
      port: 80

routing:
  host_rules:
    - host: "*"
      backend: default-backend
```

---

# ⚠️ Troubleshooting

## Pipeline fails immediately
You likely selected an invalid combination in the UI.

Example:
- Internal + Global

Check GitHub Actions logs for the exact validation error.

---

## Terraform fails on Apply (Health Check)
Ensure:
- The health check port is exposed by your application.
- The application responds successfully on the configured path.

---

## Terraform fails on Apply (Subnet)
If deploying an Internal Application Load Balancer:
- A Proxy-Only Subnet must already exist in the target region before running the pipeline.
