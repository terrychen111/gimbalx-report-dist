# 下载渠道

GimbalX Report v1.0.0 试用包有三条下载通道，内容完全一致（同一批文件、同一套 SHA256）。

| 通道 | 地址 | 说明 |
|---|---|---|
| **Gitee（国内，首选）** | <https://gitee.com/szliren/gimbalx-report-dist/releases/tag/v1.0.0> | 国内直连，建连 1~2 秒 |
| **AtomGit（国内，备用）** | <https://atomgit.com/terrychen19710516/gimbalx-report-dist/releases/tag/v1.0.0> | 国内直连，与 GitCode 同一套服务 |
| GitHub（海外 / 备用） | <https://github.com/terrychen111/gimbalx-report-dist/releases/tag/v1.0.0> | 国内访问不稳定，可加加速前缀 |

GitHub 加速前缀（第三方服务，可能随时失效）：把完整的 GitHub 附件地址接在后面，例如

```
https://gh-proxy.com/https://github.com/terrychen111/gimbalx-report-dist/releases/download/v1.0.0/GimbalX-Report-1.0.0-win64.zip
```

## 文件清单

| 文件 | 适用 | 大小 |
|---|---|---|
| `GimbalX-Report-1.0.0-win64.zip` | Windows 10/11 x64 | 82.2 MB |
| `GimbalX-Report-1.0.0-linux-x86_64.tar.gz.part01` `part02` | Linux x86_64（glibc ≥ 2.17） | 90.0 MB + 67.6 MB |
| `SHA256SUMS` / `SHA256SUMS.parts` | 整包 / 分卷校验值 | 小文件 |
| `MERGE.md` | 分卷合并说明 | 小文件 |

## 为什么 Linux 包是两卷

Gitee 的发行版附件单文件上限 100 MB，Linux 包 156.8 MB 超限，因此切分为 90 MB + 66.8 MB 两卷。

```bash
# 下载两卷到同一目录后
cat GimbalX-Report-1.0.0-linux-x86_64.tar.gz.part01 \
    GimbalX-Report-1.0.0-linux-x86_64.tar.gz.part02 > GimbalX-Report-1.0.0-linux-x86_64.tar.gz
sha256sum -c SHA256SUMS --ignore-missing
```

```bat
:: Windows
copy /b "GimbalX-Report-1.0.0-linux-x86_64.tar.gz.part01" + "GimbalX-Report-1.0.0-linux-x86_64.tar.gz.part02" "GimbalX-Report-1.0.0-linux-x86_64.tar.gz"
```

合并后的 SHA256 应为：

```
6e5009891105176f67fc47bedebf4dc5080fd0505c8e09455460c1014b0a53c3
```

## 校验

```powershell
Get-FileHash .\GimbalX-Report-1.0.0-win64.zip -Algorithm SHA256
# bba16f37a14f347db97a4d870c91eb3de0fdb60279b916b075c251dc1cb85212
```

## 说明

- 三个通道发布的都是**编译后的安装包与文档**，源代码不公开。
- 若某条通道打不开，换另一条即可，文件内容一致。