title: springcloud eureka 服务注册与发现
date: 2018-01-12 16:32:23
categories: Spring Boot
tags: springcloud

---

# spring cloud eureka

eureka 用以服务发现、服务注册，比较流行的有consul

### 简介

eureka为netflix开源软件，分为三个部分：


> eureka服务:用以提供服务注册、发现，已一个war的形式提供
http://search.maven.org/#search%7Cga%7C1%7Ceureka-server 
或者编译源码，将war拷贝进tomcat即可提供服务

> eureka-server: 相对client端的服务端，为客户端提供服务，通常情况下为一个 
集群

> eureka-client:客户端，通过向eureka服务发现注册的可用的eureka-server，向后端发送请求

# spring cloud eureka
spring cloud eureka 分为两部分


> @EnableEurekaClient: 该注解表明应用既作为eureka实例又为eureka client 可以发现注册的服务
> @EnableEurekaServer: 该注解表明应用为eureka服务，有可以联合多个服务作为集群，对外提供服务注册以及发现功能

# client 端

pom.xml

	<?xml version="1.0" encoding="UTF-8"?>
	<project xmlns="http://maven.apache.org/POM/4.0.0"
	         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
	         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
	    <parent>
	        <artifactId>springcloud</artifactId>
	        <groupId>com.lkl.springcloud</groupId>
	        <version>1.0-SNAPSHOT</version>
	    </parent>
	    <modelVersion>4.0.0</modelVersion>
	    <artifactId>eureka</artifactId>
	    <dependencyManagement>
	        <dependencies>
	            <dependency>
	                <groupId>org.springframework.cloud</groupId>
	                <artifactId>spring-cloud-netflix</artifactId>
	                <version>1.0.7.RELEASE</version>
	                <type>pom</type>
	                <scope>import</scope>
	            </dependency>
	        </dependencies>
	    </dependencyManagement>
	    <dependencies>
	        <dependency>
	            <groupId>org.springframework.cloud</groupId>
	            <artifactId>spring-cloud-starter-eureka</artifactId>
	        </dependency>
	        <!--表示为web工程-->
	        <dependency>
	            <groupId>org.springframework.boot</groupId>
	            <artifactId>spring-boot-starter-web</artifactId>
	        </dependency>
	        <!--暴露各种指标-->
	        <dependency>
	            <groupId>org.springframework.boot</groupId>
	            <artifactId>spring-boot-starter-actuator</artifactId>
	        </dependency>
	    </dependencies>
	</project>

启动应用 Application.java

	package com.lkl.springcloud.eureka;
	import org.springframework.boot.autoconfigure.EnableAutoConfiguration;
	import org.springframework.boot.autoconfigure.SpringBootApplication;
	import org.springframework.boot.builder.SpringApplicationBuilder;
	import org.springframework.cloud.netflix.eureka.EnableEurekaClient;
	import org.springframework.context.annotation.ComponentScan;
	import org.springframework.context.annotation.Configuration;
	import org.springframework.web.bind.annotation.RequestMapping;
	import org.springframework.web.bind.annotation.RestController;
	/**
	 * Created by tearfive on 16/4/30.
	 */
	@SpringBootApplication
	@EnableEurekaClient
	@RestController
	@EnableAutoConfiguration
	public class Application {
	    @RequestMapping("/")
	    public String home() {
	        return "Hello world";
	    }
	    public static void main(String[] args) {
	        new SpringApplicationBuilder(Application.class).web(true).run(args);
	    }
	}

创建配置application.properties

	server.port=9090
	spring.application.name=eureka.client
	#eureka.client.serviceUrl.defaultZone=http://120.76.145.187:8080/eureka-server-1.4.6/v2/
	eureka.client.serviceUrl.defaultZone=http://localhost:7070/eureka/
	eureka.instance.appname=eureka.client.01
	#eureka.client.registerWithEureka=true
	#eureka.client.fetchRegistry=true

eureka.client.serviceUrl.defaultZone 配置eureka服务地址

# server端

pom.xml

	<?xml version="1.0" encoding="UTF-8"?>
	<project xmlns="http://maven.apache.org/POM/4.0.0"
	         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
	         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
	    <parent>
	        <artifactId>springcloud</artifactId>
	        <groupId>com.lkl.springcloud</groupId>
	        <version>1.0-SNAPSHOT</version>
	    </parent>
	    <modelVersion>4.0.0</modelVersion>
	    <artifactId>eureka-server</artifactId>
	    <dependencyManagement>
	        <dependencies>
	            <dependency>
	                <groupId>org.springframework.cloud</groupId>
	                <artifactId>spring-cloud-netflix</artifactId>
	                <version>1.0.7.RELEASE</version>
	                <type>pom</type>
	                <scope>import</scope>
	            </dependency>
	        </dependencies>
	    </dependencyManagement>
	    <dependencies>
	        <dependency>
	            <groupId>org.springframework.cloud</groupId>
	            <artifactId>spring-cloud-starter-eureka</artifactId>
	        </dependency>
	        <!--表示为web工程-->
	        <dependency>
	            <groupId>org.springframework.boot</groupId>
	            <artifactId>spring-boot-starter-web</artifactId>
	        </dependency>
	        <!--暴露各种指标-->
	        <dependency>
	            <groupId>org.springframework.boot</groupId>
	            <artifactId>spring-boot-starter-actuator</artifactId>
	        </dependency>
	        <dependency>
	            <groupId>org.springframework.cloud</groupId>
	            <artifactId>spring-cloud-starter-eureka-server</artifactId>
	        </dependency>
	    </dependencies>
	</project>

启动应用Application.java

	package com.lkl.springcloud.eureka;
	import org.springframework.boot.autoconfigure.SpringBootApplication;
	import org.springframework.boot.builder.SpringApplicationBuilder;
	import org.springframework.cloud.netflix.eureka.server.EnableEurekaServer;
	/**
	 * Created by tearfive on 16/4/30.
	 */
	@EnableEurekaServer
	@SpringBootApplication
	public class Application {
	    public static void main(String[] args) {
	        new SpringApplicationBuilder(Application.class).web(true).run(args);
	    }
	}

配置application.yml

	server:
	  port: 7070
	spring:
	  application:
	    name: server-01
	eureka:
	  client:
	    serviceUrl:
	      defaultZone: http://localhost:7070/eureka/
	  instance:
	    metadataMap:
	      instanceId: ${spring.application.name}:${spring.application.instance_id:${random.value}}

spring.application.name表示应用名称，集群名称需要一致 
defaultZone表示向自身注册，例子中有三个server节点构成集群，其余两个两个节点也向该端口注册 
instanceId 表示eureka instance 标识，需要唯一，如果不配置，多个节点最终只会有一个生效

同样的配置[第二个节点](https://github.com/tearfive/eureka/tree/master/eureka-server-replicas),[第三个节点](https://github.com/tearfive/eureka/tree/master/eureka-server-replicas-two)

启动所有服务端应用 
访问 [http://localhost:7070](#) 可以查看eureka注册服务信息

访问 [http://localhost:7070/eureka/apps](#) 可以查看metadata

# 服务发现

在客户端创建Component

DiscoveryService.java

	package com.lkl.springcloud.eureka;
	import org.springframework.beans.factory.annotation.Autowired;
	import org.springframework.boot.CommandLineRunner;
	import org.springframework.cloud.client.ServiceInstance;
	import org.springframework.cloud.client.discovery.DiscoveryClient;
	import org.springframework.stereotype.Component;
	import org.springframework.util.CollectionUtils;
	import org.springframework.web.bind.annotation.RequestMapping;
	import org.springframework.web.bind.annotation.RestController;
	import java.util.List;
	/**
	 * Created by tearfive on 16/4/30.
	 */
	@Component
	@RestController
	public class DiscoveryService {
	    @Autowired
	    private DiscoveryClient discoveryClient;
	    @RequestMapping("/discovery")
	    public String doDiscoveryService(){
	        StringBuilder buf = new StringBuilder();
	        List<String> serviceIds = discoveryClient.getServices();
	        if(!CollectionUtils.isEmpty(serviceIds)){
	            for(String s : serviceIds){
	                System.out.println("serviceId:" + s);
	                List<ServiceInstance> serviceInstances =  discoveryClient.getInstances(s);
	                if(!CollectionUtils.isEmpty(serviceInstances)){
	                    for(ServiceInstance si:serviceInstances){
	                        buf.append("["+si.getServiceId() +" host=" +si.getHost()+" port="+si.getPort()+" uri="+si.getUri()+"]");
	                    }
	                }else{
	                    buf.append("no service.");
	                }
	            }
	        }
	        return buf.toString();
	    }
	}

访问 [http://localhost:9090/discovery](#) 实现服务发现。

ok ~ it’s work ! more about is [here](https://github.com/tearfive/eureka)