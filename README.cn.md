Vertx Kotlin RPC over EventBus
==============================
[![CircleCI](https://circleci.com/gh/windoze/vertx-kotlin-rpc.svg?style=svg)](https://circleci.com/gh/windoze/vertx-kotlin-rpc)
[ ![Download](https://api.bintray.com/packages/windoze/maven/vertx-kotlin-rpc/images/download.svg) ](https://bintray.com/windoze/maven/vertx-kotlin-rpc/_latestVersion)

极简的Vertx/Kotlin RPC框架。


开始
----

Maven包已发布到jCenter，请先按照[说明](https://bintray.com/beta/#/bintray/jcenter)配置maven或gradle的设置。

Maven:
```xml
<dependency>
    <groupId>codes.unwritten</groupId>
    <artifactId>vertx-kotlin-rpc</artifactId>
    <version>0.4</version>
    <type>pom</type>
</dependency>
```

Gradle:
```Groovy
compile 'codes.unwritten:vertx-kotlin-rpc:0.4'
```
<hr>

在Kotlin中创建RPC service
-----------------------

```kotlin
import codes.unwritten.vertx.kotlin.rpc.RpcServerVerticle

//...

// 只要方法的签名匹配，服务可以不用实现用于定义接口的interface
class HelloSvcImpl {
    // 方法可以是suspend，也可以不是
    fun hello(name: String): String = "Hello, $name!"
}

// ...

vertx.deployVerticle(RpcServerVerticle("test-channel")
    .register("hello", HelloSvcImpl()))
```


在Kotlin中调用RPC service
---------------------

```kotlin
import codes.unwritten.vertx.kotlin.rpc.getServiceProxy

// ...

// 服务接口定义
interface HelloSvc {
    // 成员函数必须是suspend，否则在调用的时候会抛出异常
    suspend fun hello(world: String): String
}

// ...

// 创建服务的Proxy对象
val svc: HelloSvc = getServiceProxy(vertx, "test-channel", "hello")
// 调用RPC服务
assertEqual("Hello, world!", svc.hello("world"))
```


在Java中创建RPC service
----------------------

```java
import codes.unwritten.vertx.kotlin.rpc.RpcServerVerticle;

// ...

// 实现类不返回Future<T>，直接返回T
public class HelloSvcImpl {
    public String hello(String name) {
        return "Hello, " + name + "!";
    }
}

// ...

vertx.deployVerticle((new RpcServerVerticle("test-channel"))
    .register("hello", new HelloSvcImpl()));
```

在Java中调用RPC service
-------------------------------
Java没有suspend函数，所以服务接口中的每个方法必须返回`Future<T>`而不是`T`。
```Java
import io.vertx.core.Future;
import static codes.unwritten.vertx.kotlin.rpc.AsyncServiceProxyFactory.getAsyncServiceProxy;

// ...

// 方法必须返回Future<T>而不是T
interface AsyncHelloSvc {
    Future<String> hello(String world);
}

// ...

AsyncHelloSvc svc = getAsyncServiceProxy(vertx, "test-channel", "hello", AsyncHelloSvc.class);
svc.hello("world").setHandler(ar -> {
    if (ar.succeeded()) {
        assertEquals("Hello, world!", ar.result());
    } else {
        // Error handling
    }
});

```

Notes
-----

所有的参数和返回值都由[Kryo](https://github.com/EsotericSoftware/kryo)进行序列化和反序列化，请参阅文档以了解更多的细节。

Changelog
---------
* 0.4: Java API

TODO
----

* 支持方法重载
* RPC调用跟踪