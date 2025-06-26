---
title: Java泛型擦除的问题与解决方法
published: 2025-05-20
description: 反射和序列化，泛型擦除会导致问题，例如反序列化时无法推断出泛型的真实类型。为解决这一问题，可以使用如 Spring 的 ParameterizedTypeReference 来保留泛型信息，确保框架能正确处理泛型类型。
image: ''
tags: [java, 泛型]
category: '原创'
draft: false 
---


Java 的泛型是一种“语法糖”，它的设计理念是：编译时类型检查，运行时类型擦除。这带来了类型安全，但也引发了一些实际问题。下面详细讲解泛型擦除的原理、常见问题及解决办法。

1. 编译时类型检查  
在编译阶段，Java 编译器会根据泛型参数帮你做类型检查。例如：

````java
List<String> list = new ArrayList<>();
list.add("hello");    // 编译通过
list.add(123);        // 编译报错，类型不匹配
````

此时，<String> 只是编译器用来检查类型的“提示”，不会进入字节码。

2. 运行时类型擦除  
Java 编译后，泛型类型信息会被“擦除”：
- List<String> 变成 List
- Map<Integer, String> 变成 Map

运行时，JVM 并不知道泛型的具体类型，只知道原始类型。例如：

````java
List<String> list = new ArrayList<>();
list.add("hello");
String s = list.get(0);
````

JVM 只知道 list 是个 List，get(0) 返回的是 Object，编译器会自动插入类型转换代码。

3. 普通代码中，泛型擦除通常无影响  
大多数情况下，泛型擦除不会影响代码运行，因为类型检查和转换都在编译阶段完成了。

4. 反射和序列化时，泛型擦除导致问题  
但在反射和序列化框架（如 Jackson）中，运行时需要确切的类型信息。如果泛型被擦除，框架无法推断出泛型的真实类型，导致反序列化失败。

典型问题举例  
假设有如下泛型类：

````java
@Data
public class ApiResponse<T> {
    private int code;
    private String message;
    private T data;
}
````

你这样调用：

````java
ApiResponse<List<String>> response = restTemplate.exchange(
    url,                        // 请求的 URL 地址
    HttpMethod.GET,             // 请求方法是 GET
    new HttpEntity<>(headers),  // 请求头（HttpEntity 封装了 headers）
    ApiResponse.class           // 指定响应体类型（⚠️ 这里是关键）
).getBody();                    // 获取响应体内容
````

此时，Jackson 只知道你要反序列化成 ApiResponse，但不知道 T 是什么类型。于是 data 字段会被反序列化成 LinkedHashMap 或 ArrayList<Object>，而不是你期望的 List<String>。

5. 解决方法：保留泛型类型信息  
正确做法是明确告诉框架泛型的具体类型。Spring 提供了 ParameterizedTypeReference，可以保留泛型信息：

````java
ApiResponse<List<String>> response = restTemplate.exchange(
    url,
    HttpMethod.GET,
    new HttpEntity<>(headers),
    new ParameterizedTypeReference<ApiResponse<List<String>>>() {}
).getBody();
````

这样，Jackson 就能正确反序列化 data 字段为 List<String>。

6. 总结

| 场景             | 是否需要运行时泛型信息 | 泛型擦除影响           |
|------------------|----------------------|------------------------|
| 普通代码编译执行 | 否                   | 无影响                 |
| 反射、序列化     | 是                   | 需补救手段，否则失败   |

一句话总结：
- 只用 ApiResponse.class，泛型被擦除，反序列化失败，data 是 LinkedHashMap 或 Object
- 用 new ParameterizedTypeReference<ApiResponse<List<String>>>() {}，泛型信息保留，data 正确变成 List<String>

核心记忆：  
Java 泛型只在编译期有效，运行时会被擦除。遇到反射、序列化等需要泛型信息的场景，必须用特殊写法（如 ParameterizedTypeReference）保留泛型类型。