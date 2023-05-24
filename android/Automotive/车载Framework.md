[TOC]

# Android Automotive平台

Android Automotive是通过Android的通用框架，语言和API来实现的一个全栈，开源，高度可定制的平台。

## 1. Android Automotive与整个Android生态系统的关系

- Android Automotive是Android的一部分。 Android Automotive不是Android的分支或并行开发，它与手机，平板电脑等安卓设备上的Android具有相同的代码库，并且位于相同的存储库中。它基于经过10多年开发的强大平台和功能集，可利用现有的安全模型，兼容性程序，开发人员工具和基础架构，同时继续具有高度可定制性和可移植性，完全免费和开源的特点。
- Android Automotive扩展了Android 。在将Android打造为功能齐全的信息娱乐平台的过程中，我们添加了对汽车特定要求，功能和技术的支持。就像今天用于移动设备的Android一样，Android Automotive将是一个完整的汽车信息娱乐平台。



## 2. Android Automotive架构

![img](https:////upload-images.jianshu.io/upload_images/19292381-8055b0be3cd39c07.png?imageMogr2/auto-orient/strip|imageView2/2/w/1061/format/webp)

 



![img](img/车载Framework.png)



Automotive

Android Automative是在原先Android的系统架构上增加了一些与车相关的（图中虚线框中绿色背景的）模块。

- Car App ：包括OEM和第三方开发的App
- Car API ：提供给汽车App特有的接口
- Car Service ：系统中与车相关的服务，主要是基于CarProperty实现Vechile相关的一些策略
- Vehicle Network Service ：汽车的网络服务
- Vehicle HAL ：汽车的硬件抽象层描述，定义 OEM 可以实现的车辆属性的接口

# Android CarAPI

![img](https:////upload-images.jianshu.io/upload_images/19292381-93222119bf648599.png?imageMogr2/auto-orient/strip|imageView2/2/w/1097/format/webp)

CarAPI

· annotation：包含了两个注解。
 · app
 · menu：车辆应用菜单相关API。
 · cluster：仪表盘相关API。
 · render：渲染相关API。
 · content
 · pm：应用包相关API。
 · diagnostic：包含与汽车诊断相关的API。
 · hardware：车辆硬件相关API。
 · cabin：座舱相关API。
 · hvac：通风空调相关API。
 · property：属性相关API(实现定制的property)。
 · radio：收音机相关API。
 · input：输入相关API。
 · media：多媒体相关API。
 · navigation：导航相关API。
 · settings：设置相关API。
 · vms：汽车监测相关API

这些api集合中，我们可以通过CarpropertyManager去实现定制的property功能，简要类图：



![img](https:////upload-images.jianshu.io/upload_images/19292381-3b92c1729c3d08e0.png?imageMogr2/auto-orient/strip|imageView2/2/w/670/format/webp)

## 1.CarpropertyManager内部方法

- 通过registerListener注册自定义的propertyId以及property变更通知的callback

![img](https:////upload-images.jianshu.io/upload_images/19292381-ceab5e392386c739.png?imageMogr2/auto-orient/strip|imageView2/2/w/1200/format/webp)

- 解注册CarpropertyEventListener

  ![img](https:////upload-images.jianshu.io/upload_images/19292381-48a717a743c90bd9.png?imageMogr2/auto-orient/strip|imageView2/2/w/1200/format/webp)

- 获取所有property, 返回一个元素类型是CarpropertyConfig的list

  ![img](https:////upload-images.jianshu.io/upload_images/19292381-60cf7b2927396174.png?imageMogr2/auto-orient/strip|imageView2/2/w/899/format/webp)

- 获取property状态

  ![img](https:////upload-images.jianshu.io/upload_images/19292381-d9ce5228ebf55e96.png?imageMogr2/auto-orient/strip|imageView2/2/w/904/format/webp)

- 获取property value

  ![img](https:////upload-images.jianshu.io/upload_images/19292381-3debbae89e705945.png?imageMogr2/auto-orient/strip|imageView2/2/w/974/format/webp)

- 设置property value

  ![img](https:////upload-images.jianshu.io/upload_images/19292381-9a94072289e2c153.png?imageMogr2/auto-orient/strip|imageView2/2/w/874/format/webp)

## 2.CarpropertyManager与service层交互的AIDL接口

![img](https:////upload-images.jianshu.io/upload_images/19292381-c22b6ccc2b562524.png?imageMogr2/auto-orient/strip|imageView2/2/w/1155/format/webp)

![img](https:////upload-images.jianshu.io/upload_images/19292381-347232fa0f5e839b.png?imageMogr2/auto-orient/strip|imageView2/2/w/1155/format/webp)

需要注意的是ICarProperty是同步接口，ICarPropertyEventListener是异步接口。

onEvent传上来的是参数是CarPropertyEvent的list，CarPropertyEvent中包含event type与CarPropertyValue;
 eventType包含PROPERTY_EVENT_PROPERTY_CHANGE与PROPERTY_EVENT_ERROR，分别对应listener中的onPropertyChanged和PROPERTY_EVENT_ERROR， CarPropertyValue则包含具体的propId、propValue等具体属性信息。

# Android CarService



```undefined
代码目录： packages/services/Car/service
```

- CarService并非一个服务，而是一系列的服务。这些服务都在ICarImpl.java构造函数中列了出来

  ![img](https:////upload-images.jianshu.io/upload_images/19292381-30e14636af4c1df3.png?imageMogr2/auto-orient/strip|imageView2/2/w/922/format/webp)

## 1.CarService相关服务启动流程

- SystemServer.java 启动CarServiceHelperService服务（frameworks/base/services/java/com/android/server/SystemServer.java）

  ![img](https:////upload-images.jianshu.io/upload_images/19292381-041f8a227d01fc33.png?imageMogr2/auto-orient/strip|imageView2/2/w/1200/format/webp)

- CarServiceHelperService.java ，绑定carservice服务 (frameworks/opt/car/services/src/com/android/internal/car/CarServiceHelperService.java)

  ![img](https:////upload-images.jianshu.io/upload_images/19292381-8253d8a5d7a05fe6.png?imageMogr2/auto-orient/strip|imageView2/2/w/1200/format/webp)

- CarService.java ，创建ICarImpl实例，并调用init方法. (packages/services/Car/service/src/com/android/car/CarService.java)

  ![img](https:////upload-images.jianshu.io/upload_images/19292381-b542654d3b44068e.png?imageMogr2/auto-orient/strip|imageView2/2/w/712/format/webp)

- ICarImpl.java 构造函数中启动服务, 调用对应服务得init方法. (/packages/services/Car/service/src/com/android/car/ICarImpl.java)

  ![img](https:////upload-images.jianshu.io/upload_images/19292381-f0bd49134ff77868.png?imageMogr2/auto-orient/strip|imageView2/2/w/539/format/webp)

## 2.CarPropertyService

- 实现了ICarProperty， CarServiceBase,  PropertyHalService.PropertyHalListener 接口中的方法

![img](https:////upload-images.jianshu.io/upload_images/19292381-55466067384aa24f.png?imageMogr2/auto-orient/strip|imageView2/2/w/1200/format/webp)

- registerListener调用PropertyHalService的subscribeProperty方法，同时会将最新值同步上层。

  ![img](https:////upload-images.jianshu.io/upload_images/19292381-0e4134effd22f76b.png?imageMogr2/auto-orient/strip|imageView2/2/w/954/format/webp)

- getPropertyList 、setProperty 、getProperty 主要检查了读写权限。

![img](img/getPropertyList.webp)



