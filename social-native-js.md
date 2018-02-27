## Social Verification



### 流程图



![](http://on-img.com/chart_image/5a952471e4b04932f89cde03.png)





### native call js



```
socialVerify(context, ontid, encryptedPrivateKey, passwd) {
    // ...
}
```



`context`:

**claim:twitter_authentication** is for twitter authentication.

**claim:facebook_authentication** is for facebook authentication.

**claim:github_authentication** is for github authentication.

**claim:linkedin_authentication** is for linkedin authentication.





### js return claim to native



```
window.prompt('Ont://getSocialClaim?params=' + result)
```



`result`:

```
{
  Error: 0,
  Data: {
      // claim content
  },
  Desc: 'SUCCESS'
}
```



| Error | Desc                       |
| ----- | -------------------------- |
| 0     | SUCCESS                    |
| 101   | PRIVATE KEY PASSWORD ERROR |
| 102   | PARAMETERS ERROR           |
| 103   | OAUTH ERROR                |
| 104   | USER CANCELLED             |
| 105   | BLOCKCHAIN ERROR 入链失败  |



### 将 claim 发送到 区块链



1、构造交易；

2、发送交易；

3、websocket事件等待入链；



如果入链时间过长，或者一直没有入链，则APP一直监听不到prompt，可能APP原生还需要设置一个timeout