---
layout: post
title:  "Using Apache Kafka in Spring Boot (The easy Way!)"
date:   "2022-01-10"
tags: [ "spring", "spring boot", "java", "kafka", "apache kafka" ]
authors: [ "hl" ]
---

Recently I tried to integrate [Apache Kafka](https://kafka.apache.org/) into a Spring Boot application.
However, nearly every tutorial and guide I found on the web felt like a cumbersome way in contrast
to the usual magic in Spring Boot.

After several hours of trial and error I have figured out how to leverage the Spring Boot auto 
configuration and out-of-the-box deserializing in a type-safe manner. 

We are going to build a testable skeleton with Kafka Listeners, try deserializing messages, accessing 
headers and I will show you some common pitfalls. The final code can be found in my [java-dojo repository on GitHub](https://github.com/tobi6112/java-dojo/tree/main/spring-boot-kafka-example).

# Prerequisites

We will use Java 17 with Spring Boot 2.6.2 as these are the latest versions at the time of writing with 
the following dependencies:
```xml
<dependencies>
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter</artifactId>
    </dependency>
    <dependency>
        <groupId>org.springframework.kafka</groupId>
        <artifactId>spring-kafka</artifactId>
    </dependency>
    <dependency>
        <groupId>com.fasterxml.jackson.core</groupId>
        <artifactId>jackson-annotations</artifactId>
    </dependency>

    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-test</artifactId>
        <scope>test</scope>
        <exclusions>
            <exclusion>
                <groupId>org.junit.vintage</groupId>
                <artifactId>junit-vintage-engine</artifactId>
            </exclusion>
        </exclusions>
    </dependency>
    <dependency>
        <groupId>org.springframework.kafka</groupId>
        <artifactId>spring-kafka-test</artifactId>
        <scope>test</scope>
    </dependency>
    <dependency>
        <groupId>org.awaitility</groupId>
        <artifactId>awaitility</artifactId>
        <version>4.1.1</version>
        <scope>test</scope>
    </dependency>
</dependencies>
```

Also we will use a Kafka Broker. For the sake of simplicity we will use [Vectorized Redpanda](https://vectorized.io/redpanda/),
which is a Kafka-compatible streaming platform written in C++. However, you could also use Apache Kafka, this does not really
matter, since we're just interacting with the API and will not touch the broker instance itself.

Redpanda can be started from a Docker Compose file:
```yml
version: '3.7'
services:
  redpanda:
    command:
        - redpanda
        - start
        - --smp
        - '1'
        - --reserve-memory
        - 0M
        - --overprovisioned
        - --node-id
        - '0'
        - --kafka-addr
        - PLAINTEXT://0.0.0.0:29092,OUTSIDE://0.0.0.0:9092
        - --advertise-kafka-addr
        - PLAINTEXT://redpanda:29092,OUTSIDE://localhost:9092
    image: docker.vectorized.io/vectorized/redpanda:v21.11.2
    container_name: redpanda
    ports:
        - "9092:9092"
        - "29092:29092"
```

# Skeleton

At first we will apply the very basic configuration, and apply some structure that is required to launch and test the application.

The following configuration applies:
```properties
spring.kafka.bootstrap-servers=localhost:9092
spring.kafka.consumer.group-id=group1
```

Next up we will define the messages we are going to listen for. 
The first messages is going to be published into the `user-id` topic and will contain UUIDs as plain string value.
The second message will be published into the `user` topic and will contain a user object as a JSON Object. In order to deserialize 
it into a Java object, we will define a Java record, which conformes to the JSON Schema.
```java
public record User(
    @JsonProperty("name") String name,
    @JsonProperty("birthday") LocalDate birthday,
    @JsonProperty("hobbies") Set<String> hobbies
) {}
```

Next up, we will introduce the corresponding Kafka Consumers by defining listeners for each topic. These listeners will simply log the
received message and put the message content into a Set.
```java
@Component
public class KafkaListeners {
  private final static Logger log = LoggerFactory.getLogger(KafkaListeners.class);
  
  public Set<String> userIds = new HashSet<>();
  public Set<User> users = new HashSet<>();

  @KafkaListener(topics = "user-id")
  public void userIdListener(ConsumerRecord<String, String> record) {
    log.info("Received a Message in user-id topic: {}", record);
    this.userIds.add(record.value());
  }

  @KafkaListener(topics = "user")
  public void userListener(ConsumerRecord<String, User> record) {
    log.info("Received a Message in user topic: {}", record);
    this.users.add(record.value());
  }
}
```

Last but not least we will define some test cases to automaticaly verify that our application is working. General speaking we are going to
publish a message and checking the lists in the consumer whether it contains the published message.
```java
@SpringBootTest
class KafkaListenersTest {

  @Autowired
  KafkaListeners kafkaListeners;

  @Autowired
  KafkaTemplate<Object, Object> kafkaTemplate;

  @BeforeEach
  void setUp() {
    kafkaListeners.users.clear();
    kafkaListeners.userIds.clear();
  }

  @Test
  void userIdListener() {
    // Given
    var userId = UUID.randomUUID().toString();

    // When
    kafkaTemplate.send("user-id", userId);

    // Then
    await().atMost(15, TimeUnit.SECONDS).until(() -> kafkaListeners.userIds.size() > 0);
    assertThat(kafkaListeners.userIds).containsExactlyInAnyOrder(userId);
  }

  @Test
  void userListener() {
    // Given
    var user = """
        {
          "name": "Carl",
          "birthday": "2000-01-01",
          "hobbies": [
            "football",
            "coding"
          ]
        }
        """;

    // When
    kafkaTemplate.send("user", user);

    // Then
    var expectedUser = new User("Carl",
        LocalDate.of(2000, 1, 1),
        Set.of("football", "coding"));

    await().atMost(15, TimeUnit.SECONDS).until(() -> kafkaListeners.users.size() > 0);
    assertThat(kafkaListeners.users).containsExactlyInAnyOrder(expectedUser);
  }
}
```

Running the tests will give you the following Output:
```
[ERROR] Tests run: 2, Failures: 0, Errors: 1, Skipped: 0
```
Wow - one test is already passing and we didn't do anything - so it shouldn't be so hard to make the second one pass... Right?

However, this will be our skeleton for the next steps. 

# Leveraging Spring Boot Autoconfig

... TODO

# Apply JSON Deserialization

## The Cumbersome Way

Looking at the output from the failing test, we see that the JSON Deserializion is not working properly. The Exception message is:
```
java.lang.ClassCastException: class java.lang.String cannot be cast to class comgithub.tobi6112.springbootkafkaexample.User
```

Well, this should be an easy fix. Just add a JSON Deserializer that is shipped already by Spring Kafka as stated in the docs. Just add the following configuration:

```properties
spring.kafka.consumer.value-deserializer=org.springframework.kafka.support.serializer.JsonDeserializer
```

Running the failing test again, we see it is still failing, but with another exception - so our configuration had some inpact - Yay! This time the Exception is:

```
Caused by: java.lang.IllegalStateException: No type information in headers and no default type provided
```

Hint: You might have noticed that your listener is trapped inside a endless loop and will propably flood your stdout. I will adress that later on in [Error Handling](#error-handling).

What this exeception is telling you is, that the provided JsonDeserializer does not know the type to deserialize to and is using the Kafka. The JsonDeserializer inspects the record Headers and searches for a `__TypeId__` Header in which a type identifier is located, and will use this header to determine the deserialization type by resolving it from a mapping configration.

So in order to resolve this exception we will need to add the type header to the produced record - and therefore need to modifiy the test case. We will choose `user` as the type identifier.
The test will no look like the following
```java
@Test
void userListener() {
// Given
var user = """
    {
        "name": "Carl",
        "birthday": "2000-01-01",
        "hobbies": [
        "football",
        "coding"
        ]
    }
    """;
var recordToSend = new ProducerRecord<Object, Object>("user", user);
recordToSend.headers().add("__TypeId__", "user".getBytes());

// When
kafkaTemplate.send(recordToSend);

// Then
var expectedUser = new User("Carl",
    LocalDate.of(2000, 1, 1),
    Set.of("football", "coding"));

await().atMost(15, TimeUnit.SECONDS).until(() -> kafkaListeners.users.size() > 0);
assertThat(kafkaListeners.users).containsExactlyInAnyOrder(expectedUser);
}
```

Next we need to define the type mapping to tell the JSONDeserializer in which type it needs to be deserialized. This can be done by adding the following configuration:
```properties
spring.kafka.consumer.properties.[spring.json.type.mapping]=user:com.github.tobi6112.springbootkafkaexample.User
```

If you run the test case again, you will see it will succeed. Nice. That was kinda easy. Let's run the other test again, to verify it too. Uh... Oh... It fails. We had the exception already, just add a type header and a type Mapping to String and it should work. We choose `user-id` as type identifier and map it to `java.lang.String`. That should work now... Well: it doesn't.
```
Caused by: com.fasterxml.jackson.core.JsonParseException: Unexpected character ('b' (code 98)) in numeric value: Exponent indicator not followed by a digit
```
Remember that we're publishing plain string values to the topic, and we are now trying to deserialize it with a JsonDeserializer which expects a JSON value. So we need to tell the kafka listener for the topic `user-id` to use a StringDeserializer. We can do this by two ways. Either we provide a new [KafkaListenerContainerFactory](https://docs.spring.io/spring-kafka/api/org/springframework/kafka/config/KafkaListenerContainerFactory.html) Bean and refernce its qualifier in the `@KafkaListener` annotation. 
The over possiblity we have is directly setting these properties in the `@KafkaListener` annotation.

### Using the ConsumerFactory

First you need to keep in mind, that you will need to define tw ContainerFactories. One for the JSON Deserialization and one for the String deserialzion. We will define the ContainerFactory for JSON Deserialization as the Primary bean so it will be automatically picked up by Spring Boot when instantiating a new KafkaListener.
```java
@Bean(name = "stringContainerFactory")
public ConcurrentKafkaListenerContainerFactory<Object, Object> kafkaStringConsumerFactory(
    ConcurrentKafkaListenerContainerFactoryConfigurer configurer,
    ObjectProvider<ConsumerFactory<Object, Object>> kafkaConsumerFactory,
    KafkaProperties properties
) {
    ConcurrentKafkaListenerContainerFactory<Object, Object> factory = new ConcurrentKafkaListenerContainerFactory<>();
    configurer.configure(factory, kafkaConsumerFactory
        .getIfAvailable(() -> {
            var consumerFactory = new DefaultKafkaConsumerFactory<>(properties.buildConsumerProperties());

            consumerFactory.setValueDeserializer(new ParseStringDeserializer<>());

            return consumerFactory;
        }));
    return factory;
}

@Primary
@Bean(name = "defaultContainerFactory")
public ConcurrentKafkaListenerContainerFactory<Object, Object> kafkaDefaultConsumerFactory(
    ConcurrentKafkaListenerContainerFactoryConfigurer configurer,
    ObjectProvider<ConsumerFactory<Object, Object>> kafkaConsumerFactory,
    KafkaProperties properties
) {
    ConcurrentKafkaListenerContainerFactory<Object, Object> factory = new ConcurrentKafkaListenerContainerFactory<>();
    configurer.configure(factory, kafkaConsumerFactory
        .getIfAvailable(() -> new DefaultKafkaConsumerFactory<>(properties.buildConsumerProperties())));
    return factory;
}
```

And then configure the KafkaListener to use the provided ContainerFactory.
```java
@KafkaListener(topics = "user-id", containerFactory = "stringContainerFactory") 
public void userIdListener(ConsumerRecord<String, String> record) {
    log.info("Received a Message in user-id topic: {}", record);
    this.userIds.add(record.value());
}
```

At least now you should be questioning this approach, there must be a better way.

### Using Properties in Annotation

We will just add the property to the annotation. Then this consumer will use the StringDeserializer to deserialize the message content. 
```java
@KafkaListener(topics = "user-id", properties = {
    "spring.deserializer.value.delegate.class=org.apache.kafka.common.serialization.StringDeserializer"
}) 
public void userIdListener(ConsumerRecord<String, String> record) {
    log.info("Received a Message in user-id topic: {}", record);
    this.userIds.add(record.value());
}
```

Well, know both tests are succeeding. 

## The Smart Way

I hope you didn't quit after reading the previous paragraph. If you are familar with Spring Boot you should notice, that there was a lot of manual work required to do a rather simple task. What if I tell you, that you can reduce the amount of code an configuration to a bare minimum and let Spring Boot do all the work for you?


// TODO


# Kafka Record vs. Message

// TODO: Emphasize the issues with kafka records, like type sneaking and missing validation

# Error Handling

I won't deep-dive into this topic here, instead I want to encourage you to take a look at the excellent Blog Post by [Confluent](https://www.confluent.io/de-de/blog/spring-kafka-can-your-kafka-consumers-handle-a-poison-pill/) 
which explains the difficulties of handling Exceptions within Kafka Consumers and also looking 
at the Sprinng Documentation [Using ErrorHandlingDeserializer](https://docs.spring.io/spring-kafka/reference/html/#error-handling-deserializer).

However, the following configuration applies for this example project focussing on deserialization 
issues and no DLT's.
```properties
spring.kafka.consumer.key-deserializer=org.springframework.kafka.support.serializer.ErrorHandlingDeserializer
spring.kafka.consumer.value-deserializer=org.springframework.kafka.support.serializer.ErrorHandlingDeserializer
spring.kafka.consumer.properties.[spring.deserializer.value.delegate.class]=org.springframework.kafka.support.serializer.JsonDeserializer
spring.kafka.consumer.properties.[spring.deserializer.key.delegate.class]=org.apache.kafka.common.serialization.StringDeserializer
```

Also we will define an Error Handler Bean:
```java
@Bean
public CommonLoggingErrorHandler errorHandler() {
  return new CommonLoggingErrorHandler();
}
```