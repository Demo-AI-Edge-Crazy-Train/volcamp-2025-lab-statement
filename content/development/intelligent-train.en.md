+++
title = "Predict command"
draft= false
weight= 4
+++

In this section we will import the new model trained before.


- Open a new terminal

![terminal](/images/dev-section/new-terminal-bash.png)

- Run the commands below, replace user_id with your assigned user name: 

```
cd intelligent-train
curl -o models/model.onnx http://minio.minio:9000/<user_id>/models/model.onnx
```
If you have not finish the training of the model use the commands below,

```
cd intelligent-train
curl -o models/model.onnx http://minio.minio:9000/model-registry/models/model.onnx
```