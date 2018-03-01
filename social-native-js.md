## Social Verification



### native call js



```
http://42.159.233.39:8000?context=claim:linkedin_authentication&ontid=6469643a6f6e743a5452616a31684377615135336264525450635a78596950415a364d61376a6351564b&encryptedPrivateKey=6PYRC4fgNSq7uVC7dUCLbb9GpjnTcFwLqDMQ2zAAX7NNqH47tfirgsNEQw&deviceCode=
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



# 