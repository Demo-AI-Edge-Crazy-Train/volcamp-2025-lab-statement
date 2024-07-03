+++
title = "Monitoring"
draft = false
weight = 7
+++


Le **monitoring-app** est une application qui surveille l'état et le comportement du train et de ses composants associés. Ce microservice est en charge de  :

1. **La collecte de données** : L'application **monitoring-appp** collecte des données à partir d'un topic kafka. Cela inclut les événements produit par **train-ceq-app**.

2. **L'Analyse des données** : Une fois les données collectées, l'application **monitoring-appp** ajoute à l'image d'origine les prédiction calculées précedement.

3. **La Visualisation des données** : L'application **monitoring-appp** fournit une interface utilisateur pour visualiser les données du train en temps réel. 



Dans le projet **monitoring-app**, vous allez modifier certaines propriétés et le code via les instructions ci-dessous :

1. **Modifier le fichier de configuration** : Ouvrez le fichier de configuration de votre application. Il s'agit du fichier nommé `src/main/resources/application.properties`. Ajoutez les propriétés suivantes :

```properties
mp.messaging.incoming.train-monitoring.connector=smallrye-kafka
mp.messaging.incoming.train-monitoring.topic=${KAFKA_TOPIC_MONITORING_NAME:train-monitoring}
mp.messaging.incoming.train-monitoring.cloud-events=false
mp.messaging.incoming.train-monitoring.value.deserializer=org.apache.kafka.common.serialization.StringDeserializer
```

Ces propriétés configurent l'application pour utiliser le connecteur SmallRye Kafka pour lire les messages du topic Kafka `train-monitoring`. Le deserialiseur est configuré pour convertir les messages de Kafka, qui sont des bytes, en chaînes de caractères.

2. **Modifier la classe ImageProcessing** : Ouvrez le fichier `src/main/java/org/redhat/demo/crazytrain/processing/ImageProcessing.java`.

Ajoutez l'annotation `@Incoming("train-monitoring")` à la méthode `process`. Ci-dessous le résultat :

```java
@Incoming("train-monitoring")
public void process(String message) {
    // code existant
}
```

L'annotation `@Incoming` indique que cette méthode doit être appelée chaque fois qu'un message est lu du canal `train-monitoring`. Le message est passé à la méthode en tant que paramètre.

Ces modifications permettent à notre application de consommer des messages du topic Kafka **train-monitoring** et de les traiter avec la méthode **process** de la classe **ImageProcessing**.

4. Vérification du code 

La classe **src/main/java/org/redhat/demo/crazytrain/processing/ImageProcessing.java** devrait ressembler à : 

```Java
package org.redhat.demo.crazytrain.processing;

import java.util.Base64;
import java.util.concurrent.TimeUnit;

import org.eclipse.microprofile.reactive.messaging.Incoming;

import jakarta.ws.rs.core.MediaType;
import jakarta.ws.rs.GET;
import jakarta.ws.rs.Path;
import jakarta.ws.rs.Produces;
import jakarta.annotation.PostConstruct;
import jakarta.inject.Inject;


import org.eclipse.microprofile.config.inject.ConfigProperty;

import org.jboss.logging.Logger;
import org.opencv.core.CvType;
import org.opencv.core.Mat;

import org.redhat.demo.crazytrain.services.SaveService;

import com.fasterxml.jackson.core.JsonProcessingException;
import com.fasterxml.jackson.databind.JsonMappingException;
import com.fasterxml.jackson.databind.JsonNode;
import com.fasterxml.jackson.databind.ObjectMapper;

import io.micrometer.core.instrument.MeterRegistry;
import io.micrometer.core.instrument.Timer;
import io.smallrye.mutiny.Multi;
import io.smallrye.mutiny.operators.multi.processors.BroadcastProcessor;

@Path("/train-monitoring")
public class ImageProcessing {
  private static final Logger LOGGER = Logger.getLogger(ImageProcessing.class);
  private final BroadcastProcessor<String> broadcastProcessor = BroadcastProcessor.create();
  @Inject
  SaveService saveService;
  @ConfigProperty(name = "monitoring.saveImage")
  boolean saveImage;

  @ConfigProperty(name = "monitoring.tmpFolder") 
  String tmpFolder;

    @Inject
    MeterRegistry registry;

    Timer timer;

    @PostConstruct
    void init() {
        timer = Timer.builder("image.processing.time")
            .description("Time taken to get a message from Kafka and process it")
            .register(registry);
    }

  @Incoming("train-monitoring")
  public void process(String result) {
    LOGGER.debug("Consumer kafka recived  : "+result);
    long start = System.nanoTime();

    ObjectMapper mapper = new ObjectMapper();
    JsonNode jsonNode;
    try {
          jsonNode = mapper.readTree(result);
          JsonNode data = jsonNode.get("data");
          String imageBytesBase64  = data.get("image").asText();
            broadcastProcessor.onNext(imageBytesBase64);
            long end = System.nanoTime();
            timer.record(end - start, TimeUnit.NANOSECONDS);
    } catch (JsonMappingException e) {
      // TODO Auto-generated catch block
      e.printStackTrace();
    } catch (JsonProcessingException e) {
      // TODO Auto-generated catch block
      e.printStackTrace();
    }
  }  

  @GET
  @Produces(MediaType.SERVER_SENT_EVENTS)
  public Multi<String> stream() {
      return broadcastProcessor.toHotStream();
  }

  
}
```

Le fichier **application.properties** doit ressemble à : 

```properties
%dev.quarkus.http.port=8086
# Configure the Kafka source 
kafka.bootstrap.servers=${KAFKA_BOOTSTRAP_SERVERS:localhost:9092}
mp.messaging.incoming.train-monitoring.connector=smallrye-kafka
mp.messaging.incoming.train-monitoring.topic=${KAFKA_TOPIC_MONITORING_NAME:train-monitoring}
mp.messaging.incoming.train-monitoring.cloud-events=false
mp.messaging.incoming.train-monitoring.value.deserializer=org.apache.kafka.common.serialization.StringDeserializer

mp.messaging.outgoing.commands-out.connector=smallrye-kafka
mp.messaging.outgoing.commands-out.topic=${KAFKA_TOPIC_COMMAND_CAPTURE_NAME:train-command-capture}
mp.messaging.outgoing.commands-out.value.serializer=org.apache.kafka.common.serialization.StringSerializer

monitoring.saveImage=${SAVE_IMAGE:false}
monitoring.tmpFolder=${TMP_FOLDER:/tmp/crazy-train-images}
quarkus.log.level=${LOGGER_LEVEL:INFO}

quarkus.swagger-ui.always-include=true

%dev.kafka.topic.train-command-capture.replication.factor=1
%dev.kafka.topic.train-monitoring.replication.factor=1
```



5. Compilation du projet

Avant de committer vos modifications, vous devez construire le projet  pour vous assurer qu'il n'y a pas d'erreurs de compilation.

- Ouverez un nouveau terminal

![terminal](/images/dev-section/new-terminal-bash.png)

- Lancez la commande ci-dessous 

```
./mvnw clean package
```