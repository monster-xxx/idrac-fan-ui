✨ 核心特性
📊 可视化运行看板：并发拉取并展示实时最高温度、风扇均速 (RPM) 以及整机实时功耗 (Watts)。

🛡️ 24/7 智能高温守护：独立的后台守护线程（每 30 秒轮询）。当检测到任意核心温度超过设定的安全阈值（如 70°C）时，瞬间解除手动降噪，强制交还 iDRAC 官方全速接管，杜绝积热宕机风险。

🌐 双模连接引擎：支持 LANPLUS 网络模式（跨设备远程控制）与 /dev/ipmi0 本地直通模式（宿主机极速通信）。

💾 配置防丢持久化：结合 Docker Volume 与 Browser LocalStorage，无论容器重启还是网页刷新，你的账号密码与守护设置永远在线。

📦 群晖 DSM 原生融合：提供底层 Python 打包脚本，一键生成符合 DSM 7.3 极其严格校验规范的 .spk 安装包，实现桌面级窗口化体验。

🛠️ 前置准备工作
在开始部署之前，请确保你的 Dell 服务器已开启 IPMI 网络访问权限：

登录 iDRAC 官方 Web 管理页面。

导航至 iDRAC 设置 -> 网络 -> IPMI 设置。

勾选 启用通过 LAN 的 IPMI (Enable IPMI over LAN) 并应用。

记下你的 iDRAC IP 地址、管理员账号和密码。

🚀 Docker 手把手部署教程
推荐使用 Docker 方式部署，支持任意 Linux 宿主机（Ubuntu, Debian, CentOS 等）或群晖 NAS 环境。

方式一：使用 Docker CLI（推荐快速上手）
步骤 1：创建本地数据挂载目录
为了保证你的配置和守护进程数据不丢失，先在宿主机创建一个数据文件夹。

Bash
mkdir -p /path/to/your/data/idrac-fan-ui-data
步骤 2：一键启动容器
(如果你不需要“本地直通模式”，可以去掉 --device /dev/ipmi0:/dev/ipmi0 参数)

Bash
docker run -d \
  --name idrac-fan-ui \
  -p 18080:8080 \
  --device /dev/ipmi0:/dev/ipmi0 \
  -v /path/to/your/data/idrac-fan-ui-data:/app/data \
  --restart unless-stopped \
  monster-xxx/idrac-fan-ui:latest 
  # 注：若你自行构建了镜像，请替换为你自己的镜像名，如 idrac-fan-ui:latest
方式二：使用 Docker Compose
在任意目录下创建 docker-compose.yml 文件：

YAML
version: '3.8'
services:
  idrac-fan-ui:
    image: idrac-fan-ui:latest # 请替换为实际镜像名
    container_name: idrac-fan-ui
    ports:
      - "18080:8080"
    volumes:
      - ./data:/app/data
    devices:
      - /dev/ipmi0:/dev/ipmi0 # 仅在需要本地模式时保留
    restart: unless-stopped
运行启动命令：

Bash
docker-compose up -d
📖 Web UI 使用指南
容器启动成功后，打开浏览器访问：http://<宿主机IP>:18080

1. 建立连接
在节点连接参数区域，选择你的模式（推荐使用 网络模式，最稳定且无驱动权限问题）。

输入 iDRAC 的 IP、用户名和密码。这些数据会同时保存在你的浏览器和后端实体文件中。

2. 模式说明
🟢 全自动 (iDRAC 托管)：将风扇控制权交还给 Dell 官方主板，主板会根据负载自动变速（噪音较大）。

🔵 安静模式 (固定 15%)：深夜无感降噪模式，强制将所有风扇锁定在极低转速，适合轻负载的 HomeLab 玩家。

🟠 锁定指定转速：拖动滑块，精细控制 5% - 100% 的任意转速。

3. 配置安全守护（非常重要！）
在 散热模式切换 模块中，输入你期望的守护阈值（建议 70°C）。
必须勾选 “启用后台自动全速保护” 并点击 “应用并保存”。
此时，你的设置将被推送到后端 Python 进程。无论你是否关闭网页，哪怕容器在后台默默运行，只要服务器核心温度飙升超过阈值，它都会瞬间执行“自救”，把风扇拉到全速。

❓ 常见问题排查 (FAQ)
Q1：为什么“实时功耗”显示为 N/A？

A：功耗数据的抓取依赖于硬件支持。如果你的 Dell 服务器（如基础款 R230）配备的是单体非冗余电源（不带热插拔和 PMBus 芯片），iDRAC 硬件本身是无法测量功耗的，因此无法显示。热插拔双电源机型通常可正常显示。

Q2：提示 Error: Unable to establish IPMI v2 / RMCP+ session？

A：通常是因为 iDRAC 的 IP、账号或密码填写错误，或者在 iDRAC 控制台中没有开启 IPMI over LAN 权限。
