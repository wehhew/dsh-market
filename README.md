# #1191 本地 checkout 换用发布归档的实机对照

官方 Windows DSH Desktop 0.2.2-rc.3（Electron 44.7.0、内嵌 Node 24.21.0、pnpm 11.28.5），分别用独立 profile。两边用同一份 `dsh-chat-in-harness@1.1.1` GitHub 发布归档，SHA-256 为 `857e3ba8da5d6abdbace956f47e7d8c2333bdb38edc737928cde789cb6b097a0`。本地依赖用官方 CLI 安装成 `link:`。测试进程把同一归档的下载交给可控本地代理，所以内容与原 URL 一致。

| 市场包 | UI 请求 | 线上版本 | 原 checkout |
| --- | --- | --- | --- |
| npm 正式版 1.73.19 | HTTP 200 | 实体目录 1.1.1，归档文件一致 | 13 个文件全部消失 |
| 正式版修补包 56926731… | HTTP 200 | 实体目录 1.1.1，归档文件一致 | 13 个文件全部保留，各自的 SHA-256 一致 |

两边的 UI 最终都报告需要重启。原目录差异见 [逐文件哈希和实际请求](desktop-restore-comparison.json)。另有 [直接连接 GitHub 时的正式版结果](desktop-public-network-baseline.json)。

使用同一个 Desktop 安装包提供的 Node 直接调用最新先行版上的显式卸载清理函数。原函数遇到实际 NTFS junction 会留下链接。换用 `unlinkSync` 后只摘掉清单已移除的链接，指向的文件哈希保持一致。清单仍有声明的链接和普通目录均保留。 [官方 Desktop 内嵌 Node 对照](nightly-uninstall-junction-comparison.json)

## 新发布 1.73.20 后的复测

原版市场和补丁分别基于 `88de957`，补丁提交为 `87d1f97`。正式版依然完成了归档安装并清空原 checkout。修后完成安装，13 个文件的 SHA-256 与操作前一致。两边都是 HTTP 200，包内容与公开归档一致。 [最新正式版逐文件对照](desktop-v1.73.20-comparison.json)

另在隔离 profile 布置一次中断后磁盘状态：原 `link:` 及锁文件还在，junction 缺失，有保存的原链接记录和已退出进程留下的宿主写锁。官方 Desktop 实际启动后接回原链接，记录清到 0，旧锁清除，原 13 个文件哈希一致。启动验收有意布置磁盘状态，路由层另有官方桥 deadline 回归。 [实际启动恢复](desktop-v1.73.20-interrupted-recovery.json)
