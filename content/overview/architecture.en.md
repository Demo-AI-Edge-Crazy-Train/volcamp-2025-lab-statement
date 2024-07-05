---
title: "Architecture"
weight: 2
---

## Architecture

![global architecture](/images/architecture-global.png)

The train is equipped with a motor and a Lego Hub. The Lego Hub receives acceleration, deceleration and braking commands via the Bluetooth Low Energy protocol.

We have integrated an Nvidia Jetson Orin card into the Lego train. The Nvidia Jetson Orin card is a System on Chip (SoC) that integrates all the components needed for our mission: CPU, RAM, storage and a powerful GPU to accelerate calculations. This card receives the video stream from the on-board camera and transmits orders to the Lego Hub via the Bluetooth protocol. It is powered by a portable battery for the duration of the mission.

We are operating in an Edge Computing environment. On the Nvidia Jetson card, we are installing Red Hat Device Edge, a variant of RHEL adapted to the constraints of Edge Computing. We deploy Microshift, Red Hat's version of Kubernetes designed for the Edge. We then deploy our microservices, an MQTT broker and the artificial intelligence model on Microshift, using an over-the-air mechanism.

For the duration of the mission, the Jetson is connected to an OpenShift cluster in the AWS cloud via a 5G connection. In the AWS cloud, we have an RHEL 9 virtual machine that enables us to build our RHEL images for the Jetson. Our video surveillance application runs in the OpenShift cluster, which allows us to remotely monitor the train's on-board camera. The video stream is relayed from the Jetson via a Kafka broker.

In addition, MLOps pipelines are being set up to train the artificial intelligence model, as well as CI/CD pipelines to build the container images of our microservices for x86 and ARM architectures.