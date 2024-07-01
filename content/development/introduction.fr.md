+++
title = "Introduction"
draft: false
weight: 2
+++

Voici une description de chaque service et de leur interaction :

**train-capture-image-app** : C'est un service Quarkus qui permet de démarrer, tester et d'arrêter la capture vidéo. Il expose des points de terminaison RESTful qui peuvent être appelés par d'autres services ou clients pour contrôler la capture vidéo.

**intelligent-train** : Ce service est  responsable de l'analyse des données de la vidéo capturée. Il  utilise des techniques d'apprentissage automatique pour interpréter les données de la vidéo et de fournir des informations basées sur cette analyse.

**train-ceq-app** : est un service de gestion des événements qui reçoit des informations de l'Intelligent Train et d'autres services, et déclenche des actions appropriées. Par exemple, si l'Intelligent Train détecte un obstacle sur la voie, le service déclenche un événement pour arrêter le train.

**train-controller** : Comme son nom l'indique, ce service est probablement responsable du contrôle du train lui-même. Il recevrait des commandes de services comme le train-ceq et effectuerait des actions sur le train, comme le démarrage, l'arrêt, le changement de vitesse, etc.

**train-monitoring** : Ce service est responsable de la surveillance de l'ensemble du système. Il recueillerait des données de tous les autres services, comme les événements déclenchés, les actions effectuées, l'état du train, etc., et fournirait une vue d'ensemble de l'état du système. Il pourrait également fournir des alertes ou des notifications en cas de problèmes détectés.

Chaque service est indépendant et communique avec les autres en mode asynchrone (MQTT/Kafka). Cela permet une grande flexibilité et évolutivité, car chaque service peut être développé, déployé et mis à l'échelle indépendamment des autres.