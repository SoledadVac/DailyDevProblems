# 生成秘钥

生成秘钥key：

```
openssl genrsa -des3 -out server.key 2048
```
然后你就获得了一个server.key文件.
以后使用此文件(通过openssl提供的命令或API)可能经常回要求输入密码,如果想去除输入密码的步骤可以使用以下命令:
```
openssl rsa -in server.key -out server.key
```
# 服务器证书的申请文件server.csr
```
openssl req -new -key server.key -out server.csr
```
其中Country Name填CN,Common Name填主机名也可以不填,如果不填浏览器会认为不安全.(例如你以后的url为https://abcd/xxxx....这里就可以填abcd),其他的都可以不填.
# 创建CA证书
```
openssl req -new -x509 -key server.key -out ca.crt -days 3650
```
# 创建自当前日期起有效期为期十年的服务器证书server.crt
```
openssl x509 -req -days 3650 -in server.csr -CA ca.crt -CAkey server.key -CAcreateserial -out server.crt
```
查看生成的文件：
```
[root@bj-esbp-mid1 ~]# ls
anaconda-ks.cfg  ca.crt  ca.srl  install.log  install.log.syslog  server.crt  server.csr  server.key
```
ls你的文件夹,可以看到一共生成了5个文件:
ca.crt ca.srl `server.crt` server.csr `server.key`
其中,server.crt和server.key就是你的nginx需要的证书文件.
# nginx配置

set http rewrite to https:
```
 server {
        listen       80;
        server_name  10.0.1.101;
        rewrite ^ https://$http_host$request_uri? permanent;    # force redirect http to https
```
set https config:
```
 # HTTPS server
    #
    server{
        listen       443 ssl;
        server_name  10.0.1.101;
        ssl                on;
        ssl_certificate     /usr/local/nginx/conf/bjesserver.crt;#配置证书位置
        ssl_certificate_key  /usr/local/nginx/conf/bjesserver.key;#配置秘钥位置
        ssl_session_timeout  5m;
        ssl_protocols  SSLv2 SSLv3 TLSv1;
        ssl_ciphers  ALL:!ADH:!EXPORT56:RC4+RSA:+HIGH:+MEDIUM:+LOW:+SSLv2:+EXP;
        ssl_prefer_server_ciphers   on;
        location / {
            root   /data/www/ESBP_WEB/dist;
            index  index.html index.htm;
        }

    }
```
`finally,restart not reload,caz reload doesn't work!!!!!`
