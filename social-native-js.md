## Social Verification

  

### native call js

  

```
http://42.159.233.39:8000?context=claim:linkedin_authentication&ontid=6469643a6f6e743a545241746f73555a484e53694c687a426448616379784d5834426733636a57793372&encryptedPrivateKey=6PYQWiEH4p4CigWKeS1YXa7NavtNGMGxo28JDPyEUwr42QrTetLhwwpn5R&deviceCode=device90894eb863304481b55b9ef595d7b8b3&lang=zh_hans
```

  

`context`:  

**claim:twitter_authentication** is for twitter authentication.

**claim:facebook_authentication** is for facebook authentication.

**claim:github_authentication** is for github authentication.

**claim:linkedin_authentication** is for linkedin authentication.

  

`ontid`    

需要传hex编码的 `6469643a6f6e743a545241746f73555a484e53694c687a426448616379784d5834426733636a57793372`，原文为 `did:ont:TRAtosUZHNSiLhzBdHacyxMX4Bg3cjWy3r`  

​    

`encryptedPrivateKey`  

`6PYQWiEH4p4CigWKeS1YXa7NavtNGMGxo28JDPyEUwr42QrTetLhwwpn5R`  

  

`deviceCode`  

  `device90894eb863304481b55b9ef595d7b8b3`



`lang`:   

​    `en_us`     英文  

​    `zh_hans`   中文简体  

  

### js return to native



```
window.prompt('Ont://returnToNative?params=' + result)
```



`result`:

```
{
  Error: 0,
  Desc: 'SUCCESS'
}
```



| Error | Desc            |
| ----- | --------------- |
| 0     | SUCCESS         |
| 101   | 用户取消        |
| 102   | 网络故障        |
| 103   | 超时            |
| 104   | 参数错误        |
| 105   | 托管失败        |
| 106   | 入链失败        |
| 107   | onchain签名失败 |



**H5中的流程类似于一次事务，如果有任何错误发生，Native不需要对具体错误进行处理，只需要监听到prompt事件就把当前webview销毁，用户只能重来一次；**

# 