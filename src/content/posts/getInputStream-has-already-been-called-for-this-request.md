---
title: getInputStream() has already been called for this request 异常原因及解决方案
published: 2025-04-30
description: >
  在Spring Web开发中，IllegalStateException: getInputStream() has already been called for this request异常常因重复读取请求体引发。
  由于Servlet规范限制，请求体只能被读取一次。解决方案包括：
  1) 使用Spring的ContentCachingRequestWrapper缓存请求数据；
  2) 自定义HttpServletRequestWrapper实现懒加载缓存。
  两种方法都能避免流被重复读取，确保请求体只获取一次并支持多次访问。
tags: [java, http, spring]
category: '原创'
draft: false
---

### 一、问题描述
在 Spring Web 项目开发中，我们可能会遇到如下异常：
```java
java.lang.IllegalStateException: getInputStream() has already been called for this request
```
这种异常通常发生在对 `HttpServletRequest` 对象重复读取请求体（Request Body）时。无论是使用 `getInputStream()` 还是 `getReader()` 方法，请求体在一次请求中只能被读取一次，这源于 Servlet 规范的设计。

一个常见的场景是：  
在 Spring Security 中，`UsernamePasswordAuthenticationFilter` 等认证过滤器通常会在请求体被读取之前执行。这样设计的目的是为了高效的认证过程。然而，如果认证过滤器在认证过程中读取了请求体，后续的 Spring 组件（如参数绑定、认证、过滤等）也会尝试读取请求体，这就可能会遇到流已被消耗的问题。

### 问题原因
在 Spring 框架中，`HttpServletRequest` 的输入流（通过 `getInputStream()` 或 `getReader()` 访问）是一次性资源。这意味着一旦被读取，就不能再次读取。如果您在 Spring 处理请求之前就读取了这个流，Spring 将无法再次读取，从而导致异常。

根据 Servlet 规范，`getInputStream()` 和 `getReader()` 方法只能选择其中一个调用，并且只能调用一次。如果在请求生命周期中已经调用了其中一个方法，再次调用将抛出 `IllegalStateException` 异常。

Spring 框架在处理请求时，通常需要读取请求体来完成参数绑定、认证、安全过滤、异常处理等功能。因此，第一次读取请求体的操作应由 Spring 自行控制。如果您在 Spring 处理请求之前就读取了请求体，Spring 将无法再次读取，从而导致异常。

为了避免这种情况，建议使用 Spring 提供的 `ContentCachingRequestWrapper` 类，它会在第一次读取请求体时缓存内容，之后的读取操作将使用缓存的数据，而不会再次读取原始流。这样可以确保请求体只被读取一次，同时允许多次访问请求体内容。

因此，为了确保 Spring 框架能够正常处理请求，您应避免在 Spring 处理请求之前读取 `HttpServletRequest` 的输入流。

### 解决方案（使用 Spring 多次读取 `HttpServletRequest` 的正文）
#### 1. 使用 `ContentCachingRequestWrapper`
Spring 提供了一个现成的工具类来防止这种问题，即 `org.springframework.web.util.ContentCachingRequestWrapper`。它内部缓存了请求体数据，即使多次调用 `getInputStream()` 或 `getReader()` 也不会抛出错误。

您可以在 `Filter` 中将请求包装为 `ContentCachingRequestWrapper`，例如：
```java
import org.springframework.web.util.ContentCachingRequestWrapper;

@Override
protected void doFilterInternal(HttpServletRequest request, HttpServletResponse response, FilterChain filterChain)
        throws ServletException, IOException {
    ContentCachingRequestWrapper wrappedRequest = new ContentCachingRequestWrapper(request);
    filterChain.doFilter(wrappedRequest, response);
}
```
⚡ **注意**：一定要在调用 `getInputStream()` 或 `getReader()` 之前将请求包装。

#### 2. 自己实现 `RequestWrapper`
如果您希望自定义 `RequestWrapper`，可以实现一个标准版的 `CustomHttpServletRequestWrapper`。该类的特点包括：
- 构造器不直接操作原生流，避免破坏状态。
- 懒加载缓存，第一次调用 `getInputStream()` 或 `getReader()` 时，才真正读取一次流并缓存。
- 后续调用都从缓存中获取，而不会重新读取原生流。

以下是实现代码：
```java
import javax.servlet.ReadListener;
import javax.servlet.ServletInputStream;
import javax.servlet.http.HttpServletRequest;
import javax.servlet.http.HttpServletRequestWrapper;
import java.io.*;
import java.nio.charset.StandardCharsets;

public class CustomHttpServletRequestWrapper extends HttpServletRequestWrapper {

    private byte[] cachedBody; // 缓存的请求体

    public CustomHttpServletRequestWrapper(HttpServletRequest request) {
        super(request);
        // 注意：这里不动 request 的流
    }

    // 读取并缓存 body
    private void cacheInputStream() throws IOException {
        if (cachedBody == null) {
            InputStream requestInputStream = super.getInputStream();
            ByteArrayOutputStream baos = new ByteArrayOutputStream();
            byte[] buffer = new byte[1024];
            int len;
            while ((len = requestInputStream.read(buffer)) != -1) {
                baos.write(buffer, 0, len);
            }
            cachedBody = baos.toByteArray();
        }
    }

    @Override
    public ServletInputStream getInputStream() throws IOException {
        cacheInputStream();
        final ByteArrayInputStream byteArrayInputStream = new ByteArrayInputStream(cachedBody);

        return new ServletInputStream() {
            @Override
            public boolean isFinished() {
                return byteArrayInputStream.available() == 0;
            }

            @Override
            public boolean isReady() {
                return true; // 流总是准备好
            }

            @Override
            public void setReadListener(ReadListener readListener) {
                // 不实现异步
                throw new UnsupportedOperationException();
            }

            @Override
            public int read() {
                return byteArrayInputStream.read();
            }
        };
    }

    @Override
    public BufferedReader getReader() throws IOException {
        cacheInputStream();
        return new BufferedReader(new InputStreamReader(new ByteArrayInputStream(cachedBody), StandardCharsets.UTF_8));
    }

    // 提供一个直接获取 body 的方法
    public String getBody() throws IOException {
        cacheInputStream();
        return new String(cachedBody, StandardCharsets.UTF_8);
    }
}
```

### 3. 总结
- `HttpServletRequest` 的输入流在一次请求中只能被读取一次。
- 如果在 Spring 处理请求之前读取了请求体，后续的读取操作将抛出 `IllegalStateException` 异常。
- 使用 `ContentCachingRequestWrapper` 或自定义 `HttpServletRequestWrapper` 可以解决此问题，确保请求体只读取一次并支持多次访问。

### 参考资料：
- [博客园](https://www.cnblogs.com/kevinblandy/p/14742800.html)
- [Baeldung](https://www.baeldung.com/spring-reading-httpservletrequest-multiple-times?utm_source=chatgpt.com#bd-the-implementation-of-httpServletRequest)