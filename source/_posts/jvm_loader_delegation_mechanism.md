---
title: JVM类加载器及双亲委派模型
date: 2017-05-16 21:23:25 
author: Zak
avatar: /images/avatar.png
authorLink: http://www.wuzguo.com
authorAbout: https://github.com/wzguo
authorDesc: 一个自强不息，艰苦奋斗的「十八线码农」
categories: 后台
tags: 
	- JVM
	- 虚拟机
keywords: JVM，Java虚拟机
photos:
	- /blog/images/201705/1.png
description: 类加载器及双亲委派模型的介绍
---

####一、JVM的构成

Java 虚拟机（Java virtual machine，JVM）是运行 Java 程序必不可少的机制，它实现Java语言最重要的特征：平台无关性，即编译后的 Java 程序指令并不直接在硬件系统的 CPU 上执行，而是由 JVM 执行。

JVM屏蔽了与具体平台相关的信息，使Java语言编译程序只需要生成在JVM上运行的目标字节码（.class）文件，就可以在多种平台上不加修改地运行。（一次编译，到处运行）

classloader 把硬盘上的class 文件加载到JVM中的运行时数据区域,，但是它不负责这个类文件能否执行，而这个是执行引擎负责的。执行引擎在执行字节码时，把字节码解释成具体平台上的机器指令执行。

下图是JAVA虚拟机的结构图，每个Java虚拟机都有一个类装载子系统，它根据给定的**全限定名**来装入类型（类或接口）。同样，每个Java虚拟机都有一个执行引擎，它负责执行那些包含在被装载类的方法中的指令。

![Image](/blog/images/201705/Image.png)

**==Java虚拟机= 类加载器（classloader） + 执行引擎（execution engine） + 运行时数据区域 （runtime data area）==**



####二、什么是ClassLoader
大家都知道，当我们写好一个Java程序之后，不是管是CS还是BS应用，都是由若干个.class文件组织而成的一个完整的Java应用程序，当程序在运行时，即会调用该程序的一个入口函数来调用系统的相关功能，而这些功能都被封装在不同的class文件当中，所以经常要从这个class文件中要调用另外一个class文件中的方法，如果另外一个文件不存在的，则会引发系统异常。而程序在启动的时候，并不会一次性加载程序所要用的所有class文件，而是根据程序的需要，通过Java的类加载机制（ClassLoader）来动态加载某个class文件到内存当中的，从而只有class文件被载入到了内存之后，才能被其它class所引用。所以ClassLoader就是用来动态加载class文件到内存当中用的。

类加载器 classloader 是具有层次结构的，也就是父子关系。其中，Bootstrap 是所有类加载器的父亲。如下图所示：![img](/blog/images/201705/1.png)


```java
public class ClassLoaderTest {

    public static void main(String[] args) {
        ClassLoader loader = ClassLoaderTest.class.getClassLoader();
        while (loader != null) {
            System.out.println(loader.getClass().getName());
            loader = loader.getParent();
        }
        System.out.println(loader);
    }
}
```

```java
sun.misc.Launcher$AppClassLoader
sun.misc.Launcher$ExtClassLoader
null
```
####三、JVM默认提供的三个ClassLoader

1. BootStrap ClassLoader：称为启动类加载器（根类加载器），是Java类加载层次中最顶层的类加载器，负责加载JDK中的核心类库，如：rt.jar、resources.jar、charsets.jar等，可通过如下程序获得该类加载器从哪些地方加载了相关的jar或class文件：

```java
public class ClassLoaderTest {
    public static void main(String[] args) {
        URL[] urls = sun.misc.Launcher.getBootstrapClassPath().getURLs();
        for (int i = 0; i < urls.length; i++) {
            System.out.println(urls[i].toExternalForm());
        }
    }

}
```

控制台打印的结果：
```java
file:/D:/Java/jdk1.7.0_80/jre/lib/resources.jar
file:/D:/Java/jdk1.7.0_80/jre/lib/rt.jar
file:/D:/Java/jdk1.7.0_80/jre/lib/sunrsasign.jar
file:/D:/Java/jdk1.7.0_80/jre/lib/jsse.jar
file:/D:/Java/jdk1.7.0_80/jre/lib/jce.jar
file:/D:/Java/jdk1.7.0_80/jre/lib/charsets.jar
file:/D:/Java/jdk1.7.0_80/jre/lib/jfr.jar
file:/D:/Java/jdk1.7.0_80/jre/classes
```

其实上述结果也是通过查找sun.boot.class.path这个系统属性所得知的。

```java
public class ClassLoaderTest {
    
    public static void main(String[] args) {
        // Java的入口程序sun.misc.Launcher中定义
        final String strPath = System.getProperty("sun.boot.class.path");

        File[] dirs;
        if (strPath != null) {
            StringTokenizer tokenizer = new StringTokenizer(strPath, File.pathSeparator);
            int count = tokenizer.countTokens();
            dirs = new File[count];
            for (int i = 0; i < count; i++) {
                dirs[i] = new File(tokenizer.nextToken());
            }
        } else {
            dirs = new File[0];
        }

        for (File f : dirs) {
            System.out.println(f.getAbsolutePath());
        }
    }
}
```

2. Extension ClassLoader：称为扩展类加载器，负责加载Java的扩展类库，默认加载JAVA_HOME/jre/lib/ext/目下的所有jar。

```java
public class ClassLoaderTest {

    public static void main(String[] args) {
        // Java的入口程序sun.misc.Launcher中定义
        final String strPath = System.getProperty("java.ext.dirs");

        File[] dirs;
        if (strPath != null) {
            StringTokenizer tokenizer = new StringTokenizer(strPath, File.pathSeparator);
            int count = tokenizer.countTokens();
            dirs = new File[count];
            for (int i = 0; i < count; i++) {
                dirs[i] = new File(tokenizer.nextToken());
            }
        } else {
            dirs = new File[0];
        }

        for (File f : dirs) {
            System.out.println(f.getAbsolutePath());
        }
    }
}
```
控制台打印的结果：
```java
D:\Java\jdk1.7.0_80\jre\lib\ext
C:\WINDOWS\Sun\Java\lib\ext
```
3. App ClassLoader：称为系统类加载器，负责加载应用程序classpath目录下的所有jar和class文件。

```java
public class ClassLoaderTest {

    public static void main(String[] args) {
        // Java的入口程序sun.misc.Launcher中定义
        final String strPath = System.getProperty("java.class.path");

        File[] dirs;
        if (strPath != null) {
            StringTokenizer tokenizer = new StringTokenizer(strPath, File.pathSeparator);
            int count = tokenizer.countTokens();
            dirs = new File[count];
            for (int i = 0; i < count; i++) {
                dirs[i] = new File(tokenizer.nextToken());
            }
        } else {
            dirs = new File[0];
        }

        for (File f : dirs) {
            System.out.println(f.getAbsolutePath());
        }
    }
}
```
控制台打印的结果：
```java
D:\Java\jdk1.7.0_80\jre\lib\charsets.jar
D:\Java\jdk1.7.0_80\jre\lib\deploy.jar
D:\Java\jdk1.7.0_80\jre\lib\ext\access-bridge-64.jar
D:\Java\jdk1.7.0_80\jre\lib\ext\dnsns.jar
D:\Java\jdk1.7.0_80\jre\lib\ext\jaccess.jar
D:\Java\jdk1.7.0_80\jre\lib\ext\localedata.jar
D:\Java\jdk1.7.0_80\jre\lib\ext\sunec.jar
D:\Java\jdk1.7.0_80\jre\lib\ext\sunjce_provider.jar
D:\Java\jdk1.7.0_80\jre\lib\ext\sunmscapi.jar
D:\Java\jdk1.7.0_80\jre\lib\ext\zipfs.jar
D:\Java\jdk1.7.0_80\jre\lib\javaws.jar
D:\Java\jdk1.7.0_80\jre\lib\jce.jar
D:\Java\jdk1.7.0_80\jre\lib\jfr.jar
D:\Java\jdk1.7.0_80\jre\lib\jfxrt.jar
D:\Java\jdk1.7.0_80\jre\lib\jsse.jar
D:\Java\jdk1.7.0_80\jre\lib\management-agent.jar
D:\Java\jdk1.7.0_80\jre\lib\plugin.jar
D:\Java\jdk1.7.0_80\jre\lib\resources.jar
D:\Java\jdk1.7.0_80\jre\lib\rt.jar
**F:\Intellij\classLoaderTest\target\classes**
**D:\Program Files\JetBrains\IntelliJ IDEA 2017.1\lib\idea_rt.jar**
```

**注意事项：**

- 除了Java默认提供的三个ClassLoader之外，用户还可以根据需要定义自已的ClassLoader，而这些**自定义的ClassLoader都必须继承自java.lang.ClassLoader类**。

- Extension ClassLoader和App ClassLoader也继承自ClassLoader类。但是Bootstrap ClassLoader不继承自ClassLoader，它不是一个普通的Java类，底层**由操作系统的本地语言编写**（**C/C++/VB**），已嵌入到了JVM内核当中，当JVM启动后，Bootstrap ClassLoader也随着启动，负责加载完核心类库后，并构造Extension ClassLoader和App ClassLoader类加载器。
```java
public class ClassLoaderTest {
    public static void main(String[] args) {
        // System类 java.lang.System 位于rt.jar包中
        // 打印结果： null
        System.out.println(System.class.getClassLoader());
    }
}
```


- 自定义类加载器的加载过程也符合双起委派机制，可以通过XXX.class.getClassLoader().getClass().getName()方法获取XXX类的类加载器。


```java
public class ClassLoaderTest {
    public static void main(String[] args) {
        System.out.println(ClassLoaderTest.class.getClassLoader().getClass().getName());
    }
}
```

```java
sun.misc.Launcher$AppClassLoader
```

#### 四、类加载器的工作机制

类的加载指的是将类的.class文件中的二进制数据读入到内存中，将其放在运行时数据区的方法区内，然后在堆区创建一个java.lang.Class对象，用来封装类在方法区内的数据结构，并且向Java程序员提供了访问方法区内的数据结构的接口。

![](/blog/images/201705/11.png)

1. 类使用方式，Java程序对类的使用方式可分为两种：

   - 主动使用

   - 被动使用

   所有的Java虚拟机实现必须在每个类或接口被Java程序**”首次主动使用”**时才初始化他们，主动使用的场景有以下六种：

   - 创建类的实例
   - 访问某个类或接口的静态变量，或者对该静态变量赋值
   - 调用类的静态方法
   - 反射（如 `Class.forName("com.github.wuzguo.Test")`）
   - 初始化一个类的子类
   - Java虚拟机启动时被标明为启动类的类（Java Test， 或者包含main方法的类）

   除了以上六种情况，其他使用Java类的方式都被看作是对类的被动使用，都不会导致类的初始化。

2. 类的加载方式，加载.class文件的方式有以下几种：
   - 从本地系统中直接加载
   - 通过网络下载.class文件
   - 从zip，jar等归档文件中加载.class文件
   - 从专有数据库中提取.class文件
   - 将Java源文件动态编译为.class文件

3. 类的加载流程，类装载器子系统除了要定位和导入二进制class文件外，还必须负责验证被导入类的正确性，为类变量分配并初始化内存，以及帮助解析符号引用。这些动作必须严格按以下顺序进行：

   ![](/blog/images/201705/2.png)

   1. 装载：查找并装载类型的二进制数据，JVM规范允许类加载器在预料某个类将要被使用时就预先加载它，如果在预先加载的过程中遇到了.class文件缺失或存在错误，类加载器必须在程序首次主动使用该类时才报告错误（LinkageError错误）如果这个类一直没有被程序主动使用，那么类加载器就不会报告错误

   2. 连接：指向验证、准备、以及解析（可选）类被加载后，就进入连接阶段。连接就是将已经读入到内存的类的二进制数据合并到虚拟机的运行时环境中去。
      - 验证：确保被导入类型的正确性。类的验证主要包括以下内容：
        - 类文件的结构检查：确保类文件遵从Java类文件的固定格式。

          ![](/blog/images/201705/12.png)

        - 语义检查：确保类本身符合Java语言的语法规定，比如验证final类型的类有没有子类，final类型的方法有没有被覆盖。

        - 字节码验证：确保字节码可以被Java虚拟机安全地执行。字节码流代表Java方法（包含静态方法和实例方法），他是由被称作操作码的单字节指令组成的序列，每一个操作码后都跟着一个或多个操作数。字节码验证步骤会检查每个操作码是否合法，即是否有着合法的操作数。

        - 二进制兼容性的验证：确保相互引用的类之间协调一致，例如：在worker类的gotowork方法中会调用car类的run方法，java虚拟机在验证worker类时，会检查在方法区是否存在car类的run方法，假如不存在（当worker类和car类不兼容，就会出现这种问题），就会抛出NoSuchMethodError错误。

      - 准备：为类变量分配内存，并将其初始化为默认值
      - 解析：把类型中的符号引用转换为直接引用。　　

      3. 初始化：把类变量（成员变量）初始化为正确初始值。　


每个JAVA虚拟机实现都必须有一个启动类装载器，它知道怎么装载受信任的类。每个类装载器都有自己的命名空间，其中维护着由它装载的类型。所以一个Java程序可以多次装载具有同一个全限定名的多个类型。这样一个类型的全限定名就不足以确定在一个Java虚拟机中的唯一性。因此，当多个类装载器都装载了同名的类型时，为了惟一地标识该类型，还要在类型名称前加上装载该类型（指出它所位于的命名空间）的类装载器标识。

#### 五、双亲委派模型（Parent Delegation Model）

类的加载过程采用双亲委托机制，这种机制能更好的保证 Java 平台的安全。该模型要求除了顶层的Bootstrap class loader启动类加载器外，其余的类加载器都应当有自己的父类加载器。子类加载器和父类加载器不是以继承（Inheritance）的关系来实现，而是通过组合（Composition）关系来复用父加载器的代码。每个类加载器都有自己的命名空间（由该加载器及所有父类加载器所加载的类组成，在同一个命名空间中，不会出现类的完整名字（包括类的包名）相同的两个类；在不同的命名空间中，有可能会出现类的完整名字（包括类的包名）相同的两个类。

![](/blog/images/201705/9.png)

双亲委派模型的工作过程为：

1. 前 ClassLoader 首先从自己已经加载的类中查询是否此类已经加载，如果已经加载则直接返回原来已经加载的类。每个类加载器都有自己的加载缓存，当一个类被加载了以后就会放入缓存，等下次加载的时候就可以直接返回了。

2. 当前 classLoader 的缓存中没有找到被加载的类的时候，委托父类加载器去加载，父类加载器采用同样的策略，首先查看自己的缓存，然后委托父类的父类去加载，一直到 bootstrap ClassLoader。

3. 当所有的父类加载器都没有加载的时候，再由当前的类加载器加载，并将其放入它自己的缓存中，以便下次有加载请求的时候直接返回。

   如下图所示：

![](/blog/images/201705/6.png)

​	loader2首先从自己的命名空间查找Sample类是否已经被加载，如果已经加载就直接返回代表Sample类的class对象的引用。如果Sample类还没有被加载。loader2首先请求loader1代为加载，loader1再请求系统类加载器代为加载，系统类加载器再请求扩展类加载器代为加载，扩展类加载器再请求根类加载器代为加载。若根类加载器和扩展类加载器都不能加载，则系统类加载器尝试加载，若能加载成功，则将Sample类所对应的class对象的引用返回给loader1，loader1再将引用返回给loader2，从而成功将Sample类加载进虚拟机。若系统类加载器不能加载Sample类，则loader1尝试加载sample类，若loader1也不能成功加载，则loader2尝试加载。若所有的父加载器及loader2本身都不能加载，则抛出ClassNotFoundException异常。

![](/blog/images/201705/3.png)

以下是ClassLoader抽象类的代码片段：

```java
protected Class<?> loadClass(String name, boolean resolve) throws ClassNotFoundException {
    synchronized (getClassLoadingLock(name)) {
        // First, check if the class has already been loaded
        Class<?> c = findLoadedClass(name);
        if (c == null) {
            long t0 = System.nanoTime();
            try {
                if (parent != null) {
                    c = parent.loadClass(name, false);
                } else {
                    c = findBootstrapClassOrNull(name);
                }
            } catch (ClassNotFoundException e) {
                // ClassNotFoundException thrown if class not found
                // from the non-null parent class loader
            }

            if (c == null) {
                // If still not found, then invoke findClass in order
                // to find the class.
                long t1 = System.nanoTime();
                c = findClass(name);

                // this is the defining class loader; record the stats
                sun.misc.PerfCounter.getParentDelegationTime().addTime(t1 - t0);
                sun.misc.PerfCounter.getFindClassTime().addElapsedTimeFrom(t1);
                sun.misc.PerfCounter.getFindClasses().increment();
            }
        }
        if (resolve) {
            resolveClass(c);
        }
        return c;
    }
}
```

使用这种模型来组织类加载器之间的关系的好处：

- 避免重复加载，当父加载器已经加载了该类的时候，子类加载器就没有必要子再加载一次。

- 考虑到安全因素，我们试想一下，如果不使用这种委托模式，那我们就可以随时使用自定义的String来动态替代java核心api中定义的类型，这样会存在非常大的安全隐患，而双亲委托的方式，就可以避免这种情况，因为String已经在启动时就被引导类加载器（Bootstrcp ClassLoader）加载，所以**用户自定义的ClassLoader永远也无法加载一个自己写的String，除非你改变JDK中ClassLoader搜索类的默认算法。**

####六、自定义类加载器

运行自定义加载器相关的代码，深入理解双亲委派模型。

请查看GitHub源代码：[clazzLoader](https://github.com/wzguo/ClazzLoader)

####七、类的卸载
当类被加载、连接和初始化后，它的生命周期就开始了，当代码该类的Class对象不再被引用，Class对象就会结束生命周期，该类在方法区的数据也会被卸载，从而结束该类的生命周期。
由java虚拟机自带的加载器加载的类在虚拟机的生命周期总始终不会被卸载，java虚拟机本身会始终引用这些类加载器，而这些类加载器则会始终引用它们所加载的类的Class对象，因此这些Class对象始终是可触及的。由用户自定义的类加载器所加载的类是可以被卸载的。
