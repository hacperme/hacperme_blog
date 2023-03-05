---
id: 1682
title: '用 Python 实现多看阅读签到领书币'
date: '2018-09-30T14:17:04+08:00'
author: hacper
excerpt: "对 Android 多看阅读 APP 进行抓包，可以获取其每天签到领书币的请求接口。\n请求头 Cookie 中的 token 来自账号登录成功后返回的数据里，请求体中的_t 参数是当前时间（时间戳），_c 是由时间戳和 device_id 计算得来的。\n\n用 Apk Extractor Lite 提取出多看阅读的安装包，再用 MT管理器 打开，反编译后可以得到生成 _t _C 参数的 JAVA 程序：\n类似的，可以写成 Python 程序：\n"
layout: post
guid: 'https://hacperme.com/?p=1682'
permalink: /2018/09/30/duokan_python/
Steempress_sp_steem_publish:
    - '0'
views:
    - '3270'
classic-editor-remember:
    - classic-editor
categories:
    - 笔记
tags:
    - python
    - 多看阅读
    - 签到
---

![image.png](https://ipfs.busy.org/ipfs/QmZHQJL6NNq3geu6Q7EhBS9wju86xZATqwn6rtNHr8s8mi)

对 Android 多看阅读 APP 进行抓包，可以获取其每天签到领书币的请求接口。

```http
POST /checkin/v0/checkin HTTP/1.1
Host: www.duokan.com
Content-Type: application/x-www-form-urlencoded
Accept-Encoding:  gzip,deflate
Cookie:  device_id=***************************;app_id=DkReader.Android;build=562180926;channel=TBVP2Y;user_type=1;device_hash=###################;device_hash_set=null;token=XXXXXXXXXXXXXXXXXXXXXXX;

_t=1538224732&_c=10319

```

请求头 Cookie 中的 token 来自账号登录成功后返回的数据里，请求体中的\_t 参数是当前时间（时间戳），\_c 是由时间戳和 device\_id 计算得来的。

用 Apk Extractor Lite 提取出多看阅读的安装包，再用 MT管理器 打开，反编译后可以得到生成 \_t \_C 参数的 JAVA 程序：

```java
public static String[] genCsrfCode() {
        String[] split = (ReaderEnv.get().getDeviceId() + '&' + ((int) (System.currentTimeMillis() / 1000))).split("");
        int i = 0;
        for (int i2 = 1; i2 < split.length; i2++) {
            i = generate(i, split[i2]);
        }
        return new String[]{"_t", String.valueOf(r4), "_c", String.valueOf(i)};
    }


private static int generate(int i, String str) {
        return ((i * 131) + str.getBytes()[0]) % 65536;
    }

```

类似的，可以写成 Python 程序：

```python
import requests
from datetime import datetime
import time

```

```python
config = {
    'device_id': '************************',
    'app_id': 'DkReader.Android',
    'build': '562180926',
    'device_hash': '######################',
    'token': 'XXXXXXXXXXXXXXXXXXXXXXXXXXXXXX..'
}

```

```python
def get_csrf_params(device_id):
    t = int(datetime.now().timestamp())
    string = device_id + '&' + str(t)
    i = 0
    c = 0
    while i < len(string):
        c = (c * 131 + string[i].encode()[0]) % 65536
        i += 1
    return t, c

```

签到：

```python
def duokanqd(config):
    url = 'https://www.duokan.com/checkin/v0/checkin'
    Cookie = "device_id={device_id};app_id={app_id};build={build};channel=TBVP2Y;user_type=1;device_hash={device_hash};\
device_hash_set=null;token={token};".format(**config)
    headers = {
        'Content-Type':
        'application/x-www-form-urlencoded',
        'User-Agent':
        'Dalvik/2.1.0 (Linux; U; Android 8.0.0; PRA-AL00X Build/HONORPRA-AL00X)',
        'Accept-Encoding':
        'gzip,deflate',
        'Cookie':
        Cookie
    }

    _t, _c = get_csrf_params(config.get('device_id', 'D0063006284a6677'))
    payload = {'_t': _t, '_c': _c}
    rsp = requests.post(url, headers=headers, data=payload)
    return rsp.json()

```

```python
print(duokanqd(config))

```

```json
{'msg': '今日已签到', 'result': 500002}

```

写个定时任务，每天执行一下这个签到程序就可以自动领书币了。  
抓包获取的 token 总会到失效的一天，得通过多看的登录接口更新 token，其登录API为：

```http
POST /dk_id/api/wx/login HTTP/1.1
Host: www.duokan.com
Content-Type: application/x-www-form-urlencoded
Cookie:  device_id=XXXXXXXXXXXXXXXX;device_hash=********************
User-Agent:  Dalvik/2.1.0 (Linux; U; Android 8.0.0; PRA-AL00X Build/HONORPRA-AL00X)
Accept-Encoding:  gzip

package=com.duokan.reader&code=XXXXXXXXXXXXX&_t=1538236192&_c=60082

```

这是我用微信账号授权登录的请求，code 这个参数是使用微信授权登录的一个临时令牌，code 是一次性的，用过就失效了。code 这个参数获取不了，想要自动登录多看账号的想法失败。

如果不用微信登录，而直接用小米的账号密码登录，是不是可以成功登录多看呢？得注册个小米账号试试。