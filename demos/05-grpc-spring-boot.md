# Example using Spring Boot to create a gRPC client-server
This is a simple example project using Spring Boot to create a gRPC demo. It can be done on any standard Java platform, I have used IntelliJ and VSCode. 

## Step 1: Create a Spring Web project
- Use you IDE to create a Maven Spring Boot project called grpc-demo with Spring Web dependency that compiles to a jar file. 

## Step 2: Create the protobuf file
- In the ``src/main`` directory of your code tree, create a directory called ``proto``
- In the ``proto`` directory create a file called ``ChatMessages.proto`` with the following content:
```
syntax = "proto3";
option java_multiple_files = true;

import "google/protobuf/timestamp.proto";
package com.example.grpcdemo.proto; // make sure this is whatever the package this file resides in
message ChatMessage {
    string from = 1;
    string message = 2;
}
message ChatMessageFromServer {
    google.protobuf.Timestamp timestamp = 1;
    ChatMessage message = 2;
}
service ChatService {
    rpc chat(stream ChatMessage) returns (stream ChatMessageFromServer);
}

```
The ``java_multiple_files`` option creates java classes in multiple files instead of one large file.

## Step 3: Edit the POM.xml file
- Find the ``<plugins>`` section within ``<build>`` in your ``pom.xml`` file. This file is located in the top level of your project.
- Add the following plugin
```
        <plugin>
                <groupId>org.xolstice.maven.plugins</groupId>
                <artifactId>protobuf-maven-plugin</artifactId>
                <version>0.6.1</version>
                <configuration>
                    <protocArtifact>com.google.protobuf:protoc:3.3.0:exe:${os.detected.classifier}</protocArtifact>
                    <pluginId>grpc-java</pluginId>
                    <pluginArtifact>io.grpc:protoc-gen-grpc-java:1.4.0:exe:${os.detected.classifier}</pluginArtifact>
                    <!--protoSourceRoot>${basedir}/src/main/resources/proto</protoSourceRoot-->
                </configuration>
                <executions>
                    <execution>
                        <goals>
                            <goal>compile</goal>
                            <goal>compile-custom</goal>
                        </goals>
                    </execution>
		        </executions>
	   </plugin>
```
- In the top level of the ``<build>`` tag, add the following extensions:
```
        <extensions>
            <extension>
                <groupId>kr.motd.maven</groupId>
                <artifactId>os-maven-plugin</artifactId>
                <version>1.6.1</version>
            </extension>
        </extensions>
```
- In the ``<dependencies>`` tag, add the following dependency:
```
        <dependency>
            <groupId>net.devh</groupId>
            <artifactId>grpc-spring-boot-starter</artifactId>
            <version>2.12.0.RELEASE</version>
        </dependency>
```
## Step 4: Generate the gRPC classes
- In a command shell in the top level directory of your project, run ``./mvnw clean -X compile``. This will use the configuration to invoke ``protoc`` on your protobuf file and automatically generate the stub files for your gRPC client and server.

## Step 5: Add the generated sources to your pom.xml
- Now that the sources have been generated, you can add them to your ``pom.xml`` as well. If you had attempted to do this earlier you would have got an error and not been able to build because your pom would refer to files that didn't exist.
- Look for the ``<executions>`` tag that you inserted in step 3, and insert the following child tag:
```
                    <execution>
                        <id>add-source</id>
                        <phase>generate-sources</phase>
                        <configuration>
                            <sources>
                                <source>${project.build.directory}/target/generated-sources/protobuf</source>
                                <source>${project.build.directory}/target/generated-sources/protobuf/grpc-java</source>
                            </sources>
                        </configuration>
                    </execution>

```

## Step 6: Create a controllers package and a ChatServiceImpl class
- In your ``src/com/example/grpcdemo`` directory, create a new package/directory called ``controllers``
- In that package, create a new Java class called ``ChatServiceImpl`` that extends ``ChatServiceGrpc.ChatServiceImplBase`` from your generated sources.
```    
package com.example.grpcdemo.controllers;

import com.example.grpcdemo.proto.ChatServiceGrpc;

public class ChatServiceImpl extends ChatServiceGrpc.ChatServiceImplBase {
}
```
- Override the ``chat`` method and keep a list of the Client StreamObservers, and return a new StreamObserver, defining the necessary overrides in the return:

```

public class ChatServiceImpl extends ChatServiceGrpc.ChatServiceImplBase {

    private static LinkedHashSet<StreamObserver<ChatMessageFromServer>> chatClients = new LinkedHashSet<>();

    @Override
    public StreamObserver<ChatMessage> chat(StreamObserver<ChatMessageFromServer> responseObserver) {
        chatClients.add(responseObserver);

        return new StreamObserver<>() {
            @Override
            public void onNext(ChatMessage chatMessage) {

            }

            @Override
            public void onError(Throwable throwable) {
            }

            @Override
            public void onCompleted() {
            }
        };
    }
```
- The ``chatClients`` list is augmented with a client everytime a client sends a chat message to the server.
- The ``chat`` method returns a ``StreamObserver`` object, which must override the ``onNext``, ``onError`` and ``onCompleted`` methods.
- Now fill out the override of onNext with a method that iterates through all the client StreamObservers and publishes the chatMessage to each:
```
            @Override
            public void onNext(ChatMessage chatMessage) {
                for (StreamObserver<ChatMessageFromServer> chatClient : chatClients) {
                    chatClient.onNext(ChatMessageFromServer.newBuilder().setMessage(chatMessage).build());
                }

            }
```
- The onError and onCompleted overrides can be completed as follows:
```
            @Override
            public void onError(Throwable throwable) {
                responseObserver.onError(throwable);
            }

            @Override
            public void onCompleted() {
                chatClients.remove(responseObserver);
            }
```
- Your IDE should prompt you for all the correct imports, the completed file is as follows:
```
package com.example.grpcdemo.controllers;

import java.util.LinkedHashSet;

import com.example.grpcdemo.proto.ChatMessage;
import com.example.grpcdemo.proto.ChatMessageFromServer;
import com.example.grpcdemo.proto.ChatServiceGrpc;
import io.grpc.stub.StreamObserver;

public class ChatServiceImpl extends ChatServiceGrpc.ChatServiceImplBase {

        private static LinkedHashSet<StreamObserver<ChatMessageFromServer>> chatClients = new LinkedHashSet<>();

        @Override
        public StreamObserver<ChatMessage> chat(StreamObserver<ChatMessageFromServer> responseObserver) {
            chatClients.add(responseObserver);
    
            return new StreamObserver<>() {
                @Override
                public void onNext(ChatMessage chatMessage) {
                    for (StreamObserver<ChatMessageFromServer> chatClient : chatClients) {
                        chatClient.onNext(ChatMessageFromServer.newBuilder().setMessage(chatMessage).build());
                    }
                }
    
                @Override
                public void onError(Throwable throwable) {
                    responseObserver.onError(throwable);
                }
    
                @Override
                public void onCompleted() {
                    chatClients.remove(responseObserver);
                }
            };
        }
}
```
## Step 7: Create a test application.properties
The gRPC server code is now complete. We will test it using the Spring Boot testing framework
- Create a directory called ``resources`` under ``src/test`` and add a new ``application.properties`` file to it. This contains the variables necessary to specify that channel that will be used for the client-server:

```
grpc.server.inProcessName=test
grpc.server.port=-1
grpc.client.inProcess.address=in-process:test
```
- Setting the ``grpc.server.port`` to -1 forces the server to work locally. This isn't all that interesting as there are much easier ways to get classes to talk to each other in a Java project, however the code is unchanged if the server and client are on separate machines. 

## Step 8: Edit the Tests class
- Open the auto-generated ``GrpcDemoApplicationTests.java`` file in the ``test/java/com/example/grpcdemo`` package. 
- We will use the ``@GrpcClient`` annotation to create two clients, and use ``inProcess`` argument so the system knows to use the ``inprocess`` connection.
  - The address value comes from the ``grpc.client.inProcess.address`` we set in the ``test/application.properties`` file.

```
@SpringBootTest
class GrpcDemoApplicationTests {

    @GrpcClient("inProcess")
    private ChatServiceGrpc.ChatServiceStub chatClient1;

    @GrpcClient("inProcess")
    private ChatServiceGrpc.ChatServiceStub chatClient2;
}
```
- Now we can write the test class. In this we will need to create two StreamObservers (1 for each chat client), then add received messages to a list in onNext so we can validate our test.
- Leave the other methods empty.

```
    private StreamObserver<ChatMessage> generateObserver(List messages)
    {
        return new StreamObserver<>() {
            @Override
            public void onNext(ChatMessage chatMessage) {
                messages.add(chatMessage);
                System.out.println(chatMessage.getMessage());
            }

            @Override
            public void onError(Throwable throwable) {

            }

            @Override
            public void onCompleted() {

            }
        };
    }
```
- Now we need to register the client observers with the server and start sending messages.
```
    private ChatMessage generateChatMessage(String message, String from)
    {
        return ChatMessage.newBuilder()
                .setMessage(message)
                .setFrom(from)
                .build();
    }

    @Test
    public void testChat()
    {
        List messages = new ArrayList<ChatMessageFromServer>();

        StreamObserver chatClient1Observer = generateObserver(messages);
        StreamObserver chatClient2Observer = generateObserver(messages);

        chatClient1.chat(chatClient1Observer);
        chatClient2.chat(chatClient2Observer);

        chatClient1Observer.onNext(generateChatMessage("Hello Chat Client2", "ChatClient1"));
        chatClient2Observer.onNext(generateChatMessage("Hello Chat Client1", "ChatClient2"));

        Assert.notEmpty(messages, "Validate Messages are populated");
    }
```
- The full test file should look like this:
```

package com.example.grpcdemo;

import java.util.ArrayList;
import java.util.List;

import org.junit.jupiter.api.Test;
import org.springframework.boot.test.context.SpringBootTest;
import org.springframework.util.Assert;

import com.example.grpcdemo.proto.ChatMessage;
import com.example.grpcdemo.proto.ChatMessageFromServer;
import com.example.grpcdemo.proto.ChatServiceGrpc;

import io.grpc.stub.StreamObserver;
import net.devh.boot.grpc.client.inject.GrpcClient;

@SpringBootTest
class GrpcDemoApplicationTests {

    @GrpcClient("inProcess")
    private ChatServiceGrpc.ChatServiceStub chatClient1;

    @GrpcClient("inProcess")
    private ChatServiceGrpc.ChatServiceStub chatClient2;

	private StreamObserver<ChatMessage> generateObserver(List messages)
    {
        return new StreamObserver<>() {
            @Override
            public void onNext(ChatMessage chatMessage) {
                messages.add(chatMessage);
                System.out.println(chatMessage.getMessage());
            }

            @Override
            public void onError(Throwable throwable) {

            }

            @Override
            public void onCompleted() {

            }
        };
    }
	
    private ChatMessage generateChatMessage(String message, String from)
    {
        return ChatMessage.newBuilder()
                .setMessage(message)
                .setFrom(from)
                .build();
    }

    @Test
    public void testChat()
    {
        List messages = new ArrayList<ChatMessageFromServer>();

        StreamObserver chatClient1Observer = generateObserver(messages);
        StreamObserver chatClient2Observer = generateObserver(messages);

        chatClient1.chat(chatClient1Observer);
        chatClient2.chat(chatClient2Observer);

        chatClient1Observer.onNext(generateChatMessage("Hello Chat Client2", "ChatClient1"));
        chatClient2Observer.onNext(generateChatMessage("Hello Chat Client1", "ChatClient2"));

        Assert.notEmpty(messages, "Validate Messages are populated");
    }
}
```

## Step 9: Test your gRPC server
- Use your IDE to run the ``testChat`` method (in VSCode a green arrow will appear to the right of the method that you can use to run the code).
- If all has been implemented properly, you should get an output that looks like this:

```
 .   ____          _            __ _ _
 /\\ / ___'_ __ _ _(_)_ __  __ _ \ \ \ \
( ( )\___ | '_ | '_| | '_ \/ _` | \ \ \ \
 \\/  ___)| |_)| | | | | || (_| |  ) ) ) )
  '  |____| .__|_| |_|_| |_\__, | / / / /
 =========|_|==============|___/=/_/_/_/
 :: Spring Boot ::                (v2.7.0)

2022-06-22 10:50:24.715  INFO 25009 --- [           main] c.e.grpcdemo.GrpcDemoApplicationTests    : Starting GrpcDemoApplicationTests using Java 11.0.15 on ip-172-31-7-63.eu-west-2.compute.internal with PID 25009 (started by trainer in /home/trainer/grpc-demo/grpc-demo)
2022-06-22 10:50:24.722  INFO 25009 --- [           main] c.e.grpcdemo.GrpcDemoApplicationTests    : No active profile set, falling back to 1 default profile: "default"
2022-06-22 10:50:27.234  INFO 25009 --- [           main] g.s.a.GrpcServerFactoryAutoConfiguration : 'grpc.server.in-process-name' is set: Creating InProcessGrpcServerFactory
2022-06-22 10:50:27.446  INFO 25009 --- [           main] n.d.b.g.s.s.AbstractGrpcServerFactory    : Registered gRPC service: grpc.health.v1.Health, bean: grpcHealthService, class: io.grpc.services.HealthServiceImpl
2022-06-22 10:50:27.447  INFO 25009 --- [           main] n.d.b.g.s.s.AbstractGrpcServerFactory    : Registered gRPC service: grpc.reflection.v1alpha.ServerReflection, bean: protoReflectionService, class: io.grpc.protobuf.services.ProtoReflectionService
2022-06-22 10:50:27.457  INFO 25009 --- [           main] n.d.b.g.s.s.GrpcServerLifecycle          : gRPC Server started, listening on address: in-process:test, port: -1
2022-06-22 10:50:27.475  INFO 25009 --- [           main] c.e.grpcdemo.GrpcDemoApplicationTests    : Started GrpcDemoApplicationTests in 3.126 seconds (JVM running for 4.347)
2022-06-22 10:50:27.508  INFO 25009 --- [           main] n.d.b.g.c.a.GrpcClientAutoConfiguration  : Detected grpc-netty-shaded: Creating ShadedNettyChannelFactory + InProcessChannelFactory
Hello Chat Client2
Hello Chat Client1
2022-06-22 10:50:28.061  INFO 25009 --- [ionShutdownHook] n.d.b.g.s.s.GrpcServerLifecycle          : Completed gRPC server shutdown
```
- The two ``Hello Chat ...`` lines indicate the gRPC communication is working as expected.