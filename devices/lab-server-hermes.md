# 实验室服务器 Hermes 常用命令

本文记录实验室服务器上 Hermes 的常用配置、VPN/VPS 代理和日常命令。只记录用法，不记录订阅链接、节点密码、VPS IP、SSH key、token 等敏感信息。

## 1. 快速结论

- 日常推荐：使用 Mihomo/Clash VPN，端口 `7890`。
- 不推荐：旧 VPS SSH 代理，端口 `7891`，当前状态经常不可用。
- 启动 Hermes 前，如需访问 GPT/OpenAI，先在同一个 shell 中加载代理环境变量。

```bash
bash ~/.hermes/vpn/start-vpn.sh
source ~/.hermes/vpn/proxy-env.sh
python3 ~/.hermes/vpn/switch-best-node.py
hermes
```

## 2. VPN / Mihomo 配置

- VPN 目录：`~/.hermes/vpn/`
- Mihomo 配置：`~/.hermes/vpn/config.yaml`，包含敏感节点信息，不要上传 GitHub
- 本地混合代理端口：`http://127.0.0.1:7890`
- Clash/Mihomo 控制 API：`http://127.0.0.1:9090`
- 加载代理脚本：`~/.hermes/vpn/proxy-env.sh`
- 取消代理脚本：`~/.hermes/vpn/unproxy-env.sh`

### 启动、状态、关闭

```bash
# 启动 VPN
bash ~/.hermes/vpn/start-vpn.sh
# 或使用别名
hermes-vpn-start

# 查看状态
bash ~/.hermes/vpn/status-vpn.sh
# 或使用别名
hermes-vpn-status

# 停止 VPN
bash ~/.hermes/vpn/stop-vpn.sh
# 或使用别名
hermes-vpn-stop
```

### 让当前 shell 使用 VPN

```bash
# 加载代理环境变量
source ~/.hermes/vpn/proxy-env.sh

# 取消当前 shell 的代理环境变量，不停止 Mihomo
source ~/.hermes/vpn/unproxy-env.sh
```

`proxy-env.sh` 会设置：

```bash
http_proxy=http://127.0.0.1:7890
https_proxy=http://127.0.0.1:7890
all_proxy=http://127.0.0.1:7890
```

## 3. VPN 节点检查和切换

```bash
# 查看当前选中的节点
curl --noproxy "*" -s \
  'http://127.0.0.1:9090/proxies/%E8%8A%82%E7%82%B9%E9%80%89%E6%8B%A9' \
  | python3 -c 'import sys,json; print(json.load(sys.stdin).get("now", "unknown"))'

# 测试当前代理出口 IP
curl -x http://127.0.0.1:7890 https://api.ipify.org

# 测试 ChatGPT 可达性
curl -x http://127.0.0.1:7890 -I https://chatgpt.com
```

自动选择节点：

```bash
# 使用缓存结果，切换到当前最快可用 GPT 节点
python3 ~/.hermes/vpn/switch-best-node.py

# 重新完整扫描节点后再切换，耗时较长
python3 ~/.hermes/vpn/switch-best-node.py --test

# 全量测试 GPT/OpenAI 可用节点，耗时约 10-15 分钟
python3 ~/.hermes/vpn/test-gpt-nodes.py
```

手动切换到指定节点：

```bash
curl --noproxy "*" -X PUT \
  'http://127.0.0.1:9090/proxies/%E8%8A%82%E7%82%B9%E9%80%89%E6%8B%A9' \
  -H 'Content-Type: application/json' \
  -d '{"name":"节点名称"}'
```

经验：如果 GPT-5.5 / OpenAI Codex 出现 `HTTP 403` 或 `Connection error`，优先尝试新加坡、韩国、台湾、日本节点；香港节点有时会被 OpenAI 风控。

## 4. VPN 自动故障切换

```bash
# 后台运行自动切换守护进程
nohup bash ~/.hermes/vpn/auto-failover.sh > /tmp/hermes-vpn-auto-failover.log 2>&1 &

# 查看日志
tail -f /tmp/hermes-vpn-auto-failover.log
```

作用：定期检测 ChatGPT 可达性；连续失败后自动切换到缓存中的可用节点。

## 5. VPS SSH 代理（旧方案 / 备用）

- VPS 代理目录：`~/.vps-proxy/`
- 加载代理脚本：`~/.vps-proxy/env.sh`
- 取消代理脚本：`~/.vps-proxy/unenv.sh`
- 本地代理端口：`http://127.0.0.1:7891`
- 当前状态：经常不可用，日志里常见 SSH timeout；日常不要优先使用

常用命令：

```bash
# 首次配置 SSH key，通常需要交互输入一次 VPS 密码
vps-proxy-setup-key

# 启动 VPS SSH 隧道代理
vps-proxy-start

# 查看 VPS 代理状态
vps-proxy-status

# 当前 shell 使用 VPS 代理
source ~/.vps-proxy/env.sh

# 取消当前 shell 的 VPS 代理环境变量
source ~/.vps-proxy/unenv.sh

# 停止 VPS 代理
vps-proxy-stop
```

重要提醒：

- `~/.vps-proxy/env.sh` 指向 `7891`，不是 Mihomo 的 `7890`。
- 当前推荐使用 `~/.hermes/vpn/proxy-env.sh`，不要误用旧 VPS 代理。
- 不要把 VPS IP、SSH key、日志、认证信息上传到 GitHub。

## 6. 当前 Hermes 关键配置摘要

配置文件：

```bash
~/.hermes/config.yaml
~/.hermes/.env
```

当前常用模型/执行设置：

```
主模型：openai-codex:gpt-5.5
主上下文：272000
compression 触发阈值：默认 50%；gpt-5.5/Codex 路由会自动抬到约 85%
压缩模型：deepseek:deepseek-v4-pro
agent.max_turns：180
goals.max_turns：100
reasoning_effort：xhigh
```

### 压缩模型说明

Hermes 默认 `auxiliary.compression.provider=auto`、`model=""`，并不是"切换到什么主模型就自动用什么模型做压缩"。compression（上下文压缩/摘要模型）应优先选稳定、长上下文、摘要可靠的模型。当前机器上 `openai-codex` 容易受 VPN/登录状态影响，推荐固定用 DeepSeek 做压缩。

```bash
# 推荐：固定用 DeepSeek 做上下文压缩
hermes config set auxiliary.compression.provider deepseek
hermes config set auxiliary.compression.model deepseek-v4-pro

# 如需恢复默认 auto
hermes config set auxiliary.compression.provider auto
hermes config set auxiliary.compression.model ""
```

修改配置后，在 Hermes 会话内执行 `/reset` 让其生效。

### 当前 MoA 配置

```
reference 1：deepseek:deepseek-v4-pro
reference 2：openai-codex:gpt-5.5
aggregator：openai-codex:gpt-5.5
```

说明：MoA 适合复杂审查、方案比较、代码/论文评估；简单任务不用一直开。`/moa` 可临时启用一轮，长期默认开启会更慢、更贵。

## 7. Hermes 常用命令

```bash
# 启动
hermes

# 查看/切换模型
hermes model

# 检查配置
hermes config check

# 查看配置路径
hermes config path

# 查看 MoA 配置
hermes moa list

# 重新登录 OpenAI Codex
hermes login --provider openai-codex

# 检查健康状态
hermes doctor

# 查看 skill
hermes skills list
```

交互会话内常用 slash command：

```
/reset              新会话，让新配置生效
/model              查看或切换模型
/goal status        查看长期目标状态
/goal resume        继续长期目标
/goal clear         清除长期目标
/footer off         关闭 final footer
/statusbar          切换状态栏
/verbose            切换详细日志级别
```

## 8. 常见坑

- `ALL_PROXY` 必须是 `http://127.0.0.1:7890`，不要改成 `socks5://`；当前 Hermes/httpx 环境没有 socksio。
- 查询本地 `9090` 控制 API 时要加 `--noproxy "*"`，避免请求被代理到 7890/7891。
- 测试外网代理时不要把 `--noproxy "*"` 和 `-x http://127.0.0.1:7890` 混用；`--noproxy "*"` 会绕过代理。
- `subscription.url`、`subscription.raw`、`config.yaml`、`mihomo.log` 都可能包含敏感信息，不要提交到 GitHub。
- `.env`、`auth.json`、VPS SSH key、VPS tunnel log 都不要提交到 GitHub。
- 系统 git 版本 1.8.3.1 导致 `hermes update` 误判已最新（`git fetch origin main` 只更新 FETCH_HEAD 不更新 origin/main），需手动 fetch/pull 或升级 git。