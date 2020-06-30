---
title: Intellij IDEA打开SpringBoot工程的问题

date: 2018-03-07 15:29:45

author: Zak

avatar: /blog/images/avatar.png

authorLink: http://www.wuzguo.com

authorAbout: https://github.com/wzguo

authorDesc: 一个自强不息，艰苦奋斗的「十八线码农」

categories: 其他

tags:
	- Intellij IDEA

	- SpringBoot

keywords: Intellij IDEA，SpringBoot

photos:
	- /blog/images/avatar.png

description: 解决Intellij IDEA打开SpringBoot工程读取不到配置文件的问题
---

本人习惯使用Intellij IDEA做Java项目开发，最近在新项目中使用SpringBoot时出现一个很奇怪的现象，新建的SpringBoot的项目在我机器上跑的好好的，在其他同事的机器上就是跑不起来，程序直接退出：

```bash
......

Disconnected from the target VM, address: '127.0.0.1:49362', transport: 'socket'
Process finished with exit code 1
```

实在没办法只能debug调试，在`org.springframework.boot.devtools.restart.RestartLauncher` 类的`run`方法中抛出异常：

```java
@Override
public void run() {
   try {
      Class<?> mainClass = getContextClassLoader().loadClass(this.mainClassName);
      Method mainMethod = mainClass.getDeclaredMethod("main", String[].class);
      mainMethod.invoke(null, new Object[] { this.args });
   }
   catch (Throwable ex) {
      this.error = ex;
      getUncaughtExceptionHandler().uncaughtException(this, ex);
   }
}
```

发现没有读取到配置文件。

**经过我不懈努力，主要是找两个环境之间的差异，后来发现是IDEA生成的`iml`文件有问题**，需要找到跟启动的项目名称相同的`.iml`文件（假如你的项目为 hello，那么IDEA一定会给你生成一个 `hello.iml`文件，一般在项目的`.idea`目录下或者项目根目录下。）

解决方案：
​    **在 `hello.iml`文件中加上` <sourceFolder url="file://$MODULE_DIR$/src/main/resources" type="java-resource" />`  后问题得到解决。**

```xml
<?xml version="1.0" encoding="UTF-8"?>
  <module org.jetbrains.idea.maven.project.MavenProjectsManager.isMavenModule="true" type="JAVA_MODULE" version="4">
    <component name="FacetManager">
      <facet type="Spring" name="Spring">
        <configuration/>
      </facet>
    </component>
    <component name="NewModuleRootManager" LANGUAGE_LEVEL="JDK_1_8">
      <output url="file://$MODULE_DIR$/target/classes" />
      <output-test url="file://$MODULE_DIR$/target/test-classes" />
      <content url="file://$MODULE_DIR$">
        <sourceFolder url="file://$MODULE_DIR$/conf/dev" type="java-resource" />
        <sourceFolder url="file://$MODULE_DIR$/src/main/java" isTestSource="false" />
        <sourceFolder url="file://$MODULE_DIR$/src/main/resources" type="java-resource" />   // ##就是这条##
        <sourceFolder url="file://$MODULE_DIR$/src/test/java" isTestSource="true" />
        <excludeFolder url="file://$MODULE_DIR$/target" />
      </content>
    
   ......
```

完。