+++
title = "Pipelines CI/CD"
draft = false
weight = 1
[[resources]]
  src = '**.png'
+++

Dans cette étape, vous devrez déployer les pipelines CI/CD qui construiront les images de conteneur des cinq composants clés du train :

- **capture-app**
- **intelligent-train**
- **monitoring-app**
- **train-ceq-app**
- **train-controller**

Pour vous aider, un Chart Helm est présent dans le mono repo de l'application (dossier `tekton-pipelines`).

Vous déploierez les pipelines tekton depuis votre environnement OpenShift DevSpaces (ce sera plus simple).

Pour cela, ouvrez un terminal dans VScode.

- Ouvrez le menu hamburger (trois traits horizontaux en haut, à gauche) depuis votre workspace DevSpaces.
- Cliquez sur **Terminal** > **New Terminal**.

Depuis le terminal, découvrez les projets auxquels vous avez accès.

```sh
oc get projects
```

Vous devriez voir deux projets OpenShift :

- Votre workspace DevSpaces (`$USERID-devspaces-$RANDOM`)
- Le projet de test (`$USERID-test`)

Récupérez le nom du projet de test dans une variable d'environnement.

```sh
TEST_NS=$(oc get projects -o name -l env=test | cut -d / -f 2 | head -n 1)
echo "Using namespace $TEST_NS"
```

Générez les manifests YAML des pipelines tekton.

```sh
helm template pipelines /projects/rivieradev-app/tekton-pipelines --set namespace="$TEST_NS" > /projects/rivieradev-app/tekton-pipelines.yaml
```

Ouvrez le fichier YAML généré dans VScode.

```sh
code-oss /projects/rivieradev-app/tekton-pipelines.yaml
```

Observez les objets Kubernetes générés.

Créez les objets dans votre projet OpenShift de test.

```sh
helm template pipelines /projects/rivieradev-app/tekton-pipelines --set namespace="$TEST_NS" | oc apply -f -
helm template pipelines /projects/rivieradev-app/tekton-pipelines --set namespace="$TEST_NS" | oc create -f -
```

Les commandes `oc apply` et `oc create` sont utilisées successivement, ce qui peut être surprenant.
Les pipelinesrun ont un nom généré aléatoirement et ne peuvent donc pas être créé par la commande `oc apply`.
Et si vous souhaitez modifier une variable Helm et itérer, il est important d'utiliser `oc apply` pour mettre à jour les objets existants.

Normalement, les pipelines doivent démarrer immédiatement.
Suivez leur avancée à l'aide de la commande suivante.

```sh
watch tkn -n "$TEST_NS" pipelineruns list
```

Vous pouvez aussi suivre les logs d'un pipeline à l'aide de la commande suivante (un menu vous demandera de choisir le pipelinerun à afficher).

```sh
tkn -n "$TEST_NS" pipelineruns logs -f
```

Les pipelines mettent environ 20 minutes pour les plus lents à se terminer.
Pendant que ça compile, c'est le moment de passer à l'étape suivante !
