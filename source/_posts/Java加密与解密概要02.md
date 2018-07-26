---
title: Java加密与解密概要02
date: 2017-06-14 21:03:01
categories: 加密与解密
---

> 以下内容为《Java加密与解密的艺术》一书与慕课网“搞定Java加解密”的学习笔记，其中大部分程序来自网络。

<!-- more -->

## Java加密与解密概要02

### 非对称加密算法

- 不同于对称加密算法，非对称加密算法的加密与解密采用不同的密钥，所以称之为非对称加密算法；
- 同对称加密算法相比，非对称加密算法会更加安全，但同时带来的问题是加密与解密速度要比对称加密算法慢许多；
- 现如今许多B2B或B2C应用均使用了非对称加密算法作为数据加密的核心算法；

#### DH（Diffie-Hellman）算法

- DH算法（密钥交换算法）是非对称加密算法的基础，是通信双方通过信息交换协商密钥的算法，DH仅能用于密钥分配，不能用于加密或解密消息；
- 有关DH算法的原理可以看[这里](https://www.zhihu.com/question/29383090)

#### DH算法的JDK实现

由于美国的出口限制，sun通过权限文件对加密算法实现的jar包做了限制，直接运行程序会报错密钥长度不对，解决办法在[这里](http://www.360doc.com/content/15/0407/09/9552892_461201223.shtml).

DHCoder.java

    package info.yuyublog.encryption;
    
    import java.security.Key;
    import java.security.KeyFactory;
    import java.security.KeyPair;
    import java.security.KeyPairGenerator;
    import java.security.PublicKey;
    import java.security.PrivateKey;
    import java.security.spec.PKCS8EncodedKeySpec;
    import java.security.spec.X509EncodedKeySpec;
    import java.util.HashMap;
    import java.util.Map;
    
    import javax.crypto.Cipher;
    import javax.crypto.KeyAgreement;
    import javax.crypto.SecretKey;
    import javax.crypto.interfaces.DHPrivateKey;
    import javax.crypto.interfaces.DHPublicKey;
    import javax.crypto.spec.DHParameterSpec;
    import javax.crypto.spec.SecretKeySpec;
    
    public abstract class DHCoder {
    
        /**
         * 非对称加密密钥算法
         */
        public static final String KEY_ALGORITHM = "DH";
    
        /**
         * 本地密钥算法，即对称加密密钥算法，可选DES、DESede和AES算法
         */
        public static final String SECRET_KEY_ALGORITHM = "AES";
    
        /**
         * 默认密钥长度
         * 
         * DH算法默认密钥长度为1024 密钥长度必须是64的倍数，其范围在512到1024位之间。
         */
        private static final int KEY_SIZE = 512;
    
        /**
         * 公钥
         */
        private static final String PUBLIC_KEY = "DHPublicKey";
    
        /**
         * 私钥
         */
        private static final String PRIVATE_KEY = "DHPrivateKey";
    
        /**
         * 初始化甲方密钥
         * 
         * @return Map 甲方密钥Map
         * @throws Exception
         */
        public static Map<String, Object> initKey() throws Exception {
    
            // 实例化密钥对生成器
            KeyPairGenerator keyPairGenerator = KeyPairGenerator
                    .getInstance(KEY_ALGORITHM);
    
            // 初始化密钥对生成器
            keyPairGenerator.initialize(KEY_SIZE);
    
            // 生成密钥对
            KeyPair keyPair = keyPairGenerator.generateKeyPair();
    
            // 甲方公钥
            DHPublicKey publicKey = (DHPublicKey) keyPair.getPublic();
    
            // 甲方私钥
            DHPrivateKey privateKey = (DHPrivateKey) keyPair.getPrivate();
    
            // 将密钥对存储在Map中
            Map<String, Object> keyMap = new HashMap<String, Object>(2);
    
            keyMap.put(PUBLIC_KEY, publicKey);
            keyMap.put(PRIVATE_KEY, privateKey);
    
            return keyMap;
        }
    
        /**
         * 初始化乙方密钥
         * 
         * @param key
         *            甲方公钥
         * @return Map 乙方密钥Map
         * @throws Exception
         */
        public static Map<String, Object> initKey(byte[] key) throws Exception {
    
            // 解析甲方公钥
            // 转换公钥材料
            X509EncodedKeySpec x509KeySpec = new X509EncodedKeySpec(key);
    
            // 实例化密钥工厂
            KeyFactory keyFactory = KeyFactory.getInstance(KEY_ALGORITHM);
    
            // 产生公钥
            PublicKey pubKey = keyFactory.generatePublic(x509KeySpec);
    
            // 由甲方公钥构建乙方密钥
            DHParameterSpec dhParamSpec = ((DHPublicKey) pubKey).getParams();
    
            // 实例化密钥对生成器
            KeyPairGenerator keyPairGenerator = KeyPairGenerator
                    .getInstance(keyFactory.getAlgorithm());
    
            // 初始化密钥对生成器
            keyPairGenerator.initialize(dhParamSpec);
    
            // 产生密钥对
            KeyPair keyPair = keyPairGenerator.genKeyPair();
    
            // 乙方公钥
            DHPublicKey publicKey = (DHPublicKey) keyPair.getPublic();
    
            // 乙方私钥
            DHPrivateKey privateKey = (DHPrivateKey) keyPair.getPrivate();
    
            // 将密钥对存储在Map中
            Map<String, Object> keyMap = new HashMap<String, Object>(2);
    
            keyMap.put(PUBLIC_KEY, publicKey);
            keyMap.put(PRIVATE_KEY, privateKey);
    
            return keyMap;
        }
    
        /**
         * 加密
         * 
         * @param data
         *            待加密数据
         * @param key
         *            密钥
         * @return byte[] 加密数据
         * @throws Exception
         */
        public static byte[] encrypt(byte[] data, byte[] key) throws Exception {
    
            // 生成本地密钥
            SecretKey secretKey = new SecretKeySpec(key, SECRET_KEY_ALGORITHM);
    
            // 数据加密
            Cipher cipher = Cipher.getInstance(secretKey.getAlgorithm());
    
            cipher.init(Cipher.ENCRYPT_MODE, secretKey);
    
            return cipher.doFinal(data);
        }
    
        /**
         * 解密<br>
         * 
         * @param data
         *            待解密数据
         * @param key
         *            密钥
         * @return byte[] 解密数据
         * @throws Exception
         */
        public static byte[] decrypt(byte[] data, byte[] key) throws Exception {
    
            // 生成本地密钥
            SecretKey secretKey = new SecretKeySpec(key, SECRET_KEY_ALGORITHM);
    
            // 数据解密
            Cipher cipher = Cipher.getInstance(secretKey.getAlgorithm());
    
            cipher.init(Cipher.DECRYPT_MODE, secretKey);
    
            return cipher.doFinal(data);
        }
    
        /**
         * 构建密钥
         * 
         * @param publicKey
         *            公钥
         * @param privateKey
         *            私钥
         * @return byte[] 本地密钥
         * @throws Exception
         */
        public static byte[] getSecretKey(byte[] publicKey, byte[] privateKey)
                throws Exception {
    
            // 实例化密钥工厂
            KeyFactory keyFactory = KeyFactory.getInstance(KEY_ALGORITHM);
    
            // 初始化公钥
            // 密钥材料转换
            X509EncodedKeySpec x509KeySpec = new X509EncodedKeySpec(publicKey);
    
            // 产生公钥
            PublicKey pubKey = keyFactory.generatePublic(x509KeySpec);
    
            // 初始化私钥
            // 密钥材料转换
            PKCS8EncodedKeySpec pkcs8KeySpec = new PKCS8EncodedKeySpec(privateKey);
    
            // 产生私钥
            PrivateKey priKey = keyFactory.generatePrivate(pkcs8KeySpec);
    
            // 实例化
            KeyAgreement keyAgree = KeyAgreement.getInstance(keyFactory
                    .getAlgorithm());
    
            // 初始化
            keyAgree.init(priKey);
    
            keyAgree.doPhase(pubKey, true);
    
            // 生成本地密钥
            SecretKey secretKey = keyAgree.generateSecret(SECRET_KEY_ALGORITHM);
    
            return secretKey.getEncoded();
        }
    
        /**
         * 取得私钥
         * 
         * @param keyMap
         *            密钥Map
         * @return byte[] 私钥
         * @throws Exception
         */
        public static byte[] getPrivateKey(Map<String, Object> keyMap)
                throws Exception {
    
            Key key = (Key) keyMap.get(PRIVATE_KEY);
    
            return key.getEncoded();
        }
    
        /**
         * 取得公钥
         * 
         * @param keyMap
         *            密钥Map
         * @return byte[] 公钥
         * @throws Exception
         */
        public static byte[] getPublicKey(Map<String, Object> keyMap)
                throws Exception {
    
            Key key = (Key) keyMap.get(PUBLIC_KEY);
    
            return key.getEncoded();
        }
    }

DHCoderTest.java

    package info.yuyublog.encryption;
    
    import static org.junit.Assert.*;
    
    import java.util.Map;
    
    import org.apache.commons.codec.binary.Base64;
    import org.junit.Before;
    import org.junit.Test;
    
    public class DHCoderTest {
    
        /**
         * 甲方公钥
         */
        private byte[] publicKey1;
        /**
         * 甲方私钥
         */
        private byte[] privateKey1;
    
        /**
         * 甲方本地密钥
         */
        private byte[] key1;
    
        /**
         * 乙方公钥
         */
        private byte[] publicKey2;
    
        /**
         * 乙方私钥
         */
        private byte[] privateKey2;
    
        /**
         * 乙方本地密钥
         */
        private byte[] key2;
    
        /**
         * 初始化密钥
         * 
         * @throws Exception
         */
        @Before
        public final void initKey() throws Exception {
    
            // 生成甲方密钥对儿
            Map<String, Object> keyMap1 = DHCoder.initKey();
    
            publicKey1 = DHCoder.getPublicKey(keyMap1);
            privateKey1 = DHCoder.getPrivateKey(keyMap1);
    
            System.err.println("甲方公钥:\n" + Base64.encodeBase64String(publicKey1));
            System.err.println("甲方私钥:\n" + Base64.encodeBase64String(privateKey1));
    
            // 由甲方公钥产生本地密钥对儿
            Map<String, Object> keyMap2 = DHCoder.initKey(publicKey1);
    
            publicKey2 = DHCoder.getPublicKey(keyMap2);
            privateKey2 = DHCoder.getPrivateKey(keyMap2);
    
            System.err.println("乙方公钥:\n" + Base64.encodeBase64String(publicKey2));
            System.err.println("乙方私钥:\n" + Base64.encodeBase64String(privateKey2));
    
            key1 = DHCoder.getSecretKey(publicKey2, privateKey1);
            System.err.println("甲方本地密钥:\n" + Base64.encodeBase64String(key1));
    
            key2 = DHCoder.getSecretKey(publicKey1, privateKey2);
            System.err.println("乙方本地密钥:\n" + Base64.encodeBase64String(key2));
    
            assertArrayEquals(key1, key2);
        }
    
        /**
         * 校验
         * 
         * @throws Exception
         */
        @Test
        public final void test() throws Exception {
    
            System.err.println("\n=====甲方向乙方发送加密数据=====");
            String input1 = "密码交换算法 ";
            System.err.println("原文: " + input1);
            
            System.err.println("---使用甲方本地密钥对数据加密---");
    
            // 使用甲方本地密钥对数据加密
            byte[] code1 = DHCoder.encrypt(input1.getBytes(), key1);
    
            System.err.println("加密: " + Base64.encodeBase64String(code1));
    
            System.err.println("---使用乙方本地密钥对数据解密---");
    
            // 使用乙方本地密钥对数据解密
            byte[] decode1 = DHCoder.decrypt(code1, key2);
            String output1 = (new String(decode1));
    
            System.err.println("解密: " + output1);
    
            assertEquals(input1, output1);
    
            System.err.println("\n=====乙方向甲方发送加密数据=====");
            String input2 = "DH";
            System.err.println("原文: " + input2);
    
            System.err.println("---使用乙方本地密钥对数据加密---");
    
            // 使用乙方本地密钥对数据加密
            byte[] code2 = DHCoder.encrypt(input2.getBytes(), key2);
    
            System.err.println("加密: " + Base64.encodeBase64String(code2));
    
            System.err.println("---使用甲方本地密钥对数据解密---");
    
            // 使用甲方本地密钥对数据解密
            byte[] decode2 = DHCoder.decrypt(code2, key1);
            String output2 = (new String(decode2));
    
            System.err.println("解密: " + output2);
    
            // 校验
            assertEquals(input2, output2);
        }
    }

输出结果

    甲方公钥:
    MIHfMIGXBgkqhkiG9w0BAwEwgYkCQQD8poLOjhLKuibvzPcRDlJtsHiwXt7LzR60ogjzrhYXrgHzW5Gkfm32NBPF4S7QiZvNEyrNUNmRUb3EPuc3WS4XAkBnhHGyepz0TukaScUUfbGpqvJE8FpDTWSGkx0tFCcbnjUDC3H9c9oXkGmzLik1Yw4cIGI1TQ2iCmxBblC+eUykAgIBgANDAAJAIr2aRBlhBro1Oi2mAcTSkUD45z8Xvrf+EXZNtbe+FP9u6NTu//daekR5dUTjI25IsGl1iq9I7947yaTo37ZM/w==
    甲方私钥:
    MIHSAgEAMIGXBgkqhkiG9w0BAwEwgYkCQQD8poLOjhLKuibvzPcRDlJtsHiwXt7LzR60ogjzrhYXrgHzW5Gkfm32NBPF4S7QiZvNEyrNUNmRUb3EPuc3WS4XAkBnhHGyepz0TukaScUUfbGpqvJE8FpDTWSGkx0tFCcbnjUDC3H9c9oXkGmzLik1Yw4cIGI1TQ2iCmxBblC+eUykAgIBgAQzAjEA47mBfW/HEOjCaI3X7ohvA1FxMluaNYOy/gUcsD4ENS0+fg6DWKBD3rjr4vcSIe5v
    乙方公钥:
    MIHgMIGXBgkqhkiG9w0BAwEwgYkCQQD8poLOjhLKuibvzPcRDlJtsHiwXt7LzR60ogjzrhYXrgHzW5Gkfm32NBPF4S7QiZvNEyrNUNmRUb3EPuc3WS4XAkBnhHGyepz0TukaScUUfbGpqvJE8FpDTWSGkx0tFCcbnjUDC3H9c9oXkGmzLik1Yw4cIGI1TQ2iCmxBblC+eUykAgIBgANEAAJBAIBBvLJI0jgxWHCZXBGsgA8XX4et0vUTQ8c5vg8RnFK7h+14ycorG4i0vlMI7p/xl/Oi/KTpqIUJAcZq0tDWZ1M=
    乙方私钥:
    MIHRAgEAMIGXBgkqhkiG9w0BAwEwgYkCQQD8poLOjhLKuibvzPcRDlJtsHiwXt7LzR60ogjzrhYXrgHzW5Gkfm32NBPF4S7QiZvNEyrNUNmRUb3EPuc3WS4XAkBnhHGyepz0TukaScUUfbGpqvJE8FpDTWSGkx0tFCcbnjUDC3H9c9oXkGmzLik1Yw4cIGI1TQ2iCmxBblC+eUykAgIBgAQyAjBTgfEHtiLUphn3rCJxRbjYWwAmW51j2BD7bZelhsMGBtqCahHwdEru294DhWs578Q=
    甲方本地密钥:
    qYAbEU1UV4VGPt6/sBXXrBqMTo9UqVrNtkTPsqZSnf4=
    乙方本地密钥:
    qYAbEU1UV4VGPt6/sBXXrBqMTo9UqVrNtkTPsqZSnf4=
    
    =====甲方向乙方发送加密数据=====
    原文: 密码交换算法 
    ---使用甲方本地密钥对数据加密---
    加密: dDjgT8wuUtCeNFPq25hADIPk/oEfb7ZUKKLFKqm76wk=
    ---使用乙方本地密钥对数据解密---
    解密: 密码交换算法 
    
    =====乙方向甲方发送加密数据=====
    原文: DH
    ---使用乙方本地密钥对数据加密---
    加密: BFNJwm5w0wgHgSKt5RQF4w==
    ---使用甲方本地密钥对数据解密---
    解密: DH


#### RSA算法

- RSA算法核心基于大数因子分解难题，目前是唯一被广泛接受并实现的通用公开加密算法，目前已经成为非对称加密算法国际标准；
- 有关RSA算法的原理，可以在[这里](http://blog.csdn.net/xingqingly/article/details/72626140)找到，另外，[阮一峰的博客](http://www.ruanyifeng.com/blog/2013/06/rsa_algorithm_part_one.html)中也作出了详细解释;

#### RSA算法的JDK实现

RSACoder.java

    package info.yuyublog.encryption;
    
    import java.security.Key;
    import java.security.KeyFactory;
    import java.security.KeyPair;
    import java.security.KeyPairGenerator;
    import java.security.PrivateKey;
    import java.security.PublicKey;
    import java.security.interfaces.RSAPrivateKey;
    import java.security.interfaces.RSAPublicKey;
    import java.security.spec.PKCS8EncodedKeySpec;
    import java.security.spec.X509EncodedKeySpec;
    import java.util.HashMap;
    import java.util.Map;
    
    import javax.crypto.Cipher;
    
    public abstract class RSACoder {
    
        /**
         * 非对称加密密钥算法
         */
        public static final String KEY_ALGORITHM = "RSA";
    
        /**
         * 公钥
         */
        private static final String PUBLIC_KEY = "RSAPublicKey";
    
        /**
         * 私钥
         */
        private static final String PRIVATE_KEY = "RSAPrivateKey";
    
        /**
         * RSA密钥长度 
         * 默认1024位，
         * 密钥长度必须是64的倍数， 
         * 范围在512至65536位之间。
         */
        private static final int KEY_SIZE = 512;
    
        /**
         * 私钥解密
         * 
         * @param data
         *            待解密数据
         * @param key
         *            私钥
         * @return byte[] 解密数据
         * @throws Exception
         */
        public static byte[] decryptByPrivateKey(byte[] data, byte[] key)
                throws Exception {
    
            // 取得私钥
            PKCS8EncodedKeySpec pkcs8KeySpec = new PKCS8EncodedKeySpec(key);
    
            KeyFactory keyFactory = KeyFactory.getInstance(KEY_ALGORITHM);
    
            // 生成私钥
            PrivateKey privateKey = keyFactory.generatePrivate(pkcs8KeySpec);
    
            // 对数据解密
            Cipher cipher = Cipher.getInstance(keyFactory.getAlgorithm());
    
            cipher.init(Cipher.DECRYPT_MODE, privateKey);
    
            return cipher.doFinal(data);
        }
    
        /**
         * 公钥解密
         * 
         * @param data
         *            待解密数据
         * @param key
         *            公钥
         * @return byte[] 解密数据
         * @throws Exception
         */
        public static byte[] decryptByPublicKey(byte[] data, byte[] key)
                throws Exception {
    
            // 取得公钥
            X509EncodedKeySpec x509KeySpec = new X509EncodedKeySpec(key);
    
            KeyFactory keyFactory = KeyFactory.getInstance(KEY_ALGORITHM);
    
            // 生成公钥
            PublicKey publicKey = keyFactory.generatePublic(x509KeySpec);
    
            // 对数据解密
            Cipher cipher = Cipher.getInstance(keyFactory.getAlgorithm());
    
            cipher.init(Cipher.DECRYPT_MODE, publicKey);
    
            return cipher.doFinal(data);
        }
    
        /**
         * 公钥加密
         * 
         * @param data
         *            待加密数据
         * @param key
         *            公钥
         * @return byte[] 加密数据
         * @throws Exception
         */
        public static byte[] encryptByPublicKey(byte[] data, byte[] key)
                throws Exception {
    
            // 取得公钥
            X509EncodedKeySpec x509KeySpec = new X509EncodedKeySpec(key);
    
            KeyFactory keyFactory = KeyFactory.getInstance(KEY_ALGORITHM);
    
            PublicKey publicKey = keyFactory.generatePublic(x509KeySpec);
    
            // 对数据加密
            Cipher cipher = Cipher.getInstance(keyFactory.getAlgorithm());
    
            cipher.init(Cipher.ENCRYPT_MODE, publicKey);
    
            return cipher.doFinal(data);
        }
    
        /**
         * 私钥加密
         * 
         * @param data
         *            待加密数据
         * @param key
         *            私钥
         * @return byte[] 加密数据
         * @throws Exception
         */
        public static byte[] encryptByPrivateKey(byte[] data, byte[] key)
                throws Exception {
    
            // 取得私钥
            PKCS8EncodedKeySpec pkcs8KeySpec = new PKCS8EncodedKeySpec(key);
    
            KeyFactory keyFactory = KeyFactory.getInstance(KEY_ALGORITHM);
    
            // 生成私钥
            PrivateKey privateKey = keyFactory.generatePrivate(pkcs8KeySpec);
    
            // 对数据加密
            Cipher cipher = Cipher.getInstance(keyFactory.getAlgorithm());
    
            cipher.init(Cipher.ENCRYPT_MODE, privateKey);
    
            return cipher.doFinal(data);
        }
    
        /**
         * 取得私钥
         * 
         * @param keyMap
         *            密钥Map
         * @return byte[] 私钥
         * @throws Exception
         */
        public static byte[] getPrivateKey(Map<String, Object> keyMap)
                throws Exception {
    
            Key key = (Key) keyMap.get(PRIVATE_KEY);
    
            return key.getEncoded();
        }
    
        /**
         * 取得公钥
         * 
         * @param keyMap
         *            密钥Map
         * @return byte[] 公钥
         * @throws Exception
         */
        public static byte[] getPublicKey(Map<String, Object> keyMap)
                throws Exception {
    
            Key key = (Key) keyMap.get(PUBLIC_KEY);
    
            return key.getEncoded();
        }
    
        /**
         * 初始化密钥
         * 
         * @return Map 密钥Map
         * @throws Exception
         */
        public static Map<String, Object> initKey() throws Exception {
    
            // 实例化密钥对生成器
            KeyPairGenerator keyPairGen = KeyPairGenerator
                    .getInstance(KEY_ALGORITHM);
    
            // 初始化密钥对生成器
            keyPairGen.initialize(KEY_SIZE);
    
            // 生成密钥对
            KeyPair keyPair = keyPairGen.generateKeyPair();
    
            // 公钥
            RSAPublicKey publicKey = (RSAPublicKey) keyPair.getPublic();
    
            // 私钥
            RSAPrivateKey privateKey = (RSAPrivateKey) keyPair.getPrivate();
    
            // 封装密钥
            Map<String, Object> keyMap = new HashMap<String, Object>(2);
    
            keyMap.put(PUBLIC_KEY, publicKey);
            keyMap.put(PRIVATE_KEY, privateKey);
    
            return keyMap;
        }
    }

DHCoderTest.java

    package info.yuyublog.encryption;
    
    import static org.junit.Assert.*;
    
    import java.util.Map;
    
    import org.apache.commons.codec.binary.Base64;
    import org.junit.Before;
    import org.junit.Test;
    
    public class DHCoderTest {
    
        /**
         * 甲方公钥
         */
        private byte[] publicKey1;
        /**
         * 甲方私钥
         */
        private byte[] privateKey1;
    
        /**
         * 甲方本地密钥
         */
        private byte[] key1;
    
        /**
         * 乙方公钥
         */
        private byte[] publicKey2;
    
        /**
         * 乙方私钥
         */
        private byte[] privateKey2;
    
        /**
         * 乙方本地密钥
         */
        private byte[] key2;
    
        /**
         * 初始化密钥
         * 
         * @throws Exception
         */
        @Before
        public final void initKey() throws Exception {
    
            // 生成甲方密钥对儿
            Map<String, Object> keyMap1 = DHCoder.initKey();
    
            publicKey1 = DHCoder.getPublicKey(keyMap1);
            privateKey1 = DHCoder.getPrivateKey(keyMap1);
    
            System.err.println("甲方公钥:\n" + Base64.encodeBase64String(publicKey1));
            System.err.println("甲方私钥:\n" + Base64.encodeBase64String(privateKey1));
    
            // 由甲方公钥产生本地密钥对儿
            Map<String, Object> keyMap2 = DHCoder.initKey(publicKey1);
    
            publicKey2 = DHCoder.getPublicKey(keyMap2);
            privateKey2 = DHCoder.getPrivateKey(keyMap2);
    
            System.err.println("乙方公钥:\n" + Base64.encodeBase64String(publicKey2));
            System.err.println("乙方私钥:\n" + Base64.encodeBase64String(privateKey2));
    
            key1 = DHCoder.getSecretKey(publicKey2, privateKey1);
            System.err.println("甲方本地密钥:\n" + Base64.encodeBase64String(key1));
    
            key2 = DHCoder.getSecretKey(publicKey1, privateKey2);
            System.err.println("乙方本地密钥:\n" + Base64.encodeBase64String(key2));
    
            assertArrayEquals(key1, key2);
        }
    
        /**
         * 校验
         * 
         * @throws Exception
         */
        @Test
        public final void test() throws Exception {
    
            System.err.println("\n=====甲方向乙方发送加密数据=====");
            String input1 = "密码交换算法 ";
            System.err.println("原文: " + input1);
            
            System.err.println("---使用甲方本地密钥对数据加密---");
    
            // 使用甲方本地密钥对数据加密
            byte[] code1 = DHCoder.encrypt(input1.getBytes(), key1);
    
            System.err.println("加密: " + Base64.encodeBase64String(code1));
    
            System.err.println("---使用乙方本地密钥对数据解密---");
    
            // 使用乙方本地密钥对数据解密
            byte[] decode1 = DHCoder.decrypt(code1, key2);
            String output1 = (new String(decode1));
    
            System.err.println("解密: " + output1);
    
            assertEquals(input1, output1);
    
            System.err.println("\n=====乙方向甲方发送加密数据=====");
            String input2 = "DH";
            System.err.println("原文: " + input2);
    
            System.err.println("---使用乙方本地密钥对数据加密---");
    
            // 使用乙方本地密钥对数据加密
            byte[] code2 = DHCoder.encrypt(input2.getBytes(), key2);
    
            System.err.println("加密: " + Base64.encodeBase64String(code2));
    
            System.err.println("---使用甲方本地密钥对数据解密---");
    
            // 使用甲方本地密钥对数据解密
            byte[] decode2 = DHCoder.decrypt(code2, key1);
            String output2 = (new String(decode2));
    
            System.err.println("解密: " + output2);
    
            // 校验
            assertEquals(input2, output2);
        }
    }

输出结果

    公钥: 
    MFwwDQYJKoZIhvcNAQEBBQADSwAwSAJBALOSClHIPTn8wrJf2F6ouo+83Zc/HLx44TEJ3w6+yJsqy9x/kpJ5bEe+WtNdKs4UgQIc8TdJv9qM5quTixxBg8ECAwEAAQ==
    私钥： 
    MIIBVQIBADANBgkqhkiG9w0BAQEFAASCAT8wggE7AgEAAkEAs5IKUcg9OfzCsl/YXqi6j7zdlz8cvHjhMQnfDr7ImyrL3H+SknlsR75a010qzhSBAhzxN0m/2ozmq5OLHEGDwQIDAQABAkByjPQWTaV5K1vMTEYLxJkWfoXhKPqc5IPLM5/emSgBiwyQsszXyuDJdrymUaOTqai0+b+kT4yhMdi9hPFDbENRAiEA9GrkXAnErUQ4SjLM2jChZRDaRZ3hw+rQ08mxYJ3ioJUCIQC8FHeuj5Dgw4NIxBb8qWUYutn4Et/bD8Z/A0hEvHbvfQIhALqBqpnU1mCE0xNDanAAddeiIXzH+hO+5fUGTjT0pY91AiA5MKAT3GPZdJn/DmFPAtNS6b5NyK95FRrulDjtbcFcoQIhAJn7IkDrTO6zXrg/4tnRSgAxZsj/uHZfzXmtaJ3jU5Nk
    
    ---私钥加密——公钥解密---
    原文:
    RSA加密算法
    加密后:
    Z2z/6ZpEEA5OmEAcRUWI9rMFAmOVFR/cg0YJ5HWZrg2bGN02iiF48C3wFMRCxvu56ztxfwCj5/XZWQ7vthJ/vQ==
    解密后:
    RSA加密算法
    
    ---公钥加密——私钥解密---
    原文:
    RSA Encypt Algorithm
    加密后:
    XwnBCJdhaABwErP6kze/B/Iznp1dt/JZ9pt5ODljO8oB5v+CTurQX1YKBkzfI0l4PRaJN75Amv1jDSJrp8iGuQ==
    解密后: RSA Encypt Algorithm

### 数字签名算法

- 数字签名算法可以看作是一种带有密钥的消息摘要算法，是许多网络安全机制的基础，例如现在网站使用的https协议的安全就由数字签名实现；
- 有关数字签名算法更通俗的解释可以参看[阮一峰的博客](http://www.ruanyifeng.com/blog/2011/08/what_is_a_digital_signature.html);
- 与摘要值相同，签名值也常以十六进制字符串表示；
- 数字签名算法常用的包括RSA,DSS与ECDSA算法三种；