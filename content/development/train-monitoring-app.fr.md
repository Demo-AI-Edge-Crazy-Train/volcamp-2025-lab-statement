+++
title = "Service train-monitoring-app"
draft = false
weight = 7
+++


Le `train-monitoring-app` est une application qui surveille l'état et le comportement du train et de ses composants associés. Ce microservice est en charge de la /l' :

1. **Collecte de données** : L'application `train-monitoring-app` collecte des données à partir d'un topic kafka. Cela inclut les événements produit par `train-ceq-app`.

2. **Analyse des données** : Une fois les données collectées, l'application `train-monitoring-app` ajoute à l'image d'origine les prédiction calculées précedement.

3. **Visualisation des données** : L'application `train-monitoring-app` fournit une interface utilisateur pour visualiser les données du train en temps réel. 



Dans le projet `train-monitoring-app`, vous allez modifier certaines propriétés et le code, suivez les instructions ci-dessous :

1. **Modifier le fichier de configuration** : Ouvrez le fichier de configuration de votre application. Il s'agit généralement d'un fichier nommé `application.properties` ou `application.yml`. Ajoutez les propriétés suivantes :

```properties
mp.messaging.incoming.train-monitoring.connector=smallrye-kafka
mp.messaging.incoming.train-monitoring.topic=${KAFKA_TOPIC_MONITORING_NAME:train-monitoring}
mp.messaging.incoming.train-monitoring.cloud-events=false
mp.messaging.incoming.train-monitoring.value.deserializer=org.apache.kafka.common.serialization.StringDeserializer
```

Ces propriétés configurent l'application pour utiliser le connecteur SmallRye Kafka pour lire les messages du topic Kafka `train-monitoring`. Le deserialiseur est configuré pour convertir les messages de Kafka, qui sont des bytes, en chaînes de caractères.

2. **Modifier la classe ImageProcessing** : Ouvrez le fichier `ImageProcessing.java`. Le chemin exact dépend de la structure de votre projet, mais il pourrait ressembler à quelque chose comme `src/main/java/org/redhat/demo/crazytrain/ImageProcessing.java`.

Ajoutez l'annotation `@Incoming("train-monitoring")` à la méthode `process`. Cela pourrait ressembler à ceci :

```java
@Incoming("train-monitoring")
public void process(String message) {
    // code existant
}
```

L'annotation `@Incoming` indique que cette méthode doit être appelée chaque fois qu'un message est lu du canal `train-monitoring`. Le message est passé à la méthode en tant que paramètre.

Notez que la méthode `process` doit être capable de gérer des chaînes de caractères, puisque c'est ce que nous avons configuré comme deserialiseur. Si la méthode `process` nécessite un type de données différent, vous devrez peut-être modifier le deserialiseur ou convertir les données à l'intérieur de la méthode `process`.

Ces modifications permettent à votre application de consommer des messages du topic Kafka `train-monitoring` et de les traiter avec la méthode `process` de la classe `ImageProcessing`.


3. Compilation du projet

Avant de committer vos modifications, vous devez construire le projet  pour vous assurer qu'il n'y a pas d'erreurs de compilation.
Ouvrez un terminal à la racine du projet train-capture-app.

```
./mvnw clean package
```