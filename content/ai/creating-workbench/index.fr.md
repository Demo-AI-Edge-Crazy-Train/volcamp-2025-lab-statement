+++
title = "Création du Wokrbench"
draft= false
weight= 3
[[ressources]]
  src = '**.png'
+++

## Lancer un Wokrbench

* Une fois que la Data Connection et le Pipeline Server ont été entièrement créés, vous pouvez passer à la création du workbench.
![02-03-create-wb.png](02-03-create-wb.png)
* Assurez-vous qu'il présente les caractéristiques suivantes :  
    * Choisissez un nom, comme par exemple : `My Workbench`  
    * Sélectionnez l'image : `CUSTOM Crazy train lab`
    * Taille du conteneur : `Small`
    * Conservez les paramètres de stockage par défaut
    * En bas, cochez "**Use a data connection**".
    * Faites défiler vers le bas jusqu'à "**Use existing data connection**"
    * Sélectionnez dans la liste la Data Connection "**pipelines**" que vous avez précédemment créée.
    * Cela devrait ressembler à ce qui suit :
![02-02-launch-workbench-01.png](02-02-launch-workbench-01.png)
![02-02-launch-workbench-02.png](02-02-launch-workbench-02.png)
* Créez le workbench et attendez que son statut passe en "Running".
* Une fois que c'est le cas, cliquez sur le lien **Open** pour vous y connecter.
![02-03-open-link.png](02-03-open-link.png)

* Authentifiez-vous avec les mêmes informations que précédemment.
* Il vous sera demandé d'accepter les paramètres suivants :
![02-02-accept.png](02-02-accept.png)

* Faites-le
* Vous devriez maintenant voir ceci :
![02-02-jupyter.png](02-02-jupyter.png)

## Git-Clone le repo du labo

Nous allons cloner le contenu de notre repo Git afin que vous puissiez accéder à tout le matériel nécessaire à l'entrainement du modèle d'IA.

* Utilisation de l'interface Git :
  * Ouvrez l'interface utilisateur Git dans Jupyter :
![git-clone-1.png](git-clone-1.png)
* Entrez l'URL du dépôt Git : ``{{< param gitAIRepoUrl >}}``. Sélectionnez également "Download the repository".
![git-clone-2.png](git-clone-2.png)

A ce stade, votre projet est prêt pour le travail que nous voulons y faire.