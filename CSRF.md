Cross Site Request Forgery (CSRF)

# 什么是 跨站伪造请求 攻击？

银行转账表单如下，可以从当前登录用户的账户向另一个账户转账
```html
<form method="post"
    action="/transfer">
<input type="text"
    name="amount"/>
<input type="text"
    name="routingNumber"/>
<input type="text"
    name="account"/>
<input type="submit"
    value="Transfer"/>
</form>
```
对应的http request如下：
```text
POST /transfer HTTP/1.1
Host: bank.example.com
Cookie: JSESSIONID=randomid
Content-Type: application/x-www-form-urlencoded

amount=100.00&routingNumber=1234&account=9876
```
现在用户登录到银行网站，但没有登出；并且访问了一个 恶意站点， 站点的网页表单如下：
```html
<form method="post"
    action="https://bank.example.com/transfer">
<input type="hidden"
    name="amount"
    value="100.00"/>
<input type="hidden"
    name="routingNumber"
    value="evilsRoutingNumber"/>
<input type="hidden"
    name="account"
    value="evilsAccountNumber"/>
<input type="submit"
    value="Win Money!"/>
</form>
```
如果用户点了`Win Money!`按钮，就会无意识的转$100给恶意用户。因为即使恶意站点不知道 cookies， 但是浏览器也会自动填充银行request中的cookie。更糟糕的是， JavaScript可以自动完成这个操作，而不需要用户点击按钮;如果访问了遭到XSS 攻击的网站，就更容易发生CRSF。

## XXS attack

Cross-site Scripting
人们经常将跨站脚本攻击（Cross Site Scripting）缩写为CSS，但这会与层叠样式表（Cascading Style Sheets，CSS）的缩写混淆。因此，有人将跨站脚本攻击缩写为XSS。

攻击者可以使用户在浏览器中执行其预定义的恶意脚本。

XSS攻击通常指的是通过利用网页开发时留下的漏洞，通过巧妙的方法注入恶意指令代码到网页，使用户加载并执行攻击者恶意制造的网页程序。这些恶意网页程序通常是JavaScript，但实际上也可以包括Java、 VBScript、ActiveX、 Flash 或者甚至是普通的HTML。攻击成功后，攻击者可能得到包括但不限于更高的权限（如执行一些操作）、私密网页内容、会话和cookie等各种内容。

##### XXS 原理
HTML是一种超文本标记语言，通过将一些字符特殊地对待来区别文本和标记，例如，小于符号（<）被看作是HTML标签的开始，<title>与</title>之间的字符是页面的标题等等。当动态页面中插入的内容含有这些特殊字符（如<）时，用户浏览器会将其误认为是插入了HTML标签，当这些HTML标签引入了一段JavaScript脚本时，这些脚本程序就将会在用户浏览器中执行。所以，当这些特殊字符不能被动态页面检查或检查出现失误时，就将会产生XSS漏洞。

# 防御 CSRF 攻击

从受害者站点发出的http请求 与 从恶意站点发出的请求完全一样，这使CSRF攻击成为可能。
只要确保http 请求中的某些内容，恶意站点无法提供，就能区分出来。

spring提供了两种防御CSRF的机制
+ `Synchronizer Token Pattern`
+ `SameSite Attribute` on your session cookie
两种机制都需要`Safe Methods Must be Idempotent(幂等)`,也就是`GET``HEAD``OPTIONS``TRACE`类型的HTTP请求不能改变应用的状态，即幂等。

# `Synchronizer Token Pattern`

这是防御CSRF攻击最主要、最全面的的方式。这个方案确保每个HTTP request 除了session cookie之外，还要获取一个安全的随机生成值CSRF token。
server端会验证每个提交过来的http request中的CSRFtoken，如果跟真实值不一样，请求会被拒绝。
CRSF token作为http request的一部分，并且不通过浏览器自动填充。比如放在http的query参数或者header中，但是不能放到cookie中，因为cookie会被浏览器自动渲染，并且cookie作为通用的header 并不安全。
可以将CRSF token认证只作用到改变应用状态的请求中，这就要求`Sage Methods Must be Idempotent`。这样就可以让外部站点连到我们的站点，而不是拒绝所有外部请求，更加合理；而且如果在`GET`请求中包含CSRF token 可能造成泄漏。

加了CSRF token的请求表单示例如下：
```html
<form method="post"
    action="/transfer">
<input type="hidden"
    name="_csrf"
    value="4bfd1575-3ad1-4d21-96c7-4ef2d9f86721"/>
<input type="text"
    name="amount"/>
<input type="text"
    name="routingNumber"/>
<input type="hidden"
    name="account"/>
<input type="submit"
    value="Transfer"/>
</form>
```
CSRF token被隐藏在表单中，外部站点因为同源策略，无法获取到这个表单，也就无法获知token。

对应的http request如下：
```text
POST /transfer HTTP/1.1
Host: bank.example.com
Cookie: JSESSIONID=randomid
Content-Type: application/x-www-form-urlencoded

amount=100.00&routingNumber=1234&account=9876&_csrf=4bfd1575-3ad1-4d21-96c7-4ef2d9f86721
```

CSRF token被放在query参数中传递，字段是`_csrf`。

# `SameSite Attribute`



