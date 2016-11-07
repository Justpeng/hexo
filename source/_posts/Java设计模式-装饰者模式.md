---
title: Java设计模式-装饰者模式
date: 2016-10-02 17:58:27
description: "Java设计模式学习笔记"
categories: [装饰者模式]
tags: [设计模式]
---

## 原理
项目：咖啡馆订单系统
- 咖啡种类：Espresso,ShortBlack,LongBlack,Decaf
- 调料：Milk,Soy,Chocolate
- 扩展性好：有新品种咖啡，无需改动之前的，不会对原有代码有影响、改动方便、维护方便

1 装饰者模式就像打包一个快递
 - 主体：陶瓷、衣服
 - 包装：报纸填充、塑料泡沫、纸板
2 Component:主体超类 或再添加一中间层(实现公共功能)

3 ConcreteComponent 待包装的主体，如Decaf 和 Decorator 

4 装饰者模式：动态的将新功能附加到对象上。在对象功能拓展方面，它比继承更有弹性

## Java内置装饰者
Java的IO结构
InputStream是超类 FilterInputStream是中间类 BufferedInputStream属于装饰类

## 关键点
- 开放、关闭原则：对添加新代码开放，对已测试好功能关闭
- 易拓展