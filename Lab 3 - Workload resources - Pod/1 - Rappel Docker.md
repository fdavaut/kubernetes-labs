# Rappel Docker - Manipuler Docker et Dockerfile

Cet TP sera effectué sur la machine `minikube-server`

## Explorez l'environnement Docker avec les commandes de base

Commandes de base de Docker

<details><summary>Correction</summary>

```bash
docker --help
```
```bash
docker ps
```
```bash
docker image ls
```
```bash
docker container ls
```

</details>

## Ecrire Dockerfile

Créer un Simple Dockerfile qui lance un conteneur nginx avec les fichiers Web contenus dans le répertoire `"Ressources"`
L'image de base sera la dernière version de Nginx et les ports 80 et 443 devront être exposés

<details><summary>Correction</summary>

Créer ce fichier Dockerfile dans le répertoire `"Ressources"`

```Dockerfile
FROM nginx:latest

WORKDIR /usr/share/nginx/html
ADD . .

EXPOSE 80 443 	

CMD ["nginx", "-g", "daemon off;"]
```

</details>

## Construire l'image de votre application - Build & Tag

Créer l'image (build) avec comme tag `logindockerhub/lab1-simpleapp:1.0`

<details><summary>Correction</summary>

```Bash
sudo docker build -t mpakoupete/lab1-simpleapp:1.0 .
```

</details>

Faites un push de l'image dans le regitry Docker

<details><summary>Correction</summary>

```Bash
sudo docker login
sudo docker push mpakoupete/lab1-simpleapp:1.0
```

</details>

## Lancer le conteneur

Lancer à présent le conteneur en mode détaché avec comme nom `simpleapp`

<details><summary>Correction</summary>

```Bash
sudo docker run --name simpleapp -d mpakoupete/lab1-simpleapp:1.0
```

</details>

Arrivez-vous à accéder aux site sur la machine host ?

<details><summary>Correction</summary>

Non. Car il faut faire un Port mapping pour accéder à l'intérieur du conteneur

</details>

Accéder à l'intérieur du conteneur et vérifier qu'avec le `curl localhost` le site est bien accessible.

<details><summary>Correction</summary>

```Bash
sudo docker ps
# identifiez l'ID de votre conteneur et accédez à l'intérieur du conteneur
sudo docker exec -it b0792f56c7e0 sh
```

</details>

## Lancer un autre conteneur

Lancez un autre conteneur `simpleapp2` avec le mapping de ports sur le 8080 de votre host
Vérifiez que le site web s'affiche bien `http://localhost:8080`

<details><summary>Correction</summary>

```Bash
sudo docker run --name simpleapp2 -p8080:80 -d mpakoupete/lab1-simpleapp:1.0
```

</details>

## Apportez une modification à l'image et mise à jour de l'image - Rebuild après modification

Faite une modification du fichier `index.html` et faites à nouveau un build de l'image avec pour tag :2
Lancez un autre conteneur `simpleapp2` à partir de cette nouvelle image avec le mapping de ports sur le 8081 de votre poste local.
Vérifiez que le site web s'affiche bien `http://localhost:8081`

<details><summary>Correction</summary>

```Bash
sudo docker build -t mpakoupete/lab1-simpleapp:2.0 .
sudo docker run --name simpleapp02 -p8081:80 -d mpakoupete/lab1-simpleapp:2.0
```

</details>

## Manipulation Docker

* Lister les images
* Lister les conteneurs
* Stoper le premier conteneur `simpleapp`
* Lister à nouveau les conteneurs en cours d'exécution et ensuite lister tous les conteneurs y compris ceux qui sont stoppés
* supprimer la première image de votre host
* Visualisez les logs du conteneur `simpleapp2` lorsque vous accédez au site web

<details><summary>Correction</summary>

```Bash
sudo docker images
sudo docker image ls
sudo docker container ls
sudo docker stop simpleapp
sudo docker container ls -a
sudo docker logs simpleapp2
sudo docker logs -f  simpleapp2
sudo docker container inspect simpleapp2
```

</details>
