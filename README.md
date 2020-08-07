# h5-channel-document
用于说明H5渠道接入的文档

## 身份识别
通过在访问时携带query【channel】申明自身渠道。例如: https://m.jinhui365.cn?channel=jinhui, 这里的channel用作渠道身份识别。

## 其他必要信息
字段 |说明 | 举例
---|---|---
appKey| 种cookie|
appSecert | 种cookie |

## 免登对接
通过在访问时携带query【token】作为用户信息。

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

### 完整请求路由
https://m.jinhui365.cn?channel=jinhui&token=QOFO1M83pTmsElWhQWNSOX8l7Do%2BwKLjn47yw80JNsa1wI0M1IMELxLS1%2Fq4JNy3kXPusmndgzo%3D

## 通信定义
原生APP需在全局定义一个`jinhuiApp`的全局变量，可被H5端通过`window.jinhuiApp`获取到，该变量为json类型，以下为内部定义：
字段|类型|说明
---|---|---
sendApp| function | { id: '', type: '', data: {} } json string
sendWeb| function | { id: '', message: { code: 0, message: "" }, data: {} } json string

H5向原生APP发送消息使用**sendApp**方法，原生APP回复消息通过**sendWeb**方法。固定一条消息通过id进行标识。**type**字段可进行业务扩展。

> PS: message内部code用于判断是否调用成功，成功为0，失败为其他。message用作失败时的错误信息。

### type定义
类型 | type字段 | 参数 | 返回值说明
---|---|---|---
栈长查询 | canGoBack | 无 | message.code == 0 为可以进行页面回退
关闭webview | closeWebview | 无 | 无


### Android 原生webview支持
业务实现涉及图片文件选择，及文件下载，需要原生webview做以下支持：

1. 设置WebChromeClient，实现对openFileChooser的支持，支持选择文件和图片，参考如下代码：
   <pre>
   <code>
    private ValueCallback<Uri> uploadFile;
    private ValueCallback<Uri[]> uploadFiles;
    webView.setWebChromeClient(new JHWebChromeClient() {
            // For Android 3.0+
            public void openFileChooser(ValueCallback<Uri> uploadMsg, String acceptType) {
                Log.i("test", "openFileChooser 1");
                uploadFile = uploadMsg;
                openFileChooseProcess();
            }

            // For Android < 3.0
            public void openFileChooser(ValueCallback<Uri> uploadMsgs) {
                Log.i("test", "openFileChooser 2");
                uploadFile = uploadMsgs;
                openFileChooseProcess();
            }

            // For Android  > 4.1.1
            public void openFileChooser(ValueCallback<Uri> uploadMsg, String acceptType, String capture) {
                Log.i("test", "openFileChooser 3");
                uploadFile = uploadMsg;
                openFileChooseProcess();
            }

            // For Android  >= 5.0
            public boolean onShowFileChooser(WebView webView,
                                             ValueCallback<Uri[]> filePathCallback,
                                             FileChooserParams fileChooserParams) {
                Log.i("test", "openFileChooser 4:" + filePathCallback.toString());
                uploadFiles = filePathCallback;
                openFileChooseProcess();
                return true;
            }
        });
        private void openFileChooseProcess() {
        Intent i = new Intent(Intent.ACTION_GET_CONTENT);
        i.addCategory(Intent.CATEGORY_OPENABLE);
        i.setType("*/*");
        startActivityForResult(Intent.createChooser(i, "上传文件"), 0);
    }

    @Override
    protected void onActivityResult(int requestCode, int resultCode, @Nullable Intent data) {
        super.onActivityResult(requestCode, resultCode, data);
        if (requestCode == 0) {
            if (resultCode == RESULT_OK) {
                if (null != uploadFile) {
                    Uri result = data == null ? null
                            : data.getData();
                    uploadFile.onReceiveValue(result);
                    uploadFile = null;
                }
                if (null != uploadFiles) {
                    Uri result = data == null ? null
                            : data.getData();
                    uploadFiles.onReceiveValue(new Uri[]{result});
                    uploadFiles = null;
                }
            } else if (resultCode == RESULT_CANCELED) {
                if (null != uploadFile) {
                    uploadFile.onReceiveValue(null);
                    uploadFile = null;
                }
            }
        }
    }

   </code>
   </pre>

2. 设置DownloadListener，实现onDownloadStart，进行下载文件的处理，参考如下代码,实现默认打开系统浏览器进行下载：
   <pre>
   <code>
     webView.setDownloadListener(new DownloadListener() {
            @Override
            public void onDownloadStart(String s, String s1, String s2, String s3, long l) {
                try {
                    Uri uri = Uri.parse(s);
                    Intent intent = new Intent(Intent.ACTION_VIEW, uri);
                    startActivity(intent);
                } catch (Exception e) {

                }
            }
        });
   </code>
   </pre>

### iOS原生App配置
#### 在Info.plist中添加相机、相册权限
```
        <key>NSCameraUsageDescription</key>
        <string>需要使用相机拍照进行实名认证、银行卡识别</string>
	<key>NSPhotoLibraryAddUsageDescription</key>
	<string>需要保存身份证、银行卡照片到相册</string>
	<key>NSPhotoLibraryUsageDescription</key>
	<string>需要读取相册中的身份证照片进行实名认证</string>
```
