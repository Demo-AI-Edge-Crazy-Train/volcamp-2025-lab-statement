+++
title = "Réentraînement du modèle"
draft= false
weight= 4
[[ressources]]
  src = '**.png'
+++

Dans cette section, vous naviguerez à travers le code python utilisé pour réentraîner le modèle. Vous adapterez ensuite un pipeline de science des données et l'exécuterez sur Openshift. Enfin, vous visualiserez votre pipeline dans le tableau de bord Openshift AI et récupérerez ses résultats.

**ATTENTION:** Vous n'exécuterez que les premières étapes de l'entraînement du modèle dans les Notebooks Jupyter. L'entrainement complet se déroulera sur Openshift afin de limiter la RAM nécessaire à chaque participant. Dans le cas contraire, l'exécution du modèle d'apprentissage (*transfer-learning.ipynb*) plantera pour cause OOM Killed (dépassement de mémoire allouée). Votre pod sur Openshift sera supprimé et recréé automatiquement. Rien de bien méchant mais votre environnement sera indisponible pendant une minute.

## Naviguez dans le code

Vous avez précédemment cloné un repo git. Vous devriez voir dans le navigateur de fichiers sur la gauche un dossier qui porte le même nom que le projet git : *{{< param gitRepoName >}}*. **Cliquez** dessus. A partir de là, vous devriez voir plusieurs objets :
  - Le dossier *utils/* contient des aides et des dépendances pour l'apprentissage du modèle, comme des fonctions python ou des mappers
  - Le dossier *inference/* contient du matériel pour interroger les modèles après les avoir déployés. Nous utiliserons ce contenu plus tard dans le lab.
  - Le fichier *traffic-signs.pipeline* est un pipeline de science des données généré avec Elyra. Elyra fournit une interface utilisateur graphique qui vous permet de glisser-déposer vos Notebooks ou vos scripts python et de les lier ensemble pour créer des étapes. Vous pouvez exécuter ce pipeline dans Openshift à partir de l'interface graphique.
  - Le notebook *labeling-extraction.ipynb* récupère les images étiquetées avec label studio. Il télécharge les images ainsi que leurs annotations correspondantes.
  - Le Notebook *synthetic-data.ipynb* génère des données synthétiques aléatoires. Il s'agit de données créées artificiellement qui ajouteront des données supplémentaires à l'entraînement du modèle.  
  - Le Notebook *transfer-learning.ipynb* est l'entraînement du modèle lui-même.
  - Le Notebook *comparison.ipynb* compare le modèle de base (qui ne reconnaît pas les panneaux de signalisation Lego) à celui que vous allez entraîner (qui, espérons-le, les reconnaît). Nous voulons nous assurer qu'il n'y a pas de régression dans la nouvelle version du modèle.


### Extraire les images et leurs annotations

Cliquez sur *labeling-extraction.ipynb*. Exécutez le notebook en entier en utilisant l'icône en haut et cliquez sur "restart the kernel" quand on vous le demande (voir ci-dessous) :
![run-notebook](run-notebook.png)
![restart-kernel](restart-kernel.png)

Vous avez peut-être remarqué que ces scripts ont créé un répertoire *dataset*. Ce répertoire *dataset* contient un sous-répertoire *labels* et un sous-répertoire *images*. Il contient les images et les annotations extraites. Dans le même Notebook descendez à la section "Select a random image and display its boundind boxes". Réexécutez cette cellule. Elle choisit une image aléatoire dans le dossier *dataset/images* et affiche dans le notebook les carrés corrspondant à l'annotation sauvegardés dans le dossier *dataset/labels*.

### Générer des données synthétiques

Vous pouvez fermer le Notebook précédent et ouvrir le Notebook *synthetic-data.ipynb*. Ce Notebook génère des données synthétiques de manière aléatoire. Il s'agit de données créées artificiellement qui ajouteront des données supplémentaires à l'entraînement du modèle. Exécutez l'ensemble du Notebook comme expliqué dans la section précédente. Jetez un coup d'œil au code et voyez les exemples dans les sections de visualisation. Exécutez à nouveau l'étape de visualisation pour afficher d'autres exemples de données synthétiques.

### Examiner l'étape d'apprentissage du modèle

Attention à ne pas exécuter le notebook suivant. Il ferait crasher votre environnement car nous avons limité la RAM consommable par environnement.  
Ouvrez le Notebook *transfer-learning.ipynb* et regardez simplement le code.

### Examiner l'étape de comparaison

Attention à ne pas exécuter le notebook suivant. Il ferait crasher votre environnement car nous avons limité la RAM consommable par environnement.  
Ouvrez le notebook *comparison.ipynb* et regardez simplement le code.

## Adapter le pipeline de science des données

Vous allez maintenant adapter un pipeline de science des données pour que votre entraînement soit accéléré au moyen un GPU. Nous avons déployés sur Openshfit quelques petits GPUs partagés où l'entraînement sera exécuté. Cela devrait prendre moins de 8 minutes pour l'ensemble du pipeline.

*  Ouvrez le pipeline de science des données *traffic-signs.pipeline*.

Vous voyez ici une interface graphique où vous pouvez créer et exécuter vos pipelines de science des données. Le pipeline a été créé en glissant-déposant les Notebooks depuis l'explorateur de fichiers.

### Fixer le pipeline

Vous pouvez remarquer que ce pipeline a 4 étapes et 2 liaisons. Il manque 1 liaison entre la troisième (*transfer-learning*) et la quatrième étape (*comparison*). Cliquez sur le point noir à droite de la troisième étape (*transfer-learning*).  Maintenez la touche de la souris enfoncée jusqu'à ce que vous atteigniez le point noir du côté gauche de la quatrième étape (*comparaison*).

Vous devriez obtenir le résultat suivant :
![full-pipeline](full-pipeline.png)

### Examiner les propriétés d'un nœud

Cliquez avec le bouton droit de la souris sur la deuxième étape du pipeline (**synthetic-data**). Un menu s'ouvre. Cliquez sur "Open Properties". Elles apparaissent sur le côté droit. Faites défiler vers le bas et voyez quelques propriétés telles que :
  - Runtime Image : Il s'agit de l'image du conteneur qui sera utilisée pour exécuter le code python extrait de vos Notebook.
  - CPU request : Est la quantité de CPU qui devrait être disponible sur le nœud pour cette étape en particulier.
  - RAM limit : Quantité maximale de RAM autorisée pour cette étape.
  - Pipeline Parameters : Rendre les paramètres de pipeline déclarés globalement disponibles pour cette étape particulière.
  - File Dependencies : Fichiers qui doivent être disponibles sur le conteneur pour l'exécution de l'étape. Ici, nous avons besoin de tout le répertoire *utils/*.
  - Output Files : Ces fichiers générés pendant l'exécution seront disponibles pour les étapes suivantes du pipeline.
  - Kubernetes Secret : Monter un secret à l'intérieur du conteneur. Ici, nous rendons les informations d'identification du stockage d'objets disponibles en tant que variable d'environnement pendant l'exécution.

Vous pouvez remarquer qu'en haut du menu de droite des propriétés, 3 panneaux différents sont disponibles (pipeline properties, pipeline parameters, node properties). N'hésitez pas à naviguer vers les autres panneaux.

### Demander un GPU pour l'étape "transfer-learning".

**Fermer les propriétés** ouvertes à l'étape précédente.
![close-properties](close-properties.png)
Vous allez maintenant **travailler sur la troisième étape**. Encore une fois, veillez à modifier les propriéts de l'étape d'entrainement du modèle et non pas les propriétés vues à l'étape précédente.
Cliquez avec le bouton droit de la souris sur la troisième étape du pipeline (**transfer-learning**). Un menu s'ouvre. Cliquez sur "Open Properties". Elles apparaissent sur le côté droit. Cherchez la propriété **GPU** et sélectionnez **1**. Cela demandera 1 GPU pour l'entraînement de votre modèle. 
![full-pipeline-gpu-count](full-pipeline-gpu-count.png)
Descendez jusqu'en bas.  
Les nœuds contenant des GPU ont des "Taints". Cela signifie que par défaut, aucun conteneur ne peut être executé sur les nœuds avec des taints. Nous devons ajouter une "toleration" pour permettre à l'étape d'entrainement d'utiliser un GPU. Les taints et les "toleration" fonctionnent ensemble pour s'assurer que les pods ne sont pas exexcutés sur des nœuds inappropriés.  
**Click Add** sous la propriété **Kubernetes Tolerations** (en bas du menu des propriétés du nœud). Remplissez les champs comme suit :
- Key : tapez **```nvidia.com/gpu```**
- Operator : sélectionnez **Exists**
- Effect : sélectionnez **NoSchedule**.

Vous devriez avoir à la fin :
![full-pipeline-toleration](full-pipeline-toleration.png)

## Exécuter le pipeline

Il est maintenant temps d'exécuter le pipeline sur Openshift. Cliquez sur le bouton "Run Pipeline" en haut de l'interface graphique d'Elyra. Voir ci-dessous :
![elyra-run](elyra-run.png)
**Si** vous avez un popup vous avertissant que le pipeline  n'est **pas sauvegardé**, cliquez sur le bouton "**Save and Submit**".  
Remplissez les configurations. **Choisissez 10 epochs** comme paramètre du pipeline. On peut définir une epoch comme le nombre de passages d’un dataset d’entraînement par un algorithme. Un nombre insuffisant d'epoch rendra votre modèle inefficace. Trop d'époques entraînera un overfit du modèle (et donc une inefficacité dans la prédiction de nouvelles données) :
![elyra-run-config](elyra-run-config.png)
Après quelques instants, vous verrez s'afficher une fenêtre popup de succès. Dans ce popup, vous pouvez cliquer sur "Run Details" pour sauter certaines instructions de la prochaine session.
![elyra-run-success](elyra-run-success.png)


## Visualiser vos pipelines

### Récupérer les exécutions de pipelines

**Si vous avez manqué le raccourci** de la fenêtre popup d'elyra, suivez ces étapes pour récupérer votre pipeline. Sinon, passez au paragraphe suivant.
Vous pouvez maintenant retourner au tableau de bord d'Openshift AI : [https://rhods-dashboard-redhat-ods-applications.apps.{{< param openshift_domain >}}](https://rhods-dashboard-redhat-ods-applications.apps.{{< param openshift_domain >}})
Sur le côté gauche **cliquez sur "Data Science Pipelines", puis sur "Runs "**. Vous pouvez voir ici l'exécution du pipeline. Un run devrait être visible puisque vous en avez créé un dans la section précédente. Cliquez dessus.

Vous pouvez voir le statut de votre pipeline. Si vous cliquez sur un nœud, il affiche des informations telles que le panneau "Logs". Si vous sélectionnez le conteneur "Main" dans ce panneau, vous verrez les logs associés à l'exécution des notebooks :
![pipeline-run-logs](pipeline-run-logs.png)

Attendez que le pipeline se termine. Vous devriez avoir quelque chose comme ça :
![pipeline-run-succedded](pipeline-run-succedded.png)

## Récupérer les output des pipelines

Toutes les output du pipeline sont sauvegardées dans le stockage objet. Connectez-vous à la console S3 en utilisant ce lien : [{{< param minioConsole >}}]({{< param minioConsole >}}). **Connectez-vous avec le même nom d'utilisateur** que nous vous avons donné au début du lab. **Le mot de passe est ``{{< param minioPass >}}``**. Vous devriez voir plusieurs buckets. Cliquez sur celui qui correspond à votre nom d'utilisateur. Il devrait y avoir un fichier *results.csv* dans lequel se trouve les métriques liées à l'entrainement du modèle que vous pouvez télécharger si vous le souhaitez. Il devrait également y avoir un répertoire qui correspond à votre pipeline. Il commence par "traffic-sign". Ouvrez-le. Vous y trouverez des fichiers html, ipynb et des archives. Cliquez sur le fichier *comparison.html*. Un menu apparaît sur le côté droit. Cliquez sur télécharger et ouvrez ce fichier localement sur votre navigateur. Notez la différence entre les scores du modèle de base et du nouveau modèle. Dans cet exemple, nous avons perdu en précision sur l'ensemble de données original. Mais nous pouvons maintenant détecter les panneaux de signalisation "lego" avec le nouveau modèle.
![output-base-model](output-base-model.png)
![output-new-model](output-new-model.png)
