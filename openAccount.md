# 平安互联网账户开户接入文档
---


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
location.href = url;
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

- `ciphertext`: 参数串AES加密后的16进制密文
- `signature`：参数串RSA私钥加签的16进制签名，注：对生成的密文(ciphertext)进行签名

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
    ```java
    
package com.paic.ibank.utils;
import java.security.KeyFactory;
import java.security.KeyPair;
import java.security.KeyPairGenerator;
import java.security.PrivateKey;
import java.security.PublicKey;
import java.security.SecureRandom;
import java.security.Signature;
import java.security.interfaces.RSAPrivateKey;
import java.security.interfaces.RSAPublicKey;
import java.security.spec.PKCS8EncodedKeySpec;
import java.security.spec.X509EncodedKeySpec;

/**  
 * 加解签算法  
 * @author 
 */   
public class RSASignature {

	

	private static final String KEY_ALGORITHM = "RSA";

	private static final String SIGNATURE_ALGORITHM = "SHA1withRSA";
	
	private static final String SIGNATURE_ALGORITHM_LU = "SHA256withRSA";
	
	private static final int KEYSIZE = 1024;
	
	
	/**  
     * 生成RSA公私密钥对 （byteToHexString 转成十六进制字符串）  
     * @param null
     * @return void 
     * */   
	private static void genKey() throws Exception {
    	
    	KeyPairGenerator keygen = KeyPairGenerator.getInstance(KEY_ALGORITHM);
		SecureRandom random = new SecureRandom();
		keygen.initialize(KEYSIZE, random);
		// 取得密钥对
		KeyPair kp = keygen.generateKeyPair();
		RSAPrivateKey privateKey = (RSAPrivateKey) kp.getPrivate();
		String privateKeyString = byteToHexString(privateKey.getEncoded());
		System.out.println("PRIVATE_KEY:" );
		System.out.println(privateKeyString);

		RSAPublicKey publicKey = (RSAPublicKey) kp.getPublic();
		String publicKeyString = byteToHexString(publicKey.getEncoded());
		System.out.println("PUBLIC_KEY:");
		System.out.println(publicKeyString);
	}
    
	/**  
     * 加签数据  
     * @param String data 待加签数据  
     * @param String privateKey  私钥
     * @return String signedData  加签值（十六进制）
     * */   
	public static String sign(String data, String privateKey) throws Exception{
		try{
			byte[] signData = sign(data, hexStringToByte(privateKey));
			
			return byteToHexString(signData);
			
		}catch (Exception e){
			throw new Exception("signature.sign.error : " + e.getMessage());
		}
	}	
	
	public static String signForLu(String data, String privateKey) throws Exception{
		try{
			byte[] signData = signForLu(data, hexStringToByte(privateKey));
			
			return byteToHexString(signData);
			
		}catch (Exception e){
			throw new Exception("signature.sign.error : " + e.getMessage());
		}
	}	
	
	/**  
     * 加签数据  
     * @param String data 待加签数据  
     * @param byte[] privateKey  私钥
     * @return byte[] signedData
     * */   
	public static byte[] sign(String data, byte[] privateKeyBytes) throws Exception{
		
		try{
			PKCS8EncodedKeySpec priPKCS8 = new PKCS8EncodedKeySpec(privateKeyBytes);
			KeyFactory keyf = KeyFactory.getInstance(KEY_ALGORITHM);
			PrivateKey key = keyf.generatePrivate(priPKCS8);
		
			// 进行签名服务
			Signature signature=Signature.getInstance(SIGNATURE_ALGORITHM);
			signature.initSign(key);
			signature.update(data.getBytes());
			byte[] signData = signature.sign();
		
			// 返回签名结果
			return signData;
		}	catch(Exception e){
			throw new Exception("signature.sign.error : " + e.getMessage());
		}
	}
	
	public static byte[] signForLu(String data, byte[] privateKeyBytes) throws Exception{
		
		try{
			PKCS8EncodedKeySpec priPKCS8 = new PKCS8EncodedKeySpec(privateKeyBytes);
			KeyFactory keyf = KeyFactory.getInstance(KEY_ALGORITHM);
			PrivateKey key = keyf.generatePrivate(priPKCS8);
		
			// 进行签名服务
			Signature signature=Signature.getInstance(SIGNATURE_ALGORITHM_LU);
			signature.initSign(key);
			signature.update(data.getBytes());
			byte[] signData = signature.sign();
		
			// 返回签名结果
			return signData;
		}	catch(Exception e){
			throw new Exception("signature.sign.error : " + e.getMessage());
		}
	}
	
	
	/**
	 * 根据对签名数据使用签名者的公钥来解密后验证是否与原数据相同。从而确认用户签名正确
	 * @param String data 被签名数据
	 * @param String signStr 使用该用户的私钥生成的已签名数据（十六进制）
	 * @param String publicKey 公钥（十六进制）
	 * @return true或false，验证成功为true。
	 * @throws Exception
	 */
	public static boolean verify(String data, String signStr,String publicKey)throws Exception{
		try{
			return verify(data, hexStringToByte(signStr), hexStringToByte(publicKey));
		}
		catch(Exception e){
			throw new Exception("signature.verify.error : " + e.getMessage());
		}
	}
	
	/**
	 * 根据对签名数据使用签名者的公钥来解密后验证是否与原数据相同。从而确认用户签名正确
	 * @param String data 被签名数据
	 * @param byte[] signStr 使用该用户的私钥生成的已签名数据
	 * @param String publicKey 公钥
	 * @return true或false，验证成功为true。
	 * @throws Exception
	 */
	public static boolean verify(String data, byte[] signStrBytes,byte[] publicKeyBytes)throws Exception{
		try{
			KeyFactory keyFactory = KeyFactory.getInstance(KEY_ALGORITHM);
			PublicKey pubKey = keyFactory.generatePublic(new X509EncodedKeySpec(publicKeyBytes));
			
			// 进行验证签名服务
			Signature signature=Signature.getInstance(SIGNATURE_ALGORITHM);
			signature.initVerify(pubKey);
			signature.update(data.getBytes());
			return signature.verify(signStrBytes);
		}
		catch(Exception e){
			throw new Exception("signature.verify.error : " + e.getMessage());
		}
	}
	
	public static boolean verifyForLu(String data, String signStr,String publicKey)throws Exception{
		try{
			return verifyForLu(data, hexStringToByte(signStr), hexStringToByte(publicKey));
		}
		catch(Exception e){
			throw e;
		}
	}
	
	public static boolean verifyForLu(String data, byte[] signStrBytes,byte[] publicKeyBytes)throws Exception{
		try{
			KeyFactory keyFactory = KeyFactory.getInstance(KEY_ALGORITHM);
			PublicKey pubKey = keyFactory.generatePublic(new X509EncodedKeySpec(publicKeyBytes));
			
			// 进行验证签名服务
			Signature signature=Signature.getInstance(SIGNATURE_ALGORITHM_LU);
			signature.initVerify(pubKey);
			signature.update(data.getBytes());
			return signature.verify(signStrBytes);
		}
		catch(Exception e){
			throw e;
		}
	}

	/**  
     * 二进制byte[]转十六进制string  
     */  
    public static String byteToHexString(byte[] bytes){      
         StringBuffer sb = new StringBuffer();      
         for (int i = 0; i < bytes.length; i++) {      
              String strHex=Integer.toHexString(bytes[i]);      
              if(strHex.length() > 3){      
                     sb.append(strHex.substring(6));      
              } else {   
                   if(strHex.length() < 2){   
                      sb.append("0" + strHex);   
                   } else {   
                      sb.append(strHex);      
                   }      
              }   
         }   
        return  sb.toString();      
    }
    
    /**  
     * 十六进制string转二进制byte[]  
     */  
    public static byte[] hexStringToByte(String s) throws Exception{      
        byte[] baKeyword = new byte[s.length() / 2];      
        for (int i = 0; i < baKeyword.length; i++) {      
            try {      
                baKeyword[i] = (byte) (0xff & Integer.parseInt(s.substring(i * 2, i * 2 + 2), 16));      
            } catch (Exception e) {      
                System.out.println("十六进制转byte发生错误！！！");      
                throw(e);     
            }      
        }      
        return baKeyword;      
    } 

    public static void main(String[] args) throws Exception {   
    	try{
    		String public_key = "30820122300d06092a864886f70d01010105000382010f003082010a0282010100977c40553543ec360001df14bb89f0731825ca983e96ece7a28e48aafdbfae81472111162b28e61afca9cba11321ca28abfe5131301b9f01f9dd4d270895784cd198fff92e7638b0c44591e3ac6a57430410d4d14c772654b50cb4220a33c5b2bd3fafb7e7ba8d5c9b253487a1fca8c5eb952fc2c7682cbba406a9b57c6e95b95c4d7b15f82b2e11df433926e67e0be7dfa0235dcf35c160e2f1b3926e81d6d290f641edc14ce3945ba805905085fd6fcbedf69535ac401059b41e0b26fc8642cd09bc49ce9d5efeb4a5382b9d10f2a0f6ad4ce94145dabbd7d7587290c186d4979a0f862e26e5474400b104c2f9ae1065210f48543caa718a0d3cffd7d1130f0203010001";
    	    String aes_key = "a28da58c834822fdb1bad0fd352e793e";
    		String source = "上海陆家嘴国际金融资产交易市场股份有限公司";
    		String ciphertext = "上海陆家嘴国际金融资产交易市场股份有限公司";
    		
    		String sign = "19e4161388d61fb5ad7841cc48a2cb7b4683d5e7b88ac6d7686be5c8f15446a048aec22e875594c0357eacda95407738cb419941b8ba317581cbade418081700d6cbe99f1f913f712e5789dc75c0b276db6244d43551d8974ea1c6e608829f5b5d7562a7214b763ddf94559ff0183dc94ffa01a3ba05cc6469578b2ff76faef8536db7c3466caa63b15f9191f63149920b7b8271881176aebe60168f2098eaa0a742a48adf56d2afb52ecd5d764ae9385a7e9e6509342d4885aea13617c9a53de867b00d49d20d15d94cff9041e86e10d1a88664e5e95bcf460b8fd712f57473a7b9f0eb19e4721068ffd8bddf5e5572018610300dda0d280fb580ab5a3d5518";
    		System.out.println(verifyForLu(ciphertext, sign, public_key));
    		
//    		String decryptData = AESHelper.decrypt(ciphertext, aes_key);
//    		System.out.println(decryptData);
    		
    	}catch (Exception e){
    		e.printStackTrace();
    	}
    	
    }
}

    ```
   
> 5）生成对称密码（AES）的密钥
  
    使用AesTest的initAesKey()方法生成两套密钥（对应测试环境和生产环境），保留密钥并提供给互联网账户服务端开发人员配置


