---
title: "OpenShift"
weight: 4
---

## OpenShift cluster

TODO

## Détails du cluster OpenShift

* **URL de la console du cluster OCP :** `{{< param ocpConsole >}}`

* **URL de l'API du cluster OCP :** `{{< param ocpApi >}}`

Il existe un utilisateur OpenShift dédié pour chaque utilisateur.
Sur votre table, vous trouverez une affiche avec les informations pertinentes.
Pour vous connecter à votre cluster Openshift, [cliquez sur ce lien]({{< param ocpConsole >}}) et renseignez votre nom d'utilisateur et votre mot de passe. Vous aurez accès au *Terminal Web* en cliquant sur l'icône **>_** en haut à droite. Le Terminal Web fournit le client *oc*.

Tout au long du lab, merci de ne pas utiliser **userX** mais l'utilisateur qu'on vous a attribué. 
L'utilisateur du stockage objet (**Minio**) est le même que celui décrit ci-dessus, le mot de passe est : **minio123**.