# Yaml  files

## Chart.yaml

Every time the chart version updates, we have to run `helm update`

```yaml
apiVersion: v1
appVersion: "1.0"
description: Titanic helm chart for K8s
name: titanic-app
version: 0.1.0
```

## values.yaml

We can many value as we want, the main purpose of value is to deploy different services or the same services on different environments e.g values-stg.yaml, values-prod.yaml

The basic `values.yaml` file contain:

```yaml
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

```yaml
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

## Template directory

### \_helpers.tpl

Content some variable to simplify the works.

```yaml
{/* vim: set filetype=mustache: */}}

{{/*
Create a default scoped fully qualified app name.
We truncate at 63 chars because some Kubernetes name fields are limited to this (by the DNS naming spec).
*/}}
{{- define "fullname" -}}
{{- .Release.Name | trunc 63 -}}
{{- end -}}

{{/* Standard labels which follow common convention
https://docs.helm.sh/chart_best_practices/#standard-labels
 */}}
{{- define "labels.standard" }}
heritage: {{ .Release.Service | quote }}
release: {{ .Release.Name | quote }}
chart: {{ .Chart.Name }}-{{ .Chart.Version | replace "+" "_" }}
app: {{ template "fullname" $ }}
{{- end }}
```

### deployment.yaml

For deploy the service to pod

```yaml
{{- $componentName := .Values.componentName }}

apiVersion: extensions/v1beta1
kind: Deployment
metadata:
  name: {{ template "fullname" . }}
  labels:
    {{ include "labels.standard" $ | indent 4 }}
    component: {{ $componentName }}
spec:
  replicas: {{ .Values.replicaCount }}
  selector:
    matchLabels:
      app: {{ template "fullname" . }}
      component: {{ $componentName }}
  template:
    metadata:
      labels:
        {{- include "labels.standard" $ | indent 8 }}
        component: {{ $componentName }}
    spec:
      containers:
        - name: {{ $componentName }}
          image: {{ .Values.image.repository }}:{{ .Values.image.tag }}
          imagePullPolicy: {{ .Values.image.pullPolicy }}
          ports:
            - name: http
              containerPort: {{ .Values.service.internalPort }}
          livenessProbe:
            httpGet:
              path: /heartbeat
              port: {{ .Values.service.internalPort }}
          readinessProbe:
            httpGet:
              path: /heartbeat
              port: {{ .Values.service.internalPort }}
          resources: {{- toYaml .Values.resources | nindent 12 }}
        {{- if .Values.vault.enable }}
        volumes:
          - name: eta-config
            secret:
              secretName: {{ template "fullname" . }}
              items:
                - key: {{ .Values.vault.configPaht }}
                  path: config.yml
        {{- end }}
```

### ingress.yaml

For nginx or Nodeport

```yaml
{{- $componentName := .Values.componentName }}
{{- if .Values.ingress.enabled -}}
apiVersion: extensions/v1beta1
kind: Ingress
metadata:
  annotations:
{{ toYaml .Values.ingress.annotations | indent 4 }}
  name: {{ template "fullname" $ }}-{{ $componentName }}
  labels:
    {{- include "labels.standard" $ | indent 4 }}
    component: {{ $componentName }}
spec:
  rules:
    - host: {{ .Values.ingress.host }}
      http:
        paths:
          - backend:
              serviceName: {{ template "fullname" $ }}
              servicePort: {{ .Values.service.internalPort }}
{{- end }}
```

### service.yaml

```text
{{- $componentName := .Values.componentName }}

apiVersion: v1
kind: Service
metadata:
  name: {{ template "fullname" . }}
  labels:
    {{- include "labels.standard" $ | indent 4 }}
spec:
  type: {{ .Values.service.type }}
  ports:
  - port: {{ .Values.service.internalPort }}
    protocol: TCP
    name: {{ .Values.service.name }}
  selector:
    app: {{ template "fullname" . }}
    component: {{ $componentName }}

```

### secrets.yaml

`secrets` is used for storing key/password in Kubernetes secrets in `key, value` pair format. 

It will be replaced with `secretClaim` if we use the `vautl`

```text
# This secret is used to set the initial credentials of the node container.
apiVersion: v1
kind: Secret
metadata:
  name: {{ .Values.initSecretName }}
  namespace: {{ .Values.namespace }}
  labels:
    app: {{ template "storageos.name" . }}
    chart: {{ template "storageos.chart" . }}
    heritage: {{ .Release.Service }}
    release: {{ .Release.Name }}
type: "kubernetes.io/storageos"
data:
  username: {{ default "" .Values.api.username | b64enc | quote }}
  password: {{ default "" .Values.api.password | b64enc | quote }}

```

### secretClaim.yaml

![Process of secretClaim](../../.gitbook/assets/image%20%281%29.png)

The `secretClaim` will be used by the `Vault Controller`. Based on the Path and Name, Vault controller will look inside the `Vault server`, get the secret and create K8s secrets. K8s secrets store the value which get from vault server.

Example code for the secretClaim

```text
{{- if .Values.vault.enable }}
kind: SecretClaim
apiVersion: vaultproject.io/v1
metadata:
  name: {{ template "fullname" . }}
  labels:
    {{- include "labels.standard" . | indent 4 }}
spec:
  type: Opaque
  path: "{{ .Values.vault.path }}"
  renew: 3900
{{- end }}
```

### configmap.yaml

In Kubernetes, a `ConfigMap` is simply a container for storing configuration data. Other things, like pods, can access the data in a ConfigMap.

Because ConfigMaps are basic resources, they make a great starting point for us.

```text
apiVersion: v1
kind: ConfigMap
metadata:
  name: {{ template "fullname" . }}
  labels:
    {{- include "labels.standard" . | indent 4 }}
data:
  {{- range $key, $value :=  .Values.config }}
  {{ $key | upper | replace "-" "_" }}: {{ $value | quote }}
  {{- end }}
```

