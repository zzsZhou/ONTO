## Social Verification



### native call js



```
http://42.159.233.39:8000?context=claim:linkedin_authentication&ontid=6469643a6f6e743a5452616a31684377615135336264525450635a78596950415a364d61376a6351564b&encryptedPrivateKey=6PYRC4fgNSq7uVC7dUCLbb9GpjnTcFwLqDMQ2zAAX7NNqH47tfirgsNEQw&deviceCode=123456789
```



`context`:

**claim:twitter_authentication** is for twitter authentication.

**claim:facebook_authentication** is for facebook authentication.

**claim:github_authentication** is for github authentication.

**claim:linkedin_authentication** is for linkedin authentication.





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



| Error | Desc     |
| ----- | -------- |
| 0     | SUCCESS  |
| 101   | 用户取消 |
| 102   | 网络故障 |
| 103   | 超时     |
| 104   | 参数错误 |
| 105   | 托管错误 |
| 106   | 入链错误 |



**H5中的流程类似于一次事务，如果有任何错误发生，Native不需要对具体错误进行处理，只需要监听到prompt事件就把当前webview销毁，用户只能重来一次；**

# 