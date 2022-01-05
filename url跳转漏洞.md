## 漏洞原理
url重定向漏洞也称url任意跳转漏洞，网站信任了用户的输入导致恶意攻击，url重定向主要用来钓鱼，比如url跳转中最常见的跳转在登陆口，支付口，也就是一旦登陆将会跳转任意自己构造的网站，如果设置成自己的url则会造成钓鱼。

## 漏洞分类
- 无需登录跳转
- 需要登录跳转

## 案例
抓取跳转处的数据包，将路径修改为需要他跳转的路径，已百度为例

1. 抓取数据包

![image](https://user-images.githubusercontent.com/71583369/148241787-ac0c4517-04a4-475d-ba63-d66bd542ddf0.png)

2. 修改location处为百度地址

![image](https://user-images.githubusercontent.com/71583369/148241884-116489f5-0356-40e5-bf31-bed59764a932.png)

3. 成功跳转

![image](https://user-images.githubusercontent.com/71583369/148241932-1340b548-de02-4fc5-8ca9-1a463510ae58.png)

## url跳转常出现地方
1. 登陆跳转我认为是最常见的跳转类型，认证完后会跳转，所以在登陆的时候建议多观察url参数
2. 用户分享、收藏内容过后，会跳转
3. 跨站点认证、授权后，会跳转
4. 站内点击其它网址链接时，会跳转
5. 在一些用户交互页面也会出现跳转，如请填写对客服评价，评价成功跳转主页，填写问卷，等等业务，注意观察url。
6. 业务完成后跳转这可以归结为一类跳转，比如修改密码，修改完成后跳转登陆页面，绑定银行卡，绑定成功后返回银行卡充值等页面，或者说给定一个链接办理VIP，但是你需要认证身份才能访问这个业务，这个时候通常会给定一个链接，认证之后跳转到刚刚要办理VIP的页面。

## url跳转常见参数
```
redirect
url
redirectUrl
callback
return_url
toUrl
ReturnUrl
fromUrl
redUrl
request
redirect_to
redirect_url
jump
jump_to
target
to
goto
link
linkto
domain
oauth_callback
```
## url跳转bypass
```
1.最常用的@绕过
url=http://www.aaaa.com@www.xxx.com(要跳转的页面)他有的可能验证只要存在aaaa.com就允许访问，做个@解析，实际上我们是跳转到xxx.com的
2.?号绕过
url=http://www.aaaa.com?www.xxx.com
3.#绕过
url=http://www.aaaa.com#www.xxx.com
4.斜杠/绕过
url=http://www.aaaa.com/www.xxx.com
等等等等
```
## 如何修复
1.使用白名单

2.在可能的情况下，让用户提供在服务器端映射到完整目标 URL 的短名称、ID 或令牌

3.不允许将 URL 作为目标的用户输入

4.如果无法避免用户输入，请确保提供的值有效、适用于应用程序，并且已为用户授权

5.从应用程序中删除重定向功能，并将指向它的链接替换为指向相关目标 URL 的直接链接





