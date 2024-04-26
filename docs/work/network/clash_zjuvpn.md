---
tags:
  - clash
  - TUN
  - zjuvpn
---

# Clash TUN实现校外网访问ZJUVPN+节点分流

zju-connect：https://github.com/Mythologyli/zju-connect

ZJU-Rule：https://github.com/Mythologyli/ZJU-Rule

socks2shadowsocks：https://github.com/Srar/socks2shadowsocks

本方案在Linux服务器上搭建，使用Docker容器部署

### Step1

创建zju-connect的config.toml配置文件，位置 `/etc/zju-connect/config.toml`  ，文件内容[Example](https://github.com/Mythologyli/zju-connect/blob/main/config.toml.example)

修改username，password两个参数即可，其他无需改动

### Step2

创建运行zju-connect容器

```bash
docker run -d \
  --name zju-connect \
  -v /etc/zju-connect/config.toml:/home/nonroot/config.toml \
  --expose 1080 \
  --restart unless-stopped \
  mythologyli/zju-connect
```

### Step3

编译socks2shadowsocks镜像

先使用git拉取仓库代码

`git clone https://github.com/Srar/socks2shadowsocks.git`

修改 `socks2shadowsocks/dockerfile` 文件为以下内容

```docker
FROM node:8.16.2-alpine
ENV TIME_ZONE=Asia/Shanghai
WORKDIR /app
COPY . /app
RUN npm install --registry=https://registry.npm.taobao.org && npm run build
CMD ["sh", "-c", "node build/example.js --localPort ${LOCAL_PORT} --localCipher ${LOCAL_CIPHER} --localPassword ${LOCAL_PASSWORD} --socks5Addr ${SOCKS5_ADDR} --socks5Port ${SOCKS5_PORT}"]
```

保存后进入socks2shadowsocks文件夹，构建镜像

```bash
docker build -t socks2shadowsocks:latest .
```

### Step4

创建运行socks2shadowsocks容器，将zju-connect的socks5代理转换为shadowsocks，提高传输安全性

```bash
docker run -d \
  --name socks2shadowsocks \
  --link zju-connect:zju \
  -e LOCAL_PORT=3489 \
  -e LOCAL_CIPHER=rc4-md5 \
  -e LOCAL_PASSWORD=YourPassword \
  -e SOCKS5_ADDR=zju \
  -e SOCKS5_PORT=1080 \
  -p 3489:3489 \
  --restart unless-stopped \
  socks2shadowsocks:latest
```

### Step5

Clash订阅文件中添加以下节点

```yaml
  - {name: ZJU Connect, server:Your Sever IP, port: 3489, cipher: rc4-md5, password: "YourPassword", udp: true, type: ss}
```

分流规则添加以下内容

```yaml
rules:
# https://github.com/SubConv/ZJU-Rule/blob/main/Clash/ZJU-IP.list
  - IP-CIDR,210.32.174.2/32,DIRECT,no-resolve # rvpn ip
  - IP-CIDR,218.108.88.254/32,DIRECT,no-resolve
  - IP-CIDR,124.160.105.200/32,DIRECT,no-resolve
  - IP-CIDR,61.175.193.50/32,DIRECT,no-resolve
  - IP-CIDR,210.32.137.90/32,ZJUNET,no-resolve # 图书馆
  - IP-CIDR,210.32.137.198/32,ZJUNET,no-resolve

  - IP-CIDR,10.0.0.0/8,ZJUNET,no-resolve
  - IP-CIDR,58.196.192.0/19,ZJUNET,no-resolve
  - IP-CIDR,58.196.224.0/20,ZJUNET,no-resolve
  - IP-CIDR,58.200.100.0/24,ZJUNET,no-resolve
  - IP-CIDR,210.32.0.0/20,ZJUNET,no-resolve
  - IP-CIDR,210.32.128.0/19,ZJUNET,no-resolve
  - IP-CIDR,210.32.160.0/21,ZJUNET,no-resolve
  - IP-CIDR,210.32.168.0/22,ZJUNET,no-resolve
  - IP-CIDR,210.32.172.0/23,ZJUNET,no-resolve
  - IP-CIDR,210.32.174.0/24,ZJUNET,no-resolve
  - IP-CIDR,210.32.176.0/20,ZJUNET,no-resolve
  - IP-CIDR,222.205.0.0/17,ZJUNET,no-resolve

  - DOMAIN-SUFFIX,mirror.zju.edu.cn,DOMESTIC
  - DOMAIN-SUFFIX,mirrors.zju.edu.cn,DOMESTIC
  - DOMAIN-SUFFIX,webvpn.zju.edu.cn,DIRECT
  - DOMAIN-SUFFIX,rvpn.zju.edu.cn,DIRECT
# https://github.com/SubConv/ZJU-Rule/blob/main/Clash/ZJU.list
  - DOMAIN-SUFFIX,zju.edu.cn,ZJUNET # 主域名
  - DOMAIN-KEYWORD,cc98,ZJUNET # CC98
  - DOMAIN-KEYWORD,nexushd,ZJUNET # NexusHD
  - DOMAIN-SUFFIX,icsr.wiki,ZJUNET # ICSR WIKI
  - DOMAIN-SUFFIX,zjusec.com,ZJUNET # ZJU School-Bus
  - DOMAIN-SUFFIX,zjusec.net,ZJUNET
  - DOMAIN-SUFFIX,zjusec.top,ZJUNET
  - DOMAIN-SUFFIX,zjusct.io,ZJUNET # ZJU SCT
  - DOMAIN-SUFFIX,zjueva.net,ZJUNET # ZJU EVA
  - DOMAIN-SUFFIX,zjuqsc.com,ZJUNET # ZJU QSC
  - DOMAIN-KEYWORD,chalaoshi,ZJUNET # 查老师
  - DOMAIN-SUFFIX,acm.org,ZJUNET # 通过 IP 认证的资源
  - DOMAIN-SUFFIX,cnki.net,ZJUNET
  - DOMAIN-SUFFIX,gtadata.com,ZJUNET
  - DOMAIN-SUFFIX,jstor.org,ZJUNET
  - DOMAIN-SUFFIX,webofscience.com,ZJUNET
  - DOMAIN-SUFFIX,inoteexpress.com,ZJUNET
  - DOMAIN-SUFFIX,pnas.org,ZJUNET
  - DOMAIN-SUFFIX,cnpereading.com,ZJUNET
  - DOMAIN-SUFFIX,sciencemag.org,ZJUNET
  - DOMAIN-SUFFIX,cas.org,ZJUNET
  - DOMAIN-SUFFIX,webofknowledge.com,ZJUNET
  - DOMAIN-SUFFIX,pkulaw.com,ZJUNET
  - DOMAIN-SUFFIX,sslibrary.com,ZJUNET
  - DOMAIN-SUFFIX,serialssolutions.com,ZJUNET
  - DOMAIN-SUFFIX,duxiu.com,ZJUNET
  - DOMAIN-SUFFIX,wanfangdata.com.cn,ZJUNET
  - DOMAIN-SUFFIX,koolearn.com,ZJUNET
  - DOMAIN-SUFFIX,cssci.nju.edu.cn,ZJUNET
  - DOMAIN-SUFFIX,science.org,ZJUNET
  - DOMAIN-SUFFIX,aps.org,ZJUNET
  - DOMAIN-SUFFIX,incopat.com,ZJUNET
  - DOMAIN-SUFFIX,oup.com,ZJUNET # NSTL
  - DOMAIN-SUFFIX,ajtmh.org,ZJUNET
  - DOMAIN-SUFFIX,futuremedicine.com,ZJUNET
  - DOMAIN-SUFFIX,tandfonline.com,ZJUNET
  - DOMAIN-SUFFIX,genetics.org,ZJUNET
  - DOMAIN-SUFFIX,healthaffairs.org,ZJUNET
  - DOMAIN-SUFFIX,rsna.org,ZJUNET
  - DOMAIN-SUFFIX,iospress.com,ZJUNET
  - DOMAIN-SUFFIX,allenpress.com,ZJUNET
  - DOMAIN-SUFFIX,asabe.org,ZJUNET
  - DOMAIN-SUFFIX,geoscienceworld.org,ZJUNET
  - DOMAIN-SUFFIX,sagepub.com,ZJUNET
  - DOMAIN-SUFFIX,ajnr.org,ZJUNET
  - DOMAIN-SUFFIX,ajhp.org,ZJUNET
  - DOMAIN-SUFFIX,annals.org,ZJUNET
  - DOMAIN-SUFFIX,esajournals.org,ZJUNET
  - DOMAIN-SUFFIX,informs.org,ZJUNET
  - DOMAIN-SUFFIX,cshlpress.com,ZJUNET
  - DOMAIN-SUFFIX,nrcresearchpress.cn,ZJUNET
  - DOMAIN-SUFFIX,royalsocietypublishing.org,ZJUNET
  - DOMAIN-SUFFIX,oxfordjournals.org,ZJUNET
  - DOMAIN-SUFFIX,aspbjournals.org,ZJUNET
  - DOMAIN-SUFFIX,sciencesocieties.org,ZJUNET
  - DOMAIN-SUFFIX,degruyter.com,ZJUNET
  - DOMAIN-SUFFIX,cshprotocols.org,ZJUNET
  - DOMAIN-SUFFIX,liebertonline.com,ZJUNET
  - DOMAIN-SUFFIX,polymerjournals.com,ZJUNET
  - DOMAIN-SUFFIX,csiro.au,ZJUNET
  - DOMAIN-SUFFIX,iop.org,ZJUNET
  - DOMAIN-SUFFIX,electrochem.org,ZJUNET
  - DOMAIN-SUFFIX,ametsoc.org,ZJUNET
  - DOMAIN-SUFFIX,portlandpress.com,ZJUNET
  - DOMAIN-SUFFIX,nrcresearchpress.com,ZJUNET
  - DOMAIN-SUFFIX,arabidopsis.org,ZJUNET
  - DOMAIN-SUFFIX,springerlink.com,ZJUNET
  - DOMAIN-SUFFIX,highwire.org,ZJUNET
  - DOMAIN-SUFFIX,ovid.com,ZJUNET
  - DOMAIN-SUFFIX,rsc.org,ZJUNET
  - DOMAIN-SUFFIX,bmj.org,ZJUNET
  - DOMAIN-SUFFIX,aip.org,ZJUNET
  - DOMAIN-SUFFIX,springer.com,ZJUNET
  - DOMAIN-SUFFIX,iwaponline.com,ZJUNET
  - DOMAIN-SUFFIX,rsnajnls.org,ZJUNET
  - DOMAIN-SUFFIX,karger.com,ZJUNET
  - DOMAIN-SUFFIX,plantcell.org,ZJUNET
  - DOMAIN-SUFFIX,jamanetwork.com,ZJUNET
  - DOMAIN-SUFFIX,nejm.org,ZJUNET
  - DOMAIN-SUFFIX,icevirtuallibrary.com,ZJUNET
  - DOMAIN-SUFFIX,cdnsciencepub.com,ZJUNET
# https://github.com/SubConv/ZJU-Rule/blob/main/Clash/ZJU-More-Scholar.list
  - DOMAIN-SUFFIX,canvas-user-content.com,AUTOPROXY
  - DOMAIN-KEYWORD,illinois,AUTOPROXY
  - DOMAIN-SUFFIX,iclicker.com,AUTOPROXY
  - DOMAIN-SUFFIX,emerald.com,AUTOPROXY
  - DOMAIN-SUFFIX,ieee.org,AUTOPROXY
  - DOMAIN-SUFFIX,proquest.com,AUTOPROXY
  - DOMAIN-SUFFIX,sciencedirect.com,AUTOPROXY
  - DOMAIN-SUFFIX,sciencedirectassets.com,AUTOPROXY
  - DOMAIN-SUFFIX,nature.com,AUTOPROXY
  - DOMAIN-SUFFIX,acs.org,AUTOPROXY
  - DOMAIN-SUFFIX,taylorfrancis.com,AUTOPROXY
  - DOMAIN-SUFFIX,osapublishing.org,AUTOPROXY
  - DOMAIN-SUFFIX,clarivate.com,AUTOPROXY
  - DOMAIN-SUFFIX,gale.com,AUTOPROXY
  - DOMAIN-SUFFIX,worldscientific.com,AUTOPROXY
  - DOMAIN-SUFFIX,siam.org,AUTOPROXY
  - DOMAIN-SUFFIX,ascelibrary.org,AUTOPROXY
  - DOMAIN-SUFFIX,scitation.org,AUTOPROXY
  - DOMAIN-SUFFIX,wiley.com,AUTOPROXY
```
