---
title: 用 Claude Code + Obsidian + Git 构建个人研究知识库
tags:
  - 教程
  - Obsidian
  - Git
  - VSCode
  - CC
---
# 用 Claude Code + Obsidian + Git 构建个人研究知识库

> 本教程以构建自己的 llm-wiki 为实际案例，手把手讲解如何将三个工具结合起来，持续积累、整理和检索研究资料。

---
## 目录

1. [四个工具是什么](#一四个工具是什么)
2. [为什么组合使用](#二为什么组合使用)
3. [环境准备](#三环境准备)
4. [Claude Code 用法详解](#四claude-code-用法详解)
5. [Obsidian 用法详解](#五obsidian-用法详解)
6. [Git 工作流](#六git-工作流)
7. [把它们串起来：一次完整循环](#七把它们串起来一次完整循环)

---

## 一、四个工具是什么

### Obsidian

Obsidian 是一款**本地 Markdown 笔记软件**。你的所有笔记都存储在自己的电脑上（不依赖云端），以普通 `.md` 文本文件的形式保存，可以用任何编辑器打开。

它最核心的特性是**双向链接**：在一篇笔记中写 `[[另一篇笔记]]`，两篇笔记就被关联起来了。积累足够多的笔记后，Obsidian 能渲染出一张知识图谱，直观展示概念之间的关联网络。

### Git

Git 是一个**版本控制工具**，可以把你对文件的每一次修改都记录下来，像"存档点"一样。随时可以查看历史、对比差异、或回退到之前的版本。

对知识库来说，Git 的价值在于：
- **防止意外删除**：误操作后可以版本回退
- **记录演变历程**：知道某个观点是什么时候加进来的
- **多设备同步**：通过 GitHub 在不同电脑间同步

### Claude Code

Claude Code 是 Anthropic 官方出品的 **AI 命令行工具**。在终端里输入 `claude`，就能与 AI 对话——不只是聊天，还能让 AI 直接读取、创建、修改你项目里的文件，执行复杂的批量处理任务。

### VSCode

VSCode（Visual Studio Code）是微软出品的**轻量级代码编辑器**，免费、跨平台、插件生态丰富。它能打开任意文件夹作为工作区，提供文件树、搜索、Git 集成、终端等开箱即用的功能。

Obsidian 只显示 `.md` 文件，看不到隐藏文件（如 `.gitignore`、`.obsidian/`、`CLAUDE.md` 等配置文件）和非 Markdown 文件。VSCode 补上这块短板：

- **查看与编辑配置文件**：调整 `.gitignore`、`CLAUDE.md`、`.obsidian/` 下的 JSON 配置
- **集成终端**：直接在编辑器内启动 `claude`、运行 `git` 命令，不用切窗口

---

## 二、为什么组合使用

单独使用每个工具都有局限：

| 单独使用 | 局限 |
|---|---|
| 只用 Obsidian | 整理归纳全靠手工，耗时费力 |
| 只用 Claude Code | AI 生成的内容散落在对话中，无法持续积累 |
| 只用 Git | 纯代码工具，没有可视化浏览界面 |

三者组合后，**分工清晰，互补**：

```
原始素材
    ↓
Claude Code（批量处理、智能提取）
    ↓
Markdown 文件（存储在本地）
    ↓
Obsidian（组织、浏览、关联）
    ↓
Git（版本控制、备份、历史追踪）
```

**实际效果**：你只需要提供原材料，Claude Code 帮你整理成结构化笔记，Obsidian 让你在笔记间自由跳转，Git 保证数据安全。

---

## 三、环境准备

### 1. Obsidian

安装后打开 Obsidian，选中你的项目文件夹。Obsidian 会把该文件夹下所有 `.md` 文件作为笔记管理。

> 本项目中，`/snowball` 整个目录就是一个 `Vault`。

### 2. Git

初始化 Git 仓库（新项目）：
```
cd 你的项目目录
git init
```

### 3. VSCode

安装 VSCode 命令行插件后，使用
```
code 你的项目目录
```
命令打开当前文件夹。

---

## 四、Claude Code 用法详解

Claude Code 在这套流程里扮演的是"知识库维护者"——读原始素材、写笔记、改交叉引用、追加日志。这一节讲清楚日常用得到的几个核心机制。

### 1. 启动与基本对话

在你的项目目录下打开终端：

```bash
cd 你的项目目录
claude
```

首次运行会引导你登录 Anthropic 账号。之后每次进项目，敲一下 `claude` 就进入对话界面。

几个最基础的操作：

| 操作           | 用法                        |
| ------------ | ------------------------- |
| 发送消息         | 直接打字，回车                   |
| 中断 Claude 执行 | 按 `Esc`（它会停下，但保留上下文）      |
| 清空当前输入内容     | 连按两次 `Esc`                |
| 多行输入         | `Option+Enter` 换行         |
| 退出会话         | `Ctrl+C` 两次，或输入 `/exit`命令 |
| 短暂切回命令行      | 输入`!`                     |
| 指定文件或文件夹     | 输入`@`                     |
| 查看可用命令       | 输入`/`                     |

### 2. CLAUDE.md：项目记忆

> Claude Code 每次进入这个目录时，会**自动读取**项目根目录的 `CLAUDE.md` 作为上下文（context）的一部分——这是一条很重要的约定。

在 llm-wiki 模式下，`CLAUDE.md` 就是 [`llm-wiki.md`](llm-wiki.md) 里讲的"schema 层"——你把 wiki 的组织规则写在这里，Claude Code 才会按规则维护，而不是每次都临时发挥。

**生成初版**：在 Claude Code 里敲 `/init`，它会扫描当前目录、给出一份初版 `CLAUDE.md`。然后你手动调整。

一份**最小可用**的 schema 大致长这样（写在 `CLAUDE.md` 里）：

```markdown
# 项目说明：个人 LLM Wiki

## 目录约定
- raw/        # 原始素材（只读，不要修改）
- wiki/       # 你（Claude）维护的笔记
  - wiki/index.md   # 全部页面的索引
  - wiki/log.md     # 按时间排序的操作日志
  - wiki/entities/  # 实体页（人物、组织、概念）
  - wiki/topics/    # 主题页

## Ingest 流程（用户说"ingest xxx"时执行）
1. 读 raw/xxx
2. 在 wiki/ 下写一篇摘要，文件名贴合标题
3. 更新或新建相关实体/主题页，加上 [[双向链接]]
4. 在 wiki/index.md 增加条目
5. 在 wiki/log.md 末尾追加：## [YYYY-MM-DD] ingest | 标题

## 写作风格
- 中文为主，专业术语保留英文
- 摘要 200~500 字，关键观点用列表
```

你后续可以根据自己的需求往里加规则。

**`CLAUDE.local.md`：本地私有补充**

除了会被 git 提交、团队/多设备共享的 `CLAUDE.md` 之外，Claude Code 还会自动读取同目录下的 `CLAUDE.local.md`（如果存在）。它的定位是**只属于这台电脑、只属于你自己**的补充上下文：

- **典型用途**：本机路径、个人偏好的写作风格、临时性的实验性指令、不便公开的项目背景、私人备注
- **关键约定**：把 `CLAUDE.local.md` 写进 `.gitignore`，让它**不进 git 历史**，也就不会同步到 GitHub 或其他设备
- **优先级**：内容会和 `CLAUDE.md` 一起注入上下文，可以覆盖或补充共享规则——例如 `CLAUDE.md` 规定"摘要 200~500 字"，你可以在 `CLAUDE.local.md` 里写"我个人偏好 100 字以内的极简摘要"

一份示例 `CLAUDE.local.md`：

```markdown
# 本机私有上下文

## 个人偏好
- 回答里少用客套话，直接给结论
- 代码示例优先用 Python，其次 TypeScript

## 本机路径
- 我的素材备份目录在 ~/Dropbox/raw-backup/

## 临时任务
- 最近在重构 wiki/topics/，旧文件先别动
```

> **小结**：`CLAUDE.md` 是"项目规则"，跟仓库走；`CLAUDE.local.md` 是"个人批注"，留在本机。两者并存，互不干扰。

### 3. 常用 Slash 命令

在对话框里输入 `/` 会弹出命令菜单。最常用的几个：

| 命令             | 作用              | 什么时候用                             |
| -------------- | --------------- | --------------------------------- |
| `/init`        | 生成 CLAUDE.md 初版 | 新项目第一次配置                          |
| `/rewind`      | 回滚              | 回滚到之前的对话                          |
| `/resume`      | 恢复以前的会话         | 想接着上次断掉的对话继续                      |
| `/model`       | 切换模型            | 在 Opus / Sonnet / Haiku 之间换       |
| `/effort`      | 调整模型思考时间        | 复杂任务用 high effort，简单任务 low effort 即可 |
| `/config`      | 打开设置            | 调主题、输出风格等等                        |
| `/permissions` | 管理工具授权          | 让某些命令不再每次询问                       |
| `/clear`       | 清空当前会话上下文       | 切换到无关的新任务时                        |
| `/plan`        | 进入"计划模式"        | 改动复杂或不确定时，先让 Claude 给方案、不动文件      |
| `/btw`         | 不污染主对话的快问快答     | 中途想问个小问题，又不想打断当前任务（对话内容不会进入上下文）   |
| `/diff`        | 看未提交的改动         | 回顾这次会话改了哪些文件                      |
| `/help`        | 帮助              | 忘了某个命令时                           |

### 4. Plan / Edit / Auto 三种工作模式

按 `Shift+Tab` 可以在三种模式间循环切换：

| 模式 | 行为 | 适用场景 |
|---|---|---|
| **Plan**（计划） | 只读，给方案，不改文件 | 大改动前先讨论；不确定文件结构时探索 |
| **Edit**（默认） | 每次写文件前征求确认 | 日常使用，安全 |
| **Auto**（自动接受） | 直接改、不问 | 你完全信任当前任务，例如批量格式化 |

> 经验法则：**复杂任务先 `/plan` → 确认方向 → 切回 Edit 让它干活。**

### 5. .gitignore 与文件可见性

Claude Code 默认**尊重 `.gitignore`**——被 ignore 的文件它读不到、也不会改。这个性质很有用：

- 想写点暂时不想被 LLM 看到的草稿？放到一个被 ignore 的目录里
- 敏感信息（个人日记、API key）？写进 `.gitignore`，一次解决两个问题：既不进 git 历史也不进 LLM 上下文

这一点和后面第六章的 Git 配置是联动的——下面会再展开。

### 6. 用 `settings.json` 的 deny 规则进一步约束 Claude

`.gitignore` 是"顺带"挡住 Claude——它本职是 git 的过滤器，只是 Claude Code 也尊重它。如果你想让某些文件进 git，但不让 Claude 读/改，就要用 Claude Code 自己的权限机制：项目根目录下 `.claude/settings.json`（或 `.claude/settings.local.json`）里的 `permissions.deny` 字段。

一个最小例子：

```json
{
  "permissions": {
    "deny": [
      "Read(./.obsidian/**)"
    ]
  }
}
```

它和 `.gitignore` 互不影响——一份文件可以"进 git 但 Claude 看不到"，也可以反过来。

**`settings.json` vs `settings.local.json`**：前者会被 git 提交、团队共享；后者只属于本机（建议加进 `.gitignore`）。和 `CLAUDE.md` / `CLAUDE.local.md` 是同样的分工。

### 7. Plugins：扩展 Claude Code 的能力

Plugin 是 Claude Code 的**扩展机制**——一个 plugin 可以打包 slash 命令、subagent、hook、MCP server 等多种能力，一次安装全部到位。装一个好的 plugin，相当于给 Claude Code 现成搬来一整套工作流。

**安装与管理**

在对话框输入 `/plugin` 进入插件管理界面：

- 浏览市场（marketplace）里现成的 plugin
- 选中安装、启用/禁用、卸载，全部在这里完成

如果想加官方市场之外的源（比如团队私有仓库或第三方市场）：

```
/plugin marketplace add <git-url-or-path>
```

之后再 `/plugin` 就能在新加的市场里挑选。

**plugin 装在哪里**

- **用户级**：`~/.claude/plugins/`，跟着账号走，所有项目都能用
- **项目级**：`.claude/plugins/`，跟着仓库走

如果团队多人协作，把共用 plugin 放到项目级——别人 `git clone` 下来环境就齐了。但记得这个目录默认在 `.gitignore` 里被忽略（见前面第 5 小节的示例），团队共享时要从忽略列表里去掉。
### 8. Skills：可复用的技能包

Skill 比 plugin 更轻量——一个 skill 就是**一份文本文件，加上一些执行命令的脚本**，里面写清楚"这个技能干什么、按什么步骤干"。Claude Code 在理解你意图的时候，如果发现某个 skill 的描述匹配，就会主动调用它；你也可以用 `/` 命令显式触发。

**Skill 文件长什么样**

放在 `~/.claude/skills/<skill-name>/`（用户级）或 `.claude/skills/<skill-name>/`（项目级），例如：

```markdown
---
name: ingest-paper
description: 把一篇学术论文 PDF 整理成 wiki 摘要，提取作者、摘要、关键观点
---

# Ingest Paper

接到任务后按以下步骤执行：
1. 用 Read 工具读 PDF 全文
2. 提取标题、作者、年份、摘要、3-5 个关键观点
3. 在 wiki/papers/ 下建一篇笔记，文件名 = "作者-年份-标题"
4. 在正文中给作者建 [[实体页]] 双向链接
5. 把这篇笔记追加到 wiki/index.md 的"论文"区块
```

frontmatter 的 `description` 是 Claude 决定"这个 skill 是否相关"的依据——写得越具体、越贴近触发场景，自动调用的命中率越高。

**两种触发方式**

- **自动调用**：你正常对话，Claude 识别到意图匹配就自己用。比如你说"帮我整理一下 raw/2024-attention.pdf 这篇论文"，匹配到上面这个 skill 就会按 5 个步骤执行
- **手动调用**：在对话框输入 `/<skill-name>`，比如 `/ingest-paper`，直接触发

**Skill / CLAUDE.md / Plugin 的分工**

三者经常一起用，但定位不同：

| 机制          | 触发方式            | 适合放什么                       |
| ----------- | --------------- | --------------------------- |
| `CLAUDE.md` | 每次进入项目自动注入      | 全局规则、目录约定、写作风格              |
| Skill       | 按需调用（自动识别或 `/`） | 某个具体任务的"标准动作"               |
| Plugin      | 安装后常驻           | 一组命令/agent/hook/MCP 的集合，可分发 |

经验法则：**全局规则写进 `CLAUDE.md`；具体任务的固定流程拆成 skill；要打包分发给别人用就做成 plugin。**

---

## 五、Obsidian 用法详解

Obsidian 的角色是**人类一侧的浏览界面**。Claude Code 在写、Git 在存档、Obsidian 让你看得见。

### 1. Vault 与 `.obsidian/` 目录

Vault（库）就是任意一个文件夹。你在 Obsidian 里"打开"一个文件夹，它就成了 Vault。打开的瞬间，Obsidian 会在这个文件夹里创建一个隐藏目录：

```
你的项目/
└── .obsidian/        ← Obsidian 的所有配置都在这里
    ├── plugins/
    ├── core-plugins.json
    └── workspace.json
```

**关键点：所有 Obsidian 配置都跟着 Vault 走。** 这意味着：

- 把整个项目目录（**包括 `.obsidian/`**）一起 git 提交
- 换电脑 `git clone` 下来，Obsidian 配置自动还原——插件、主题、快捷键、布局，都不用重配

这是为什么后面第六章建议**不要**把 `.obsidian/` 整个 ignore 掉。

### 2. 插件安装位置

Obsidian 第三方插件实际安装在：

```
.obsidian/plugins/<plugin-name>/
```

每个插件就是一个目录，里面是 JS 代码和配置。安装步骤：

1. 打开 Obsidian → 设置 → **第三方插件** → 关闭"安全模式"
2. 点"浏览"，搜索想要的插件，安装、启用
3. 装完后去 `.obsidian/plugins/` 下能看到对应目录

> 如果你想把这套环境分享给别人，把整个项目目录打包就够了——他们 clone 下来打开 Obsidian，插件已经装好。

推荐先装这几个（按需选）：

- **Obsidian Web Clipper**（浏览器扩展，不在 Obsidian 内装）：把网页一键转 markdown 落到 Vault
- **Claudian**：在 Obsidian 内直接调起 Claude Code 的桥接插件，让你在笔记界面里就能跟 Claude 对话、让它改当前 Vault 的文件，省去切终端
- **Git**：把 git 操作集成进 Obsidian 命令面板，提交、拉取、推送、查看历史都不用离开笔记界面
### 3. 双向链接 `[[...]]` 与图谱视图

在笔记里写 `[[某个名字]]`，Obsidian 自动建立链接。被链接的页面如果还不存在，点一下就能创建。

侧边栏的"图谱视图"会把所有链接可视化成一张网。在 llm-wiki 模式下，这张图就是 Claude Code 工作成果的全景——哪些页面是中心、哪些是孤岛，一眼可见。**lint 操作（健康检查）就是看这张图找问题。**

### 4. Web Clipper：把网页变成原始素材

安装浏览器扩展 "Obsidian Web Clipper" 后：

1. 打开任意网页
2. 点扩展图标
3. 选择目标 Vault 和保存路径（建议设为 `raw/`）
4. 一键转成 markdown 存进来

这就是 ingest 流程的入口——网页变成 `raw/` 下的一个文件，剩下交给 Claude Code。

---
## 六、Git 工作流

这一节是重点。你不需要变成 Git 高手，但下面这些命令要熟。

### 1. 初始化与第一次提交

```bash
cd 你的项目目录
git init                    # 创建仓库
git status                  # 看当前状态
git add .                   # 把所有文件加入暂存区
git commit -m "初始化知识库"   # 创建第一个版本
```

`git status` 会成为你最常用的命令——任何时候不确定"我改了什么"，敲它一下。

### 2. `.gitignore` 应该写什么

在项目根目录创建 `.gitignore` 文件，告诉 Git 哪些文件不要追踪：

```gitignore
# macOS / Windows 系统垃圾
.DS_Store
Thumbs.db

# Obsidian：保留配置，但忽略易抖动的本地状态
.obsidian/workspace*
.obsidian/cache
.trash/

# 草稿与隐私（同时也对 Claude Code 不可见）
drafts/
private/
*.local.md

# Claude Code 自己的本地缓存（如果有）
.claude/
```
### 3. 日常提交节奏

建议的节奏：**每完成一次有意义的改动就 commit 一次。**

在 llm-wiki 模式下，最自然的"一次"就是 ingest 完一篇素材：

```bash
git add .
git commit -m "ingest: 文章标题"
```

写 commit message 嫌麻烦？让 Claude Code 帮你：

> 你：帮我看一下 git status，给这次改动写一个 commit message 然后提交

### 4. 多 branch 与多 agent 并行工作

知识库做久了，你会发现想同时干好几件事。如果都在同一个目录里让一个 Claude 串着干，互相打断不说，commit 历史也会乱成一片。

Git 的 **branch** + **worktree** 机制正好能解决这个问题——给每个任务开一个独立的工作副本，让多个 Claude Code 实例真正并行跑。

**思路**：一个仓库，多个工作目录，每个目录跟一个 branch 绑定。

```bash
# 当前 branch 是 main
mkdir .trees

# 给"ingest 这周的论文"开一个worktree
git worktree add .trees/ingest-papers
```

现在磁盘上有三个目录，对应三个 branch：

| 目录                      | branch          | 用途            |
| ----------------------- | --------------- | ------------- |
| `/`                     | `main`          | 主线，干净的"已发布"状态 |
| `/.trees/ingest-papers` | `ingest-papers` | 本周新素材的 ingest |

**开多个 Claude，各管一摊**：

```bash
# 终端 1
cd / && claude

# 终端 2
cd /.trees/ingest-papers && claude
```

每个 Claude 看到的是各自 branch 的文件状态，互不干扰。主分支是干净的，ingest 那边继续正常工作。

**合并回主线**：任务做完、自己 review 过，回到主仓库合并：

```bash
git merge ingest-papers     # 把这周的 ingest 并进 main
git worktree remove .trees/ingest-papers  # 用完清掉
git branch -d ingest-papers
```

**冲突怎么办**：两个 branch 同时改了同一个 entity 页（很常见——ingest 加引用），merge 时会冲突。这时候让 Claude 帮忙：

> 你：现在在 merge `ingest-papers` 到 main，有冲突，帮我看一下 `git status`，结合两边的意图把冲突解掉

Claude 能读两边的 diff、理解各自的意图，给出比手工三方合并更靠谱的方案。

**经验法则**：

- **短任务**：在 main 上直接干，commit 完事
- **长任务 / 实验性改动**：开 worktree + branch
- **想并行跑不相关的任务**：开 worktree，每个挂一个 Claude

> Plan 模式（`/plan`）和 worktree 是搭配用的：复杂任务先在 worktree 里 `/plan` 出方案、跑出结果，确认无误再合并；不行就直接 `git worktree remove` 丢掉，主分支干净不受影响。

### 5. 推送到 GitHub

想多设备同步、或做异地备份，把仓库推到 GitHub：

```bash
git remote add origin git@github.com:你的用户名/my-wiki.git
git push origin main
```
### 6. 多设备同步

在另一台电脑上：

```bash
git clone git@github.com:你的用户名/my-wiki.git
cd my-wiki
```

打开 Obsidian → Open folder as vault → 选这个目录。因为 `.obsidian/` 跟着仓库一起 clone 下来了，**插件、主题、快捷键、布局全部还原**。

继续工作时记得先 `git pull` 拉最新，结束时 `git push`。

---

## 七、把它们串起来：一次完整循环

讲到这里，每个工具的角色应该清晰了。一次最典型的工作流长这样：

```
1. 浏览器看到一篇有意思的文章
   ↓ Web Clipper 一键剪藏
2. 文章变成 raw/某文章.md
   ↓ git status 能看到这个新文件
3. 在 Claude Code 里说："ingest raw/某文章.md，按 CLAUDE.md 流程办"
   ↓ Claude Code 读源 → 写摘要 → 更新实体页 → 加索引 → 追加日志
4. 切到 Obsidian，跟着 [[双向链接]] 浏览 Claude 写的内容
   ↓ 发现哪里写得不对就跟 Claude 讨论调整
5. 满意了：git add . && git commit -m "ingest: 某文章"
   ↓
6. 隔几天再回来：在 Claude Code 里发问、做对比、要图表
   ↓ 好的回答让 Claude 写回 wiki/，下次还能用
7. 周末花十分钟：让 Claude 跑一次 lint，找矛盾、孤岛页、漏链接
```

---
