# PV, PVC

## PersistentVolume

Créer un PersistentVolume avec les caractéristiques suivantes :
* Nom : pv1
* Mode d'accès :ReadWriteOnce
* Type Local Storage dont le répertoire de persistance des données  est `/opt`
* Faite en sorte que ce dernier soit créé par affinité uniquement sur les Worker nodes `k8s-worker-1` et `k8s-worker-2` 

<details><summary>Correction</summary>

```yaml
apiVersion: v1
kind: PersistentVolume
metadata:
  name: pv1
spec:
  capacity:
    storage: 10Gi
  volumeMode: Filesystem
  accessModes:
  - ReadWriteOnce
  persistentVolumeReclaimPolicy: Delete
  storageClassName: 
  local:
    path: /opt
  nodeAffinity:
    required:
      nodeSelectorTerms:
      - matchExpressions:
        - key: kubernetes.io/hostname
          operator: In
          values:
          - k8s-worker-1
          - k8s-worker-2
```

```bash
vagrant@k8s-master-1:~$ kubectl apply -f pv1.yml
persistentvolume/pv1 created

vagrant@k8s-master-1:~$ kubectl get pv
NAME            CAPACITY   ACCESS MODES   RECLAIM POLICY   STATUS      CLAIM                                                STORAGECLASS   REASON   AGE
els-pv-volume   10Gi       RWO            Retain           Released    default/elasticsearch-data-quickstart-es-default-0                           27d
pv1             10Gi       RWO            Delete           Available                                                                                6s
```

</details>

## PersistentVolumeClaim

<details><summary>Correction</summary>

```yaml
apiVersion: v1
kind: PersistentVolume
metadata:
  name: pv1
spec:
  capacity:
    storage: 10Gi
  volumeMode: Filesystem
  accessModes:
  - ReadWriteOnce
  persistentVolumeReclaimPolicy: Delete
  storageClassName: 
  local:
    path: /opt
  nodeAffinity:
    required:
      nodeSelectorTerms:
      - matchExpressions:
        - key: kubernetes.io/hostname
          operator: In
          values:
          - k8s-worker-1
          - k8s-worker-2
---
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: pvc1
spec:
  accessModes:
    - ReadWriteOnce
  resources:
    requests:
      storage: 3Gi
---
apiVersion: v1
kind: Pod
metadata:
  name: pvlab
spec:
  containers:
    - name: pvlab-container
      image: nginx
      ports:
        - containerPort: 80
          name: "http-server"
      volumeMounts:
        - mountPath: "/usr/share/nginx/html"
          name: wew-storage
  volumes:
    - name: wew-storage
      persistentVolumeClaim:
        claimName: pvc1
```

```bash
vagrant@k8s-master-1:~$ kubectl apply -f pv1.yml
persistentvolume/pv1 created
vagrant@k8s-master-1:~$ kubectl get pv
NAME            CAPACITY   ACCESS MODES   RECLAIM POLICY   STATUS      CLAIM                                                STORAGECLASS   REASON   AGE
els-pv-volume   10Gi       RWO            Retain           Released    default/elasticsearch-data-quickstart-es-default-0                           27d
pv1             10Gi       RWO            Delete           Available                                                                                6s
vagrant@k8s-master-1:~$ vim pvc1.yml
vagrant@k8s-master-1:~$ kubectl apply -f pvc1.yml
persistentvolumeclaim/pvc1 created
vagrant@k8s-master-1:~$ kubectl get pv,pvc
NAME                             CAPACITY   ACCESS MODES   RECLAIM POLICY   STATUS     CLAIM                                                STORAGECLASS   REASON   AGE
persistentvolume/els-pv-volume   10Gi       RWO            Retain           Released   default/elasticsearch-data-quickstart-es-default-0                           27d
persistentvolume/pv1             10Gi       RWO            Delete           Bound      default/pvc1                                                                 117s

NAME                         STATUS   VOLUME   CAPACITY   ACCESS MODES   STORAGECLASS   AGE
persistentvolumeclaim/pvc1   Bound    pv1      10Gi       RWO                           7s
vagrant@k8s-master-1:~$
```


```bash
vagrant@k8s-master-1:~$ kubectl get pv
NAME            CAPACITY   ACCESS MODES   RECLAIM POLICY   STATUS     CLAIM                                                STORAGECLASS   REASON   AGE
els-pv-volume   10Gi       RWO            Retain           Released   default/elasticsearch-data-quickstart-es-default-0                           27d
pv1             10Gi       RWO            Delete           Bound      default/pvc1                                                                 2m35s
vagrant@k8s-master-1:~$ kubectl get pvc
NAME   STATUS   VOLUME   CAPACITY   ACCESS MODES   STORAGECLASS   AGE
pvc1   Bound    pv1      10Gi       RWO                           47s
vagrant@k8s-master-1:~$ vim pvlab-pod.ymal
vagrant@k8s-master-1:~$ kubectl apply -f pvlab-pod.ymal
pod/pvlab created
vagrant@k8s-master-1:~$ kubectl get pods
NAME                        READY   STATUS    RESTARTS   AGE
elastic1                    1/1     Running   0          4h39m
frontend-86d67d9884-5f6zr   1/1     Running   0          117m
frontend-86d67d9884-dftl2   1/1     Running   0          117m
nginx-pod                   1/1     Running   0          4h38m
pvlab                       1/1     Running   0          45s
robots-68548b4489-7hrrn     1/1     Running   0          52m
robots-68548b4489-tqvjw     1/1     Running   0          52m
web1                        1/1     Running   0          4h51m
webserver                   1/1     Running   0          135m
vagrant@k8s-master-1:~$ kubectl exec -it pvlab -- sh
# ls /usr/share/nginx/html
VBoxGuestAdditions-6.1.34  cni	containerd
# touch file-from-pod-pvlab
# touch /usr/share/nginx/html/file-from-pod-pvlab
#


## sur le node on voit bien que le ficher aparait
vagrant@k8s-worker-2:~$ ls /opt/
VBoxGuestAdditions-6.1.34  cni  containerd  file-from-pod-pvlab
vagrant@k8s-worker-2:~$
```

######### Storage Class

```yaml
---
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: pvc1
spec:
  storageClassName: premium-rwo
  accessModes:
    - ReadWriteOnce
  resources:
    requests:
      storage: 3Gi
---
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: pvc1
spec:
  accessModes:
    - ReadWriteOnce
  resources:
    requests:
      storage: 3Gi
---
apiVersion: v1
kind: Pod
metadata:
  name: pvlab
spec:
  containers:
    - name: pvlab-container
      image: nginx
      ports:
        - containerPort: 80
          name: "http-server"
      volumeMounts:
        - mountPath: "/usr/share/nginx/html"
          name: wew-storage
  volumes:
    - name: wew-storage
      persistentVolumeClaim:
        claimName: pvc1
```

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



