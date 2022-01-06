# Day 19 任意文件下载【读取】漏洞

**漏洞危害**

- ​	获取网站的后端源码造成源码泄露
- ​	造成数据库等敏感文件泄露
  - 短信接口KEY
  - Email发送账户密码

**任意文件下载漏洞案例1**

某网站下载功能处进行抓包

![image-20210726234728513](C:\Users\10637\AppData\Roaming\Typora\typora-user-images\image-20210726234728513.png)

burp抓包

![image-20210726234943950](C:\Users\10637\AppData\Roaming\Typora\typora-user-images\image-20210726234943950.png)

抓包分析path后接文件路径。

文件下载的内容如下

![image-20210726235033784](C:\Users\10637\AppData\Roaming\Typora\typora-user-images\image-20210726235033784.png)

修改文件路径和文件名，如图

![image-20210726235230527](C:\Users\10637\AppData\Roaming\Typora\typora-user-images\image-20210726235230527.png)

下载index.php文件，查看源代码，发现其框架为thinkphp

通过框架判断出存放敏感数据的文件

![image-20210726235555340](C:\Users\10637\AppData\Roaming\Typora\typora-user-images\image-20210726235555340.png)

获取到数据库的敏感信息。

**任意文件下载漏洞案例2**

对某网站进行任意文件下来漏洞挖掘，**首先要从网站上查找下载的功能点**。<!--这里有被自己蠢哭，没有找下载的功能点，而是直接御剑扫的后台。连传参都不清楚怎么传，就拿path一直试，卧槽，真是秀逗了。-->

这里找到下载的功能点

![image-20210727000212059](C:\Users\10637\AppData\Roaming\Typora\typora-user-images\image-20210727000212059.png)

点击下载，开启抓包

![image-20210727000330261](C:\Users\10637\AppData\Roaming\Typora\typora-user-images\image-20210727000330261.png)

可以看到是通过fpath来提交的参数，fname不需要修改。将fpath修改为fpath=index.php

![image-20210727000735927](C:\Users\10637\AppData\Roaming\Typora\typora-user-images\image-20210727000735927.png)

将数据包提交，发现源码

![image-20210727000814139](C:\Users\10637\AppData\Roaming\Typora\typora-user-images\image-20210727000814139.png)

首先说下include_once的含义

### include_once

> include_once 函数会将指定的文件载入并执行里面的程序；此行为和 include 语句类似，唯一区别是如果该文件中已经被包含过，则不会再次包含。

然后将文件名修改为config.php

![image-20210727001215204](C:\Users\10637\AppData\Roaming\Typora\typora-user-images\image-20210727001215204.png)

再将require_once的文件名添加到fpath中

![image-20210727001349244](C:\Users\10637\AppData\Roaming\Typora\typora-user-images\image-20210727001349244.png)

![image-20210727001519788](C:\Users\10637\AppData\Roaming\Typora\typora-user-images\image-20210727001519788.png)



得到如下代码，通过代码分析，发现两处路径

一次尝试，查找敏感数据

`$_CONFIG = parse_ini_file(dirname(__FILE__).'/config.ini',true);`

`JPDO::initConnFactory(dirname(__FILE__).'/JPDO.ini');`

**dirname(FILE) 取到的是当前文件的绝对路径**

这样就很好理解，尝试了好几次，不知道代码什么意思，直接就把文件名填上去了，就是出不来数据。

添加文件名

![image-20210727001935909](C:\Users\10637\AppData\Roaming\Typora\typora-user-images\image-20210727001935909.png)

没有想要的数据，添加下一个文件

![image-20210727002050301](C:\Users\10637\AppData\Roaming\Typora\typora-user-images\image-20210727002050301.png)

得到数据的登录密码

通过phpmyadmin可以直接登录数据库，进行操作。

![image-20210727002158433](C:\Users\10637\AppData\Roaming\Typora\typora-user-images\image-20210727002158433.png)

登录成功。