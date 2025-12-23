# Linux文件下载以及进行压缩和解压


<!--more-->

**在Linux系统使用中，我们经常需要下载文件、对文件进行压缩和解压操作。**

**本文将介绍一些常用的Linux命令，帮助你轻松完成这些操作。**

## 1.文件下载

### 1.1 wget命令

**wget命令是Linux中最常用的下载命令之一。它支持HTTP、HTTPS、FTP等多种协议，并可以设置断点续传等功能。**

基本用法:

wget [选项] URL地址
常用选项:

**-c: 断点重连，继续下载未完成的文件。**
**-O: 指定下载文件的保存名称。**
**-P: 指定下载文件的保存路径。**
**-q: 静默模式，不显示下载进度。**

```shell
 
# 下载文件并保存为默认名称
wget https://www.example.com/file.txt
 
# 下载文件并保存为默认名称，-c为断点重连，如果断网后，下次会接续
wget -c https://www.example.com/file.txt
 
# 下载文件并指定保存名称
wget -O myfile.txt https://www.example.com/file.txt
 
# 下载文件并指定保存路径
wget -P /tmp https://www.example.com/file.txt
 
# 静默模式下载文件
wget -q https://www.example.com/file.txt
wget还有许多其他的选项，可以通过man wget或wget help查看。

```

### 1.2 curl命令

**curl命令也是一个常用的下载命令，它支持的功能与wget类似。**

**基本用法:**

**curl [选项] URL地址**
**常用选项:**

**-o: 指定下载文件的保存名称。**
**-L: 跟踪重定向。**
**-s: 静默模式，不显示下载进度。**

```shell
# 下载文件并保存为默认名称
curl https://www.example.com/file.txt
 
# 下载文件并指定保存名称
curl -o myfile.txt https://www.example.com/file.txt
 
# 跟踪重定向并下载文件
curl -L https://www.example.com/redirect.html
 
# 静默模式下载文件
curl -s https://www.example.com/file.txt
curl还有许多其他的选项，可以通过man curl 或则 curl help查看

```

## 2.文件压缩

### 2.1 gzip命令

**gzip命令是Linux中最常用的压缩命令之一，它可以对文件进行 gzip 压缩格式的压缩。**

**基本用法:**

**gzip [选项] 文件名**
**常用选项:**

**-c: 压缩后不删除源文件。**
**-d: 解压缩文件。**
**-f: 强制压缩文件，即使文件已存在。**
**-9: 使用最高压缩级别。**

```shell
# 压缩文件
gzip file.txt

# 解压缩文件
gzip -d file.txt.gz

# 强制压缩文件
gzip -f file.txt

# 使用最高压缩级别压缩文件
gzip -9 file.txt
```

### 2.2 bzip2命令

**bzip2命令也是一个常用的压缩命令，它可以对文件进行 bzip2 压缩格式的压缩。**

**基本用法:**

**bzip2 [选项] 文件名**
**常用选项:**

**-c: 压缩后不删除源文件。**
**-d: 解压缩文件。**
**-f: 强制压缩文件，即使文件已存在。**
**-9: 使用最高压缩级别。**

```shell
# 压缩文件
bzip2 file.txt

# 解压缩文件
bzip2 -d file.txt.bz2

# 强制压缩文件
bzip2 -f file.txt

# 使用最高压缩级别压缩文件
bzip2 -9 file.txt
```

## 3.文件解压

### 3.1 tar命令

**tar命令是Linux中常用的解压命令，它可以解压 tar 格式的压缩包。**

**基本用法:**

**tar [选项] 文件名**
**常用选项:**

**-c: 创建压缩包。**
**-x: 解压缩文件。**
**-f: 指定压缩包文件名。**
**-v: 显示详细信息。**

```
# 解压缩文件
tar -xvf file.tar.gz

# 解压缩文件并显示详细信息
tar -xvzf file.tar.gz

# 创建压缩包
tar -cf myfile.tar file1 file2
```

### 3.2 unzip命令

**unzip命令是Linux中常用的解压命令，它可以解压 zip 格式的压缩包。**

**基本用法:**

**unzip [选项] 文件名**
**常用选项:**

**-d: 指定解压文件的保存路径。**
**-l: 查看压缩包内容。**
**-q: 静默模式，不显示解压进度。**

```shell
# 解压缩文件
unzip file.zip
```

本文介绍了Linux中三个常用的命令：wget、gzip和tar，分别用于文件的下载、压缩和解压。这些命令都有很多的选项，可以根据不同的需求进行调整。通过掌握这些命令，我们可以更方便地管理和处理文件。希望本文对你有所帮助。



---

> 作者: <no value>  
> URL: https://jiang1bo.github.io/posts/68ced8b/  

