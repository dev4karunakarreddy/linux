# **Spring Boot WhatsApp Messaging with AWS Pinpoint**

## **Introduction**
This guide explains how to build a **Spring Boot application** to send WhatsApp messages using **AWS Pinpoint**.

## **Summary**
In this guide, we are developing a **Spring Boot application** that integrates with **AWS Pinpoint** to send WhatsApp messages. 
The purpose of this implementation is to provide a scalable and efficient way for businesses to send notifications, alerts, and promotional messages via WhatsApp. 
AWS Pinpoint enables organizations to send messages across multiple communication channels, including WhatsApp, without directly handling third-party API complexities. 
This solution is beneficial for developers and beginners who want to implement automated messaging, as it abstracts the complexities of authentication, message templating, and delivery status tracking. 
The approach ensures **secure credential management**, **template-based message sending**, and **REST API exposure** to allow easy integration with external applications.

## **Prerequisites**
- AWS Account
- WhatsApp Business Account linked with AWS Pinpoint
- Java 11 or later
- Maven
- Spring Boot

---

## **1. Setup AWS Credentials**
Store your AWS credentials securely:

**`application.properties`**
```properties
aws.accessKey=YOUR_AWS_ACCESS_KEY
aws.secretKey=YOUR_AWS_SECRET_KEY
aws.region=us-east-1
aws.pinpoint.appId=YOUR_PINPOINT_APPLICATION_ID
```

> **Note:** For production, use AWS Secrets Manager or IAM roles instead of hardcoding credentials.

---

## **2. Add Dependencies**
Include AWS SDK in `pom.xml`:

```xml
<dependency>
    <groupId>com.amazonaws</groupId>
    <artifactId>aws-java-sdk-pinpoint</artifactId>
    <version>1.12.653</version>
</dependency>
```

---

## **3. Configure AWS Pinpoint Client**

### **`AwsConfig.java`**
```java
package com.example.config;

import com.amazonaws.auth.AWSStaticCredentialsProvider;
import com.amazonaws.auth.BasicAWSCredentials;
import com.amazonaws.services.pinpoint.AmazonPinpoint;
import com.amazonaws.services.pinpoint.AmazonPinpointClientBuilder;
import org.springframework.beans.factory.annotation.Value;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;

@Configuration
public class AwsConfig {
    
    @Value("${aws.accessKey}")
    private String accessKey;

    @Value("${aws.secretKey}")
    private String secretKey;

    @Value("${aws.region}")
    private String region;

    @Bean
    public AmazonPinpoint amazonPinpoint() {
        BasicAWSCredentials awsCreds = new BasicAWSCredentials(accessKey, secretKey);
        return AmazonPinpointClientBuilder.standard()
                .withRegion(region)
                .withCredentials(new AWSStaticCredentialsProvider(awsCreds))
                .build();
    }
}
```

---

## **4. Implement WhatsApp Messaging Service**

### **`WhatsAppService.java`**
```java
package com.example.service;

import com.amazonaws.services.pinpoint.AmazonPinpoint;
import com.amazonaws.services.pinpoint.model.*;
import org.springframework.beans.factory.annotation.Value;
import org.springframework.stereotype.Service;

import java.util.HashMap;
import java.util.Map;

@Service
public class WhatsAppService {

    private final AmazonPinpoint amazonPinpoint;

    @Value("${aws.pinpoint.appId}")
    private String applicationId;

    public WhatsAppService(AmazonPinpoint amazonPinpoint) {
        this.amazonPinpoint = amazonPinpoint;
    }

    public String sendWhatsAppMessage(String phoneNumber, String messageText, String templateName) {
        Map<String, AddressConfiguration> addressMap = new HashMap<>();
        addressMap.put(phoneNumber, new AddressConfiguration()
                .withChannelType(ChannelType.WHATSAPP.toString()));

        WhatsAppMessage whatsAppMessage = new WhatsAppMessage()
                .withBody(messageText)
                .withTemplateName(templateName)
                .withTemplateLanguageCode("en")
                .withTemplateParameters(new HashMap<>());

        DirectMessageConfiguration directMessageConfiguration = new DirectMessageConfiguration()
                .withWhatsAppMessage(whatsAppMessage);

        MessageRequest messageRequest = new MessageRequest()
                .withAddresses(addressMap)
                .withMessageConfiguration(directMessageConfiguration);

        SendMessagesRequest sendMessagesRequest = new SendMessagesRequest()
                .withApplicationId(applicationId)
                .withMessageRequest(messageRequest);

        SendMessagesResult result = amazonPinpoint.sendMessages(sendMessagesRequest);

        return "Message sent with Request ID: " + result.getMessageResponse().getRequestId();
    }
}
```

---

## **5. Create REST Controller**

### **`WhatsAppController.java`**
```java
package com.example.controller;

import com.example.service.WhatsAppService;
import org.springframework.web.bind.annotation.*;

@RestController
@RequestMapping("/whatsapp")
public class WhatsAppController {

    private final WhatsAppService whatsAppService;

    public WhatsAppController(WhatsAppService whatsAppService) {
        this.whatsAppService = whatsAppService;
    }

    @PostMapping("/send")
    public String sendMessage(@RequestParam String phoneNumber, 
                              @RequestParam String message,
                              @RequestParam String template) {
        return whatsAppService.sendWhatsAppMessage(phoneNumber, message, template);
    }
}
```

---

## **6. Run and Test the Application**

### **Start the application**
```sh
mvn spring-boot:run
```

### **Test with `curl`**
```sh
curl -X POST "http://localhost:8080/whatsapp/send?phoneNumber=%2B1234567890&message=Hello%20from%20AWS&template=my_template"
```

---

## **7. AWS Setup Checklist**
- ✅ **Create a WhatsApp Business Account** via AWS Pinpoint
- ✅ **Verify Phone Number** with AWS and Meta
- ✅ **Create a Message Template** in AWS Pinpoint
- ✅ **Enable WhatsApp Channel** in AWS Pinpoint

---

## **Conclusion**
✅ **Uses AWS Pinpoint for WhatsApp Messaging**  
✅ **Secure AWS Configuration with Secret Key & Access Key**  
✅ **Spring Boot REST API for Sending Messages**  
✅ **Works with WhatsApp Message Templates**  

---

Let me know if you need any modifications! 🚀

