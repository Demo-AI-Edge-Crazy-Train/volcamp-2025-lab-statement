+++
title = "Monitoring"
draft = false
weight = 7
+++


The **monitoring-appp** is an application which monitors the status and behaviour of the train and its associated components. This microservice is responsible for /l' :

1. **Data collection**: The **monitoring-appp** application collects data from a kafka topic. This includes events produced by **train-ceq-app**.

2. **Data analysis**: Once the data has been collected, the **monitoring-appp** application adds the predictions calculated above to the original image.

3. **Data visualisation**: The **monitoring-appp** application provides a user interface for viewing train data in real time. 



In the **monitoring-appp** project, you will modify certain properties and the code, follow the instructions below:

1. **Modify the configuration file**: Open the configuration file for your application. This is the file named `src/main/resources/application.properties`. Add the following properties:

```properties
mp.messaging.incoming.train-monitoring.connector=smallrye-kafka
mp.messaging.incoming.train-monitoring.topic=${KAFKA_TOPIC_MONITORING_NAME:train-monitoring}
mp.messaging.incoming.train-monitoring.cloud-events=false
mp.messaging.incoming.train-monitoring.value.deserializer=org.apache.kafka.common.serialization.StringDeserializer
```

These properties configure the application to use the SmallRye Kafka connector to read messages from the Kafka `train-monitoring` topic. The deserializer is configured to convert the Kafka messages, which are bytes, into character strings.

2. **Modify the ImageProcessing class: Open the file `src/main/java/org/redhat/demo/crazytrain/ImageProcessing.java`.

Add the `@Incoming("train-monitoring")` annotation to the `process` method. Below is the result:

```java
@Incoming("train-monitoring")
public void process(String message) {
    // existing code
}
```

The `@Incoming` annotation indicates that this method should be called whenever a message is read from the `train-monitoring` channel. The message is passed to the method as a parameter.

These modifications allow our application to consume messages from the Kafka **train-monitoring** topic and process them with the **process** method of the **ImageProcessing** class.

4. Checking the code 

The **src/main/java/org/redhat/demo/crazytrain/ImageProcessing.java** class should look like this: 

```java
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
    LOGGER.debug("Consumer kafka recived : "+result);
    long start = System.nanoTime();

    ObjectMapper mapper = new ObjectMapper();
    JsonNode jsonNode;
    try {
          jsonNode = mapper.readTree(result);
          JsonNode data = jsonNode.get("data");
          String imageBytesBase64 = data.get("image").asText();
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

The **application.properties** file should look like : 

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



5. Compiling the project

Before committing your changes, you need to build the project to ensure that there are no compilation errors.

- Open a new terminal

![terminal](/images/dev-section/new-terminal-bash.png)

- Run the command below 

```
./mvnw clean package
```