# macOS 单机版安装（免安装包，无需 Docker）

适用于 **Apple Silicon（arm64）/ Intel（x86_64）** 的 macOS 12 及以上。
包内自带 CPython 运行时与依赖，**不需要**系统 Python、pip、Node.js 或 Docker。

> 构建说明：macOS 包必须在 macOS 上打（tar 需保留符号链接与可执行位）。
> 仓库提供 `scripts/build-single-macos.sh`，本次交付未在 macOS 实机验证构建与安装，
> 详见文末「验证状态」。如果你手头没有 macOS 构建机，可先用 Docker Desktop 或源码方式。

## 1. 解压

```bash
tar -xzf 万向报表（GimbalX Report）-1.0.0-macos-arm64.tar.gz     # Intel 用 -macos-x86_64
cd GimbalX-Report-1.0.0-macos-arm64
```

建议放在 `~/qrpt` 或 `/opt/gimbalx`（可写目录，勿放 `/Applications` 外的只读位置）。

## 2. 首次运行（处理 Gatekeeper）

从网络下载的包会带 quarantine 属性，首次执行可能被拦：

```bash
xattr -dr com.apple.quarantine .
./start.sh
```

浏览器打开 <http://127.0.0.1:8090>，首次进入 `/setup`：创建管理员 → 记机器码（可跳过先试用）→ 完成登录。

> 默认监听 `0.0.0.0:8090`，同一局域网的其他电脑可用 `http://<本机IP>:8090` 打开
> （macOS 首次会弹“是否允许传入连接”，选允许；或 `HOST=127.0.0.1` 只留本机）。
> 试用版同一时间只允许一个登录会话，新登录会把旧会话顶下线；正式版按 `.lic` 的 `seats` 放行，满了拒绝新登录。

`Ctrl+C` 停止。

## 3. 开机自启（launchd）

```bash
./tools/install-service.sh              # 当前用户登录后自启（LaunchAgent）
sudo ./tools/install-service.sh --system # 开机自启（LaunchDaemon，需 root）
./tools/install-service.sh --port 8091   # 换端口
```

```bash
launchctl print gui/$(id -u)/com.gimbalx.report | head -20
tail -f logs/service.err.log
```

卸载：`./tools/uninstall-service.sh`（`--system` 对应守护进程）。

## 4. 配置（.env）

与 Linux/Windows 版一致：`HOST`、`PORT`、`DB_SQLITE_PATH`、`DATA_ROOT`、
`SECRET_KEY`（JWT 签名）、`CONFIG_ENCRYPTION_KEY`（数据源密码加密）、`CORS_ORIGINS`、`SCHEDULER_ENABLED`。
改完 `launchctl kickstart -k gui/$(id -u)/com.gimbalx.report` 或重启服务生效。

数据在 `data/`（`qrpt.db`、`reports/`、`datasource_files/`），备份直接复制该目录与 `.env`。

## 5. 升级

```bash
launchctl bootout gui/$(id -u)/com.gimbalx.report 2>/dev/null || true
tar -xzf GimbalX-Report-<新版本>-macos-arm64.tar.gz
cp -a 旧目录/data 新目录/ && cp -a 旧目录/.env 新目录/
cd 新目录 && ./tools/install-service.sh
```

`start.sh` 每次启动都会自动执行数据库迁移。

## 6. 授权

试用版渲染 > 2 页时每页带 `[试用版 TRIAL]` 水印（预览与导出一致）；≤ 2 页无水印。
「系统设置 → 授权」复制机器码发给作者，收到 `.lic` 后上传即转正式版，无需重装。
macOS 机器码取 `IOPlatformUUID`（`ioreg -rd1 -c IOPlatformExpertDevice`）。
详见 [licensing.md](licensing.md)。

## 7. 常见问题

| 现象 | 处理 |
|---|---|
| “无法打开，因为无法验证开发者” | `xattr -dr com.apple.quarantine .` 或右键 → 打开 |
| `bad CPU type in executable` | 包与机器架构不符（arm64 包不能在 Intel 上跑）；用对应架构重下 |
| 端口占用 | `lsof -nP -iTCP:8090 -sTCP:LISTEN`，或改 `.env` 的 `PORT` |
| 服务反复重启 | 看 `logs/service.err.log`；确认目录可写、`.env` 存在 |

## 8. 验证状态

- 构建脚本与 launchd 脚本已随仓库提供，并通过 `bash -n` 语法检查与 plist 结构校验；
- **未在 macOS 实机执行**：构建、Gatekeeper 放行、launchd 自启均属未验证项；
- 需要交付 macOS 包时，请在一台 macOS 上执行：

```bash
cd frontend && npm install && npm run build
cd .. && bash scripts/build-single-macos.sh --version 1.0.0
```