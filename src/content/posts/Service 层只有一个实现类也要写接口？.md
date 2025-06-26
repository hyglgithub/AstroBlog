---
title: Service 层只有一个实现类也要写接口？
published: 2025-06-20
description: 本文将探讨“为什么只有一个实现类也需要接口”，并逐步引出接口 + 实现类的典型用途——策略模式。我们将通过两个典型的实战案例：“登录策略”和“支付策略”，讲解策略模式在 Spring 中的落地方式。
tags: [spring boot, java]
category: 原创
draft: false
---

## 一、为什么 Service 层要写接口 + 实现类？

在实际开发中，我们常看到这样的结构：

```java
public interface UserService {
    User getUserById(Long id);
}

@Service
public class UserServiceImpl implements UserService {
    @Override
    public User getUserById(Long id) {
        return userMapper.selectById(id);
    }
}
```

虽然现在只有一个实现类，定义接口的作用也十分重要：

* **解耦调用方和实现逻辑**：接口定义了服务契约，调用方只需关注接口，而无需了解具体的实现细节，这使得替换底层实现变得更加容易。
* **便于单元测试**：通过接口，我们可以轻松注入 Mock 实现，从而在单元测试中隔离 Service 层的依赖，提高测试的独立性和效率。
* **支持多策略扩展**：接口为未来可能的多种实现提供了扩展点，配合 Spring 容器的自动装配能力，可以方便地实现多策略的动态切换。
* **规范开发结构**：定义接口有助于规范团队的开发结构，使代码更加清晰、易于理解和维护，对团队协作更友好。

因此，即使当前仅一个实现类，也建议预留接口，这是“**面向扩展编程**”的体现。

## 二、接口 + 实现类能实现策略模式

**策略模式**的核心思想是：将一组具有相同行为特征但实现方式不同的策略封装成独立类，在运行时根据不同情况选择合适的策略执行。

在 Spring 中，我们可以通过接口定义策略行为，通过多个 `@Component` 实现不同的策略，最后在业务层（如 Service）中通过注入 `Map<String, Bean>` 或 `List<Bean>` 来动态选择具体策略。

这种模式的优点包括：

* **解耦 if-else 分支判断**：避免了大量的 `if-else` 或 `switch-case` 语句，使代码更简洁、可读性更高。
* **新增策略无需改动旧代码**：符合**开闭原则 (OCP)**，即对扩展开放，对修改关闭。当需要增加新的策略时，只需新增一个实现类，无需修改现有代码。
* **配合 Spring 注解，扩展灵活**：Spring 强大的依赖注入能力使得策略的注册和选择变得非常方便。


## 三、接口 + 实现类的两个实际例子

### 示例一：登录策略（使用 List 注入）

这个示例支持多种登录方式，如用户名密码登录、手机号验证码登录等。

1. **定义登录接口**

   ```java
   public interface LoginService {
       boolean supports(String loginType); // 是否支持某种登录方式
       String login(Map<String, String> loginParam);
   }
   ```

2. **用户名密码登录实现类**

   ```java
   @Service
   public class UsernamePasswordLoginService implements LoginService {
       @Override
       public boolean supports(String loginType) {
           return "username".equalsIgnoreCase(loginType);
       }

       @Override
       public String login(Map<String, String> loginParam) {
           return "账号密码登录成功: " + loginParam.get("username");
       }
   }
   ```

3. **手机验证码登录实现类**

   ```java
   @Service
   public class PhoneCodeLoginService implements LoginService {
       @Override
       public boolean supports(String loginType) {
           return "phone".equalsIgnoreCase(loginType);
       }

       @Override
       public String login(Map<String, String> loginParam) {
           return "手机验证码登录成功: " + loginParam.get("phone");
       }
   }
   ```

4. **策略选择器（List 注入）**

   ```java
   @Service
   public class LoginServiceFactory {
       private final List<LoginService> loginServices;

       public LoginServiceFactory(List<LoginService> loginServices) {
           this.loginServices = loginServices;
       }

       public LoginService getService(String loginType) {
           for (LoginService service : loginServices) {
               if (service.supports(loginType)) {
                   return service;
               }
           }
           throw new IllegalArgumentException("不支持的登录方式: " + loginType);
       }
   }
   ```

5. **Controller 示例**

   ```java
   @RestController
   @RequestMapping("/login")
   public class LoginController {
       private final LoginServiceFactory factory;

       public LoginController(LoginServiceFactory factory) {
           this.factory = factory;
       }

       @PostMapping
       public String login(@RequestParam String loginType,
                           @RequestParam Map<String, String> loginParam) {
           return factory.getService(loginType).login(loginParam);
       }
   }
   ```

6. **测试示例**

* 用户名密码登录：
  请求：`POST /login?loginType=username`
  参数体：`username=alice`
  返回：`账号密码登录成功: alice`

* 手机验证码登录：
  请求：`POST /login?loginType=phone`
  参数体：`phone=13800000000`
  返回：`手机验证码登录成功: 13800000000`


### 示例二：支付策略（使用 Map 注入）

这个示例支持多种支付方式，如支付宝、微信、银行卡等。

1. **定义支付接口**

   ```java
   public interface PayStrategy {
       String pay(String orderId, double amount);
   }
   ```

2. **支付策略实现类（带名字注入）**

   ```java
   @Component("alipay")
   public class AliPayStrategy implements PayStrategy {
       public String pay(String orderId, double amount) {
           return "支付宝支付成功，订单：" + orderId + "，金额：" + amount;
       }
   }

   @Component("wechatpay")
   public class WechatPayStrategy implements PayStrategy {
       public String pay(String orderId, double amount) {
           return "微信支付成功，订单：" + orderId + "，金额：" + amount;
       }
   }

   @Component("bankcardpay")
   public class BankCardPayStrategy implements PayStrategy {
       public String pay(String orderId, double amount) {
           return "银行卡支付成功，订单：" + orderId + "，金额：" + amount;
       }
   }
   ```

3. **支付服务类（Map 注入）**

   ```java
   @Service
   public class PayService {
       private final Map<String, PayStrategy> strategyMap;

       public PayService(Map<String, PayStrategy> strategyMap) {
           this.strategyMap = strategyMap;
       }

       public String pay(String type, String orderId, double amount) {
           PayStrategy strategy = strategyMap.get(type);
           if (strategy == null) {
               throw new IllegalArgumentException("不支持的支付方式：" + type);
           }
           return strategy.pay(orderId, amount);
       }
   }
   ```

4. **Controller 示例**

   ```java
   @RestController
   @RequestMapping("/pay")
   public class PayController {
       private final PayService payService;

       public PayController(PayService payService) {
           this.payService = payService;
       }

       @GetMapping
       public String pay(@RequestParam String type,
                         @RequestParam String orderId,
                         @RequestParam double amount) {
           return payService.pay(type, orderId, amount);
       }
   }
   ```

5. **测试示例**

* **支付宝支付**：
  `GET /pay?type=alipay&orderId=123&amount=100.0`
* **微信支付**：
  `GET /pay?type=wechatpay&orderId=456&amount=200.0`
* **银行卡支付**：
  `GET /pay?type=bankcardpay&orderId=789&amount=300.0`


## 四、List 注入和 Map 注入的区别

| 对比项        | Map 注入方式                              | List 注入方式                  |
| :--------- | :------------------------------------ | :------------------------- |
| **注入方式**   | `Map<String, Bean>`                   | `List<Bean>`               |
| **键值规则**   | `key` 为 Bean 名称（`@Component("name")`） | Spring 自动按注册顺序注入           |
| **策略选择方式** | 直接通过 `key` 查找对应策略，效率更高                | 遍历 List，调用 `supports()` 判断 |
| **灵活性**    | 结构固定，适合简单枚举型策略切换                      | 支持复杂条件逻辑，更灵活               |
| **推荐使用场景** | 策略类型明确，`key` 和实现一一对应                  | 策略种类多、判断逻辑复杂、需组合判断等场景      |


## 五、总结

Spring Boot 的 Service 层使用接口 + 实现类是为了实现**解耦、扩展、测试友好**以及**支持策略模式**。策略模式在实际业务中非常实用，如登录、支付、通知等场景。可选的注入方式包括 `Map<String, Bean>`（用于快速定位策略）和 `List<Bean>`（用于灵活支持多条件判断）。使用策略模式后，新增策略只需添加类，而 Controller 和主业务逻辑无需修改，这完全符合**开闭原则 (OCP)**。
