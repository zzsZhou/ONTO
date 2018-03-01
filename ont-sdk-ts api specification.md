# ONT SDK 接口文档

Notice:This sdk exports everything in one global variable "Ont". You should use any class or object as the attribute of Ont.For example, If you want to use class Account, you should use Ont.Account.

注意：这个sdk 到处了一个包含所有内容的全局变量“Ont”。使用时需要在Ont的命名空间下。如如使用类Account，需要写成“Ont.Account”。

## 1 钱包 Wallet

钱包Wallet是一个Json格式的数据存储文件。在本体Ontology中， Wallet既可存储数字身份，也可以存储数字资产。

### Wallet 数据规范

```
{
	name: string;
    ontid: string;
    createTime: string;
    version: string;
    scrypt: {
        "n": number;
        "r": number;
        "p": number;
    };
    identities: Array<Identity>;
    accounts: Array<Account>;
    extra: null;
}
```

`name` 是用户为钱包所取的名称。

```ontid``` 是钱包唯一ontid。

```createTime``` 是ISO格式表示的钱包的创建时间，如 : "2018-02-06T03:05:12.360Z"

`version` 目前为固定值1.0，留待未来功能升级使用。

`scrypt` 是加密算法所需的参数，该算法是在钱包加密和解密私钥时使用。

`identities` 是钱包中所有数字身份对象的数组

```accounts``` 是钱包中所有数字资产账户对象的数组

```extra``` 是客户端由开发者用来存储额外数据字段，可以为null。

希望了解更多钱包数据规范请参考[Wallet_File_Specification](https://github.com/ontio/opendoc/blob/master/resources/specifications/Wallet_File_Specification.md).

### 1.1 创建钱包

用户可以通过传递钱包名称和密码来创建新的钱包。创建过程分为两步：**创建钱包文件**和**注册ONT ID**。

**创建钱包文件**所需参数说明如下：

**name** 钱包名称

**password** 密码。用来加密解密私钥。

**callback** 原生里定义的回调函数名，用来接收sdk返回的结果。关于sdk和原生交互的详细情况见 [5.sdk和原生的交互](#5)

该方法会通过回调将创建好的钱包文件，返回给原生。此时，钱包还并未真正创建，需要将钱包中身份的ONT ID注册到链上，当链推送回正确消息时，创建成功。

```
var walletDataStr = Ont.SDK.createWallet('zhangsan', '123456', 'callback')
console.log(walletDataStr)
```

**注册ONT ID** 所需参数说明如下：

**walletDataStr** 上一步创建的钱包文件，JSON字符串格式

**callback** 原生的回调函数名

````
//register ONT ID
Ont.SDK.registerOntid(walletDataStr, callback)
````

该方法会启动一个websocket客户端，将参数构建为交易对象，发到链上，并监听链返回交易确认的消息，这个过程预计需要6秒，所以在回调被调用前，原生应显示等待状态。通过判断回调返回的消息，确认注册是否成功。

如果返回的消息中包含错误，错误码详见[7.错误码](#7) ，原生可以根据错误情况，选择重新注册，或其他处理。

## 1.2 导入身份

导入身份需要用户提供加密后的私钥和密码。

导入的过程是：先检查加密后的私钥和密码是否正确。然后根据私钥生成ONT ID, 并检查ONT ID是否在链上。如果不存在，回调返回相应错误码；如果存在，则生成身份和钱包文件，把身份加入到钱包中，回调返回钱包文件。

```
Ont.SDK.importIdentity(encryptedPrivateKey, password, callback)
```

## 2. Identity 

### Class identity has following structure:

```
{
  ontid : string,
  label : string,
  isDefault : boolean,
  lock : boolean,
  controls : array of controlData,
  extra : null
}
```

`ontid` is the ontid of the identity.

`label` is a label that the user has made to the identity.

`isDefault` indicates whether the identity is the default identity.

`lock` indicates whether the identity is locked by user. The client shouldn't update the infomation in a locked identity.

`controls` is an array of Controller objects which describe the details of each controller in the identity.

`extra` is an object that is defined by the implementor of the client for storing extra data. This field can be null.

### Class controlData has following structure:

```
{
  algorithm: string;
  parameters: {
  	curve: string;
  };
  id: string;
  key: string;
}
```

`algorithm` is the algorithms used in encryption system.

`parameters` is the array of parameter objects used in encryption system.

`id` is the identify of this control.

`key` is the private key of the account in the NEP-2 format. This field can be null (for watch-only address or non-standard address).

```curve``` is the name of the elliptic curve.

### 2.1 Create an identity

You need to pass these parameters:

**privateKey** the private key

**password** the password

**label** the label of the identity

**callback** the callback name on native side. Sdk will send back result with it.

This method will generate one Ont ID for the idenity and register the Ont ID to the blockchain. If the register process meets error, sdk will send result with error info back to native side. 

```
//call createSecp256r1, return a identityData json string
var identityDataStr = Ont.SDK.createIdentity(privateKey, password, label, callback)
console.log(identityDataStr)
```

## 3. Acccount

### Class account has following basic structure:

```
{
  address : string,
  label : string,
  isDefault : boolean,
  lock : boolean,
  algorithm : boolean,
  parameters : {
    curve : string
  },
  key : string,
  contract : {}
  extra : null
}
```

```address``` is the base58 encoded address of the acccount.

```label``` is a label that user has made to the account.

`isDefault` indicates whether the account is the default change account.

`lock` indicates whether the account is locked by user. The client shouldn't spend the funds in a locked account.

`algorithm` is the algorithms used in encryption system.

`parameters` is the parameter objects used in encryption system.

`key` is the private key of the account in the NEP-2 format. This field can be null (for watch-only address or non-standard address).

`contract` is a Contract object which describes the details of the contract. This field can be null (for watch-only address).

`extra` is an object that is defined by the implementor of the client for storing extra data. This field can be null.

### Contract has the following structure:

```
{
  script : string,
  parameters : [],
  deployed : boolean
}
```

`script` is the script code of the contract. This field can be null if the contract has been deployed to the blockchain.

`parameters` is an array of Parameter objects which describe the details of each parameter in the contract function.

`deployed` indicates whether the contract has been deployed to the blockchain.

### 3.1 Create an Account

You need to pass these parameters:

**privateKey** the private key

**password** the password

**label** the label of the identity

**callback** the callback name on native side. Sdk will send back result with it.

This method will send result back to native side .

```
//create an account by using secp256r1, return a json string
var accountDataStr = Ont.SDK.createAccount( privateKey, '123456', 'zhangsan', callback)
console.log(accountDataStr)
```

## 4 Claim

### Claim has following structure:

```
{
  unsignedData : string,
  signedData : string,
  Context : string,
  Id : string,
  Content : {},
  Metadata : Metadata,
  Signature : Signature
}
```

```unsignedData``` is the json string of claim body, which includes Context, Id, Claim and Metadata.

```signedData ``` is the json string of signed claim, which includes claim body and signature.

```Context``` is the identity of claim template. We have following alternative values :

**claim:eid_authentication** is for Chinese citizen ceritification.

**claim:twitter_authentication** is for twitter authentication.

**claim:facebook_authentication** is for facebook authentication.

**claim:github_authentication** is for github authentication.

**claim:linkedin_authentication** is for linkedin authentication.

```Id``` is the identity of the claim.

```Content``` is the content of the claim, which is a json object.

```Metadata``` is the metadata of the claim.

### Class Metadata has following structure:

```
{
  CreateTime : datetime,
  Issuer : string,
  Subject : string,
  Expires : date,
  Revocation : string,
  Crl : string
}
```

```Createtime``` is the creation time of the claim. It's in ISO format: yyyyMMdd'T'HHmmss'Z' .

```Issuer``` is the ontid of the claim issuer. **For self-certified claim, this is the user's ontid. **

```Subject``` is the subject of the claim. **For self-certified claim, this is the user's ontid.**

```Expires``` is the expire time of the claim. The value can be null if no expires.

```Revocation``` is the revocation method of the claim.**For self-certified claim, this value is no need.**

```Crl``` is the link to the revocation list..**For self-certified claim, this value is no need.**

### Class Signature has following structure:

```
{
	Format : string,
    Algorithm : string,
    Value : string
}
```

```Format``` is the the format of the signature.

```Algorithm``` is the signature algorightm.

```Value``` is the signed value of the signature.

### 4.1 Sign a self-certified claim

You should pass following parameters to create a signed claim:

**context** the identity of claim template.

**claimData** is the json string of the claim content. Here we have a simple example below.

**ontid** is user's Ont Id.

**encryptedPrivateKey** is user's encrypted privateKey .

**password** is user's password.

This method will check the ontid, encryptedPrivateKey and password. It will return signed json string as result. If anything goes wrong, it will return result with error info.

```
//sign a claim with specific parameters and user's private key.
var claimBody = {name:'zhangsan', age:'25'}
var claimData = JSON.stringify(claimBody)
var claim = Ont.SDK.signSelfClaim(context, claimData, ontid, encryptedPrivateKey,password, callback)
```

### 4.2 Make a signature

This is a common method to make a signature for any data you want. You need to offer data string and private key. There is one optional  parameter called "callback", which is the callback name on native side.

```
let signatureValue = Ont.core.signatureData(data, privateKey, callback)
console.log(signatureValue)
```

### 4.3 Decrypt encrypted privte key with password

For secuity, user can only access encrypted privte key. User can get the private key with password by decrypting the encrypted private key.

This method will return the private key. If encryptedPrivateKey or password is error, it will return result with error info.

```
var privateKey = Ont.SDK.decryptEncryptedPrivateKey( encryptedPrivateKey, password, callback)
```

### 4.4 Encrypt private key with password

Here is the method to encrypt private key with password.

```
var encryptedPrivateKey = Ont.SDK.encryptPrivateKey( privateKey, password, callback)
```

### 4.5 发送认证声明到链上并验证是否发送成功

用户可以通过websocket发送声明到链上，并且通过持续监听websocket服务器返回的消息，判断发送是否成功。

用户需要传以下参数：

**path** 是发送的属性在DDO中存储的完整路径。

**value** 发送的属性值

**ontid** 用户身份的ONT ID

**privateKey** 用户的私钥

用户可以监听到websocket后台推送的消息，如果是Event的推送，可以得到如下结果：

```
{
	"Action": "Notify",
	"Desc": "SUCCESS",
	"Error": 0,
	"Result": {
		"Container": "ea02f7d3c828c79c65c198e016554d6c8ea7a7502dc164d649afe2c0059aa2b1",
		"CodeHash": "8665eebe481029ea4e1fcf32aad2edbbf1728beb",
		"State": [{
			"Value": [{
				"Value": "417474726962757465"
			}, {
				"Value": "757064617465"
			}, {
				"Value": "6469643a6f6e743a5452616a31684377615135336264525450635a78596950415a364d61376a6351564b"
			}, {
				"Value": "436c616d3a74776974746572"
			}]
		}],
		"BlockHeight": 37566
	},
	"Version": "1.0.0"
}
```



```
//1. call sdk to make the parameter that will be sent to websocket server
var param = Ont.SDK.buildClaimTx( path, value, ontid, privateKey)

//2.send websocket request and listen messages
//websocket服务器地址
const socket_url = 'ws://192.168.3.128:20335'
const socket = new WebSocket(socket_url)
    socket.onopen = () => {
        console.log('connected')
        //发送请求
        socket.send(param)
    }
    //监听消息
    socket.onmessage = (event) => {
		//验证结果
		let res 
        if(typeof event.data === 'string') {
            res = JSON.parse(event.data)
        }
        //解析后台推送的Event通知
        //通过简单的判断区块高度，得知上链成功，
        if(res.Result.BlockHeight) {
            //通知客户端
        }
        
    }
    socket.onerror = (event) => {
        //no server or server is stopped
        console.log(event)
        socket.close()
    }
```



### 5   sdk和原生的交互 

### 5.1 Native call sdk

This sdk will export everything in one js file and one global variable named "Ont". You can call methods directly.For example, use  the "jsBridge" technology. If you want to get the return value of sdk's methods. Normally，You should pass one extra parameter named "callback", it's the function name on the native side. Sdk will call the callback to send result back to native.

### 5.2 Sdk send result back to native

There is one protocol between webview and native.

```
Ont://callback?params=result
```

```Ont``` is fixed.

```callback``` is the callback name on native side.

```params=result``` is sent to native side to get the result. Result is json string.

```
// native side call sdk to create a wallet, the callback name is 'getWalletDataStr'
var walletDataStr = Ont.Wallet.create('zhangsan', '123456', 'getWalletDataStr')
//sdk side will call prompt()
window.prompt('Ont://getWalletDataStr?params={walletDataStr}')
/*
* native side will intercept prompt and get the url
* then, native side can use getWalletDataStr method to handle result
*/
getWalletDataStr(walletSataStr) {
  //......
}
```

### 6 Qrcode specification for backup identity

The Qrcode should contain following content:

```
identity json string & encryptedPrivateKey & ontid
```

### 7 错误码

| 返回代码 | 描述信息                      | 说明                              |
| :------- | ----------------------------- | --------------------------------- |
| 0        | SUCCESS                       | 成功                              |
| 41001    | SESSION_EXPIRED               | 会话无效或已过期（ 需要重新登录） |
| 41002    | SERVICE_CEILING               | 达到服务上限                      |
| 41003    | ILLEGAL_DATAFORMAT            | 不合法数据格式                    |
| 42001    | INVALID_METHOD                | 无效的方法                        |
| 42002    | INVALID_PARAMS                | 无效的参数                        |
| 42003    | INVALID_TOKEN                 | 无效的令牌                        |
| 43001    | INVALID_TRANSACTION           | 无效的交易                        |
| 43002    | INVALID_ASSET                 | 无效的资产                        |
| 43003    | INVALID_BLOCK                 | 无效的块                          |
| 44001    | UNKNOWN_TRANSACTION           | 找不到交易                        |
| 44002    | UNKNOWN_ASSET                 | 找不到资产                        |
| 44003    | UNKNOWN_BLOCK                 | 找不到块                          |
| 45001    | INVALID_VERSION               | 协议版本错误                      |
| 45002    | INTERNAL_ERROR                | 内部错误                          |
| 60000    | NETWORK_ERROR                 | 网络错误                          |
| 60001    | DB_OP_ERROR                   | 数据库操作错误                    |
| 60002    | NO_BALANCE                    | 余额不足                          |
| 60003    | Decrypto_ERROR                | 解密错误                          |
| 60004    | Encrypto_ERROR                | 加密错误                          |
| 60005    | Deserialize_BLOCK_ERROR       | 反序列化Block错误                 |
| 60006    | Deserialize_TRANSACTION_ERROR | 反序列化Transaction错误           |
| 60007    | ComposeIssTransaction_ERROR   | 组合交易Iss错误                   |
| 60008    | ComposeTrfTransaction_ERROR   | 组合交易Trf错误                   |
| 60009    | Signature_INCOMPLETE          | 签名未完成                        |
| 60011    | IllegalArgument               | 不合法参数                        |
| 60012    | IllegalAddress                | 不合法地址                        |
| 60013    | IllegalAssetId                | 不合法资产编号                    |
| 60014    | IllegalAmount                 | 不合法数值                        |
| 60015    | IllegalTxid                   | 不合法交易编号                    |







