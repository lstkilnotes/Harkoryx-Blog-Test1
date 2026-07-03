# AGENTS.md — AI 编辑指引

> 本文件是 AI Agent 的入口文档。详细工作流见 [`.ai/WORKFLOW.md`](.ai/WORKFLOW.md)

---

## Astro 开发指南

When starting the dev server, use background mode:

```
astro dev --background
```

Manage the background server with `astro dev stop`, `astro dev status`, and `astro dev logs`.

Full documentation: https://docs.astro.build

Consult these guides before working on related tasks:

- [Adding pages, dynamic routes, or middleware](https://docs.astro.build/en/guides/routing/)
- [Working with Astro components](https://docs.astro.build/en/basics/astro-components/)
- [Using React, Vue, Svelte, or other framework components](https://docs.astro.build/en/guides/framework-components/)
- [Adding or managing content](https://docs.astro.build/en/guides/content-collections/)
- [Adding styles or using Tailwind](https://docs.astro.build/en/guides/styling/)
- [Supporting multiple languages](https://docs.astro.build/en/guides/internationalization/)

---

## ⚠️ 核心规则

1. **绝不直接在 `main` 分支编辑** — 所有改动经分支 → 验证 → PR → 主人审核 → 合并
2. **绝不自行合并 PR** — 合并权只在主人手中
3. **每次编辑前必须初始化** — 运行 `./blog-init.sh <prefix> <type> <name>`

## 分支前缀

| Agent | 前缀 | 示例 |
|-------|------|------|
| 绫 (main) | `aya/` | `aya/content/my-article` |
| 萌華 (spicy_leisure) | `moka/` | `moka/feat/add-tags` |
| 塞娜 (basic_exp) | `sena/` | `sena/fix/rss-link` |
| 主人 | `owner/` | `owner/content/my-article` |

## 快速开始

```bash
# 初始化（必须）
./blog-init.sh <prefix> <type> <name>

# 编辑后验证
npm run check && npm run build

# 提交 + 推送
git add -A && git commit -m "<type>: <描述>"
git push -u origin <branch-name>

# 创建 PR，通知主人审核 ⛔
```

## 详细文档

- **完整工作流**：[`.ai/WORKFLOW.md`](.ai/WORKFLOW.md) — 初始化流程、安全防护、冲突处理、worktree 管理
- **Astro 开发文档**：https://docs.astro.build
