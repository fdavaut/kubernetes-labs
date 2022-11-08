# Storage Class

Sur un Cluster GKE lister les SC disponibles

<details><summary>Correction</summary>

```yaml

```bash
$ kubectl get sc premium-rwo -o yaml
```
Résultat de la commande :

```yaml
allowVolumeExpansion: true
apiVersion: storage.k8s.io/v1
kind: StorageClass
metadata:
  annotations:
    components.gke.io/component-name: pdcsi
    components.gke.io/component-version: 0.11.8
    components.gke.io/layer: addon
  creationTimestamp: "2022-10-05T01:31:56Z"
  labels:
    addonmanager.kubernetes.io/mode: EnsureExists
    k8s-app: gcp-compute-persistent-disk-csi-driver
  name: premium-rwo
  resourceVersion: "608"
  uid: ddf2b6d2-ac71-4764-9658-509badd8a59b
parameters:
  type: pd-ssd
provisioner: pd.csi.storage.gke.io
reclaimPolicy: Delete
volumeBindingMode: WaitForFirstConsumer
```

Observons le fait que création de PVC et d'un Pod va engendrer la création du PV

```bash
$ kubectl apply -f pvc1.yml
# Nous avons créé le PVC

$ kubectl get sc,pv,pvc 
NAME                                             PROVISIONER             RECLAIMPOLICY   VOLUMEBINDINGMODE      ALLOWVOLUMEEXPANSION   AGE
storageclass.storage.k8s.io/premium-rwo          pd.csi.storage.gke.io   Delete          WaitForFirstConsumer   true                   11h
storageclass.storage.k8s.io/standard (default)   kubernetes.io/gce-pd    Delete          Immediate              true                   11h
storageclass.storage.k8s.io/standard-rwo         pd.csi.storage.gke.io   Delete          WaitForFirstConsumer   true                   11h

NAME                         STATUS    VOLUME   CAPACITY   ACCESS MODES   STORAGECLASS   AGE
persistentvolumeclaim/pvc1   Pending                                      premium-rwo    71s

# On voit que le PVC est en mode pending parce qu'il attend d'être consommé par un Pod (VOLUMEBINDINGMODE = WaitForFirstConsumer)

$ kubectl apply -f pod.yml

# On remarquera que quand le Pod devient Up il y'a des changements : création de PV et Status Bound, PVC passe de Status Pending à Bound 

$ kubectl get pods,sc,pv,pvc 
NAME        READY   STATUS    RESTARTS   AGE
pod/pvlab   1/1     Running   0          25s

NAME                                             PROVISIONER             RECLAIMPOLICY   VOLUMEBINDINGMODE      ALLOWVOLUMEEXPANSION   AGE
storageclass.storage.k8s.io/premium-rwo          pd.csi.storage.gke.io   Delete          WaitForFirstConsumer   true                   12h
storageclass.storage.k8s.io/standard (default)   kubernetes.io/gce-pd    Delete          Immediate              true                   11h
storageclass.storage.k8s.io/standard-rwo         pd.csi.storage.gke.io   Delete          WaitForFirstConsumer   true                   12h

NAME                                                        CAPACITY   ACCESS MODES   RECLAIM POLICY   STATUS   CLAIM          STORAGECLASS   REASON   AGE
persistentvolume/pvc-5bd5facd-2532-4789-bc25-f37337b004ec   3Gi        RWO            Delete           Bound    default/pvc1   premium-rwo             20s

NAME                         STATUS   VOLUME                                     CAPACITY   ACCESS MODES   STORAGECLASS   AGE
persistentvolumeclaim/pvc1   Bound    pvc-5bd5facd-2532-4789-bc25-f37337b004ec   3Gi        RWO            premium-rwo    92s

```

</details>