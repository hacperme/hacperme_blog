---
id: 1619
title: 'Learn Python with Steem #09 笔记'
date: '2018-08-20T06:00:24+08:00'
author: hacper
excerpt: "## 划重点\n\n- 遍历字典\n\n 利用dict.items()方法，用一个循环语句遍历整个字典的所有元素。\n\n\n- 转换 Reputation 和 Voting Power 的原始数据\n\n 直接获取的某些数据是生的，需要煮(转换)一下才能吃。\n\n\n- 解析时间\n \n 使用Python的dateutil和datetime模块来解析和处理时间数据。"
layout: post
guid: 'https://hacperme.com/?p=1619'
permalink: /2018/08/20/learn-python-with-steem-09-note/
Steempress_sp_steem_publish:
    - '1'
views:
    - '1306'
classic-editor-remember:
    - classic-editor
Steempress_sp_steem_update:
    - '1'
steempress_sp_permlink:
    - learnpythonwithsteem09-z0dpwgv61i
steempress_sp_author:
    - yjcps
categories:
    - 笔记
tags:
    - blog
    - cn
    - cn-programming
    - da-learnpythonwithsteem
    - python
---

# Learn Python with Steem #09 笔记

- - - - - -

\[toc\]

## 划重点

- 遍历字典 利用dict.items()方法，用一个循环语句遍历整个字典的所有元素。
- 转换 Reputation 和 Voting Power 的原始数据
  
  直接获取的某些数据是生的，需要煮(转换)一下才能吃。
- 解析时间
  
  使用Python的dateutil和datetime模块来解析和处理时间数据。

## 编程练习

```python
from steem import Steem
from steem.converter import Converter
# import sys
import math
from dateutil import parser
from dateutil.tz import *
from datetime import datetime, timezone, timedelta
from pprint import pprint

```

```python
class Steemains(Steem):
    def __init__(self, _account_name='yjcps'):
        Steem.__init__(self)
        self.account_name = _account_name
        self.account_info = self.get_account(self.account_name)

    @property
    def view_account_info(self):
        post_count = self.account_info.get('post_count')
        balance = self.account_info.get('balance')
        created = self.parse_date(
            self.account_info.get('created'))
        sbd_balance = self.account_info.get('sbd_balance')
        vesting_shares = self.parse_vests(
            self.account_info.get('vesting_shares'))
        delegated_vesting_shares = self.parse_vests(
            self.account_info.get('delegated_vesting_shares'))

        converter = Converter()
        sp = converter.vests_to_sp(vesting_shares)
        delegated_sp = converter.vests_to_sp(delegated_vesting_shares)

        voting_power = self.parse_voting_power(
            self.account_info.get('voting_power'))
        reputation = self.parse_reputation(
            self.account_info.get('reputation'))
        last_post_date = self.parse_date(
            self.account_info.get('last_root_post'))
        time_since_last_post = datetime.utcnow().replace(
            tzinfo=timezone.utc) - last_post_date
        days_since_last_post = time_since_last_post.days

        #         print(_timedelta)

        return {
            'account_name': self.account_name,
            'balance': balance,
            'created':
            self.utc_2_local_date(created).strftime(
                '%Y-%m-%d-%a %H:%M:%S'),
            'sbd_balance': sbd_balance,
            'vesting_shares': vesting_shares,
            'delegated_vesting_shares': delegated_vesting_shares,
            'sp': sp,
            'delegated_sp': delegated_sp,
            'post_count': post_count,
            'voting_power': voting_power,
            'reputation': reputation,
            'last_post_date':
            self.utc_2_local_date(last_post_date).strftime(
                '%Y-%m-%d-%a %H:%M:%S'),
            'days_since_last_post': days_since_last_post
        }

    def get_post_histry(self):
        pass

    @staticmethod
    def utc_2_local_date(_utcdate):
        _timedelta = datetime.now() - datetime.utcnow()
        return _utcdate.astimezone(timezone(_timedelta))

    @staticmethod
    def parse_vests(_vests):
        return float(_vests.split()[0])

    @staticmethod
    def parse_voting_power(voting_power):
        return int(voting_power) / 100

    @staticmethod
    def parse_reputation(raw_reputation):
        return (math.log10(int(raw_reputation)) - 9) * 9 + 25

    @staticmethod
    def parse_date(_date):
        utc_date = parser.parse(_date).replace(tzinfo=timezone.utc)
        return utc_date

```

```python
yjcps = Steemains('yjcps')
pprint(yjcps.view_account_info)

```

```
{'account_name': 'yjcps',
 'balance': '0.437 STEEM',
 'created': '2018-01-04-Thu 13:25:18',
 'days_since_last_post': 0,
 'delegated_sp': 57.07436926020711,
 'delegated_vesting_shares': 115562.393455,
 'last_post_date': '2018-08-19-Sun 07:01:57',
 'post_count': 261,
 'reputation': 51.49449059489242,
 'sbd_balance': '4.499 SBD',
 'sp': 121.35204133768676,
 'vesting_shares': 245709.808613,
 'voting_power': 68.73}

```

```python
deanliu = Steemains('deanliu')
pprint(deanliu.view_account_info)
del deanliu

```

```
{'account_name': 'deanliu',
 'balance': '105.649 STEEM',
 'created': '2016-07-14-Thu 13:56:39',
 'days_since_last_post': 0,
 'delegated_sp': 3028.3048743103254,
 'delegated_vesting_shares': 6131616.465781,
 'last_post_date': '2018-08-19-Sun 11:23:24',
 'post_count': 9998,
 'reputation': 74.33099682076877,
 'sbd_balance': '189.594 SBD',
 'sp': 17333.498512941853,
 'vesting_shares': 35096322.630247,
 'voting_power': 55.17}

```

```python
dapeng = Steemains('dapeng')
pprint(dapeng.view_account_info)
del dapeng

```

```
{'account_name': 'dapeng',
 'balance': '229.299 STEEM',
 'created': '2016-10-14-Fri 19:03:39',
 'days_since_last_post': 2,
 'delegated_sp': 3909.296719340023,
 'delegated_vesting_shares': 7915420.822822,
 'last_post_date': '2018-08-16-Thu 20:57:21',
 'post_count': 5926,
 'reputation': 67.26589131476406,
 'sbd_balance': '30.081 SBD',
 'sp': 5186.613554620994,
 'vesting_shares': 10501691.705077,
 'voting_power': 69.86}

```

```python
for key,value in yjcps.view_account_info.items():
    print(key,':',value)

```

```
account_name : yjcps
balance : 0.437 STEEM
created : 2018-01-04-Thu 13:25:18
sbd_balance : 4.499 SBD
vesting_shares : 245709.808613
delegated_vesting_shares : 115562.393455
sp : 121.35205839875916
delegated_sp : 57.07437728438151
post_count : 261
voting_power : 68.73
reputation : 51.49449059489242
last_post_date : 2018-08-19-Sun 07:01:57
days_since_last_post : 0

```

```python
display_message = '''
Username: \t{account_name}
Reputation: \t{reputation}
Created:\t{created}
Last Post: \t{last_post_date} ({days_since_last_post} days ago)
===========================================
STEEM Balance:\t{balance}
SBD Balance:\t{sbd_balance}
SP: \t\t{sp}
Delegated SP:\t{delegated_sp}
Total Posts:\t{post_count}
Voting Power:\t{voting_power}%
'''.format(**yjcps.view_account_info)
print(display_message)

```

```
Username:   yjcps
Reputation:     51.49449059489242
Created:    2018-01-04-Thu 13:25:18
Last Post:  2018-08-19-Sun 07:01:57 (0 days ago)
===========================================
STEEM Balance:  0.437 STEEM
SBD Balance:    4.499 SBD
SP:         121.35205862624704
Delegated SP:   57.07437749836572
Total Posts:    261
Voting Power:   68.73%

```

```python
pprint(yjcps.account_info)

```

```
{'active': {'account_auths': [],
            'key_auths': [['STM52aJdPyehuxigiDfYngKBL8PSAcESNmYENdaVnwRuKoiP1M9eu',
                           1]],
            'weight_threshold': 1},
 'average_bandwidth': '56831815207',
 'average_market_bandwidth': 1279817315,
 'balance': '0.437 STEEM',
 'can_vote': True,
 'comment_count': 0,
 'created': '2018-01-04T05:25:18',
 'curation_rewards': 231,
 'delegated_vesting_shares': '115562.393455 VESTS',
 'guest_bloggers': [],
 'id': 556829,
 'json_metadata': '{"profile":{"name":"hacper","profile_image":"https://s.gravatar.com/avatar/6f1c379a4a2c2190a4aa0921836e98b1?s=80","location":"GuangDong, '
                  'China","about":"最好的选择莫过于投资自己，写作是其一！","website":"https://steemit.com/@yjcps","cover_image":"https://cdn.steemitimages.com/DQmTvZbcaXYAcpTAjyNLp4qa9ttMu2Eo5wHSSzTaQSK4tPj/%E5%9B%BE%E7%89%87.png","bitcoin":"123YPnYjJmANKtiBr4DTsGJcncxxu2q6y3","ethereum":"0x50be5c7e721b8e146167dccb35e3c94156e13f58"}}',
 'last_account_recovery': '1970-01-01T00:00:00',
 'last_account_update': '2018-08-10T02:42:45',
 'last_bandwidth_update': '2018-08-19T02:06:15',
 'last_market_bandwidth_update': '2018-08-16T01:42:00',
 'last_owner_update': '2018-01-04T14:49:21',
 'last_post': '2018-08-18T23:01:57',
 'last_root_post': '2018-08-18T23:01:57',
 'last_vote_time': '2018-08-19T02:06:15',
 'lifetime_bandwidth': '615056000000',
 'lifetime_market_bandwidth': '22160000000',
 'lifetime_vote_count': 0,
 'market_history': [],
 'memo_key': 'STM6spBZkmNSe9fccLvsms7bGjSRaE4gsRbTAxNLnEiFHyhP9ww1f',
 'mined': False,
 'name': 'yjcps',
 'next_vesting_withdrawal': '1969-12-31T23:59:59',
 'other_history': [],
 'owner': {'account_auths': [],
           'key_auths': [['STM5gJExNnQn5aLjpp8cn7nngfYsutN6NCJmKfn9dFgji6fZrHkwH',
                          1]],
           'weight_threshold': 1},
 'post_count': 261,
 'post_history': [],
 'posting': {'account_auths': [['busy.app', 1],
                               ['dlive.app', 1],
                               ['dtube.app', 1],
                               ['fundition.app', 1],
                               ['partiko-steemcon', 1],
                               ['smartsteem', 1],
                               ['steemauto', 1],
                               ['steemgg.app', 1],
                               ['steemhunt.com', 1]],
             'key_auths': [['STM8bbH2Sfq72PshhVeNpSRQAa4XeiDwTNq4LdY8hxFMZkhfSGZU9',
                            1]],
             'weight_threshold': 1},
 'posting_rewards': 54307,
 'proxied_vsf_votes': [0, 0, 0, 0],
 'proxy': '',
 'received_vesting_shares': '0.000000 VESTS',
 'recovery_account': 'skenan',
 'reputation': '878683129879',
 'reset_account': 'null',
 'reward_sbd_balance': '0.000 SBD',
 'reward_steem_balance': '0.000 STEEM',
 'reward_vesting_balance': '0.000000 VESTS',
 'reward_vesting_steem': '0.000 STEEM',
 'savings_balance': '0.000 STEEM',
 'savings_sbd_balance': '0.000 SBD',
 'savings_sbd_last_interest_payment': '1970-01-01T00:00:00',
 'savings_sbd_seconds': '0',
 'savings_sbd_seconds_last_update': '1970-01-01T00:00:00',
 'savings_withdraw_requests': 0,
 'sbd_balance': '4.499 SBD',
 'sbd_last_interest_payment': '2018-08-14T00:32:57',
 'sbd_seconds': '1907954703',
 'sbd_seconds_last_update': '2018-08-18T22:40:57',
 'tags_usage': [],
 'to_withdraw': 0,
 'transfer_history': [],
 'vesting_balance': '0.000 STEEM',
 'vesting_shares': '245709.808613 VESTS',
 'vesting_withdraw_rate': '0.000000 VESTS',
 'vote_history': [],
 'voting_power': 6873,
 'withdraw_routes': 0,
 'withdrawn': 0,
 'witness_votes': ['abit',
                   'bobdos',
                   'ety001',
                   'justyy',
                   'skenan',
                   'therealwolf'],
 'witnesses_voted_for': 6}

```

```python
del yjcps

```

## 补充

### 解析日期字符串

[python-dateutil](http://labix.org/python-dateutil#head-a23e8ae0a661d77b89dfb3476f85b26f0b30349c)

```python
# ISO format
dt = parser.parse("2018-01-04T14:49:21")
print(dt)
print(type(dt))

```

```
2018-01-04 14:49:21
<class 'datetime.datetime'>

```

```python
# 设置时区
TZOFFSETS = {"BRST": -10800}
dt = parser.parse("Thu Sep 25 10:36:28 BRST 2003",  tzinfos=TZOFFSETS)
print(dt)
print(type(dt))

```

```
2003-09-25 10:36:28-03:00
<class 'datetime.datetime'>

```

```python
dt = parser.parse("2003-09-25T10:49:41.5-03:00")
print(dt)

```

```
2003-09-25 10:49:41.500000-03:00

```

```python
dt = parser.parse("20030925T104941")
print(dt)

```

```
2003-09-25 10:49:41

```

```python
dt = parser.parse("2003.Sep.25")
print(dt)

```

```
2003-09-25 00:00:00

```

```python
dt = parser.parse("Tuesday, April 12, 1952 AD 3:30:42pm PST", ignoretz=True)
print(dt)

```

```
1952-04-12 15:30:42

```

### 时区转换

```python
# 当前本地时间
print(datetime.now())
# 当前UTC时间
print(datetime.utcnow())

print()

# 时差
dt = datetime.now() - datetime.utcnow()
print(dt)

print()

# 设置时区为UTC+0:00
utc_dt = datetime.utcnow().replace(tzinfo=timezone.utc)
print(utc_dt)

# 设置时区为UTC+8:00
_bj_dt = utc_dt.astimezone(timezone(dt))
print(_bj_dt)

# 设置时区为UTC+5:00
bj_dt = utc_dt.astimezone(timezone(timedelta(hours=5)))
print(bj_dt)



```

```
2018-08-19 11:35:33.051849
2018-08-19 03:35:33.051849

8:00:00

2018-08-19 03:35:33.052827+00:00
2018-08-19 11:35:33.052827+08:00
2018-08-19 08:35:33.052827+05:00

```

### 时间格式转换

```python
dt = parser.parse("2018-08-18T23:01:57")
print(dt)

```

```
2018-08-18 23:01:57

```

```python
# datetime 2 timestamp
timestamp = dt.timestamp()
print(timestamp)

print()

# timestamp 2 datetime
print(datetime.fromtimestamp(timestamp))
print(datetime.utcfromtimestamp(timestamp))

```

```
1534604517.0

2018-08-18 23:01:57
2018-08-18 15:01:57

```

```python
# datetime 2 string
str_time = dt.strftime('%Y/%m/%d %H:%M:%S')
print(str_time)
print(type(str_time))

```

```
2018/08/18 23:01:57
<class 'str'>

```

- - - - - -

## \[DA series - Learn Python with Steem\]

- [\[DA series - Learn Python with Steem #01\] 安裝Python、文字編輯器與哈囉！](https://busy.org/@deanliu/da-series-learn-python-with-steem-01-python)
- [\[DA series - Learn Python with Steem #02\] 變數與資料型態](https://busy.org/@deanliu/da-series-learn-python-with-steem-02)
- [\[DA series - Learn Python with Steem #03\] 邏輯判斷](https://busy.org/@deanliu/da-series-learn-python-with-steem-03)
- [\[DA series - Learn Python with Steem #04\] 迴圈](https://busy.org/@deanliu/da-series-learn-python-with-steem-04)
- [\[DA series – Learn Python with Steem #05\] 基本資料結構](https://busy.org/@yjcps/learnpythonwithsteem05-n09ajzsh91)
- [\[DA series - Learn Python with Steem #06\] 函式](https://busy.org/@deanliu/da-series-learn-python-with-steem-06)
- [\[DA series - Learn Python with Steem #07\] 類別](https://busy.org/@deanliu/da-series-learn-python-with-steem-07)
- [\[DA series - Learn Python with Steem #08\] 函式庫(Modules)的安裝與使用，準備好玩Steem！](https://busy.org/@deanliu/da-series-learn-python-with-steem-08-modules-steem)
- [\[DA series - Learn Python with Steem #09\] Steem 小工具DIY #1 - 我的Steem小偵探](https://busy.org/@deanliu/da-series-learn-python-with-steem-09-steem-diy-1-steem)
- [\[DA series - Learn Python with Steem #10\] Steem 小工具DIY #2 - 我的文章列表（一）](https://busy.org/@deanliu/da-series-learn-python-with-steem-10-steem-diy-2)

## 我的笔记：

- [Learn Python with Steem #01 笔记](https://busy.org/@yjcps/learn-python-with-steem-01)
- [Learn Python with Steem #02 笔记](https://busy.org/@yjcps/learnpythonwithsteem02-2ilwe1ti59)
- [Learn Python with Steem #03 笔记](https://busy.org/@yjcps/learnpythonwithsteem03-l5w15fszh9)
- [Learn Python with Steem #04 笔记](https://busy.org/@yjcps/learnpythonwithsteem04-0d8jly8ypt)
- [Learn Python with Steem #05 笔记](https://busy.org/@yjcps/learnpythonwithsteem05-n09ajzsh91)
- [Learn Python with Steem #06 笔记](https://busy.org/@yjcps/learnpythonwithsteem06-c0tgbg3puu)
- [Learn Python with Steem #07 笔记](https://busy.org/@yjcps/learnpythonwithsteem07-2tx2pvwskh)
- [Learn Python with Steem #08 笔记](https://busy.org/@yjcps/learnpythonwithsteem08-5w4cklx91x)