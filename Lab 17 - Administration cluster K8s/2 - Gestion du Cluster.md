# Opération courrantes

## Dashboard Kubernetes

Installez le Dashboard Kubernetes sur un de vos clusters

Procédure d'installation : [Doc officiel](https://kubernetes.io/docs/tasks/access-application-cluster/web-ui-dashboard/)

```bash
kubectl apply -f https://raw.githubusercontent.com/kubernetes/dashboard/v2.6.1/aio/deploy/recommended.yaml
```

```bash
kubectl get all -n kubernetes-dashboard
```

Faites un forwarding de ports pour y accéder

```bash
kubectl port-forward -n kubernetes-dashboard service/kubernetes-dashboard 8443:443
```
## Maintenance des Nodes

Afficher les nodes de votre cluster

```bash
kubectl get nodes
```

Il se trouve que vous souhaiter mettre à jour le node `k8s-worker-1` pour cela vous souhaitez évencer les Pods de ce Noeud, le rendre non schedulable.
Que feriez vous ?


<details><summary>Correction</summary>
_"Vous pouvez utiliser kubectl drain pour expulser en toute sécurité tous vos pods d'un nœud avant d'effectuer une maintenance sur le nœud (par exemple, une mise à niveau du noyau, une maintenance matérielle, etc.) Les expulsions sécurisées permettent aux conteneurs du pod de se terminer de manière élégante. "_ [Kubernetes](https://kubernetes.io/docs/tasks/administer-cluster/safely-drain-node/#use-kubectl-drain-to-remove-a-node-from-service)

```bash
kubectl drain k8s-worker-1
```

</details>

```bash

```


```bash

```


```bash

```


```bash

```