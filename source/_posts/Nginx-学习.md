---
title: Nginx 专题
date: 2018-08-20 09:40:56
tags:
    - Nginx
categories:
    - Nginx
---
<iframe frameborder="no" border="0" marginwidth="0" marginheight="0" width=330
height=86 src="//music.163.com/outchain/player?type=2&id=543607345&auto=1&height=66"></iframe>

#### Nginx安装

##### 安装依赖
###### gcc 安装
安装 nginx 需要先将官网下载的源码进行编译，编译依赖 gcc 环境，如果没有 gcc 环境，则需要安装：

    yum install -y gcc-c++
###### PCRE pcre-devel 安装
PCRE(Perl Compatible Regular Expressions) 是一个Perl库，包括 perl 兼容的正则表达式库。nginx 的 http 模块使用 pcre 来解析正则表达式，所以需要在 linux 上安装 pcre 库，pcre-devel 是使用 pcre 开发的一个二次开发库。nginx也需要此库。命令：

    yum install -y pcre pcre-devel
###### zlib 安装
zlib 库提供了很多种压缩和解压缩的方式， nginx 使用 zlib 对 http 包的内容进行 gzip ，所以需要在 Centos 上安装 zlib 库。

    yum install -y zlib zlib-devel
###### OpenSSL 安装
OpenSSL 是一个强大的安全套接字层密码库，囊括主要的密码算法、常用的密钥和证书封装管理功能及 SSL 协议，并提供丰富的应用程序供测试或其它目的使用。
nginx 不仅支持 http 协议，还支持 https（即在ssl协议上传输http），所以需要在 Centos 安装 OpenSSL 库。
    
    yum install -y openssl openssl-devel

##### 安装nginx
###### 下载nginx

    wget https://nginx.org/download/nginx-1.12.1.tar.gz
###### 编译安装

    tar -zxvf nginx-1.12.1.tar.gz
    cd nginx-1.12.1
    ./configure
    make
    make install
###### 查找安装路径
    
    whereis nginx
    
#### SSL 解决
```text
很早之前我就在关注 Let's Encrypt 这个免费、自动化、开放的证书签发服务。它由 ISRG（Internet Security Research Group，互联网安全研究小组）提供服务，而 ISRG 是来自于美国加利福尼亚州的一个公益组织。Let's Encrypt 得到了 Mozilla、Cisco、Akamai、Electronic Frontier Foundation 和 Chrome 等众多公司和机构的支持，发展十分迅猛。
申请 Let's Encrypt 证书不但免费，还非常简单，虽然每次只有 90 天的有效期，但可以通过脚本定期更新，配好之后一劳永逸。经过一段时间的观望，我也正式启用 Let's Encrypt 证书了，本文记录本站申请过程和遇到的问题。
我没有使用 Let's Encrypt 官网提供的工具来申请证书，而是用了 acme-tiny 这个更为小巧的开源工具。以下内容基本按照 acme-tiny 的说明文档写的，省略了一些我不需要的步骤。
ACME 全称是 Automated Certificate Management Environment，直译过来是自动化证书管理环境的意思，Let's Encrypt 的证书签发过程使用的就是 ACME 协议。有关 ACME 协议的更多资料可以在这个仓库找到。
创建帐号
首先创建一个目录，例如 ssl，用来存放各种临时文件和最后的证书文件。进入这个目录，创建一个 RSA 私钥用于 Let's Encrypt 识别你的身份：
openssl genrsa 4096 > account.key
创建 CSR 文件
接着就可以生成 CSR（Certificate Signing Request，证书签名请求）文件了。在这之前，还需要创建域名私钥（一定不要使用上面的账户私钥），根据证书不同类型，域名私钥也可以选择 RSA 和 ECC 两种不同类型。以下两种方式请根据实际情况二选一。
```
##### 创建 RSA 私钥（兼容性好）：

    openssl genrsa 4096 > domain.key

##### 创建 ECC 私钥（部分老旧操作系统、浏览器不支持。优点是证书体积小）：

    openssl ecparam -genkey -name secp256r1 | openssl ec -out domain.key
    # 应该是选择一个就行。
    #secp384r1
    openssl ecparam -genkey -name secp384r1 | openssl ec -out domain.key


##### 生成CSR
有了私钥文件，就可以生成 CSR 文件了。在 CSR 中推荐至少把域名带 www 和不带 www 的两种情况都加进去，其它子域可以根据需要添加（目前一张证书最多可以包含 100 个域名）：

    openssl req -new -sha256 -key domain.key -subj "/" -reqexts SAN -config <(cat /etc/ssl/openssl.cnf <(printf "[SAN]\nsubjectAltName=DNS:yoursite.com,DNS:www.yoursite.com")) > domain.csr

执行这一步时，如果提示找不到 /etc/ssl/openssl.cnf 文件，请看看 /usr/local/openssl/ssl/openssl.cnf 是否存在。如果还是不行，也可以使用交互方式创建 CSR（需要注意 Common Name 必须为你的域名）：

    openssl req -new -sha256 -key domain.key -out domain.csr

##### 配置验证服务
我们知道，CA 在签发 DV（Domain Validation）证书时，需要验证域名所有权。传统 CA 的验证方式一般是往 admin@yoursite.com 发验证邮件，而 Let's Encrypt 是在你的服务器上生成一个随机验证文件，再通过创建 CSR 时指定的域名访问，如果可以访问则表明你对这个域名有控制权。
首先创建用于存放验证文件的目录，例如：

    mkdir ~/www/challenges/

然后配置一个 HTTP 服务，以 Nginx 为例：
    
    server{
            server_name www.yoursite.com yoursite.com;
        
            location ^~ /.well-known/acme-challenge/ {
                alias /home/xxx/www/challenges/;
                try_files $uri =404;
            }
        
            location / {
                rewrite ^/(.*)$ https://yoursite.com/$1 permanent;
            }
    }

以上配置优先查找 ~/www/challenges/ 目录下的文件，如果找不到就重定向到 HTTPS 地址。这个验证服务以后更新证书还要用到，建议一直保留。
##### 获取网站证书
先把 acme-tiny 脚本保存到之前的 ssl 目录：

    wget https://raw.githubusercontent.com/diafygi/acme-tiny/master/acme_tiny.py

指定账户私钥、CSR 以及验证目录，执行脚本：

    python acme_tiny.py --account-key ./account.key --csr ./domain.csr --acme-dir ~/www/challenges/ > ./signed.crt

如果一切正常，当前目录下就会生成一个 signed.crt，这就是申请好的证书文件。
如果你把域名 DNS 解析放在国内，这一步很可能会遇到类似这样的错误：
ValueError: Wrote file to /home/xxx/www/challenges/oJbvpIhkwkBGBAQUklWJXyC8VbWAdQqlgpwUJkgC1Vg, but couldn't download http://www.yoursite.com/.well-known/acme-challenge/oJbvpIhkwkBGBAQUklWJXyC8VbWAdQqlgpwUJkgC1Vg

这是因为你的域名很可能在国外无法解析，可以找台国外 VPS 验证下。我的域名最近从 DNSPod 换到了阿里云解析，最后又换到了 CloudXNS，就是因为最近前两家在国外都很不稳定。如果你也遇到了类似情况，可以暂时使用国外的 DNS 解析服务商，例如 dns.he.net。如果还是搞不定，也可以试试「Neilpang/le」这个工具的 DNS Mode。
搞定网站证书后，还要下载 Let's Encrypt 的中间证书。我在之前的文章中讲过，配置 HTTPS 证书时既不要漏掉中间证书，也不要包含根证书。在 Nginx 配置中，需要把中间证书和网站证书合在一起：

    wget -O - https://letsencrypt.org/certs/lets-encrypt-x3-cross-signed.pem > intermediate.pem
    cat signed.crt intermediate.pem > chained.pem

为了后续能顺利启用 OCSP Stapling，我们再把根证书和中间证书合在一起：

    wget -O - https://letsencrypt.org/certs/isrgrootx1.pem > root.pem
    cat intermediate.pem root.pem > full_chained.pem

最终，修改 Nginx 中有关证书的配置并 reload 服务即可：

    ssl_certificate     ~/www/ssl/chained.pem;
    ssl_certificate_key ~/www/ssl/domain.key;

Nginx 中与 HTTPS 有关的配置项很多，这里不一一列举了。如有需要，请参考本站配置。
##### 配置自动更新
Let's Encrypt 签发的证书只有 90 天有效期，推荐使用脚本定期更新。例如我就创建了一个 renew_cert.sh 并通过 chmod a+x renew_cert.sh 赋予执行权限。文件内容如下：
```shell
#!/bin/bash

cd /home/xxx/www/ssl/
python acme_tiny.py --account-key account.key --csr domain.csr --acme-dir /home/xxx/www/challenges/ > signed.crt || exit
wget -O - https://letsencrypt.org/certs/lets-encrypt-x3-cross-signed.pem > intermediate.pem
cat signed.crt intermediate.pem > chained.pem
service nginx reload

crontab 中使用绝对路径比较保险，crontab -e 加入以下内容：
0 0 1 * * /home/xxx/shell/renew_cert.sh >/dev/null 2>&1
```

这样以后证书每个月都会自动更新，一劳永逸。实际上，Let's Encrypt 官方将证书有效期定为 90 天一方面是为了更安全，更重要的是鼓励用户采用自动化部署方案。
