---
title: Selenium的基本使用

date: 2018-03-15 14:54:12

author: Zak

avatar: /blog/images/avatar.png

authorLink: http://www.wuzguo.com

authorAbout: https://github.com/wuzguo

authorDesc: 一个追求进步的「十八线码农」

categories: 其他

tags:
	- Selenium

	- WebDriver

	- Python

keywords: Selenium，WebDriver，Python

photos:
	- /blog/images/201803/9.jpg

description: Python+Selenium安装及环境配置以及基本使用
---


1. Selenium 简介

   Selenium 是 ThroughtWorks 一个强大的基于浏览器的开源自动化测试工具，它通常用来编写Web应用的自动化测试。
   Selenium 2，又名 WebDriver，它的主要新功能是集成了 Selenium 1.0 以及 WebDriver（WebDriver 曾经是 Selenium 的竞争对手）。也就是说 Selenium 2 是 Selenium 和 WebDriver 两个项目的合并，即 Selenium 2 兼容 Selenium，它既支持 Selenium API 也支持 WebDriver API。

2. 安装使用

   - 依赖包下载

     Python 下载地址：https://www.python.org/downloads/(下载2.7版本) 

     Selenium RC获取地址：http://selenium-release.storage.googleapis.com/index.html  各种版本

     Selenium IDE获取地址：http://release.seleniumhq.org/selenium-ide/2.9.0/   各种版本

     Selenium(Webdriver)下载地址:https://pypi.python.org/pypi/selenium 各种版本

     Chromedriver获取地址：https://npm.taobao.org/mirrors/chromedriver/ 各种版本

   - 安装

     具体过程赘述。python 安装完成后需要将安装目录加入环境变量，然后在再命令行敲 `python -V` 能正确显示版本号就表示配置完成。我本地的执行效果如下：

     ![](/blog/images/201803/8.jpg)

   - 安装Python的Selenium 依赖包

     1. 通过pip 安装

        `pip install selenium` 

     2. 通过下载包安装
        直接下载Selenium包：https://pypi.python.org/pypi/selenium，解压后进入目录，然后执行：

        `python setup.py install`

     3. 如果在IDE（如：PyCharm）中，引入   `from selenium import webdriver` 后提示未安装`selenium` 报错，可以直接点击结报错的按钮安装。

3. 简单使用

   环境以及服务都安装完成后，就可以写代码测试，我这里使用Python代码写了个简单的demo，如下：

   ```python
   # coding=utf-8

   from selenium import webdriver
   import time

   if __name__ == "__main__":
       path ='D:/test/selenium/chromedriver.exe'
       driver = webdriver.Chrome(executable_path = path)
       driver.implicitly_wait(5)
       driver.maximize_window()
       driver.get("http://www.baidu.com")

       time.sleep(3)
       driver.quit()
   ```

     如果执行正常，Selenium 会在本地打开Chrome浏览器，并进入百度首页，3秒钟后正常关闭浏览器。

4. 远程调用

   以上只能在本地使用，如果要远程执行用例，需要启动 Selenium 服务端，由于Selenium 是一个jar包，所以需要依赖jre，没有安装JDK的需要自己安装，我在本地直接执行命令如下：`java -jar  selenium-server-standalone-2.53.1.jar` ，输入以下信息表示启动成功：

   ![](/blog/images/201803/9.jpg)

   然后执行以下脚本：

   ```python
   # coding=utf-8
   import time

   import selenium
   from selenium import webdriver
   from selenium.webdriver import DesiredCapabilities

   if __name__ == "__main__":
       driver = selenium.webdriver.remote.webdriver.WebDriver(command_executor="http://172.16.0.55:4444/wd/hub",
                                                              desired_capabilities=DesiredCapabilities.CHROME)
   driver.implicitly_wait(5)
   driver.maximize_window()
   driver.get("http://www.baidu.com")

   time.sleep(3)
   driver.quit()

   ```

   效果同上，只是在远程机器上打开浏览器。

5. 各种异常

   - Selenium 放大浏览器报错`disconnected:unable to connect renderer`解决办法

     ![](/blog/images/201803/7.jpg)

     问题的原因是chrome浏览器版本太高跟`chromedriver.exe`不兼容导致的，解决方案：

     1. 降低chrome版本。
     2. 升高`chromedriver.exe`版本，地址：https://npm.taobao.org/mirrors/chromedriver/，我采用这种方式顺利解决。

   - 报错 `'chromedriver' executable needs to be in PATH.`解决办法

     这种错误是执行脚本找不到`chromedriver`导致的，解决方案：

     1. 将 `chromedriver.exe` 文件放在PATH环境变量中，需要注意的是PATH只需要指定 `chromedriver.exe` 文件所在的目录即可。

     ![](/blog/images/201803/5.jpg)

     ![](/blog/images/201803/6.jpg)

     2. 直接将`chromedriver.exe` 文件放在.py程序的目录下。
     3. 在代码中设置：

     ```python
         path ='D:/test/selenium/chromedriver.exe'
         driver = webdriver.Chrome(executable_path = path)
     ```