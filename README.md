# GimbalX Report 万向报表 — 免安装的 Web 报表设计器

> 解压即可用的**类 Excel 带状（Band）报表设计器**：不用装 Java、不用 Docker、不用 Python、不用数据库中间件。
> 浏览器里拖拽排版，连 MySQL / Oracle / SQL Server / PostgreSQL / ClickHouse，预览分页，
> 导出 **PDF / Excel / Word / RTF / SVG / HTML / CSV / JSON / XML**。
> 支持内网私有化部署与**离线授权**，数据不出内网。

**中文** | 本仓库只发布编译打包后的安装包与文档，源码为商业私有，不在此公开。

---

## 30 秒上手（Windows）

1. 下载 `GimbalX-Report-1.0.0-win64.zip`（79.4 MB），解压到任意可写目录（**不要**放 `C:\Program Files`）；
2. 双击 `start.bat`，浏览器打开 <http://127.0.0.1:8090>（同网段其他电脑用 `http://<本机IP>:8090`）；
3. 首次进入初始化向导：创建管理员 →（可选）导入授权 → 完成；
4. 进「报表 → 新建」，左侧字典面板加数据源写 SQL，从「组件」页签拖入列头 / 数据区 / 表格带，切「预览」看效果。

需要开机自启：安装 [NSSM](https://nssm.cc/download) 后双击 `install-service.bat`（服务名 `GimbalXReport`）。

Linux（x86_64，glibc ≥ 2.17）：

```bash
cat GimbalX-Report-1.0.0-linux-x86_64.tar.gz.part01 \
    GimbalX-Report-1.0.0-linux-x86_64.tar.gz.part02 > GimbalX-Report-1.0.0-linux-x86_64.tar.gz
tar -xzf GimbalX-Report-1.0.0-linux-x86_64.tar.gz
cd GimbalX-Report-1.0.0-linux-x86_64 && ./start.sh      # http://127.0.0.1:8090
sudo ./tools/install-service.sh                          # 可选：systemd 开机自启
```

## 下载

| 文件 | 适用 | 大小 | 链接 |
|---|---|---|---|
| `GimbalX-Report-1.0.0-win64.zip` | Windows 10/11 x64 | 79.4 MB | [下载](https://github.com/terrychen111/gimbalx-report-dist/releases/download/v1.0.0/GimbalX-Report-1.0.0-win64.zip) |
| `GimbalX-Report-1.0.0-linux-x86_64.tar.gz.part01` | Linux x86_64 | 90.0 MB | [下载](https://github.com/terrychen111/gimbalx-report-dist/releases/download/v1.0.0/GimbalX-Report-1.0.0-linux-x86_64.tar.gz.part01) |
| `GimbalX-Report-1.0.0-linux-x86_64.tar.gz.part02` | Linux x86_64 | 66.8 MB | [下载](https://github.com/terrychen111/gimbalx-report-dist/releases/download/v1.0.0/GimbalX-Report-1.0.0-linux-x86_64.tar.gz.part02) |
| `SHA256SUMS` / `SHA256SUMS.parts` | 校验值 | — | [下载](https://github.com/terrychen111/gimbalx-report-dist/releases/download/v1.0.0/SHA256SUMS) |
| `MERGE.md` | 分卷合并说明 | — | [下载](https://github.com/terrychen111/gimbalx-report-dist/releases/download/v1.0.0/MERGE.md) |

全部附件见 [发行版 v1.0.0](https://github.com/terrychen111/gimbalx-report-dist/releases/tag/v1.0.0) · 其他下载通道见 [DOWNLOAD-MIRRORS.md](https://github.com/terrychen111/gimbalx-report-dist/blob/main/DOWNLOAD-MIRRORS.md)

Linux 包为什么是两卷：单文件附件上限 100 MB，157 MB 的包切成 90 + 66.8 MB，`cat` 合并后 SHA256 为
`6e5009891105176f67fc47bedebf4dc5080fd0505c8e09455460c1014b0a53c3`。

## 它解决什么问题

| 场景 | 常见做法 | 用本产品的做法 |
|---|---|---|
| 客户现场只有内网、IT 不给装环境 | 带一堆安装包和依赖过去，装一天 | U 盘拷一个 zip，解压双击，5 分钟出报表 |
| 要类 Excel 的表头表尾、分组小计、交叉表 | 自研导出模块 / 用开源库拼 | 带区模型 + 交叉表 + 子报表，拖拽完成 |
| 数据不能出内网 | 云 BI 用不了 | 全离线，授权也不联网、不激活、不上报 |
| 交付给客户后要能自己改报表 | 每次改动都要回厂改代码 | 客户自己在浏览器里改模板、导出 |

## 功能

- **设计器**：带区布局、标尺吸附、多选对齐/分布、层级前后、组合解组、撤销重做、格式刷、版本管理、水印、参数管理
- **元素**：文本、图片、线条、矩形、形状、表格带、交叉表、图表、条码 / 二维码、数据标签、子报表
- **数据源**：MySQL、PostgreSQL、SQL Server、Oracle、ClickHouse、ODBC，以及 CSV / Excel / JSON / XML；多数据源之间用 DuckDB 做内存 JOIN
- **表达式**：计算字段、`IIF()` / `SUM()` / `COUNT()` 等函数、条件样式（按值变色、数据条）
- **输出**：分页预览；PDF（矢量）、Excel、Word、RTF、SVG、HTML；数据导出 CSV / JSON / XML
- **运行端**：只读浏览页、Embed 短时 Token 嵌入自有系统、客户与授权管理、调用日志、审计日志、定时任务
- **部署形态**：Windows / Linux 免安装包、单容器 Docker、systemd 常驻、设计端 + 运行端双机（MySQL 8）

## 试用版 vs 正式版

同一份安装包、同一套代码，区别只在授权状态：

| | 试用版（默认） | 正式版 |
|---|---|---|
| 功能 / 期限 | 完整 / 不限 | 完整 / 按授权 |
| 并发会话 | 1 个（新登录顶掉旧登录） | `.lic` 的 `seats`（1 / 3 / 5 / 不限） |
| 渲染 > 2 页的报表 | 每页叠加 `[试用版 TRIAL]` 水印 | 无水印 |
| 渲染 ≤ 2 页 | 无水印 | 无水印 |
| CSV / JSON / XML 数据导出 | 不受影响 | 不受影响 |

正式版**不需要重新下载或重装**：在「系统设置 → 授权」复制机器码，收到 `.lic` 后导入即生效。
授权由离线私钥签名、程序内置公钥验签，不联网、不采集数据。

## 适合 / 不适合

**适合**：中小企业 IT、制造业与贸易的信息部门、做 ERP / MES / 进销存的集成商与外包团队、需要在客户内网交付报表的人。

**不太适合**：需要大数据量实时 BI 大屏、需要公有云多租户 SaaS、需要复杂数据治理与血缘分析的团队。

## 文档

| 文档 | 内容 |
|---|---|
| [安装总览](docs/install-options.md) | 全部安装形态（免安装包 / Docker / 源码 / 服务器版）怎么选 |
| [Windows 安装](docs/install-single-windows.md) | 解压、启动、开机自启、端口与防火墙 |
| [Linux 安装](docs/install-single-linux.md) | tar.gz 部署、systemd、监听地址与反代 |
| [Docker 单容器](docs/install-single-docker.md) | compose 起服务、环境变量、数据持久化 |
| [授权说明](docs/licensing.md) | 试用/正式差异、机器码、`.lic` 内容、换机与扩容 |
| [第三方许可](THIRD-PARTY-LICENSES.txt) | 全部第三方组件与许可证 |

## 教程（每篇都能跑通，带建表 SQL 和验收清单）

| # | 教程 | 你会得到 | 前提 |
|---|---|---|---|
| 1 | [10 分钟做出第一张报表](docs/tutorials/01-quickstart-csv.md) | 不装数据库，用 CSV 走通「建报表 → 拖带区 → 导 PDF」 | 一台 Windows |
| 2 | [连 MySQL 做分组小计报表](docs/tutorials/02-mysql-grouped-report.md) | 数据源、SQL 参数、组头组脚、小计总计、条件样式 | 教程 1 + 一个 MySQL |
| 3 | [交叉表：区域 × 月份](docs/tutorials/03-crosstab.md) | 行/列/值配置、行列合计、空值处理 | 教程 2 |
| 4 | [主从报表：订单 + 明细](docs/tutorials/04-subreport-master-detail.md) | 子报表、参数传递、禁止跨页断开 | 教程 2 |
| 5 | [导出与打印](docs/tutorials/05-export-and-print.md) | 页面设置、页码、PDF 中文字体、Excel 保真、批量与定时导出 | 任一可预览的报表 |

## 常见问题

**需要装 Java / Python / Docker 吗？** 都不需要，免安装包内含运行时。

**数据会外传吗？** 不会。只连你配置的数据源；授权本地离线验签，不联网、不激活、不上报。

**能嵌到我们自己的系统里吗？** 可以，运行端支持只读浏览页与 Embed 短时 Token，也提供 REST API 触发渲染导出。

**支持哪些数据库？** MySQL、PostgreSQL、SQL Server、Oracle、ClickHouse、ODBC、CSV / Excel / JSON / XML。

**macOS 能用吗？** 本期未提供 macOS 二进制包（需在 Mac 上构建），可先用 Docker Desktop 跑单容器版。

**源码开源吗？** 不开源。需要二次开发或白标可谈 OEM 合作。

## 联系

<p align="center">
  <img src="docs/assets/wechat-qr.png" alt="作者微信二维码" width="160"><br>
  <em>加作者微信：咨询授权 / 报价 / 私有化部署</em>
</p>

其他镜像通道：[Gitee](https://gitee.com/szliren/gimbalx-report-dist) · [AtomGit](https://atomgit.com/terrychen19710516/gimbalx-report-dist) · [下载渠道说明](https://github.com/terrychen111/gimbalx-report-dist/blob/main/DOWNLOAD-MIRRORS.md)

---

关键词：报表工具、报表设计器、类 Excel 报表、免安装报表软件、私有化报表、内网报表系统、
离线部署报表、单机报表软件、PDF 报表导出、交叉表设计器、Web 报表设计器