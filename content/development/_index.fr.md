+++
title = "Développement (1h)"
weight = 2
chapter = true
pre = "<b>2. </b>"
+++

### Partie 1

# Développement

## Objectif 

L'objectif de cette partie est de vous familiariser avec les différents microservices de l'application Crazy Train. Vous allez modifier le code de plusieurs projets Quarkus, Python et Nodejs, comprendre comment ils interagissent et tester l'application dans son ensemble.

Voici une description de chaque service et de leur interaction :

**train-capture-image-app** : C'est un service Quarkus qui permet de démarrer, tester et d'arrêter la capture vidéo. Il expose des points de terminaison RESTful qui peuvent être appelés par d'autres services ou clients pour contrôler la capture vidéo.

**intelligent-train** : Ce service est  responsable de l'analyse des données de la vidéo capturée. Il  utilise des techniques d'apprentissage automatique pour interpréter les données de la vidéo et de fournir des informations basées sur cette analyse.

**train-ceq-app** : est un service de gestion des événements qui reçoit des informations de l'Intelligent Train et d'autres services, et déclenche des actions appropriées. Par exemple, si l'Intelligent Train détecte un obstacle sur la voie, le service déclenche un événement pour arrêter le train.

**train-controller** : Comme son nom l'indique, ce service est probablement responsable du contrôle du train lui-même. Il recevrait des commandes de services comme le train-ceq et effectuerait des actions sur le train, comme le démarrage, l'arrêt, le changement de vitesse, etc.

**train-monitoring** : Ce service est responsable de la surveillance de l'ensemble du système. Il recueillerait des données de tous les autres services, comme les événements déclenchés, les actions effectuées, l'état du train, etc., et fournirait une vue d'ensemble de l'état du système. Il pourrait également fournir des alertes ou des notifications en cas de problèmes détectés.

Chaque service est indépendant et communique avec les autres en mode asynchrone (MQTT/Kafka). Cela permet une grande flexibilité et évolutivité, car chaque service peut être développé, déployé et mis à l'échelle indépendamment des autres.

## Instructions

### 1. Modification du projet train-capture-image-app

**train-capture-image-app** est construite avec Quarkus, un framework Java complet, natif de Kubernetes, conçu pour les machines virtuelles Java (JVM) et la compilation native, optimisant Java spécifiquement pour les conteneurs et lui permettant de devenir une plateforme efficace pour les environnements serverless, cloud et Kubernetes.

La fonctionnalité principale de la Capture-App est de contrôler la capture vidéo. Elle offre la possibilité de démarrer et d'arrêter la capture vidéo, probablement via des points de terminaison RESTful exposés. Ces points de terminaison peuvent être appelés à partir de n'importe quel client (comme un navigateur web ou une commande curl dans un terminal) qui supporte HTTP, ce qui en fait une solution flexible et interopérable pour contrôler la capture vidéo.


Dans le projet `train-capture-app/capture-app`, vous allez ajouter deux nouvelles propriétés dans le fichier `application.properties` et modifier la classe `ScheduledCapture.java` pour charger ces propriétés.

1. Ouvrez le fichier `src/main/resources/application.properties`.
2. Ajoutez les deux nouvelles propriétés à la fin du fichier :

```properties
%dev.capture.mock=true
%dev.capture.videoPath=chemin/vers/votre/video.mp4
```

3. Enregistrez vos modifications.

4. Ouvrez le fichier `src/main/java/com/train/capture/app/ScheduledCapture.java`.
5. Ajoutez les annotations `@ConfigProperty` pour charger les nouvelles propriétés. Ajoutez ces lignes en haut de la classe, juste en dessous de la déclaration de la classe :

```java
@ConfigProperty(name = "capture.mock")
boolean mock;

@ConfigProperty(name = "capture.videoPath")
String videoPath;
```

4. Vérification du code 

La classe `src/main/java/com/train/capture/app/ScheduledCapture.java` :

```Java
package org.redhat.demo.crazytrain.captureimage;

import jakarta.enterprise.context.ApplicationScoped;
import jakarta.enterprise.event.Observes;
import jakarta.inject.Inject;
import jakarta.ws.rs.GET;
import jakarta.ws.rs.POST;
import jakarta.ws.rs.Path;
import jakarta.ws.rs.core.Response;
import jakarta.ws.rs.core.Response.ResponseBuilder;

import java.util.ArrayList;
import java.util.List;

import org.eclipse.microprofile.config.inject.ConfigProperty;
import org.eclipse.paho.client.mqttv3.MqttException;
import org.jboss.logging.Logger;
import org.opencv.core.Mat;
import org.opencv.core.Size;
import org.opencv.imgproc.Imgproc;
import org.opencv.videoio.VideoCapture;
import org.opencv.videoio.Videoio;
import org.redhat.demo.crazytrain.mqtt.MqttPublisher;
import org.redhat.demo.crazytrain.util.Util;

import io.quarkus.runtime.StartupEvent;
import io.quarkus.scheduler.Scheduled;
import io.vertx.mutiny.core.Vertx;


/**
 * ScheduledCapture is a service that captures images from a camera using the OpenCV library
 */

@ApplicationScoped
@Path("/capture")
public class ScheduledCapture {
    private  VideoCapture camera; 

    @Inject
    ImageCaptureService imageCaptureService;

    @Inject
    ImageService imageService;

    @Inject
    Vertx vertx;
    // interval in milliseconds
    @ConfigProperty(name = "capture.interval")
    int interval;
    // tmpFolder is the folder where the images are saved
    @ConfigProperty(name = "capture.tmpFolder") 
    String tmpFolder;
    // broker is the MQTT broker
    @ConfigProperty(name = "capture.brokerMqtt")
    String broker;
    // topic is the MQTT topic
    @ConfigProperty(name = "capture.topic")
    String topic;
    // nbImgSec is the number of images captured every second
    @ConfigProperty(name = "capture.periodicCapture")
    int periodicCapture;

    @ConfigProperty(name = "capture.saveImage")
    boolean saveImage;

    @ConfigProperty(name = "capture.videoDeviceIndex")
    int videoDeviceIndex;

    @ConfigProperty(name = "capture.mock")
    boolean mock;

    @ConfigProperty(name = "capture.videoPath")
    String videoPath;

    @ConfigProperty(name = "capture.videoPeriodicCapture")
    int videoPeriodicCapture;

    MqttPublisher mqttPublisher = null;

    private Long timerId;

    private volatile boolean stopRequested = false;
    private Thread testThread;

    private static final Logger LOGGER = Logger.getLogger(ScheduledCapture.class);
    Util util = null;
    // Start the camera when the application starts and set the resolution
    void onStart(@Observes StartupEvent ev) {
        Logger.getLogger(ScheduledCapture.class).info("The application is starting...");
        if(!mock){
            camera = new VideoCapture(videoDeviceIndex); 
            camera.set(Videoio.CAP_PROP_FRAME_WIDTH, 640); // Max resolution for Logitech C505
            camera.set(Videoio.CAP_PROP_FRAME_HEIGHT, 480); // Max resolution for Logitech C505
            //camera.set(Videoio.CAP_PROP_AUTOFOCUS, 0); // Try to disable autofocus
            //camera.set(Videoio.CAP_PROP_FOCUS, 255); // Try to disable autofocus
            camera.set(Videoio.CAP_PROP_EXPOSURE, 15); // Try to set exposure
        }
        util = new Util();
        mqttPublisher = new MqttPublisher(broker.trim(), topic.trim());

    }

    void readVideo(String videoPath) {        
        VideoCapture capture = new VideoCapture(videoPath);
        if (!capture.isOpened()) {
            throw new IllegalArgumentException("Video file not found at " + videoPath);
        }
        double fps = capture.get(Videoio.CAP_PROP_FPS);
        int frameSkip = (int) (fps/8);
        int count = 0;
        Mat frame = new Mat();
        while (!stopRequested) { // Continue reading the video until a stop request is received
            while (capture.read(frame)) {
                if (count % frameSkip == 0) {
                    // Publish the image to the MQTT broker
                    long timestamp = System.currentTimeMillis();
                    if(util != null) {
                        long start2 = System.nanoTime();
                        String jsonMessage = util.matToJson(frame, timestamp);
                        long end2 = System.nanoTime();
                        LOGGER.debugf("Time to convert image to json: %d ms", (end2 - start2) / 1000000);
                        LOGGER.debugf("JSON Message with id %s", jsonMessage);
                        try {
                            long start3 = System.nanoTime();
                            mqttPublisher.publish(jsonMessage);
                            long end3 = System.nanoTime();
                            LOGGER.debugf("Time to publish image: %d ms", (end3 - start3) / 1000000);
                            LOGGER.debugf("Message with id %s published to topic: %s", timestamp, topic);
                        } catch (MqttException e) {
                            e.printStackTrace();
                        }
                    }
                    if(saveImage){
                        String filepath = tmpFolder+"/" + timestamp + ".jpg";
                        imageService.saveImageAsync(frame, filepath).thenAccept(success -> {
                                if (success) {
                                    LOGGER.debug("Frame saved successfully");
                                } else {
                                    LOGGER.error("Failed to save frame");
                                }
                            });
                    }
                }
                count++;
                if (stopRequested) { // Check if stop has been requested inside the inner loop as well
                    break;
                }
            }
            capture.set(Videoio.CAP_PROP_POS_FRAMES, 0); // Reset the video to the first frame
        }
        capture.release();
    }

    // Capture and save a defined number of images every second
    void captureAndSaveImage() {
        LOGGER.debugf("The Thread name is %s" + Thread.currentThread().getName());
            // Capture the image
            long start = System.nanoTime();
            Mat image = imageCaptureService.captureImage(this.camera);
            long end = System.nanoTime();
            LOGGER.debugf("Time to capture image: %d ms", (end - start) / 1000000);
            // Publish the image to the MQTT broker
            long timestamp = System.currentTimeMillis();
            if(util != null) {
                long start2 = System.nanoTime();
                String jsonMessage = util.matToJson(image, timestamp);
                long end2 = System.nanoTime();
                LOGGER.debugf("Time to convert image to json: %d ms", (end2 - start2) / 1000000);
                LOGGER.debugf("JSON Message with id %s", jsonMessage);
                try {
                    long start3 = System.nanoTime();
                    mqttPublisher.publish(jsonMessage);
                       // Check if stop has been requested
                    if (stopRequested) {
                        // Stop capture and release camera
                        vertx.cancelTimer(timerId);
                        timerId = null;
                        imageCaptureService.releaseCamera(this.camera);
                        mqttPublisher.disconnect();
                        LOGGER.info("Capture stopped");
                        return;
                    }
                    long end3 = System.nanoTime();
                    LOGGER.debugf("Time to publish image: %d ms", (end3 - start3) / 1000000);
                    LOGGER.debugf("Message with id %s published to topic: %s", timestamp, topic);
                } catch (MqttException e) {
                    e.printStackTrace();
                }
            }
            // Save the image to the file system (asynchronously)
            if(saveImage){
                String filepath = tmpFolder+"/" + timestamp + ".jpg";
                imageService.saveImageAsync(image, filepath).thenAccept(success -> {
                        if (success) {
                            LOGGER.debug("Image saved successfully");
                        } else {
                            LOGGER.error("Failed to save image");
                        }
                    });
            }
    }
    @POST
    @Path("/start")
    public Response start() {
        LOGGER.info("Capture started");
        stopRequested = false;
        mqttPublisher.connect();
        //captureEnabled = true;
        if (timerId != null) {
            return Response.status(Response.Status.BAD_REQUEST).entity("Capture is already running").build();
        }
        timerId = vertx.setPeriodic(periodicCapture, id -> captureAndSaveImage());
        return Response.ok("Capture started").build();
    }

    @POST
    @Path("/test")
    public Response test() {
        LOGGER.info("Test started");
        stopRequested = false;
        mqttPublisher.connect();
        if (timerId != null) {
            return Response.status(Response.Status.BAD_REQUEST).entity("Capture is already running").build();
        }
        testThread = new Thread(() -> readVideo(videoPath));
        testThread.start();
        return Response.ok("read video from file started").build();
    }

    @POST
    @Path("/stop")
    public Response stop() {
        stopRequested = true;
        LOGGER.info("Stop requested");
        if (testThread != null) {
            try {
                testThread.join(); // Wait for the testThread to finish
            } catch (InterruptedException e) {
                Thread.currentThread().interrupt(); // Restore interrupted status
            }
            testThread = null;
        }
        return Response.ok("Stop requested").build();       
  
    }
}
```

Le fichier application.properties
```properties
%dev.quarkus.http.port=8082
%dev.capture.mock=true
%dev.catpure.videoPath=/Users/mouchan/projects/Demo-AI-Edge-Crazy-Train/train-capture-image-app/capture-app/src/main/resources/track7.mp4
%dev.catpure.videoPeriodicCapture=30
quarkus.kafka.devservices.enabled=false
quarkus.swagger-ui.always-include=true
#quarkus.native.container-build=true
#quarkus.native.native-image-xmx=16g
capture.videoDeviceIndex=${VIDE0_DEVICE_INDEX:0}
capture.dropbox.token=${DROPBOX_TOKEN:null}
capture.tmpFolder=${TMP_FOLDER:/Users/mouchan/crazy-train-images}
capture.interval=${INTERVAL:100}
capture.periodicCapture=${PERIODIC_CAPTURE:30}
capture.brokerMqtt=${MQTT_BROKER:tcp://localhost:1883}
capture.topic=${MQTT_TOPIC:train-image}
capture.videoPath=${VIDEO_PATH:/Users/mouchan/projects/Demo-AI-Edge-Crazy-Train/train-capture-image-app/capture-app/src/main/resources/track-stop.mp4}
capture.videoPeriodicCapture=${VIDEO_PERIODIC_CAPTURE:30}
capture.saveImage=${SAVE_IMAGE:false}
capture.mock=${MOCK:false}
quarkus.log.level=${LOGGER_LEVEL:INFO}
```

5. Compilation du projet 
Avant de committer vos modifications, vous devez construire le projet  pour vous assurer qu'il n'y a pas d'erreurs de compilation.


Ouvrez un terminal à la racine du projet train-capture-app.

```
./mvnw clean package
```

6. Démarrer le service en mode Dev
```
./mvnw quarkus:dev
```


### 2. Modification du micorservice intelligent-train

### 3. Modification du micorservice Train-ceq-app

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


### 4. Modification du micorservice train-controller


### 5. Modification des propriétés et du code du projet train-monitoring-app

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

### 4. Démarrer tous les services

Pour lancer les services :

1. Ouvrez le terminal intégré. Vous pouvez le faire en allant dans le menu "View" et en sélectionnant "Terminal", ou en utilisant le raccourci clavier correspondant.

2. Dans le terminal, vous pouvez lancer les commandes définies dans le devfile en utilisant la commande `dev` suivie de l'ID de la commande. Par exemple, pour lancer la commande `start-all-apps`, vous pouvez utiliser la commande suivante :

```bash
dev start-all-apps
```

4. De même, pour arrêter toutes les applications, vous pouvez utiliser la commande suivante :

```bash
dev stop-all-apps
```

Ces commandes lanceront ou arrêteront toutes les applications définies dans le devfile. Assurez-vous que vous êtes dans le bon espace de travail et que vous avez les permissions nécessaires pour exécuter ces commandes.

### 5. Test de l'application

Maintenant que vous avez modifié plusieurs parties de l'application, il est temps de tester l'application dans son ensemble.

1. lancer la commande suivante
```
curl -X 'POST'   'http://localhost:8082/capture/test'   -H 'accept: */*'
```

2. a partir de votre browser visualiser le résultat en utilisant ce lien http://localhost:8086/

## Conclusion

Félicitations, vous avez terminé le lab ! Vous devriez maintenant avoir une meilleure compréhension de l'application Train et de son architecture basée sur les microservices. Continuez à explorer et à modifier le code pour en apprendre encore plus.