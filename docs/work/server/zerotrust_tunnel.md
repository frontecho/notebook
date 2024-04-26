# 服务器配置ZeroTrust proxy及Warp Tunnel

安装warp-cli

```bash
# Add cloudflare gpg key
curl https://pkg.cloudflareclient.com/pubkey.gpg | sudo gpg --yes --dearmor --output /usr/share/keyrings/cloudflare-warp-archive-keyring.gpg

# Add this repo to your apt repositories
echo "deb [arch=amd64 signed-by=/usr/share/keyrings/cloudflare-warp-archive-keyring.gpg] https://pkg.cloudflareclient.com/ $(lsb_release -cs) main" | sudo tee /etc/apt/sources.list.d/cloudflare-client.list

# Install
sudo apt-get update && sudo apt-get install cloudflare-warp
```

安装cloudflared来配置Zero Trust Tunnel

```bash
# Add cloudflare gpg key
sudo mkdir -p --mode=0755 /usr/share/keyrings
curl -fsSL https://pkg.cloudflare.com/cloudflare-main.gpg | sudo tee /usr/share/keyrings/cloudflare-main.gpg >/dev/null

# Add this repo to your apt repositories
echo 'deb [signed-by=/usr/share/keyrings/cloudflare-main.gpg] https://pkg.cloudflare.com/cloudflared jammy main' | sudo tee /etc/apt/sources.list.d/cloudflared.list

# install cloudflared
sudo apt-get update && sudo apt-get install cloudflared
```

默认 Warp 的工作模式是类似 TUN 的模式，但实际上只需要让它提供一个 Socks 的代理接口。为了在 Zero Trust 中使用这个功能，可以在 Cloudflare Zero Trust 管理面板中新建一个设备 Profile（Settings - Device Settings - Create Profile），规则按照自己的条件来选，我这里选择了用户 + Linux 操作系统。之后在 Service mode 中选择 Proxy mode（默认是 Gateway with WARP）：

!https://oss.init.blog/zero-trust-warp-mode.webp

在随便一个浏览器里面打开 `https://[teams-id].cloudflareaccess.com/warp` 这个链接，会要求你登陆自己的组织，登陆成功后会打开这样一个页面：

!https://oss.init.blog/zero-trust-warp-logins.webp

把 Authentication 按钮 `href` 单引号内的内容复制出来 `com.cloudflare.warp://...`，之后在服务器上执行：

```bash
warp-cli teams-enroll-token [href]

warp-cli connect
```

检查ip新息

无ZeroTrust Proxy：`curl ipinfo.io`

使用ZeroTrust Proxy：`curl ipinfo.io -x socks5://127.0.0.1:40000`

Cloudflare Zero Trust 提供了一个 Tunnel 的功能，相当于一个内网穿透隧道。

首先创建一个 Tunnel（Access - Tunnels - Create a Tunnel），随便给一个域名或者子域，并在 Configure 里配置到 SSH 服务的映射。

在 Tunnel 的管理页面会有一个 Token，使用下面这个命令连接到你刚刚创建的 Tunnel：

`sudo cloudflared service install [your-tunnel-token]`

重启cloudflared

```
sudo systemctl daemon-reload
sudo systemctl restart cloudflared
```

配置代理（V2ray）

```bash
{
    "tag": "warp",
    "protocol": "socks",
    "settings": {
        "servers": [
            {
                "address": "127.0.0.1",
                "port": 40000,
                "users": []
            }
        ]
    }
}
```

https://init.blog/warp-zero-trust-vless-ws/

https://sspai.com/post/79278#!

[https://blog.hellowood.dev/posts/使用cloudflare-tunnels通过web-ssh访问服务器/](https://blog.hellowood.dev/posts/%E4%BD%BF%E7%94%A8cloudflare-tunnels%E9%80%9A%E8%BF%87web-ssh%E8%AE%BF%E9%97%AE%E6%9C%8D%E5%8A%A1%E5%99%A8/)