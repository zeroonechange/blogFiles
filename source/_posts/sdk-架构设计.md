---
title:  sdk架构设计
date: 2021-11-15 01:46:37
---

##  sdk架构设计



####  目前架构  
common 放一些常用的工具类，网络请求类
ability   放各种能力  例如 tts，nlp，识别
sdk  放各种定制能力  在ability中挑选然后组装到一起
business 放各种定制的demo  使用定制的sdk 
用了组件化技术，可以控制business各个demo的

但是如何打出合适的sdk  aar包呢？ 

在sdk中
OSClient  封装各种ability的mananger  暴露给business使用  

但是business 没用复用demo逻辑   每个business都需要写一个不同的东西 

应该在ability中提供单独的测试用例？ 
也不行，sdk如何集成然后打包？

APT?   在编译时根据配置文件自动生成OSClient文件 

#### 目标是什么？   可以打定制的sdk   可以复用demo中的页面  

sdk用配置文件来限制  

demo中使用了不同的ability   有耦合 
sdk只不过是对ability的封装  

一个ability写一个页面  用OSClient去封装
sdk用APT 再去封装一层  把各种OSClient加进来  生成一个类 
然后替换掉实际UI中的类名 

生成sdk 需要把生成的类复制  然后打包多个module 

组件化  模块  

为什么要沿用之前老的思想呢？ 
为什么非要 OSClient 呢    一个Manager对应一个activity 

或者OSClient 全方法  多个activity

根据配置   把单独的activity复制到文件中  然后打出对应的sdk  没用到就忽略掉 



#### 解决方案 

开发时  将每个带注解的manager 中的public类   都封装下   整出一个OSClient   根据这个去调试
然后依赖给business的activity使用    如果activity可以复用   就提取出来  放到一个公共的地方 
打sdk包时  将生成的OSClient和各个module的文件 打成 aar包

技术难点:  
1.APT - 自动生成代码
2.高级进阶  Dagger 2 
3.aar打包




