# 2.2 Java

[TOC]

## 基础

### 获取文件最后的修改时间

```java
import java.io.File;
import java.util.Date;
 
public class Main {
    public static void main(String[] args) {
        File file = new File("Main.java");
        Long lastModified = file.lastModified();
        Date date = new Date(lastModified);
        System.out.println(date);
    }
}
```

### JAR无法读取配置文件

```java
public static String readFileByStream(String fileName){
    try {
        StringBuilder stringBuffer = new StringBuilder();
        InputStream stream = FilesUtils.class.getClassLoader().getResourceAsStream(fileName);
        if (stream == null) {
            return null;
        }
        BufferedReader br = new BufferedReader(new InputStreamReader(stream, StandardCharsets.UTF_8));
        String line;
        while ((line = br.readLine()) != null) {
            stringBuffer.append(line);
        }
        return stringBuffer.toString();
    } catch (Exception e) {
        e.printStackTrace();
    }
    return null;
}
```

### 读取JSON文件中的unicode

使用`JSONObject`转化一遍。

## IDEA

### 开启控制台颜色

```shell
# Edit Configurations，点击，找到VM options添加
-Dspring.output.ansi.enabled=ALWAYS
```

### 添加启动命令

`Edit Configurations`->`Env Vars: "spring.profiles.active=dev"`

## Spring Boot

### 打包

为方便程序的安装与运行，将`JAR`与运行脚本打包为`RPM`包。

### 开启SSL[^1]

1. 生成证书

    ```cmd
    keytool -genkey -keyalg RSA -keysize 2048 -validity 3650 -keystore application.keystore -storepass PASS@2019 -storetype PKCS12
    ```

    

2. 添加配置到配置文件`application.properties`

    ```properties
    server.port=443
    server.http.port=80
    
    ########    SSL    ########
    server.ssl.key-store=classpath:application.keystore
    server.ssl.key-store-password=PASS@2019
    server.ssl.key-store-type=PKCS12
    ```

    

3. 在启动类`*Application.java`配置连接器，同时启用HTTP和HTTPS

    ```java
    // 兼容HTTP 和 HTTPS
    @Bean
    public ServletWebServerFactory servletContainer(){
        TomcatServletWebServerFactory tomcat = new TomcatServletWebServerFactory();
        tomcat.addAdditionalTomcatConnectors(createHTTPConnector());
        return tomcat;
    }
    
    // HTTP自动跳转到HTTPS
    @Bean
    public TomcatServletWebServerFactory servletContainer() {
        TomcatServletWebServerFactory tomcat = new TomcatServletWebServerFactory() {
            @Override
            protected void postProcessContext(Context context) {
                SecurityConstraint securityConstraint = new SecurityConstraint();
                securityConstraint.setUserConstraint("CONFIDENTIAL");
                SecurityCollection collection = new SecurityCollection();
                collection.addPattern("/*");
                securityConstraint.addCollection(collection);
                context.addConstraint(securityConstraint);
            }
        };
        tomcat.addAdditionalTomcatConnectors(httpConnector());
        return tomcat;
    }
    
    private Connector createHTTPConnector() {
        Connector connector = new Connector("org.apache.coyote.http11.Http11NioProtocol");
        connector.setScheme("http");
        connector.setSecure(false);
        connector.setPort(httpPort);
        connector.setRedirectPort(httpsPort);
        return connector;
    }
    ```

### 启用Actuator

```xml
<dependency>
<groupId>org.springframework.boot</groupId>
<artifactId>spring-boot-starter-actuator</artifactId>
</dependency>
```

```properties
management.endpoints.web.exposure.include=env,beans,health,autoconfig
management.server.port=8110
management.server.servlet.context-path=/
management.server.ssl.enabled=false
management.endpoint.health.show-details=always
```

查看状态

`GET: http://{host}:{port}/actuator`

```json
{
    "_links": {
        "self": {
            "href": "http://localhost:8110/actuator",
            "templated": false
        },
        "beans": {
            "href": "http://localhost:8110/actuator/beans",
            "templated": false
        },
        "health-component-instance": {
            "href": "http://localhost:8110/actuator/health/{component}/{instance}",
            "templated": true
        },
        "health-component": {
            "href": "http://localhost:8110/actuator/health/{component}",
            "templated": true
        },
        "health": {
            "href": "http://localhost:8110/actuator/health",
            "templated": false
        },
        "env": {
            "href": "http://localhost:8110/actuator/env",
            "templated": false
        },
        "env-toMatch": {
            "href": "http://localhost:8110/actuator/env/{toMatch}",
            "templated": true
        }
    }
}
```

### 创建非Web项目



## Logback(Slf4j)配置文件

在`Spring Boot`中默认使用`Logback`作为日志插件，`Logback`的默认配置文件为`logback-spring.xml`，文件内容示例如下。

```xml
<?xml version="1.0" encoding="UTF-8"?>
<configuration scan="true" scanPeriod="60 seconds" debug="false">
    <contextName>logback</contextName>
    <property name="log.path" value="/var/log"/>
    <property name="app.name" value="APP-NAME"/>
    <property name="log.pattern" value="%red(%d{yyyy-MM-dd HH:mm:ss}) %green([%thread]) %highlight(%-5level) %boldMagenta(%logger{10}) - %cyan(%msg%n)"/>
    <!--输出到控制台-->
    <appender name="console" class="ch.qos.logback.core.ConsoleAppender">
        <!-- <filter class="ch.qos.logback.classic.filter.ThresholdFilter">
             <level>ERROR</level>
         </filter>-->
        <encoder>
            <pattern>${log.pattern}</pattern>
            <!--<pattern>%d{HH:mm:ss.SSS} %contextName [%thread] %-5level %logger{36} - %msg%n</pattern>-->
        </encoder>
    </appender>

    <!--输出到文件1. 不限定大小-->
    <appender name="file" class="ch.qos.logback.core.rolling.RollingFileAppender">
        <rollingPolicy class="ch.qos.logback.core.rolling.TimeBasedRollingPolicy">
            <fileNamePattern>${log.path}/${app.name}.%d{yyyy-MM-dd}.log</fileNamePattern>
        </rollingPolicy>
        <encoder>
            <pattern>${log.pattern}</pattern>
            <!--<pattern>%d{HH:mm:ss.SSS} %contextName [%thread] %-5level %logger{36} - %msg%n</pattern>-->
        </encoder>
    </appender>
    
    <!--输出到文件2. 限定大小，保留天数，最大空间占用-->
    <appender name="file" class="ch.qos.logback.core.rolling.RollingFileAppender">
        <rollingPolicy class="ch.qos.logback.core.rolling.SizeAndTimeBasedRollingPolicy">
            <fileNamePattern>${log.path}/${app.name}.%d{yyyy-MM-dd}.%i.log</fileNamePattern>
            <!--日志文件保留天数-->
            <MaxHistory>30</MaxHistory>
            <maxFileSize>128MB</maxFileSize>
            <totalSizeCap>1024MB</totalSizeCap>
        </rollingPolicy>
        <encoder>
            <pattern>${log.pattern}</pattern>
            <!--<pattern>%d{HH:mm:ss.SSS} %contextName [%thread] %-5level %logger{36} - %msg%n</pattern>-->
        </encoder>
    </appender>

    <root level="info">
        <appender-ref ref="console"/>
        <appender-ref ref="file"/>
    </root>
</configuration>
```

## 异常

###  The server sockets created using the LocalRMIServerSocketFactory

**日志错误**

```
AM sun.rmi.transport.tcp.TCPTransport$AcceptLoop executeAcceptLoop
WARNING: RMI TCP Accept-0: accept loop for ServerSocket[addr=0.0.0.0/0.0.0.0,localport=32658]s
java.io.IOException: The server sockets created using the LocalRMIServerSocketFactory only aconnections from clients running on the host where the RMI remote objects have been exported.
	at sun.management.jmxremote.LocalRMIServerSocketFactory$1.accept(LocalRMIServerSockety.java:114)
	at sun.rmi.transport.tcp.TCPTransport$AcceptLoop.executeAcceptLoop(TCPTransport.java:
	at sun.rmi.transport.tcp.TCPTransport$AcceptLoop.run(TCPTransport.java:372)
	at java.lang.Thread.run(Thread.java:745)
```

**解决方法**

```shell
1) fix etc hosts or 2) disable local host checks.
Disabling local host checking can be done in two ways:
a) system-wide: uncomment the line
  # com.sun.management.jmxremote.local.only=false
  in jre/lib/management/management.properties
b) process based: pass -Dcom.sun.management.jmxremote.local.only=false
   on the java command line (attachee side)
```

## Maven基础

```shell
# 不执行测试用例，也不编译测试用例类
mvn package -Dmaven.test.skip=true
# 不执行测试用例，但编译测试用例类生成相应的class文件至target/test-classes下
mvn package -DskipTests

# mvn源更新后，刷新本地缓存
mvn clean package -U
```



[^1]: springboot 2.X 配置SSL证书