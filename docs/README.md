# 万向报表（GimbalX Report）

基于 Band（带区）模型的 Web 报表设计器：浏览器里拖拽设计、连数据源、预览分页、导出 PDF / Excel / Word / RTF / SVG / HTML / CSV / JSON / XML，支持参数、子报表、交叉表、图表、定时任务与运行端只读部署。

- 前端：Vue 3 + TypeScript + Vite + Element Plus + Pinia
- 后端：FastAPI + SQLAlchemy 2.0(async) + ReportLab / openpyxl / python-docx
- 数据库：SQLite（单机）或 MySQL 8（服务器）
- 部署：Windows / Linux / macOS 单机免安装包（内置运行时）、单容器 Docker、Linux systemd、双机（设计端 + 运行端）

> English README: [README.en.md](README.en.md)

## 30 秒上手（Windows 单机版）

1. 下载 `GimbalX-Report-<版本>-win64.zip`，解压到任意目录（不要放 C:\Program Files）。
2. 双击 `start.bat`，浏览器打开 <http://127.0.0.1:8090>；同网段其他电脑用 `http://<本机IP>:8090`
   （默认已监听 0.0.0.0，Windows 防火墙需放行 TCP 8090；只给本机用就改 `.env` 的 `HOST=127.0.0.1`）。
3. 首次打开进入初始化向导：创建管理员 →（可选）导入授权 → 完成。
4. 需要开机自启：安装 [NSSM](https://nssm.cc/download) 后双击 `install-service.bat`。

> 试用版同一时间只允许**一个登录会话**（新登录会把旧会话顶下线，防账号共享）；
> 正式版按 `.lic` 里的 `seats`（并发席位）放行，席位满了新登录会被拒绝。任何版本下同一账号只保留一个会话。

Docker 单容器：

```bash
cp .env.single.example .env.single         # 修改 SECRET_KEY / CONFIG_ENCRYPTION_KEY
docker compose -f deploy/docker/docker-compose.single.yml --env-file .env.single up -d --build
# 打开 http://<主机>:8090
```

详细步骤见 [docs/install-single-windows.md](docs/install-single-windows.md)、[docs/install-single-docker.md](docs/install-single-docker.md)。

## 试用版与正式版

**同一份安装包、同一套代码**，区别只在授权状态：

| | 试用版（默认） | 正式版 |
|---|---|---|
| 功能 | 完整 | 完整 |
| 使用期限 | 不限 | 按授权 |
| 并发登录会话 | 1 个（新登录顶掉旧登录） | `.lic` 的 `seats`（默认 1，可签 3/5/不限），满了拒绝新登录 |
| 同一账号多设备 | 只保留最新会话 | 只保留最新会话 |
| 渲染页数 > 2 的报表 | 整份文档每页带 `[试用版 TRIAL]` 水印 | 无水印 |
| 渲染页数 ≤ 2 的报表 | 无水印 | 无水印 |
| 数据导出（CSV/JSON/XML） | 不受水印影响 | 不受水印影响 |

试用版足够完成真实报表的设计、联调与验证（2 页以内导出无水印，可直接给业务看样张）。
多人同时用时为每人建独立账号，并按人数购买席位；不要共用同一个 admin 账号（会被互相踢下线）。

### 升级到正式版（不重装、不重新下载）

1. 打开「系统设置 → 授权」，复制**机器码**（形如 `XXXX-XXXX-XXXX-XXXX`）。
2. 把机器码和购买信息发给作者（见下方微信二维码），收到 `.lic` 授权文件。
3. 在同一页面点「导入授权文件」，立即生效。

授权文件由作者用离线私钥（Ed25519）签名，程序内置公钥验签；换机或重装后用新机器码重新签发即可。授权可绑定多台机器（一份 `.lic` 内可含多个机器码），并用 `seats` 指定并发席位数（购买 5 人同时使用就签 `--seats 5`）。

离线签发不需要激活服务器，程序不会外连任何授权服务。发布包里的授权/会话校验核心是 Cython 编译的二进制模块，不提供可改的 `.py`。

<p align="center">
  <img src="frontend/public/assets/wechat-qr.png" alt="作者微信二维码" width="180">
  <br>
  <em>加作者微信咨询授权 / 获取正式版</em>
</p>

授权说明详见 [docs/licensing.md](docs/licensing.md)。

## 下载形态

| 形态 | 适用场景 | 目录/文件 |
|---|---|---|
| Windows 单机包 | 个人/小团队，Windows Server 或 Win10/11 | `GimbalX-Report-<版本>-win64.zip`（内置 Python + SQLite） |
| Linux 单机包（免安装） | 个人/小团队，Linux x86_64/aarch64，**不需要 Docker 和系统 Python** | `GimbalX-Report-<版本>-linux-x86_64.tar.gz`（内置 CPython + SQLite，含 systemd 脚本） |
| macOS 单机包（免安装） | 个人/小团队，macOS 12+（Apple Silicon/Intel），**不需要 Docker** | `GimbalX-Report-<版本>-macos-arm64.tar.gz`（内置 CPython，含 launchd 脚本；需在 macOS 上构建） |
| Docker 单容器 | 有 Docker 的 Linux/Windows，单机或小规模 | `deploy/docker/docker-compose.single.yml` |
| 服务器版（设计端 + 运行端） | 生产环境、MySQL 8 MGR、只读运行端 | `Dockerfile`、`docker-compose.prod.yml`、`deploy/linux/*` |

单机各形态都是**同一套代码**，可随时迁移到服务器版：把 `data/`（SQLite 或导出 SQL）和存储目录带走即可。
Linux 免安装包安装步骤见 [docs/install-single-linux.md](docs/install-single-linux.md)；
全部安装形态（含不用 Docker 的源码安装、macOS、Alpine）见 [docs/install-options.md](docs/install-options.md)。

## 功能概览

- 设计器：Band 布局、标尺吸附、多选/对齐、撤销重做、字典面板、表达式、条件样式、水印、版本管理
- 数据：MySQL / PostgreSQL / SQL Server / Oracle / ClickHouse / ODBC / CSV / Excel / JSON / XML，多数据源关联（DuckDB）
- 报表元素：文本、图片、线条、矩形、形状、表格、交叉表、图表、条码/二维码、子报表、数据标签
- 输出：分页预览、PDF（矢量）、Excel、Word、RTF、SVG、HTML、CSV / JSON / XML
- 运行端：只读部署、Embed 短期 Token、客户与授权管理、调用日志、定时任务、审计日志

## 目录结构

```
backend/          FastAPI 服务、渲染引擎、导出引擎、授权模块
frontend/         Vue 3 设计器与运行端
deploy/           Docker / Linux / Windows 部署脚本与单机版打包
samples/          可导入示例报表、嵌入页、REST 调用清单（node samples/serve.mjs 起样例站）
scripts/          本地开发、打包、发布树导出
docs/             安装、授权、部署文档
```

## 开发

```bash
# 后端（端口 8006，本仓库约定）
cd backend && pip install -e ".[dev]"
uvicorn app.main:app --reload --port 8006

# 前端（5173，代理 /api → 8006）
cd frontend && npm install && npm run dev
```

常见问题、接口说明见 `docs/`。二次开发请先阅读 [CONTRIBUTING.md](CONTRIBUTING.md)。

## 许可

源码可见，商业使用需获得正式授权；试用条款见 [LICENSE](LICENSE)。第三方组件许可见 [THIRD-PARTY-NOTICES.md](THIRD-PARTY-NOTICES.md)。

## 安全

漏洞请私下联系作者（不要开公开 Issue），见 [SECURITY.md](SECURITY.md)。