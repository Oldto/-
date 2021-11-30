# CSRF漏洞
## 漏洞描述
CSRF：CSRF（Cross-site request forgery）跨站请求伪造，也被称为“One Click Attack”或者Session Riding，通常缩写为CSRF或者XSRF，是一种对网站的恶意利用。尽管听起来像跨站脚本（XSS），但它与XSS非常不同，XSS利用站点内的信任用户，而CSRF则通过伪装成受信任用户的请求来利用受信任的网站。与XSS攻击相比，CSRF攻击往往不大流行（因此对其进行防范的资源也相当稀少）和难以防范，所以被认为比XSS更具危险性。

攻击者通过伪造一个链接，将链接发送给目标用户，欺骗用户点击，正如上述概念所说，该链接实际上是获取某网站信任用户信息的链接，目的是在用户电脑上登陆该网站，一旦用户点击了该链接，实际上攻击就完成了。
#
举个例子：构造一个网站的链接，如删除网站中的某个内容`http：//xxx.com/del.php?id=1`,只有管理员才能删除，这时我们伪造这个链接，把他发给管理员，诱导他点击这个链接，随后该内容就会被删除(前提是用户已登录了这个网站，还有就是必须使用相同的浏览器打开，不同的浏览器是不成功的；因为这个和浏览器的同源策略有关)。然而管理员却毫不知情。说简单点，就是伪造一个链接，让用户点击，然后就能达到我们需要的操作，比如添加一个账号、增加、删除某条信息等。
## 漏洞利用
## Low CSRF Source
源码
```
<?php

if( isset( $_GET[ 'Change' ] ) ) {
    // Get input
    $pass_new  = $_GET[ 'password_new' ];
    $pass_conf = $_GET[ 'password_conf' ];

    // Do the passwords match?
    if( $pass_new == $pass_conf ) {
        // They do!
        $pass_new = ((isset($GLOBALS["___mysqli_ston"]) && is_object($GLOBALS["___mysqli_ston"])) ? mysqli_real_escape_string($GLOBALS["___mysqli_ston"],  $pass_new ) : ((trigger_error("[MySQLConverterToo] Fix the mysql_escape_string() call! This code does not work.", E_USER_ERROR)) ? "" : ""));
        $pass_new = md5( $pass_new );

        // Update the database
        $insert = "UPDATE `users` SET password = '$pass_new' WHERE user = '" . dvwaCurrentUser() . "';";
        $result = mysqli_query($GLOBALS["___mysqli_ston"],  $insert ) or die( '<pre>' . ((is_object($GLOBALS["___mysqli_ston"])) ? mysqli_error($GLOBALS["___mysqli_ston"]) : (($___mysqli_res = mysqli_connect_error()) ? $___mysqli_res : false)) . '</pre>' );

        // Feedback for the user
        echo "<pre>Password Changed.</pre>";
    }
    else {
        // Issue with passwords matching
        echo "<pre>Passwords did not match.</pre>";
    }

    ((is_null($___mysqli_res = mysqli_close($GLOBALS["___mysqli_ston"]))) ? false : $___mysqli_res);
}

?> 
```
这段代码只是判断了两次输入的密码是否一致，然后就直接带到了数据库中更新。没有任何限制。我们使用burp进行抓吧，制作poc。






