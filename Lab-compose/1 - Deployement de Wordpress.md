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


## Déploiement de la Base de données

### Secrets d'accès à la DB MySQL

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

Si vous exécuté cette commande par exemple vous verrez le password en clair. Au temps vous dire que les type Secret ne sont pas véritablement un moyen de sécuriser son mot de passe.

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
</details>

### PVC MySQL

Gestion de la persistance des données.
Créez Un PVC pour MySQL qui utilisera par défaut le StorageClass de votre cluster
* nom : mysql-pv-claim
* accessModes : ReadWriteOnce
* storage: 20Gi

<details><summary>Correction</summary>

```yaml
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
```

</details>

### Déploiement MySQL

Créer un déploiement pour la Base de données MySQL avec les caractéristiques suivantes
* Nom: `wordpress-mysql`
* image : `mysql:5.6`
* Importer une variable d'environnement `MYSQL_ROOT_PASSWORD` dont la valeur est celle du secret : `mysql-pass`, clé : `password`
* le port du conteneur à exposer est : `3306`
* livenessProbe => On indiquera à K8s que le Pod doit être considéré Up lorsque le port 3306 est accessible
* Monter un volume persistent par le PVC précédement créé qui sera monté sur le répertoire `/var/lib/mysql`

<details><summary>Correction</summary>

```yaml
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

</details>

### Service MySQL

Créez un service qui exposera le déploiement précédement créé
* nom du service : `wordpress-mysql`
* Type : `clusterIP` . Parce que nous n'aurons pas besoin que la BD soit exposé à l'extérieur. Il faut juste qu'elle soit accéssible au serveur web qui lui est dans le cluster.

<details><summary>Correction</summary>

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
```

</details>

## Déploiement du Frontend Wordpress

### PVC Wordpress

Gestion de la persistance des données.
Créez Un PVC pour l'appli Wordpress qui utilisera par défaut le StorageClass de votre cluster
* nom : wp-pv-claim
* accessModes : ReadWriteOnce
* storage: 20Gi

<details><summary>Correction</summary>

```yaml
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
```

</details>

### Déploiement Wordpress

Créer un déploiement pour l'application Wordpress avec les caractéristiques suivantes
* Nom: `wordpress`
* image : `wordpress:4.8-apache`
* Importer les variable d'environnement suivantes
  * `WORDPRESS_DB_HOST` dont la valeur est le nom du service de la Base de données
  * `WORDPRESS_DB_PASSWORD` dont la valeur est celle du secret : `mysql-pass`, clé : `password`
* le port du conteneur à exposer est : `80`
* Monter un volume persistent par le PVC précédement créé qui sera monté sur le répertoire `/var/www/html`

<details><summary>Correction</summary>

```yaml
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

</details>

### Service Wordpress de type LoadBalancer

Créez un service qui exposera le déploiement de l'application Wordpress précédement créé
* nom du service : `wordpress`
* Type : `LoadBalancer` . Parce que nous aurons besoin que les utilisateurs y accèdent de l'extérieur. Si vous êtes sur un cluster local, optez pour `NodePort`

<details><summary>Correction</summary>

```yaml
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
```

</details>

## Résultat final

On peut rassembler l'ensemble des manifest dans 2 séparés `db-wordpress.yml` et `front-wordpress.yml`

<details><summary>Correction</summary>

```bash
$ kubectl apply -f db-wordpress.yml
secret/mysql-pass created
service/wordpress-mysql created
persistentvolumeclaim/mysql-pv-claim created
deployment.apps/wordpress-mysql created

$ kubectl apply -f front-wordpress.yml
service/wordpress created
persistentvolumeclaim/wp-pv-claim created
deployment.apps/wordpress created

$ kubectl get deployment,pods,svc,secret,pvc,pv,sc

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
