# GimbalX Report（万向报表）v1.0.0

> 本版起产品更名为 **GimbalX Report / 万向报表**（原 qRptDesigner）。功能与授权体系不变：
> 安装包、目录、服务名、授权编号前缀随之更新，详见文末「更名与兼容」。

首个公开版本：单机版 / Docker 单容器 / 服务器版共用同一套代码，试用与正式版只差一份授权文件。
单机版提供 **Windows 与 Linux 两种“不用 Docker”的免安装包**，macOS 构建脚本随源码提供。

## 下载 / Downloads

| 文件 | 说明 |
|---|---|
| `GimbalX-Report-1.0.0-win64.zip` | Windows 10/11 x64 免安装包（内置 Python 运行时，79.4 MB） |
| `GimbalX-Report-1.0.0-linux-x86_64.tar.gz` | Linux x86_64 免安装包（内置 CPython 运行时，156.8 MB，无需 Docker/系统 Python） |
| `docker-compose.single.yml` | Docker 单容器部署文件（配合发布的镜像标签使用） |
| `SHA256SUMS` | 校验值：`sha256sum -c SHA256SUMS`（Windows：`Get-FileHash` 对比） |

## 本次重建（2026-11-02）

- 修正依赖缺失：`python-barcode`、`qrcode[pil]` 之前未声明，导致**条码 / 二维码导出退化成纯文本**；
  现已写入 `backend/pyproject.toml` 与 `deploy/single/requirements-single.txt`，两个免安装包均已重建并实测
  导出 PDF 内嵌位图（Windows 包与 Linux 包各自验证）。
- 打包时不再生成第三方字节码缓存：整包 `.pyc` 数量为 0（pip 加 `--no-compile` + 打包前整包清理校验）。
- Docker 单容器镜像同步改为编译授权核心（镜像内无明文 `.py`）。

## Windows 单机版（不用 Docker）

1. 解压 `GimbalX-Report-1.0.0-win64.zip` 到任意可写目录（不要放 `C:\Program Files`，避免权限问题）；
2. 双击 `start.bat`，浏览器打开 <http://127.0.0.1:8090>；
3. 首次进入 `/setup`：创建管理员 → 记下机器码（可跳过授权先试用）→ 完成；
4. 需要开机自启：右键 `install-service.bat` → 以管理员身份运行（NSSM，默认服务名 `GimbalXReport`）。

详细说明见包内 `docs/install-single-windows.md`。

## Linux 单机版（不用 Docker）

```bash
tar -xzf GimbalX-Report-1.0.0-linux-x86_64.tar.gz
cd GimbalX-Report-1.0.0-linux-x86_64
./start.sh                     # 前台试运行：http://127.0.0.1:8090
sudo ./tools/install-service.sh # 注册 systemd 开机自启（可选，服务名 gimbalx-report）
```

- 自带 CPython 3.12 与全部依赖，**不需要** Docker、系统 Python、pip 或 Node.js；
- 要求 glibc ≥ 2.17（CentOS 7 / Ubuntu 16.04 及以上），支持 x86_64；ARM 服务器可用
  `scripts/build-single-linux.sh --arch aarch64` 在对应机器上构建；
- 默认监听 `0.0.0.0:8090`：同网段浏览器可直接用 `http://<主机IP>:8090` 打开
  （放行防火墙 `sudo ufw allow 8090/tcp`）；只想本机用就 `install-service.sh --bind 127.0.0.1`，
  对公网建议加 nginx 反代。

详细说明见包内 `docs/install-single-linux.md`。

## macOS

macOS 包必须在 macOS 上构建（`scripts/build-single-macos.sh`，Apple Silicon/Intel 均可），
本期未附构建产物，也未在 macOS 实机验证；本版本可用 Docker Desktop 或源码方式运行。

## 试用与正式版

- **试用版**：功能不限、时间不限；报表渲染结果超过 2 页时，整份文档每页叠加
  `[试用版 TRIAL]` 水印（预览与 PDF/Excel/Word/HTML/SVG/RTF 导出同样生效）；≤ 2 页无水印。
- **试用版单会话**：同一时间只允许一个登录会话，新登录会把旧会话顶下线（页面提示
  “账号已在其他设备登录”），用于防止账号共享。
- **正式版按席位授权**：`.lic` 带 `seats`（并发会话数，默认 1，可签 3/5/不限）。席位用满后**新登录被拒绝**
  并提示“授权并发已满（N/N）”，已在线的人不受影响；需要扩容就加机器码/席位重新签发。
  同一账号在任何版本都只保留 1 个会话（第二台设备登录会踢掉第一台），避免一个 admin 账号全公司共用。
- **授权核心已编译**：发布包里的授权验签、席位数校验、会话校验（`app/core/licensing`、`sessions`、
  `dependencies`、`app/api/v1/auth`、`license` 等模块）是 Cython 编译后的二进制（`.pyd` / `.so`），
  包内**不含**对应 `.py`，也不含 `__pycache__` 字节码（构建脚本会校验，发现即失败），普通改文本无法绕过授权。
  **Docker 单容器镜像同样编译**：镜像里这 6 个模块只有 `.so`，构建时会校验并以此失败。
- **正式版**：不需要重新下载或重装。在「系统设置 → 授权」复制机器码发给作者，收到 `.lic`
  后上传即生效；也可在首次 `/setup` 向导里直接导入。授权按机器绑定，可一份文件包含多台机器码；
  离线校验，不联网、不回传数据。
- **升级提示**：本版新增 `user_sessions` 表记录在线会话。从旧版覆盖安装后，所有已登录的浏览器需要
  **重新登录一次**（旧令牌没有会话记录会被判为失效）；数据与授权不受影响。

## Docker 单容器

```bash
cp .env.single.example .env.single      # 修改 SECRET_KEY / CONFIG_ENCRYPTION_KEY
docker compose -f docker-compose.single.yml --env-file .env.single up -d --build
# 打开 http://<host>:8090 ，同样走 /setup 向导
```

重建容器不想掉授权：把宿主机的 `/etc/machine-id` 内容写进 `QRPT_MACHINE_ID`。

## 校验

```bash
sha256sum -c SHA256SUMS
# Windows PowerShell
Get-FileHash GimbalX-Report-1.0.0-win64.zip -Algorithm SHA256
```

## 更名与兼容（v1.0.0）

| 项目 | 旧 | 新 |
|---|---|---|
| 产品名 | qRptDesigner | GimbalX Report / 万向报表 |
| 安装包与目录 | `qRptDesigner-1.0.0-*` | `GimbalX-Report-1.0.0-*` |
| systemd / launchd | `qrpt-designer` / `com.qrpt.designer` | `gimbalx-report` / `com.gimbalx.report` |
| Windows 服务 | `qRptDesigner` | `GimbalXReport` |
| 授权编号前缀 | `QRPT-…`（历史授权**继续可用**） | `GMBX-…` |

- 内部标识按兼容性保留、**不影响使用**：环境变量 `QRPT_*`、SQLite 文件名 `qrpt.db`、MySQL 库名
  `qrpt` / `qrpt_runtime`、API Key 前缀 `qrpt_`；升级时无需改 `.env`。
- 从旧包升级：解压新包 → 把旧目录的 `data/` 与 `.env` 复制过来 → 若注册过服务，按新服务名重装
  （`tools/install-service.sh` / `install-service.bat`）。

## 已知限制

- 单机版默认监听 `0.0.0.0:8090`（同网段可访问）；对公网请用服务器版 + HTTPS 反代，并先改管理员密码。
- 试用版水印只在“渲染页数 > 2”时出现，属产品策略，不提供关闭开关。
- Linux 包为 glibc 构建；Alpine 等 musl 系统需用 `--libc musl` 另行构建（实验性）。
- macOS 包本期未提供构建产物，也未实机验证。
- 试用/正式版都不含示例客户数据；首次启动数据库为空。

## 许可

源码可见，商业使用需正式授权；条款见 [LICENSE](LICENSE)。第三方组件见
[THIRD-PARTY-NOTICES.md](THIRD-PARTY-NOTICES.md)。

---

## English summary

**GimbalX Report** (formerly qRptDesigner) ships Windows x64 and Linux x86_64 portable builds (both
bundled Python runtimes, no Docker or system Python required), plus a Docker single-container compose
file. Trial is fully functional with a `[TRIAL]` watermark on every page when a report renders more
than 2 pages. Licensed edition = same build + an offline `.lic` bound to your machine code; no
reinstall, no activation server, no telemetry.

- Windows: unzip `GimbalX-Report-1.0.0-win64.zip`, run `start.bat`, open <http://127.0.0.1:8090>,
  finish `/setup`. Optional service install via `install-service.bat` (NSSM, service `GimbalXReport`).
- Linux: `tar -xzf ...tar.gz && cd GimbalX-Report-1.0.0-linux-x86_64 && ./start.sh`; optional
  `sudo ./tools/install-service.sh` for systemd auto-start (service `gimbalx-report`). Requires
  glibc ≥ 2.17. See `docs/install-single-linux.md`.
- macOS: no binary in this release — build with `scripts/build-single-macos.sh` on a Mac,
  or use Docker Desktop / run from source.
- All forms (including Docker-free source installs) are compared in `docs/install-options.md`.

## Rebuild 2026-11-03 (same version, new package)

v1.0.0 was re-packed to fix a Windows-only defect in the shipped launcher:

- `start.bat` used LF line endings, so `cmd.exe` failed with errors like
  `'l' is not recognized` / `'hon.exe' is not recognized` right after the user unzipped;
- the same file had an unescaped `|` inside an `echo` line, which `cmd.exe` treated as a pipe
  (`'局域网' is not recognized`), and it was written as GBK while the script switched the console
  to UTF-8 (`chcp 65001`), garbling the Chinese banner.

Both are fixed: batch files are now written as UTF-8 without BOM with CRLF line endings and no bare pipe.
Package docs were refreshed at the same time (`docs/licensing.md` no longer has the "secondary
development" section, and the README QR image now ships as `docs/assets/wechat-qr.png`).

- New Windows package SHA256: `bba16f37a14f347db97a4d870c91eb3de0fdb60279b916b075c251dc1cb85212`
- Linux tarball is unchanged: `6e5009891105176f67fc47bedebf4dc5080fd0505c8e09455460c1014b0a53c3`

If you already downloaded the old zip, please download again — the old one cannot start on Windows
by double-clicking `start.bat`.
