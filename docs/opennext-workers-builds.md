# OpenNext / Next.js Workers Builds 部署避坑

> 将 Next.js / OpenNext 项目（例如 `@opennextjs/cloudflare` 构建的应用）接入 Cloudflare Workers Builds 时，构建命令、部署命令、预览分支命令和环境变量边界很容易混淆。本指南记录一套实战 SOP。

## 适用场景

- Next.js 应用使用 `@opennextjs/cloudflare` 部署到 **Cloudflare Workers**。
- 仓库里有 `wrangler.jsonc`，`main` 指向 `.open-next/worker.js`，`assets.directory` 指向 `.open-next/assets`。
- 希望通过 Workers Builds 连接 GitHub，push 后自动构建/部署。
- 需要把运行时变量、secrets、构建期变量分开管理，避免 fork 同步上游时反复冲突。

## 推荐部署模型

```text
GitHub fork / repo
  → Workers Builds 运行 build command
  → OpenNext 生成 .open-next/worker.js + .open-next/assets
  → deploy command 上传 Worker + assets
  → Cloudflare Worker production
```

不要把敏感值提交到仓库。`wrangler.jsonc` 只保留非敏感、通用、上游友好的配置；部署环境差异优先放 Cloudflare Dashboard 的变量和 secrets 中。

## 推荐命令

### Build command

```bash
corepack enable && corepack prepare pnpm@10.30.3 --activate && pnpm install --frozen-lockfile && pnpm build:worker
```

说明：

- `corepack enable` / `corepack prepare` 确保 Cloudflare 构建环境使用项目要求的 pnpm 版本。
- `pnpm install --frozen-lockfile` 保证依赖与锁文件一致。
- `pnpm build:worker` 通常执行 `opennextjs-cloudflare build`，生成 `.open-next` 产物。

### Production deploy command

```bash
pnpm exec opennextjs-cloudflare deploy -- --keep-vars
```

`--keep-vars` 很关键：它让 Wrangler/OpenNext 部署时保留 Cloudflare Dashboard 中已有的 vars 和 secrets，避免只用仓库里的 `wrangler.jsonc` 覆盖远程配置。

### Non-production branch deploy command

Cloudflare Workers Builds 的“非生产分支部署命令”不是生产部署命令。默认值通常是：

```bash
npx wrangler versions upload
```

这会上传一个 Worker Version 用于预览，不会把 production 流量切过去。对于大多数项目，保留默认值是合理的。

如果你明确希望预览分支也执行同一套 OpenNext 部署流程，可以填：

```bash
pnpm exec opennextjs-cloudflare deploy -- --keep-vars
```

但要理解它可能会部署到同一个 Worker，需要确认你的分支/预览环境隔离策略。

## 环境变量分层

### Runtime variables（Worker 运行时变量）

在 Worker / Workers Builds 的 runtime variables 中配置非敏感运行时默认值，例如：

```text
DEPLOYMENT_MODE=hosted
RATE_LIMIT_STORE=upstash
DOCUMENT_PARSE_JOB_STORE=upstash
PLUGIN_REGISTRY_STORE=upstash
BYOK_ALLOW_EPHEMERAL_KEY=false
NEXT_PUBLIC_SITE_URL=https://example.com
```

### Secrets（Worker secrets）

敏感值用 Worker secrets：

```text
ACCESS_PASSWORD
BYOK_PRIVATE_KEY_PEM
BYOK_KEY_ID
UPSTASH_REDIS_REST_URL
UPSTASH_REDIS_REST_TOKEN
```

### Build variables（构建期变量）

`NEXT_PUBLIC_*` 这类变量可能参与 Next.js 构建，应在 Workers Builds 的 build variables 中也配置一份：

```text
NODE_VERSION=22
PNPM_VERSION=10.30.3
NEXT_PUBLIC_SITE_URL=https://example.com
```

运行时变量不一定自动出现在构建步骤里；构建期需要的变量必须放 build variables。

## `wrangler.jsonc` 管理建议

如果你的项目是 fork 上游仓库，长期维护时不要为了部署环境把大量自定义变量写进 `wrangler.jsonc`，否则上游更新该文件时容易冲突。

推荐：

```jsonc
{
  "name": "your-worker",
  "main": ".open-next/worker.js",
  "compatibility_date": "2026-07-01",
  "compatibility_flags": ["nodejs_compat"],
  "keep_vars": true,
  "assets": {
    "directory": ".open-next/assets",
    "binding": "ASSETS"
  },
  "vars": {
    "DEPLOYMENT_MODE": "hosted"
  }
}
```

把站点域名、Upstash、访问密码、BYOK 等部署环境细节放到 Cloudflare Dashboard，而不是 fork 补丁里。

## 常见坑点

### 坑 1：把“非生产分支部署命令”误当生产部署命令

看到 `npx wrangler versions upload` 时不要立刻改掉。它是预览分支默认行为：上传版本但不推广到 production。生产分支仍应使用 production deploy command。

### 坑 2：`versions upload` 后站点没更新

`wrangler versions upload` 只上传版本；如果要切生产流量，还需要 `wrangler versions deploy <version-id>@100`。因此生产部署不要只填 `versions upload`。

### 坑 3：GitHub 连接成功但 Wrangler 仍显示 `Unknown (deployment)`

Wrangler 的 `deployments list/status` 不一定把 Workers Builds 来源显示成 GitHub。判断是否触发成功，要结合：

```bash
pnpm exec wrangler deployments list --name <worker-name>
pnpm exec wrangler deployments status --name <worker-name>
curl -I https://<custom-domain>/
```

以及 Cloudflare Dashboard 的 Build 日志。

### 坑 4：GitHub OAuth 不能靠 Cloudflare API Token 代替

Workers Builds 连接 GitHub 需要在 Dashboard 中完成 GitHub OAuth / App 授权。Cloudflare API Token 可以部署 Worker、设置 secrets、绑定域名，但不能代替 GitHub 授权连接仓库。

### 坑 5：Fork 同步覆盖部署配置

如果你修改 fork 里的 `wrangler.jsonc` 来保存环境变量，将来同步上游时可能冲突或被覆盖。更稳的是恢复上游配置，把环境差异放 Dashboard，并使用 `--keep-vars`。

## 验证清单

```bash
# 1. 检查部署版本
pnpm exec wrangler deployments list --name <worker-name>
pnpm exec wrangler deployments status --name <worker-name>

# 2. 检查自定义域名
curl -I https://<custom-domain>/

# 3. 检查关键响应头
# 常见健康 OpenNext Worker 响应包含：
# x-opennext: 1
# x-powered-by: Next.js

# 4. 确认无登录访问受保护 API 时按预期返回 401
curl -i https://<custom-domain>/api/health
```

## 一句话结论

OpenNext / Next.js Workers Builds 最稳的模式是：仓库保持上游友好，构建产物由 `pnpm build:worker` 生成，production 用 `opennextjs-cloudflare deploy -- --keep-vars`，环境变量和 secrets 放 Cloudflare Dashboard。

---

*踩坑等级：⭐⭐⭐⭐ | 出现频率：中高 | 影响范围：自动部署失败、变量丢失、预览/生产混淆*
