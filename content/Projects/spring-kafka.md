---
title: "Tutorial - Spring Boot & Apache Kafka"
date: 2025-04-07
weight: 992
tags: ["Java", "Spring", "Spring Boot", "Apache Kafka", "Event Streaming", "Producer", "Consumer"]
author: "Douglas"
showToc: true
TocOpen: false
draft: true
hidemeta: false
comments: false
description: "A tutorial on how I created two Spring Boot applications that comunicate via Apache Kafka"
canonicalURL: "https://canonical.url/to/page"
disableHLJS: true # to disable highlightjs
disableShare: false
disableHLJS: false
hideSummary: false
searchHidden: true
ShowReadingTime: true
ShowBreadCrumbs: true
ShowPostNavLinks: true
ShowWordCount: true
ShowRssButtonInSectionTermList: true
UseHugoToc: true
cover:
    image: "images/cool-setup-002.jpeg" # image path/url
    alt: "Cool Setup" # alt text
    caption: "Unfortunately not my setup" # display caption under cover
    relative: false # when using page bundles set this to true
---

Yeah. If you've seen my last tutorial on AWS S3, you may be guessing that my wife is away for another weekend. But you guessed wrong.

This time, my Technical Manager had the awesome idea to include Apache Kafka in our tech stack. And I, as the Senior dev and Tech Lead, had the privilege do choose my task.

As I did not yet know how to use Apache Kafka, I obviously chose to learn it on paid time (yay).

## What and Why Apache Kafka?

## Step 1 - Getting an Apache Kafka Instance Up and Running

The [**Apache Kafka documentation webpage**](https://kafka.apache.org/documentation/) and even the [**Quickstart**](https://kafka.apache.org/quickstart) mention a couple of ways to download, install and run Kafka.

Me? I always just look for the word "Docker" in any quickstart guide and use it. Here is what I wrote on a `getAndRunKafka.sh` shellscript to make it work:

```
#!/bin/bash

docker pull apache/kafka:4.0.0

docker run -d -p 9092:9092 apache/kafka:4.0.0
```

And yeah, that's all I did to have an instance of Kafka running on my machine.

## Step 2 - Creating your Producer Application

We usually think of communication as a Client and Server relationship. With Event Streaming, we talk about Producers and Consumers. The relationship is a little different overall.

Producers, obviously, produce the messages and add them to the Topic. The Consumer will, again, obviously, consume the messages from that Topic.

Therefore, it would make absolutely no sense to start developing on the Consumer side.

### 2.1 - Create a Project with Spring Initializr

I use the Spring Initializr via the Spring Boot Extension Pack on VSCode. You can always do it via the web, create your project, and unzip it to start coding. But what you're going to do is:

1. Create a Maven project
2. Spring Boot Version 3.4.3
3. Java
4. Name it as you wish
5. I've used Java 17 and JAR packaging
6. You'll add dependencies: Spring for Apache Kafka, Spring Web (optional), Spring Boot Dev Tools (optional), Lombok (optional)

And you're good to go.

### 2.2 - Creating your Message

I believe the most important part of this tutorial is the Kafka configuration and usage, but I do need the message POJO ready to configure it. So, this will be our POJO:

```
@Data
public class MessageModel {
  private String name;
  private LocalDateTime time;
  private String message;
}
```

### 2.3 - Configuring for Apache Kafka

So, now we're adding some stuff to our configurations file, also know as `application.properties`:

```
spring.application.name=producer
server.port=8081

spring.kafka.bootstrap-servers=localhost:9092
```

The server port is here just to not conflict with the consumer (and with other applications in my machine hehe).

The most important thing is the address for the Kafka server.

Our next configuration file is a Java file I will name `KafkaProducerConfig.java`:

```
@Configuration
public class KafkaProducerConfig {

  @Value(value = "${spring.kafka.bootstrap-servers}")
  private String serverAddress;

  @Bean
  public ProducerFactory<String, MessageModel> messageProducerFactory() {
    Map<String, Object> configProps = new HashMap<>();

    return new DefaultKafkaProducerFactory<>(configProps);
  }

  @Bean
  public KafkaTemplate<String, MessageModel> messageKafkaTemplate() {
    return new KafkaTemplate<>(messageProducerFactory());
  }
}
```

This will be our baseline. Here we're fetching the server address from the properties file. We are also going to generate a configuration map for our message production. Finally, we are generating a KafkaTemplate using the factory we configured. That KafkaTemplate is what we'll use later to send the messages.

Our configuration method is currently empty, this is what we're adding to it:

```
@Bean
public ProducerFactory<String, MessageModel> messageProducerFactory() {
    Map<String, Object> configProps = new HashMap<>();
    configProps.put(JsonSerializer.ADD_TYPE_INFO_HEADERS, false); // this make communication more flexible
    configProps.put(ProducerConfig.BOOTSTRAP_SERVERS_CONFIG, serverAddress); // server address configuration
    configProps.put(ProducerConfig.KEY_SERIALIZER_CLASS_CONFIG, StringSerializer.class); // Key will be a String
    configProps.put(ProducerConfig.VALUE_SERIALIZER_CLASS_CONFIG, JsonSerializer.class); // Message will be an Object
    return new DefaultKafkaProducerFactory<>(configProps);
}
```

We still need one more file to configure our Topic, we'll call it `KafkaTopicConfig.java`:

```
@Configuration
public class KafkaTopicConfig {

  @Getter
  private static final String TOPIC_NAME = "douglas-messages";
  private static final int NUM_PARTITIONS = 1;
  private static final short REPLICATION_STRATEGY = 1; // no replication

  @Value(value = "${spring.kafka.bootstrap-servers}")
  private String serverAddress;

  @Bean
  public KafkaAdmin kafkaAdmin() {
    Map<String, Object> configs = new HashMap<>();
    configs.put(AdminClientConfig.BOOTSTRAP_SERVERS_CONFIG, serverAddress);
    return new KafkaAdmin(configs);
  }

  @Bean
  public NewTopic topicMessages() {
    return new NewTopic(TOPIC_NAME, NUM_PARTITIONS, REPLICATION_STRATEGY);
  }
}
```

Here we are also fetching the server address so we can create out Topic on the right Kafka instance. We are also creating the Topic with the name and configurations we want.

If you're still wondering who will create that topic, you need to understand Spring Framework a little better.

### 2.4 - Sending the Message

We have already configured how our application will communicate via Kafka, but not when it will do it.

For that, we will create a `MessageService.java` that will send our messages via our topic.

```
@Service
public class MessageService {

  @Autowired
  private KafkaTemplate<String, MessageModel> kafkaTemplate;

  public void sendMessage(MessageModel msg) {
    System.out.println("Sending message on Kafka Topic: " + msg.toString());
    msg.setTime(LocalDateTime.now());
    kafkaTemplate.send(KafkaTopicConfig.getTOPIC_NAME(), msg);
  }
}
```

This should be more than enough for us to send messages via Apache Kafka with Spring Boot. The next section is optional but will make easier for us to trigger the sending action.

### 2.5 - Triggering via POST (Optional)

If you're implementing this in a real project, this should do for your Producer. If you're following this tutorial with me, we need some way to trigger the sending of those messages.

I chose to do that with a Rest Controller. This is what our `MessageController.java` looks like:

```
@RestController
@RequestMapping("/message")
public class MessageController {

  @Autowired
  private MessageService service;

  @PostMapping("/")
  public void postMessage(@RequestBody MessageModel msg) {
    service.sendMessage(msg);
  }
}
```

Now, if I use Postman to send a Post Request to our endpoint with this data:

```
{
    "name": "Douglas",
    "time": null,
    "message": "Olá, mundo!"
}
```

This gets printed on the Terminal:

`Sending message on Kafka Topic: MessageModel(name=Douglas, time=2025-04-07T18:01:37.907616782, message=Olá, mundo!)`
