---
title: SpringBoot+Dubbo简单demo实践
date: 2021-07-02 13:56:41
tags:
    - Dubbo
categories: 
    - Java
    - 框架
    - Dubbo
---
# SpringBoot + Dubbo 简单 demo 实践
休息了一个月终于又要上班了，下家公司的项目用的是Dubbo+Zookeeper，由于之前只用过Spring Cloud，所以提前了解一下Dubbo的使用，搭了个简单的 demo 感受 Dubbo 和 Spring Cloud 的区别。
大概流程如下：
1. Zookeeper搭建
2. Dubbo可视化管理界面搭建
3. 接口层定义
4. 服务层实现
5. 消费层调用

## Dubbo基本工作原理
Dubbo 是一款 RPC 服务框架，它最大的优势在于提供了面向接口代理的服务编程模型，对开发者屏蔽了底层的远程通信细节。同时 Dubbo 也是一款服务治理框架，它为分布式部署的微服务提供了服务发现、流量调度等服务治理解决方案。

下图是 Dubbo 的基本工作原理图(在官网找的)，服务提供者与服务消费者之间通过注册中心协调地址，通过约定的协议实现数据交换。
![Dubbo 基本工作原理图](https://img-blog.csdnimg.cn/20210702113353557.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80MjEwMTE1NQ==,size_16,color_FFFFFF,t_70#pic_center)
接下来就搭建一个 demo ，感受一下 Dubbo 的使用和 Spring Cloud 有什么区别。

## Zookeeper搭建
### Zookeeper下载
Dubbo 使用 Zookeeper 作为注册中心，提供服务注册和发现，跟Spring Cloud的 eureka 或 nacos 类似。

所以第一步当然是去[官网](https://zookeeper.apache.org/)下载 Zookeeper 啦，我下载的版本是3.6.3。

### 修改配置
下载解压后，进入 conf 文件夹可以看到有个 zoo_sample.cfg 文件，这个是配置文件示例，现在复制并更名为 zoo.cfg ，根据自己的需要对配置进行更改，我改了一下数据路径。
Zookeeper 的端口号默认是2181，可以根据自己的需要对 clientPort 的值进行更改。

```
dataDir=E:/zookeeper/data
clientPort=2181
```
### 启动
进入bin文件夹可以看到里面有很多脚本，zkServer.cmd 和 zkServer.sh 就是启动脚本，我的电脑系统是 windows ，所以直接双击 zkServer.cmd 即可启动。

![启动Zookeeper](https://img-blog.csdnimg.cn/20210702113029700.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80MjEwMTE1NQ==,size_16,color_FFFFFF,t_70#pic_center)

## Dubbo 可视化管理界面搭建
Zookeeper 启动完成之后，我们需要一个可视化界面方便对服务进行观察和管理。

### 下载项目
Dubbo-admin的Github地址是 https://github.com/apache/dubbo-admin ，直接 git clone 下来。项目里有 readme.md 介绍启动流程，参考 readme.md 启动项目。

### 修改配置
打开 dubbo-admin\dubbo-admin-server\src\main\resources\application.properties 可以修改配置，指定 Zookeeper 地址，我的 Zookeeper 是默认配置，所以不用修改。

### 编译、打包项目
进入到项目里编译并打包项目。

```
mvn clean package -Dmaven.test.skip=true
```

### 启动项目
可以直接在项目根目录运行 mvn --projects dubbo-admin-server spring-boot:run 启动项目，也可以进入 dubbo-admin-distribution\target 运行里面的jar包。

![启动 dubbo-admin](https://img-blog.csdnimg.cn/20210702120409974.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80MjEwMTE1NQ==,size_16,color_FFFFFF,t_70#pic_center)



** 启动失败 **
我刚开始启动失败了，这个 dubbo-admin 的启动端口是 8080，但是我发现 8080 端口已经被占用了，最后发现是 Zookeeper 占用的，原来是 Zookeeper 内嵌了一个管理控制台，通过 jetty 启动，这个 jetty 的端口就是 8080，于是我去改了以下 Zookeeper 的 zoo.cfg ，新增了一行配置把 Zookeeper 的管理控制台端口改为 8081。

```
admin.serverPort=8081
```

### 进入管理界面
启动成功后就可以访问 http://127.0.0.1:8080 了，默认的账号密码都是root。
![登录](https://img-blog.csdnimg.cn/20210702120820273.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80MjEwMTE1NQ==,size_16,color_FFFFFF,t_70#pic_center)
![Dubbo Admin](https://img-blog.csdnimg.cn/20210702120905244.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80MjEwMTE1NQ==,size_16,color_FFFFFF,t_70#pic_center)
## 工程创建
接下来就要创建工程了，先创建个 maven 多模块工程，里面需要包含三个模块，一个是 api 模块，一个是服务提供者模块，一个是服务消费者模块。

### api模块
假设我这个服务是一个用户查询的服务，那我就需要先创建一个 dto 和一个接口。
![api 模块](https://img-blog.csdnimg.cn/20210702122334353.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80MjEwMTE1NQ==,size_16,color_FFFFFF,t_70)
#### 创建 UserInfo 类
创建一个用户信息类，需要注意的是，dto 要实现 Serializable 接口进行序列化。当然，也可以使用其它的序列化方式，hessian2、json等。

```
public class UserInfo implements Serializable {
    private static final long serialVersionUID = 2908612893636115165L;

    private String userName;

    private Long userId;

    public static long getSerialVersionUID() {
        return serialVersionUID;
    }

    public String getUserName() {
        return userName;
    }

    public void setUserName(String userName) {
        this.userName = userName;
    }

    public Long getUserId() {
        return userId;
    }

    public void setUserId(Long userId) {
        this.userId = userId;
    }
}
```

#### 定义用户服务接口
简单定义一个接口。

```
public interface UserService {
    public List<UserInfo> getUserInfoList();
}
```

### 服务提供者模块
现在创建一个服务提供者模块。

#### 依赖
这个模块需要 Dubbo 和 Zookeeper 相关依赖，再加上这个模块是要实现 UserService 接口，所以也需要刚才创建的 api 模块依赖。

```
    <properties>
        <java.version>1.8</java.version>
        <dubbo.version>2.7.5</dubbo.version>
        <curator.version>2.12.0</curator.version>
    </properties>
    <dependencies>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter</artifactId>
        </dependency>

        <dependency>
            <groupId>org.apache.dubbo</groupId>
            <artifactId>dubbo-spring-boot-starter</artifactId>
            <version>${dubbo.version}</version>
        </dependency>

        <!-- Zookeeper dependencies -->
        <dependency>
            <groupId>org.apache.dubbo</groupId>
            <artifactId>dubbo-dependencies-zookeeper</artifactId>
            <version>${dubbo.version}</version>
            <type>pom</type>
            <exclusions>
                <exclusion>
                    <groupId>org.slf4j</groupId>
                    <artifactId>slf4j-log4j12</artifactId>
                </exclusion>
            </exclusions>
        </dependency>

        <dependency>
            <groupId>org.apache.curator</groupId>
            <artifactId>curator-framework</artifactId>
            <version>${curator.version}</version>
        </dependency>

        <!-- https://mvnrepository.com/artifact/org.apache.curator/curator-recipes -->
        <dependency>
            <groupId>org.apache.curator</groupId>
            <artifactId>curator-recipes</artifactId>
            <version>${curator.version}</version>
        </dependency>

        <dependency>
            <groupId>com.handsomedong</groupId>
            <artifactId>dubbo-api</artifactId>
            <version>0.0.1-SNAPSHOT</version>
        </dependency>

    </dependencies>
```

#### 配置
配置文件主要需要以下配置：
![配置文件](https://img-blog.csdnimg.cn/20210702124035217.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80MjEwMTE1NQ==,size_16,color_FFFFFF,t_70)


根据上图的配置，修改 application.yml，我的配置如下：
```
server:
  port: 9000

debug: true

dubbo:
  application:
    name: user-service-provider
  registry:
    address: 127.0.0.1:2181
    protocol: zookeeper
  protocol:
    name: dubbo
    port: 19000
  monitor:
    protocol: registry
```

#### 用户服务类具体实现
实现用户服务接口，注意这里的 @Service 注解，不是 Spring 的注解，而是 Dubbo 的注解！包的全路径是 org.apache.dubbo.config.annotation.Service。
并且需要注意的是，它必须要实现 UserService 接口。

```
@Service
@Component
public class UserServiceImpl implements UserService {
    @Override
    public List<UserInfo> getUserInfoList() {
        List<UserInfo> userInfoList = new ArrayList<>();
        for (int i = 0; i < 10; i++) {
            UserInfo userInfo = new UserInfo();
            userInfo.setUserId(Integer.valueOf(i).longValue());
            userInfo.setUserName("七里翔" + i);
            userInfoList.add(userInfo);
        }

        return userInfoList;
    }
}
```

#### SpringBoot 启动类
最后在启动类要加上 @EnableDubbo 注解。

```
@EnableDubbo
@SpringBootApplication
public class UserProviderApplication {

    public static void main(String[] args) {
        SpringApplication.run(UserProviderApplication.class, args);
    }

}
```

#### 查看 Dubbo Admin
现在再去 Dubbo Admin 上面就可以看到这个服务注册成功了！点击详情也可以看到服务的具体信息。
![Dubbo Admin](https://img-blog.csdnimg.cn/20210702124820859.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80MjEwMTE1NQ==,size_16,color_FFFFFF,t_70)
![服务详情](https://img-blog.csdnimg.cn/20210702124935459.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80MjEwMTE1NQ==,size_16,color_FFFFFF,t_70)
### 服务消费者模块
最后就是创建服务消费者模块了，在该模块下进行服务调用。

#### 依赖
依赖基本和服务提供者相同，这个就不多BB了。

```
    <properties>
        <java.version>1.8</java.version>
        <dubbo.version>2.7.5</dubbo.version>
        <curator.version>2.12.0</curator.version>
    </properties>
    <dependencies>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter</artifactId>
        </dependency>

        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-web</artifactId>
        </dependency>

        <dependency>
            <groupId>org.apache.dubbo</groupId>
            <artifactId>dubbo-spring-boot-starter</artifactId>
            <version>${dubbo.version}</version>
        </dependency>

        <!-- Zookeeper dependencies -->
        <dependency>
            <groupId>org.apache.dubbo</groupId>
            <artifactId>dubbo-dependencies-zookeeper</artifactId>
            <version>${dubbo.version}</version>
            <type>pom</type>
            <exclusions>
                <exclusion>
                    <groupId>org.slf4j</groupId>
                    <artifactId>slf4j-log4j12</artifactId>
                </exclusion>
            </exclusions>
        </dependency>

        <dependency>
            <groupId>org.apache.curator</groupId>
            <artifactId>curator-framework</artifactId>
            <version>${curator.version}</version>
        </dependency>

        <!-- https://mvnrepository.com/artifact/org.apache.curator/curator-recipes -->
        <dependency>
            <groupId>org.apache.curator</groupId>
            <artifactId>curator-recipes</artifactId>
            <version>${curator.version}</version>
        </dependency>

        <dependency>
            <groupId>com.handsomedong</groupId>
            <artifactId>dubbo-api</artifactId>
            <version>0.0.1-SNAPSHOT</version>
        </dependency>
    </dependencies>
```

#### 配置
application.yml

```
server:
  port: 10000
debug: true

dubbo:
  application:
    name: user-service-consumer
  registry:
    address: zookeeper://127.0.0.1:2181
  monitor:
    protocol: registr

```

#### 服务调用
我这里直接创建个 Controller 来测试。

```
@RestController
@RequestMapping("test")
public class TestController {
    @Reference	//dubbo
    UserService userService;

    @GetMapping("test")
    public List<UserInfo> test() {
        List<UserInfo> userInfoList = userService.getUserInfoList();
        return userInfoList;
    }

}
```

#### SpringBoot 启动类
这个启动类同样要加上 @EnableDubbo 注解。

```
@EnableDubbo
@SpringBootApplication
public class UserConsumerApplication {

    public static void main(String[] args) {
        SpringApplication.run(UserConsumerApplication.class, args);
    }

}
```

### 测试
启动服务消费者后，断点调试、用 postman 测试一下。
![断点调试](https://img-blog.csdnimg.cn/2021070213174837.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80MjEwMTE1NQ==,size_16,color_FFFFFF,t_70#pic_center)

![postman](https://img-blog.csdnimg.cn/20210702131812576.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80MjEwMTE1NQ==,size_16,color_FFFFFF,t_70)
可以看到已经大功告成了！！！

## 总结
### Dubbo 和 Spring Cloud 区别
1. Dubbo 耦合较高，有一定约束。这是我使用中感受到的最大的区别，Dubbo 需要我们定义个抽象接口，然后提供方实现这个接口，消费方再通过这个接口进行调用。提供方和消费方都依赖于这个接口的模块，所以整个项目是有一定的耦合的。而Spring Cloud 我们使用 Feign 就比较自由，比较轻量化，通过 REST 调用，更灵活。这不能单纯地说是优点或者是缺点吧，毕竟接口定义过轻也很容易多版本的情况下导致接口文档与实际所展现功能不一致导致服务集成时的问题，或许对版本控制比较严格有时候反而会更加稳定。
2. Dubbo底层是使用Netty这样的NIO框架，是基于TCP协议传输的；而 Spring Cloud 是基于 Http 协议+ REST 接口调用远程过程的通信。在传输协议和序列化方式上，Dubbo 的性能要比 Spring Cloud 更高。因为 Dubbo 是长连接并且用二进制传输，占用带宽更少。而 Spring Cloud 使用的是短连接并且使用 JSON 序列化，消耗更大。
3. Dubbo 只是一个 RPC 框架，而 Spring Cloud 是一整套微服务的解决方案，因此它们的领域是不一样的。
4. 社区方面，Spring Cloud 要比 Dubbo 活跃很多很多很多很多……Dubbo 好久没更新了，而 Spring Cloud 目前还一直在更新，很活跃！


![Dubbo 流泪](https://img-blog.csdnimg.cn/20210702134836992.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80MjEwMTE1NQ==,size_16,color_FFFFFF,t_70)
