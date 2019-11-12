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
如果用户点了`Win Money!`按钮，就会无意识的转$100给恶意用户。因为即使恶意站点不知道 cookies， 但是浏览器也会自动填充银行request中的cookie。更糟糕的是， JavaScript可以自动完成这个操作，而不需要用户点击按钮。
