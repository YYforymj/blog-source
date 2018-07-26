---
title: Java加密与解密概要01
date: 2017-06-14 15:03:01
categories: 加密与解密
---

> 以下内容为《Java加密与解密的艺术》一书与慕课网“搞定Java加解密”的学习笔记，其中大部分程序来自网络。

<!-- more -->

## Java加密与解密概要01

### 一些基础

- [科克霍夫原则](https://zh.wikipedia.org/wiki/%E6%9F%AF%E5%85%8B%E9%9C%8D%E5%A4%AB%E5%8E%9F%E5%89%87)：数据的安全基于密钥而不是算法的保密。即系统安全取决于密钥，算法公开，密钥保密。这也是现代密码学设计的基本原则；
- 分组密码：将要加密的内容分为固定长度的组，用同一密钥和算法对每一组加密；多用于网络加密；
- 流密码（序列密码）：加密时每次加密一个字节，即逐位加密；

### Base64

#### Base64原理

- Base64就是一种 基于64个可打印字符来表示二进制数据的表示方法；
- 字符选用了"A-Z、a-z、0-9、+、/"64个可打印字符。数值代表字符的索引，这个是标准Base64协议规定的，不能更改

    ![Base64编码表](\images\Java加密与解密概要01\encryptionbase64.png)

- Base64的实现：用4个Base64编码表示3个ASCII码字符，具体如下：
  
    ![Base64编码表](\images\Java加密与解密概要01\encryptionBASE64_example01.png)

- 以上例子是基于内容可以被3整除的情况下，但如果内如无法被3整除，则需要补0；理论上Base64编码是4位1组，如果不够4位则以=补位（其实不补=也不影响解码，但=可以作为结束符号使用）：

    ![Base64编码表](\images\Java加密与解密概要01\encryptionBASE64_example02.png)

- Base64在Java中的实现：Jdk、Commons Codec、Bouncy Castle；
- Base64的应用场景：email、中文或图片传输等；

#### Base64的JDK实现

    package info.yuyublog.encryption;
    
    import com.sun.org.apache.xml.internal.security.utils.Base64;
    
    public class Base64Test {
    
        public static void main(String[] args) throws Exception     
        {     
            String asB64 = Base64.encode("some string".getBytes("utf-8"));
    
            System.out.println(asB64);      
    
            byte[] btarrB64 = Base64.decode(asB64);
    
            String resB64 = new String(btarrB64);
    
            System.out.println(resB64);
        }
    }

输出结果：

    c29tZSBzdHJpbmc=
    some string

### 消息摘要算法

- 特点：
    - 无论输入的消息有多长，计算出来的消息摘要的长度总是固定的。
    - 一般地，只要输入的消息不同，对其进行摘要以后产生的摘要消息也必不相同；但相同的输入必会产生相同的输出。
    - 只能进行正向的信息摘要，而无法从摘要中恢复出任何的消息，甚至根本就找不到任何与原信息相关的信息（不可逆性）。
- 分为MD(Message Digest),SHA(Secure Hash Algorithm),MAC(Message Authentication code)

#### MD算法（Message Digest）

- MD算法包含MD2、MD4、MD5等算法，MD5由MD4发展而来，MD4由MD2发展而来，现在常用的是MD5算法，不过由于密码学的发展，MD5的安全性也已大大下降；
- 有关MD5算法的原理与具体实现可以参考[这篇博客](http://blog.csdn.net/xiaofengcanyuexj/article/details/37698801)。
- MD5一般用于传输文件的一致性验证（例如下载网站上附加的MD5）、数字证书、安全访问认证等（安全访问认证由于用户密码比较简单的话极易利用[彩虹表](http://baike.sogou.com/v8046018.htm?fromTitle=%E5%BD%A9%E8%99%B9%E8%A1%A8)破解，所以安全性较低，使用时最好使用[加盐加密](https://libuchao.com/2013/07/05/password-salt)）。

#### MD5算法的JDK实现

    package info.yuyublog.encryption;
    
    import java.security.MessageDigest;
    
    public class MD5Test {
        public final static String MD5(String s) {
            char hexDigits[]={'0','1','2','3','4','5','6','7','8','9','A','B','C','D','E','F'};       
            try {
                byte[] btInput = s.getBytes();
                // 获得MD5摘要算法的 MessageDigest 对象
                MessageDigest mdInst = MessageDigest.getInstance("MD5");
                // 使用指定的字节更新摘要
                mdInst.update(btInput);
                // 获得密文
                byte[] md = mdInst.digest();
                // 把密文转换成十六进制的字符串形式
                int j = md.length;
                char str[] = new char[j * 2];
                int k = 0;
                for (int i = 0; i < j; i++) {
                    byte byte0 = md[i];
                    str[k++] = hexDigits[byte0 >>> 4 & 0xf];
                    str[k++] = hexDigits[byte0 & 0xf];
                }
                return new String(str);
            } catch (Exception e) {
                e.printStackTrace();
                return null;
            }
        }
        public static void main(String[] args) {
            System.out.println(MD5Test.MD5("加密"));
        }
    }

输出结果:

    56563EDF23B9D717DC63981B8836FC60


#### SHA算法（Secure Hash Algorithm）

- [SHA算法](https://zh.wikipedia.org/wiki/SHA%E5%AE%B6%E6%97%8F)严格意义上来说可以分为SHA-1、SHA-2、SHA-3三代算法，其中后缀的数字代表代数。SHA-1是第一代算法，目前[谷歌已经宣告实现了对其的碰撞](http://www.infoq.com/cn/news/2017/02/google-first-sha1-collision)，安全性相对较低。SHA-2是第二代算法的统称，包含SHA-224、SHA-256、SHA-384、SHA-512四个算法。目前SHA-1与SHA-2这5个算法是比较常用的。
- SHA算法现在一般用于网站根证书(SHA-1即将淘汰但仍占有很大一部分，SHA-2是趋势)、加密数据传输等等。

#### SHA-256算法JDK实现

    package info.yuyublog.encryption;
    
    import java.io.UnsupportedEncodingException;
    import java.security.MessageDigest;
    import java.security.NoSuchAlgorithmException;
    
    public class SHA256Test {
         /**
         *  利用java原生的摘要实现SHA256加密
         * @param str 加密后的报文
         * @return
         */
        public static String getSHA256StrJava(String str){
            MessageDigest messageDigest;
            String encodeStr = "";
            try {
                messageDigest = MessageDigest.getInstance("SHA-256");
                messageDigest.update(str.getBytes("UTF-8"));
                encodeStr = byte2Hex(messageDigest.digest());
            } catch (NoSuchAlgorithmException e) {
                e.printStackTrace();
            } catch (UnsupportedEncodingException e) {
                e.printStackTrace();
            }
            return encodeStr;
        }
    
        /**
         * 将byte转为16进制
         * @param bytes
         * @return
         */
        private static String byte2Hex(byte[] bytes){
            StringBuffer stringBuffer = new StringBuffer();
            String temp = null;
            for (int i=0;i<bytes.length;i++){
                temp = Integer.toHexString(bytes[i] & 0xFF);
                if (temp.length()==1){
                    //1得到一位的进行补0操作
                    stringBuffer.append("0");
                }
                stringBuffer.append(temp);
            }
            return stringBuffer.toString();
        }
        public static void main(String[] args) {
            String src = new String("Hello world!");
            String des = getSHA256StrJava(src);
            System.out.println(des);
        }
    }

输出结果:

    c0535e4be2b79ffd93291305436bf889314e4a3faec05ecffcbb7df31ad9e51a

#### MAC算法（Message Authentication Code）

- 含有密钥散列函数算法，兼容了MD与SHA算分的特性，并在此基础上加入了密钥；

### 对称加密算法

加密密钥与解密密钥相同，加密算法是解密算法的逆运算。常用的有DES、AES与PBE.

#### DES算法（Data Encryption Standard）

- [DES算法](https://zh.wikipedia.org/wiki/%E8%B3%87%E6%96%99%E5%8A%A0%E5%AF%86%E6%A8%99%E6%BA%96)现在是不安全的（已经有实例化的破译机），但它是所有对称加密算法的起源，各种加密算法均由研究DES发展而来。
- DES算法的原理可以在[这里](http://blog.csdn.net/qq_27570955/article/details/52442092)找到。

#### DES算法的JDK实现

    package info.yuyublog.encryption;
    
    import java.security.SecureRandom;
    
    import javax.crypto.Cipher;
    import javax.crypto.SecretKey;
    import javax.crypto.SecretKeyFactory;
    import javax.crypto.spec.DESKeySpec;
    
    public class DESTest {
    
        public static byte[] desCrypto(byte[] datasource, String password) {
            try {
                SecureRandom random = new SecureRandom();
                DESKeySpec desKey = new DESKeySpec(password.getBytes());
                // 创建一个密匙工厂，然后用它把DESKeySpec转换成
                SecretKeyFactory keyFactory = SecretKeyFactory.getInstance("DES");
                SecretKey securekey = keyFactory.generateSecret(desKey);
                // Cipher对象实际完成加密操作
                Cipher cipher = Cipher.getInstance("DES");
                // 用密匙初始化Cipher对象
                cipher.init(Cipher.ENCRYPT_MODE, securekey, random);
                // 获取数据并加密
                return cipher.doFinal(datasource);
            } catch (Throwable e) {
                e.printStackTrace();
            }
            return null;
        }
    
        private static byte[] decrypt(byte[] src, String password) throws Exception {
            // DES算法要求有一个可信任的随机数源
            SecureRandom random = new SecureRandom();
            // 创建一个DESKeySpec对象
            DESKeySpec desKey = new DESKeySpec(password.getBytes());
            // 创建一个密匙工厂
            SecretKeyFactory keyFactory = SecretKeyFactory.getInstance("DES");
            // 将DESKeySpec对象转换成SecretKey对象
            SecretKey securekey = keyFactory.generateSecret(desKey);
            // Cipher对象实际完成解密操作
            Cipher cipher = Cipher.getInstance("DES");
            // 用密匙初始化Cipher对象
            cipher.init(Cipher.DECRYPT_MODE, securekey, random);
            // 开始解密操作
            return cipher.doFinal(src);
        }
    
        private static String byte2Hex(byte[] bytes){
            StringBuffer stringBuffer = new StringBuffer();
            String temp = null;
            for (int i=0;i<bytes.length;i++){
                temp = Integer.toHexString(bytes[i] & 0xFF);
                if (temp.length()==1){
                    //1得到一位的进行补0操作
                    stringBuffer.append("0");
                }
                stringBuffer.append(temp);
            }
            return stringBuffer.toString();
        }
        
        public static void main(String[] args) throws Exception {
            // 待加密内容
            String str = "测试内容";
            // 密码，长度要是8的倍数
            String password = "12345678";
            byte[] result = DESTest.desCrypto(str.getBytes(), password);
            System.out.println("加密后内容为：" + byte2Hex(result));
    
            // 直接将如上内容解密
            try {
                byte[] decryResult = DESTest.decrypt(result, password);
                System.out.println("解密后内容为：" + new String(decryResult));
            } catch (Exception e1) {
                e1.printStackTrace();
            }
        }
    }

输出结果:

    加密后内容为：a5d37c3d8502d503cc7b1eb62cef4b79
    解密后内容为：测试内容


#### DESede

- DESede 即三重DES加密算法，也被称为3DES或者Triple DES。使用三(或两)个不同的密钥对数据块进行三次(或两次)DES加密(加密一次要比进行普通加密的三次要快)。三重DES的强度大约和112bit的密钥强度相当。通过迭代次数的提高了安全性，但同时也造成了加密效率低的问题。
- 到目前为止，还没有人给出攻击三重DES的有效方法。
- 三重DES有四种模型
    - DES-EEE3，使用三个不同密钥，顺序进行三次加密变换。
    - DES-EDE3，使用三个不同密钥，依次进行加密-解密-加密变换。
    - DES-EEE2，其中密钥K1=K3，顺序进行三次加密变换。
    - DES-EDE2， 其中密钥K1=K3，依次进行加密-解密-加密变换。

#### DESede JDK实现

    package info.yuyublog.encryption;
    
    import java.security.Key;
    
    import javax.crypto.Cipher;
    import javax.crypto.KeyGenerator;
    import javax.crypto.SecretKey;
    import javax.crypto.SecretKeyFactory;
    import javax.crypto.spec.DESedeKeySpec;
    
    public class DESedeTest {
        private static final String KEY_ALGORITHM = "DESede";  
          
        private static final String DEFAULT_CIPHER_ALGORITHM = "DESede/ECB/ISO10126Padding";  
          
        /** 
         * 初始化密钥 
         *  
         * @return byte[] 密钥  
         * @throws Exception 
         */  
        public static byte[] initSecretKey() throws Exception{  
            //返回生成指定算法的秘密密钥的 KeyGenerator 对象  
            KeyGenerator kg = KeyGenerator.getInstance(KEY_ALGORITHM);  
            //初始化此密钥生成器，使其具有确定的密钥大小  
            kg.init(168);  
            //生成一个密钥  
            SecretKey  secretKey = kg.generateKey();  
            return secretKey.getEncoded();  
        }  
          
        /** 
         * 转换密钥 
         *  
         * @param key   二进制密钥 
         * @return Key  密钥 
         * @throws Exception 
         */  
        private static Key toKey(byte[] key) throws Exception{  
            //实例化DES密钥规则  
            DESedeKeySpec dks = new DESedeKeySpec(key);  
            //实例化密钥工厂  
            SecretKeyFactory skf = SecretKeyFactory.getInstance(KEY_ALGORITHM);  
            //生成密钥  
            SecretKey  secretKey = skf.generateSecret(dks);  
            return secretKey;  
        }  
          
        /** 
         * 加密 
         *  
         * @param data  待加密数据 
         * @param key   密钥 
         * @return byte[]   加密数据 
         * @throws Exception 
         */  
        public static byte[] encrypt(byte[] data,Key key) throws Exception{  
            return encrypt(data, key,DEFAULT_CIPHER_ALGORITHM);  
        }  
          
        /** 
         * 加密 
         *  
         * @param data  待加密数据 
         * @param key   二进制密钥 
         * @return byte[]   加密数据 
         * @throws Exception 
         */  
        public static byte[] encrypt(byte[] data,byte[] key) throws Exception{  
            return encrypt(data, key,DEFAULT_CIPHER_ALGORITHM);  
        }  


​          
        /** 
         * 加密 
         *  
         * @param data  待加密数据 
         * @param key   二进制密钥 
         * @param cipherAlgorithm   加密算法/工作模式/填充方式 
         * @return byte[]   加密数据 
         * @throws Exception 
         */  
        public static byte[] encrypt(byte[] data,byte[] key,String cipherAlgorithm) throws Exception{  
            //还原密钥  
            Key k = toKey(key);  
            return encrypt(data, k, cipherAlgorithm);  
        }  
          
        /** 
         * 加密 
         *  
         * @param data  待加密数据 
         * @param key   密钥 
         * @param cipherAlgorithm   加密算法/工作模式/填充方式 
         * @return byte[]   加密数据 
         * @throws Exception 
         */  
        public static byte[] encrypt(byte[] data,Key key,String cipherAlgorithm) throws Exception{  
            //实例化  
            Cipher cipher = Cipher.getInstance(cipherAlgorithm);  
            //使用密钥初始化，设置为加密模式  
            cipher.init(Cipher.ENCRYPT_MODE, key);  
            //执行操作  
            return cipher.doFinal(data);  
        }  


​          
​          
        /** 
         * 解密 
         *  
         * @param data  待解密数据 
         * @param key   二进制密钥 
         * @return byte[]   解密数据 
         * @throws Exception 
         */  
        public static byte[] decrypt(byte[] data,byte[] key) throws Exception{  
            return decrypt(data, key,DEFAULT_CIPHER_ALGORITHM);  
        }  
          
        /** 
         * 解密 
         *  
         * @param data  待解密数据 
         * @param key   密钥 
         * @return byte[]   解密数据 
         * @throws Exception 
         */  
        public static byte[] decrypt(byte[] data,Key key) throws Exception{  
            return decrypt(data, key,DEFAULT_CIPHER_ALGORITHM);  
        }  
          
        /** 
         * 解密 
         *  
         * @param data  待解密数据 
         * @param key   二进制密钥 
         * @param cipherAlgorithm   加密算法/工作模式/填充方式 
         * @return byte[]   解密数据 
         * @throws Exception 
         */  
        public static byte[] decrypt(byte[] data,byte[] key,String cipherAlgorithm) throws Exception{  
            //还原密钥  
            Key k = toKey(key);  
            return decrypt(data, k, cipherAlgorithm);  
        }  
      
        /** 
         * 解密 
         *  
         * @param data  待解密数据 
         * @param key   密钥 
         * @param cipherAlgorithm   加密算法/工作模式/填充方式 
         * @return byte[]   解密数据 
         * @throws Exception 
         */  
        public static byte[] decrypt(byte[] data,Key key,String cipherAlgorithm) throws Exception{  
            //实例化  
            Cipher cipher = Cipher.getInstance(cipherAlgorithm);  
            //使用密钥初始化，设置为解密模式  
            cipher.init(Cipher.DECRYPT_MODE, key);  
            //执行操作  
            return cipher.doFinal(data);  
        }  
          
        private static String  showByteArray(byte[] data){  
            if(null == data){  
                return null;  
            }  
            StringBuilder sb = new StringBuilder("{");  
            for(byte b:data){  
                sb.append(b).append(",");  
            }  
            sb.deleteCharAt(sb.length()-1);  
            sb.append("}");  
            return sb.toString();  
        }  
        
        private static String byte2Hex(byte[] bytes){
            StringBuffer stringBuffer = new StringBuffer();
            String temp = null;
            for (int i=0;i<bytes.length;i++){
                temp = Integer.toHexString(bytes[i] & 0xFF);
                if (temp.length()==1){
                    //1得到一位的进行补0操作
                    stringBuffer.append("0");
                }
                stringBuffer.append(temp);
            }
            return stringBuffer.toString();
        }
          
        public static void main(String[] args) throws Exception {  
            byte[] key = initSecretKey();  
            System.out.println("key："+ showByteArray(key));  
              
            Key k = toKey(key);  
               
            String data ="123456789";  
            System.out.println("加密前数据: string:"+data);  
            System.out.println("加密前数据: byte[]:"+showByteArray(data.getBytes()));  
            System.out.println();  
            byte[] encryptData = encrypt(data.getBytes(), k);  
            System.out.println("加密后数据: byte[]:"+showByteArray(encryptData));  
            System.out.println("加密后数据: hexStr:"+byte2Hex(encryptData));  
            System.out.println();  
            byte[] decryptData = decrypt(encryptData, k);  
            System.out.println("解密后数据: byte[]:"+showByteArray(decryptData));  
            System.out.println("解密后数据: string:"+new String(decryptData));  
              
        }  
    }

输出结果:

    key：{50,-33,-50,35,26,-56,104,121,47,103,112,31,-88,69,-88,55,-22,50,-88,-110,-23,-101,82,-122}
    加密前数据: string:123456789
    加密前数据: byte[]:{49,50,51,52,53,54,55,56,57}
    加密后数据: byte[]:{66,71,-7,21,-21,-34,72,55,67,75,-41,-122,117,92,96,-29}
    加密后数据: hexStr:4247f915ebde4837434bd786755c60e3
    解密后数据: byte[]:{49,50,51,52,53,54,55,56,57}
    解密后数据: string:123456789

#### AES算法（Advanced Encryption Standard）

- 又称为Rijndael算法，由于DESede的低效与较慢的运算速度满足不了安全需求，AES应运而生。DES使用56位密钥，比较容易被破解，而AES可以使用128、192、和256位密钥，并且用128位分组加密和解密数据，所以安全性较高，目前已经变成目前对称加密中最流行算法之一；
- AES目前尚未有官方的破译报告；
- 原理参见[这里](http://www.cnblogs.com/luop/p/4334160.html)；
- 现在常应用于Secure SHell等;

#### AES JDK实现

    package info.yuyublog.encryption;
    
    import java.io.UnsupportedEncodingException;
    import java.security.InvalidKeyException;
    import java.security.NoSuchAlgorithmException;
    
    import javax.crypto.BadPaddingException;
    import javax.crypto.Cipher;
    import javax.crypto.IllegalBlockSizeException;
    import javax.crypto.KeyGenerator;
    import javax.crypto.NoSuchPaddingException;
    import javax.crypto.SecretKey;
    import javax.crypto.spec.SecretKeySpec;
    
    public class AESTest {
        /**
         * 加密
         * 
         * @param content 需要加密的内容
         * @param key     加密密码
         * @return
         */
        public static byte[] encrypt(String content, String key) {
            try {
                KeyGenerator kgen = KeyGenerator.getInstance("AES");
                kgen.init(128, new java.security.SecureRandom(key.getBytes()));
                SecretKey secretKey = kgen.generateKey();
                byte[] enCodeFormat = secretKey.getEncoded();
                SecretKeySpec secretKeySpec = new SecretKeySpec(enCodeFormat, "AES");
                Cipher cipher = Cipher.getInstance("AES");// 创建密码器
                byte[] byteContent = content.getBytes("UTF-8");
                cipher.init(Cipher.ENCRYPT_MODE, secretKeySpec);// 初始化
                byte[] result = cipher.doFinal(byteContent);
                return result; // 加密
            } catch (Exception e) {
                e.printStackTrace();
            } 
            return null;
        }
        /**
         - 解密
         - 
         - @param content 待解密内容
         - @param key     解密密钥
         - @return
         */
        public static byte[] decrypt(byte[] content, String key) {
            try {
                KeyGenerator kgen = KeyGenerator.getInstance("AES");
                kgen.init(128, new java.security.SecureRandom(key.getBytes()));
                SecretKey secretKey = kgen.generateKey();
                byte[] enCodeFormat = secretKey.getEncoded();
                SecretKeySpec secretKeySpec = new SecretKeySpec(enCodeFormat, "AES");
                Cipher cipher = Cipher.getInstance("AES");// 创建密码器
                cipher.init(Cipher.DECRYPT_MODE, secretKeySpec);// 初始化
                byte[] result = cipher.doFinal(content);
                return result; // 加密
            } catch (Exception e) {
                e.printStackTrace();
            } 
            return null;
        }
    
        /**
         - 字符串加密
         - 
         - @param content  要加密的字符串
         - @param key      加密的AES Key
         - @return
         */
        public static String encryptString(String content, String key) {
            return parseByte2HexStr(encrypt(content, key));
        }
    
        /**
         - 字符串解密
         - 
         - @param content   要解密的字符串
         - @param key       解密的AES Key
         - @return
         */
        public static String decryptString(String content, String key) {
            byte[] decryptFrom = parseHexStr2Byte(content);
            byte[] decryptResult = decrypt(decryptFrom, key);
            return new String(decryptResult);
        }
    
        /**
         - 将16进制转换为二进制
         - 
         - @param hexStr
         - @return
         */
        public static byte[] parseHexStr2Byte(String hexStr) {
            if (hexStr.length() < 1)
                return null;
            byte[] result = new byte[hexStr.length() / 2];
            for (int i = 0; i < hexStr.length() / 2; i++) {
                int high = Integer.parseInt(hexStr.substring(i * 2, i * 2 + 1), 16);
                int low = Integer.parseInt(hexStr.substring(i * 2 + 1, i * 2 + 2), 16);
                result[i] = (byte) (high * 16 + low);
            }
            return result;
        }
    
        /**
         - 将二进制转换成16进制
         - 
         - @param buf
         - @return
         */
        public static String parseByte2HexStr(byte buf[]) {
            StringBuffer sb = new StringBuffer();
            for (int i = 0; i < buf.length; i++) {
                String hex = Integer.toHexString(buf[i] & 0xFF);
                if (hex.length() == 1) {
                    hex = '0' + hex;
                }
                sb.append(hex.toUpperCase());
            }
            return sb.toString();
        }
    
        public static void main(String aregs[]) {
            String content = "协议加密Test&123";
            String key = "25d55ad283aa400af464c76d713c07ad";
            try {
                // 加密
                System.out.println("加密前：" + content);
                String encrypt = AESTest.encryptString(content, key);
                System.out.println("加密后：" + encrypt);
                // 解密
                String decrypt = AESTest.decryptString(encrypt, key);
                System.out.println("解密后：" + decrypt);
            } catch (Exception e) {
                e.printStackTrace();
            }
    
        }
    }

输出结果:

    加密前：协议加密Test&123
    加密后：7D07155438880C51E50433C17D9AFF8DCB2A9596AF46B2ABA33C18918053E3E0
    解密后：协议加密Test&123

#### PBE（Password Based Encryption）

- PBE算法是一种基于口令的加密算法，其特点在于口令是由用户自己掌握的，采用随机数杂凑多重加密等方法保证数据的安全性。
- PBE算法没有密钥的概念，密钥在其它对称加密算法中是经过算法计算得出来的，PBE算法则是使用口令替代了密钥。PBE算法并没有真正构建新的加密/解密算法，而是对我们已经知道的对称加密算法（如DES算法）做了包装。使用PBE算法对数据做加解密操作的时候，其实是使用了DES或者是AES等其它对称加密算法做了相应的操作。
- 由于仅仅使用口令十分不安全，所以PBE中引入了盐来提升安全性；

#### PBE JDK实现

    package info.yuyublog.encryption;
    
    import java.util.Arrays;
    
    import javax.crypto.Cipher;
    import javax.crypto.SecretKey;
    import javax.crypto.SecretKeyFactory;
    import javax.crypto.spec.PBEKeySpec;
    import javax.crypto.spec.PBEParameterSpec;
    
    public class PBETest {
        static final String KEY_ALGORITHM = "PBEWithMD5AndDES";
        static byte[] salt = "hello123".getBytes(); // 盐：Salt must be 8 bytes long
        static int iterationCount = 888; // 循环次数
        static Cipher cipher;
    
        public static void main(String[] args) throws Exception {
            byte[] encrypt = encrypt("PBETest");
            System.out.println("PBE加密后：" + Arrays.toString(encrypt));
    
            System.out.println("PBE解密后：" + decrypt(encrypt));
        }
    
        /**
         * 使用PBE 算法 加密
         * 
         * @return 加密后的字符数组
         * @throws Exception
         */
        static byte[] encrypt(String str) throws Exception {
            cipher = Cipher.getInstance(KEY_ALGORITHM);
    
            // 使用SecretKeyFactory 生成key
            SecretKeyFactory factory = SecretKeyFactory.getInstance(KEY_ALGORITHM);
            PBEKeySpec keySpec = new PBEKeySpec("password".toCharArray());
            SecretKey key = factory.generateSecret(keySpec);
            System.out.println("key:" + Arrays.toString(key.getEncoded()));
    
            cipher.init(Cipher.ENCRYPT_MODE, key, new PBEParameterSpec(salt, iterationCount));// 使用加密模式初始化                             
            return cipher.doFinal(str.getBytes()); // 按单部分操作加密或解密数据，或者结束一个多部分操作。
        }
    
        /**
         * 
         * @param encrypt
         * @return
         * @throws Exception
         */
        static String decrypt(byte[] encrypt) throws Exception {
            cipher = Cipher.getInstance(KEY_ALGORITHM);
    
            // 使用SecretKeyFactory 生成key
            SecretKeyFactory factory = SecretKeyFactory.getInstance(KEY_ALGORITHM);
            PBEKeySpec keySpec = new PBEKeySpec("password".toCharArray());
            SecretKey key = factory.generateSecret(keySpec);
            System.out.println("key:" + Arrays.toString(key.getEncoded()));
    
            cipher.init(Cipher.DECRYPT_MODE, key, new PBEParameterSpec(salt, iterationCount));// 使用加密模式初始化
            byte[] result = cipher.doFinal(encrypt); // 按单部分操作加密或解密数据，或者结束一个多部分操作。
    
            return new String(result);
        }
    }

输出结果：

    key:[112, 97, 115, 115, 119, 111, 114, 100]
    PBE加密后：[-4, 57, 100, 25, 91, 55, 112, -76]
    key:[112, 97, 115, 115, 119, 111, 114, 100]
    PBE解密后：PBETest
