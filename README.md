<div align="center">

# 🛡️☁️ Cloudflare Ops Survival Guide

**Cloudflare 运维踩坑宝典 — 深夜排错凝结而成的实战血泪史**

[![GitHub stars](https://img.shields.io/github/stars/Mer3y1338/cloudflare-ops-survival-guide?style=flat-square)](https://github.com/Mer3y1338/cloudflare-ops-survival-guide/stargazers)
[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg?style=flat-square)](https://opensource.org/licenses/MIT)
[![Docs](https://img.shields.io/badge/docs-13%20guides-blue?style=flat-square)](#目录-contents)
[![PRs Welcome](https://img.shields.io/badge/PRs-welcome-brightgreen.svg?style=flat-square)](https://github.com/Mer3y1338/cloudflare-ops-survival-guide/pulls)

*这不是一份 API 翻译文档，而是一份由无数次 `525`、`522`、`9106` 报错和深夜排错凝结而成的实战 SOP。*

**[快速开始](#目录-contents)** · **[贡献指南](#贡献-contributing)** · **[Issues](https://github.com/Mer3y1338/cloudflare-ops-survival-guide/issues)**

</div>

---

## 为什么会有这个项目？

在使用 Cloudflare（以及腾讯 EdgeOne 等类似边缘计算/CDN平台）时，官方文档通常只告诉你“快乐路径（Happy Path）”。但实战中，你往往会遇到：
- 为什么开启了代理，源站明明活着却一直报 `525 SSL handshake failed`？
- 为什么 `wrangler deploy` 一直报 `Authentication failed [code: 9106]`，明明 Token 是对的？
- 为什么部署的 React SPA 页面一片空白，只剩下标题？
- 为什么国内用户访问总是遇到玄学的 403 白页？

本项目旨在收录那些**官方文档不讲、但实战中极其致命的坑点与排查 SOP**。

## 目录 (Contents)

### 0. 基础认证与准备 (Auth & Setup)
- [如何正确获取 Cloudflare API Token 与 Account ID](docs/how-to-get-api-token.md)

### 1. SSL/TLS 与回源故障排查 (Origin SSL & Routing)
- [Error 525 (SSL handshake failed) 终极排查指南](docs/error-525-survival-guide.md)
- [EdgeOne / 非 CF CDN 的 521/525 回源变体](docs/edgeone-origin-521-525.md)
- [源站端口与 Origin Rules：端口 override 的适用边界与冲突陷阱](docs/origin-rules-port-mismatch.md)

### 2. Pages & Workers 部署避坑 (Deployment Pitfalls)
- [Wrangler 部署 9106/6111 假 Token 死循环陷阱](docs/wrangler-9106-token-trap.md)
- [Pages 部署 SPA (React/Vite) 出现白屏的根因与修复](docs/pages-spa-white-screen.md)
- [Error 1014: CNAME Cross-User Banned 跨账户绑定拦截](docs/error-1014-cname-banned.md)
- [Error 100117: 根域名 (Apex Domain) 绑定冲突](docs/error-100117-apex-domain.md)

### 3. 架构与国内网络适配 (Architecture & CN Network)
- [Pages Functions：全栈应用 (Express/API) 迁移指南](docs/express-to-pages-functions.md)
- [403 白页与 ICP 备案拦截的混合内容 (Mixed Content) 原理](docs/icp-403-mixed-content.md)
- [Worker 单例状态陷阱 (Works Once Then Fails)](docs/worker-singleton-trap.md)

### 4. 安全与侦察 (Security & Recon)
- [Cloudflare 真实源 IP 泄露的常见途径与防御](docs/origin-ip-leak-recon.md)
- [522 Connection Timed Out 的真实含义与误区](docs/error-522-myth.md)

---

## 贡献 (Contributing)

欢迎提交 PR 补充你遇到的玄学报错和填坑记录！我们不需要基础的 API 教程，我们需要的是**“为什么它坏了，以及如何真正修好它”**。

## 许可证 (License)

MIT License
