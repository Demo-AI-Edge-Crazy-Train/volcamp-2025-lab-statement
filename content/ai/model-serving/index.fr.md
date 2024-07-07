+++
title = "Déploiement du modèle"
draft= false
weight= 6
[[ressources]]
  src = '**.png'
+++

A ce stade, vous allez déployer le modèle que vous venez de créer dans le serveur de modèle d'Openshift AI. Si quelque chose s'est mal passé pendant l'entrainement du modèle, vous pouvez toujours faire cette section. Suivez simplement la première section "Fallback".

Une fois de plus, dans les objets suivants que vous allez créer, **vous êtes priés de remplacer "userX "** par votre véritable ID d'utilisateur.

## Fallback - Vous pouvez sauter cette section si vous avez entrainer votre modèle avec succès

* Dans votre projet Data Science, créez une "Data Connection" qui se réfère à la registry de modèles globale où nous avons stocké un modèle par défaut. Pour ce faire, allez dans votre projet de science des données, faites défiler vers le bas et cliquez sur "Data Connection" ou cliquez directement sur l'onglet "Data Connection" dans le menu supérieur. Veuillez vous référer à [cette section](/ai/creating-project/#create-a-data-connection-for-the-pipeline-server) si vous avez des difficultés à créer une "Data Connection".
* Voici les informations que vous devez saisir :
    - Name: ```Model Registry```
    - Access Key: ```userX``` - **Change with your USER ID**
    - Secret Key: ```{{< param minioPass >}}```
    - Endpoint: ```{{< param minioEndpoint >}}```
    - Region: ```none```
    - Bucket: ```{{< param baseModelBucket >}}```

## Créer un serveur de modèle

Dans votre projet, créez un serveur de modèle. Vous pouvez cliquer ici pour voir tous les modèles déployés :
![go-to-models](go-to-models.png)

* Cliquez sur **Add model server**
![add-model-server.png](add-model-server.png)

* Voici les informations que vous devez entrer :

- Model server name: ```{{< param newModelServerName >}}```
- Serving runtime: ```OpenVINO Model Server```
- Number of model server replicas to deploy: ```1```
- Model server size ```{{< param newModelServerSize >}}```
- Model route ```unchecked```
- Token authorization ```unchecked```

* Le résultat devrait ressembler à ceci:
![add-model-server-config.png](add-model-server-config.png)

* Vous pouvez cliquer sur **Add** pour créer le serveur de modèle.

## Déployer le modèle

Dans votre projet, sous **Models and model servers**, sélectionnez **Deploy model**.

* Cliquez sur **Déployer le modèle**
![select-deploy-model.png](select-deploy-model.png)

* Voici les informations que vous devrez entrer. **Si vous avez suivi le fallback, veuillez remplacer la "Existing data connection - Name" par le nom de la "Data Connection" que vous avez créée (Model Registry)**:

    - Model name: ```{{< param newModelName >}}```
    - Model server: ```{{< param newModelServerName >}}```
    - Model server - Model framework: ```onnx-1```
    - Existing data connection - Name: ```{{< param newModelDataConnection >}}``` - **FOR FALLBACK track: use ```Model Registry```**
    - Existing data connection - Path: ```{{< param newModelPath >}}```

* Le résultat devrait ressembler à ceci
![deploy-a-model.png](deploy-a-model.png)

* Cliquez sur **Deploy model**.
* Si le modèle est déployé avec succès, vous verrez son statut en vert après quelques secondes.
![model-deployed-success.png](model-deployed-success.png)

Nous allons maintenant confirmer que le modèle fonctionne bien en l'interrogeant !

## Interroger le modèle deployé

Une fois que le modèle est servi, nous pouvons l'utiliser comme un endpoint qui peut être requêté. Nous envoyons une requête REST ou gRPC au modèle et obtenons un résultat. Cela s'applique à toute personne travaillant au sein de notre cluster. Il peut s'agir de collègues ou d'applications.

* Tout d'abord, nous devons obtenir l'URL du serveur de modèle.
* Pour ce faire, cliquez sur le lien **Internal Service** dans la colonne **Inference endpoint**.
* Dans le popup, vous verrez quelques URLs pour notre serveur de modèle.
![inference-url.png](inference-url.png)

* Notez ou copiez le **RestUrl**, qui devrait être quelque chose comme `http://modelmesh-serving.{userX}:8008`

Nous allons maintenant utiliser cette URL pour interroger le modèle. Retournez dans votre workbench, c'est-à-dire dans l'environnement jupyter notebooks.

- Dans votre workbench, naviguez vers le notebook `inference/inference.ipynb`. **Mettez à jour la variable** "RestUrl" avec l'url copié précédemment dans votre presse papier.
- Exécutez les cellules du notebook, et assurez-vous que vous comprenez ce qui se passe.

La première section interroge le modèle de base qui a été déployé globalement pour tout le monde. La deuxième section prend endpoint RestUrl et interroge le modèle que vous avez formé et déployé. Vous devriez constater qu'avec le modèle de base, seuls les panneaux de signalisation de limitation de vitesse sont reconnus. Après le réapprentissage du modèle, vous avez maintenant un modèle qui peut mieux détecter les panneaux de signalisation Lego. Félicitations !
