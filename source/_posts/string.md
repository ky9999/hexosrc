title: 'java备忘:关于String'
tags:
  - java
categories:
  - coding
author: sola
date: 2019-03-27 20:05:00
---
**完结**

- `s=new String("abc")`会在常量池创建个对象，但这个对象只是方便那些直接引用常量池对象的操作（如`s2="abc"`,`s3=new String("abc").intern()`）不再在堆中创建对象，继续`s2=new String("abc")`依然会在堆中创建新对象
- `s3=new String("abc").intern()`至多创建一个对象（常量池中），永远不会在堆中创建对象，`s2=new String("abc")`至少创建一个对象（堆中）

**补充5：看了下jdk源码** 

```java
private finnal char value[];
public String() {
    this .value = new char[0]；
}
```
- jdk1.6的`substring()`因为怎么截取都会保留全的char[]，造成内存泄漏，1.7以后`substring()`会创建新的

**补充4：`intern（）`的真正作用**
-  看了https://blog.csdn.net/ShelleyLittlehero/article/details/81196418
- 一张图
 ![avatar](https://img-blog.csdn.net/20180725093405756?watermark/2/text/aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L1NoZWxsZXlMaXR0bGVoZXJv/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)
-  *因为常量池只能将确定值的字符串加入，所有工程中遇到大量非确定值字符串（如`String s=s1+s2`）,`intern()`这时候就起作用了，它强行吧`s`这个非确定值加入到常量池，以后用到就不用在堆中新创建对象了，节省内存；其他时候加不加`intern（）`效果其实是一样的（指都会在常量池创建对象）（不过返回的地址不一样，intern返回常量池的对象的引用，不加intern额外再堆里创建一个对象）~*

**补充3：总结下美团那篇**
-  第一例 
```java
public static void main(String[] args) {
    String s = new String("1");  //在heap和常量池创建一个对象1 s在heap(*其值指向常量池的那个对象 ？*)
    s.intern(); //常量池已有1，不动
    String s2 = "1"; //在常量池创建一个对象1 常量池已有 s2指向它 s2在常量池
    System.out.println(s == s2);//false false

    String s3 = new String("1") + new String("1"); //在heap创建一个对象11 常量池一个对象1 s3在heap
    s3.intern();//常量池没11 创建1个11（1.6） 复制来一份heap区的11对象的引用（1.7）
    String s4 = "11"; //在常量池创建一个11 常量池已有 指向它 s4在常量池（1.6） heap（1.7)
    System.out.println(s3 == s4);//false true
}
```
- 第二例(变了下顺序)
```java
public static void main(String[] args) {
    String s = new String("1");
    String s2 = "1";
    s.intern();
    System.out.println(s == s2);//false false 和第一例没啥区别

    String s3 = new String("1") + new String("1");//heap的11 常量池的1 s3在heap
    String s4 = "11";//常量池没有11 在常量池创建一个对象11 s4在常量池
    s3.intern();//常量池已有 指向它
    System.out.println(s3 == s4);//false false
}
```


**补充2：总结以前的几点认识误区**
- `s1.intern()` intern方法只是只是看常量池有没有已存在的相同串，没有就创建一份新对象（1.6）或者一份指向这个堆对象的引用（1.7+）放到常量池，并不会
把s1指向常量池里的对象（或地址）！除非`s1=s1.intern()`
-  -  `s2=new String("sola")`会同时在堆和常量池创建一个对象 ***如果常量池已有 则只在堆里创建一个对象（依然会创建，只要有new就一定会创建新对象（新地址），只是这个对象的值是指向常量池（相当于转了一道，见上图），而不是整个对象直接指向常量池里的对象！）***
   - `s3="sola"` 只在常量池创建 ***如果常量池已有 就指向它***
   -  `String s3 = new String("1") + new String("1")`不会在常量池创建“11”
- java的string和c的char[]完全不同


**补充：关于intern（）**

https://tech.meituan.com/2014/03/06/in-depth-understanding-string-intern.html


**Back---重回String设计的初衷：**

Java中的String被设计成不可变的，出于以下几点考虑：

1. 字符串常量池的需要。字符串常量池的诞生是为了提升效率和减少内存分配。可以说我们编程有百分之八十的时间在处理字符串，而处理的字符串中有很大概率会出现重复的情况。正因为String的不可变性，常量池很容易被管理和优化。

2. 安全性考虑。正因为使用字符串的场景如此之多，所以设计成不可变可以有效的防止字符串被有意或者无意的篡改。从java源码中String的设计中我们不难发现，该类被final修饰，同时所有的属性都被final修饰，在源码中也未暴露任何成员变量的修改方法。（当然如果我们想，通过反射或者Unsafe直接操作内存的手段也可以实现对所谓不可变String的修改）。

3. 作为HashMap、HashTable等hash型数据key的必要。因为不可变的设计，jvm底层很容易在缓存String对象的时候缓存其hashcode，这样在执行效率上会大大提升。



**Deeper---直接来看例子:**

首先来试试下面程序的运行结果是否与预想的一致：

```java
String s1 = new String("aaa");
String s2 = "aaa";
System.out.println(s1 == s2);    // false

s1 = new String("bbb").intern();
s2 = "bbb";
System.out.println(s1 == s2);    // true

s1 = "ccc";
s2 = "ccc";
System.out.println(s1 == s2);    // true

s1 = new String("ddd").intern();
s2 = new String("ddd").intern();
System.out.println(s1 == s2);    // true

s1 = "ab" + "cd";
s2 = "abcd";    
System.out.println(s1 == s2);    // true

String temp = "hh";
s1 = "a" + temp;
// 如果调用s1.intern 则最终返回true
s2 = "ahh";
System.out.println(s1 == s2);    // false

temp = "hh".intern();
s1 = "a" + temp;
s2 = "ahh";
System.out.println(s1 == s2);    // false

temp = "hh".intern();
s1 = ("a" + temp).intern();
s2 = "ahh";
System.out.println(s1 == s2);    // true

s1 = new String("1");    // 同时会生成堆中的对象 以及常量池中1的对象，但是此时s1是指向堆中的对象的
s1.intern();            // 常量池中的已经存在
s2 = "1";
System.out.println(s1 == s2);    // false

String s3 = new String("1") + new String("1");    // 此时生成了四个对象 常量池中的"1" + 2个堆中的"1" + s3指向的堆中的对象（注此时常量池不会生成"11"）
s3.intern();    // jdk1.7之后，常量池不仅仅可以存储对象，还可以存储对象的引用，会直接将s3的地址存储在常量池
String s4 = "11";    // jdk1.7之后，常量池中的地址其实就是s3的地址
System.out.println(s3 == s4); // jdk1.7之前false， jdk1.7之后true

s3 = new String("2") + new String("2");
s4 = "22";        // 常量池中不存在22，所以会新开辟一个存储22对象的常量池地址
s3.intern();    // 常量池22的地址和s3的地址不同
System.out.println(s3 == s4); // false

// 对于什么时候会在常量池存储字符串对象，我想我们可以基本得出结论: 1. 显示调用String的intern方法的时候; 2. 直接声明字符串字面常量的时候，例如: String a = "aaa";
// 3. 字符串直接常量相加的时候，例如: String c = "aa" + "bb";  其中的aa/bb只要有任何一个不是字符串字面常量形式，都不会在常量池生成"aabb". 且此时jvm做了优化，不//   会同时生成"aa"和"bb"在字符串常量池中

// 对于什么时候会在常量池存储字符串对象，我想我们可以基本得出结论: 1. 显示调用String的intern方法的时候; 2. 直接声明字符串字面常量的时候，例如: String a = "aaa";
// 3. 字符串直接常量相加的时候，例如: String c = "aa" + "bb";  其中的aa/bb只要有任何一个不是字符串字面常量形式，都不会在常量池生成"aabb". 且此时jvm做了优化，不//   会同时生成"aa"和"bb"在字符串常量池中
```

如果有出入的话，再来看看具体的字节码分析：
```java
/**
 * 字节码为：
 *   0:   ldc     #16; //String 11   --- 从常量池加载字符串常量11
     2:   astore_1                   --- 将11的引用存到本地变量1，其实就是将s指向常量池中11的位置
 */
String s = "11";    

/**
 * 0:   new     #16; //class java/lang/String    --- 新开辟了一个地址，存储new出来的对象
   3:   dup                                      --- 将new出来的对象复制了一份到栈顶（也就是s1最终指向的是堆中的另一个存储字符串11的地址）
   4:   ldc     #18; //String 11　　　　　　　　　　
   6:   invokespecial   #20; //Method java/lang/String."<init>":(Ljava/lang/String;)V
   9:   astore_1
 */
String s1 = new String("11");

/**
 * 0:   new     #16; //class java/lang/StringBuilder                       --- 可以看到jdk对字符串拼接做了优化，先是建了一个StringBuilder对象
   3:   dup
   4:   new     #18; //class java/lang/String                              --- 创建String对象
   7:   dup
   8:   ldc     #20; //String 1                                            --- 从常量池加载了1（此时常量池和堆中都会存字符串对象）
   10:  invokespecial   #22; //Method java/lang/String."<init>":(Ljava/lang/String;)V                    --- 初始化String("1")对象
   13:  invokestatic    #25; //Method java/lang/String.valueOf:(Ljava/lang/Object;)Ljava/lang/String;
   16:  invokespecial   #29; //Method java/lang/StringBuilder."<init>":(Ljava/lang/String;)V             --- 初始化StringBuilder对象
   19:  new     #18; //class java/lang/String
   22:  dup
   23:  ldc     #20; //String 1
   25:  invokespecial   #22; //Method java/lang/String."<init>":(Ljava/lang/String;)V
   28:  invokevirtual   #30; //Method java/lang/StringBuilder.append:(Ljava/lang/String;)Ljava/lang/StringBuilder;
   31:  invokevirtual   #34; //Method java/lang/StringBuilder.toString:()Ljava/lang/String;
   34:  astore_1                                                                                          ---从上可以看到实际上常量池目前只存了1
  36:  invokevirtual   #38; //Method java/lang/String.intern:()Ljava/lang/String;  --- 调用String.intern中，jdk1.7以后，常量池也是堆中的一部分且常量池可以存引用，这里直接存的是s2的引用
  39:  pop                                                                                                --- 这里直接返回的是栈顶的元素
 */
String s2 = new String("1") + new String("1");
s2.intern();

/**
 * 0:   ldc     #16; //String abc        --- 可以看到此时常量池直接存储的是:abc, 而不会a、b、c各存一份
   2:   astore_1
 */
String s3 = "a" + "b" + "c";

/**    
0:   new     #16; //class java/lang/StringBuilder
3:   dup
4:   ldc     #18; //String why                --- 常量池的why
6:   invokespecial   #20; //Method java/lang/StringBuilder."<init>":(Ljava/lang/String;)V
9:   ldc     #23; //String true                --- 常量池的true
11:  invokevirtual   #25; //Method java/lang/StringBuilder.append:(Ljava/lang/String;)Ljava/lang/StringBuilder;
14:  invokevirtual   #29; //Method java/lang/StringBuilder.toString:()Ljava/lang/String;
17:  astore_1
*/
String s1 = new StringBuilder("why").append("true").toString();
System.out.println(s1 == s1.intern());                            // jdk1.7之前为false，之后为true
```

下面我们延伸一下来讲讲字符串拼接的优化问题：

```java
/**
 * 字节码为：
 *   0:   ldc     #16; //String 11   --- 从常量池加载字符串常量11
     2:   astore_1                   --- 将11的引用存到本地变量1，其实就是将s指向常量池中11的位置
 */
String s = "11";    

/**
 * 0:   new     #16; //class java/lang/String    --- 新开辟了一个地址，存储new出来的对象
   3:   dup                                      --- 将new出来的对象复制了一份到栈顶（也就是s1最终指向的是堆中的另一个存储字符串11的地址）
   4:   ldc     #18; //String 11　　　　　　　　　　
   6:   invokespecial   #20; //Method java/lang/String."<init>":(Ljava/lang/String;)V
   9:   astore_1
 */
String s1 = new String("11");

/**
 * 0:   new     #16; //class java/lang/StringBuilder                       --- 可以看到jdk对字符串拼接做了优化，先是建了一个StringBuilder对象
   3:   dup
   4:   new     #18; //class java/lang/String                              --- 创建String对象
   7:   dup
   8:   ldc     #20; //String 1                                            --- 从常量池加载了1（此时常量池和堆中都会存字符串对象）
   10:  invokespecial   #22; //Method java/lang/String."<init>":(Ljava/lang/String;)V                    --- 初始化String("1")对象
   13:  invokestatic    #25; //Method java/lang/String.valueOf:(Ljava/lang/Object;)Ljava/lang/String;
   16:  invokespecial   #29; //Method java/lang/StringBuilder."<init>":(Ljava/lang/String;)V             --- 初始化StringBuilder对象
   19:  new     #18; //class java/lang/String
   22:  dup
   23:  ldc     #20; //String 1
   25:  invokespecial   #22; //Method java/lang/String."<init>":(Ljava/lang/String;)V
   28:  invokevirtual   #30; //Method java/lang/StringBuilder.append:(Ljava/lang/String;)Ljava/lang/StringBuilder;
   31:  invokevirtual   #34; //Method java/lang/StringBuilder.toString:()Ljava/lang/String;
   34:  astore_1                                                                                          ---从上可以看到实际上常量池目前只存了1
  36:  invokevirtual   #38; //Method java/lang/String.intern:()Ljava/lang/String;  --- 调用String.intern中，jdk1.7以后，常量池也是堆中的一部分且常量池可以存引用，这里直接存的是s2的引用
  39:  pop                                                                                                --- 这里直接返回的是栈顶的元素
 */
String s2 = new String("1") + new String("1");
s2.intern();

/**
 * 0:   ldc     #16; //String abc        --- 可以看到此时常量池直接存储的是:abc, 而不会a、b、c各存一份
   2:   astore_1
 */
String s3 = "a" + "b" + "c";

/**    
0:   new     #16; //class java/lang/StringBuilder
3:   dup
4:   ldc     #18; //String why                --- 常量池的why
6:   invokespecial   #20; //Method java/lang/StringBuilder."<init>":(Ljava/lang/String;)V
9:   ldc     #23; //String true                --- 常量池的true
11:  invokevirtual   #25; //Method java/lang/StringBuilder.append:(Ljava/lang/String;)Ljava/lang/StringBuilder;
14:  invokevirtual   #29; //Method java/lang/StringBuilder.toString:()Ljava/lang/String;
17:  astore_1
*/
String s1 = new StringBuilder("why").append("true").toString();
System.out.println(s1 == s1.intern());                            // jdk1.7之前为false，之后为true
```


**Where---String.intern的使用**：

我们直接看一个例子来结束String.intern之旅吧：

```java
Integer[] DB_DATA = new Integer[10];
Random random = new Random(10 * 10000);
for (int i = 0; i < DB_DATA.length; i++) {
    DB_DATA[i] = random.nextInt();
}
long t = System.currentTimeMillis();
for (int i = 0; i < MAX; i++) {
    arr[i] = new String(String.valueOf(DB_DATA[i % DB_DATA.length]));                // --- 每次都要new一个对象
    // arr[i] = new String(String.valueOf(DB_DATA[i % DB_DATA.length])).intern();    --- 其实虽然这么多字符串，但是类型最多为10个，大部分重复的字符串,大大减少内存
}
System.out.println((System.currentTimeMillis() - t) + "ms");
System.gc();
```

**参考链接：**
https://www.cnblogs.com/Kidezyq/p/8040338.html

http://www.360doc.com/content/14/0721/16/1073512_396062351.shtml

https://www.cnblogs.com/SaraMoring/p/5713732.html