# h5-channel-document
用于说明H5渠道接入的文档

## 免登对接
### 身份识别
通过在访问时携带query申明自身渠道。例如: https://m.jinhui365.cn?channel=jinhui。这里的channel用作渠道身份识别。

### 必要信息说明
在访问时，携带必要信息，必要信息为3DES加密后的字符串，作为query携带。具体内容如下：

名称|	字段	| 举例
---|---|---
身份证号|	idNo| 330103197001010719
姓名|	name|	日终测试1
手机号|	mobile|	13800000000
资金账号|	bankAccount	|550000507

将上面四个字段的值通过【|】进行分割后拼接成字符串，例如：330103197001010719|日终测试1|13800000000|550000507。

#### 必要信息加密
3DES参数名|	值
---|---
加密模式|	ECB
填充方式|	pkcs7padding
字符串编码	|utf-8
密钥	|通过协商确定，例如：f6c94964aec7607f68082003

#### 加密过程
加密结果为：URLEncoder.encode(Base64.encodeBase64String(encryptMode(value.getBytes(CHARSET))), CHARSET);

1. DESede加密后的结果：encryptMode(value.getBytes(CHARSET))
2. base64编码：Base64.encodeBase64String
3. url编码：URLEncoder.encode



进行3DES加密后为：QOFO1M83pTmsElWhQWNSOX8l7Do%2BwKLjn47yw80JNsa1wI0M1IMELxLS1%2Fq4JNy3kXPusmndgzo%3D

### 结果
完整请求路由：https://m.jinhui365.cn?channel=jinhui&token=QOFO1M83pTmsElWhQWNSOX8l7Do%2BwKLjn47yw80JNsa1wI0M1IMELxLS1%2Fq4JNy3kXPusmndgzo%3D

