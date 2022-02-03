# **TP2**

# First steps into the CI world

## Testcontainers
*testcontainers* est une librairie java qui permet de faire des tests sur des conteneurs.

## Github actions configurations
Commentaires sur le fichier .main.yml

## Workflowx
On a séparé les différents jobs du main dans plusieurs fichiers. On a donc le main qui ne fait que tester le backend et qui ne se lance que s'il y a des changements dans backend. A la suite de ce lancement, le build and push du backend est appelé, également que s'il y a eu des changements dans le dossier concerné. De la même manière, le frontend, le main frontend et la base de données sont build and push uniquement s'il y a eu des changements dans leur dossier respectif.