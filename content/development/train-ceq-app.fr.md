+++
title = "Service intelligent-train"
draft: false
weight: 5
+++

Le `train-ceq-app` est une application basée sur Apache Camel, une bibliothèque Java pour la mise en œuvre des intégrations d'applications à l'aide de l'Enterprise Integration Patterns (EIP). 

L'application `train-ceq-app` est principalement composée de routes Camel définies dans le fichier `PostProcessingRoute.xml`. Ces routes définissent comment les messages sont consommés, transformés et transmis à d'autres services ou destinations.

Voici une explication détaillée du fonctionnement de la route `postproscesing-route` :

1. **Consommation des messages** : La route commence par consommer les messages du broker MQTT à l'aide de l'URI `paho:{{train.mqtt.source.topicName}}?brokerUrl={{train.mqtt.brokerUrl}}`. Les messages consommés sont ensuite enregistrés dans le log.

2. **Sauvegarde du message initial** : Le message initial est sauvegardé dans l'entête du message sous le nom "origin" pour pouvoir être récupéré plus tard.

3. **Extraction de l'ID du message** : L'ID du message est extrait à l'aide de l'expression JSONPath `$.id` et est enregistré dans l'entête du message.

4. **Désérialisation du message** : Le message est désérialisé en un objet Java de type `org.redhat.demo.crazytrain.model.Result` à l'aide de la bibliothèque Jackson.

5. **Traitement du message** : Le message est ensuite traité par le `CommandProcessor`.

6. **Publication du message** : Le message traité est publié sur un autre topic MQTT à l'aide de l'URI `paho:{{command.mqtt.destination.topicName}}?brokerUrl={{train.mqtt.brokerUrl}}`.

7. **Récupération du message initial** : Le message initial sauvegardé dans l'entête est récupéré et remis dans le corps du message.

8. **Génération de l'événement cloud** : Un événement cloud est généré à partir du message initial et est traité par le `CloudEventProcessor`.

9. **Publication de l'événement cloud** : L'événement cloud est publié sur un topic Kafka à l'aide de l'URI `kafka:{{monitoring.kafka.destination.topicName}}?brokers={{train.kafka.brokerUrl}}`.

La route `command-capture-image` fonctionne de manière similaire, mais consomme les messages d'un topic Kafka, extrait une commande à partir du message, et envoie une requête HTTP POST à une URL spécifiée avec la commande comme paramètre.



Dans une route camel les données sont continuellement transformés par les différentes actions qui se trouvent dans le fichier `PostProcessingRoute.xml. Parfois il est nécessaire d'effectuer un contrôle sur le message d'origine et non pas sur le message transformé. Il est de bonne pratique de sauvgrader le message initial afin de pouvoir le récupérer plus tard. Avec Camel ceci se fait via les propriétés de l'entete du message. Demande à l'utilisateur d'ajouter l'instruction suivante ' ${body}' et ensuite cette instruction '${header.origin}' 


Dans le projet `train-ceq-app`, vous allez modifier le fichier `PostProcessingRoute.xml` pour sauvegarder le message initial et pouvoir le récupérer plus tard.

1. Ouvrez le fichier `src/main/resources/PostProcessingRoute.xml`.
2. Ajoutez l'instruction suivante juste après la ligne `<log loggingLevel="DEBUG" message="${body}"/>` :

```xml
<setHeader name="origin"><simple>${body}</simple></setHeader>
```

Cette instruction sauvegarde le message initial dans l'entête du message sous le nom "origin".

3. Ensuite, ajoutez l'instruction suivante juste après la ligne `<toD uri="paho:{{command.mqtt.destination.topicName}}?brokerUrl={{train.mqtt.brokerUrl}}"/>` :

```xml
<setBody><simple>${header.origin}</simple></setBody>
```

Cette instruction récupère le message initial sauvegardé dans l'entête et le remet dans le corps du message.

4. Enregistrez vos modifications.

Maintenant, la route `src/main/resources/PostProcessingRoute.xml` sauvegarde le message initial et le récupère plus tard. Cela permet d'effectuer un contrôle sur le message d'origine et non pas sur le message transformé.

Voici à quoi devrait ressembler la route après vos modifications :

```xml
<routes xmlns="http://camel.apache.org/schema/spring">
    <route id="postproscesing-route">  
        <from uri="paho:{{train.mqtt.source.topicName}}?brokerUrl={{train.mqtt.brokerUrl}}"/>     
        <log loggingLevel="DEBUG" message="MQTT message received:"/>
        <log loggingLevel="DEBUG" message="${body}"/>
        <setHeader name="origin"><simple>${body}</simple></setHeader>
        <setHeader name="id"><jsonpath>$.id</jsonpath></setHeader>
        <log message="Id of the message: ${header.id}"/>
        <unmarshal> <json library="Jackson" unmarshalType="org.redhat.demo.crazytrain.model.Result"/></unmarshal>
        <log message="unmarshalling done"/>
        <process ref="CommandProcessor"/>
        <log message="Train Command: ${body}"/>
        <toD uri="paho:{{command.mqtt.destination.topicName}}?brokerUrl={{train.mqtt.brokerUrl}}"/>
        <setBody><simple>${header.origin}</simple></setBody>
        <convertBodyTo type="java.lang.String"/>
        <log loggingLevel="DEBUG" message="generating cloud event ${body}"/>
        <process ref="CloudEventProcessor"/>
        <log  loggingLevel="DEBUG" message="${body}"/>
        <toD uri="kafka:{{monitoring.kafka.destination.topicName}}?brokers={{train.kafka.brokerUrl}}"/>
        <log loggingLevel="DEBUG" message="written into kafka"/>
    </route>
    <route id="command-capture-image">
        <from uri="kafka:{{monitoring.kafka.source.topicName}}?brokers={{train.kafka.brokerUrl}}"/>
        <setHeader name="command"><jsonpath>$.command</jsonpath></setHeader>
        <toD uri="{{train.http.url}}/${header.command}?httpMethod=POST" />
    </route>
</routes>
```
5. Compilation du projet 
Avant de committer vos modifications, vous devez construire le projet  pour vous assurer qu'il n'y a pas d'erreurs de compilation.


Ouvrez un terminal à la racine du projet train-capture-app.

```
./mvnw clean package
```
