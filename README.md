# Hi CA SSL证书 部署教程
2022 年 7 月 23 日，国产数字证书品牌 HiCA 正式上线，提供免费的半年 IP 或者通配符 DV SSL 证书，非常实用
官网：https://www1.hi.cn/
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
安装很简单, 一个命令：
curl  https://get.acme.sh | sh -s email=my@example.com
