# secrets.yaml

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

