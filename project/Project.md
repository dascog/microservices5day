# Microservices Final Group Project

This project is a "paper" project in which you design a microservices application from scratch and indicate which patterns would be applied and where. The goal of the project is to present your microservices application to the group in a way that you show a knowledge of the key technologies introduced and how they can be effectively utilized. You only have 15mins to present, so you need to keep it concise.
## Step 1  
- Describe an application that you think could/should be implemented as microservices.
- You application should include a User or Client element and a Billing element. 
- The rest is up to you!

## Step 2
- Using either the Decomposition by Business Capability pattern or the Decomposition by Subdomain pattern show the services you would decompose you application into.
    - If you are using Business Capability, talk about how each of your services correspond to "something a business does to generate value"
    - If you are using Subdomain, classify each of your subdomains as either Core, Supporting and Generic.
- Define which data will be collected by the databases associated with each service - try to get the best data decoupling you can.
    - Note where you might have conflicts with multiple services requiring the same data.
- Try to describe each service as clearly as possible. 
  
## Step 3
- Decide on the API calls you want for each of your externally facing APIs
- Use SwaggerHub to implement as many of your REST APIs as possible. 
- Use the API Gateway Pattern, and API Composition pattern (and possibly the CQRS Pattern) to show how different calls are routed within your microservices array

## Step 4
- Write a list of all the commands or queries that need to be communicated between your services.
- Mark them as synchronous-asynchronous and request-response, publish/subscribe or notification.
- On the basis of this info, decide which messaging pattern you would like to use and why.
- For whichever pattern you have chosen, describe the passage of at least two of your key client requests through your microservice (using a Saga if you are using messaging).
- If you are using RPI, indicate where you would implement Circuit Breakers.
- Optional: Write a protobuf for one of your internal APIs that would be suitable for gRPC.

## Step 5
- Write a Saga state machine to describe at least one transaction in your system that utilizes at least one asynchronous messaging protocol. 
- If you don't have any asynchronous messaging, pick a transaction and show how it could be implemented using a Saga with async messaging.

## Step 6
- Talk about the deployment of your microservices application. What functionality would you include in a microservices chassis? What functionality are you expecting to get from a service mesh? What off-the-shelf products will you use? What patterns are you left to implement yourself?

## Step 7
- Present your conclusions in a 15min presentation to the whole group. Use whatever presentation medium you think will communicate your ideas, choices and technologies as clearly as possible. 
- It may help to think of this as a technical sales pitch to upper management for the application you are presenting.
