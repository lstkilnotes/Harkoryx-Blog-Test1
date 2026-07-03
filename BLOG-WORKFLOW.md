# Harkoryx Blog — 工作流

> **核心原则：绝不直接在 `main` 分支上编辑。所有改动经分支 → 验证 → 推送 → PR → 主人审核 → 合并。**

---

## 项目概览

| 项目 | 值 |
|------|-----|
| 框架 | Astro 7.0.6 + MDX + Sitemap + RSS |
| 部署 | Cloudflare Pages（从 GitHub `main` 分支自动部署） |
| 仓库 | `https://github.com/lstkilnotes/Harkoryx-Blog-Test1` |
| 本地路径 | `/root/user_documents/Long_Term_Projects/Personal_Blog_Test/Harkoryx-Blog-Test1` |

### 关键目录结构

```
src/
├── content/blog/       ← 📝 博客文章（md/mdx）
├── components/         ← 🧩 UI 组件
├── layouts/            ← 📐 页面布局
├── pages/              ← 🗺️ 路由页面
│   └── blog/           ← 博客列表 + 文章页
├── assets/             ← 🖼️ 图片、字体等静态资源
├── styles/             ← 🎨 全局样式
├── consts.ts           ← 站点常量（标题、描述）
└── content.config.ts   ← 内容集合 schema 定义
```

### 文章 Frontmatter Schema

```yaml
title: string          # 必填
description: string    # 必填
pubDate: date          # 必填，如 'Jul 08 2022'
updatedDate: date      # 可选
heroImage: image       # 可选，相对路径
```

---

## 多人协作架构

### 参与者

| 参与者 | 身份 | 分支前缀 | Git worktree 路径 |
|--------|------|----------|-------------------|
| 主人 | 人类（最终审核者） | `owner/` | 直接在主 repo 操作 |
| 绫 🦋 | AI Agent (`main`) | `aya/` | `<repo>/worktrees/aya/` |
| 萌華 🌸 | AI Agent (`spicy_leisure`) | `moka/` | `<repo>/worktrees/moka/` |
| 塞娜 💎 | AI Agent (`basic_exp`) | `sena/` | `<repo>/worktrees/sena/` |

### 核心问题：版本同步

**问题**：多人编辑同一 repo 时，可能出现——
1. 基于过时的 `main` 创建分支 → 合并时冲突
2. 两人同时改同一文件 → 互相覆盖
3. Agent A 的本地 `main` 落后于远程 → 不知道别人已经改了什么

**解决方案：Git Worktree + 每次操作前强制 pull**

### Git Worktree 方案

每个 Agent 使用独立的 **git worktree**（共享同一个 `.git` 仓库，但拥有独立的工作目录和分支）。这样：

- ✅ 各 Agent 不会互相干扰工作目录
- ✅ 共享同一个本地仓库对象，节省空间
- ✅ 每个 worktree 只能在一个分支上，避免误切
- ✅ 所有 worktree 共享同一个 pre-push 钩子

```bash
# 仓库根目录
REPO=/root/user_documents/Long_Term_Projects/Personal_Blog_Test/Harkoryx-Blog-Test1

# 创建各 Agent 的 worktree（首次设置，一次性操作）
cd $REPO
git worktree add worktrees/aya    -b aya/work  # 绫的工作树
git worktree add worktrees/moka   -b moka/work # 萌華的工作树
git worktree add worktrees/sena   -b sena/work # 塞娜的工作树
```

### 操作前同步协议（所有参与者必须遵守）

**每次开始新操作前，必须执行：**

```bash
# 1. 切回待命分支
#    Agent: git checkout <prefix>/work（如 aya/work）
#    主人:  无需，直接在 main 上

# 2. 更新远程 main
#    git fetch origin main

# 3. 基于最新的远程 main 创建分支
#    git checkout -b <prefix>/<type>/<name> origin/main
```

**这确保了：**
- 从待命分支出发，不会误在旧工作分支上操作
- 每次都从**最新的远程 main** 创建分支
- 不会基于过时的本地状态工作
- 减少合并冲突的概率

---

## 初始化流程（每次编辑前必须执行）

> **核心原则：每次开始新任务前必须初始化。绝不跳过。**

### 快速方式：使用初始化脚本

```bash
# 用法: ./blog-init.sh <prefix> <type> <name>
#   prefix: aya / moka / sena / owner
#   type:   content / feat / fix / style
#   name:   分支名

# 示例
./blog-init.sh aya content my-first-article
./blog-init.sh moka feat add-tags
./blog-init.sh owner fix rss-link
```

脚本会自动完成以下 7 步：

### 手动方式：7 步初始化

| 步骤 | 操作 | 目的 |
|:----:|------|------|
| 1 | `git status` | 检查工作区是否干净，有未提交改动则拒绝开始新任务 |
| 2 | `git checkout <idle-branch>` | 切回待命分支（如 `aya/work`），确保从干净状态出发 |
| 3 | `git fetch origin main` | 同步 GitHub 上最新的 main |
| 4 | `git checkout -b <prefix>/<type>/<name> origin/main` | 基于**最新远程 main** 创建分支 |
| 5 | `ls node_modules/ || npm install` | 确认依赖就绪 |
| 6 | `npm run check`（即 `astro check`）| 类型检查——改之前先确认当前代码无类型错误 |
| 7 | `npm run build`（即 `astro build`）| 构建验证——改之前先确认当前代码能构建通过 |

**第 2 步容易被忽略**：如果上次工作后忘了切回待命分支，当前可能还停留在旧的工作分支上。初始化时先切回待命分支，再从 `origin/main` 创建新分支。

**第 4 步最关键**：必须从 `origin/main` 创建分支，不是本地 `main`。本地 main 可能落后于远程。

各参与者的待命分支：

| 参与者 | 待命分支 |
|--------|----------|
| 主人 | `main` |
| 绫 | `aya/work` |
| 萌華 | `moka/work` |
| 塞娜 | `sena/work` |

---

## 安全工作流

### 分支命名规范

| 参与者 | 格式 | 示例 |
|--------|------|------|
| 主人 | `owner/<type>/<name>` | `owner/content/my-article` |
| 绫 | `aya/<type>/<name>` | `aya/content/my-article` |
| 萌華 | `moka/<type>/<name>` | `moka/feat/add-tags` |
| 塞娜 | `sena/<type>/<name>` | `sena/fix/rss-link` |

`<type>` 取值：`content`（文章）、`feat`（功能）、`fix`（修复）、`style`（样式）

### 流程总览

```
origin/main ──→ fetch ──→ 创建分支 ──→ 编辑 ──→ 构建验证 ──→ push ──→ PR ──→ ⛔ 主人审核 ──→ 合并
                  ↑                                                                    ↑
           每次操作前强制执行                                              只有主人能合并
```

> **⚠️ 核心规则：PR 创建后，任何 Agent 绝不自行合并。必须等主人在 GitHub 上审核并确认合并。**

### 一、撰写文章（低风险）

```bash
# 1. 进入自己的 worktree
cd $REPO/worktrees/<my-worktree>

# 2. 同步远程 main
git fetch origin main

# 3. 基于最新 main 创建文章分支
git checkout -b <prefix>/content/<article-slug> origin/main

# 4. 在 src/content/blog/ 下创建 .md 或 .mdx 文件

# 5. 类型检查 + 构建验证
npm run check && npm run build

# 6. 提交 + 推送
git add -A && git commit -m "content: 添加文章 <title>"
git push -u origin <prefix>/content/<article-slug>

# 7. 创建 PR，通知主人审核
#    ⛔ Agent 绝不自行合并 PR
```

### 二、修改代码（较高风险）

```bash
# 1. 进入自己的 worktree
cd $REPO/worktrees/<my-worktree>

# 2. 同步远程 main
git fetch origin main

# 3. 基于最新 main 创建功能分支
git checkout -b <prefix>/feat/<name> origin/main

# 4. 编辑代码...

# 5. 类型检查 + 构建验证（必须通过）
npm run check && npm run build

# 6. 可选：本地预览
npx astro dev --background
#    访问 http://localhost:4321 确认效果
#    完成后: npx astro dev stop

# 7. 提交 + 推送
git add -A && git commit -m "feat: <描述>"
git push -u origin <prefix>/feat/<name>

# 8. 创建 PR，通知主人审核
#    ⛔ Agent 绝不自行合并 PR
```

### 三、紧急修复（热修复）

```bash
# 1. 进入自己的 worktree
cd $REPO/worktrees/<my-worktree>

# 2. 同步远程 main
git fetch origin main

# 3. 基于最新 main 创建修复分支
git checkout -b <prefix>/fix/<name> origin/main

# 4. 修复 + 验证
npm run check && npm run build

# 5. 提交 + 推送
git add -A && git commit -m "fix: <描述>"
git push -u origin <prefix>/fix/<name>

# 6. 创建 PR，通知主人审核
#    ⛔ Agent 绝不自行合并 PR
```

### 四、主人直接操作

主人可以直接在主 repo 目录操作，也可以使用 worktree。流程相同：
```bash
cd $REPO  # 或 cd $REPO/worktrees/<某个>
git fetch origin main
git checkout -b owner/<type>/<name> origin/main
# ... 编辑 → 验证 → push → PR → 自己在 GitHub 审核合并
```

---

## 安全防护措施（五层）

### 1. GitHub Branch Protection（服务端硬约束 🏆）

- ✅ Require a pull request before merging — 禁止直接推 `main`
- ✅ Require approvals (1) — PR 必须有审核
- ✅ Do not allow bypassing — admin 也不能绕过
- **无法绕过**：即使本地 `--no-verify` 或 API 调用，GitHub 都会拒绝

### 2. Git pre-push 钩子（本地提醒层）

已安装在 `.git/hooks/pre-push`。当检测到直接推送至 `main` 时会拒绝。
可被 `--no-verify` 绕过，但配合 GitHub Branch Protection 形成双重保障。

### 3. Check + 构建验证（必须通过）

每次推送前**必须**运行 `npm run check`（类型检查）和 `npm run build`（构建）并确认成功。
- `check` 能提前发现 frontmatter schema 违规、组件 prop 类型错误
- `build` 确认代码能正常编译
如果任一步失败，说明改动有问题，不得推送。

### 4. Git Worktree 隔离（多人协作层）

- 各 Agent 在独立 worktree 中操作，不会互相覆盖工作目录
- 每个 worktree 只能在一个分支上
- 共享同一 `.git` 仓库和 pre-push 钩子

### 5. 主人审核（最终防线）⛔

- **PR 创建后，任何 Agent 绝不自行合并**
- Agent 完成工作后通知主人，由主人在 GitHub 上审核并合并
- 这是最关键的安全措施——即使前四层全部通过，最终合并权只在主人手中

---

## Agent 操作规范

### ⛔ 最高优先级：绝不自行合并 PR

Agent 可以创建分支、编辑文件、构建验证、提交推送、创建 PR——
但**合并 PR 这一步，永远等主人亲自操作**。

### 操作前同步（强制）

每次开始新任务前，必须：
```bash
# 1. 切回待命分支
git checkout <prefix>/work    # 如 aya/work
# 2. 同步远程
git fetch origin main
# 3. 基于最新远程 main 创建分支
git checkout -b <prefix>/<type>/<name> origin/main
```
**绝不基于本地已有的分支继续工作**，除非是明确要继续上次未完成的任务。

### 写文章时

1. **自动生成 frontmatter**：确保符合 `content.config.ts` 的 schema
2. **slug 规范**：使用小写英文 + 连字符，如 `my-article-slug`
3. **日期格式**：`MMM DD YYYY`（如 `Jul 03 2026`）
4. **验证**：写完后运行 `npm run check && npm run build`
5. **提交信息**：使用 Conventional Commits 格式

### 改代码时

1. **先读后改**：理解现有代码再修改
2. **最小改动**：只改必要的部分
3. **验证**：改完后必须通过 `npm run check && npm run build`
4. **提交信息**：使用 Conventional Commits 格式

### Conventional Commits 格式

| 类型 | 用途 |
|------|------|
| `content:` | 新增/修改文章 |
| `feat:` | 新功能 |
| `fix:` | 修复 bug |
| `style:` | 样式调整 |
| `refactor:` | 代码重构 |
| `chore:` | 杂项（配置、依赖等） |

---

## Worktree 管理命令

```bash
REPO=/root/user_documents/Long_Term_Projects/Personal_Blog_Test/Harkoryx-Blog-Test1

# 查看所有 worktree
cd $REPO && git worktree list

# 创建 Agent worktree（首次）
cd $REPO
git worktree add worktrees/aya  -b aya/work
git worktree add worktrees/moka -b moka/work
git worktree add worktrees/sena -b sena/work

# 清理已合并的 Agent 分支
cd $REPO
git worktree remove worktrees/aya  # 如果需要
git branch -d aya/work

# 清理已合并的远程分支
git fetch --prune
```

---

## 常用命令速查

```bash
REPO=/root/user_documents/Long_Term_Projects/Personal_Blog_Test/Harkoryx-Blog-Test1

# check + 构建验证
cd $REPO && npm run check && npm run build

# 本地开发服务器
npx astro dev --background    # 启动
npx astro dev stop            # 停止

# 同步 + 创建分支（每次操作前）
git fetch origin main
git checkout -b <prefix>/<type>/<name> origin/main

# 提交
git add -A && git commit -m "type: 描述"

# 推送
git push -u origin <branch-name>

# 创建 PR（需要 gh cli）
gh pr create --title "标题" --body "描述"

# 清理已合并的分支
git branch -d <branch-name>            # 删除本地
git push origin --delete <branch-name>  # 删除远程
```

---

## 冲突处理

当 PR 合并时出现冲突：

1. **文章冲突**（最常见）：两人同时改了 `src/content/blog/` 下不同文件 → 通常不会冲突
2. **代码冲突**：两人同时改了同一组件 → 需要手动解决

```bash
# 在 worktree 中解决冲突
git fetch origin main
git rebase origin/main
# 解决冲突...
git add <冲突文件>
git rebase --continue
npm run check && npm run build  # 验证
git push --force-with-lease  # 安全强制推送（rebase 后需要）
```

---

*最后更新：2026-07-03*
