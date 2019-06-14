---
layout:     post
title:      运行ss服务报错
subtitle:   undefined symbol,EVP_CIPHER_CTX_cleanup
date:       2019-04-30
author:     WJ
header-img: img/post-bg-android.jpg
catalog: true
tags:
    - shadowsocks
---

## 解决:shadowsocks2.8.2启动报undefined symbol: EVP_CIPHER_CTX_cleanup错误

##### 错误如下
```py
INFO: loading config from ss.json 
2018-04-14 12:32:13 INFO loading libcrypto from libcrypto.so.1.1 
Traceback (most recent call last): 
File “/usr/local/bin/sslocal”, line 11, in 
sys.exit(main()) 
File “/usr/local/lib/python2.7/dist-packages/shadowsocks/local.py”, line 39, in main 
config = shell.get_config(True) 
File “/usr/local/lib/python2.7/dist-packages/shadowsocks/shell.py”, line 262, in get_config 
check_config(config, is_local) 
File “/usr/local/lib/python2.7/dist-packages/shadowsocks/shell.py”, line 124, in check_config 
encrypt.try_cipher(config[‘password’], config[‘method’]) 
File “/usr/local/lib/python2.7/dist-packages/shadowsocks/encrypt.py”, line 44, in try_cipher 
Encryptor(key, method) 
File “/usr/local/lib/python2.7/dist-packages/shadowsocks/encrypt.py”, line 83, in init 
random_string(self._method_info[1])) 
File “/usr/local/lib/python2.7/dist-packages/shadowsocks/encrypt.py”, line 109, in get_cipher 
return m[2](method, key, iv, op) 
File “/usr/local/lib/python2.7/dist-packages/shadowsocks/crypto/openssl.py”, line 76, in init 
load_openssl() 
File “/usr/local/lib/python2.7/dist-packages/shadowsocks/crypto/openssl.py”, line 52, in load_openssl 
libcrypto.EVP_CIPHER_CTX_cleanup.argtypes = (c_void_p,) 
File “/usr/lib/python2.7/ctypes/init.py”, line 375, in getattr 
func = self.getitem(name) 
File “/usr/lib/python2.7/ctypes/init.py”, line 380, in getitem 
func = self._FuncPtr((name_or_ordinal, self)) 
AttributeError: /usr/lib/x86_64-Linux-gnu/libcrypto.so.1.1: undefined symbol: EVP_CIPHER_CTX_cleanup
```

**这个问题是由于在openssl1.1.0版本中，废弃了EVP_CIPHER_CTX_cleanup函数，如官网中所说：**
```shell
EVP_CIPHER_CTX was made opaque in OpenSSL 1.1.0. As a result, EVP_CIPHER_CTX_reset() appeared and EVP_CIPHER_CTX_cleanup() disappeared. 
```

#### 解决方法:
1. 用vim打开文件：vim /usr/local/lib/python2.7/dist-packages/shadowsocks/crypto/openssl.py (该路径请根据自己的系统情况自行修改，如果不知道该文件在哪里的话，可以使用find命令查找文件位置)

2. 跳转到52行（shadowsocks2.8.2版本，其他版本搜索一下cleanup）

3. 进入编辑模式

4. 将第52行`libcrypto.EVP_CIPHER_CTX_cleanup.argtypes = (c_void_p,)`
改为`libcrypto.EVP_CIPHER_CTX_reset.argtypes = (c_void_p,)`

5. 再次搜索cleanup（全文件共2处，此处位于111行），将`libcrypto.EVP_CIPHER_CTX_cleanup(self._ctx)`
改为`libcrypto.EVP_CIPHER_CTX_reset(self._ctx)`

6. 保存并退出

7. 启动shadowsocks服务：service shadowsocks start 或 sslocal -c ss配置文件目录
