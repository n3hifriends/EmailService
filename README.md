
# **Email Notification with Kafka and Spring Boot**

This project demonstrates how to integrate **Kafka** for asynchronous communication between **Spring Boot** applications. The flow involves sending emails using an SMTP service triggered by Kafka messages, which are produced by a user authentication service.

### **Project Structure**

- **EmailService**: A Spring Boot microservice that sends emails using SMTP when it receives messages from Kafka.
- **UserAuthenticationService**: A Spring Boot microservice that produces Kafka messages, which **EmailService** listens to and processes.
- **Kafka**: Used as the message broker to facilitate communication between the two services.

---

### **Technologies Used**

- **Spring Boot**: Framework for building microservices.
- **Kafka**: Messaging system for asynchronous communication between services.
- **Spring Kafka**: Kafka integration for Spring Boot applications.
- **SMTP**: Simple Mail Transfer Protocol for sending emails.

---

### **Architecture Overview**

1. **EmailService** listens to a Kafka topic.
2. **UserAuthenticationService** produces messages to the Kafka topic when certain events (e.g., user registration) occur.
3. Upon receiving a message, **EmailService** sends an email using SMTP (via Spring Boot Mail integration).

---

### **EmailService Setup (Kafka Listener + SMTP)**

1. **Add Dependencies** in `pom.xml` for Kafka and SMTP:
   ```xml
   <dependency>
       <groupId>org.springframework.kafka</groupId>
       <artifactId>spring-kafka</artifactId>
   </dependency>
   <dependency>
       <groupId>org.springframework.boot</groupId>
       <artifactId>spring-boot-starter-mail</artifactId>
   </dependency>
   ```

2. **Kafka Listener Configuration**:
   ```java
   import org.springframework.kafka.annotation.EnableKafka;
   import org.springframework.kafka.annotation.KafkaListener;
   import org.springframework.stereotype.Service;

   @Service
   public class EmailService {

       private final JavaMailSender javaMailSender;

       public EmailService(JavaMailSender javaMailSender) {
           this.javaMailSender = javaMailSender;
       }

       @KafkaListener(topics = "signup", groupId = "EmailService")
       public void listen(String message) {
           // Assume the message contains user email and other details
           sendEmail(message);
       }

       public void sendEmail(String message) {
           SimpleMailMessage mailMessage = new SimpleMailMessage();
           mailMessage.setTo("recipient@example.com");
           mailMessage.setSubject("Welcome to Our Service!");
           mailMessage.setText("Hello! Welcome to our service. Your details: " + message);

           javaMailSender.send(mailMessage);
       }
   }
   ```

3. **Configure SMTP and Kafka Settings** in `application.properties`:

   ```properties
   # SMTP Settings
   spring.mail.host=smtp.gmail.com
   spring.mail.port=587
   spring.mail.username=your-email@gmail.com
   spring.mail.password=your-email-password
   spring.mail.protocol=smtp
   spring.mail.tls=true

   # Kafka Consumer Configuration
   spring.kafka.consumer.bootstrap-servers=localhost:9092
   spring.kafka.consumer.group-id=email-group
   spring.kafka.consumer.topic=user-registration
   ```

4. **Start EmailService**: Run the `EmailService` class. It will start listening to Kafka messages from the `user-registration` topic.

---

### **UserAuthenticationService Setup (Kafka Producer)**

1. **Add Dependencies** in `pom.xml` for Kafka:
   ```xml
   <dependency>
       <groupId>org.springframework.kafka</groupId>
       <artifactId>spring-kafka</artifactId>
   </dependency>
   ```

2. **Kafka Producer Configuration**:
   ```java
   import org.springframework.kafka.core.KafkaTemplate;
   import org.springframework.stereotype.Service;

   @Service
   public class UserRegistrationService {

       private final KafkaTemplate<String, String> kafkaTemplate;

       public UserRegistrationService(KafkaTemplate<String, String> kafkaTemplate) {
           this.kafkaTemplate = kafkaTemplate;
       }

       public void registerUser(String userDetails) {
           // Produce Kafka message on user registration
           kafkaTemplate.send("user-registration", userDetails);
       }
   }
   ```

3. **Configure Kafka Producer Settings** in `application.properties`:

   ```properties
   # Kafka Producer Configuration
   spring.kafka.producer.bootstrap-servers=localhost:9092
   spring.kafka.producer.key-serializer=org.apache.kafka.common.serialization.StringSerializer
   spring.kafka.producer.value-serializer=org.apache.kafka.common.serialization.StringSerializer
   ```

4. **Start UserAuthenticationService**: Run the `UserAuthenticationService` class. When a user registers, the service will produce a Kafka message to the `user-registration` topic.

---

### **Kafka Setup**

1. **Start Kafka Server**:
   - Download and extract the Kafka binaries from [Kafka official download page](https://kafka.apache.org/downloads).
   - Start **Zookeeper**:
     ```bash
     bin/zookeeper-server-start.sh config/zookeeper.properties
     ```
   - Start **Kafka Broker**:
     ```bash
     bin/kafka-server-start.sh config/server.properties
     ```

2. **Create Kafka Topic**:
   ```bash
   bin/kafka-topics.sh --create --topic user-registration --bootstrap-server localhost:9092 --partitions 1 --replication-factor 1
   ```

3. **Verify Kafka Topic**:
   ```bash
   bin/kafka-topics.sh --list --bootstrap-server localhost:9092
   ```

---

### **Service Flow**

1. **UserAuthenticationService** produces a message to the `user-registration` topic in Kafka after a user registers.
2. **EmailService** listens to this topic, receives the message, and processes it to send an email.

---

### **Running the Application**

1. **Start Kafka**:
   - Make sure you have Kafka and Zookeeper running on your machine.

2. **Start EmailService**:
   - Run the Spring Boot application `EmailService`.

3. **Start UserAuthenticationService**:
   - Run the Spring Boot application `UserAuthenticationService`.

4. **Test the Flow**:
   - Simulate a user registration in **UserAuthenticationService** (for example, through a REST API or a direct method call).
   - **EmailService** will listen to the Kafka topic, consume the message, and send an email.

---

### **Summary**

- **EmailService** is a Spring Boot microservice that listens to Kafka messages and sends emails using SMTP.
- **UserAuthenticationService** is a Spring Boot microservice that produces Kafka messages when a user registers.
- Kafka is used for asynchronous communication between these services, ensuring that email notifications are sent based on user registration events.
  
This architecture allows for decoupled communication between services, making the system more scalable and resilient.
