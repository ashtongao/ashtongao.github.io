---
layout:     post
title:      详解Annotation的使用
subtitle:   详解Annotation的使用
date:       2019-02-01
author:     Ashton
header-img: img/post-bg-os-metro.jpg
catalog: true
tags:
    - Java
    - Annotation
---


# 一、注解（Java Annotation）摘要

Java Annotaion 从JDK 5.0引入，原生只有三个注解，但是可以自定义自己的注解

内置的原生注解（Build-In Java Annotations）

* @Override
* @SuperessWarnings
* @Deprecated


内置注解，用于修饰自定义注解的（Build-In Java Annotions used in other annotations）

* @Target
* @Retention
* @Inherited
* @Documented

后面会详细介绍每一个注解的含义

# 二、原生注解

### 1、 `@Override`

标记方法被重写，编译机会校验方法是否可以被重写，如重写了静态方法等编译错误

```java
class Animal {
    void eat() {
        System.out.println("Animal eat");
    }
}

class Dog extends Animal {
    @Override
    void eat() {
        System.out.println("Dog eat");
    }
}

public class TestOverride {
    public static void main(String[] args) {
        Animal dog = new Dog();
        dog.eat();
    }
}
```

### 2、`@SuperessWarnings`

忽略编译器的warning，看如下例子

```java
import java.util.*;

class TestSuppressWarnings {
    public static void main(String[] args) {
        ArrayList l = new ArraryList();
        l.add("a");
        l.add("b");
        l.add("c");

        for (Object item: l) {
            System.out.println(item);
        }
    }
}
```

如果直接使用会产生以下错误

> 注: TestSuppreWarnings.java使用了未经检查或不安全的操作。
> 
> 注: 有关详细信息, 请使用 -Xlint:unchecked 重新编译。

可以在main方法前加上忽略注解`@SuperessWarnings("unchecked")`，即可编译通过

```java
    @SuppressWarnings("unchecked")
    public static void main() ...
```

### 3、`@Deprecated`

标记某个方法已经废弃，让编译器报错

```java

class A  {
    void normalFun() {
        System.out.println("normal function");
    }

    @Deprecated
    void deprecatedFun() {
        System.out.print("deprecated function");
    }
}

public class TestDeprecated {
    public static void main(String[] args) {
        A a = new A();
        a.normalFun();
        a.deprecatedFun();
    }
}
```

> 注: TestDeprecated.java使用或覆盖了已过时的 API。
> 
> 注: 有关详细信息, 请使用 -Xlint:deprecation 重新编译。


# 三、自定义注解(Custom Annotation)

自定义注解是通过 `@interface MyAnnotation{}` 定义来实现的

## 1、 自定义注解的类型

* Marker Annotation
* Single-Value Annotation
* Multi-Value Annotation

![](https://ashtongao.github.io/img/2019-02-01-详解Annotation的使用/2019-02-01-详解Annotation的使用-20190201160217.png)

### Marker Annotation

实现时很简单，就只是标记有这样一个注解

```java
@interface MyMakerAnnotation {}
```

> Deprecated 和 @Override 注解就是属于marker annotation

### Single-Value Annotation

单值的注解的实现

```java
@interface MySingleValueAnnotation {
    int value() default 0;
}
```

使用时，直接传入

```java
@MySingleValueAnnotation(value=10)
```

### Multi-Value Annotation

多值注解

```java
@interface MyMultiValueAnnotation {
    int value1();
    String value2() default "";
    String value3() default "xyz";
}
```

使用时

```java
@MyMultivalueAnnotation(value1 = 10, value2 = "Test", value3 = "Test1");
```

## 2、 修饰自定义注解

Java 中还有四种内置的注解，用于修饰自定义注解的

听起来很绕，实际上很简单，Java 内置的就三种注解（Override，SuppressWarnings，Deprecated），如果要使用其他注解，只能通过自定义注解

而以下的注解都是用于修饰自定义注解的，并不能直接使用，一共有四个

* @Target
* @Retention
* @Inherited
* @Documented

### `@Target`

Element Types| 可以应用注解的类型
--|--
TYPE|	class, interface or enumeration
FIELD|	fields
METHOD|	methods
CONSTRUCTOR|	constructors
LOCAL_VARIABLE|	local variables
ANNOTATION_TYPE|	annotation type
PARAMETER|	parameter

Target 用于标记Annotation使用了哪种类型，比如用于修饰Class

```java
import java.lang.annotation.ElementType;

@Target(ElementType.TYPE)
@interface MyAnnotation {
    int value1();
    String value2();
}
```

那当要修饰多个时是怎样呢，比如既要修饰Class 也要修饰Method

```java
@Target({ElmentType.TYPE, ElementType.FIELD, ElementType.METHOD})
@interface MyAnnotation {
    int value1();
    String value2();
}
```

### `@Retention`

Retention用于标记注解会被保留到哪个阶段，SOURCE是源文件，CLASS是Class文件，RUNTIME是运行时生效

RetentionPolicy	| Availability
--|--
RetentionPolicy.SOURCE |	refers to the source code, discarded during compilation. It will not be available in the compiled class.
RetentionPolicy.CLASS |	refers to the .class file, available to java compiler but not to JVM . It is included in the class file.
RetentionPolicy.RUNTIME |	refers to the runtime, available to java compiler and JVM .


```java
@Retention(RetentionPolicy.RUNTIME)
@Target(ElementType.TYPE)
@interface MyAnnotation {
    int value1();
    String value2();
}
```

可以看下使用的例子，我们创建一个运行时，同时修饰METHOD的注解

```java
import java.lang.annotation.*;
import java.lang.reflect.*;

@Retention(RetentionPolicy.RUNTIME)
@Target(ElementType.METHOD)
@interface MyAnnotation {
    int value();
}

class Hello {
    @MyAnnotation(value = 10)
    public void sayHello() {
        System.out.println("Hello Annotation");
    }
}

public class TestCustomAnnotation {
    public static void main(String[] args) throws Exception {
        Hello h = new Hello();
        h.sayHello();

        Method m = h.getClass().getMethod("sayHello");
        MyAnnotation annotation = m.getAnnotation(MyAnnotation.class);
        System.out.println("Value is " + annotation.value());

    }
}
```

> Hello Annotation
> Value is 10

### `@Inherited`

标记这个注解也会被子类继承

```java
@Inherited
@interface ForEveryOne {}

@interface ForEveryOne {}
class SuperClass {}

class SubClass extends Superclass {};
```

### `@Documented`

标记这是一个用于文档说明的注解，会被Javadoc工具记录

# 四、总结

* Java 内置的注解只有三种：Override SuppressWarnings Deprecated
* Java 在三种内置注解之外，又提供了自定义注解，同时提供了另外四个注解来帮助自定义
* 自定义注解中，四种注解分别的作用：Target(确定修饰类型)，Retention（确定应用阶段），Inherited（确定是否继承），Documented（确定是否是文档说明）

## Refrence

[Java Annotation](https://www.javatpoint.com/java-annotation)
[Java Custom Annotation](https://www.javatpoint.com/custom-annotation)
[Java Annotation认知(包括框架图、详细绍、示例说明)](https://www.cnblogs.com/skywang12345/p/3344137.html)

