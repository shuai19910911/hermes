# 实验室台式linux

IP: 192.168.110.161 | 用户: shuai | 公网: 210.79.151.26

## VPS 代理

VPS: root@95.169.2.30 (SSH免密)

```bash
vps-on      # 启动隧道+开启代理
vps-off     # 关闭隧道+关闭代理
vps-status  # 查看状态
```

## SSH

```
ssh shuai@192.168.110.161
```

## 远程桌面

xrdp 端口 3389，域名 shuaibaba.store

## 相关服务

| 服务 | 说明 |
|------|------|
| Clash Verge | GUI代理管理(托盘) |
| cloudflared | ~/.local/bin/ 隧道 |
