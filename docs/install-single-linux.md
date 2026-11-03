# Linux 单机版安装（免安装包，无需 Docker）

适用于 **x86_64 / aarch64** 的 Linux 服务器或桌面，包内自带 CPython 运行时与全部依赖，
**不需要**系统 Python、pip、Node.js 或 Docker。

- 系统要求：glibc ≥ 2.17（CentOS 7、Ubuntu 16.04 及以上）、1 GB 内存、500 MB 磁盘；
- 可选：systemd（用于开机自启）；需要 HTTPS 时前置 nginx/caddy；
- 架构：官方包为 x86_64；ARM 服务器可在目标机器上用 `scripts/build-single-linux.sh --arch aarch64` 自行构建；
- Alpine 等 musl 发行版：用 `--libc musl` 构建（实验性，未在 Alpine 实测）。

## 1. 下载与解压

```bash
tar -xzf 万向报表（GimbalX Report）-1.0.0-linux-x86_64.tar.gz
cd GimbalX-Report-1.0.0-linux-x86_64
```

建议放在可写目录，例如 `/opt/gimbalx`（需 root）或 `~/qrpt`：

```bash
sudo mkdir -p /opt/gimbalx && sudo tar -xzf GimbalX-Report-1.0.0-linux-x86_64.tar.gz -C /opt/gimbalx --strip-components=1
cd /opt/gimbalx
```

> 不要放在 `/usr`、`/etc` 等系统只读目录；运行用户需要对 `data/` 和 `logs/` 有写权限。

## 2. 前台试运行

```bash
./start.sh
```

浏览器打开 <http://127.0.0.1:8090>，首次会进入 `/setup`：

1. 创建管理员（用户名至少 3 位、密码至少 8 位）；
2. 记下机器码（可先「跳过，先试用」）；
3. 完成后用刚建的账号登录。

`Ctrl+C` 停止。确认可用后再注册为服务。

### 2.1 安装自检清单（建议逐条做一遍）

```bash
# 1) 校验完整性（值见发布页 SHA256SUMS）
sha256sum GimbalX-Report-1.0.0-linux-x86_64.tar.gz

# 2) 确认不依赖系统 Python / Docker（无需任何额外安装）
./runtime/python/bin/python3 -V          # 内置运行时，应输出 Python 3.12.x
env -i PATH=/usr/bin:/bin ./start.sh     # 最小环境也能启动（另开一个终端验证）

# 3) 服务是否起来、监听是否正确
curl -s -o /dev/null -w '%{http_code}\n' http://127.0.0.1:8090/          # 200
curl -s http://127.0.0.1:8090/api/v1/setup/status                        # initialized/机器码
curl -s http://127.0.0.1:8090/api/v1/license/status                      # trial=true 表示试用版
ss -ltnp | grep 8090                                                     # 默认 0.0.0.0:8090（可局域网访问）
```

要点：

- **浏览器**：Firefox、Chrome/Chromium、Edge 均可（服务端渲染的是标准前端，不挑浏览器）；不要用 `w3m`/`lynx` 这类文本浏览器。
- **访问地址**：单机包默认监听 `0.0.0.0:8090`，同网段浏览器直接打开 `http://<本机IP>:8090` 即可
  （Ubuntu 需放行防火墙：`sudo ufw allow 8090/tcp`；云服务器还要在安全组放行）。
  只想本机用就 `HOST=127.0.0.1`（或 `install-service.sh --bind 127.0.0.1`）后重启。
  跨网段/公网建议走 [§7 反向代理](#7-反向代理--https可选)，不要直接把 8090 裸奔到公网。
- **首次初始化**：`/setup` 三步（管理员 → 授权 → 完成），完成后点「去登录」，用刚建的账号登录；若页面没跳转，直接访问 <http://127.0.0.1:8090/login>（旧版打包的已知缺陷，最新包已修复）。
- **并发会话**：试用版同一时间只允许一个登录会话，新登录会把旧会话顶下线（页面提示“账号已在其他设备登录”）；正式版按 `.lic` 的 `seats`（并发席位，默认 1）放行，满了新登录返回 403「授权并发已满（N/N）」。同一账号始终只保留一个会话，多人使用请各自建账号。
- **systemd 安装后**：`systemctl is-active gimbalx-report` 应为 `active`，且 `curl -I http://127.0.0.1:8090/` 返回 200；`tools/install-service.sh --port/--bind` 会同步改写 `.env` 的 `PORT`/`HOST`，两者必须一致。
- **试用版预期**：报表渲染 **超过 2 页**时每一页都有 `[试用版 TRIAL]` 水印（≤2 页没有），导入 `.lic` 后立即消失。
- **数据位置**：`data/qrpt.db` + `storage/`，升级/迁移只搬这两处即可。

## 3. 注册为 systemd 服务（开机自启，推荐）

```bash
sudo ./tools/install-service.sh                 # 默认服务名 gimbalx-report，监听 0.0.0.0:8090
sudo ./tools/install-service.sh --bind 127.0.0.1  # 只允许本机访问
sudo ./tools/install-service.sh --user qrpt     # 以专用用户运行（推荐，需先 useradd）
sudo ./tools/install-service.sh --port 8091     # 换端口
```

脚本会写入 `/etc/systemd/system/gimbalx-report.service` 并 `enable --now`。
常用命令：

```bash
sudo systemctl status gimbalx-report
sudo systemctl restart gimbalx-report
sudo journalctl -u gimbalx-report -f
tail -f logs/service.err.log
```

> 服务默认监听 `0.0.0.0:8090`，同网段可直接访问。要对公网提供访问，见第 7 节反代；
> 只想本机用则设 `HOST=127.0.0.1`（或 `install-service.sh --bind 127.0.0.1`）。

## 4. 配置（.env）

首次启动前可编辑 `.env`：

| 变量 | 默认 | 说明 |
|---|---|---|
| `HOST` | `0.0.0.0` | 监听地址；`0.0.0.0` 对局域网开放，`127.0.0.1` 仅本机 |
| `PORT` | `8090` | 监听端口（改后 systemd 服务也要用 `--port` 重装） |
| `DB_SQLITE_PATH` | `./data/qrpt.db` | SQLite 数据库位置 |
| `DATA_ROOT` | `./data` | 数据根目录（报表、上传文件） |
| `SECRET_KEY` | 随机生成 | JWT 签名密钥，**不要外传**；更换后所有登录失效 |
| `CONFIG_ENCRYPTION_KEY` | 随机生成 | 数据源密码加密密钥，**更换后已存密码无法解密** |
| `CORS_ORIGINS` | 本机地址 | 前后端同机可保持默认 |
| `SCHEDULER_ENABLED` | `false` | 定时任务；单机可开 |
| `TZ` | `Asia/Shanghai` | 时区 |

修改 `HOST/PORT` 后：

```bash
sudo systemctl restart gimbalx-report
```

## 5. 数据与备份

所有状态都在安装目录下：

```
data/qrpt.db               SQLite 数据库（用户、报表定义、授权）
data/reports/              报表文件与导出缓存
data/datasource_files/     上传的 CSV/Excel 等数据源文件
.env                       配置与密钥
```

备份（服务运行中也可执行，SQLite 用 `.backup` 更稳妥）：

```bash
sqlite3 data/qrpt.db ".backup '/backup/qrpt-$(date +%F).db'" 2>/dev/null \
  || cp data/qrpt.db "/backup/qrpt-$(date +%F).db"
tar -czf "/backup/gimbalx-data-$(date +%F).tar.gz" data .env
```

恢复：停服务 → 覆盖 `data/` 与 `.env` → 启服务。

## 6. 升级（保留数据）

```bash
sudo systemctl stop gimbalx-report
cd /opt
sudo tar -xzf GimbalX-Report-<新版本>-linux-x86_64.tar.gz          # 解压到新目录
sudo cp -a gimbalx/old/data GimbalX-Report-<新版本>-linux-x86_64/     # 只带 data 与 .env
sudo cp -a gimbalx/old/.env GimbalX-Report-<新版本>-linux-x86_64/
sudo ./GimbalX-Report-<新版本>-linux-x86_64/tools/install-service.sh --dir /opt/GimbalX-Report-<新版本>-linux-x86_64
```

或者原地升级：停服务 → 解压覆盖程序文件（**不要覆盖 `data/` 与 `.env`**）→ 启服务。
`start.sh` 每次启动都会自动执行数据库迁移（`app.cli db-upgrade`），无需手工跑 Alembic。

## 7. 反向代理 / HTTPS（可选）

nginx 示例（`/etc/nginx/conf.d/qrpt.conf`）：

```nginx
server {
    listen 80;
    server_name report.example.com;
    client_max_body_size 200m;

    location / {
        proxy_pass http://127.0.0.1:8090;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
        proxy_read_timeout 300s;
    }
}
```

HTTPS 用 `certbot --nginx` 或公司证书；配置好后把 `.env` 的
`CORS_ORIGINS` 改成对外地址（如 `https://report.example.com`）并重启服务。

防火墙（仅放行反代端口，不要直接暴露 8090）：

```bash
sudo firewall-cmd --permanent --add-service=http --add-service=https && sudo firewall-cmd --reload
# 或 ufw allow 80,443/tcp
```

## 8. 授权（试用 → 正式）

- 试用版：渲染结果超过 2 页时整份文档每页带 `[试用版 TRIAL]` 水印（预览与导出都生效）；≤ 2 页无水印；功能和时间不限制。
- 正式版：登录后「系统设置 → 授权」复制机器码 → 发给作者 → 收到 `.lic` 后上传，立即生效、无需重启、无需重装。
- 机器码来源：`/etc/machine-id`；容器或克隆场景可用 `QRPT_MACHINE_ID` 固定。
- 详细说明见 [licensing.md](licensing.md)。

## 9. 故障排查

| 现象 | 处理 |
|---|---|
| `./start.sh: Permission denied` | `chmod +x start.sh`；确认解压时保留了权限 |
| 启动报 `Address already in use` | `ss -lntp \| grep 8090` 找占用进程，或改 `.env` 的 `PORT` |
| `GLIBC_2.xx not found` | 系统过旧（< CentOS 7）；改用 Docker 版或升级系统 |
| 页面能开但登录 401/白屏 | 看 `logs/service.err.log` 与 `journalctl -u gimbalx-report`；确认 `.env` 的 `CORS_ORIGINS` |
| 上传报 413 | 反代调大 `client_max_body_size`（nginx） |
| 服务起不来 | `systemctl status gimbalx-report -l`；确认 `WorkingDirectory` 与 `--dir` 一致 |

日志：前台运行直接输出；systemd 写 `logs/service.out.log` / `logs/service.err.log`。

## 10. 卸载

```bash
sudo ./tools/uninstall-service.sh      # 停止并删除 systemd 单元，保留 data/ 与 .env
rm -rf /opt/gimbalx                        # 确认不再需要后删除程序与数据
```

## 11. 自行构建 Linux 包

仓库脚本基于 python-build-standalone（自带 CPython，不依赖目标机 Python）：

```bash
cd frontend && npm ci && npm run build && cd ..
bash scripts/build-single-linux.sh --version 1.0.0          # 产物在 dist-single-linux/
# 其他架构/运行库：
bash scripts/build-single-linux.sh --arch aarch64
bash scripts/build-single-linux.sh --libc musl              # Alpine（实验性）
# 已有解压好的运行时（例如二次打包）：
bash scripts/build-single-linux.sh --reuse-runtime
```

构建机需要 `curl`、`tar` 和联网下载运行时/依赖；目标机不需要。