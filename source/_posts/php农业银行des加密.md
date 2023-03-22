---
title: php农业银行des加密
date: 2023-03-22 15:49:51
categories: [编程]
tags:
 - [php]
---

对接农行支付，快E付有额度限制，后来采用跳转掌银app方式
<!-- more -->

生成的url采用des加密

JAVA版
``` java
public static void main(String[] args) {
    try {
        String str = encrypt("wlfffEF=16794457132262171782", "*****");
        str = URLEncoder.encode(str, "UTF-8");
        System.out.println("https://wx.abchina.com/webank/main-view/openTagForId?id=6HV%2FDdixEes%3D&dynamicData=" + str);

    } catch (Exception e) {
        System.out.println("error:"+e.getMessage());
    }
    
    SpringApplication.run(DemoApplication.class, args);
}

public static String encrypt(String data, String sKey) {
    try {
        byte[] key = sKey.getBytes();

        IvParameterSpec iv = new IvParameterSpec(key);
        System.out.println("iv:"+iv);
        DESKeySpec desKey = new DESKeySpec(key);

        SecretKeyFactory keyFactory = SecretKeyFactory.getInstance("DES");
        SecretKey securekey = keyFactory.generateSecret(desKey);

        Cipher cipher = Cipher.getInstance("DES/CBC/PKCS5Padding");

        cipher.init(Cipher.ENCRYPT_MODE, securekey, iv);

        return Base64.getEncoder().encodeToString(cipher.doFinal(data.getBytes()));
    } catch (Throwable e) {
        e.printStackTrace();
    }
    return null;
}
```

php版
``` php
$token = "wlfffEF=" . $result['TOKEN'];
$iv = "******";
$result = openssl_encrypt($token, 'DES-CBC', $iv, OPENSSL_RAW_DATA, $iv);
$result = base64_encode($result);
$result = urlencode($result);

$url = "https://wx.abchina.com/webank/main-view/openTagForId?id=6HV%2FDdixEes%3D&dynamicData=" . $result;
```