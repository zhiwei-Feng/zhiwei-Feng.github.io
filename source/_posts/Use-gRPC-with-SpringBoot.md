---
title: Use gRPC with SpringBoot
date: 2022-05-29 20:45:45
categories:
- Java
tags:
- SpringBoot
- gRPC
---

gRPC相比于REST HTTP请求在接口规范、传输性能和流式通信方面更好,因此广泛应用分布式系统和微服务的通信过程中. 本文主要是针对目前gRPC-Java和SpringBoot的入门级教程不够统一而写,也是使用过程的记录.感谢[grpc-spring-boot-starter](https://github.com/yidongnan/grpc-spring-boot-starter)提供了开箱即用的gRPC服务端和客户端的接口实现,因此我们可以专注业务实现.

<!-- more -->

## 项目简述
该Demo主要实现一个gRPC服务端和一个gRPC客户端,服务端提供一个api接收请求参数name,返回消息`Hello ==> <name>`. 本项目基于maven构建的多module工程实现.

## 预备
**为了避免出现版本兼容问题**,我们在parent pom文件中定义好springboot、java、grpc和对应grpc-spring-boot-starter版本,如下:
```
<parent>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-parent</artifactId>
    <version>2.5.8</version>
    <relativePath/> <!-- lookup parent from repository -->
</parent>
<properties>
    <java.version>11</java.version>
    <grpc.version>1.42.2</grpc.version>
    <grpc.starter.version>2.13.1.RELEASE</grpc.starter.version>
</properties>
```

## grpc interface
### 1. proto文件定义
我们新建一个子module为`grpc`, 作为公用proto文件和对应protobuf生成的文件的存放处(后续的gRPC server和client项目都以此module作为依赖),其中proto文件放在`src/main/proto`目录下.进行相关定义.
```protobuf
syntax = "proto3";

package service;

option java_multiple_files = true;
option java_package = "com.fengzw.minimall.minimalluser.service";
option java_outer_classname = "HelloWorldProto";

// The greeting service definition.
service MyService {
  // Sends a greeting
  rpc SayHello (HelloRequest) returns (HelloReply) {}
}

// The request message containing the user's name.
message HelloRequest {
  string name = 1;
}

// The response message containing the greetings
message HelloReply {
  string message = 1;
}
```

### 2. maven配置
同时在module的pom文件中配置好maven插件和相关依赖,如下:
```
<dependency>
    <groupId>io.grpc</groupId>
    <artifactId>grpc-netty-shaded</artifactId>
    <version>${grpc.version}</version>
    <scope>runtime</scope>
</dependency>
<dependency>
    <groupId>io.grpc</groupId>
    <artifactId>grpc-protobuf</artifactId>
    <version>${grpc.version}</version>
</dependency>
<dependency>
    <groupId>io.grpc</groupId>
    <artifactId>grpc-stub</artifactId>
    <version>${grpc.version}</version>
</dependency>
<dependency> <!-- necessary for Java 9+ -->
    <groupId>org.apache.tomcat</groupId>
    <artifactId>annotations-api</artifactId>
    <version>6.0.53</version>
    <scope>provided</scope>
</dependency>


<build>
    <extensions>
        <extension>
            <groupId>kr.motd.maven</groupId>
            <artifactId>os-maven-plugin</artifactId>
            <version>1.6.2</version>
        </extension>
    </extensions>
    <plugins>
        <plugin>
            <groupId>org.xolstice.maven.plugins</groupId>
            <artifactId>protobuf-maven-plugin</artifactId>
            <version>0.6.1</version>
            <configuration>
                <protocArtifact>com.google.protobuf:protoc:3.19.2:exe:${os.detected.classifier}</protocArtifact>
                <pluginId>grpc-java</pluginId>
                <pluginArtifact>io.grpc:protoc-gen-grpc-java:1.45.1:exe:${os.detected.classifier}</pluginArtifact>
                <protoSourceRoot>src/main/proto</protoSourceRoot>
                <outputDirectory>src/main/java</outputDirectory>
                <clearOutputDirectory>false</clearOutputDirectory>
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
        <plugin>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-maven-plugin</artifactId>
        </plugin>
    </plugins>
</build>
```
通过编译该module可以自动生成对应的gRPC文件到`src/main/java`的目录下.

## 服务端module
### 1. maven配置
新建一个module作为gRPC服务端,配置好对应的pom文件:(包括grpc module和server-starter依赖)
```
<dependency>
    <groupId>com.fengzw.minimall</groupId>
    <artifactId>grpc</artifactId>
    <version>0.0.1-SNAPSHOT</version>
</dependency>
<dependency>
    <groupId>net.devh</groupId>
    <artifactId>grpc-server-spring-boot-starter</artifactId>
    <version>${grpc.starter.version}</version>
</dependency>
```

### 2. 定义服务service
构建一个service提供gRPC server服务,如下:
```java
@GrpcService
public class GrpcServerService extends MyServiceGrpc.MyServiceImplBase {

    @Override
    public void sayHello(HelloRequest request, StreamObserver<HelloReply> responseObserver) {
        System.out.println("Hello ==> " + request.getName());
        HelloReply reply = HelloReply.newBuilder().setMessage("Hello ==> " + request.getName()).build();
        responseObserver.onNext(reply);
        responseObserver.onCompleted();
    }

    @Override
    public void sayHelloAgain(HelloRequest request, StreamObserver<HelloReply> responseObserver) {
        super.sayHelloAgain(request, responseObserver);
    }
}
```
如果你顺利编译了grpc module,那么MyServiceGrpc文件和相应的model文件会出现在grpc的`src/main/java`目录下. 启动该module,你会在SpringBoot的启动日志见到如下信息:
```
2022-05-29 20:47:18.223  INFO 16841 --- [  restartedMain] n.d.b.g.s.s.AbstractGrpcServerFactory    : Registered gRPC service: service.MyService, bean: grpcServerService, class: com.fengzw.minimall.minimalluser.service.GrpcServerService
2022-05-29 20:47:18.223  INFO 16841 --- [  restartedMain] n.d.b.g.s.s.AbstractGrpcServerFactory    : Registered gRPC service: grpc.health.v1.Health, bean: grpcHealthService, class: io.grpc.protobuf.services.HealthServiceImpl
2022-05-29 20:47:18.223  INFO 16841 --- [  restartedMain] n.d.b.g.s.s.AbstractGrpcServerFactory    : Registered gRPC service: grpc.reflection.v1alpha.ServerReflection, bean: protoReflectionService, class: io.grpc.protobuf.services.ProtoReflectionService
2022-05-29 20:47:18.281  INFO 16841 --- [  restartedMain] n.d.b.g.s.s.GrpcServerLifecycle          : gRPC Server started, listening on address: *, port: 9091
```
此时通过idea的http request你可以调用grpc请求来测试该grpc service的状态.
```
GRPC localhost:9090/service.MyService/SayHello

{
  "name": "request"
}
```
你会看到如下输出.
```
{
  "message": "Hello \u003d\u003d\u003e request"
}
```
这样我们的服务端就实现好了.

## 客户端module
### 1. maven配置
新建一个module作为grpc 客户端, 如服务端一样, 我们同样需要在maven pom文件中依赖grpc module, 并依赖client-starter依赖如下:
```
<dependency>
    <groupId>com.fengzw.minimall</groupId>
    <artifactId>gprc</artifactId>
    <version>0.0.1-SNAPSHOT</version>
</dependency>
<dependency>
    <groupId>net.devh</groupId>
    <artifactId>grpc-client-spring-boot-starter</artifactId>
    <version>${grpc.starter.version}</version>
</dependency>
```

### 2. springboot配置文件中配置grpc server信息
首先我们需要在配置文件`application.properties`中定义好grpc server的相关信息:
```
grpc.client.user-server.address=static://127.0.0.1:9091
grpc.client.user-server.enable-keep-alive=true
grpc.client.user-server.keep-alive-without-calls=true
grpc.client.user-server.negotiation-type=plaintext
```
> WARNING: 注意如果是同一台服务器来进行服务端和客户端的测试, 那么服务端需要设置grpc server的端口在除了9090之外的端口, 否则会引起调用时报错
> `INTERNAL: http2 exception`
> `Caused by: io.netty.handler.codec.http2.Http2Exception: First received frame was not SETTINGS. Hex dump for first 5 bytes: 485454502f`

### 3. 定义客户端方法
然后我们在一个spring service组件下定义我们的客户端方法,如下:
```java
@Service
public class DemoService {
    @GrpcClient("user-server")
    private MyServiceGrpc.MyServiceBlockingStub myServiceStub;

    public String hello(String name) {
        HelloRequest request = HelloRequest.newBuilder().setName(name).build();
        return myServiceStub.sayHello(request).getMessage();
    }
}
```
这里`@GrpcClient`中的value表示grpc server的名称, 在配置文件中进行了定义. 我们可以通过定义一个controller来对外提空一个rest api来进行grpc客户端调用, 如下:
```java
@Autowired
private DemoService service;

@GetMapping("/hello")
public String hello() {
    return service.hello("demo");
}
```
这样客户端就全部配置完成了.

## 总结
可以看到相比于Go语言中对gRPC的使用, `grpc-spring-boot-starter`极大地简化了非业务相关的代码.
