---
title: 证书管理系统文档主页
---

# 证书管理系统文档

欢迎来到本项目的文档主页。本页面汇总了与证书管理、远程桌面配置、VPN 加密以及仓库工作流相关的技术文档，便于快速导航与查阅。


## 文档目录

- [Windows 10 远程桌面证书配置指南](windows-rdp-certificate-setup.md)
  - 配置 RDP 使用受信任证书：PFX 导入（含完整链）、注册表 `SSLCertificateSHA1Hash`、私钥权限（Network Service）、常见错误 0x8009030D 处理。
- [Diffie-Hellman 密钥交换在 OpenVPN 中的应用](dh-key-exchange.md)
  - 介绍 DH 的数学原理、在 TLS/OpenVPN 中的角色、生成与安全实践建议。
- [Git 分支合并指南：从单分支到 Git Flow](git-merge-guide.md)
  - 从线性开发迁移到 Git Flow 的分支策略、合并流程与常见操作示例。
- [Certbot DNS-01 与 CSR 实践指南](certbot-dns01-csr-guide.md)
  - 基于 DNS-01 挑战的证书申请与 CSR 流程说明，适合需要自动化与批量申请的场景。