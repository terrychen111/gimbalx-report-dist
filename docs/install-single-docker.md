# Docker 单容器版（单机/小团队）

一个容器同时提供 API 与前端，数据落在 `/data` 卷，默认 SQLite。

## 1. 准备

- Docker 20.10+（含 compose v2）
- 2 GB 内存、1 CPU 起步
- 端口 8090

```bash
git clone <本仓库地址> qrpt && cd qrpt
cp .env.single.example .env.single
```

编辑 `.env.single`，至少修改：

```ini
SECRET_KEY=<随机 48 位以上字符串>
CONFIG_ENCRYPTION_KEY=<随机 48 位以上字符串>
GIMBALX_PORT=8090
CORS_ORIGINS=http://localhost:8090
```

生成随机串：

```bash
openssl rand -base64 36
```

## 2. 启动

```bash
docker compose -f deploy/docker/docker-compose.single.yml --env-file .env.single up -d --build
docker compose -f deploy/docker/docker-compose.single.yml logs -f gimbalx
```

浏览器打开 <http://<主机>:8090>，自动进入 `/setup`：

1. 创建管理员；
2. 显示机器码，可导入 `.lic`（没有就跳过，先试用）；
3. 完成并登录。

## 3. 授权

- 试用版：报表渲染 > 2 页时整份文档每页带 `[试用版 TRIAL]` 水印，功能不限。
- 正式版：设置 → 授权 → 复制机器码 → 找作者签发 `.lic` → 导入，立即生效。
- 机器码默认取容器内 `/etc/machine-id`，**容器重建后会变**。要保证重建/迁移后授权不失效，
  在 `.env.single` 里固定为宿主机 machine-id：

  ```bash
  # Linux 宿主机
  echo "QRPT_MACHINE_ID=$(cat /etc/machine-id)" >> .env.single
  docker compose ... up -d   # 机器码从此稳定，导入的 .lic 一直有效
  ```

  也可以把宿主机文件挂进容器并设置 `QRPT_MACHINE_ID_FILE=/host-machine-id`。
- 完全离线，不访问任何授权服务。

如果重建容器后授权丢失，优先检查是否固定了 `QRPT_MACHINE_ID`，以及 `/data` 卷是否仍在（授权存在数据库里）。

## 4. 数据与备份

| 内容 | 容器内路径 |
|---|---|
| SQLite 数据库 | `/data/qrpt.db` |
| 报表/图片 | `/data/reports/` |
| 数据源文件 | `/data/datasource_files/` |
| 日志 | `/data/logs/` |

备份：

```bash
docker run --rm -v gimbalx-data:/data -v "$PWD":/backup alpine \
  tar czf /backup/gimbalx-data-$(date +%F).tar.gz -C /data .
```

## 5. 升级

```bash
docker compose -f deploy/docker/docker-compose.single.yml pull   # 或重新 build
docker compose -f deploy/docker/docker-compose.single.yml up -d
```

数据卷不变，启动时自动执行数据库迁移。建议升级前先备份。

## 6. 反向代理与 HTTPS

容器只监听 8090，建议前面放 Nginx/Caddy 终止 TLS：

```nginx
server {
    listen 443 ssl;
    server_name reports.example.com;
    client_max_body_size 100m;

    location / {
        proxy_pass http://127.0.0.1:8090;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
    }
}
```

同时把 `.env.single` 的 `CORS_ORIGINS` 改为 `https://reports.example.com` 并重启。

## 7. 常见问题

- **启动即退出**：看日志，多为 `SECRET_KEY` 未设置或为空。
- **容器健康检查失败**：确认 8090 未被占用；`curl http://127.0.0.1:8090/api/v1/health/live`。
- **导出 PDF 中文乱码**：镜像已内置 Noto CJK；如自建镜像请保留 `fonts-noto-cjk`。
- **需要 MySQL**：把 `.env.single` 中 `DB_TYPE=sqlite` 换成 `mysql` 及连接参数，
  并去掉 compose 中默认的 SQLite 路径；服务器版编排见 `docker-compose.prod.yml`。