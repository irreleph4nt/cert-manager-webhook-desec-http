[![Build](https://github.com/irreleph4nt/cert-manager-webhook-desec-http/actions/workflows/publish.yml/badge.svg)](https://github.com/irreleph4nt/cert-manager-webhook-desec-http/actions/workflows/publish.yml)

# ACME webhook for deSEC DNS API (http client version)
Usage:
```bash
helm install desec-http oci://ghcr.io/irreleph4nt/charts/cert-manager-webhook-desec-http -f values.yaml -n cert-manager
```

Testing:
```bash
TEST_DOMAIN_NAME=<domain name> TEST_SECRET=$(echo -n '<DESEC API TOKEN>' | base64) make test
```

# Version History
| desec-http    | built with          | notable features         |
| ------------- | ------------------- | ------------------------ |
| v1.0.1        | cert-manager v1.13  | deSEC API Rate limiting<br>log.SetLogger(...) fix  |
| v1.0.0        | cert-manager v1.11  | initial release          |

# Example Issuer
```yaml
apiVersion: cert-manager.io/v1
kind: ClusterIssuer
metadata:
  name: desec-http
  namespace: cert-manager
spec:
  acme:
    email: <YOUR ACME E-MAIL ADDRESS>
    server: https://acme-v02.api.letsencrypt.org/directory
    privateKeySecretRef:
      name: acme-account
    solvers:
    - dns01:
        webhook:
          groupName: <YOUR GROUP NAME>
          solverName: desec-http
          config:
            apiUrl: https://desec.io/api/v1
            domainName: <YOUR DNS ZONE>
            secretName: desec-access-token
            secretKeyName: desec-token
```

# Example Secret
```yaml
apiVersion: v1
kind: Secret
type: Opaque
metadata:
  name: desec-access-token
  namespace: cert-manager
stringData:
  desec-token: <YOUR DESEC TOKEN>
```

# Example values.yaml
```yaml
groupName: <YOUR DNS ZONE>

certManager:
  serviceAccountName: cert-manager
  namespace: cert-manager

image:
  repository: ghcr.io/irreleph4nt/cert-manager-webhook-desec-http
  tag: ""
  pullPolicy: IfNotPresent

replicaCount: 1

nameOverride: ""
fullnameOverride: ""

service:
  type: ClusterIP
  port: 443

secretName:
- desec-access-token

resources:
  limits:
     cpu: 250m
     memory: 256Mi
  requests:
     cpu: 250m
     memory: 256Mi

podSecurityContext:
  enabled: true
  fsGroup: 1001

containerSecurityContext:
  enabled: true
  runAsUser: 1001
  readOnlyRootFilesystem: true
  runAsNonRoot: true
```

# Credits
This webhook was inspired by [dmahmalat/cert-manager-webhook-google-domains](https://github.com/dmahmalat/cert-manager-webhook-google-domains), which solves DNS01 challenges
by interacting with Google's public ACME API over HTTP requests. In that way, desec-http is more similar to it than to [kmorning/cert-manager-webhook-desec](https://github.com/kmorning/cert-manager-webhook-desec), which re-implements parts of the deSEC API in GO to achieve the same result.
