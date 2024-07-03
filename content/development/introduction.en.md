+++
title = "Introduction"
draft= false
weight= 2
+++

The **Crazy Train** application is made up of several microservices. The image below describes the overall operation

![architecture](/images/dev-section/architecture.png)

Below is a description of each service:

**train-capture-image-app**: This is a Quarkus service that allows you to start, test and stop video capture. It exposes RESTful endpoints that can be called by other services or clients to control video capture.

**intelligent-train**: This service is responsible for analysing the data from the captured video. It uses machine learning techniques to interpret the video data and provide information based on this analysis.

**train-ceq-app**: is an event management service that receives information from Intelligent Train and other services, and triggers appropriate actions. For example, if the Intelligent Train detects an obstacle on the track, the service triggers an event to stop the train.

**train-controller**: As its name suggests, this service is probably responsible for controlling the train itself. It would receive commands from services such as train-ceq and perform actions on the train, such as starting, stopping, changing speed, etc.

**train-monitoring-app**: This service is responsible for monitoring the entire system. It would collect data from all the other services, such as events triggered, actions performed, train status, etc., and provide an overview of the state of the system. It could also provide alerts or notifications in the event of problems being detected.

Each service is independent and communicates with the others asynchronously (MQTT/Kafka). This allows great flexibility and scalability, as each service can be developed, deployed and scaled independently of the others.