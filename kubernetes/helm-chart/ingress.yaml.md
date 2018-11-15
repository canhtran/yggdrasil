# ingress.yaml

```text
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

