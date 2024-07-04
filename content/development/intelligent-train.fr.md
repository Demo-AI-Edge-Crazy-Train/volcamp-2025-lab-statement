+++
title = "Predict command"
draft= false
weight= 4
+++

Dans cette section nous allons importer le nouveau model entraîné dans l'activité précédente du lab.


- Ouvrez un nouveau terminal

![terminal](/images/dev-section/new-terminal-bash.png)

- Lancez les commandes ci-dessous, remplacez user_id par le nom d'utiliseur qui vous a été assigné : 

```
cd intelligent-train
curl -o models/model.onnx http://minio.minio:9000/<user_id>/models/model.onnx
```

Si vous n'avez pas fini l'entraînement du modèle, utilisez ces commandes :

```
cd intelligent-train
curl -o models/model.onnx http://minio.minio:9000/model-registry/models/model.onnx
```
