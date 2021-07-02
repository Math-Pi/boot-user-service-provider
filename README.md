# Dubbo 集成SpringBoot

## 一、API(分包)

> 建议将服务接口、服务模型、服务异常等均放在 API 包中，因为服务模型和异常也是 API 的一部分，这样做也符合分包原则：重用发布等价原则(REP)，共同重用原则(CRP)。

- bean
  - UserAddress
- service
  - OrderService
  - UserService

## 二、服务提供者

### 2.1 引入Maven依赖

1. API包
2. web开发包
3. dubbo集成SpringBoot包

`pom.xml`

```xml
<dependencies>
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter</artifactId>
    </dependency>

    <dependency>
        <groupId>com.cjm.gmall</groupId>
        <artifactId>gmall-interface</artifactId>
        <version>0.0.1-SNAPSHOT</version>
    </dependency>
	<!-- web开发包 -->
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-web</artifactId>
        <version>2.5.2</version>
    </dependency>
    <!-- dubbo集成SpringBoot包 -->
    <dependency>
        <groupId>com.alibaba.boot</groupId>
        <artifactId>dubbo-spring-boot-starter</artifactId>
        <version>0.2.0</version>
    </dependency>

    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-test</artifactId>
        <scope>test</scope>
    </dependency>
</dependencies>
```

### 2.2 编写实现类

`UserServiceImpl`

```java
/**
 * 1、将服务提供者注册到注册中心
 * 		1）导入dubbo依赖（2.6.2）、操作zookeeper的客户端依赖(curator)
 * 		2）配置服务提供者
 * 2、让服务消费者去注册中心订阅服务提供者的服务地址
 * @author 陈嘉名
 *
 */
@Service
@Component
public class UserServiceImpl implements UserService {

	@Override
	public List<UserAddress> getUserAddressList(String userId) {
		System.out.println("UserServiceImpl.....old...");
		// TODO Auto-generated method stub
		UserAddress address1 = new UserAddress(1, "北京市昌平区宏福科技园综合楼3层", "1", "李老师", "010-56253825", "Y");
		UserAddress address2 = new UserAddress(2, "深圳市宝安区西部硅谷大厦B座3层（深圳分校）", "1", "王老师", "010-56253825", "N");
		/*try {
			Thread.sleep(4000);
		} catch (InterruptedException e) {
			// TODO Auto-generated catch block
			e.printStackTrace();
		}*/
		return Arrays.asList(address1,address2);
	}

}
```

注意：@Service是`com.alibaba.dubbo.config.annotation.Service`用于dubbo暴露服务而非`org.springframework.stereotype.Service`用于注入Spring容器

### 2.3 编写控制类

```java
@Controller
public class OrderController {

	@Autowired
	private OrderService orderService;
    
	@RequestMapping("/initOrder")
	@ResponseBody
	public List<UserAddress> initOrder(@RequestParam("uid") String userId) {
		return orderService.initOrder(userId);
	}
}
```

### 2.4 application.properties

1. 服务名：`dubbo.application.name`
2. 注册中心：协议-`dubbo.registry.protocol`;地址-`dubbo.registry.address`
3. 通信协议：协议名-`dubbo.protocol.name`;端口-`dubbo.protocol.port`
4. 暴露服务：`@com.alibaba.dubbo.config.annotation.Service`

`application.properties`

```properties
dubbo.application.name=user-service-provider
dubbo.registry.address=127.0.0.1:2181
dubbo.registry.protocol=zookeeper

dubbo.protocol.name=dubbo
dubbo.protocol.port=20880

dubbo.monitor.protocol=registry
```

### 2.5 启动类

> 添加`@EnableDubbo`主键开启基于注解的dubbo功能

`BootUserServiceProviderApplication`

```java
@EnableDubbo	//开启基于注解的dubbo功能
@SpringBootApplication
public class BootUserServiceProviderApplication {

	public static void main(String[] args) {
		SpringApplication.run(BootUserServiceProviderApplication.class, args);
	}

}
```

## 三、服务消费者

### 3.1 引入Maven依赖

1. API包
2. dubbo集成SpringBoot包

`pom.xml`

```xml
<dependencies>
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter</artifactId>
    </dependency>

    <dependency>
        <groupId>com.cjm.gmall</groupId>
        <artifactId>gmall-interface</artifactId>
        <version>0.0.1-SNAPSHOT</version>
    </dependency>

    <!-- dubbo集成SpringBoot包 -->
    <dependency>
        <groupId>com.alibaba.boot</groupId>
        <artifactId>dubbo-spring-boot-starter</artifactId>
        <version>0.2.0</version>
    </dependency>

    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-test</artifactId>
        <scope>test</scope>
    </dependency>
</dependencies>
```

### 3.2 编写实现类

`OrderUserviceImpl`

```java
/**
 * 1、将服务提供者注册到注册中心
 * 		1）导入dubbo依赖（2.6.2）、操作zookeeper的客户端依赖(curator)
 * 		2）配置服务提供者
 * 2、让服务消费者去注册中心订阅服务提供者的服务地址
 * @author 陈嘉名
 *
 */
@Service
public class OrderUserviceImpl implements OrderService{

	@Reference
	private UserService userService;
	@Override
	public List<UserAddress> initOrder(String userId) {
		// TODO Auto-generated method stub
		System.out.println("用户id："+userId);
		//1、查询用户的收货地址
		List<UserAddress> addressList= userService.getUserAddressList(userId);
		return addressList;
	}
}
```

注意：`@Reference`注入的是分布式中的远程服务对象，@Resource和@Autowired注入的是本地spring容器中的对象。

### 3.3 application.properties

1. 服务名：`dubbo.application.name`
2. 注册中心：协议-`dubbo.registry.protocol`;地址-`dubbo.registry.address`
3. 引用服务：`@com.alibaba.dubbo.config.annotation.Reference`

`application.properties`

```properties
server.port=8081

dubbo.application.name=boot-order-service-consumer
dubbo.registry.address=zookeeper://127.0.0.1:2181
dubbo.monitor.protocol=registry
```

### 3.4 启动类

`BootOrderServiceConsumerApplication`

```java
@EnableDubbo
@SpringBootApplication
public class BootOrderServiceConsumerApplication {

	public static void main(String[] args) {
		SpringApplication.run(BootOrderServiceConsumerApplication.class, args);
	}

}
```

