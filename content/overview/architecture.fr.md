---
title: "Architecture"
weight: 2
---

## Architecture

![global architecture](/images/architecture-global.png)


Le train est équipé d’un moteur et d’un Hub Lego. Le Hub Lego reçoit les ordres d’accélération, décélération et freinage via le protocole Bluetooth Low Energy.

Nous avons intégré une carte Nvidia Jetson Orin dans le train Lego. La carte Nvidia Jetson Orin est un System on Chip (SoC) qui intègre tous les composants nécessaires à notre mission : CPU, RAM, stockage et un puissant GPU pour accélérer les calculs. Cette carte reçoit le flux vidéo de la caméra embarquée et transmet les ordres au Hub Lego via le protocole Bluetooth. Elle est alimentée par une batterie portable pour la durée de la mission.

Nous opérons dans un environnement de Edge Computing. Sur la carte Nvidia Jetson, nous installons Red Hat Device Edge, une variante de RHEL adaptée aux contraintes du Edge Computing. Nous y déployons Microshift, la version de Kubernetes de Red Hat conçue pour le Edge. Ensuite, nous déployons nos microservices, un broker MQTT et le modèle d’intelligence artificielle sur Microshift, en utilisant un mécanisme “over-the-air”.

Pour la durée de la mission, le Jetson est connecté à un cluster OpenShift dans le cloud AWS via une connexion 5G. Dans le cloud AWS, nous disposons d’une machine virtuelle RHEL 9 qui nous permet de construire nos images RHEL pour le Jetson. Notre application de vidéo surveillance fonctionne dans le cluster OpenShift, ce qui nous permet de surveiller à distance la caméra embarquée du train. Le flux vidéo est relayé depuis le Jetson via un broker Kafka.

De plus, des pipelines MLOps sont mis en place pour entraîner le modèle d’intelligence artificielle, ainsi que des pipelines CI/CD pour construire les images de conteneurs de nos microservices pour les architectures x86 et ARM.