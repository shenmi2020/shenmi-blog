---
title: PHP银行加签加密对接
date: 2022-06-30 11:38:17
categories: [编程]
tags:
 - [php]
---

PHP对接银行加密代码记录

<!-- more -->
``` php

/**
 * 签名
 * @param string $data 签名内容
 * @param string $pub_key 请求方私钥
 */
public function getSign($data, $private_key)
{
    $data = iconv('UTF-8', 'GB2312', $data);
    $private_key = base64_encode(hex2bin($private_key));
    $private_key = chunk_split($private_key, 64, "\n");
    $private_key = "-----BEGIN RSA PRIVATE KEY-----\n$private_key-----END RSA PRIVATE KEY-----\n";
    openssl_sign($data, $signature, $private_key, OPENSSL_ALGO_SHA1);

    return strtoupper(bin2hex($signature));
}

/**
 * 验证签名
 * @param string $data 签名内容
 * @param string $sign 签名
 * @param string $pub_key 请求方公钥
 */
public function verifySign($data, $sign, $pub_key)
{
    $data = iconv('UTF-8', 'GB2312', $data);
    $sign = hex2bin($sign);
    $pub_key = base64_encode(hex2bin($pub_key));
    $pub_key = chunk_split($pub_key, 64, "\n");
    $pub_key = "-----BEGIN PUBLIC KEY-----\n$pub_key-----END PUBLIC KEY-----\n";
    $result = openssl_verify($data, $sign, $pub_key) === 1;

    return $result;
}

/**
 * 对称加密
 * @param string $data 加密内容
 * @param string $key 密钥 $key = '9B6C238C991BCA09543CB926C43B3DE4';
 * @param string $iv 向量 $iv = 'AED7966C958C149632180F51EDBC9313';
 */
public function encryptAes($data, $key, $iv)
{
    $key = hex2bin($key);
    $iv = hex2bin($iv);
    $data = iconv('UTF-8', 'GB2312', $data);
    $result = openssl_encrypt($data, 'AES-128-CBC', $key, OPENSSL_RAW_DATA, $iv);

    return strtoupper(bin2hex($result));
}

/**
 * 对称解密
 * @param string $data 加密内容
 * @param string $key 密钥 $key = '9B6C238C991BCA09543CB926C43B3DE4';
 * @param string $iv 向量 $iv = 'AED7966C958C149632180F51EDBC9313';
 */
public function decryptAes($data, $key, $iv)
{
    $data = hex2bin($data);
    $key = hex2bin($key);
    $iv = hex2bin($iv);
    
    $result = openssl_decrypt($data, 'AES-128-CBC', $key, OPENSSL_RAW_DATA, $iv);
    $result = iconv('GB2312', 'UTF-8', $result);

    return $result;
}

/**
 * 非对称解密
 * @param string $data 加密内容
 * @param string $key 服务方私钥解密
 */
public function decryptRsa($data, $key)
{
    $data = hex2bin($data);
    $key = base64_encode(hex2bin($key));
    $key = chunk_split($key, 64, "\n");
    $key = "-----BEGIN RSA PRIVATE KEY-----\n$key-----END RSA PRIVATE KEY-----\n";
    openssl_private_decrypt($data, $decrypted, $key);
    $decrypted = iconv('GB2312', 'UTF-8', $decrypted);
    
    return $decrypted;
}

/**
 * 非对称加密
 * @param string $data 加密内容
 * @param string $key 服务方公钥加密
 */
public function encryptRsa($data, $key)
{
    $data = iconv('UTF-8', 'GB2312', $data);
    $key = base64_encode(hex2bin($key));
    $key = chunk_split($key, 64, "\n");
    $key = "-----BEGIN PUBLIC KEY-----\n$key-----END PUBLIC KEY-----\n";
    openssl_public_encrypt($data, $crypted, $key);
    
    return strtoupper(bin2hex($crypted));
}    

```