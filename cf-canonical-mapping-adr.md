# Cloud Foundry to Canonical Form Field Mapping

## Overview

This section defines how Cloud Foundry (CF) manifest fields map to our container-runtime agnostic canonical form. The canonical form is designed to be generic enough to support any container orchestration platform while preserving all essential application deployment information.

## Field Mapping Reference

| Cloud Foundry Field | CF Example Value | Canonical Form Path | Canonical Example | Notes |
|-------------------|------------------|-------------------|------------------|--------|
| `name` | "my-app" | `app.name` | `name: "my-app"` | Direct mapping |
| `memory` | "1G" | `app.resources.memory` | `memory: {value: 1024, unit: "Mi"}` | Converted to standardized units |
| `disk_quota` | "512M" | `app.resources.storage` | `storage: {value: 512, unit: "Mi"}` | Converted to standardized units |
| `instances` | 3 | `app.scaling.replicas` | `replicas: 3` | Direct mapping |
| `env` | `{API_KEY: "123"}` | `app.environment.variables` | `variables: {API_KEY: "123"}` | Direct mapping |
| `services` | `["mysql-db"]` | `app.services.bindings` | `bindings: [{name: "mysql-db"}]` | Simplified to service name only |
| `routes` | `["app.domain.com/path"]` | `app.networking.endpoints` | ```endpoints:``` <br> ```- host: "app"``` <br> ```  domain: "domain.com"``` <br> ```  path: "/path"``` | Split into components |
| `command` | "node app.js" | `app.process.command` | `command: "node app.js"` | Direct mapping |
| `health-check-type` | "http" | `app.healthcheck.type` | `type: "http"` | Simplified health check types |
| `health-check-http-endpoint` | "/health" | `app.healthcheck.path` | `path: "/health"` | Only for HTTP checks |
| `timeout` | 60 | `app.healthcheck.timeout` | `timeout: 60` | In seconds |

## Canonical Form Schema

```yaml
app:
  name: string
  resources:
    memory:
      value: integer
      unit: string  # Mi or Gi
    storage:
      value: integer
      unit: string  # Mi or Gi
  scaling:
    replicas: integer
    autoscaling:  # Optional
      min: integer
      max: integer
      metrics:
        - type: string  # cpu, memory
          target: integer
  environment:
    variables:
      KEY: value
  services:
    bindings:
      - name: string
  networking:
    endpoints:
      - host: string
        domain: string
        path: string
        port: integer
        tls: boolean
  process:
    command: string
    args: [string]
  healthcheck:
    type: string  # http, tcp, exec
    path: string  # for http
    port: integer # for tcp
    timeout: integer
```

## Unmapped Cloud Foundry Fields

The following CF fields are not mapped to the canonical form as they are platform-specific or replaced by more generic concepts:

| CF Field | Reason for Exclusion |
|----------|---------------------|
| `buildpack` | Platform-specific build mechanism |
| `buildpacks` | Platform-specific build mechanism |
| `stack` | Platform-specific runtime environment |
| `docker` | Platform-specific container configuration |
| `no-route` | CF-specific routing concept |
| `random-route` | CF-specific routing feature |
| `path` | Build context handled separately |
| `metadata.annotations` | Platform-specific metadata |
| `metadata.labels` | Platform-specific metadata |

## Special Mapping Cases

1. **Memory and Storage Units**
   - CF format: Human readable (e.g., "1G", "512M")
   - Canonical: Standardized to Mi/Gi with numerical values
   ```yaml
   # CF
   memory: "1.5G"
   # Canonical
   memory: {value: 1536, unit: "Mi"}
   ```

2. **Route Parsing**
   - CF format: Single string with combined host/domain/path
   - Canonical: Split into components for flexibility
   ```yaml
   # CF
   routes: ["myapp.example.com/api"]
   # Canonical
   endpoints:
     - host: "myapp"
       domain: "example.com"
       path: "/api"
   ```

3. **Health Checks**
   - CF format: Separate fields for type and endpoint
   - Canonical: Combined under single healthcheck object
   ```yaml
   # CF
   health-check-type: "http"
   health-check-http-endpoint: "/health"
   # Canonical
   healthcheck:
     type: "http"
     path: "/health"
     timeout: 60
   ```
