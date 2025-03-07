# Sending SMS using AWS SNS in Java

## Introduction
Amazon Simple Notification Service (SNS) allows sending SMS messages programmatically using AWS SDKs. This guide demonstrates how to send an SMS using AWS SNS in Java.

## Prerequisites
- AWS account
- AWS Access Key and Secret Key
- Amazon SNS enabled for SMS
- Java Development Kit (JDK) installed
- Maven or Gradle for dependency management

## Step 1: Add AWS SDK Dependencies

### Maven Dependency
```xml
<dependencies>
    <dependency>
        <groupId>software.amazon.awssdk</groupId>
        <artifactId>sns</artifactId>
        <version>2.20.40</version>
    </dependency>
    <dependency>
        <groupId>software.amazon.awssdk</groupId>
        <artifactId>auth</artifactId>
        <version>2.20.40</version>
    </dependency>
</dependencies>
```

### Gradle Dependency
```gradle
dependencies {
    implementation 'software.amazon.awssdk:sns:2.20.40'
    implementation 'software.amazon.awssdk:auth:2.20.40'
}
```

## Step 2: Create AWS SNS Configuration Class Using AWS Credentials Builder

```java
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import software.amazon.awssdk.auth.credentials.AwsBasicCredentials;
import software.amazon.awssdk.auth.credentials.StaticCredentialsProvider;
import software.amazon.awssdk.regions.Region;
import software.amazon.awssdk.services.sns.SnsClient;

@Configuration
public class AwsSnsConfig {
    private static final String ACCESS_KEY = "YOUR_ACCESS_KEY";
    private static final String SECRET_KEY = "YOUR_SECRET_KEY";
    private static final String TOPIC_ARN = "arn:aws:sns:ap-south-1:831926601470:K-Promotions";

    @Bean
    public SnsClient snsClient() {
        return SnsClient.builder()
                .region(Region.AP_SOUTH_1) // Set appropriate region
                .credentialsProvider(StaticCredentialsProvider.create(
                        AwsBasicCredentials.create(ACCESS_KEY, SECRET_KEY)))
                .build();
    }
    
    public String getTopicArn() {
        return TOPIC_ARN;
    }
}
```

## Step 3: Implement Java Code to Send SMS

```java
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.stereotype.Service;
import software.amazon.awssdk.services.sns.SnsClient;
import software.amazon.awssdk.services.sns.model.PublishRequest;
import software.amazon.awssdk.services.sns.model.PublishResponse;

@Service
public class SnsSmsSender {
    
    @Autowired
    private SnsClient snsClient;
    
    @Autowired
    private AwsSnsConfig awsSnsConfig;

    public void sendSms(String phoneNumber, String message) {
        PublishRequest request = PublishRequest.builder()
                .message(message)
                .phoneNumber(phoneNumber)
                .topicArn(awsSnsConfig.getTopicArn())
                .build();

        PublishResponse response = snsClient.publish(request);
        System.out.println("Message sent! Message ID: " + response.messageId());
    }
}
```

## Step 4: Call SMS Sending Service
```java
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.boot.CommandLineRunner;
import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;

@SpringBootApplication
public class SnsApplication implements CommandLineRunner {

    @Autowired
    private SnsSmsSender snsSmsSender;

    public static void main(String[] args) {
        SpringApplication.run(SnsApplication.class, args);
    }

    @Override
    public void run(String... args) {
        snsSmsSender.sendSms("+91XXXXXXXXXX", "Hello, this is a test message from AWS SNS!");
    }
}
```

## Step 5: Run the Spring Boot Application
1. Replace `YOUR_ACCESS_KEY`, `YOUR_SECRET_KEY`, and `+91XXXXXXXXXX` with valid values.
2. Start the Spring Boot application.

```sh
mvn spring-boot:run
```

## Step 6: Handling Errors
Common errors and solutions:
- **Opt-in required:** If sending fails, verify that the phone number is opted in for receiving SMS.
- **Invalid region:** Ensure the correct AWS region is used.
- **IAM permissions:** Ensure the IAM user has `sns:Publish` permission.

## Conclusion
This guide provides a step-by-step process to send SMS using AWS SNS in a Spring Boot application. Ensure AWS credentials, permissions, and SNS service limits are correctly configured for smooth operation.

For more details, refer to the [AWS SNS Documentation](https://docs.aws.amazon.com/sns/latest/dg/sms_publish-to-phone.html).
