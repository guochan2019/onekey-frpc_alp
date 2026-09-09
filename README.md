# onekey-frpc_alp

一键在 **Alpine Linux** 上安装/升级/卸载 [frp](https://github.com/fatedier/frp) 客户端 (frpc)，自动获取最新版本 + 生成配置模板 + 创建 OpenRC 服务（`supervise-daemon` 崩溃自动拉起）。

> 功能与 [onekey-frpc](https://github.com/guochan2019/onekey-frpc)（Debian 直装版）完全一致，平台层适配 **apk + OpenRC**。部署目录相同（`/opt/frp`），Linux Gate 上的现成配置可直接迁移。

---

## 快速开始

> ⚠️ 需要 root 权限。适用 Alpine Linux（OpenRC）。

### 方式一：gh CLI（推荐）

```bash
gh repo clone guochan2019/onekey-frpc_alp
cd onekey-frpc_alp
chmod +x onekey-frpc_alp.sh
./onekey-frpc_alp.sh
```

### 方式二：wget

```bash
wget -qO- https://raw.githubusercontent.com/guochan2019/onekey-frpc_alp/main/onekey-frpc_alp.sh | sh
```

> 直连受限环境(网关 50.1 等)用 GitHub 镜像加速:
> ```bash
> wget -qO- https://gh-proxy.com/https://raw.githubusercontent.com/guochan2019/onekey-frpc_alp/main/onekey-frpc_alp.sh | sh
> ```

---

## 使用方式

运行脚本后显示菜单：

```
========================================
  frpc 一键安装/升级/卸载脚本 (Alpine)
  https://github.com/fatedier/frp
========================================

[INFO] 检测到 frpc 0.70.0 已安装

请选择操作：
  1. 安装 / 升级 frpc
  2. 卸载 frpc
  0. 退出
```

| 选项 | 功能 |
|------|------|
| **1** | 未安装 → 4 步完整安装；已安装 → 检测版本并升级 |
| **2** | 卸载：停止服务、删除二进制/配置 |
| **0** | 退出 |

## 安装流程

| 步骤 | 说明 |
|------|------|
| 检测 | 自动识别系统架构（amd64 / arm64 / arm / 386） |
| 1/4 | 下载并解压 frp，提取 frpc 二进制（Go 静态，musl 直接兼容） |
| 2/4 | 创建配置模板 `/opt/frp/frpc.toml`（幂等，不覆盖已有配置） |
| 3/4 | 创建 OpenRC 服务 `/etc/init.d/frpc`（supervise-daemon 自动重启） |
| 4/4 | 显示完成信息 |

---

## 目录结构

```
/opt/frp/
├── frpc.toml              # 配置文件（需手动编辑）
├── frpc.example.toml      # 官方示例配置（参考用）
├── conf.example/          # 官方完整示例（参考用）
└── frpc.log               # frpc 运行日志（自动生成，./ 相对工作目录 /opt/frp）

/usr/local/bin/frpc        # frpc 二进制
```

---

## 配置说明

安装完成后编辑 `/opt/frp/frpc.toml`，至少修改以下参数：

### 必填

| 参数 | 说明 | 示例 |
|------|------|------|
| `serverAddr` | frps 服务器 IP 或域名 | `"192.0.2.1"` |
| `serverPort` | frps 绑定的端口 | `7000` |
| `auth.token` | 认证令牌（与 frps 一致） | `"your-token"` |

### 代理示例（按需取消注释）

```toml
# SSH 隧道 — 暴露内网 SSH 到公网 6000 端口
[[proxies]]
name = "ssh"
type = "tcp"
localIP = "127.0.0.1"
localPort = 22
remotePort = 6000

# HTTP 服务 — 绑定域名
[[proxies]]
name = "web"
type = "http"
localIP = "127.0.0.1"
localPort = 80
customDomains = ["yourdomain.com"]
```

更多代理类型（stcp / xtcp / tcpmux / plugin 等）参考配置模板中的注释或 [frp 官方文档](https://github.com/fatedier/frp)。

---

## 服务管理（OpenRC）

```bash
# 启动 + 开机自启
rc-update add frpc default
rc-service frpc start

# 状态 / 重启 / 停止
rc-service frpc status
rc-service frpc restart
rc-service frpc stop

# 实时日志
tail -f /opt/frp/frpc.log

# 热加载（新增/修改代理，无需重启服务）
frpc reload

# 验证配置语法
frpc verify -c /opt/frp/frpc.toml
```

> 服务由 `supervise-daemon` 托管：进程异常退出自动拉起（对齐 systemd `Restart=on-failure`），日志相对路径经 `--chdir /opt/frp` 保证落在 `/opt/frp/frpc.log`。

### 升级 / 卸载

再次运行脚本选择对应选项即可：

```bash
sh onekey-frpc_alp.sh
# 选 1 → 升级；选 2 → 卸载
```

---

## 与 Debian 直装版差异

| 项 | Debian 版 | Alpine 版 |
|----|-----------|-----------|
| 服务管理 | systemd unit + `Restart=on-failure` | OpenRC init.d + `supervise-daemon`（等价自愈） |
| 服务命令 | `systemctl enable --now frpc` | `rc-update add frpc default && rc-service frpc start` |
| 工作目录 | systemd `WorkingDirectory=/opt/frp` | supervise-daemon `--chdir /opt/frp` |
| 部署目录 | `/opt/frp` + `/usr/local/bin/frpc` | **相同**（配置可迁移） |

---

## 架构支持

| 架构 | 支持 |
|------|------|
| x86_64 (amd64) | ✅ |
| aarch64 (arm64) | ✅ |
| armv7l | ✅ |

---

## 许可证

本项目基于 [GPL-3.0](LICENSE) 协议。frp 本身同样遵循 GPL-3.0。
