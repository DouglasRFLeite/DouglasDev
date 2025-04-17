---
title: "Tutorial - Spring Boot & Apache Kafka"
date: 2025-04-07
weight: 992
tags: ["Java", "Spring", "Spring Boot", "Apache Kafka", "Event Streaming", "Producer", "Consumer"]
author: "Douglas"
showToc: true
TocOpen: false
draft: false
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
    // ...
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

### 2.6 - Inspecting via Console (Optional)

This is another optional step that will only work if you have followed the last (or, if you are doing all of this in an existing project, it should work on your topics as well).

As we don't yet have developed our Consumer, we need another way to verify if our Producer is... you know, producing.

Kafka, when built, makes some shellcript files available for us that will help us do that.

If you've built Kafka like I have, you can access them this way, via your terminal / console.

```
docker ps # this will let you find the container id where Kafka is running
docker exec -t <CONTAINER-ID> bash # fill in your container id
# now you should be inside the container environment
cd /opt/kafka/bin
./kafka-topics.sh --bootstrap-server localhost:9092 --list # this should list the topics
/kafka-console-consumer.sh --bootstrap-server localhost:9092 --topic <TOPIC-NAME> --from-beginning
```

It may take a couple of seconds, but this last command should let you see every message that was produced and sent on that topic.

## Step 3 - Creating your Consumer Application

Now we have a functional Producer Application, we should consume the data from the Topic via Java as well.

### 3.1 - Create a Project with Spring Initializr

This should be done the exact same way as with the Producer. Reference [2.1 - Create a Project with Spring Initializr](#21---create-a-project-with-spring-initializr).

### 3.2 - Creating your Message

We also have to define the format of our message via POJO, so reference [2.2 - Creating your Message](#22---creating-your-message).

### 3.3 - Configuring for Apache Kafka

Here we are finally doing something different.

First of all, you should add some configurations to your properties file. They are similar to the ones on the producer, but different.

```
spring.application.name=consumer
server.port=8082

spring.kafka.bootstrap-servers=localhost:9092
```

We used the Producer to configure the Topic, so we don not have to do that again. We just need to configure how we are going to read from the Topic we are producing on. We will do that on the `KafkaConsumerConfig.java` file.

```
@EnableKafka
@Configuration
public class KafkaConsumerConfig {

  @Getter
  private static final String MESSAGE_TOPIC = "douglas-messages";

  @Value(value = "${spring.kafka.bootstrap-servers}")
  private String serverAddress;

  @Bean
  public ConsumerFactory<String, MessageModel> messageConsumerFactory() {
    Map<String, Object> configProps = new HashMap<>();
    // ...
    return new DefaultKafkaConsumerFactory<>(configProps);
  }

  @Bean
  public ConcurrentKafkaListenerContainerFactory<String, MessageModel> messageKafkaListenerContainerFactory() {
    ConcurrentKafkaListenerContainerFactory<String, MessageModel> listenerFactory = new ConcurrentKafkaListenerContainerFactory<>();
    listenerFactory.setConsumerFactory(messageConsumerFactory());
    return listenerFactory;
  }
}
```

You can see the structure is very similar to the one on the Producer. We also have the server from the properties and the topic name on a variable to avoid magic Strings. We have a configuration method we are looking closer into next. And, finally, we have the `ConcurrentKafkaListenerContainerFactoryCreatorOfEverythingQueenOfDragonsKhaleesi` thing.

Yeah, huge name.

But what is does is mostly generating the Consumer version of the Kafka Template. The usage is a bit different, as we'll see shortly. Now, to the configuration:

```
@Bean
  public ConsumerFactory<String, MessageModel> messageConsumerFactory() {
    Map<String, Object> configProps = new HashMap<>();

    configProps.put(ConsumerConfig.BOOTSTRAP_SERVERS_CONFIG, serverAddress); // setting the Server
    configProps.put(ConsumerConfig.GROUP_ID_CONFIG, MESSAGE_TOPIC); // setting the Topic

    // The next two configurations set an Error Handling layer on the consumer
    configProps.put(ConsumerConfig.KEY_DESERIALIZER_CLASS_CONFIG, ErrorHandlingDeserializer.class);
    configProps.put(ConsumerConfig.VALUE_DESERIALIZER_CLASS_CONFIG, ErrorHandlingDeserializer.class);

    // We need to deserialize the result of the error handler when there's no error
    configProps.put(ErrorHandlingDeserializer.KEY_DESERIALIZER_CLASS, JsonDeserializer.class);
    configProps.put(ErrorHandlingDeserializer.VALUE_DESERIALIZER_CLASS, JsonDeserializer.class);

    configProps.put(JsonDeserializer.TRUSTED_PACKAGES, "*"); // this removes a security layer we don't need
    configProps.put(JsonDeserializer.VALUE_DEFAULT_TYPE, MessageModel.class.getName()); // setting the model

    return new DefaultKafkaConsumerFactory<>(configProps);
  }
```

There's some more configuration here, but I believe the comments do justice.

### 3.4 - Listening for Messages

As we did for the Producer, the Consumer will have a `MessageService.java` file to listen for messages on that topic and act when there is a new one.

```
@Service
public class MessageService {

  @KafkaListener(topics = "douglas-messages", containerFactory = "messageKafkaListenerContainerFactory")
  public void messageListener(MessageModel msg) {
    System.out.println("Message Received by Consumer: " + msg.toString());
  }
}
```

You have probably noticed a big difference between the Producer and the Consumer now. But, overall, both codes are preetty clean and easy to understand.

## Step 4 - Making it Happen

Now we are finally making the cool stuff happen.

If you've followed everything closely so far, you should be able to run both applications and make them work together.

On this tutorial, we are doing that with a Postman POST request sent to the Producer. This is the result:

```
#This is what I sent via Postman
{
    "name": "Producer",
    "time": null,
    "message": "Hello, Consumer!"
}

# This shows on the Producers Console
Sending message on Kafka Topic: MessageModel(name=Producer, time=2025-04-08T14:12:08.338257923, message=Hello, Consumer!)

# This shows on the Consumers Console
Message Received by Consumer: MessageModel(name=Producer, time=2025-04-08T14:12:08.338257923, message=Hello, Consumer!)
```
