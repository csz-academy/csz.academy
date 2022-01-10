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
configuration and out-of-the-box deserializing in a type-safe matter. 

We are going to build a testable skeleton with Kafka Listeners, try deserializing messages, accessing 
headers and I will show you some common pitfalls. The final code can be found in my [spring-dojo repository on GitHub](https://github.com/tobi6112/spring-dojo/tree/main/spring-boot-kafka-example).

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

And that's our skeleton for the next steps. 

