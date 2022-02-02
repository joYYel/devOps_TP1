# Base de données
On télécharge les différentes images :
```bash
sudo docker pull -d httpd
sudo docker pull -d openjdk
sudo docker pull -d postgres
sudo docker pull -d adminer
```
On crée le Dockerfile puis on le build :
```bash
sudo docker build -t tp1 .
```
On crée un network :
```bash
docker network create net
```
On lance dans ce network les images de postgres et d'adminer :
```bash
sudo docker run --name bd --network net -d laurinedup/tp1
sudo docker run --name adminer --network net -p 8080:8080 -d adminer
```
On crée le dossier ./docker-entrypoint-initdb.d et on y ajoute les 2 fichiers 01-CreateScheme.sql et 02-InsertData.sql. <br/>
On ajoute dans le Dockerfile un COPY pour chaque fichier :
```bash
COPY ./docker-entrypoint-initdb.d/01-CreateScheme.sql . docker-entrypoint-initdb.d
COPY ./docker-entrypoint-initdb.d/02-InsertData.sql ./docker-entrypoint-initdb.d
```
(*COPY machine_hôte conteneur*) <br/>
Puis on rebuild le Dockerfile et on re-run les images.

## Persistance des données :
On sauvegarde les données dans temp/data si destruction du conteneur
```bash
sudo docker run -d --name bd --network net -v /tmp/data:/var/lib/postgresql/data laurinedup/tp1
```
avec *-v machine_hôte:conteneur* <br/>
**TEST :**
On ajoute une donnée via adminer, on stoppe et supprime le conteneur bd puis on le recrée, notre donnée est bien là.

# Java
## Basics
Dans le makefile on ajoute :
```bash
FROM openjdk:11
```
On copie le fichier Main dans le dossier src du conteneur
```bash
COPY Main.java /src/
```
On interprète le .java pour obtenir le .class
```bash
RUN javac /src/Main.java
```
On lance le .class
```bash
CMD java /src/Main.java
```

## Multistage build : 
1ere partie : build le projet donc possède tous les binaires et les dépendances nécessaires <br/>
2ème partie : lance le projet et c'est tout <br/>
Ça permet d'utiliser moins de ressources et limites les couches de fichier avec les COPY <br/>

# simple API
On ajoute dans le fichier de config l'identifiant, le mot de passe et le serveur postgres. <br/>
On build avec le dockerfile créé précédemment
```bash
sudo docker build -t laurinedup/api
```
On crée un nouveau network
```bash
sudo docker network create netapi
```
On lance dans le network l'image qui vient d'être créée et l'image de la base de données créée au début du tp
```bash
sudo docker run --name bdd --network apinet -d laurinedup/tp1
sudo docker run --name api --network apinet -p 8083:8080 -d laurinedup/api
```

# httpd
## Basics
On crée un nouveau Dockerfile dans un nouveau dossier contenant un dossier contenant index.html grâce à la doc on sait qu'il faut utiliser FROM httpd:2.4 et on met le index.html dans le conteneur avec :
```bash
COPY ./public_html/ /usr/local/apache2/htdocs/
```
On build et lance l'image créée :
```bash
sudo docker build -t laurinedup/front .
sudo docker run -dit --name front -p 8084:80 --network netfront laurinedup/front
```
En accédant à localhost://8084 on a notre index.html <br/>

## Configurations
On récupère le fichier de conf avec :
```bash
sudo docker run --rm httpd:2.4 cat /usr/local/apache2/conf/httpd.conf > my-httpd.conf
```

## Reverse proxy
On le modifie grâce à la doc en ajoutant notre lien vers le backend <br/>
On build et on lance la bdd, le backend et le frontend dans le même network <br/>
```bash
sudo docker run --name bdd --network netfront -p 8083:8080 -d laurinedup/tp1
sudo docker run --name api --network netfront -p 8082:8080 -d laurinedup/api
sudo docker run -dit --name front -p 80:80 --network netfront laurinedup/front
```

# Docker compose
On modifie le fichier *docker-compose.yml* en renseignant les paramètres de nos conteneurs créés précédemment. <br/>
Pour le build de chaque composant on renseigne le chemin jusqu'au dossier contenant le dockerfile. <br/>
On renseigne un networks dans lequel démarrer les conteneurs. <br/>
Dans le *depends-on* donne les conteneurs nécessaires au lancement de celui-ci (bdd pour l'API, API pour le frontend). <br/>

Puis on build avec :
```bash
sudo docker-compose build (+ service)
```
et on lance avec :
```bash
sudo docker-compose up
```
Commandes interessantes avec docker-compose :
Stopper les conteneurs lancés avec docker-compose
```bash
sudo docker-compose stop
```
Supprimer les conteneurs lancés avec docker-compose
```bash
sudo docker-compose down
```
Relancer tous les conteneurs (ou un)
```bash
sudo docker-compose restart (+ service)
```

# Docker compose
On nomme nos images qui vont être publiées :
```bash
docker tag laurinedup/tp1 laurinedup/bdd:1.0
docker tag laurinedup/tp1 laurinedup/api:1.0
docker tag laurinedup/tp1 laurinedup/front:1.0
```
On les publie :
```bash
sudo docker push laurinedup/bdd:1.0
sudo docker push laurinedup/api:1.0
sudo docker push laurinedup/front:1.0
```

