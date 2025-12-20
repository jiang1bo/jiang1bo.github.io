# Linux可执行文件添加到PATH环境变量的方法


## 输入命令的时候提示：command not found

**linux命令行下面执行某个命令的时候，首先保证该命令是否存在，若存在，但输入命令的时候若仍提示：command not found**

**这个时候就的查看PATH环境变量的设置了，当前命令是否存在于PATH环境变量中**

<!--more-->

```bash
#查看PATH：
echo $PATH
#举例说，命令 composr 在/usr/loca/bin
```

**但执行的时候提示：**

**-bash: composr: command not found**
**这个时候，通过echo $PATH，，发现composer并未在PATH环境变量中有设置，这个时候就需要把composer所在路径添加到PATH中**

## 修改PATH环境变量

所以需要修改PATH环境变量，具体如下：

```shell
#方法一：
#配置完后可以通过echo $PATH查看配置结果。
#生效方法：立即生效
#有效期限：临时改变，只能在当前的终端窗口中有效，当前窗口关闭后就会恢#复原有的path配置
#用户局限：仅对当前用户
export PATH=/usr/local/bin:$PATH

#方法二：
#通过修改.bashrc文件:
vim ~/.bashrc 
#在最后一行添上：
export PATH=/usr/local/bin:$PATH

#生效方法：（有以下两种）
#1、关闭当前终端窗口，重新打开一个新终端窗口就能生效
#2、输入“source ~/.bashrc”命令，立即生效
source ~/.bashrc

#有效期限：永久有效
#用户局限：仅对当前用户

#方法三：
 
#通过修改profile文件:
vim /etc/profile
export PATH=/usr/local/bin:$PATH
#生效方法：系统重启
#有效期限：永久有效
#用户局限：对所有用户

#方法四：
 
#通过修改environment文件:
vim /etc/environment
在PATH="/usr/local/sbin:/usr/sbin:/usr/bin:/sbin:/bin"中加入 
":/usr/local/bin"
#生效方法：系统重启
#有效期限：永久有效
#用户局限：对所有用户
```

## 注意事项

系统在环境变量中寻找程序路径时，是由前往后，由上往下的方式寻找的，找到第一个后九会直接使用它而不会再往后往下寻找了。
所以在将程序路径复制进入到path或者其他环境变量名中时，需要如果原先已经配置过这个程序的路径，那么系统会使用先找到的那个路径，所有为了保证系统使用的是我们最新配置的路径，一般我们直接把新配置的路径写在最上面或者最前面。



---

> 作者: <no value>  
> URL: https://jiang1bo.github.io/posts/d681069/  

