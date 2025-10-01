---
title: 证书管理系统文档主页
---

# 证书管理系统文档

欢迎来到本项目的文档主页。本页面汇总了与证书管理、远程桌面配置、VPN 加密以及仓库工作流相关的技术文档，便于快速导航与查阅。

如果你是首次访问，建议从“快速开始”查看常用场景的操作指南。

## 快速开始

- [Windows 10 远程桌面证书配置指南](windows-rdp-certificate-setup.md)
  - 在 Windows 上为远程桌面（RDP）配置包含完整证书链的 PFX 证书、注册表自动配置与私钥权限设置，附操作截图与故障排除。
- [Certbot DNS-01 与 CSR 实践指南](certbot-dns01-csr-guide.md)
  - 使用 DNS-01 验证生成 CSR 与自动化签发证书的实用说明与步骤（适用于自动化证书管理场景）。

## 文档目录

- [Windows 10 远程桌面证书配置指南](windows-rdp-certificate-setup.md)
  - 配置 RDP 使用受信任证书：PFX 导入（含完整链）、注册表 `SSLCertificateSHA1Hash`、私钥权限（Network Service）、常见错误 0x8009030D 处理。
- [Diffie-Hellman 密钥交换在 OpenVPN 中的应用](dh-key-exchange.md)
  - 介绍 DH 的数学原理、在 TLS/OpenVPN 中的角色、生成与安全实践建议。
- [Git 分支合并指南：从单分支到 Git Flow](git-merge-guide.md)
  - 从线性开发迁移到 Git Flow 的分支策略、合并流程与常见操作示例。
- [Certbot DNS-01 与 CSR 实践指南](certbot-dns01-csr-guide.md)
  - 基于 DNS-01 挑战的证书申请与 CSR 流程说明，适合需要自动化与批量申请的场景。

## 相关资源

- 仓库首页：[`README.md`](../README.md)
- 问题反馈与建议：在 GitHub Issues 中提交你的问题与建议。

---

版权所有 © 当前仓库所有者。本文档将随代码迭代持续更新。