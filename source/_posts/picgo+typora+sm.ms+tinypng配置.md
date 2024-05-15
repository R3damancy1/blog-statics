---
title: picgo+typora+sm.ms+tinypng配置
date: 2024-05-14 03:01:11
excerpt: picgo+typora+sm.ms+tinypng配置
categories: 杂谈
---



# picgo+typora+sm.ms+tinypng配置



## 引言

由于最近感觉github图床访问速度太慢，于是换成了sm.ms感觉会好一些，但是sm.ms

图床有大小限制，于是弄了个自动压缩的插件，下面是我在配置的过程中踩到的一些坑，记录了下来



## typora配置

在偏好设置里面配置如下

![image-20240514023823816](https://s2.loli.net/2024/05/14/Uigbjv9mdVNo4Yh.png)



## picgo配置

**下载链接**：https://github.com/PicGo

在PicGo中配置服务API Token，如果你是SM.MS服务就配置SMMS的token，如果是阿里云OSS的服务就配置阿里云的token（token就是上面第2个文章标题下生成的API Token）

![image-20240514024315478](https://s2.loli.net/2024/05/14/5yatfZgPALFED9l.png)

## sm.ms配置

SMMS图床分海外和国内，如果海外访问不了，可以通过国内进行注册申请

海外网址：[https://sm.ms/](https://link.zhihu.com/?target=https%3A//sm.ms/)

国内网址：https://smms.app/

![image-20240514024117130](https://s2.loli.net/2024/05/14/2QVkd9sMtrFAWEo.png)







点击Sign Up进行SMMS的账号注册

![image-20240514024152185](https://s2.loli.net/2024/05/14/GZwVQWNhRPa8Eez.png)





### 获取Token密钥（后面图片上传需要用）

登录SMMS系统，找到用户信息，选择API Token，点击Generate Secret Token生成token

![image-20240514024251486](https://s2.loli.net/2024/05/14/1ZeRoa87Ebw6Fsp.png)



## tinypng配置

这里要在picgo下载一个插件tinypng（**这里划重点， 后面踩了很多坑，划重点！**）

![image-20240514024652599](https://s2.loli.net/2024/05/14/QXx4HTqvRYDPigZ.png)

![image-20240514024935584](https://s2.loli.net/2024/05/14/vnDqBOGEuwkRiPJ.png)



将api填入设置中就行





## 踩坑！

### 

### picgo软件可能会打不开

```
sudo xattr -d com.apple.quarantine "/Applications/PicGo.app"
```

运行即可打开



```
npm install picgo -g
```

```


安装 picgo add compress

选择使用 picgo use transformer

参数配置 picgo config plugin compress
```

compress 选择压缩工具 默认选项

- [tinypng](https://tinypng.com/) 无损压缩，需要上传到 tinypng
- [imagemin](https://github.com/imagemin/imagemin) 压缩过程不需要经过网络，但是图片会有损耗
- image2webp 本地有损压缩，支持 GIF 格式有损压缩 注意：有些图床（比如 sm.ms）不支持 webp 图片格式，会上传失败

![image-20240514025424708](https://s2.loli.net/2024/05/14/38yNpR7JSKgYd46.png)



### **reason: certificate has expired 错误**

![image-20240514025542618](https://s2.loli.net/2024/05/14/1N6hdDuxUIzA8ik.png)



解决方法

```
1、取消ssl验证：
 
npm config set strict-ssl false
 
这个方法一般就可以解决了。
 
 
2、更换npm镜像源：
 
npm config set registry http://registry.cnpmjs.org
npm config set registry http://registry.npm.taobao.org
 
```





### 安装gui插件

```
npm install picgo -g
npm install ./picgo-plugin-<your-plugin-name>
```

electron版的PicGo配置文件的路径在不同的系统里是不同的：

- Windows: `%APPDATA%\picgo\data.json`
- Linux: `$XDG_CONFIG_HOME/picgo/data.json` or `~/.config/picgo/data.json`
- macOS: `~/Library/Application\ Support/picgo/data.json`

举例，在windows里你可以在：

`C:\Users\你的用户名\AppData\Roaming\picgo\data.json`找到它。

在linux里你可以在：

`~/.config/picgo/data.json`里找到它。

macOS同理。

此时你的插件目录比如在 `/usr/home/picgo-plugin-<your-plugin-name>`里，

在PicGo默认配置文件所在的目录下，输入：

```
npm install /usr/home/picgo-plugin-<your-plugin-name>
```



### 插件下载失败



**tingpng**地址：https://github.com/liujinpen/picgo-plugin-compress-tinypng



开梯子没用，放弃直接安装的方式，采用npm方式安装

我用的Mac，picgo路径如下： /Users/cunyu/Library/Application Support/picgo（自己参照）

解决方法： 首先，cd到这个路径下 然后Mac要安装一些插件，命令如下：

```
brew install libtool automake autoconf nasm
```

​    

这个过程可能要很久，建议开梯子或有homebrew源

**下面正式开始安装**

第一步：清理缓存

```
npm cache clean --force
```

​    

第二步：删除文件夹

把整个node_modules都删除

第三步：install一下

```
npm install picgo-plugin-compress --save --ignore-scripts --registry=https://registry.npm.taobao.org
```

​      

第四步：

```
npm install --registry=https://registry.npm.taobao.org
```





在命令行找不到目录的记得对中间的空格用反斜杠转义一下。或者打印号

```
cd "Application Support"
cd Application/Support
```

