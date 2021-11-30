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
这段代码只是判断了两次输入的密码是否一致，然后就直接带到了数据库中更新。没有任何限制。我们使用burp进行抓吧，制作poc。抓取的数据包如下
![image](https://user-images.githubusercontent.com/71583369/144040680-0a171e47-ad93-4eb7-8e40-ecc7b5114b92.png)
制作poc
![image](https://user-images.githubusercontent.com/71583369/144041080-654bcc40-7e10-48b6-bf0a-65e76d76e744.png)
复制内容，桌面创建一个html文件，将内容粘贴进去
![image](https://user-images.githubusercontent.com/71583369/144041338-c8d76dfa-3395-4d1e-8ba9-37773081d0e7.png)
#
点击测试
![image](https://user-images.githubusercontent.com/71583369/144041697-e7bc499e-15a3-455b-bbf7-e4451b4e8d12.png)
#
出现一下内容说明成功了，最后再验证一下密码就ok了
![image](https://user-images.githubusercontent.com/71583369/144041792-2d831d9f-469c-4352-8e58-286e19767007.png)
## Medium CSRF Source
源码
```
<?php

if( isset( $_GET[ 'Change' ] ) ) {
    // Checks to see where the request came from
    if( stripos( $_SERVER[ 'HTTP_REFERER' ] ,$_SERVER[ 'SERVER_NAME' ]) !== false ) {
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
    }
    else {
        // Didn't come from a trusted source
        echo "<pre>That request didn't look correct.</pre>";
    }

    ((is_null($___mysqli_res = mysqli_close($GLOBALS["___mysqli_ston"]))) ? false : $___mysqli_res);
}

?> 
```
这段代码增加了`stripos()`函数，判断referer中是否含有server_name的信息，如果存在的话直接变更密码，抓取数据包
![image](https://user-images.githubusercontent.com/71583369/144043535-23167589-72d4-4b2b-aee6-02c46d790b41.png)
#
发生错误，这个请求看起来不像是正确的。在referer中加入server_name信息
![image](https://user-images.githubusercontent.com/71583369/144044069-a34c21e8-6004-49ce-b2e8-9c25d00ddfcd.png)
#
可以看到执行成功了。然而上一个poc在执行的时候是没办法利用的，但我在网上看到有人将`1.html`改成了`主机名.html`并上传到了web服务器上，然后去访问这个链接就会带有cookie，但我觉得这从攻击者的角度来说是不可行的，我都拿到了服务器权限了再上传个这样的文件这简直就是多此一举。（可能我菜不知道怎么利用，大佬勿喷啊）
## High CSRF Source
源码
```
<?php

if( isset( $_GET[ 'Change' ] ) ) {
    // Check Anti-CSRF token
    checkToken( $_REQUEST[ 'user_token' ], $_SESSION[ 'session_token' ], 'index.php' );

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

// Generate Anti-CSRF token
generateSessionToken();

?> 
```
这段代码加入了token验证，不能用前两个级别的方法了。在这里，我们要想获取到用户的token,并提交修改密码的表单的话，就必须得把我们的攻击脚本注入到目标服务器中 。而要想注入到目标服务器，同时得发挥作用，获取用户的 token修改密码的话，就得和XSS漏洞一起结合实现了。
#
我们将如下代码通过存储型XSS插入到数据库中，这语句会弹出用户的token
`<iframe src="../csrf/" onload=alert(frames[0].document.getElementsByName('user_token')[0].value)></iframe>`
## Impossible CSRF Source
源码
```
<?php

if( isset( $_GET[ 'Change' ] ) ) {
    // Check Anti-CSRF token
    checkToken( $_REQUEST[ 'user_token' ], $_SESSION[ 'session_token' ], 'index.php' );

    // Get input
    $pass_curr = $_GET[ 'password_current' ];
    $pass_new  = $_GET[ 'password_new' ];
    $pass_conf = $_GET[ 'password_conf' ];

    // Sanitise current password input
    $pass_curr = stripslashes( $pass_curr );
    $pass_curr = ((isset($GLOBALS["___mysqli_ston"]) && is_object($GLOBALS["___mysqli_ston"])) ? mysqli_real_escape_string($GLOBALS["___mysqli_ston"],  $pass_curr ) : ((trigger_error("[MySQLConverterToo] Fix the mysql_escape_string() call! This code does not work.", E_USER_ERROR)) ? "" : ""));
    $pass_curr = md5( $pass_curr );

    // Check that the current password is correct
    $data = $db->prepare( 'SELECT password FROM users WHERE user = (:user) AND password = (:password) LIMIT 1;' );
    $data->bindParam( ':user', dvwaCurrentUser(), PDO::PARAM_STR );
    $data->bindParam( ':password', $pass_curr, PDO::PARAM_STR );
    $data->execute();

    // Do both new passwords match and does the current password match the user?
    if( ( $pass_new == $pass_conf ) && ( $data->rowCount() == 1 ) ) {
        // It does!
        $pass_new = stripslashes( $pass_new );
        $pass_new = ((isset($GLOBALS["___mysqli_ston"]) && is_object($GLOBALS["___mysqli_ston"])) ? mysqli_real_escape_string($GLOBALS["___mysqli_ston"],  $pass_new ) : ((trigger_error("[MySQLConverterToo] Fix the mysql_escape_string() call! This code does not work.", E_USER_ERROR)) ? "" : ""));
        $pass_new = md5( $pass_new );

        // Update database with new password
        $data = $db->prepare( 'UPDATE users SET password = (:password) WHERE user = (:user);' );
        $data->bindParam( ':password', $pass_new, PDO::PARAM_STR );
        $data->bindParam( ':user', dvwaCurrentUser(), PDO::PARAM_STR );
        $data->execute();

        // Feedback for the user
        echo "<pre>Password Changed.</pre>";
    }
    else {
        // Issue with passwords matching
        echo "<pre>Passwords did not match or current password incorrect.</pre>";
    }
}

// Generate Anti-CSRF token
generateSessionToken();

?> 
```
这段代码可以看到在token的基础上还需要知道用户之前的密码，攻击者不知道用户之前的密码所以无法进行CSRF攻击。
## 漏洞的特征
- 利用网站对用户标识的信任
- 欺骗用户的浏览器发送HTTP请求给目标站点
- 依靠用户标识危害网站
## 如何判断漏洞
- referer空内容判断
- 猜测referer内容判断
## 漏洞防护措施
- referer防护
- 验证码防护
- token防护（采用随机数无法被爆破猜解出来）
