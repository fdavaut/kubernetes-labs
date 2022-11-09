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

Si cela ne marche pas pour une raison ou pour une autre on va changer le type de service en `NodePort`

```bash
kubectl patch service kubernetes-dashboard  -n kubernetes-dashboard  -p '{"spec": {"type": "NodePort" }}'
```

```bash
kubectl get services -A                                                                       

NAMESPACE              NAME                        TYPE        CLUSTER-IP      EXTERNAL-IP   PORT(S)                  AGE
default                kubernetes                  ClusterIP   10.96.0.1       <none>        443/TCP                  108m
kube-system            kube-dns                    ClusterIP   10.96.0.10      <none>        53/UDP,53/TCP,9153/TCP   108m
kubernetes-dashboard   dashboard-metrics-scraper   ClusterIP   10.105.51.184   <none>        8000/TCP                 50m
kubernetes-dashboard   kubernetes-dashboard        NodePort    10.98.27.147    <none>        443:31145/TCP            50m
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

Une fois que la maintenance est terminée on marque maintenant le Node comme schedulable afin qu'il commence à recevoir les Pods

```bash
kubectl uncordon k8s-worker-1
```


```bash

```


```bash

```


```bash

```