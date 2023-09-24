---
title: "git-crypt 使用方法"
date: 2023-01-05T07:42:17+08:00
lastmod: 2023-03-04T19:20:51+08:00
tags:
   - git
   - git-crypt
categories:
    - 笔记
draft: false
---

使用git-crypt，可以对git仓库中的敏感文件进行加密，下面介绍使用方法。

## 安装

Windows 环境下，在官网 https://github.com/AGWA/git-crypt/releases 下载最新版本软件，然后将其所在路径添加到系统环境变量PATH

![](https://jsd.cdn.zzko.cn/gh/hacperme/picx_hosting@master/20210507/image-20230103232738007.32llslzam8c0.png)

打开git bash，输入 git crypt --version 验证安装情况:

```bash
$ git crypt --version
git-crypt 0.7.0
```

## 使用方法

1. 创建一个新文件目录，初始化git 仓库

   ```bash
   hacper@LAPTOP-0RHP1TGD MINGW64 /d/workspace
   $ mkdir git_crypt_test
   
   hacper@LAPTOP-0RHP1TGD MINGW64 /d/workspace
   $ cd git_crypt_test
   
   hacper@LAPTOP-0RHP1TGD MINGW64 /d/workspace/git_crypt_test
   $ git init
   Initialized empty Git repository in D:/workspace/git_crypt_test/.git/
   
   hacper@LAPTOP-0RHP1TGD MINGW64 /d/workspace/git_crypt_test (master)
   $
   ```

2. 生成 gpg 密钥（可选步骤）

   ```bash
   hacper@LAPTOP-0RHP1TGD MINGW64 /d/workspace/git_crypt_test (master)
   $ gpg --full-generate-key
   gpg (GnuPG) 2.2.29-unknown; Copyright (C) 2021 Free Software Foundation, Inc.
   This is free software: you are free to change and redistribute it.
   There is NO WARRANTY, to the extent permitted by law.
   
   Please select what kind of key you want:
      (1) RSA and RSA (default)
      (2) DSA and Elgamal
      (3) DSA (sign only)
      (4) RSA (sign only)
     (14) Existing key from card
   Your selection? 1
   
   ...
   
   hacper@LAPTOP-0RHP1TGD MINGW64 /d/workspace/git_crypt_test (master)
   $ gpg --list-keys
   gpg: checking the trustdb
   gpg: marginals needed: 3  completes needed: 1  trust model: pgp
   gpg: depth: 0  valid:   1  signed:   0  trust: 0-, 0q, 0n, 0m, 0f, 1u
   /c/Users/hacper/.gnupg/pubring.kbx
   ----------------------------------
   pub   dsa1024 2009-04-16 [SC]
         4340D13570EF945E83810964E8AD3F819AB10E78
   uid           [ unknown] The Android Open Source Project <initial-contribution@android.com>
   sub   elg2048 2009-04-16 [E]
   
   pub   rsa3072 2023-01-03 [SC]
         D8B7C1CF1522498D3B6585608F5E3E2BC9BE60E8
   uid           [ultimate] hacper <git@hacperme.com>
   sub   rsa3072 2023-01-03 [E]
   
   ```

   导出gpg密钥

   ```bash
   hacper@LAPTOP-0RHP1TGD MINGW64 /d/workspace/git_crypt_test (master)
   $ gpg --armor --output hacper_gpg.pub --export hacper
   
   hacper@LAPTOP-0RHP1TGD MINGW64 /d/workspace/git_crypt_test (master)
   $ ls
   hacper_gpg.pub  readme.md
   
   hacper@LAPTOP-0RHP1TGD MINGW64 /d/workspace/git_crypt_test (master)
   $ gpg --armor --output hacper_gpg.key --export-secret-keys hacper
   
   ```

   

   删除密钥

   ```bash
   hacper@LAPTOP-0RHP1TGD MINGW64 /d/workspace/git_crypt_test (master)
   $ gpg --delete-secret-keys hacper
   gpg (GnuPG) 2.2.29-unknown; Copyright (C) 2021 Free Software Foundation, Inc.
   This is free software: you are free to change and redistribute it.
   There is NO WARRANTY, to the extent permitted by law.
   
   
   sec  rsa3072/8F5E3E2BC9BE60E8 2023-01-03 hacper <git@hacperme.com>
   
   Delete this key from the keyring? (y/N) y
   This is a secret key! - really delete? (y/N) y
   
   hacper@LAPTOP-0RHP1TGD MINGW64 /d/workspace/git_crypt_test (master)
   $ gpg --delete-keys hacper
   gpg (GnuPG) 2.2.29-unknown; Copyright (C) 2021 Free Software Foundation, Inc.
   This is free software: you are free to change and redistribute it.
   There is NO WARRANTY, to the extent permitted by law.
   
   
   pub  rsa3072/8F5E3E2BC9BE60E8 2023-01-03 hacper <git@hacperme.com>
   
   Delete this key from the keyring? (y/N) y
   
   hacper@LAPTOP-0RHP1TGD MINGW64 /d/workspace/git_crypt_test (master)
   $
   
   hacper@LAPTOP-0RHP1TGD MINGW64 /d/workspace/git_crypt_test (master)
   $ gpg --list-keys
   gpg: checking the trustdb
   gpg: no ultimately trusted keys found
   /c/Users/hacper/.gnupg/pubring.kbx
   ----------------------------------
   pub   dsa1024 2009-04-16 [SC]
         4340D13570EF945E83810964E8AD3F819AB10E78
   uid           [ unknown] The Android Open Source Project <initial-contribution@android.com>
   sub   elg2048 2009-04-16 [E]
   
   
   hacper@LAPTOP-0RHP1TGD MINGW64 /d/workspace/git_crypt_test (master)
   
   ```

   

   导入密钥

   ```bash
   hacper@LAPTOP-0RHP1TGD MINGW64 /d/workspace/git_crypt_test (master)
   $ gpg --import hacper_gpg.pub
   gpg: key 8F5E3E2BC9BE60E8: public key "hacper <git@hacperme.com>" imported
   gpg: Total number processed: 1
   gpg:               imported: 1
   
   hacper@LAPTOP-0RHP1TGD MINGW64 /d/workspace/git_crypt_test (master)
   $ gpg --allow-secret-key-import --import hacper_gpg.key
   gpg: key 8F5E3E2BC9BE60E8: "hacper <git@hacperme.com>" not changed
   gpg: key 8F5E3E2BC9BE60E8: secret key imported
   gpg: Total number processed: 1
   gpg:              unchanged: 1
   gpg:       secret keys read: 1
   gpg:   secret keys imported: 1
   
   hacper@LAPTOP-0RHP1TGD MINGW64 /d/workspace/git_crypt_test (master)
   $ gpg --list-keys
   /c/Users/hacper/.gnupg/pubring.kbx
   ----------------------------------
   pub   dsa1024 2009-04-16 [SC]
         4340D13570EF945E83810964E8AD3F819AB10E78
   uid           [ unknown] The Android Open Source Project <initial-contribution@android.com>
   sub   elg2048 2009-04-16 [E]
   
   pub   rsa3072 2023-01-03 [SC]
         D8B7C1CF1522498D3B6585608F5E3E2BC9BE60E8
   uid           [ unknown] hacper <git@hacperme.com>
   sub   rsa3072 2023-01-03 [E]
   
   ```

   

3. 初始化git crypt，配置密钥

   ```bash
   hacper@LAPTOP-0RHP1TGD MINGW64 /d/workspace/git_crypt_test (master)
   $ git-crypt init
   Generating key...
   
   hacper@LAPTOP-0RHP1TGD MINGW64 /d/workspace/git_crypt_test (master)
   $ git-crypt add-gpg-user hacper
   [master (root-commit) ce4bba9] Add 1 git-crypt collaborator
    2 files changed, 4 insertions(+)
    create mode 100644 .git-crypt/.gitattributes
    create mode 100644 .git-crypt/keys/default/0/ABF942D38B623FCB98B98E722B3BBE58C106357F.gpg
   
   hacper@LAPTOP-0RHP1TGD MINGW64 /d/workspace/git_crypt_test (master)
   $
   
   ```

4. 创建 .gitattributes 文件，配置需要加密的文件, 示例配置仅加密 *.md 后缀的文件

   ```bash
   hacper@LAPTOP-0RHP1TGD MINGW64 /d/workspace/git_crypt_test (master)
   $ echo '*.md filter=git-crypt diff=git-crypt' > .gitattributes
   
   hacper@LAPTOP-0RHP1TGD MINGW64 /d/workspace/git_crypt_test (master)
   $ ls -al
   total 21
   drwxr-xr-x 1 hacper 197609  0 Jan  3 23:50 ./
   drwxr-xr-x 1 hacper 197609  0 Jan  3 23:33 ../
   drwxr-xr-x 1 hacper 197609  0 Jan  3 23:47 .git/
   drwxr-xr-x 1 hacper 197609  0 Jan  3 23:45 .git-crypt/
   -rw-r--r-- 1 hacper 197609 37 Jan  3 23:51 .gitattributes
   
   hacper@LAPTOP-0RHP1TGD MINGW64 /d/workspace/git_crypt_test (master)
   $ git add .gitattributes
   
   hacper@LAPTOP-0RHP1TGD MINGW64 /d/workspace/git_crypt_test (master)
   $ git commit -m "add .gitattributes"
   [master d6157c0] add .gitattributes
    1 file changed, 1 insertion(+)
    create mode 100644 .gitattributes
   
   hacper@LAPTOP-0RHP1TGD MINGW64 /d/workspace/git_crypt_test (master)
   $
   
   ```

5. 创建待加密的md文件测试

   ```bash
   hacper@LAPTOP-0RHP1TGD MINGW64 /d/workspace/git_crypt_test (master)
   $ echo "# hello " > readme.md
   
   hacper@LAPTOP-0RHP1TGD MINGW64 /d/workspace/git_crypt_test (master)
   $ ls
   readme.md
   
   hacper@LAPTOP-0RHP1TGD MINGW64 /d/workspace/git_crypt_test (master)
   $ cat readme.md
   # hello
   
   hacper@LAPTOP-0RHP1TGD MINGW64 /d/workspace/git_crypt_test (master)
   $ git add readme.md
   
   hacper@LAPTOP-0RHP1TGD MINGW64 /d/workspace/git_crypt_test (master)
   $ git commit -m "add readme.md"
   [master 09338d7] add readme.md
    1 file changed, 0 insertions(+), 0 deletions(-)
    create mode 100644 readme.md
   
   hacper@LAPTOP-0RHP1TGD MINGW64 /d/workspace/git_crypt_test (master)
   $ git-crypt lock
   
   hacper@LAPTOP-0RHP1TGD MINGW64 /d/workspace/git_crypt_test (master)
   $ cat readme.md
   GITCRYPT▒iKO▒:▒gx▒*▒▒▒&rf▒
   
   hacper@LAPTOP-0RHP1TGD MINGW64 /d/workspace/git_crypt_test (master)
   $ git-crypt status
   not encrypted: .git-crypt/.gitattributes
   not encrypted: .git-crypt/keys/default/0/ABF942D38B623FCB98B98E722B3BBE58C106357F.gpg
   not encrypted: .gitattributes
       encrypted: readme.md
   
   hacper@LAPTOP-0RHP1TGD MINGW64 /d/workspace/git_crypt_test (master)
   $ git-crypt lock
   
   hacper@LAPTOP-0RHP1TGD MINGW64 /d/workspace/git_crypt_test (master)
   $  cat readme.md
   GITCRYPT▒BN(▒▒▒JE▒&U^\x▒▒
   
   hacper@LAPTOP-0RHP1TGD MINGW64 /d/workspace/git_crypt_test (master)
   $ git-crypt unlock
   
   hacper@LAPTOP-0RHP1TGD MINGW64 /d/workspace/git_crypt_test (master)
   $  cat readme.md
   # hello
   
   
   ```

6. 导出加密密钥

   ```bash
   git-crypt export-key ./path/.s_key
   ```

7. 使用导出的密钥解密

   ```bash
   git-crypt unlock ./path/.s_key
   ```

8. 推送到远程仓库验证

   ```bash
   hacper@LAPTOP-0RHP1TGD MINGW64 /d/workspace/git_crypt_test (master)
   $ git remote add origin git@github.com:hacperme/git_crypt_test.git
   
   hacper@LAPTOP-0RHP1TGD MINGW64 /d/workspace/git_crypt_test (master)
   $ git push -u origin master
   Enumerating objects: 14, done.
   Counting objects: 100% (14/14), done.
   Delta compression using up to 8 threads
   Compressing objects: 100% (9/9), done.
   Writing objects: 100% (14/14), 1.68 KiB | 861.00 KiB/s, done.
   Total 14 (delta 1), reused 0 (delta 0), pack-reused 0
   remote: Resolving deltas: 100% (1/1), done.
   To github.com:hacperme/git_crypt_test.git
    * [new branch]      master -> master
   branch 'master' set up to track 'origin/master'.
   
   ```

   在仓库托管平台看不到明文内容，符合预期。

![](https://jsd.cdn.zzko.cn/gh/hacperme/picx_hosting@master/20210507/image-20230104010811220.2jhfuv80z1y0.png)

