<div align="center">
  <img src="cmd/meshlan/assets/meshlan-icon.png" width="96" alt="MeshLAN logo">
  <h1>MeshLAN</h1>
  <p><strong>简体中文</strong> · <a href="README.en.md">English</a> · <a href="README.ja.md">日本語</a></p>
  <p><strong>以 P2P 为主、Relay 兜底的自托管虚拟局域网与本地服务协作平台</strong></p>
  <p>
    <img alt="Go" src="https://img.shields.io/badge/Go-1.26-00ADD8?logo=go&logoColor=white">
    <img alt="Nebula" src="https://img.shields.io/badge/Nebula-1.11-315C77">
    <img alt="Windows client" src="https://img.shields.io/badge/Client-Windows-0078D4?logo=windows">
    <img alt="Linux server" src="https://img.shields.io/badge/Server-Linux-FCC624?logo=linux&logoColor=111">
    <img alt="License" src="https://img.shields.io/badge/License-MIT-2E7D65">
  </p>
</div>

MeshLAN 在 [SlackHQ Nebula](https://github.com/slackhq/nebula) 之上提供自动配对、
P2P/NAT 优化、多 Lighthouse/Relay、服务映射、MeshDNS、文件直传、访问审批、
实时拓扑、历史回放、安全更新和可选 AI 自动化。Windows 用户只需运行一个便携
客户端；控制面和中继节点由你自己的 Linux 服务器承载。

> 本仓库包含 Windows 客户端、Linux 主控制服务器和 Linux 子节点的完整源码。
> 截图来自真实原生客户端，设备标识、IP、IPv6、域名和公网端点均已遮挡。

## 截图

### 实时网络拓扑

![MeshLAN real-time topology](docs/screenshots/topology.png)

### 历史与回放

![MeshLAN history and replay](docs/screenshots/history.png)

### AI 助手

![MeshLAN AI assistant](docs/screenshots/ai-assistant.png)

## 核心能力

- **虚拟局域网**：基于 Nebula v2 证书和加密隧道连接异地 Windows 设备。
- **P2P 优先**：双栈打洞、稳定 UDP 端口、UPnP、STUN 诊断与 Route Guard。
- **Relay 兜底**：直连失败时自动中继；多节点根据延迟、抖动、丢包和健康状态择优。
- **多节点控制面**：主 Lighthouse 管理 Linux 子 Lighthouse/Relay，支持健康检查与故障切换。
- **服务映射**：把 `localhost` TCP/UDP 服务发布给同一 Mesh，支持自定义端口、暂停和健康检查。
- **访问审批**：服务可设置直接访问或手动批准；创建者只能管理自己的映射和授权。
- **MeshDNS**：设备使用 `alice.mesh`，服务使用 `api.alice.mesh` 等稳定名称。
- **HTTPS 网关**：为 `.mesh` HTTP 服务提供本地 CA、证书签发和无端口访问入口。
- **文件直传**：文件内容通过设备间加密链路传输，控制服务器只分发元数据。
- **实时可观测性**：拓扑、P2P/Relay 路径、Underlay、延迟、服务健康和实时字节动画。
- **按用户 Token 统计**：从模型响应的真实 `usage` 累计输入、输出、缓存与推理 Token，不读取对话正文。
- **历史与回放**：本地 SQLite 保留流量、路径、连接和事件；支持时间滑块与筛选。
- **AI 助手**：模型密钥只在服务端；上下文和结果端到端加密；所有修改必须由用户确认。
- **安全更新**：Ed25519 签名清单、SHA-256 校验、可选 Authenticode、健康检查和自动回滚。
- **多语言界面**：支持简体中文、繁體中文、English、日本語；界面与 AI 助手回复同步切换并在本机保存。

## 把异地设备变成一个局域网

MeshLAN 的核心不是把某一个 API 暴露到公网，而是让不同运营商、不同城市、不同 NAT
后的 Windows 设备加入同一个加密局域网。每台设备获得稳定的 Mesh IP，应用可以像访问
同一台路由器下的电脑一样访问其他设备；现有家庭网络、公司网络和默认网关不会被替换。

- 设备之间优先使用加密 P2P 直连，打洞失败才自动切换到可用 Relay；
- IPv4、IPv6 或智能双栈可按设备选择，Route Guard 避免全局代理/TUN 抢走 MeshLAN 链路；
- 拓扑图实时展示在线设备、实际 P2P/Relay 路径、延迟、流量和链路变化；
- 设备离线、换网或公网端点变化后会重新注册和建链，不需要重新分配 Mesh IP；
- 同一 Mesh 内可继续使用 ping、远程桌面、数据库、游戏服务器和自建 API 等现有协议。

```text
Windows A ── encrypted P2P ── Windows B
    │                              │
10.77.0.2                      10.77.0.3
    └──── Relay fallback（仅直连失败时）────┘
```

控制服务器负责设备授权、证书和发现信息，不承载正常的 P2P 业务数据。即使走 Relay，
中继看到的也只是 Nebula 加密数据包。

## MeshDNS：用名称访问设备和服务

Mesh IP 适合底层路由，人更适合记名称。MeshDNS 为每个用户/设备维护可修改的唯一前缀，
并根据当前设备和服务状态实时生成记录，无需在每台电脑上手写 `hosts` 文件。

假设设备前缀是 `alice`：

| 目标 | MeshDNS 名称 | 示例用途 |
|---|---|---|
| 设备本身 | `alice.mesh` | ping、RDP、SMB 或其他任意协议 |
| HTTP 服务 | `chat.alice.mesh` | 通过本地网关无端口访问 Web 服务 |
| 普通 TCP 服务 | `db.alice.mesh:5432` | 数据库、API 或自定义 TCP 协议 |
| UDP 服务 | `game.alice.mesh:27015` | 游戏服务器或其他 UDP 服务 |

用户可以修改自己的设备前缀，也可以为每个已发布服务单独修改三级前缀。创建、编辑、
暂停或删除映射后，服务目录与 MeshDNS 记录会随真实状态更新；名称冲突会在保存前提示，
不会静默指向错误设备。只有 HTTP/HTTPS 网关模式可以省略端口，普通 TCP/UDP 服务仍需
使用映射端口。

## 服务映射：共享 localhost，而不是暴露公网端口

很多开发工具只监听 `localhost`，例如本地 API、Web 面板、数据库或游戏服务器。
MeshLAN 可以把它发布到虚拟局域网，不需要在家庭路由器做端口转发，也不需要把服务
直接暴露给整个互联网。

创建映射时填写：

1. 服务名称，例如 `Chat API`；
2. 本地地址与端口，例如 `localhost` 和 `4571`；
3. TCP 或 UDP；
4. 自动分配或自定义 Mesh 端口；
5. MeshDNS 服务前缀，例如 `chat`；
6. 直接访问或需要创建者批准。

发布完成后，同一 Mesh 的成员会在“全网共享服务”中实时看到服务所有者、名称、协议、
健康状态、访问地址和端口。例如本机的 `localhost:4571` 可以发布为：

```text
http://chat.alice.mesh
# 或普通端口模式
chat.alice.mesh:20000
```

映射支持同时创建多个 TCP/UDP 服务。端口冲突会在创建前返回明确提示；创建者可以暂停
整个映射、暂停某个获准用户、查看当前连接者和流量，并随时恢复。需要审批的服务会生成
访问申请，创建者可在消息列表中同意或拒绝。所有用户都能查看共享目录，但只能修改或
删除自己创建的映射。

```text
peer request
    │
    ▼
chat.alice.mesh:20000
    │  MeshDNS + access policy
    ▼
alice (10.77.0.2)
    │  local encrypted forwarding
    ▼
localhost:4571
```

## 架构

```mermaid
flowchart LR
  subgraph Endpoints[Windows endpoints]
    A[Windows Client A\nWebView2 + Nebula]
    B[Windows Client B\nWebView2 + Nebula]
    C[Windows Client C\nWebView2 + Nebula]
  end

  subgraph Control[Self-hosted control plane]
    M[Main Linux Server\nEnrollment + Admin + Lighthouse + Relay]
    N1[Linux Child Node\nLighthouse + Relay]
    N2[Linux Child Node\nLighthouse + Relay]
  end

  A <-->|preferred encrypted P2P| B
  B <-->|preferred encrypted P2P| C
  A -.->|discovery / heartbeat| M
  B -.->|discovery / heartbeat| M
  C -.->|discovery / heartbeat| M
  M <-->|health + signed revocations| N1
  M <-->|health + signed revocations| N2
  A -.->|fallback relay| N1
  C -.->|fallback relay| M
```

### 组件

| 组件 | 平台 | 职责 |
|---|---|---|
| Windows Client | Windows 10/11 | 原生桌面窗口、配对、Nebula 服务、Route Guard、服务映射、MeshDNS、文件直传、AI 助手 |
| Main Server | Linux amd64/arm64 | 管理台、设备授权、证书签发、吊销、更新分发、主 Lighthouse/Relay、AI Provider |
| Child Node | Linux amd64/arm64 | 第二 Lighthouse/Relay、健康上报、吊销同步和多节点故障切换 |

数据流量尽可能直接在客户端之间传输。控制服务器不代理普通 P2P 业务；仅当直连不可用
并启用 Relay 时才转发 Nebula 加密数据包。

## 技术栈

| 层 | 技术 |
|---|---|
| 语言 | Go 1.26，少量 PowerShell、Bash、原生 JavaScript |
| Overlay | SlackHQ Nebula 1.11，Noise 握手，v2 证书，Lighthouse/Relay/Punchy |
| Windows UI | WebView2 原生窗口，嵌入式 HTML/CSS/JavaScript，无外部前端 CDN |
| 可视化 | SVG 拓扑、实时吞吐与历史双层曲线 |
| 持久化 | SQLite（WAL），JSON 状态，Windows DPAPI |
| 密码学 | TLS 1.3 pin、X25519、AES-256-GCM、Ed25519、SHA-256 |
| Windows 集成 | SCM、IP Helper API、WFP、计划任务、原生路由和证书库 |
| Linux 集成 | systemd、`LoadCredential`、文件权限与原子升级 |
| AI | OpenAI-compatible Chat Completions，SSE 流式输出，服务端工具白名单 |
| CI | GitHub Actions，Windows/Linux 测试、vet 和多平台构建 |

## 仓库结构

```text
.
├── cmd/meshlan/               # MeshLAN 可执行程序源码
│   ├── admin/                 # Linux 主服务端管理台
│   ├── assets/                # 图标与 Windows manifest
│   ├── deploy/                # Linux 子节点安装/升级和 systemd 配置
│   ├── web/                   # Windows 客户端 UI
│   ├── *_windows.go           # Windows 客户端与系统集成
│   ├── server.go              # 主服务端和管理 API
│   ├── node_agent.go          # Linux 子节点
│   ├── multinode.go           # 多 Lighthouse/Relay 管理
│   ├── ai_*.go                # AI 加密、流式调用、会话和工具执行
│   └── history.go             # SQLite 历史与采样
├── docs/                      # 文档与脱敏截图
├── tools/                     # 诊断脚本
├── build.ps1                  # 测试与跨平台构建
└── VERSION
```

## 快速开始

### 构建要求

- Go 1.26+
- Windows 10/11（构建和运行客户端）
- Microsoft Edge WebView2 Runtime
- Nebula 与 `nebula-cert` 1.11.x
- Linux 主机（运行主服务端和可选子节点）

### 构建全部组件

在 Windows PowerShell 中：

```powershell
git clone https://github.com/zhaoxuya520/MeshLAN.git
cd MeshLAN
.\build.ps1
```

输出位于 `dist/`：

```text
MeshLAN-Nebula-Windows.exe
meshlan-nebula-server-linux-amd64
meshlan-nebula-server-linux-arm64
meshlan-nebula-node-linux-amd64
meshlan-nebula-node-linux-arm64
SHA256SUMS.txt
```

也可以直接运行：

```powershell
go test ./...
go vet ./...
```

### 初始化主服务端

以下仅为示例；请替换公网地址、端口和文件路径：

```bash
sudo install -d -m 0700 /etc/meshlan-nebula
sudo install -m 0755 meshlan-nebula-server-linux-arm64 /usr/local/bin/meshlan-nebula

sudo /usr/local/bin/meshlan-nebula server init \
  -state /etc/meshlan-nebula/server-state.json \
  -endpoint 203.0.113.10:4242 \
  -subnet 10.77.0.0/24 \
  -nebula-port 4242 \
  -pairing-port 8443 \
  -nebula /usr/local/bin/nebula \
  -nebula-cert /usr/local/bin/nebula-cert
```

初始化命令只显示一次配对哈希和管理令牌。立即保存到密码管理器；不要写入仓库、Shell
历史、工单或日志聚合系统。随后使用 systemd 运行：

```bash
sudo /usr/local/bin/meshlan-nebula server serve \
  -state /etc/meshlan-nebula/server-state.json
```

需要放行：

- Nebula UDP 端口（示例 `4242/udp`）；
- 配对/管理 HTTPS 端口（示例 `8443/tcp`）。

生产环境应使用反向代理、有效域名证书、最小化防火墙、专用 systemd 用户、
`LoadCredential` 和离线备份。示例 credential drop-in 位于
`cmd/meshlan/deploy/systemd/20-credentials.conf`。

### 运行 Windows 客户端

```powershell
.\dist\MeshLAN-Nebula-Windows.exe
```

客户端首次运行时输入：

1. 本机设备名称；
2. 主服务端公网 IP 或域名；
3. 服务端创建的 `MLN1...` 一次性授权哈希。

客户端在本机生成 Nebula 私钥，领取证书后安装 `Nebula` Windows 服务。关闭桌面窗口
不会断开虚拟网络。

### 添加 Linux 子节点

首次安装：

```bash
export MESHLAN_RELEASE_BASE_URL=https://your-meshlan.example
curl -fsSL "$MESHLAN_RELEASE_BASE_URL/download/node/install.sh" -o /tmp/install-mesh-node.sh
sudo -E bash /tmp/install-mesh-node.sh \
  --endpoint 198.51.100.20 \
  --name edge-01
```

脚本输出 `MLNODE1...` 哈希。登录主服务端管理台，将子节点地址和哈希加入多节点页面。

原地升级：

```bash
export MESHLAN_RELEASE_BASE_URL=https://your-meshlan.example
curl -fsSL "$MESHLAN_RELEASE_BASE_URL/download/node/upgrade.sh" | sudo -E bash
```

升级脚本会校验 SHA-256、保留节点身份、重启健康检查，并在失败时回滚。

## AI 助手

AI 是可选功能。管理员可配置任意 OpenAI-compatible HTTPS 地址和模型。模型 API 密钥
只保存在主服务器的加密状态中，不下发客户端。

客户端与服务端使用 X25519 派生会话密钥，并以 AES-256-GCM 加密每次请求、流式增量和
最终计划。AI 只能调用白名单工具；任何修改操作都必须先在客户端展示计划，并由用户
明确确认。软件展示可审计工作摘要，不展示或声称提供模型隐藏思维链。

## 安全设计

- 设备私钥在端点生成，不上传控制服务器。
- 一次性授权哈希绑定设备名称和公钥。
- 服务端 CA、TLS、签名、TOTP 与 AI 密钥使用 AES-256-GCM envelope 保存。
- Windows 设备令牌使用当前用户 DPAPI。
- 吊销列表和更新清单由独立 Ed25519 密钥签名。
- Route Guard 脚本安装到受保护目录，并在高权限写入前验证固定 SHA-256。
- 管理台使用短会话，可启用 TOTP。
- 更新包校验签名、长度和 SHA-256；失败或健康检查超时自动回滚。
- 本机管理 API 仅监听 `127.0.0.1`。

请阅读 [SECURITY.md](SECURITY.md)。自托管并不等于自动安全：公网端口、防火墙、系统
更新、管理员终端、备份和代码签名仍由部署者负责。

## 隐私

- 普通 P2P 业务不经过控制服务器。
- Relay 只转发 Nebula 加密数据包。
- 文件内容不上传控制面。
- 历史数据库保存在本机，默认保留 30 天。
- 服务连接历史不记录报文内容。
- 故障报告和截图应在提交前移除地址、域名、令牌和个人身份信息。

## 参与贡献

请阅读 [CONTRIBUTING.md](CONTRIBUTING.md)。欢迎提交：

- Windows/IPv6/NAT 兼容性修复；
- 多 Relay 质量策略；
- Linux 服务部署与容器化；
- 可访问性、国际化和文档；
- 安全审计和回归测试。

## 许可证

[MIT License](LICENSE)

## 致谢

- [SlackHQ Nebula](https://github.com/slackhq/nebula)
- [WebView2](https://developer.microsoft.com/microsoft-edge/webview2/)
- [modernc.org/sqlite](https://pkg.go.dev/modernc.org/sqlite)
