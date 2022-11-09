# Opération courrantes

## Dashboard Kubernetes

Installez le Dashboard Kubernetes sur un de vos clusters

Procédure d'installation : [Doc officiel](https://kubernetes.io/docs/tasks/access-application-cluster/web-ui-dashboard/)

```bash
kubectl apply -f https://raw.githubusercontent.com/kubernetes/dashboard/v2.6.1/aio/deploy/recommended.yaml
```

```bash
kubectl proxy
```

