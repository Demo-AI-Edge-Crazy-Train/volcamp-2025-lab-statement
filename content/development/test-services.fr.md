+++
title = "Start services"
draft: false
weight: 9
+++

Maintenant que vous avez modifié plusieurs parties de l'application, il est temps de tester l'application dans son ensemble.

1. lancer la commande suivante
```
curl -X 'POST'   'http://localhost:8082/capture/test'   -H 'accept: */*'
```

2. a partir de votre browser visualiser le résultat en utilisant ce lien http://localhost:8086/