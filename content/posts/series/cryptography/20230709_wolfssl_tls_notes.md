---
title: "使用 wolfssl 实现 tls 通信示例"
date: 2023-07-09T14:02:11+08:00
lastmod: 2023-07-09T14:02:11+08:00
author: ["hacper"]
tags:
    - cryptography
    - wolfssl
    - mbedtls
    - tls
categories:
    - 笔记
description: "" # 文章描述，与搜索优化相关
summary: "介绍使用 wolfssl 实现 tls 加密通信的应用流程。" # 文章简单描述，会展示在主页
# weight: # 输入1可以顶置文章，用来给文章展示排序，不填就默认按时间排序
slug: ""
draft: false # 是否为草稿
comments: true
showToc: true # 显示目录
TocOpen: true # 自动展开目录
autonumbering: true # 目录自动编号
hidemeta: false # 是否隐藏文章的元信息，如发布日期、作者等
disableShare: true # 底部不显示分享栏
searchHidden: false # 该页面可以被搜索到
showbreadcrumbs: true #顶部显示当前路径
# mermaid: true

---

## wolfssl 介绍

wolfssl 是一个轻量级的 ssl/tls 实现，可作为嵌入式设备上实现 tls 安全通信的另一个选择。

## API 设计和实现

API 定义还是按照 [mbedtls tls 客户端应用详解](https://hacperme.com/posts/series/cryptography/20230702_mbedtls_tls_notes/)中的设计，内部实现换成 wolfssl 的使用流程。

```c
void *qtf_tls_connect(const char *host, uint16_t port, qtf_tls_conn_param_t *param);
int qtf_tls_send(void *handle, const void *buf, uint32_t len, uint32_t timeout_ms);
int qtf_tls_recv(void *handle, void *buf, uint32_t len, uint32_t timeout_ms);
int qtf_tls_close(void *handle);
```

接口实现：

实现 tls client 主要使用到 wolfssl 下面的这几个 api，相比较于 mbedtls，wolfssl 是使用流程更加简洁。

```c
// 初始化和配置
wolfSSL_Init
wolfTLSv1_2_client_method
wolfTLSv1_3_client_method
wolfSSLv23_client_method
wolfSSL_SetIORecv
wolfSSL_SetIOSend
wolfSSL_CTX_set_verify
wolfSSL_CTX_load_verify_buffer
wolfSSL_CTX_use_certificate_buffer
wolfSSL_CTX_use_PrivateKey_buffer
wolfSSL_set_fd

// 创建和释放资源
wolfSSL_CTX_new
wolfSSL_new    
wolfSSL_free
wolfSSL_CTX_free
wolfSSL_Cleanup

// 连接关闭和收发数据
wolfSSL_connect
wolfSSL_shutdown
wolfSSL_write
wolfSSL_read   
```



```c
#include "qt_idf_config.h"

#include <stdio.h>
#include <stdlib.h>
#include <string.h>

#include "qt_idf_tls.h"
#include "qt_idf_tcp.h"
#include "dlg/dlg.h"
#include "utils_timer.h"
#ifndef WOLFSSL_USER_SETTINGS
    #include <wolfssl/options.h>
#endif
#include <wolfssl/wolfcrypt/settings.h>
#include <wolfssl/ssl.h>
#include <wolfssl/wolfcrypt/wc_port.h>

typedef struct
{
    WOLFSSL_CTX* ctx;
    WOLFSSL*     ssl;
    WOLFSSL_METHOD* method;
    int fd;
}qtf_tls_handle_t;

// 自定义tcp 发送函数
static  int my_IOSend(WOLFSSL* ssl, char* buff, int sz, void* ctx)
{
    int ret = 0;
    int fd = *(int*)ctx;

    ret = qtf_tcp_send((void *)fd, (const char *)buff, sz, 1000);
    if (ret < 0)
    {
        dlg_error("qtf_tcp_send failed");
        return WOLFSSL_CBIO_ERR_GENERAL;
    }

    return ret;
}
// 自定义tcp 接收函数
static int my_IORecv(WOLFSSL* ssl, char* buff, int sz, void* ctx)
{
    int ret = 0;
    int fd = *(int*)ctx;

    ret = qtf_tcp_recv((void *)fd, (char *)buff, sz, 1000);
    if (ret < 0)
    {
        dlg_error("qtf_tcp_recv failed");
        return WOLFSSL_CBIO_ERR_GENERAL;
    }

    return ret;
}

// 初始化和配置参数
static int _tls_net_init(qtf_tls_handle_t *handle, qtf_tls_conn_param_t *param)
{
    int ret = -1;

    wolfSSL_Init();
    
    if(param->tls_version == QTF_TLS_VERSION_TLS1_2)
    {
        handle->method = wolfTLSv1_2_client_method();
    }
    else if(param->tls_version == QTF_TLS_VERSION_TLS1_3)
    {
        handle->method = wolfTLSv1_3_client_method();
    }
    else if(param->tls_version == QTF_TLS_VERSION_TLS1_2_1_3)
    {
        handle->method = wolfSSLv23_client_method();
    }
    else
    {
        dlg_error("not support tls version");
        goto error;
    }

    handle->ctx = wolfSSL_CTX_new(handle->method);
    if(!handle->ctx)
    {
        dlg_error("wolfSSL_CTX_new failed");
        goto error;
    }
	// 注册收发函数回调，使用自定义tcp收发的实现
    wolfSSL_SetIORecv(handle->ctx, my_IORecv);
    wolfSSL_SetIOSend(handle->ctx, my_IOSend);

    if(param->verify_mode == QTF_TLS_VERIFY_MODE_NONE || param->verify_mode == QTF_TLS_VERIFY_MODE_OPTIONAL)
    {
        wolfSSL_CTX_set_verify(handle->ctx, SSL_VERIFY_NONE, NULL);
    }
    else if (param->verify_mode == QTF_TLS_VERIFY_MODE_REQUIRED)
    {
        wolfSSL_CTX_set_verify(handle->ctx, SSL_VERIFY_PEER, NULL);
    }
    else
    {
        dlg_error("invalid verify mode");
        goto error;
    }

   

    if(param->auth_mode == QTF_TLS_AUTH_MODE_CERT)
    {
        if(param->ca_cert && param->ca_cert_len)
        {
            // 加载ca证书
            ret = wolfSSL_CTX_load_verify_buffer(handle->ctx, (const unsigned char *)param->ca_cert, param->ca_cert_len, SSL_FILETYPE_PEM);
            if (ret != SSL_SUCCESS)
            {
                dlg_error("wolfSSL_CTX_load_verify_buffer failed");
                goto error;
            }
        }
        else
        {
            if(param->verify_mode != QTF_TLS_VERIFY_MODE_NONE)
            {
                dlg_error("invalid ca cert");
                goto error;
            }
            dlg_info("verify mode is none");
        }
        
        if (param->client_cert && param->client_cert_len && param->client_key && param->client_key_len)
        {
            ret = wolfSSL_CTX_use_certificate_buffer(handle->ctx, (const unsigned char *)param->client_cert, param->client_cert_len, SSL_FILETYPE_PEM);
            if (ret != SSL_SUCCESS)
            {
                dlg_error("wolfSSL_CTX_use_certificate_buffer failed");
                goto error;
            }

            ret = wolfSSL_CTX_use_PrivateKey_buffer(handle->ctx, (const unsigned char *)param->client_key, param->client_key_len, SSL_FILETYPE_PEM);
            if (ret != SSL_SUCCESS)
            {
                dlg_error("wolfSSL_CTX_use_PrivateKey_buffer failed");
                goto error;
            }
        }
    }
    else if(param->auth_mode == QTF_TLS_AUTH_MODE_PSK)
    {
        
    }
    else
    {
        dlg_error("invalid auth mode");
        goto error;
    }

    handle->ssl = wolfSSL_new(handle->ctx);
    if(!handle->ssl)
    {
        dlg_error("wolfSSL_new failed");
        goto error;
    }

    ret = 0;
    return ret;

error:
    
    return ret;

}

static int __tls_net_deinit(qtf_tls_handle_t *handle)
{
    wolfSSL_shutdown(handle->ssl);
    wolfSSL_free(handle->ssl);
    wolfSSL_CTX_free(handle->ctx);
    wolfSSL_Cleanup();
    // 断开tcp连接
    qtf_tcp_close((void *)handle->fd);
    return 0;
}

void *qtf_tls_connect(const char *host, uint16_t port, qtf_tls_conn_param_t *param)
{
    qtf_tls_handle_t *handle = NULL;
    int ret = 0;

    if(!host || !param)
    {
        dlg_error("invalid param");
        goto error;
    }

    handle = (qtf_tls_handle_t *)malloc(sizeof(qtf_tls_handle_t));
    if(!handle)
    {
        dlg_error("malloc failed");
        goto error;
    }

    ret = _tls_net_init(handle, param);
    if(ret != 0)
    {
        dlg_error("tls net init failed");
        goto error;
    }

    dlg_info("Connecting to %s:%d...", host, port);
    // 先创建 tcp 连接
    handle->fd = (int)qtf_tcp_connect(host, port);
    if (handle->fd <= 0)
    {
        dlg_error("qtf_tcp_connect failed");
        goto error;
    }
    // 将套接字 fd 绑定到 ssl 操作句柄 
    wolfSSL_set_fd(handle->ssl, handle->fd);
	
    // 调用 wolfSSL_connect tls 连接
    ret = wolfSSL_connect(handle->ssl);
    if (ret != SSL_SUCCESS)
    {
        ret = wolfSSL_get_error(handle->ssl, ret);
        dlg_error("wolfSSL_connect failed ret:%d, 0x%04x",ret, ret < 0 ? -ret : ret);
        goto error;
    }
    return handle;

error:

    if(handle)
    {
        __tls_net_deinit(handle);
        free(handle);
    }

    return NULL;
}

int qtf_tls_send(void *handle, const void *buf, uint32_t len, uint32_t timeout_ms)
{
    qtf_tls_handle_t *tls_handle = (qtf_tls_handle_t *)handle;
    int ret = 0;
    int send_len = 0;
    timer_t timer;

    if(!handle || !buf || !len)
    {
        dlg_error("invalid param");
        return -1;
    }

    qtf_timer_init(&timer);
    qtf_timer_countdown_ms(&timer, timeout_ms);

    do
    {
        ret = wolfSSL_write(tls_handle->ssl, (const unsigned char *)buf + send_len, len - send_len);

        if (ret >= 0)
        {
            send_len += ret;
        }
        else
        {
            dlg_error("wolfSSL_write failed 0x%04x", ret < 0 ? -ret : ret);
            break;
        }

    } while (send_len < len && !qtf_timer_expired(&timer));

    if(send_len == 0)
    {
        return -1;
    }
    
    return send_len;
}
int qtf_tls_recv(void *handle, void *buf, uint32_t len, uint32_t timeout_ms)
{
    qtf_tls_handle_t *tls_handle = (qtf_tls_handle_t *)handle;
    int ret = 0;
    timer_t timer;
    int recv_len = 0;

    if(!handle || !buf || !len)
    {
        dlg_error("invalid param");
        return -1;
    }

    qtf_timer_init(&timer);
    qtf_timer_countdown_ms(&timer, timeout_ms);
    do
    {
        ret = wolfSSL_read(tls_handle->ssl, (unsigned char *)buf + recv_len, len - recv_len);

        if (ret >= 0)
        {
            recv_len += ret;
        }
        else
        {
            dlg_error("wolfSSL_read failed 0x%04x", ret < 0 ? -ret : ret);
            break;
        }
    } while (recv_len < len && !qtf_timer_expired(&timer));
    
    if(recv_len == 0)
    {
        return -1;
    }

    return recv_len;
}

int qtf_tls_close(void *handle)
{
    int ret = 0;
    qtf_tls_handle_t *tls_handle = (qtf_tls_handle_t *)handle;
    if (!handle)
    {
        dlg_error("invalid param");
        return -1;
    }
    __tls_net_deinit(tls_handle);
    free(tls_handle);

    return ret;
}
```

## 示例

上层封装的接口没有变，测试例子可以直接使用之前测试 mbedtls tls 连接的例子。

```c
void shell_tlstest_cmd(int argc, char *argv)
{
	qtf_tls_conn_param_t tls_param = {0};
	void *tls_handle = NULL;
	char recv_buf[1024] = {0};
	int ret = 0;

	const char request[] = "GET /get?a=1 HTTP/1.1\r\n"
		"Host: httpbin.org\r\n"
		"Connection: close\r\n"
		"\r\n";
	const char *test_ca_cert =
		{

			"-----BEGIN CERTIFICATE-----\r\n"
			"MIIDdTCCAl2gAwIBAgILBAAAAAABFUtaw5QwDQYJKoZIhvcNAQEFBQAwVzELMAkG\r\n"
			"A1UEBhMCQkUxGTAXBgNVBAoTEEdsb2JhbFNpZ24gbnYtc2ExEDAOBgNVBAsTB1Jv\r\n"
			"b3QgQ0ExGzAZBgNVBAMTEkdsb2JhbFNpZ24gUm9vdCBDQTAeFw05ODA5MDExMjAw\r\n"
			"MDBaFw0yODAxMjgxMjAwMDBaMFcxCzAJBgNVBAYTAkJFMRkwFwYDVQQKExBHbG9i\r\n"
			"YWxTaWduIG52LXNhMRAwDgYDVQQLEwdSb290IENBMRswGQYDVQQDExJHbG9iYWxT\r\n"
			"aWduIFJvb3QgQ0EwggEiMA0GCSqGSIb3DQEBAQUAA4IBDwAwggEKAoIBAQDaDuaZ\r\n"
			"jc6j40+Kfvvxi4Mla+pIH/EqsLmVEQS98GPR4mdmzxzdzxtIK+6NiY6arymAZavp\r\n"
			"xy0Sy6scTHAHoT0KMM0VjU/43dSMUBUc71DuxC73/OlS8pF94G3VNTCOXkNz8kHp\r\n"
			"1Wrjsok6Vjk4bwY8iGlbKk3Fp1S4bInMm/k8yuX9ifUSPJJ4ltbcdG6TRGHRjcdG\r\n"
			"snUOhugZitVtbNV4FpWi6cgKOOvyJBNPc1STE4U6G7weNLWLBYy5d4ux2x8gkasJ\r\n"
			"U26Qzns3dLlwR5EiUWMWea6xrkEmCMgZK9FGqkjWZCrXgzT/LCrBbBlDSgeF59N8\r\n"
			"9iFo7+ryUp9/k5DPAgMBAAGjQjBAMA4GA1UdDwEB/wQEAwIBBjAPBgNVHRMBAf8E\r\n"
			"BTADAQH/MB0GA1UdDgQWBBRge2YaRQ2XyolQL30EzTSo//z9SzANBgkqhkiG9w0B\r\n"
			"AQUFAAOCAQEA1nPnfE920I2/7LqivjTFKDK1fPxsnCwrvQmeU79rXqoRSLblCKOz\r\n"
			"yj1hTdNGCbM+w6DjY1Ub8rrvrTnhQ7k4o+YviiY776BQVvnGCv04zcQLcFGUl5gE\r\n"
			"38NflNUVyRRBnMRddWQVDf9VMOyGj/8N7yy5Y0b2qvzfvGn9LhJIZJrglfCm7ymP\r\n"
			"AbEVtQwdpf5pLGkkeB6zpxxxYu7KyJesF12KwvhHhm4qxFYxldBniYUr+WymXUad\r\n"
			"DKqC5JlR3XC321Y9YeRq4VzW9v493kHMB65jUr9TU/Qr6cf9tveCX4XSQRjbgbME\r\n"
			"HMUfpIBvFSDJ3gyICh3WZlXi/EjJKSZp4A==\r\n"
			"-----END CERTIFICATE-----"};
	tls_param.auth_mode = QTF_TLS_AUTH_MODE_CERT;
	tls_param.verify_mode = QTF_TLS_VERIFY_MODE_OPTIONAL;
	tls_param.hanshake_timeout_ms = 10000;
	tls_param.ca_cert = test_ca_cert;
	tls_param.ca_cert_len = strlen(test_ca_cert);
	tls_param.client_cert = NULL;
	tls_param.client_cert_len = 0;
	tls_param.client_key = NULL;
	tls_param.client_key_len = 0;
	tls_param.client_key_passwd = NULL;
	tls_param.client_key_passwd_len = 0;
	tls_param.psk = NULL;
	tls_param.psk_len = 0;
	tls_param.psk_id = NULL;
	tls_param.debug_level = QTF_TLS_DEBUG_LEVEL_ERROR;
	tls_param.tls_version = QTF_TLS_VERSION_TLS1_2;
	tls_param.max_frag_len = QTF_TLS_MAX_FRAG_LEN_NO_USE;

	tls_handle = qtf_tls_connect("httpbin.org", 443, &tls_param);
	if(tls_handle)
	{
		shell_printf("tls connect success\r\n");
	}
	else
	{
		shell_printf("tls connect failed\r\n");
		return;
	}

	ret = qtf_tls_send(tls_handle, request, strlen(request), 1000);
	if(ret > 0)
	{
		shell_printf("tls send success\r\n");
	}
	else
	{
		shell_printf("tls send failed\r\n");
		qtf_tls_close(tls_handle);
		return;
	}

	ret = qtf_tls_recv(tls_handle, recv_buf, sizeof(recv_buf), 1000);
	if(ret > 0)
	{
		shell_printf("tls recv success\r\n");
		shell_printf("recv:%s\r\n", recv_buf);
	}
	else
	{
		shell_printf("tls recv failed\r\n");
		qtf_tls_close(tls_handle);
		return;
	}

	qtf_tls_close(tls_handle);

}
```

测试日志：

```bash
[qt_idf_tls.c:457 qtf_tls_connect] Connecting to httpbin.org:443...
[qt_idf_tcp.c:95 qtf_tcp_connect] connect to httpbin.org:443 success fd:744
[qt_idf_tcp.c:176 qtf_tcp_send] send 242 bytes
[qt_idf_tcp.c:267 qtf_tcp_recv] recv 5 bytes
[qt_idf_tcp.c:267 qtf_tcp_recv] recv 86 bytes
[qt_idf_tcp.c:267 qtf_tcp_recv] recv 5 bytes
[qt_idf_tcp.c:267 qtf_tcp_recv] recv 4953 bytes
[qt_idf_tcp.c:267 qtf_tcp_recv] recv 5 bytes
[qt_idf_tcp.c:267 qtf_tcp_recv] recv 333 bytes
[qt_idf_tcp.c:267 qtf_tcp_recv] recv 5 bytes
[qt_idf_tcp.c:267 qtf_tcp_recv] recv 4 bytes
[qt_idf_tcp.c:176 qtf_tcp_send] send 126 bytes
[qt_idf_tcp.c:267 qtf_tcp_recv] recv 5 bytes
[qt_idf_tcp.c:267 qtf_tcp_recv] recv 1 bytes
[qt_idf_tcp.c:267 qtf_tcp_recv] recv 5 bytes
[qt_idf_tcp.c:267 qtf_tcp_recv] recv 40 bytes
tls connect success
[qt_idf_tcp.c:176 qtf_tcp_send] send 92 bytes
tls send success
[qt_idf_tcp.c:267 qtf_tcp_recv] recv 5 bytes
[qt_idf_tcp.c:267 qtf_tcp_recv] recv 468 bytes
[qt_idf_tcp.c:267 qtf_tcp_recv] recv 5 bytes
[qt_idf_tcp.c:267 qtf_tcp_recv] recv 26 bytes
[qt_idf_tls.c:651 qtf_tls_recv] wolfSSL_read failed 0x0001
tls recv success
recv:HTTP/1.1 200 OK
Date: Sun, 09 Jul 2023 06:51:29 GMT
Content-Type: application/json
Content-Length: 219
Connection: close
Server: gunicorn/19.9.0
Access-Control-Allow-Origin: *
Access-Control-Allow-Credentials: true

{
  "args": {
    "a": "1"
  },
  "headers": {
    "Host": "httpbin.org",
    "X-Amzn-Trace-Id": "Root=1-64aa58f1-17ce1f42274e9e4e44d6ea95"
  },
  "origin": "223.73.211.53",
  "url": "https://httpbin.org/get?a=1"
}

[qt_idf_tcp.c:176 qtf_tcp_send] send 31 bytes
```



## 参考资料

- [https://github.com/wolfSSL/wolfssl](https://github.com/wolfSSL/wolfssl)
- [https://github.com/wolfSSL/wolfssl-examples/tree/master/tls](https://github.com/wolfSSL/wolfssl-examples/tree/master/tls)
- [https://hacperme.com/posts/series/cryptography/20230702_mbedtls_tls_notes/](https://hacperme.com/posts/series/cryptography/20230702_mbedtls_tls_notes/)