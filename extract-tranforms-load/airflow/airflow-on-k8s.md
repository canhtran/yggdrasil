---
description: >-
  Setup airflow on Kubernetes with Kubernetes Executor and
  KubernetesPodExecutors
---

# Airflow on K8s

## Issues

### \#1 PostgreSQl increase max connection

Helm airflow does not use the stable/postgresql chart. Check [https://github.com/helm/charts/blob/master/stable/airflow/requirements.yaml](https://github.com/helm/charts/blob/master/stable/airflow/requirements.yaml), and you can find the right postgresql chart.

```text
postgresql:
    postgresConfig:
      maxConnections: "200"
```

