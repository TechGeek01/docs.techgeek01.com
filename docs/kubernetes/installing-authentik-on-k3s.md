# Installing Authentik on k3s

!!! info
	This guide assumes a working installation of k3s, already having been setup with proper certs and such, using Traefik as the reverse proxy.

# Helm chart `authentik.yaml`
```yaml
authentik:
    secret_key: "redacted"
    # This sends anonymous usage-data, stack traces on errors and
    # performance data to sentry.io, and is fully opt-in
    error_reporting:
        enabled: true
    postgresql:
        password: "redacted"

server:
    ingress:
        # Specify kubernetes ingress controller class name
        ingressClassName: traefik
        enabled: true
        hosts:
            - auth.example.com

postgresql:
    enabled: true
    auth:
        password: "redated"
redis:
    enabled: true
```

To apply, run
```bash
lubectl apply -f authentik.yaml
```
