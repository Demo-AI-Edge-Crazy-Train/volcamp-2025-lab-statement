+++
title = "Service intelligent-train"
draft= false
weight= 4
+++

In this section we will import the new model trained in the first section.


- Open a new terminal

![terminal](/images/dev-section/new-terminal-bash.png)

- Run the commands below: 

```
cd intelligent-train
curl -o models/model.onnx http://minio.minio:9000/<replace user_id with your assigned user name>/models/model.onnx
```