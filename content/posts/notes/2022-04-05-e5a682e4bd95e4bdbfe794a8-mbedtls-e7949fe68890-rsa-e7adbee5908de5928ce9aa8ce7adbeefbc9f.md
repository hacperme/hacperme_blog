---
id: 1960
title: '如何使用 mbedtls 生成 RSA 签名和验签？'
date: '2022-04-05T00:45:20+08:00'
author: hacper
excerpt: '本文是一篇使用 mbedtls 和 python 实现签名验证的应用笔记。某个一个项目需要用到 RSA 数字签名验签算法，记录了我实现签名验签功能的一些探索和验证。'
layout: post
guid: 'https://hacperme.com/?p=1960'
permalink: /2022/04/05/%e5%a6%82%e4%bd%95%e4%bd%bf%e7%94%a8-mbedtls-%e7%94%9f%e6%88%90-rsa-%e7%ad%be%e5%90%8d%e5%92%8c%e9%aa%8c%e7%ad%be%ef%bc%9f/
views:
    - '2221'
categories:
    - 笔记
tags:
    - mbedtls
    - python
    - 数字签名
---

非对称加密会生成一个密钥对：公钥和私钥。公钥公之于众，而私钥需要保密，不公开。一般密钥的使用方法如下表，对于加密的应用场景，发送者使用公钥加密消息，接收者使用私钥解密密文，目的是在发送者和接收者之间安全地、秘密地传递消息；对于数字签名的使用场景，发送者使用私钥对消息摘要进行签名，接收者使用公钥对签名进行验签，验签的作用是鉴别发送者身份的真实性和验证消息的完整性。

| <th>Algorithm</th> | <th>Sender uses..</th> | <th>Receiver uses…</th> |
|--------------------|------------------------|---------------------------|
| Encryption | Public key | Private key |
| Signature | Private key | Public key |

我们使用python和C语言来介绍如何生成 RSA 签名并验证签名，使用到的加密库是 Cryptodome 和 mbedtls。首先使用 python 生成签名，在 c 代码中进行验签；然后也在 c 代码中生成签名，在python里进行验签，目的是使用两种编程环境，方便相互验证。

## 在python中生成签名

导入 python 包，需要用到 Cryptodome 的 PublicKey、Signature、Hash 以及 base64 编码解码这几个模块。

```python
from Cryptodome.PublicKey import RSA
from Cryptodome.Signature import pkcs1_15
from Cryptodome.Hash import SHA256
from base64 import b64encode,b64decode
```

生成公钥和私钥，并输出为 PEM 格式。

```python
# 生成密钥对
keyPair = RSA.generate(bits=1024)

# 私钥
print(keyPair.export_key("PEM"))
# 公钥
print(keyPair.public_key().export_key("PEM"))
```

私钥：

```
-----BEGIN RSA PRIVATE KEY-----\n
MIICXQIBAAKBgQDTt8tp4xNp29CMxy6QS0NzpR6t8bAcv7ei3NkVM/Nzg3K5wWZR\n
aBTMovbzKCXdXYdC6GutVkG+CEetO3XHM4LhDqW0vwISTO65/XrvR3zqXD5ZjrJF\n
mtCAvkCwtMAPjqXZ/RJnd8yrXuoz5cRqVgKmq5TZlGIIiTPIklxGIGof8QIDAQAB\n
AoGAFf1BJoiD5+sBdFmsq6ZxhUWZU+ImEzpTUZpD/riEWNNGe2YLoTlg7acgZH1f\n
P2hbJ9cZdemfTuQvw52JHE0sktCUM6R0wq5rlbDj740+5yZYzs9FlUntm6UtoU9w\n
tpd62/iPxovFkguunJB2KBbtP8q0dYQntATEce1TZuS3trUCQQDl7VRYygSb3/HY\n
ij2ya1592WpgNWgmPvbpmUjGGBvjmnO8Ye1lEy6x69RmGjRrLvFfhWYwcF2HpmYQ\n
9wXKEwT1AkEA67nc/CdeT4j9jRE/QFXlhVrW8Gq8IfjXFGbGK5BqlTRbty3OpW+L\n
M9GPqiMC2XxN60peEiANlQ8aUnvbHZexjQJAcz4RGK+ov7fvL+maIuNN6SYf+zjJ\n
iuHkQBFkOGW9FMdFWxZ6Nj73GJZrTwGzZEWTFZ13KrAnMOZmIfquHCqMQQJBAL+u\n
x9ATg1FRqDyKBdEfCCDEmXuuj4VggCUK3aKXMNRbWyk9iohkh+F/Sz+icLLBreri\n
8lPy1JidS14/cRJDRBECQQCT4oNvmV5CYzqkqbgwtLPi/FIjc6Zi26DGxBzL01V+\n
yTO1ZlOOUOtY4dPBnU4COkdq6hWqum/Q6kiVj91qAUHN\n
-----END RSA PRIVATE KEY-----
```

公钥：

```
-----BEGIN PUBLIC KEY-----\n
MIGfMA0GCSqGSIb3DQEBAQUAA4GNADCBiQKBgQDTt8tp4xNp29CMxy6QS0NzpR6t\n
8bAcv7ei3NkVM/Nzg3K5wWZRaBTMovbzKCXdXYdC6GutVkG+CEetO3XHM4LhDqW0\n
vwISTO65/XrvR3zqXD5ZjrJFmtCAvkCwtMAPjqXZ/RJnd8yrXuoz5cRqVgKmq5TZ\n
lGIIiTPIklxGIGof8QIDAQAB\n
-----END PUBLIC KEY-----
```

定义将要签名数据，并计算哈希值，哈希算法使用 SHA256。

```python
# 消息
msg = b'A message for signing'

# 生成加密算子
c = pkcs1_15.new(keyPair)
h = SHA256.new()

# 计算hash，并打印为16进制字符串
h.update(msg)
print(h.digest().hex())
```

```
47f53245cd05a2b3e811ad6515000b44604b947a57d441b02125b04f4a16bb74
```

对哈希值进行签名，并输出签名数据为16进制字符串

```python
sign = c.sign(h)
print(sign.hex())
```

```
29889917f0b5f0edf080266f0e9b5f33c561fce3ccadc01fea77bd7accd2bb180630ae7b70a090b96737dc917c89098a5ab4a8f9ecf390f2487d71938f7cf726cf6fbf50c2dad5f7a0187d09d645fb273932a6404f92c412a9b034a5a24f888dc309db5e226d352cbcfd3d8f4513743c1cbf4d99f71ca73500c28c60c2d48dff
```

最终对签名数据进行 base64 编码，相比于16进制字符串数据，base64 编码得到的数据长度相对较小，这应该是对二进制数据使用 base64编码的一个优势。

```python
print(b64encode(sign))
```

```
KYiZF/C18O3wgCZvDptfM8Vh/OPMrcAf6ne9eszSuxgGMK57cKCQuWc33JF8iQmKWrSo+ezzkPJIfXGTj3z3Js9vv1DC2tX3oBh9CdZF+yc5MqZAT5LEEqmwNKWiT4iNwwnbXiJtNSy8/T2PRRN0PBy/TZn3HKc1AMKMYMLUjf8=
```

## 在c中验证签名

在c中使用 mbedtls 库进行签名验签，使用到 pk.h、md.h 和 base64.h 这几个头文件定义的接口。

先定义和实现验证签名的接口 rsa\_pkcs1v15\_sha256\_verify，输入参数有原始消息 msg，PEM 格式的公钥 public\_key\_pem，以及base64 编码后的签名数据 sign\_base64。

```c
#include "mbedtls/pk.h"
#include "mbedtls/md.h"
#include "mbedtls/base64.h"

/**
 * @brief rsa_pkcs1v15_sha256_verify
 * 
 * @param [in] msg
 * @param [in] msg_len
 * @param [in] public_key_pem
 * @param [in] sign_base64
 * @return int 
 *  -- 0  verify pass
 *  -- -1 verify faild
 */
int rsa_pkcs1v15_sha256_verify(const unsigned char *msg, size_t msg_len,
                               const char *public_key_pem, const char *sign_base64)
{
    mbedtls_pk_context pk = {0};
    unsigned char hash[32] = {0};
    int ret = 0;
    size_t sign_len = 0;
    size_t b64out_len = 0;
    unsigned char *b64out_data = NULL;

    // 初始化上下文
    mbedtls_pk_init( &pk);

    // 导入公钥
    ret = mbedtls_pk_parse_public_key(&pk, (const unsigned char *)public_key_pem, strlen(public_key_pem)+1);
    if(ret != 0)
    {
        ret = -1;
        goto exit;
    }

    // 对需要验签的数据进行 sha256 计算，生成消息摘要数据
    ret = mbedtls_md(mbedtls_md_info_from_type( MBEDTLS_MD_SHA256 ),
                     (const unsigned char *)msg, msg_len, hash);
    if(ret != 0)
    {
        ret = -1;
        goto exit;
    }

    // 对原始签名数据进行 base64 解码
    sign_len = strlen(sign_base64);
    b64out_data = malloc(sign_len*2);
    memset(b64out_data, 0, sign_len*2);
    ret = mbedtls_base64_decode(b64out_data, sign_len*2, &b64out_len, (const unsigned char *)sign_base64, sign_len);
    if(ret != 0)
    {
        ret = -1;
        goto exit;
    }

    // 验证签名

    ret = mbedtls_pk_verify(&pk, MBEDTLS_MD_SHA256, hash, sizeof (hash), b64out_data, b64out_len);

exit:
    if(b64out_data)
    {
        free(b64out_data);
    }
    mbedtls_pk_free( &pk );

    return ret;

}
```

测试验签代码如下

```c
static void test_rsa_pkcs1_verify(void)
{

    int ret = 0;
    // 公钥
    char *pub_key = "-----BEGIN PUBLIC KEY-----\n"
                    "MIGfMA0GCSqGSIb3DQEBAQUAA4GNADCBiQKBgQDTt8tp4xNp29CMxy6QS0NzpR6t\n"
                    "8bAcv7ei3NkVM/Nzg3K5wWZRaBTMovbzKCXdXYdC6GutVkG+CEetO3XHM4LhDqW0\n"
                    "vwISTO65/XrvR3zqXD5ZjrJFmtCAvkCwtMAPjqXZ/RJnd8yrXuoz5cRqVgKmq5TZ\n"
                    "lGIIiTPIklxGIGof8QIDAQAB\n"
                    "-----END PUBLIC KEY-----";
    // 原始消息
    char *msg = "A message for signing";

    // base64 编码之后的签名数据
    char * sign = "KYiZF/C18O3wgCZvDptfM8Vh/OPMrcAf6ne9eszSuxgGMK57cKCQuWc33JF8iQmKWrSo"
                  "+ezzkPJIfXGTj3z3Js9vv1DC2tX3oBh9CdZF+yc5MqZAT5LEEqmwNKWiT4iNwwnbXiJt"
                  "NSy8/T2PRRN0PBy/TZn3HKc1AMKMYMLUjf8=";

    ret = rsa_pkcs1v15_sha256_verify((const unsigned char *)msg, strlen(msg), pub_key, sign);

    printf("rsa_pkcs1v15_sha256_verify ret=%d\r\n", ret);

}
```

输出，返回值为0，验签成功。

```
rsa_pkcs1v15_sha256_verify ret=0
```

## 在c中生成签名

定义生成签名的接口 rsa\_pkcs1v15\_sha256\_sign，输入参数有原始数据msg、用于签名的私钥priavte\_key\_pem、保存签名输出数据的地址sign\_base64。

```c
int rsa_pkcs1v15_sha256_sign(const unsigned char *msg, size_t msg_len,
                               const char *priavte_key_pem, char *sign_base64, int sign_len)
{
    mbedtls_pk_context pk;
    mbedtls_entropy_context entropy;
    mbedtls_ctr_drbg_context ctr_drbg;

    uint8_t sig_buff[SIGNATURE_MAX_SIZE];
    unsigned char hash[32] = {0};
    size_t sig_len = 0;
    int ret = 0;
    char *b64_out = NULL;
    int b64_len = 0;
    const char *pers = "mbedtls_pk_sign";       // Personalization data,
    // that is device-specific identifiers. Can be NULL.

    // 初始化随机数生成器
    mbedtls_entropy_init( &entropy );
    mbedtls_ctr_drbg_init( &ctr_drbg );

    //初始化上下文
    mbedtls_pk_init( &pk );

    mbedtls_ctr_drbg_seed( &ctr_drbg,
                           mbedtls_entropy_func,
                           &entropy,
                           (const unsigned char *) pers,
                           strlen( pers ) );

    //导入私钥
    ret = mbedtls_pk_parse_key(&pk, (const unsigned char *)priavte_key_pem,
                               strlen(priavte_key_pem)+1,
                               NULL, 0);
    if(ret != 0)
    {
        ret = -1;
        goto exit;
    }

    // 计算 sha256 消息摘要
    ret = mbedtls_md(mbedtls_md_info_from_type( MBEDTLS_MD_SHA256 ),
                     (const unsigned char *)msg, msg_len, hash);
    if(ret != 0)
    {
        ret = -1;
        goto exit;
    }

    // 签名
    ret = mbedtls_pk_sign(&pk, MBEDTLS_MD_SHA256, hash, sizeof (hash), sig_buff, &sig_len, mbedtls_ctr_drbg_random, &ctr_drbg);

    if(ret != 0)
    {
        ret = -1;
        goto exit;
    }

    b64_out = malloc(sig_len*2);
    if(b64_out == NULL)
    {
        ret = -1;
        goto exit;
    }

    // 对签名数据进行 base64 编码
    ret = mbedtls_base64_encode((unsigned char *)b64_out, sig_len*2,
                                (size_t *)&b64_len, (unsigned char *)sig_buff, (size_t)sig_len);

    if(ret != 0)
    {
        ret = -1;
        goto exit;
    }

    if(sign_len<b64_len)
    {
        ret = -1;
        goto exit;
    }

    strncpy(sign_base64, b64_out, sign_len);

exit:

    if(b64_out)
    {
        free(b64_out);
    }

    mbedtls_pk_free( &pk );
    mbedtls_ctr_drbg_free( &ctr_drbg );
    mbedtls_entropy_free( &entropy );

    return ret;

}
```

生成签名的测试代码

```c
static void test_rsa_pkcs1_sign(void)
{

    int ret = 0;

    char *private_key = "-----BEGIN RSA PRIVATE KEY-----\n"
                        "MIICXQIBAAKBgQDTt8tp4xNp29CMxy6QS0NzpR6t8bAcv7ei3NkVM/Nzg3K5wWZR\n"
                        "aBTMovbzKCXdXYdC6GutVkG+CEetO3XHM4LhDqW0vwISTO65/XrvR3zqXD5ZjrJF\n"
                        "mtCAvkCwtMAPjqXZ/RJnd8yrXuoz5cRqVgKmq5TZlGIIiTPIklxGIGof8QIDAQAB\n"
                        "AoGAFf1BJoiD5+sBdFmsq6ZxhUWZU+ImEzpTUZpD/riEWNNGe2YLoTlg7acgZH1f\n"
                        "P2hbJ9cZdemfTuQvw52JHE0sktCUM6R0wq5rlbDj740+5yZYzs9FlUntm6UtoU9w\n"
                        "tpd62/iPxovFkguunJB2KBbtP8q0dYQntATEce1TZuS3trUCQQDl7VRYygSb3/HY\n"
                        "ij2ya1592WpgNWgmPvbpmUjGGBvjmnO8Ye1lEy6x69RmGjRrLvFfhWYwcF2HpmYQ\n"
                        "9wXKEwT1AkEA67nc/CdeT4j9jRE/QFXlhVrW8Gq8IfjXFGbGK5BqlTRbty3OpW+L\n"
                        "M9GPqiMC2XxN60peEiANlQ8aUnvbHZexjQJAcz4RGK+ov7fvL+maIuNN6SYf+zjJ\n"
                        "iuHkQBFkOGW9FMdFWxZ6Nj73GJZrTwGzZEWTFZ13KrAnMOZmIfquHCqMQQJBAL+u\n"
                        "x9ATg1FRqDyKBdEfCCDEmXuuj4VggCUK3aKXMNRbWyk9iohkh+F/Sz+icLLBreri\n"
                        "8lPy1JidS14/cRJDRBECQQCT4oNvmV5CYzqkqbgwtLPi/FIjc6Zi26DGxBzL01V+\n"
                        "yTO1ZlOOUOtY4dPBnU4COkdq6hWqum/Q6kiVj91qAUHN\n"
                        "-----END RSA PRIVATE KEY-----";
    char *msg = "A message for signing";

    char sign[1024] = {0};

    ret = rsa_pkcs1v15_sha256_sign((const unsigned char *)msg, strlen(msg), private_key, sign, sizeof (sign));

    printf("rsa_pkcs1v15_sha256_sign ret=%d\r\n", ret);

    if(ret == 0)
    {
        printf("sign:%s\r\n", sign);
    }

}
```

输出结果

```
rsa_pkcs1v15_sha256_verify ret=0
hash digest before sig
47F53245CD05A2B3E811AD6515000B44604B947A57D441B02125B04F4A16BB74
rsa_pkcs1v15_sha256_sign ret=0
sign:KYiZF/C18O3wgCZvDptfM8Vh/OPMrcAf6ne9eszSuxgGMK57cKCQuWc33JF8iQmKWrSo+ezzkPJIfXGTj3z3Js9vv1DC2tX3oBh9CdZF+yc5MqZAT5LEEqmwNKWiT4iNwwnbXiJtNSy8/T2PRRN0PBy/TZn3HKc1AMKMYMLUjf8=
```

签名数据 `KYiZF/C18O3wgCZvDptfM8Vh/OPMrcAf6ne9eszSuxgGMK57cKCQuWc33JF8iQmKWrSo+ezzkPJIfXGTj3z3Js9vv1DC2tX3oBh9CdZF+yc5MqZAT5LEEqmwNKWiT4iNwwnbXiJtNSy8/T2PRRN0PBy/TZn3HKc1AMKMYMLUjf8=` 与在python中生成的一致，说明这段c代码也是正确的。

## 在python中验证签名

python 验签的测试代码比较简单，如下所示

```python
sign_b64 = "KYiZF/C18O3wgCZvDptfM8Vh/OPMrcAf6ne9eszSuxgGMK57cKCQuWc33JF8iQmKWrSo+ezzkPJIfXGTj3z3Js9vv1DC2tX3oBh9CdZF+yc5MqZAT5LEEqmwNKWiT4iNwwnbXiJtNSy8/T2PRRN0PBy/TZn3HKc1AMKMYMLUjf8="

pub_key = "-----BEGIN PUBLIC KEY-----\nMIGfMA0GCSqGSIb3DQEBAQUAA4GNADCBiQKBgQDTt8tp4xNp29CMxy6QS0NzpR6t\n8bAcv7ei3NkVM/Nzg3K5wWZRaBTMovbzKCXdXYdC6GutVkG+CEetO3XHM4LhDqW0\nvwISTO65/XrvR3zqXD5ZjrJFmtCAvkCwtMAPjqXZ/RJnd8yrXuoz5cRqVgKmq5TZ\nlGIIiTPIklxGIGof8QIDAQAB\n-----END PUBLIC KEY-----"

#base64 解码
sign_data = b64decode(sign_b64)

# 导入公钥
pub = RSA.import_key(pub_key)

c = pkcs1_15.new(pub)
hashs = SHA256.new()
hashs.update(msg)

# 验签
try:
    c.verify(hashs, sign_data)
    print("verify ok")
except ValueError:
    print("verify faild")

```

打印输出验签结果，也是验签成功的。

```
verify ok
```