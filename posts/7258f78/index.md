# Linux升级openssl到最新版，yumdnf运行报错，重启系统后网络无法连接


<!--more-->

# yum/dnf运行报错，重启系统后网络无法连接

![linux升级openssl到最新版，yum/dnf运行报错，重启系统后网络无法连接，](https://jiangbos.top/wp-content/uploads/2024/10/Snipaste_2024-10-23_08-18-06-300x188.jpg)

**linux组件底层依赖openssl 1.0.0版本导致的程序在编译的时候采用的是1.0版本的openssl库。升级后，导致动态库不兼容，感觉应该是openssl版本太新了，而yum\dnf这些包管理器，要用到一些旧版本的依赖**

## 解决方法

**1.ssl降级重新编译安装旧版本**

**2.从本机或者其他机器上找到libcrypt.so.*和libssl.so.*文件，复制到问题机器的/lib64目录，在创建1.0版本的符号链接。将错误的软连接删除，恢复成为旧的链接**

```perl
[root@localhost ~]# ll /usr/lib64/libcrypto.so*
-rwxr-xr-x. 1 root root 3357304 Oct 23 18:09 /usr/lib64/libcrypto.so.1.1
lrwxrwxrwx. 1 root root 29 Oct 23 18:37 /usr/lib64/libcrypto.so.3 -> /usr/lib64/libcrypto.so.3.0.7
-rwxr-xr-x. 1 root root 4487176 Sep 19 03:33 /usr/lib64/libcrypto.so.3.0.7
[root@localhost ~]# ll /usr/lib64/libssl.so*
-rwxr-xr-x. 1 root root 689520 Oct 23 18:09 /usr/lib64/libssl.so.1.1
lrwxrwxrwx. 1 root root 26 Oct 23 18:38 /usr/lib64/libssl.so.3 -> /usr/lib64/libssl.so.3.0.7
-rwxr-xr-x. 1 root root 688352 Sep 19 03:33 /usr/lib64/libssl.so.3.0.7
 
#新建/usr/lib64/libcrypto.so.3，/usr/lib64/libcrypto.so.3两个连接，并将旧链接/usr/lib64/libcrypto.so.3.0.7，/usr/lib64/lib.so.3.0.7指向它们
[root@localhost ~]# ln -s /usr/lib64/libcrypto.so.3.0.7 /usr/lib64/libcrypto.so.3
 
[root@localhost ~]# ln -s /usr/lib64/libssl.so.3.0.7 /usr/lib64/libssl.so.3
```

<img src="https://jiangbos.top/wp-content/uploads/2024/10/Snipaste_2024-10-23_10-56-04-300x23.jpg" alt="linux升级openssl到最新版，yum/dnf运行报错，重启系统后网络无法连接，" style="zoom:200%;" />

[OpenSSL升级后导致libcrypt.so.10和libssl.so.10找不到的解决_libssl.so.10 找不到-CSDN博客](https://blog.csdn.net/yin138/article/details/110820145)

参考：

[ImportError: /usr/1ib64/1ibldap,so,2: undefined symbol: EVP-md2， version OPENSSL_1_1_0 – Justtosee – 博客园](https://www.cnblogs.com/justtosee/p/17303465.html)

[OpenSSL版本历史_openssl所有版本-CSDN博客](https://blog.csdn.net/qq237696047/article/details/119538645)

[历时3年，OpenSSL 3.0 正式发布！弃用所有低级别API – 知乎](https://zhuanlan.zhihu.com/p/412636297)

[undefined symbol: EVP_mdc2, version OPENSSL_1_1_0 · Issue #11670 · openssl/openssl](https://github.com/openssl/openssl/issues/11670#issuecomment-620745135)

[CentOS libssl.so.10及libcrypto.so.10缺失导致sudo、yum等命令报错-腾讯云开发者社区-腾讯云](https://cloud.tencent.com/developer/article/2405362)

[OpenSSL升级后导致libcrypt.so.10和libssl.so.10找不到的解决_libssl.so.10 找不到-CSDN博客](https://blog.csdn.net/yin138/article/details/110820145)

[linux系列8.2系统升级openssl导致的连锁问题 – K& – 博客园](https://www.cnblogs.com/determined-K/p/17195507.html)


---

> 作者: <no value>  
> URL: http://localhost:1313/posts/7258f78/  

