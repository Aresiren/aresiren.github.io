---
layout:     post
title:      hibernate方言扩展
subtitle:   org.hibernate.dialect包中class使用问题
date:       2019-08-20
author:     WY
header-img: img/black-and-white/lotus-root.jpg
catalog: true
tags:
    - JAVA
    - hibernate
    - dialect
---

## I 注册关键字[registerKeyword]
### 1.如何使用？
 a.简单来说，继承对应你使用的数据库种类和版本，如：oracle，11g。

 b.重载抽象类的方法[registerKeyword]，就可以达到目的了。

 代码与实现说明如下：
 ```java
 /*位置位于hibernate-core-版本.Final.jar中的org.hibernate.dialect包，但并没有对应11g，没有Oracle11gDialect只有Oracle10gDialect class文件？该如何解决！？
 打开Oracle10gDialect文件：

*/
package org.hibernate.dialect;

import org.hibernate.sql.JoinFragment;
import org.hibernate.sql.ANSIJoinFragment;


/**
 * A dialect specifically for use with Oracle 10g.
 * <p/>
 * The main difference between this dialect and {@link Oracle9iDialect}
 * is the use of "ANSI join syntax".  This dialect also retires the use
 * of the <tt>oracle.jdbc.driver</tt> package in favor of 
 * <tt>oracle.jdbc</tt>.
 *
 * @author Steve Ebersole
 */
public class Oracle10gDialect extends Oracle9iDialect {

	public Oracle10gDialect() {
		super();
	}

	public JoinFragment createOuterJoinFragment() {
		return new ANSIJoinFragment();
	}
}
 ```

从源码上看10g和9g就多了连接查询处理，其他的就没有差别了。这也就是java里面继承后又加了个新方法实现的效果！

那么，这个针对11g的扩展该怎么写，思路基本上就比较清楚了。一种：
```java
public class Oracle11gDialect extends Oracle9iDialect {
    public Oracle11gDialect() {
        super();
        //registerKeyword("关键字");如：
        registerKeyword("within");
        registerKeyword("listagg");
    }

    public JoinFragment createOuterJoinFragment() {
        return new ANSIJoinFragment();
    }
}
```
另一种：
```java
public class Oracle11gDialect extends Oracle10iDialect {
    public Oracle11gDialect() {
        super();
        //registerKeyword("关键字");如：
        registerKeyword("within");
        registerKeyword("listagg");
    }

}
```




### 2.解决了什么问题？
问题一般就是出现在写的sql或hql无法识别数据库的关键字，主要是数据库的函数名，发生在一些较为复杂查询和较少使用的数据库默认函数调用时。

## II 注册函数[registerFunction]
### 1.如何使用？
同上

```java
public class Oracle11gDialect extends Oracle10iDialect {
    public Oracle11gDialect() {
        super();
        //registerKeyword("关键字");如：
        registerKeyword("within");
        registerKeyword("listagg");

        //this.registerFunction("自定义函数名", new SQLFunctionTemplate(解析类型,"真实数据库定义名称(?1,?2)"));

        //自定义函数名与真实数据库定义名称可以不同，自定义函数名是给hibernate解析器看的，相当于别名，它对应真正的数据库函数

        //?1代表第一个参数,?2代表第二个参数，祥见官方文档

        registerFunction("second", new SQLFunctionTemplate(StandardBasicTypes.INTEGER, "to_char(?1, 'SS')"));
        registerFunction("minute", new SQLFunctionTemplate(StandardBasicTypes.INTEGER, "to_char(?1, 'MI')"));
        registerFunction("hour", new SQLFunctionTemplate(StandardBasicTypes.INTEGER, "to_char(?1, 'HH24')"));

    }

}
```
创建数据库函数：
```sql
CREATE OR REPLACE FUNCTION 函数名称 (
    --入参
    形参 数据类型
    ...
) RETURN 返回值类型 
IS （或 AS）
    --函数内使用的临时变量
    临时变量名1   数据类型;
    临时变量名2      数据类型;
    ...
BEGIN
    --函数体
    。。。
    临时变量 与 形参 在此相遇，可能有结果。
    。。。
END;

```

### 2.解决了什么问题？
问题出现在如何让hibernate能够正常调用自己定义扩展的特种函数

## III 注册自定义类型[registerHibernateType]
### 1.如何使用？
同上

```java
public class Oracle11gDialect extends Oracle10iDialect {
    public Oracle11gDialect() {
        super();
        //registerKeyword("关键字");如：
        registerKeyword("within");
        registerKeyword("listagg");

        registerHibernateType(Types.CHAR, Hibernate.STRING.getName());//将数据库的char类型转为String类型 

        registerHibernateType(Types.NVARCHAR, Hibernate.STRING.getName());//将数据库的NVARCHAR类型转为String类型 
    }

}
```


### 2.解决了什么问题？

问题出现在如何让hibernate能够正常解析自己定义扩展的类型也包含一些由于数据库驱动版本等导致解析异常问题

## part IV

##...

## 结语
> 用github上提供可部署ruby web项目，写博客练练手。

### 参考[reference]

-  [WWDC 2018 Keynote](https://developer.apple.com/videos/play/wwdc2018/101/)
- [参考文章标题](https://developer.apple.com/videos/play/wwdc2018/101/)

1. [参考文章标题1](https://developer.apple.com/videos/play/wwdc2018/101/)
2. [参考文章标题2](https://developer.apple.com/videos/play/wwdc2018/101/)
3. [参考文章标题3](https://developer.apple.com/videos/play/wwdc2018/101/)