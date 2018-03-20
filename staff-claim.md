### 公司在职证明可信声明模板

```json
{
	"Id":"2e6e38fc5516832d78307d0ebaed55332b88e163d671a30e18df9f97cfe01fb5",
	"Context":"claim:staff_authentication",
	"Content":{
		"IdNumber": "510806199002122991",
		"Name": "zhangsan",
		"Gender":"male",
		"JobTitle": "SoftwareEngineer",
		"MonthlyWages": 3000.00,
		"Hiredata": "2017-03-20",
	},
	"Metadata":{
		"CreateTime":"2017-04-01T22:01:20Z",
		"Expires":"2018-03-20",
		"Issuer":"did:ont:8uQhQMGzWxR8vw5P3UWH1j",
		"Subject":"did:ont:4XirzuHiNnTrwfjCMtBEJ6",
		"IssuerName":"onchain"
	},
	"Signature":{
		"Format":"pgp",
		"Algorithm":"ECDSAwithSHA256",
		"Value": "rsjaenrxJm8qDmhtOHNBNOCOlvz/GC1c6CMnUb7KOb1jmHbMNGB63VXhtKflwSggyu1cVBK14/0t7qELqIrNmQ=="
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
|    MonthlyWages|   Long|  月工资  |
|    Hiredata|   String|  入职时间，格式：yyyy-MM-dd  |
