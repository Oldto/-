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
















