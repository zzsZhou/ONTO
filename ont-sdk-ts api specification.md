# ONT SDK 接口文档

注意：这个sdk 到处了一个包含所有内容的全局变量“Ont”。使用时需要在Ont的命名空间下。如如使用类Account，需要写成“Ont.Account”。

## 1 钱包 Wallet

钱包Wallet是一个Json格式的数据存储文件。在本体Ontology中， Wallet既可存储数字身份，也可以存储数字资产。

### Wallet 数据规范

```
{
	name: string;
    defaultOntid: string;
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

```defaultOntid``` 是钱包默认身份的Ont ID，可在显示时使用。

```createTime``` 是ISO格式表示的钱包的创建时间，如 : "2018-02-06T03:05:12.360Z"

`version` 目前为固定值1.0，留待未来功能升级使用。

`scrypt` 是加密算法所需的参数，该算法是在钱包加密和解密私钥时使用。

`identities` 是钱包中所有数字身份对象的数组。

```accounts``` 是钱包中所有数字资产账户对象的数组。

```extra``` 是客户端由开发者用来存储额外数据字段，可以为null。

希望了解更多钱包数据规范请参考[Wallet\_File_Specification](https://github.com/ONTIO-Community/ONTO/blob/master/Wallet_File_Specification.md).

### 1.1 创建新的身份钱包

用户可以通过传递钱包名称和密码来创建新的钱包。创建过程分为两步：**创建钱包文件**和**注册ONT ID**。

**创建钱包文件**所需参数说明如下：

**name** 钱包名称

**password** 密码。用来加密解密私钥。

**callback** 原生里定义的回调函数名，用来接收sdk返回的结果。关于sdk和原生交互的详细情况见 [sdk和原生的交互](#5)。
该方法会通过回调将创建好的钱包文件，以JSON字符串的形式，返回给原生。在创建的过程中，需要将钱包中身份的ONT ID注册到链上，注册成功时，代表钱包和身份创建成功。

注册的过程是通过发送相关的交易到链上并被区块接收，这个过程比较耗时，客户端一直等待将会十分影响用户体验，所以注册的过程只需要发送交易，不需要等待确认结果。只要保证交易成功创建和发送，就极大概率上保证上链成功。

注册的过程会耗时约**5秒钟**。

```

Ont.SDK.createWallet('zhangsan', '123456', 'callback')

```

这个方法内部会创建出相应的交易对象，通过websocket或http发送请求到链上。当交易成功发送后，回调返回结果，具体结果详见[SDK和原生的交互](#5)

````
{
	error : string, //错误码 
	desc : string // 错误描述
}
````



创建的流程图详见如下：

![创建身份流程图](http://on-img.com/chart_image/5a9919cce4b09ac3a0c5cbec.png)

如果返回的消息中包含错误，错误码详见[7.错误码](#7) ，原生可以根据错误情况，选择重新注册，或其他处理。

如果返回成功结果的消息，则继续做成功创建的后续处理。

### 1.2 通过导入身份创建钱包

导入身份需要用户提供**身份的名称，加密后的私钥**和**密码**。

**label** 是要导入的身份的名称，可以为空，为空时，sdk会提供自定义生成的名称。

**encryptedPrivateKey ** 加密后私钥，

**password** 密码。

导入的过程是：先检查加密后的私钥和密码是否正确。然后根据私钥生成ONT ID, 并检查ONT ID是否在链上。如果不存在，回调会返回相应错误码；如果存在，则生成身份和钱包文件，把身份加入到钱包中，回调返回钱包文件。

导入身份的流程图如下：

![导入身份](http://on-img.com/chart_image/5a991d34e4b0337bad178b8c.png)

```
Ont.SDK.importIdentity(label,encryptedPrivateKey, password, callback)
```

### 1.3 导入身份到本地钱包

可以通过导入的方式将身份添加到本地钱包。

该方法需要传入的参数如下：

**walletDataStr** 本地钱包文件，JOSN字符串格式。

**label** 导入的身份的名称，可以为空值。

**encryptedPrivateKey** 加密后的私钥。

**password** 用户的密码。

**callback** 回调函数名。

导入的过程同[1.2 通过导入身份创建钱包](#2)

````
Ont.SDK.importIdentityWithWallet( walletDaatStr, label, encryptedPrivateKey, password, callback)
````

## 2. 身份Identity 

#### Identity 具有以下数据结构

```
{
  ontid : string,
  label : string,
  isDefault : boolean,
  lock : boolean,
  controls : array of ControlData,
  extra : null
}
```

```ontid``` 是代表身份的唯一的id

`label` 是用户给身份所取的名称。

`isDefault` 表明身份是用户默认的身份。默认值为false。

`lock` 表明身份是否被用户锁定了。客户端不能更新被锁定的身份信息。默认值为false。

`controls` 是身份的所有控制对象**ControlData**的数组。

`extra` 是客户端开发者存储额外信息的字段。可以为null。

#### ControlData 具有以下数据结构

```
{
  algorithm: string;
  parameters: {};
  id: string;
  key: string;
}
```

`algorithm` 是用来加密的算法名称。

`parameters` 是加密算法所需参数。

`id` 是control的唯一标识。

`key` 是NEP-2格式的私钥。该字段可以为null（对于只读地址或非标准地址时）。

## 3 账户 Account

#### Account 具有以下数据结构。

```
{
  address : string,
  label : string,
  isDefault : boolean,
  lock : boolean,
  algorithm : boolean,
  parameters : {},
  key : string,
  contract : {}
  extra : null
}
```

```address``` 是base58编码的账户地址。

```label``` 是账户的名称。

`isDefault` 表明账户是否是默认的账户。默认值为false。

`lock` 表明账户是否是被用户锁住的。客户端不能消费掉被锁的账户中的资金。

`algorithm` 是加密算法名称。

`parameters` 是加密算法所需参数。

`key` 是NEP-2格式的私钥。该字段可以为null（对于只读地址或非标准地址）。

`contract` 是智能合约对象。该字段可以为null（对于只读的账户地址）。

`extra` 是客户端存储额外信息的字段。该字段可以为null。

#### Contract 具有以下数据结构

```
{
  script : string,
  parameters : [],
  deployed : boolean
}
```

`script` 是智能合约的脚本。当合约已经被部署到区块链上时，该字段可以为null。

`parameters` 是智能合约中函数所需的参数对象，组成的数组。

`deployed` 表明合约是否已被部署到区块链上。默认值为false。

### 3.1 创建账户

用户需要传的参数说明如下：

**password** 密码

**label** 账户的名称

**callback** 回调函数名。

该方法会通过回调，将创建的账户返回JSON格式的

```
//create an account by using secp256r1, return a json string
var accountDataStr = Ont.SDK.createAccount( privateKey, '123456', 'zhangsan', callback)
console.log(accountDataStr)
```

### 4 声明 Claim

#### Claim 具有以下数据结构

```
{
  unsignedData : string,
  SignedData : string,
  Context : string,
  Id : string,
  Content : {},
  Metadata : Metadata,
  Signature : Signature
}
```

```unsignedData``` 是未被签名的声明对象的json格式字符串，声明对象包含Context, Id, Claim, Metadata这些字段。

```SignedData ``` 是声明对象被签名后的json格式字符串，该json包含声明对象和签名对象。

```Context``` 是声明模板的标识。目前有以下可选的值：

**claim:eid_authentication** 代表中国公民认证。

**claim:twitter_authentication** 代表twitter的认证。

**claim:facebook_authentication** 代表facebook的认证。

**claim:github_authentication** 代表github的认证。

**claim:linkedin_authentication** 代表linkedin的认证。

```Id``` 是声明对象的标识。

```Content``` 是声明的内容，格式为JSON对象。

```Metadata``` 是声明对象的元数据。

#### Metadata 具有以下数据结构

```
{
  CreateTime : datetime string
  Issuer : string,
  Subject : string,
  Expires : datetime string
  Revocation : string,
  Crl : string
}
```

```Createtime``` 是声明的创建时间。

```Issuer``` 是声明的发布者。对于自认证声明，该值是用户的ONT ID。

``Subject`` 是声明的主语。对于自认证声明，该值是用户的ONT ID。

```Expires``` 是声明的过期时间。该值可以为空，标识没有过期时间。

```Revocation``` 是声明撤销方法。对于自认证声明，不需要该值。

```Crl``` 是声明撤销列表的链接。对于自认证声明，不需要该值。

#### Signature 具有以下数据结构

```
{
	Format : string,
    Algorithm : string,
    Value : string
}
```

```Format``` 是签名的格式。

```Algorithm``` 是签名的算法。

```Value``` 是计算后的签名值。

### 4.1 构造自认证声明并签名该声明

该方法所需参数说明如下：

**context** 声明模板的标识。该值为字符串。

**claimData** 要声明的内容，JSON字符串格式。

**ontid** 用户的ONT ID。

**encryptedPrivateKey** 用户加密后的私钥。

**password** 用户的密码。

**callback** 回调函数名。该值为可选参数。

当传入参数正确，返回声明对象，该对象中的signedData是签名后的数据。

如果传入的callback，会通过回调返回声明对象的JSON格式字符串。

````
var claim = Ont.SDK.signSelfClaim(context, claimData, ontid, encryptedPrivateKey, password, callback)
````

### 4.2 工具方法 -- 生成签名

用于对输入内容生成签名的方法，该方法会返回签名后的字符串。

```
let signatureValue = Ont.core.signatureData(data, privateKey)
console.log(signatureValue)
```

### 4.3 工具方法 — 解密私钥

系统中为了安全，避免直接操作明文私钥。在需要使用明文私钥时，SDK提供了根据密码解密出明文私钥的方法。

如果能够解出私钥，该方法通过回调返回私钥；如果不能解出，会返回相应错误码。

```
var encryptedPrivateKey = Ont.SDK.decryptEncryptedPrivateKey( encryptedPrivateKey, password, callback)
```

### 4.4 发送认证声明到链上并验证是否发送成功

用户可以通过websocket发送声明到链上，并且通过持续监听websocket服务器返回的消息，判断发送是否成功。

用户需要传以下参数：

**path** 是发送的属性在DDO中存储的完整路径。规定该值为认证声明签名后的值。

**value** 发送的属性值。规定该值为如下结构：

````
{
    Context : string, //声明模板的标识
    Ontid   : string  //声明签发者的ONT ID
}
````

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
const socket_url = 'ws://52.80.115.91:20335'
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

**注意：** 发送认证上链并监听返回的过程比较耗时，如果一直等待确认上链成功，会十分影响用户体验，可以在成功发送交易后关闭websocket连接，并在等待**约5秒**后显示上链成功。

## 5   SDK和原生的交互 

### 5.1 原生调用SDK

原生可以通过于js交互的技术，直接调用SDK中暴露的内容，如“jsBridge”技术。

如果想要得到SDK中方法的返回值，需要通过传递参数callback给SDK的方法，callback是原生定义的回调方法名，SDK在方法调用结束后，通过发起prompt事件，将结果和callback发出，原生拦截prompt事件，获取到函数结果和callback，运行相应方法，处理函数结果。

### 5.2 SDK返回结果给原生

原生的webview和SDK之间定义的协议如下：

```
Ont://callback?params=result
```

```Ont``` 是固定值。

```callback``` 是原生定义的回调函数名。

```params=result``` result就是发送给原生的函数调用结果。该结果是JSON格式。

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

#### 返回结果是如下结构：

````
{
    error  : number,  //错误码
    result : string,  //JSON字符串格式的结果
    desc   : string   //描述
}
````

当错误码为0时，表示成功返回结果，result的值就是JSON格式的结果值；

当错误码不为0时，表示失败返回结果，result可能为空。

desc 描述信息，可能为空。

### 6 二维码规范

详见[Wallet\_File_Specification](https://github.com/ONTIO-Community/ONTO/blob/master/Wallet_File_Specification.md) 中有关二维码部分。

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
| 45010    | ErrTxHashDuplicate            | 交易hash重复                      |
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
| 70000    | UNKNOWN_ONTID                 | 找不到ONT ID                      |




