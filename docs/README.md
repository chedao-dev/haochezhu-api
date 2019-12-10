# 车到-好车主 API

---

## 接口协议

> 采用标准的HTTPS请求，使用URL来区分调用的方法，接口请求均采用POST方法。



**接口调用请求说明**

- 测试环境域名: `http://test2.51autogo.com:8282`

**公用请求格式**

| 参数名      | 是否必须  | 参数类型    | 参数说明                                  						  |
| ---------- | --------  | ---------- | ----------------------------------------------------------------- |
| appId		 | true      | String     | 渠道appId，由车到分配                     						  |
| sign 	     | true      | String 	  | 见签名算法。请求参数进行签名，详见【签名算法】 						  |
| rd   		 | true      | String     | 的8-16位随机字符串(0-9A-Za-z)，在加密过程中当做IV向量                |
| data       | false     | String     | JSON加密后的字符串，API中的各种参数。详见【data字段的加密方案】          |

**_请求样例：appId=CdACrmOffice&sign=137019e9294ef8972373ba2ed2fd4934&rd=buBKPc8f&data=xxxx_**

**公用响应格式**

| 参数名      | 是否必须  | 参数类型    | 参数说明                                  											  |
| ---------- | --------  | ---------- | ------------------------------------------------------------------------------------  |
| code		 | true      | String     | "0"表示成功，其他均为失败                    											  |
| msg 	     | true      | String 	  | 提示给用户的信息							 											  |
| devMsg   	 | false     | String     | 提示给开发人员的信息，当code为失败时，才会出现			         						  |
| sign       | true      | String     | 返回结果签名，详见【签名算法】								  							  |
| data       | false     | String     | JSON加密的字符串。当code为成功时，出现。详见【data字段的加密方案】，IV向量采用请求中         	  |

**结果样例**
```json
{
  "code": "0",
  "msg": "Ok",
  "data": "1GukLHG1SY-RZN942-YF5aU1lmDoVp2ocrfNSPmO-LtMEm8pzRqJDSRZIbOe6rArwNc6O_nnbr4MSAe-tb48O4EBj_hqH8vLBRxRn0AFeZJDm8yc7Sa6ewnsi68QJlOgw5ck8yEyO0QTm5mTdH2398BpGumqQ6S9ZzjLpJjXFyc",
  "sign": "dc1c4d739903fe8735ca6ea52aee6c19"
}
```
---

## 加密算法

> data字段的加密方案

**加密流程**

      密文=bytesToBase64String(                        //字节数组转成BASE64字符串(需url safe)
                ASE.encrypt(                          //AES解密
                       stringToBytes(原文,"utf-8”),    //把明文字符串转成字节数组，编码utf-8
                       KEY,
                       IV
                )
          )

**解密流程**

	  原文=bytesToString(                          //字节数组转成字符串，编码utf-8
              AES.decrypt(                            //AES加密   
                     base64StringToBytes(密文),        //把密文转成字节数组
                     KEY,
                     IV
              ),
              "utf-8”
          )

**加解密说明**

	原文：字符串，需要加密的明文文本,例如：JSON字符串
	密文：字符串，加密后的字符串
	KEY:   字符串，分配的加密KEY，16(128bit)或32(256bit)个字符
	IV：    字符串，16个字符，随机向量。使用请求协议中的rd字段，如果rd不够长在末尾补0，如果超出取前16位。
	加密协议：AES/CBC/PKCS5Padding
	字符集：加解密过程中的所有字符集，均采用UTF-8

	示例(256bit)：
	原文：{"requestId":"adfadsfasdfasdfasdfasdf"}
	KEY：APPTEST32KEY01234567890123456789
	rd：42511
	密文：
	ez0axyRO0J7Av7HRQX_lAs-P95qNjZLFDnb4tyEQpYQq3TPqh3_LII0hJbC_p-mn


	示例(128bit):
	原文：{"requestId":"adfadsfasdfasdfasdfasdf"}
	KEY：APPTEST16KEY0123
	rd：42511
	密文：
	Qu85EZ_WZDAEn092ldwEsv4gJJuiRllCt7xWLqephtQ-fPwchcobMe7gt5iN3qTW


---


## 签名算法
	将所有参数中的sign和值为空的参数去除后，按剩下的参数按参数名的ASC码升序排序，拼接为QueryString，
	并在末尾添加key=KEY(分配appId时，同时分配的秘钥)。对QueryString计算MD5，再转化为16进制字符串(小写字母)，
	既得到相应的签名。

**示例**

	appId=appidtest
        key=APPTEST16KEY0123
        data=GNFxiXyBOqsxX5cch2WymSPKZcv3gbPDXm5DQJh5BSw-_iuwRGJb3_Q086yjlqre
        rd=42511
        按ASCII码升序排序后的结果为：
            appId=appidtest                      
            data=Qu85EZ_WZDAEn092ldwEsv4gJJuiRllCt7xWLqephtQ-fPwchcobMe7gt5iN3qTW
            key=APPTEST16KEY0123
            rd=42511
        QueryString为：
                appId=appidtest&data=GNFxiXyBOqsxX5cch2WymSPKZcv3gbPDXm5DQJh5BSw-_iuwRGJb3_Q086yjlqre&rd=42511&key=APPTEST16KEY0123          
        对上述字符串的签名为：
                 3c9efc84a95a95f278b23a3fc2f99dd5


## 业务接口data字段定义

### 指定用户发送指定优惠券接口

> 指定用户发送指定优惠券

**接口调用请求说明**

- Path: `/api/coupon/send`
- Method: `POST`
- Query String Parameters:

| 参数名      | 是否必须  | 参数类型    | 参数说明                                  						  |
| ---------- | --------  | ---------- | ----------------------------------------------------------------- |
| templateId | true      | String     | 优惠券规则ID                             						  |
| mobile     | true      | String 	  | 发送优惠券用户手机号                		 						  |


**响应参数**

| 参数名      | 参数类型    | 参数说明                                  						  |
| ---------- | ---------- | ----------------------------------------------------------------- |

---

