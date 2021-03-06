# Intro
 On crée le fichier setup.yml
 et on renseigne le chemin vers la clé privée ainsi que l'adresse IP.

On teste le ping sur la machine :
```bash
ansible all -i ansible/inventory/setup.yml -m ping
```
Récupérer la version de l'OS :
```bash
ansible all -i ansible/inventory/setup.yml -m setup -a "filter=ansible_distribution*"
```
Désinstallation du service Apache :
```bash
sudo ansible all -i ansible/inventory/setup.yml -m yum -a "name=httpd state=absent" --become
```

# Playbooks
## First playbook
Exécution du playbook :
```bash
ansible-playbook -i ansible/inventory/setup.yml ansible/playbook.yml
```
Vérification de la syntaxe du playbook :
```bash
ansible-playbook --syntax-check -i ansible/inventory/setup.yml ansible/playbook.yml
```
Connexion à la machine en SSH : 
```bash
ssh -i id_rsa centos@laurine.dupont.takima.cloud
```
## Advanced playbook
Création du rôle docker: 
```bash
ansible-galaxy init roles/docker
```
Puis on déplace la partie tasks du playbook dans le main.yml du répertoire tasks du rôle docker.
Dans le playbook.yml on met à la place :
```yml
roles:
    - docker
```
pour spécifier qui doit exécuter les tâches.


On crée de la même mainère 4 autres rôles: network, database, app, proxy
```bash
ansible-galaxy init roles/network
ansible-galaxy init roles/database
ansible-galaxy init roles/app
ansible-galaxy init roles/proxy
```

Les commentaire sont dans les fichiers main de chacun de ces rôles.

# Front
Après avoir télécharger le front, on modifie le fichier my-httpd.conf afin d'effectuer une redirection selon les URLs : </br>
- si on interroge l'api, on redirige vers celle-ci (données brutes)
- on a également dupliqué ces 2 lignes pour rediriger l'utilisateur vers le front si l'URL interrogée était *localhost:80*
Pour ce faire, on a modifié dans le fichier *.env.production* la variable *VUE_APP_API_URL* pour envoyer sur l'api et non sur le port 8080. </br>
On a également ajouté dans le fichier *docker-compose* le service du front-main qui a été téléchargé.</br>