# docker-gfw
## 通过 docker-compose 一次性开启 SS + SSR + torjan

> 于 谷歌云 e2-micro（2 个 vCPU，1 GB 内存） Ubuntu 18.04 LTS 系统下测试



## 0）克隆本项目获取 docker-compose.yml 与相关配置文件
```
git clone https://github.com/fejich/docker-gfw.git \
&& cd docker-gfw
```

## 1）安装 Docker & Docker-compose & ctop
```
sudo snap install docker \
&& sudo curl -L "https://github.com/docker/compose/releases/download/1.25.4/docker-compose-$(uname -s)-$(uname -m)" -o /usr/local/bin/docker-compose \
&& sudo chmod +x /usr/local/bin/docker-compose \
&& sudo wget https://github.com/bcicen/ctop/releases/download/v0.7.3/ctop-0.7.3-linux-amd64 -O /usr/local/bin/ctop \
&& sudo chmod +x /usr/local/bin/ctop
```
> 非 Ubuntu 系统参考 Docker 官方方式安装：https://docs.docker.com/get-docker/

## 2）通过 docker-compose 命令一次性开启 SS + SSR  + torjan + nginx 4个容器
```
sudo docker-compose up -d \
&& sudo docker stop trojan
```
> torjan 需要于第 3）步正确配置域名解析并生成 SSL 证书后才能工作，所以目前先停止该容器运行

## 3）通过 acme.sh 生成 SSL 证书
1.开始前请配置好 vps 的域名解析
```
mydomain="修改成你的域名"
curl  https://get.acme.sh | sh \
&& ~/.acme.sh/acme.sh --issue -d $mydomain --webroot ./www
```
2.复制证书到 trojan 目录，重启 trojan 容器
```
~/.acme.sh/acme.sh --installcert -d $mydomain \
--cert-file      ./trojan/certificate.crt \
--key-file       ./trojan/private.key \
--reloadcmd     "sudo docker restart trojan"
```
> 目前证书在 60 天以后会自动更新, 你无需任何操作.

## 4）通过 ctop 命令管理各容器
```
sudo ctop
```
<p align="center"><img src="ctop.png" alt="ctop"/></p>
1.选择 trojan 容器，按回车选择 log view 可查看日志

2.浏览器打开 https://你的域名 ，正常情况下会打开 ariang 页面，终端窗口滚动显示相关连接信息。
  至此完成所有安装配置。

## 5）可选-开启 BBR 加速
```
echo "net.core.default_qdisc=fq" | sudo tee --append /etc/sysctl.conf \
&& echo "net.ipv4.tcp_congestion_control=bbr" | sudo tee --append /etc/sysctl.conf \
&& sudo sysctl -p
```

## 6）各服务默认设定
#### SS
+ 端口：23456
+ 密码：bing.com
+ 加密：chacha20

#### SSR 
+ 端口：23457
+ 密码：bing.com
+ 加密：none
+ 协议：origin
+ 混淆：plain

#### trojan
+ 端口：443
+ 密码：bing.com

## 7）相关链接
+ SS                     https://hub.docker.com/r/mrjin/shadowsocks
+ SSR                    https://hub.docker.com/r/winterssy/shadowsocksr
+ trojan                 https://hub.docker.com/r/trojangfw/trojan
+ nginx                  https://hub.docker.com/_/nginx
+ ctop                   https://github.com/bcicen/ctop
+ acme.sh                https://github.com/acmesh-official/acme.sh/wiki/%E8%AF%B4%E6%98%8E
