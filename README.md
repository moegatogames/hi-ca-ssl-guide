# Hi CA SSL证书 部署教程
2022 年 7 月 23 日，国产数字证书品牌 HiCA 正式上线，提供免费的半年 IP 或者通配符 DV SSL 证书，非常实用
官网：https://www1.hi.cn/

![image](https://user-images.githubusercontent.com/110012832/180933835-03363f07-4b61-4b25-9baf-abb681378870.png)

## Hi CA优势
- 通过ACME提供有效期为180天的证书（大多数厂商通过ACME提供的免费证书有效期为90天）
- 提供免费IPv4证书和IPv6证书
- 提供国内OCSP（除此之外的免费/低价证书中目前仅TrustAsia的旧版DigiCert证书和GlobalSign提供）
- 支持一次付款多次自动化续签
### 特别的：
Hi CA不提供GUI且暂不支持其它ACME客户端，请使用acme.sh。以下教程基于Linux
- 经费有限，后端可能经常出现500等错误
- 不支持.ru、.by、.su域名（DigiCert、Sectigo对俄罗斯、白俄罗斯禁运）。
- IPv6 、.onion有效期90天（CA限制）。
## 安装acme.sh
安装很简单，一个命令：

```curl  https://get.acme.sh | sh -s email=my@example.com```

普通用户和 root 用户都可以安装使用. 安装过程进行了以下几步：
1. 把 acme.sh 安装到你的 home 目录下：

```~/.acme.sh/```

并创建 一个 bash 的 alias, 方便你的使用：```alias acme.sh=~/.acme.sh/acme.sh```
自动为你创建 cronjob, 每天 0:00 自动检测所有的证书, 如果快过期了, 需要更新, 则会自动更新证书。
更高级的安装选项请参考: https://github.com/Neilpang/acme.sh/wiki/How-to-install
安装过程不会污染已有的系统任何功能和文件, 所有的修改都只在安装目录中: ```~/.acme.sh/```
如果上面官方下载地址失败 或者 太慢，可以选用国内的备用地址

```curl https://gitcode.net/cert/cn-acme.sh/-/raw/master/install.sh?inline=false | sh -s email=admin@example.com```

安装完成后，需要重新打开命令行（如果是 SSH，请重新连接，比如关闭窗口，重新使用密钥或密码登录），以使acme.sh相关命令生效。
对root用户，acme.sh默认安装在/root目录下

```cd ~/.acme.sh/```

## 申请单域名或IP证书
acme.sh 实现了 ACME 协议支持的所有验证协议. 一般有两种方式验证: http 和 dns 验证
http 方式需要在你的网站根目录下放置一个文件，来验证你的域名所有权,完成验证.。然后就可以生成证书了

```acme.sh  --issue  -d mydomain.com -d www.mydomain.com  --webroot  /home/wwwroot/mydomain.com/ --server https://acme.hi.cn/directory```

对于IP证书，申请证书的IP就是域名
### 简洁教程

```./acme.sh --issue -d 11.4.5.14 --webroot [你的网站目录] --server https://acme.hi.cn/directory```

```./acme.sh --issue -d example.com --webroot [你的网站目录] --server https://acme.hi.cn/directory```

需要开启80端口（ACME协议限制）
### 详细教程
签发 IP或单域名SSL证书 使用 HTTP 文件验证
首先需要启动一个 Web 服务器，要求是访问需要签发证书的 IP或域名 能够直接访问到该目录，例如你需要为11.4.5.14或114514.com签发证书，则通过http://11.4.5.14或http://114514.com需要可以访问到你在上述命令里填写的网站目录下的内容
可以使用各种面板安装环境，或者直接安装网页服务器如Nginx后绑定目录
<details><summary>申请IP证书额外内容</summary>
<p>
如果服务器上已经有多个站点，若未设置默认站点，直接访问IP会访问第一个或最后一个添加的站点。请注意，如果你设置了直接访问IP返回403等规则，需要暂时删除这些规则。如果不清楚直接访问IP会访问到哪个目录，可以自行使用IP访问你的服务器测试。也可以直接创建站点，域名为你的IP，这样直接使用IP访问就会被定向到这个站点的文件夹
</p>
</details>
![image](https://user-images.githubusercontent.com/110012832/180934634-650f1b4a-32e8-43b6-854a-05f4b2bcc05b.png)
安装网页服务器和查找路径请参考网上的其它教程
安装完成后，进入到你的服务器终端，输入以下命令行
```acme.sh --issue -d [你的IP/域名] --webroot [网站根目录] --server https://acme.hi.cn/directory```
正确格式例如
```acme.sh --issue -d 11.4.5.14 --webroot /www/wwwroot/114514 --server https://acme.hi.cn/directory```
```acme.sh --issue -d 1145:1419:1981:0114:5141:9198:1011:4514 --webroot /home/wwwroot/ --server https://acme.hi.cn/directory```
稍等片刻，提示签发成功即可下载证书，位于 ```/root/.acme.sh/你的域名``` 比如 ```/root/.acme.sh/11.4.5.14```
若提示 acme.sh 命令不存在，直接 cd 到 /root 目录下的 .acme.sh 目录下只用相对路径执行命令即可。
只需要指定域名, 并指定域名所在的网站根目录. acme.sh 会全自动的生成验证文件, 并放到网站的根目录, 然后自动完成验证. 最后会聪明的删除验证文件. 整个过程没有任何副作用。
如果你用的 apache服务器, acme.sh 还可以智能的从 apache的配置中自动完成验证, 你不需要指定网站根目录：
```acme.sh --issue  -d mydomain.com   --apache --server https://acme.hi.cn/directory```
如果你使用Nginx或者反代，acme.sh还可以智能地更改Nginx配置文件自动完成验证，从而不需要指定网站根目录：
```acme.sh --issue  -d mydomain.com   --nginx --server https://acme.hi.cn/directory```
注意，无论是 apache 还是 nginx 模式，acme.sh在完成验证之后，会恢复到之前的状态，都不会私自更改你本身的配置。好处是你不用担心配置被搞坏，也有一个缺点，你需要自己配置 ssl 的配置，否则只能成功生成证书，你的网站还是无法访问https。但是为了安全, 你还是自己手动改配置吧。
如果你还没有运行任何 web 服务，80 端口是空闲的，那么 acme.sh 还能假装自己是一个webserver，临时监听80 端口，完成验证：
```acme.sh  --issue -d mydomain.com   --standalone --server https://acme.hi.cn/directory```
更高级的用法请参考: https://github.com/Neilpang/acme.sh/wiki/How-to-issue-a-cert
### 常见问题
- IP证书只能通过HTTP验证签发，因此只能签发单域名证书，不能为多个IP签发一张证书
- 签发ECC证书：在命令行最后添加--keylength ec-256 或 -k ec-256
- [warning]对使用HTTP验证的证书，网站目录设置不正确就无法签发成功[/warning]











