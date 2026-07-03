# 实验室台式linux

IP: 192.168.110.161 | 用户: shuai

## VPN / 代理

启动代理（Clash Meta，端口 7890）：

```
verge-mihomo -f /tmp/vpn_clash.yaml -d /tmp &
```

更新订阅（连不上时执行）：

```
curl -sL -A "clash-verge/2.5.1" "订阅链接" -o /tmp/vpn_clash.yaml
```

停止代理：

```
pkill -f verge-mihomo
```

## SSH

```
ssh shuai@192.168.110.161
```

## 远程桌面

xrdp 端口 3389，RayLink 桌面应用。

## 安装的代理工具

| 工具 | 路径 |
|------|------|
| sing-box | ~/.local/bin/sing-box |
| xray | ~/.local/bin/xray |
| trojan-go | ~/.local/bin/trojan-go |
| verge-mihomo | /usr/bin/verge-mihomo |
