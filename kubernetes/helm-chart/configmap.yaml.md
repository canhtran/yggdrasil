---
description: 'https://docs.helm.sh/chart_template_guide/'
---

# configmap.yaml

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

