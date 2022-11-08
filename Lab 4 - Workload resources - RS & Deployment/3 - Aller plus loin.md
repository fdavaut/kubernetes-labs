# Deployment


De la manière déclarative, créez un déploiement du nom de `robots`, contenant 2 pods, l'image est busybox et la commande qu'il doit exécuter est le script suivant `while true; do tr -dc A-Za-z0-9 </dev/urandom| head -c 13 ; echo '' && sleep 1; done`

<details><summary>Correction</summary>

Pour générer les fichier YAML correspondant, il existe une astuce simple qui est de combiner l'utilisation de kubectl avec l'option `--dry-run=client -o yaml` 
l'option `--dry-run=client`indique à l'PAI server que nous somme entrain de faire des simulations et qu'il ne devra pas créer la ressource
l'otion `- o yaml` est de dire que nous voulons un output de l'objet au format YAML. 

```bash
# Etape 1 => créer un fichier du script

$ cat << _EOF > script.sh
#!/bin/sh
while true; do tr -dc A-Za-z0-9 </dev/urandom| head -c 13 ; echo '' && sleep 1; done
_EOF 

# Etape 2 => créer le fichier YAML du déploiement

$ kubectl create deployment robots --replicas 2  --dry-run=client -o yaml --image busybox -- /bin/sh -c "`cat script.sh`" > robots.yml

# Etape 3 => Visualisez le manifest créé et apportez des modifications si nécessaires
$ cat robots.yml
```

Le fichier YAML créé :

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  creationTimestamp: null
  labels:
    app: robots
  name: robots
spec:
  replicas: 2
  selector:
    matchLabels:
      app: robots
  strategy: {}
  template:
    metadata:
      creationTimestamp: null
      labels:
        app: robots
    spec:
      containers:
      - command:
        - /bin/sh
        - -c
        - |-
          #!/bin/sh
          while true; do tr -dc A-Za-z0-9 </dev/urandom| head -c 13 ; echo '' && sleep 1; done
        image: busybox
        name: busybox
        resources: {}
status: {}
```

Pour déployer donc nous ferons

```bash
kubectl apply -f robots.yml
```

Normalement si le déploiement est bien fait, vous pouvez voir les logs des pods


```bash
$ kubectl logs -f robots-577dbd8d5d-25mdb
F1buq9n5dhonu
QpcV5rcnf5yqm
eSn3raQVvBf0T
TA92AZM9Asjqr
```

</details>