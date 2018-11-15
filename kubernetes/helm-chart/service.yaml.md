# service.yaml

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

