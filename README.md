# 设备使用说明

## 1. VPN / 代理

**启动代理：**
```
verge-mihomo -f /tmp/vpn_clash.yaml -d /tmp &
```

**更新订阅（供应商端口会变动，连不上时先执行这个）：**
```
curl -sL -A "clash-verge/2.5.1" "https://cat.cn-ping.com/sabusuku?token=38d1175f71be3f3ff6c203b489d81219" -o /tmp/vpn_clash.yaml
```

**停止代理：**
```
pkill -f verge-mihomo
```

代理端口：`127.0.0.1:7890`（HTTP + SOCKS5）

---

## 2. SSH 远程连接

```
ssh shuai@192.168.110.161
```

---

## 3. 远程桌面

### xrdp（已配置）
端口 3389，用 Windows 远程桌面或 Remmina 连接 `192.168.110.161`

### RayLink
桌面应用，需手动打开（不需要 VPN）

---

## 4. 常用命令

```
# 系统负载
uptime

# 查看端口占用
ss -tlnp | grep -E '7890|3389|22'

# 查看进程
ps aux | grep -E 'verge|xray|trojan'
```
