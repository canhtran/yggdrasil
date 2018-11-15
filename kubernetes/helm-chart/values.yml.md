# values.yaml

We can many value as we want, the main purpose of value is to deploy different services.

The basic `values.yaml` file contain:

```text
environment: "staging"

vault:
  enable: false # toogle vault support
  path: config # key name in vault secret

relicaCount: 1

resources:
  limits:
    cpu: 500m
    memory: 512Mi
  requests:
    cpu: 100m
    memory: 512Mi
```

Optionals that can put in other

```text
componentName: etatravel

ingress:
  enabled: true
  annotations:
    kubernetes.io/ingress.class: "nginx-internal"
    external-dns.honestbee.io/enable: "true" # must be quoted
    nginx.ingress.kubernetes.io/force-ssl-redirect: "true"
  path: /
  host: eta-travel.internal.honestbee.com

image:
  repository: quay.io/honestbee/hb-data-travel-time
  tag: latest-eta-travel
  pullPolicy: IfNotPresent

service:
  name: eta-travel
  type: ClusterIP
  externalPort: 80
  internalPort: 5000
```



