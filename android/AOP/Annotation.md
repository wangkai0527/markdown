[TOC]

# Annotation
Java 注解是 Java1.5 中引入的概念，是用来标注代码的元数据。

定义一个注解使用 `@interface` 关键字。
>public @interface Test{}

## 元注解
元注解是注解的注解，用来标注注解的元数据。
可以通过元注解来控制注解的一些属性与行为。
Java 中元注解共有四种：Retention、Inherited、Documented、Target。

## 注解应用场景
注解，单独是没意义的
使用场景：
1. 规范参数类型
2. 注解+APT 用于生成一些java文件 
3. 注解+代码埋点    AspectJ arouter
4. 注解+反射+动态代理   XUtils,Lifecycle


例如：
枚举每一项都是一个Object，比较占内存(12+内容 8对齐)
@IntDef
这种方案比枚举缩小了3倍内存空间



# APT
APT（Annotation Processing Tool）即注解处理器，它是一种处理注解的工具，也是javac中的一个工具，由javac调用。
通过APT可以在编译时获取到注解和被注解对象的相关信息，在拿到这些信息后可以根据需求来自动的生成一些代码，以此提高开发效率。
简单来说，可以在javac把.java编译成.class时，做一些修改，执行我们想执行的代码。

获取注解及生成代码都是在代码编译时候完成的，相比反射在运行时处理注解大大提高了程序性能。

APT技术被广泛的运用在Java框架中，包括Android项以及Java后台项目
butterknife dagger2 Eventbus hilt databinding 阿里的ARouter路由框架...

APT的核心是AbstractProcessor类

1. 需要注册
在对应的`build.gradle`增加

>annotationProcessor 'com.google.auto.service:auto-service:1.0-rc4'
compileOnly 'com.google.auto.service:auto-service:1.0-rc4'

1. `@AutoService(Process.class)`
2. `public class AnnotationCompiler extends AbstractProcessor`
3. 支持的版本`getSupportedSourceVersion()`
4. 能用来处理那些注释`getSupportedAnnotationTypes()`
5. 定义一个用来生成APT目录下面的文件的对象`Filer filer`
6. 所有注解的处理在`process()`
编译时打印`processingEnv.getMessager().printMessage(Diagnostic.Kind.NOTE, "set------"+set);`










