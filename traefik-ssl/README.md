### Setup

In order to activate SSL, you need to add the following 
custom config `/var/lib/rancher/k3s/server/manifests/traefik-config.yaml`

For the initial creation of the pod, make sure to uncomment the security context.
Then remove it and exec into the pod, to change permissions.

```
apiVersion: helm.cattle.io/v1
kind: HelmChartConfig
metadata:
  name: traefik
  namespace: kube-system
spec:
  valuesContent: |-
    additionalArguments:
      - "--serversTransport.insecureSkipVerify=true"
      - "--certificatesresolvers.default.acme.email=REPLACE_ME"
      - "--certificatesresolvers.default.acme.storage=/data/acme.json"
      - "--certificatesresolvers.default.acme.httpchallenge.entrypoint=web"
      # - "--certificatesresolvers.default.acme.caserver=https://acme-staging-v02.api.letsencrypt.org/directory"
      - "--certificatesresolvers.wildcard-resolver.acme.dnschallenge=true"
      - "--certificatesresolvers.wildcard-resolver.acme.dnschallenge.provider=cloudflare"
      - "--certificatesresolvers.wildcard-resolver.acme.email=REPLACE_ME"
      - "--certificatesresolvers.wildcard-resolver.acme.storage=/data/acme-wildcard.json"
      # - "--certificatesresolvers.wildcard-resolver.acme.caserver=https://acme-staging-v02.api.letsencrypt.org/directory"
    # podSecurityContext:
    #   fsGroup: 65532
    persistence:
      enabled: true
      name: traefik-pvc
    ports:
      web:
        exposedPort: 80
      websecure:
        exposedPort: 443
      local-websecure:
        expose:
            default: true
        port: 5443
        exposedPort: 5443
        tls:
            enabled: true
    env:
      - name: CF_DNS_API_TOKEN
        value: REPLACE_ME
```