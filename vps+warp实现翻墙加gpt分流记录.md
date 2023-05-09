# 购买VPN，域名，使用x-ui

参考：[搭梯子 vps推荐 – 2023年 5月](https://www.triadprogram.com/%E6%90%AD%E6%A2%AF%E5%AD%90-vps%E6%8E%A8%E8%8D%90/)

注意：x-ui启动xray之后，如果在浏览器不能访问x-ui面版，需要关闭一下VPS的防火墙：

```
systemctl stop ufw
```

# 使用cloudflare warp分流流量

参考：[Cloudflare WARP 教程：VPS 免费接入 IPv4/IPv6 双栈网络，IPv6 访问 IPv4 网络](https://p3terx.com/archives/use-cloudflare-warp-to-add-extra-ipv4-or-ipv6-network-support-to-vps-servers-for-free.html)

[Cloudflare WARP 一键安装脚本 使用教程] (https://p3terx.com/archives/cloudflare-warp-configuration-script.html)

安装cloudflare-warp并设置socks5代理:

```
bash <(curl -fsSL git.io/warp.sh) s5
```

配置好后会在40000端口上开启 socks5代理

# 配置x-ui面板 xray配置

参考： [X-ui面板配置分流规则，实现一个节点访问不同网站按需分流到不同的IP](https://xtrojan.pro/blog/x-ui-panel-configuration-shunting-rules.html)

下面是根据参考文章修改的能够分流chatgpt和bing的配置：

```
{
    "api": {
        "services": [
            "HandlerService",
            "LoggerService",
            "StatsService"
        ],
        "tag": "api"
    },
    "inbounds": [
        {
            "listen": "127.0.0.1",
            "port": 62789,
            "protocol": "dokodemo-door",
            "settings": {
                "address": "127.0.0.1"
            },
            "tag": "api"
        }
    ],
    "outbounds": [
        {
            "tag": "IP4-out",
            "protocol": "freedom",
            "settings": {}
        },
        {
            "tag": "IP6-out",
            "protocol": "freedom",
            "settings": {
                "domainStrategy": "UseIPv6"
            }
        },
        {
            "tag": "socks5-warp",
            "protocol": "socks",
            "settings": {
                "servers": [
                    {
                        "address": "127.0.0.1",
                        "port": 40000
                    }
                ]
            }
        }
    ],
    "policy": {
        "system": {
            "statsInboundDownlink": true,
            "statsInboundUplink": true
        }
    },
    "routing": {
        "rules": [
            {
                "type": "field",
                "outboundTag": "IP6-out",
                "domain": [
                    "ipget.net"
                ]
            },
            {
                "type": "field",
                "outboundTag": "socks5-warp",
                "domain": [
                    "geosite:google",
                    "openai.com",
                    "ai.com",
                    "bing.com"
                ]
            },
            {
                "type": "field",
                "outboundTag": "IP4-out",
                "network": "udp,tcp"
            },
            {
                "inboundTag": [
                    "api"
                ],
                "outboundTag": "api",
                "type": "field"
            },
            {
                "ip": [
                    "geoip:private"
                ],
                "outboundTag": "blocked",
                "type": "field"
            },
            {
                "outboundTag": "blocked",
                "protocol": [
                    "bittorrent"
                ],
                "type": "field"
            }
        ]
    },
    "stats": {}
}
```

把这部分配置粘贴到xray的配置中，重启面板，就可以实现分流了



# 客户端

使用clash，在x-ui入站列表，粘贴订阅链接，在转换网站[v2ray转clash节点 - v2rayse.com](https://v1.v2rayse.com/v2ray-clash/) 把vmess订阅转换为clash配置文件，然后导入 clash即可
