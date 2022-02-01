## Base de données
On télécharge les différentes images :
```bash
sudo docker pull -d httpd
sudo docker pull -d openjdk
sudo docker pull -d postgres
sudo docker pull -d adminer
```
on crée le Dockerfile puis on le build :
```bash
sudo docker build -t tp1 .
```
on crée un network :
```bash
docker network create net
```
on lance dans ce network les images de postgres et d'adminer :
```bash
sudo docker run --name bd --network net -d laurinedup/tp1
sudo docker run --name adminer --network net -p 8080:8080 -d adminer
```
On crée le dossier ./docker-entrypoint-initdb.d et on y ajoute les 2 fichiers 01-CreateScheme.sql et 02-InsertData.sql
On ajoute dans le Dockerfile un COPY pour chaque fichier :
COPY ./docker-entrypoint-initdb.d/01-CreateScheme.sql . docker-entrypoint-initdb.d
COPY ./docker-entrypoint-initdb.d/02-InsertData.sql ./docker-entrypoint-initdb.d
(*COPY machine_hôte conteneur*)
puis on rebuild le Dockerfile et on re-run les images.
# Persistance des données :
on sauvegarde les données dans temp/data si destruction du conteneur
sudo docker run -d --name bd --network net -v /tmp/data:/var/lib/postgresql/data laurinedup/tp1
avec *-v machine hôte:conteneur*
**TEST :**
on ajoute une donnée via adminer, on stoppe et supprime le conteneur bd puis on le recrée, notre données est bien là 

# Pour les ports :
*machine hôte : conteneur*

## Java
# Basics
Dans le makefile on ajoute :
FROM openjdk:11
*on copie le fichier Main dans le dossier src du conteneur*
COPY Main.java /src/ 
*on interprète le .java pour obtenir le .class*
RUN javac /src/Main.java
*on lance le .class*
CMD java /src/Main.java

# multistage build : 
1ere partie : build le projet donc possède tous les binaires et les dépendances nécessaires
2ème partie : lance le projet et c'est tout
permet d'utiliser moins de ressources et limites les couches de fichier avec les COPY


## simple API
on ajoute dans le fichier de config l'identifiant, le mot de passe et le serveur postgres.
on build avec le dockerfile créé précédemment
```bash
sudo docker build -t laurinedup/api
```
on crée un nouveau network
```bash
sudo docker network create netapi
```
on lance dans le network l'image qui vient d'être créée et l'image de la base de données créée au début du tp
```bash
sudo docker run --name bdd --network apinet -d laurinedup/tp1
sudo docker run --name api --network apinet -p 8083:8080 -d laurinedup/api
```

## httpd
# Basics
on crée un nouveau Dockerfile dans un nouveau dossier contenant un dossier contenant index.html
grâce à la doc on sait qu'il faut utiliser FROM httpd:2.4
et on met le index.html dans : COPY ./public_html/ /usr/local/apache2/htdocs/

on build et lance l'image créée :
```bash
sudo docker build -t laurinedup/front .
sudo docker run -dit --name front -p 8084:80 --network netfront laurinedup/front
```
en accédant à localhost://8084 on a notre index.html

# Configurations
on récupère le fichier de conf avec :
```bash
sudo docker run --rm httpd:2.4 cat /usr/local/apache2/conf/httpd.conf > my-httpd.conf
```

# Reverse proxy
on le modifie grâce à la doc en ajoutant notre lien vers le backend
on build et on lance la bdd, le backend et le frontend dans le même network
```bash
sudo docker run --name bdd --network netfront -p 8083:8080 -d laurinedup/tp1
sudo docker run --name api --network netfront -p 8082:8080 -d laurinedup/api
sudo docker run -dit --name front -p 80:80 --network netfront laurinedup/front
```

## Docker compose
on modifie le fichier docker-compose.yml en renseignant les paramètres de nos conteneurs créés précédemment.
pour le build de chaque composant on renseigne le chemin jusqu'au dossier contenant le dockerfile
on renseigne un networks dans lequel démarrer les conteneurs.
dans le depends-on donne les conteneurs nécessaires au lancement de celui-ci (bdd pour l'API, API pour le frontend)

puis on build avec :
```bash
sudo docker-compose build (+ service)
```
et on lance avec :
```bash
sudo docker-compose up
```
commandes interessantes avec docker-compose :
```bash
sudo docker-compose stop
sudo docker-compose down (commme rm)
sudo docker-compose restart (+ service)
```


