+++
title = "Service intelligent-train"
draft= false
weight= 4
+++

Dans cette section nous allons importer le nouveau model entraîné dans la première section.


- Ouverez un nouveau terminal

![terminal](/images/dev-section/new-terminal-bash.png)

- Lancez les commandes ci-dessous : 

```
cd intelligent-train
curl -o models/model.onnx http://minio.minio:9000/<remplacez user_id par le nom d'utiliseur qui vous a été assigné>/models/model.onnx
```
