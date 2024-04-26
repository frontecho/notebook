# 使用dae为Docker指定容器配置全局代理

docker先创建一个网络组 `your_proxy_net_name` 作为代理网络组

```bash
docker network create --driver bridge ***your_proxy_net_name***
```

使用docker compose创建网络，在Dockerfile配置中单独配置网络组为`your_proxy_net_name` 

```bash
services:
  api:
    container_name: ***your_container_name***
    ports:
      - "${PORT}:${PORT}"
    networks:
      - default
    extra_hosts:
      - "host.docker.internal:host-gateway"

networks:
  default:
    external: true
    name: ***your_proxy_net_name***
```

docker安装dae

```bash
docker run -d \
--name=dae \
--restart=always \
--network=host \
--pid=host \
--privileged \
-v /sys:/sys \
-v /etc/dae:/etc/dae \
daeuniverse/dae:latest
```

修改dae配置（路径为 `/etc/dae/config.dae` )

参考配置 https://github.com/sakarie9/network-configs/blob/main/dae/config.dae

其中 `eth0` 为流量出口网卡， `br-0123456789ab` 为刚刚创建的网络组的网卡接口名称
你可以通过 `ip a` 命令来查看所有接口，先进入到 `your_container_name` 中查看所有网卡接口，再和宿主机的网卡接口进行对比，找到ip地址相同的以 `br-` 开头的网卡接口名称填入。

```bash
global{
        log_level: info
        wan_interface: ***eth0***
        lan_interface: ***br-0123456789ab***
        auto_config_kernel_parameter: true
}
dns {
    upstream {
        googledns: 'tcp+udp://dns.google.com:53'
    }
    routing {
        request {
            fallback: googledns
        }
    }
}
group {
        my_group{
                policy: fixed(0)
        }
}
routing{
        pname(xray-linux-amd64) -> must_direct
        pname(systemd-resolved) -> must_direct
        domain(suffix: aliyun.com) -> must_direct
        fallback: my_group
}
node{
        local:'socks5://localhost:10808'
}
```

[https://blog.skyju.cc/post/dae-docker-full-proxy/#daeebpf-网卡级代理](https://blog.skyju.cc/post/dae-docker-full-proxy/#daeebpf-%E7%BD%91%E5%8D%A1%E7%BA%A7%E4%BB%A3%E7%90%86)