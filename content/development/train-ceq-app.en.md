+++
title = "Service train-ceq-app"
draft = false
weight = 5
+++

The **train-ceq-app** is an application based on Apache Camel, a Java library for implementing application integrations using Enterprise Integration Patterns (EIP). 
This application is mainly composed of Camel routes defined in the **PostProcessingRoute.xml** file. These routes define how messages are consumed, transformed and forwarded to other services or destinations.

The **postproscessing-route** performs the following operations:

1. **Message consumption**: The route first consumes messages from the MQTT broker using the **paho:{{train.mqtt.source.topicName}}?brokerUrl={{train.mqtt.brokerUrl}}** URI. The messages consumed are then recorded in the log.

2. **Saving the initial message**: The initial message is saved in the message header under the name "origin" so that it can be retrieved later.

3. **Extract Message ID**: The message ID is extracted using the JSONPath expression **$.id** and stored in the message header.

4. **Message deserialization**: The message is deserialized into a Java object of type **org.redhat.demo.crazytrain.model.Result** using the Jackson library.

5. **Message processing**: The message is then processed by the **CommandProcessor**.

6. **Publish the message**: The processed message is published on another MQTT topic using the URI **paho:{{command.mqtt.destination.topicName}}?brokerUrl={{train.mqtt.brokerUrl}}**.

7. **Retrieving the initial message**: The initial message saved in the header is retrieved and put back in the message body.

8. **Cloud Event Generation**: A cloud event is generated from the initial message and processed by the **CloudEventProcessor**.

9. **Cloud Event Publication**: The cloud event is published on a Kafka topic using the URI **kafka:{{monitoring.kafka.destination.topicName}}?brokers={{train.kafka.brokerUrl}}**.

The **command-capture-image** route works in a similar way, but consumes messages from a Kafka topic, extracts a command from the message, and sends an HTTP POST request to a specified URL with the command as a parameter.



In a camel route the data is continuously transformed by various actions. Sometimes it is necessary to perform a check on the original message and not on the transformed message. It is good practice to save the original message so that it can be retrieved later. With Camel this is done via the properties of the message header. 


In the **train-ceq-app** project, you are going to modify the **PostProcessingRoute.xml** file to save the initial message so that you can retrieve it later.

1. Open the file `src/main/resources/PostProcessingRoute.xml`.
2. Add the following instruction just after the line `<log loggingLevel="DEBUG" message="${body}"/>` :

```xml
<setHeader name="origin"><simple>${body}</simple></setHeader>
```

This instruction saves the initial message in the message header under the name "origin".

3. Next, add the following statement just after the line `<toD uri="paho:{{command.mqtt.destination.topicName}}?brokerUrl={{train.mqtt.brokerUrl}}"/>` :

```xml
<setBody><simple>${header.origin}</simple></setBody>
```

This instruction retrieves the original message saved in the header and puts it back in the message body.

4. Save your changes.

Now the `src/main/resources/PostProcessingRoute.xml` route saves the original message and retrieves it later. This allows you to check the original message and not the transformed message.

This is what the route should look like after your changes:

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
        <log loggingLevel="DEBUG" message="${body}"/>
        <toD uri="kafka:{{monitoring.kafka.destination.topicName}}?brokers={{train.kafka.brokerUrl}}"/>
        <log loggingLevel="DEBUG" message="written into kafka"/>
    </route>
    <route id="command-capture-image">
        <from uri="kafka:{{monitoring.kafka.source.topicName}}?brokers={{train.kafka.brokerUrl}}"/>
        <setHeader name="command"><jsonpath>$.command</jsonpath></setHeader>
        <toD uri="{{train.http.url}}/${header.command}?httpMethod=POST" />
    </route>
```

5. Compiling the project

Before committing your changes, you need to build the project to ensure that there are no compilation errors.

- Open a new terminal

![terminal](/images/dev-section/new-terminal-bash.png)

- Run the command below 

```
./mvnw clean package
```
