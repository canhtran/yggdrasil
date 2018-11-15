# deployment.yaml

```text
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

