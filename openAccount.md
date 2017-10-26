### 测试环境地址
https://bank-static-stg.pingan.com.cn/brac-ia/universal_h5/pages/openAccount/entrance.html?channel=XXX&login=1&source=XXX&outerSource=XXX&um=XXX&umType=X&showTitle=X&navigatingButtonName=X&returnUrl=X&entrance=X&cardNo=XXX


### 生产环境地址
https://bank-static.pingan.com.cn/brac-ia/universal_h5/pages/openAccount/entrance.html?channel=XXX&login=1&source=XXX&outerSource=XXX&um=XXX&umType=X&showTitle=X&navigatingButtonName=X&returnUrl=X&entrance=X&cardNo=XXX


口袋内打开页面方式：
```javascript
//方式一
bow.navigator.forward({
    url: url
)};
//方式二
location.href =url;
```

| 参数       | 描述          | 必输 | 备注  |
| ------------- |:-------------:|:-------------:|-------------|
| channel       | 渠道           | Y | C0001 - 营业柜台<br>C0002 - 网上银行<br>C0003 - 口袋银行<br>C0004 - 爱客<br>C0005 - 橙E网<br>C0006 - 智能投顾<br>C0007 - 千人千面<br>C0008 - 橙子银行<br>C0009 - 口袋插件<br>C0010 - 厅堂<br>C0011 - 银保通<br>C0012 - 新口袋银行<br>C0013 - 聚合首页<br>如不是以上渠道，请联系产品经理刘晓楠（EX_WLJR_LIUXIAONAN@pingan.com.cn）分配|
| login       | 是否具有登录态           | N|0 - 登录区外<br>1 - 登录区内<br><br>如不上送，默认登录区外|
| source       | 开户来源           | N|请联系产品经理刘晓楠（EX_WLJR_LIUXIAONAN@pingan.com.cn）确认数值|
| outerSource       | 二级开户来源         |N|请联系产品经理刘晓楠（EX_WLJR_LIUXIAONAN@pingan.com.cn）确认数值|
| um      | 推荐人代码        |N|
|umType     | 推荐人展示形式       |N|1 - 显示且可修改<br>2 - 显示且不可修改<br>3 - 不显<br><br>如不上送，默认显示且可修改<br>请联系产品经理刘晓楠（EX_WLJR_LIUXIAONAN@pingan.com.cn）确认数值|
|showTitle    |是否显示头部       |N|<br>1 - 显示<br>0 - 不显示<br><br>如不上送，默认显示<br>请联系产品经理刘晓楠（EX_WLJR_LIUXIAONAN@pingan.com.cn）确认数值|
|halfUrl  |终止注册返回地址      |N|URL需要做encodeURIComponent()处理<br>URL有白名单控制，在白名单内才允许跳转，非平安域名、bow域名的URL接入前请联系林秋桂（LINQIUGUI177）配置<br>不传则执行returnUrl的逻辑|
|navigatingButtonName  |成功页按钮文案      |N|如不上送系统默认显示“完成”|
|returnUrl  |成功页返回地址    |N|URL需要做encodeURIComponent()处理<br>URL有白名单控制，在白名单内才允许跳转，非平安域名、bow域名的URL接入前请联系林秋桂（LINQIUGUI177）配置<br>非口袋环境需要传此参数<br>不传默认执行bow.navigator.back();|
|entrance  |注册首页   |N|0：绑卡页；1:用户信息页；默认绑卡页|
|cardNo  |银行卡号   |N|登录区内有效，且卡号带出后页面不允许修改|
|bowNavigatorBack  |返回方式   |N|1 – 执行bow.navigator.back({url: url});<br><br>不传默认执行location.href|
|cipherText  |参数数据加密   |N|参考“信任渠道身份证影像审核”|
|signature  |参数签名   |N|参考“信任渠道身份证影像审核”|

### 信任渠道身份证影像审核
> 1）请求参数

`ciphertext`: 参数串AES加密后的16进制密文
`signature`：参数串RSA私钥加签的16进制签名，注：对生成的密文(ciphertext)进行签名

> 2）ciphertext具体参数

| 参数名        | 数据类型           | 是否必输  |参数名称|备注|
|:-------------:|:-------------:|:-----:|:-----:|:-----:|
| timestamp    | String(25) | 必输|时间戳|系统毫秒数
| trustyIdPhotosVerification     | String      |非必输 |信任身份影像审核|值为true或false，默认为false

> 3）密文验证逻辑

  1.	输入参数明文使用AES加密，渠道方与互联网账户使用相同密钥（渠道方提供密钥）； 
  2.	输入参数密文使用RSA私钥加签，互联网账户使用公钥验证签名。（渠道方提供公钥给互联网账户）
  3.	从密文中取时间戳验证当前解密时间与传入的时间戳之间的时间是否超过超时时间（5分钟）要求，超过则不受理。
  
> 4）生成公钥密码（RSA）的公钥和私钥
  
    使用RSASignature的genKey()方法生成两套公钥和私钥（对应测试环境和生产环境），保留私钥，并提供公钥给互联网账户服务端开发人员配置
   
> 5）生成对称密码（AES）的密钥
  
    使用AesTest的initAesKey()方法生成两套密钥（对应测试环境和生产环境），保留密钥并提供给互联网账户服务端开发人员配置

