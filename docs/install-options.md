# 安装形态总览（含全部“不用 Docker”的方式）

万向报表（GimbalX Report） 是同一套代码，按你的机器和习惯任选一种；**不用 Docker 也能完整使用**，
单机版三种包都自带运行时，不需要系统 Python、pip 或 Node.js。

## 一、按操作系统选

| 系统 | 方式 | 需要 Docker？ | 需要 Python/Node？ | 产物 / 文档 |
|---|---|---|---|---|
| Windows 10/11、Server | **免安装 zip** | 否 | 否（包内自带 Python） | `GimbalX-Report-<版本>-win64.zip` → [install-single-windows.md](install-single-windows.md) |
| Windows | Docker Desktop 单容器 | 是 | 否 | `docker-compose.single.yml` → [install-single-docker.md](install-single-docker.md) |
| Linux x86_64 / aarch64（glibc） | **免安装 tar.gz** | 否 | 否（包内自带 CPython） | `GimbalX-Report-<版本>-linux-x86_64.tar.gz` → [install-single-linux.md](install-single-linux.md) |
| Linux（Alpine 等 musl） | 免安装 tar.gz（musl 构建） | 否 | 否 | `...-linux-x86_64-musl.tar.gz`（`--libc musl` 构建，实验性） |
| Linux | 源码安装（venv） | 否 | 是：Python 3.12 + Node 18+ | 见本文第三节 |
| Linux | Docker 单容器 | 是 | 否 | `docker-compose.single.yml` |
| macOS 12+（Apple Silicon/Intel） | **免安装 tar.gz** | 否 | 否（包内自带 CPython） | `...-macos-arm64.tar.gz` → [install-single-macos.md](install-single-macos.md)（需在 macOS 上构建） |
| macOS | Docker Desktop 单容器 | 是 | 否 | `docker-compose.single.yml` |
| 任意系统 | 源码安装 | 否 | 是：Python 3.12 + Node 18+ | 见本文第三节 |

> 免安装包与 Docker 镜像内容一致（同版本同代码）；单机数据在 `data/`，换形态时把
> `data/` 与 `.env` 带过去即可，无需导出导入。

## 二、单机版三种“不用 Docker”的包

| 包 | 启动 | 开机自启 | 自带运行时 |
|---|---|---|---|
| Windows zip | 双击 `start.bat` | `install-service.bat`（NSSM） | CPython 3.12（embeddable） |
| Linux tar.gz | `./start.sh` | `sudo ./tools/install-service.sh`（systemd） | CPython 3.12（python-build-standalone） |
| macOS tar.gz | `./start.sh` | `./tools/install-service.sh`（launchd） | CPython 3.12（python-build-standalone） |

系统要求：Windows 10/11 x64；Linux glibc ≥ 2.17（CentOS 7 / Ubuntu 16.04+）；macOS 12+。
免安装包默认监听 `0.0.0.0:8090`，同网段浏览器可直接用 `http://<主机IP>:8090` 打开（放行防火墙即可）；
只想本机用就改 `.env` 的 `HOST=127.0.0.1`；对公网请用反向代理（nginx/caddy）。
**试用版同一时间只允许一个登录会话**（新登录会把旧会话顶下线）；正式版按 `.lic` 的 `seats` 限制并发会话数，席位用满后新登录被拒绝。任何版本下同一账号只保留一个会话。

## 三、源码安装（任何系统，不用 Docker）

适合已有 Python 3.12 和 Node 18+ 的开发者，或需要改代码的场景。

```bash
# 1. 前端构建（生成 frontend/dist）
cd frontend
npm ci
npm run build

# 2. 后端依赖（独立虚拟环境，避免污染系统 Python）
cd ../backend
python3.12 -m venv .venv
. .venv/bin/activate                 # Windows: .venv\Scripts\Activate.ps1
pip install -r ../deploy/single/requirements-single.txt

# 3. 配置
cp ../.env.single.example .env       # 编辑 SECRET_KEY / CONFIG_ENCRYPTION_KEY / 端口
#   SERVE_FRONTEND=true
#   FRONTEND_DIST_PATH=../frontend/dist

# 4. 初始化并启动
python -m app.cli db-upgrade
python -m uvicorn app.main:app --host 127.0.0.1 --port 8090
```

开机自启（源码方式）可参考单机包的 systemd/launchd 单元，把 `ExecStart` 换成
`/path/to/backend/.venv/bin/python -m uvicorn app.main:app --host 127.0.0.1 --port 8090`，
`WorkingDirectory` 指向 `backend/`。

> Windows 源码方式把 `python3.12` 换成 `py -3.12`，激活脚本用 `.venv\Scripts\Activate.ps1`。

## 四、服务器版（多人使用）

- **Docker 方式**：`docker-compose.prod.yml`（设计端 + 运行端 + MySQL 8 MGR），见 `docs/deployment-runbook.md`；
- **不用 Docker**：同一套后端可直接以 systemd 常驻 + 外部 MySQL 8 + nginx 反代运行：
  安装依赖同上（`requirements-single.txt` 已含 MySQL 驱动），`.env` 设 `DB_TYPE=mysql` 与连接参数，
  `SERVE_FRONTEND=true`，运行端另起一个实例并设 `APP_MODE=runtime`。生产级一键脚本本期未提供。

## 五、怎么选

| 你的情况 | 建议 |
|---|---|
| 只有一台 Windows，点开就用 | Windows 免安装 zip |
| 只有一台 Linux 服务器，不想装 Docker | Linux 免安装 tar.gz |
| 公司禁用 Docker，机器已有 Python | 源码安装 |
| 用 Mac 开发/演示 | macOS 包（需 macOS 构建）或 Docker Desktop |
| 多人访问、要 HTTPS、要只读运行端 | 服务器版（Docker 或原生 systemd + MySQL） |

授权在以上所有形态中通用：试用版渲染 > 2 页带水印；导入 `.lic` 即转正式版，见
[licensing.md](licensing.md)。