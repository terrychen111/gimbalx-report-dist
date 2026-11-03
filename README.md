# GimbalX Report（万向报表）— 试用下载

企业级报表设计与分析平台：类 Excel 的带状报表设计器、多格式导出（PDF / Excel / Word / HTML / CSV / SVG / RTF）、
数据源管理（MySQL / Oracle / SQL Server / PostgreSQL / ClickHouse）、看板与定时任务。
**单机免安装包内置运行时，不需要 Docker、不需要系统 Python。**

> 本仓库只发布**编译打包后的安装包与文档**，源代码为商业私有，不在此公开。

**国内下载**（GitHub 慢或打不开时用这些）：
[Gitee](https://gitee.com/szliren/gimbalx-report-dist/releases/tag/v1.0.0) ·
[AtomGit](https://atomgit.com/terrychen19710516/gimbalx-report-dist/releases/tag/v1.0.0) ·
详细说明见 [DOWNLOAD-MIRRORS.md](DOWNLOAD-MIRRORS.md)

## 下载

| 文件 | 适用 | 大小 |
|---|---|---|
| [GimbalX-Report-1.0.0-win64.zip](https://github.com/terrychen111/gimbalx-report-dist/releases/download/v1.0.0/GimbalX-Report-1.0.0-win64.zip) | Windows 10/11 x64 | 79.4 MB |
| [GimbalX-Report-1.0.0-linux-x86_64.tar.gz](https://github.com/terrychen111/gimbalx-report-dist/releases/download/v1.0.0/GimbalX-Report-1.0.0-linux-x86_64.tar.gz) | Linux x86_64（glibc ≥ 2.17，CentOS 7 / Ubuntu 16.04 起） | 156.8 MB |
| [SHA256SUMS](https://github.com/terrychen111/gimbalx-report-dist/releases/download/v1.0.0/SHA256SUMS) | 校验值 | — |

全部附件在 [Releases](https://github.com/terrychen111/gimbalx-report-dist/releases) 页面。

### 国内下载慢？

把整段地址接到加速前缀后面即可（第三方加速站，随时可能失效）：

```
https://gh-proxy.com/https://github.com/terrychen111/gimbalx-report-dist/releases/download/v1.0.0/GimbalX-Report-1.0.0-win64.zip
https://gh-proxy.com/https://github.com/terrychen111/gimbalx-report-dist/releases/download/v1.0.0/GimbalX-Report-1.0.0-linux-x86_64.tar.gz
```

备用前缀：`https://ghproxy.net/`（用法相同，略慢）。

## 快速开始

### Windows（不用 Docker）

1. 解压 `GimbalX-Report-1.0.0-win64.zip` 到**可写目录**（不要放 `C:\Program Files`，避免权限问题）
2. 双击 `start.bat`，浏览器打开 <http://127.0.0.1:8090>
3. 首次进入 `/setup`：创建管理员 → 记下机器码（可跳过授权先试用）→ 完成
4. 需要开机自启用管理员身份运行 `install-service.bat`（注册 Windows 服务 `GimbalXReport`）

详见 [docs/install-single-windows.md](docs/install-single-windows.md)。

### Linux（不用 Docker）

```bash
tar -xzf GimbalX-Report-1.0.0-linux-x86_64.tar.gz
cd GimbalX-Report-1.0.0-linux-x86_64
./start.sh                        # 前台试运行：http://127.0.0.1:8090
sudo ./tools/install-service.sh   # 可选：注册 systemd 开机自启（服务名 gimbalx-report）
```

默认监听 `0.0.0.0:8090`，同网段浏览器可直接用 `http://<主机IP>:8090` 打开（放行防火墙 `sudo ufw allow 8090/tcp`）。
详见 [docs/install-single-linux.md](docs/install-single-linux.md)。

### Docker 单容器

见 [docs/install-single-docker.md](docs/install-single-docker.md) 与发布页的 `docker-compose.single.yml`。

## 校验

```powershell
# Windows
Get-FileHash .\GimbalX-Report-1.0.0-win64.zip -Algorithm SHA256
```

```bash
# Linux
sha256sum -c SHA256SUMS
```

## 试用与授权

- **试用版**：功能与时间都不限制；报表渲染结果**超过 2 页**时每页叠加 `[试用版 TRIAL]` 水印
  （预览与 PDF / Excel / Word / HTML / SVG / RTF 导出同样生效），≤ 2 页无水印。
- **试用版单会话**：同一时间只允许一个登录会话，新登录会把旧会话顶下线（防账号共享）。
- **正式版按席位授权**：`.lic` 文件含 `seats`（并发会话数，默认 1）。席位占满后新登录被拒绝并提示
  「授权并发已满（N/N）」，已在线的人不受影响。按机器码绑定，离线校验，不联网、不回传数据。
- 授权核心（验签、席位、会话校验）在发布包中是 Cython 编译后的二进制，包内不含对应 `.py`，
  改文本无法绕过授权。

授权细节见 [docs/licensing.md](docs/licensing.md)。

## 系统要求

| 平台 | 要求 |
|---|---|
| Windows | Windows 10 / 11 x64，可写目录 ≥ 2 GB |
| Linux | x86_64，glibc ≥ 2.17（CentOS 7 / Ubuntu 16.04 及以上），≥ 2 GB 空间 |
| 浏览器 | Chrome / Edge / Firefox 最新两个大版本 |

## 反馈

使用中遇到问题请附上：操作系统与版本、启动方式（解压包 / 服务 / Docker）、
`logs/` 下的错误日志片段、复现步骤。请勿在公开 issue 里贴出机器码或许可证文件。
