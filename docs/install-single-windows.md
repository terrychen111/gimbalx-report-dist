# Windows 单机版安装与运维

适用于 `万向报表（GimbalX Report）-<版本>-win64.zip`，目标机器**无需安装 Python / Node / MySQL**。

## 1. 环境要求

| 项目 | 要求 |
|---|---|
| 操作系统 | Windows 10 / 11 / Server 2016+，x64 |
| 内存 | 建议 ≥ 4 GB（导出大报表时 8 GB 更稳） |
| 磁盘 | 安装包约 150–250 MB，运行数据另计 |
| 端口 | 默认 8090（只用本机时无需放行防火墙） |
| 权限 | 双击 `start.bat` 普通用户即可；装服务需管理员 |

安装目录不要放在 `C:\Program Files`（服务账户写入日志/数据会受限），推荐 `D:\万向报表`。

## 2. 解压与首次启动

1. 解压 zip 到 `D:\万向报表`，目录内应有：
   ```
   app\  runtime\  frontend\dist\  alembic\  alembic.ini  data\
   tools\  docs\  .env  start.bat  install-service.bat  README.txt
   ```
2. 双击 `start.bat`。脚本会先执行数据库迁移，再启动服务。
3. 浏览器打开 <http://127.0.0.1:8090>，自动进入 `/setup` 初始化向导：
   - 第 1 步：创建管理员（用户名 / 密码 ≥ 8 位 / 邮箱选填）。
   - 第 2 步：显示本机机器码；有 `.lic` 就选择导入，没有就「跳过，先试用」。
   - 第 3 步：完成，用刚创建的账号登录。
4. 控制台窗口需保持打开；关闭窗口即停止服务。要长期运行请装成 Windows 服务。

## 3. 安装为 Windows 服务（开机自启）

1. 下载 [NSSM](https://nssm.cc/download)，解压得到 `nssm.exe`（例如 `C:\tools\nssm.exe`）。
2. 右键「以管理员身份运行」`install-service.bat`，或在包目录执行：
   ```powershell
   powershell -ExecutionPolicy Bypass -File tools\install-service.ps1 -NssmExe C:\tools\nssm.exe
   ```
3. 常用参数：
   - `-Port 8090`：监听端口
   - `-ServiceName GimbalXReport`：服务名
   - `-OpenFirewall`：同时放行防火墙（局域网访问时用）
   - `-BindHost 127.0.0.1`：只允许本机访问（默认 `0.0.0.0`，同网段可直接用 `http://本机IP:8090`）
4. 服务日志：`logs\service.out.log` / `logs\service.err.log`（10 MB 自动轮转）。
5. 停止 / 卸载：
   ```powershell
   & C:\tools\nssm.exe stop GimbalXReport
   & C:\tools\nssm.exe remove GimbalXReport confirm
   ```

> 局域网访问：单机包默认已监听 `0.0.0.0:8090`，同网段浏览器直接打开 `http://本机IP:8090` 即可；
> 需在 Windows 防火墙放行 TCP 8090（`install-service.ps1 -OpenFirewall` 会自动放行）。
> 只给本机用就把 `.env` 的 `HOST=0.0.0.0` 改成 `127.0.0.1` 并重启服务。
>
> 安全提示：对外暴露前请改掉默认管理员密码，并尽量用防火墙限制可访问网段；
> 试用版同一时间只允许一个登录会话（新登录会把旧会话顶下线）；正式版按 `.lic` 的 `seats` 放行，
> 席位满了新登录会被拒绝。任何版本下同一账号都只保留一个会话。

## 4. 授权（试用 → 正式）

- 未导入授权即为试用版：报表渲染超过 2 页时整份文档每页带 `[试用版 TRIAL]` 水印，功能不受限。
- 升级步骤：
  1. 登录后进入「系统设置 → 授权」，复制机器码（`XXXX-XXXX-XXXX-XXXX`）。
  2. 把机器码和购买信息发给作者（README 中的微信二维码）。
  3. 收到 `.lic` 文件后，在同一页面点「导入授权文件」，立即生效，无需重启。
- 换机器/重装系统后机器码会变，需要用新机器码重新签发；原授权文件仍可留在旧机器。
- 授权是离线的，程序不会连接任何授权服务器。

详细规则见 [licensing.md](licensing.md)。

## 5. 数据、备份与升级

| 内容 | 位置 |
|---|---|
| SQLite 数据库 | `data\qrpt.db` |
| 报表模板/图片 | `data\reports\` |
| 数据源文件（CSV/Excel 等） | `data\datasource_files\` |
| 导出临时文件 | `data\temp\`（可清空） |
| 日志 | `logs\` |

备份：停止服务后直接复制整个 `data\` 目录即可（建议每天一次，保留 7–30 天）。
升级：解压新版本到新目录 → 停止旧服务 → 复制旧目录的 `data\` 和 `.env` 到新目录 →
用 `start.bat` 验证 → 再切换服务目录（`nssm set GimbalXReport AppDirectory <新目录>`）。
数据库结构变化由启动时的 `db-upgrade` 自动处理。

## 6. 改用 MySQL（可选）

单机版默认 SQLite；数据量大或需要多人并发时建议 MySQL 8：

1. 安装 MySQL 8，建库与账号：
   ```sql
   CREATE DATABASE qrpt DEFAULT CHARACTER SET utf8mb4;
   CREATE USER 'qrpt'@'127.0.0.1' IDENTIFIED BY '强密码';
   GRANT ALL PRIVILEGES ON qrpt.* TO 'qrpt'@'127.0.0.1';
   ```
2. 编辑 `.env`：
   ```ini
   DB_TYPE=mysql
   DB_HOST=127.0.0.1
   DB_PORT=3306
   DB_NAME=qrpt
   DB_USER=qrpt
   DB_PASSWORD=强密码
   DB_AUTO_MIGRATE=true
   ```
3. 重启服务，首次启动自动建表。
4. 从 SQLite 迁移：在旧环境导出 SQL 后导入 MySQL（可用 `deploy/migrate/` 下脚本，或联系作者）。

## 7. 离线安装（目标机器无外网）

发布包已内置运行时和全部依赖，**解压即用，无需联网**。只有两点需要注意：

- 如需 Oracle / SQL Server 数据源：把对应驱动 wheel（`oracledb` / `pyodbc` + ODBC Driver 18）复制到
  目标机，解压到 `runtime\Lib\site-packages`，再按驱动要求安装系统组件。
- PDF 导出使用 ReportLab（矢量），不依赖浏览器；如需像素级精确排版，可自行安装 Playwright 与 Chromium
  并设置 `PLAYWRIGHT_ENABLED=true`。

## 8. 常见问题

**Q：双击 start.bat 闪退？**
A：用 `cmd` 进入目录执行 `start.bat` 查看报错；多数是端口被占用或 `.env` 里 `SECRET_KEY` 为空。
改端口：编辑 `.env` 的 `HTTP_PORT`，同时修改 `start.bat` 中的 `--port` 与 `CORS_ORIGINS`。

**Q：浏览器打开是空白页？**
A：确认 `frontend\dist\index.html` 存在；确认 `SERVE_FRONTEND=true` 且 `FRONTEND_DIST_PATH=./frontend/dist`。

**Q：PDF 中文是方块？**
A：单机版调用 Windows 系统字体（微软雅黑/宋体）。精简版系统若缺失，请安装「微软雅黑」字体后重启服务。

**Q：忘记管理员密码？**
A：在包目录执行
```powershell
runtime\python.exe -m app.cli create-admin --username admin --password 新密码 --force
```

**Q：导入授权提示「机器码不匹配」？**
A：`.lic` 是给别的机器签发的。复制本机机器码重新申请；同一份授权可包含多台机器，但必须由作者加入。

**Q：能直接拷贝整个目录到另一台机器吗？**
A：可以迁移数据，但机器码会变，试用状态继续有效；正式授权需为新机器重新签发 `.lic`。