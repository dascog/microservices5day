# Docker and Docker Compose for an ActiveMQ Messaging Service

In this example you will create the necessary Dockerfiles and fill out the given docker-compose.yaml file to allow your messaging service to run with docker-compose up. The Dockerfile for ActiveMQ and the docker-compose entries are included for you.

## Step 1 
Download the file in the project to your docker VM:

`` git clone https://bitbucket.org/dascog/mq-docker-before.git ``

## Step 2
Create the Dockerfiles in both of the Mq_SpringBootApp subdirectories

- In each case there is a single target file in the ./target directory.
- You need to create a docker image ``FROM openjdk:11.0``
- Then you need to ADD the target jar file ( `` ADD target/....jar app.jar `` )
- Finally create an entry point 
``
ENTRYPOINT ["java","-jar","/app.jar"]
``

## Step 3

- Open docker-compose.yaml file
- Under the appsend: entry, create the following attributes:
    ```
       container_name: appsendmq   
       build:   
           context: ./MqSenderSpringBootApp   
        image: emps/appsendmq:1.0.0   
        links:   
            - activemq:activemq   
    ```
- Repeat under the apprecv: entry. All the entries will be identical except you use recv in place of send.

## Step 4

- ``docker-compose up`` - that's it!
