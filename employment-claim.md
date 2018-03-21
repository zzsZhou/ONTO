### 公司在职证明可信声明模板

**在职证明是由第三方机构即各个公司签发，所以包含完整性证明MerkleProof**

```json
{
	"Id":"2e6e38fc5516832d78307d0ebaed55332b88e163d671a30e18df9f97cfe01fb5",
	"Context":"claim:employment_authentication",
	"Content":{
		"IdNumber": "510806199002122991",
		"Name": "zhangsan",
		"Gender":"male",
		"JobTitle": "SoftwareEngineer",
		"MonthlySalary": 3000.00,
		"Hiredata": "2017-03-20",
	},
	"Metadata":{
		"CreateTime":"2017-04-01T22:01:20Z",
		"Expires":"2018-03-20",
		"Issuer":"did:ont:TXVemA3GUbXRWz7h4pLpgeuRGX5ewx31qP",
		"Subject":"did:ont:TRdFbK1bqpTRdkPLX1tRz9Sf7Mr41pyBEe",
		"IssuerName":"onchain"
	},
	"Signature":{
		"Format":"pgp",
		"Algorithm":"ECDSAwithSHA256",
		"Value": "rsjaenrxJm8qDmhtOHNBNOCOlvz/GC1c6CMnUb7KOb1jmHbMNGB63VXhtKflwSggyu1cVBK14/0t7qELqIrNmQ=="
	},
	"Proof":{
		"Type":"MerkleProof",
		"TxnHash":"c89e76ee58ae6ad99cfab829d3bf5bd7e5b9af3e5b38713c9d76ef2dcba2c8e0",
		"BlockHeight":10,
		"MerkleRoot":"bfc2ac895685fbb01e22c61462f15f2a6e3544835731a43ae0cba82255a9f904",
		"Nodes":[
			{
				"Direction":"Right",
				"TargetHash":"2fa49b6440104c2de900699d31506845d244cc0c8c36a2fffb019ee7c0c6e2f6"
			},
			{
				"Direction":"Left",
				"TargetHash":"fc4990f9758a310e054d166da842dab1ecd15ad9f8f0122ec71946f20ae964a4"
			}
		]
	}
}
```



可信声明具体内容Content对应字段说明：

| Field     |     Type |   Description   | 
| :--------------: | :--------:| :------: |
|    IdNumber|   String|  身份证号  |
|    Name|   String|  姓名  |
|    Gender|   String| 性别   |
|    JobTitle|   String|  公司职位  |
|    MonthlySalary|   Long|  月工资  |
|    Hiredata|   String|  入职时间，格式：yyyy-MM-dd  |
