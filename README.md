#Consumer Driven Contract Testing


#Introduction

   
   In this example, we have a Consumer Driven Contract Test between a Gradle Spring Boot dummy provider (RESTful Web Service) and a Gradle dummy consumer.

   We  make use of Pact-JVM, which will provide a mock service for the consumer project to generate a Pact, and verification ability for the provider project to verify the Pact.

#Installation

 #Gradle
   
   A Gradle Wrapper allows anybody to work on the project without having to install Gradle. It ensures that the right version of Gradle that the build was designed for is shipped as part of this project repository. First clone the sample repository that includes both the dummy provider and the dummy consumer:

```
git clone http://mekak2@vstash:7990/scm/ta/consumerdrivencontracts.git

```

#Provider Setup

   Navigate into the provider directory, where you will see a sample RESTful Web Service that was built using Sprint Boot and Gradle through this [guide](https://spring.io/guides/gs/actuator-service/). Try running the service by using the following command:

```
        cd provider
        ./gradlew clean build && java -jar build/libs/gs-actuator-service-0.1.0.jar

```

   You should now be able to reach to service by navigating to http://localhost:8080/hello-world or by using curl (observe that the id increments for every request made to the service):

```
curl http://localhost:8080/hello-world

```


#Consumer Setup

   The generation of a Pact is always done from the consumer point of view. This generation can be broken down into three parts.

 #Part 1: Pact Rule
   In this part, we define a mock server’s host and port to represent the provider:
     
      
      
```

@Rule
public PactRule rule = new PactRule(Configuration.MOCK_HOST, Configuration.MOCK_HOST_PORT, this);
private DslPart helloWorldResults;

```
      
 #Part 2: Pact Fragment
   In this part, we construct a Pact Fragment, which defines the expected contract from a provider:
     

```
         
@Pact(state = "HELLO WORLD", provider = Configuration.DUMMY_PROVIDER, consumer = Configuration.DUMMY_CONSUMER)
 public PactFragment createFragment(ConsumerPactBuilder.PactDslWithProvider.PactDslWithState builder)
  {
   helloWorldResults = new PactDslJsonBody()
                     .id()
                     .stringType("content")
                     .asBody();
         
   return builder
                     .uponReceiving("get hello world response")
                     .path("/hello-world")
                     .method("GET")
                     .willRespondWith()
                     .status(200)
                     .headers(Configuration.getHeaders())
                     .body(helloWorldResults)
                     .toFragment();
   }
         
 ```
  #Part 3: Pact Verification
   In this part, we ensure that the constructed fragment in Part 2, matches the response from the mock server:
          
```
  @Test
           @PactVerification("HELLO WORLD")
           public void shouldGetHelloWorld() throws IOException
           {
               DummyConsumer restClient = new DummyConsumer(Configuration.SERVICE_URL);
               assertEquals(helloWorldResults.toString(), restClient.getHelloWorld());
           }      
         
          
```
   Try generating a Pact using Gradle from the consumer directory:
         
```
  cd ..
  cd consumer
  ./gradlew test      
        
```
        
   The Pact will then be generated and stored in the pacts directory, but it looks something like this:
      
```
 {
             "provider" : {
               "name" : "dummy-provider"
             },
             "consumer" : {
               "name" : "dummy-consumer"
             },
             "interactions" : [ {
               "providerState" : "HELLO WORLD",
               "description" : "get hello world response",
               "request" : {
                 "method" : "GET",
                 "path" : "/hello-world"
               },
               "response" : {
                 "status" : 200,
                 "headers" : {
                   "Content-Type" : "application/json;charset=UTF-8"
                 },
                 "body" : {
                   "id" : 5677679801,
                   "content" : "dugNvVPasiFRnzqpPNuq"
                 },
                 "responseMatchingRules" : {
                   "$.body.id" : {
                     "match" : "type"
                   },
                   "$.body.content" : {
                     "match" : "type"
                   }
                 }
               }
             } ],
             "metadata" : {
               "pact-specification" : {
                 "version" : "2.0.0"
               },
               "pact-jvm" : {
                 "version" : "2.1.12"
               }
             }
           }         
          
```
  #Provider
   
    Now that the Pact has been generated by the Consumer, and assuming the Provider (RESTful Web Service) is running. You can run the following command from the provider directory to ensure that the Provider meets all expectations of the Consumer:       
         
    
``` 
   cd ..
   cd provider
   ./gradlew pactVerify    
       
```   
       
      
      