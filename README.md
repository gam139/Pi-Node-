# Pi-Node-
Docker、控制面板、VPN代理与内网穿透
部署 Pi Node 无论是在 Linux 还是 Windows 平台，核心逻辑高度一致。本文总结了完整的安装流程，涵盖容器环境、控制面板、网络代理与内网穿透四大模块，适用于希望构建稳定节点的技术用户。

🧱 安装结构概览
阶段	组件	作用	是否必须
1	Docker	容器运行环境	✅ 是
2	Pi Node	控制面板与容器编排	✅ 是
3	VPN代理	网络访问代理	⚠️ 视地区而定
4	frpc	内网穿透与远程访问	❌ 可选
🟢 第一阶段：安装 Docker（容器运行环境）
Pi Node 的核心服务运行在 Docker 容器中，需先安装并启动 Docker。

Linux 安装命令
bash
sudo apt-get update
sudo apt-get install docker.io
sudo systemctl enable docker
sudo systemctl start docker
Windows 安装方式
安装 Docker Desktop

启用 WSL2 和 Hyper-V 支持

确保 Docker 服务已启动

🟡 第二阶段：安装 Pi Node 控制面板
控制面板负责容器编排、身份验证、日志管理等功能。

Linux 安装方式
APT 安装（推荐）
bash
# 添加密钥
curl -fsSL https://apt.minepi.com/repository.gpg.key | sudo gpg --dearmor -o /etc/apt/keyrings/pinetwork-archive-keyring.gpg
sudo chmod a+r /etc/apt/keyrings/pinetwork-archive-keyring.gpg

# 添加软件源
echo 'deb [arch=amd64 signed-by=/etc/apt/keyrings/pinetwork-archive-keyring.gpg] https://apt.minepi.com stable main' | sudo tee /etc/apt/sources.list.d/pinetwork.list

# 更新索引并安装
sudo apt-get update
sudo apt-get install pi-node
⚠️ 当前 apt.minepi.com 仓库可能返回 403 Forbidden，建议等待官方恢复或使用 .deb 包手动安装。

手动安装（备用）
bash
wget https://apt.minepi.com/pool/main/p/pi-node/pi-node_<version>.deb
sudo dpkg -i pi-node_<version>.deb
Windows 安装方式
下载并运行 Pi Node 安装程序

安装完成后启动控制面板并登录

🔵 第三阶段：配置 VPN 网络代理
部分地区访问 Pi Network 节点受限，需配置代理绕过网络封锁。

推荐方案
Windows：使用 v2rayN + VLESS

Linux：使用 v2ray-core + systemd 服务

示例配置（v2rayN）
json
{
  "inbounds": [
    {
      "port": 10810,
      "listen": "0.0.0.0",
      "protocol": "mixed",
      "settings": {
        "auth": "noauth",
        "udp": true
      }
    }
  ],
  "outbounds": [
    {
      "protocol": "vless",
      "settings": {
        "vnext": [
          {
            "address": "your.vpn.server",
            "port": 443,
            "users": [
              {
                "id": "uuid",
                "encryption": "none"
              }
            ]
          }
        ]
      },
      "streamSettings": {
        "network": "ws",
        "wsSettings": {
          "path": "/yourpath",
          "headers": {
            "Host": "your.vpn.host"
          }
        }
      }
    }
  ]
}
Linux 设置 APT 代理
bash
echo 'Acquire::http::Proxy "http://<windows-ip>:10810";' | sudo tee /etc/apt/apt.conf.d/99proxy
🟣 第四阶段：配置 frpc 内网穿透（可选）
如果需要让外部设备访问你的节点服务（如 Web 控制台），可使用 frpc 实现内网穿透。

frpc 配置示例
ini
[pi-node]
type = tcp
local_port = 31400
remote_port = 31400
启动 frpc
bash
./frpc -c frpc.ini
📌 建议使用稳定的 frps 服务端，如 gam139 提供的公网穿透服务。

🧠 总结与建议
Pi Node 的部署逻辑在 Linux 与 Windows 平台高度一致

Docker 是基础，控制面板是核心，VPN 是保障，frpc 是扩展

当前官方仓库访问受限，建议等待恢复或使用手动安装方案

所有配置建议模块化、可复用，适合写入 GitHub Pages 技术文档
