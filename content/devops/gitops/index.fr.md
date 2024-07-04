+++
title = "GitOps"
draft = false
weight = 2
[[resources]]
  src = '**.png'
+++

Dans cette étape, vous devrez déployer dans OpenShift les images de conteneur des cinq composants clés du train :

- **capture-app**
- **intelligent-train**
- **monitoring-app**
- **train-ceq-app**
- **train-controller**

Pour vous aider, un Chart Helm est présent dans le mono repo de l'application (dossier `deployment`).

Vous déploierez les composants depuis votre environnement OpenShift DevSpaces (ce sera plus simple).

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
helm template deployment /projects/rivieradev-app/deployment --set namespace="$TEST_NS" > /projects/rivieradev-app/deployment.yaml
```

Ouvrez le fichier YAML généré dans VScode.

```sh
code-oss /projects/rivieradev-app/deployment.yaml
```

Observez les objets Kubernetes générés.

Créez les objets dans votre projet OpenShift de test.

```sh
helm template deployment /projects/rivieradev-app/deployment --set namespace="$TEST_NS" | oc apply -f -
```

Suivez la progression des Pods l'aide de la commande suivante.

```sh
oc -n "$TEST_NS" get pods -w
```

Note: les pods ne peuvent se déployer correctement qu'une fois que les pipelines de la section précédente se sont exécuté avec succès.
