# secretClaim.yaml

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

