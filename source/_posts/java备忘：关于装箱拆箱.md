title: java备忘：关于装箱拆箱
author: sola
tags:
  - java
categories:
  - coding
date: 2019-04-12 19:47:00

---

- 自动装箱：  基本类型->对象   编译器自动调用`valueOf()`方法
  + 赋值时  `Integer num = 10` 
  + 
- 自动拆箱：  对象->基本类型 编译器自动调用`intValue()`方法
  + 赋值时：`int num1 = num`
  + 计算时  `System.out.print(num--)`
- Cache：–128到127之间的值**自动装箱**后的对象会存到Cache里复用，同一个对象、地址相同，`==`为true，但如果是`new`出来的，还是会创建新对象
- 和String有点像，见上篇
- 参考https://www.jb51.net/article/31934.htm