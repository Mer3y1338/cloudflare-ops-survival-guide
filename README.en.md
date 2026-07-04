# 🛡️☁️ Cloudflare Ops Survival Guide

**A field-tested survival guide for Cloudflare operations — battle notes distilled from late-night outages, weird edge cases, and production incidents.**

[![GitHub stars](https://img.shields.io/github/stars/Mer3y1338/cloudflare-ops-survival-guide?style=flat-square)](https://github.com/Mer3y1338/cloudflare-ops-survival-guide/stargazers)
[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg?style=flat-square)](https://opensource.org/licenses/MIT)
[![Docs](https://img.shields.io/badge/docs-13%20guides-blue?style=flat-square)](#contents)
[![PRs Welcome](https://img.shields.io/badge/PRs-welcome-brightgreen.svg?style=flat-square)](https://github.com/Mer3y1338/cloudflare-ops-survival-guide/pulls)

**Languages:** [简体中文](README.md) · **English**

*This is not a translated copy of the Cloudflare API reference. It is a practical SOP collection forged from real `525`, `522`, `9106`, routing, SSL, Pages, Workers, and CDN debugging sessions.*

**[Quick Start](#contents)** · **[Contributing](#contributing)** · **[Issues](https://github.com/Mer3y1338/cloudflare-ops-survival-guide/issues)**

---

## Why does this project exist?

When you operate Cloudflare — or similar edge/CDN platforms such as Tencent EdgeOne — the official documentation usually explains the happy path. Real production work is different. You run into questions like:

- Why does `525 SSL handshake failed` keep happening even though the origin server is alive?
- Why does `wrangler deploy` return `Authentication failed [code: 9106]` when the token looks correct?
- Why does a React/Vite SPA deployed to Pages show a blank white screen with only the page title left?
- Why do users in mainland China sometimes hit mysterious 403 white pages?

This repository collects the kinds of **dangerous, practical failure modes that official docs often skip, plus the troubleshooting SOPs that actually fix them**.

## Contents

### 0. Auth & Setup
- [How to correctly obtain a Cloudflare API Token and Account ID](docs/how-to-get-api-token.md)

### 1. Origin SSL & Routing
- [Error 525: SSL handshake failed — ultimate troubleshooting guide](docs/error-525-survival-guide.md)
- [EdgeOne / non-Cloudflare CDN 521/525 origin variants](docs/edgeone-origin-521-525.md)
- [Origin ports and Origin Rules: when port overrides help, and when they break things](docs/origin-rules-port-mismatch.md)

### 2. Pages & Workers Deployment Pitfalls
- [Wrangler 9106/6111 fake-token dead-loop trap](docs/wrangler-9106-token-trap.md)
- [Why Cloudflare Pages deployments of React/Vite SPAs show a blank page, and how to fix it](docs/pages-spa-white-screen.md)
- [Error 1014: CNAME Cross-User Banned](docs/error-1014-cname-banned.md)
- [Error 100117: Apex domain binding conflict](docs/error-100117-apex-domain.md)

### 3. Architecture & Mainland China Network Adaptation
- [Pages Functions: migrating full-stack Express/API apps](docs/express-to-pages-functions.md)
- [403 white pages and ICP-related mixed-content blocking](docs/icp-403-mixed-content.md)
- [Worker singleton state trap: “works once, then fails”](docs/worker-singleton-trap.md)

### 4. Security & Recon
- [Common ways real origin IPs leak behind Cloudflare, and how to defend against them](docs/origin-ip-leak-recon.md)
- [What 522 Connection Timed Out really means, and common misconceptions](docs/error-522-myth.md)

---

## Contributing

PRs are welcome. The most valuable contributions are not basic API tutorials, but real failure records: **what broke, why it broke, how you proved the root cause, and what actually fixed it**.

Good contributions usually include:

- the exact symptom or error code;
- the environment where it happened;
- failed assumptions or misleading official-doc paths;
- reproducible checks or commands;
- the final working fix;
- caveats, rollback notes, and prevention tips.

## License

MIT License

Special thanks to the Linux.do community and GitHub developers for their support and contributions.
