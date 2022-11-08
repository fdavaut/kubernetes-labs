# Mise en place d'un déploiement Wordpress avec persistance de données

Cette partie du Lab, nous allons reconstituer les manifestes nécessaires pour déployer une application wordpress avec sa base de données.
Les notions que nous reverons :
- Secret
- ConfigMap
- PersistentVolumeClaim
- PersistentVolume
- StorageClass
- Service LoadBalancer

Lab inspiré des manifest suivants
* Backend => https://github.com/kubernetes/examples/blob/master/mysql-wordpress-pd/mysql-deployment.yaml
* Frontend=> https://github.com/kubernetes/examples/blob/master/mysql-wordpress-pd/wordpress-deployment.yaml 


Créez un secret générique du nom de `mysql-pass` qui devra contenir la clé/valeur suivante : `passwd => "Mettez un mot de passe quelconque"` (Ex: MonPassroot01)

<details><summary>Correction</summary>

```bash
kubectl create secret generic mysql-pass --from-literal=passwd=MonPassroot01 --dry-run=client -o yaml
```

le fichier YAML généré est le suivant. Remarquez que c'est par ce que le type est secret que la valeur a été encodé en Base64. 

```yaml
apiVersion: v1
data:
  passwd: TW9uUGFzc3Jvb3QwMQ==
kind: Secret
metadata:
  creationTimestamp: null
  name: mysql-pass
```

Si vous exécuté cette commande par exemple vous verrai le password en clair. Au temps vous dire que les type Secret ne sont pas véritablement un moyen de sécuriser son mot de passe.

```bash
echo "TW9uUGFzc3Jvb3QwMQ==" | base64 --decode
MonPassroot01
```
ou bien :

```bash
$ kubectl -n mynamespace get secrets mysql-pass \
    -o 'jsonpath={.data.passwd}' | base64 -d
MonPassroot01
```

```yaml
apiVersion: v1
data:
  password: cm9vdHJvb3QwMQ==
kind: Secret
metadata:
  creationTimestamp: null
  name: mysql-pass
---
apiVersion: v1
kind: Service
metadata:
  name: wordpress
  labels:
    app: wordpress
spec:
  ports:
    - port: 80
  selector:
    app: wordpress
    tier: frontend
  type: LoadBalancer
---
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: wp-pv-claim
  labels:
    app: wordpress
spec:
  accessModes:
    - ReadWriteOnce
  resources:
    requests:
      storage: 20Gi
---
apiVersion: apps/v1 #  for k8s versions before 1.9.0 use apps/v1beta2  and before 1.8.0 use extensions/v1beta1
kind: Deployment
metadata:
  name: wordpress
  labels:
    app: wordpress
spec:
  selector:
    matchLabels:
      app: wordpress
      tier: frontend
  strategy:
    type: RollingUpdate
  template:
    metadata:
      labels:
        app: wordpress
        tier: frontend
    spec:
      containers:
      - image: wordpress:4.8-apache
        name: wordpress
        env:
        - name: WORDPRESS_DB_HOST
          value: wordpress-mysql
        - name: WORDPRESS_DB_PASSWORD
          valueFrom:
            secretKeyRef:
              name: mysql-pass
              key: password
        ports:
        - containerPort: 80
          name: wordpress
        volumeMounts:
        - name: wordpress-persistent-storage
          mountPath: /var/www/html
      volumes:
      - name: wordpress-persistent-storage
        persistentVolumeClaim:
          claimName: wp-pv-claim
```

```yaml
apiVersion: v1
kind: Service
metadata:
  name: wordpress-mysql
  labels:
    app: wordpress
spec:
  ports:
    - port: 3306
  selector:
    app: wordpress
    tier: mysql
  clusterIP: None
---
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: mysql-pv-claim
  labels:
    app: wordpress
spec:
  accessModes:
    - ReadWriteOnce
  resources:
    requests:
      storage: 20Gi
---
apiVersion: apps/v1 # for k8s versions before 1.9.0 use apps/v1beta2  and before 1.8.0 use extensions/v1beta1
kind: Deployment
metadata:
  name: wordpress-mysql
  labels:
    app: wordpress
spec:
  selector:
    matchLabels:
      app: wordpress
      tier: mysql
  strategy:
    type: Recreate
  template:
    metadata:
      labels:
        app: wordpress
        tier: mysql
    spec:
      containers:
      - image: mysql:5.6
        name: mysql
        env:
        - name: MYSQL_ROOT_PASSWORD
          valueFrom:
            secretKeyRef:
              name: mysql-pass
              key: password
        livenessProbe:
          tcpSocket:
            port: 3306
        ports:
        - containerPort: 3306
          name: mysql
        volumeMounts:
        - name: mysql-persistent-storage
          mountPath: /var/lib/mysql
      volumes:
      - name: mysql-persistent-storage
        persistentVolumeClaim:
          claimName: mysql-pv-claim
```


```bash
$ kubectl apply -f front-wordpress.yml                                                                                ✔ ╱ base  ╱ at mawaki-k8s-lab ⎈ ╱ at 16:28:37 
secret/mysql-pass created
service/wordpress created
persistentvolumeclaim/wp-pv-claim created
deployment.apps/wordpress created

$ kubectl apply -f back-wordpress.yml                                                                     ✔ ╱ took 3s  ╱ base  ╱ at mawaki-k8s-lab ⎈ ╱ at 16:26:37 
service/wordpress-mysql created
persistentvolumeclaim/mysql-pv-claim created
deployment.apps/wordpress-mysql created

$ kubectl get deployment,pods,svc,secret,pvc,pv,sc                                                                    ✔ ╱ base  ╱ at mawaki-k8s-lab ⎈ ╱ at 16:28:50 
NAME                              READY   UP-TO-DATE   AVAILABLE   AGE
deployment.apps/wordpress         1/1     1            1           35s
deployment.apps/wordpress-mysql   1/1     1            1           2m37s

NAME                                   READY   STATUS    RESTARTS   AGE
pod/pvlab                              1/1     Running   0          57m
pod/wordpress-5994d99f46-fddjd         1/1     Running   0          35s
pod/wordpress-mysql-7fc5cb7ccc-fkckl   1/1     Running   0          2m37s

NAME                      TYPE           CLUSTER-IP    EXTERNAL-IP   PORT(S)        AGE
service/kubernetes        ClusterIP      10.4.16.1     <none>        443/TCP        12h
service/wordpress         LoadBalancer   10.4.30.247   34.71.81.82   80:32020/TCP   35s
service/wordpress-mysql   ClusterIP      None          <none>        3306/TCP       2m37s

NAME                         TYPE                                  DATA   AGE
secret/default-token-r4p9c   kubernetes.io/service-account-token   3      12h
secret/mysql-pass            Opaque                                1      36s

NAME                                   STATUS   VOLUME                                     CAPACITY   ACCESS MODES   STORAGECLASS   AGE
persistentvolumeclaim/mysql-pv-claim   Bound    pvc-4d312cf3-609b-45c4-8e24-8de879dc2da3   20Gi       RWO            standard       2m37s
persistentvolumeclaim/pvc1             Bound    pvc-5bd5facd-2532-4789-bc25-f37337b004ec   3Gi        RWO            premium-rwo    58m
persistentvolumeclaim/wp-pv-claim      Bound    pvc-26903e6a-48aa-438f-8f86-8de264c38e67   20Gi       RWO            standard       35s

NAME                                                        CAPACITY   ACCESS MODES   RECLAIM POLICY   STATUS   CLAIM                    STORAGECLASS   REASON   AGE
persistentvolume/pvc-26903e6a-48aa-438f-8f86-8de264c38e67   20Gi       RWO            Delete           Bound    default/wp-pv-claim      standard                31s
persistentvolume/pvc-4d312cf3-609b-45c4-8e24-8de879dc2da3   20Gi       RWO            Delete           Bound    default/mysql-pv-claim   standard                2m33s
persistentvolume/pvc-5bd5facd-2532-4789-bc25-f37337b004ec   3Gi        RWO            Delete           Bound    default/pvc1             premium-rwo             57m

NAME                                             PROVISIONER             RECLAIMPOLICY   VOLUMEBINDINGMODE      ALLOWVOLUMEEXPANSION   AGE
storageclass.storage.k8s.io/premium-rwo          pd.csi.storage.gke.io   Delete          WaitForFirstConsumer   true                   12h
storageclass.storage.k8s.io/standard (default)   kubernetes.io/gce-pd    Delete          Immediate              true                   12h
storageclass.storage.k8s.io/standard-rwo         pd.csi.storage.gke.io   Delete          WaitForFirstConsumer   true                   12h
```

On accède avec l'IP public du loadBalancer 34.71.81.82 => Ok succès de connexion

</details>

