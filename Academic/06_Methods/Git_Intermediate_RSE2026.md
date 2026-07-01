---
title: Intermediate Git — DH-RSE Summer School 2026
date: 2026-07-01
tags: [git, RSE, version-control, methods]
source: "DH-RSE Summer School 2026 / Aman Goel (SSI)"
materials:
  - "D:/Kings/RSE summer school/Intermediate-Git-DH-RSE-Summer-School.pptx"
  - "D:/Kings/RSE summer school/Intermediate Git - DH-RSE Summer School.docx"
  - "https://carpentries-incubator.github.io/byte-sized-rse-git-intermediate/"
---

# Intermediate Git — DH-RSE Summer School 2026

## 课程结构

1. Feature Branch Workflow（理论）
2. 四种合并策略（理论）
3. Handy Git Features：Stash / Cherry-pick / Reset
4. 实操练习

---

## 1. Feature Branch Workflow

### 为什么需要分支？

基础 Git 训练通常覆盖：add/commit、push、查看历史、checkout/revert。分支细节常被省略——但这是有效协作的核心。

**核心原则：`main` 应该永远处于可工作状态。**

Feature Branch Workflow：每个功能/修复开一条独立分支，做完测完 review 完，再合并回 main。

```
main:     A---B-----------M
                \         /
feature:         C---D---E
```

### 何时建分支？

- 开发新功能
- 修复 bug
- 做实验性改动
- 任何不想立刻影响 main 的工作

---

## 2. 四种合并策略

### 2.1 Fast-Forward Merge

**前提**：main 自建立 feature 分支后没有新提交。

```
main:    A---B
              \
feature:       C---D
```

合并后：main 指针直接推进到 D，**无 merge commit**，历史为直线。

```
main:    A---B---C---D
```

- 优点：历史干净
- 缺点：看不出这段是从分支合入的

---

### 2.2 3-Way Merge（最常见）

**前提**：main 和 feature 都有新提交，已分叉。

```
main:    A---B---E
              \
feature:       C---D
```

Git 找三个点：B（共同祖先）、E（main 最新）、D（feature 最新），生成 **merge commit M**。

```
main:    A---B---E---M
              \     /
feature:       C---D
```

- 优点：历史完整，能看出合并点
- 缺点：历史图有分叉

> **Merge commit**：Git 自动生成的特殊提交，有**两个父节点**（E 和 D），本身不含业务改动，只记录"两条线在此汇合"。`git log --graph` 能可视化这个分叉结构。

---

### 2.3 Rebase & Merge

将 feature 的提交"重新播放"在 main 最新点上——改写历史，使其看起来像是在最新 main 基础上写的。

```
前：
main:    A---B---E
              \
feature:       C---D

rebase 后：
main:    A---B---E
                  \
feature:           C'---D'   ← SHA 已变，是新的提交对象
```

再 fast-forward 合入 main → 历史为直线，但 C/D 的 SHA 已重写。

> **黄金铁律（Golden Rule of Rebasing）**：
> **永远不要 rebase 公共分支**（其他人也在使用的分支）。
> 原因：你改写了 SHA，别人本地的历史与远端对不上，会产生大量冲突和混乱。
> 只对**本地**、**只有你自己用**的分支 rebase。

> **Rebase 方向规则**（课堂补充）：
> 技术上所有分支是对等的，但实践中永远应该是：**在 feature 分支上 rebase，然后 merge 进 main**。
> 不要反过来——不要把 main rebase 到 feature 上。
> 合适时机：开 PR 之前，feature 分支开发完成、准备合并时，可以 rebase 一次来整理历史。

> **Rebase 时的合并冲突**（课堂问答）：
> Rebase 和 3-way merge 都可能遇到合并冲突，区别在于**冲突发生的时机不同**：
> - 3-way merge：冲突在最终合并时暴露
> - Rebase：冲突在 rebase 阶段（合并之前）就已暴露——某种程度上是提前检查了兼容性，确认没问题后再 merge 会更有把握

> **讲师实用建议**（课堂原话）：
> "如果不确定，我宁愿多加一个 merge commit，也不去动 commit history。rebase 的风险是万一改错了，历史就乱了；而 3-way merge 所有变更都可见、都可追溯，更安全。大型项目我依然用这个策略。"

---

### 2.4 Squash & Merge

把 feature 上所有提交压缩成**一个**普通 commit，再合入 main。

```bash
git merge --squash feature-branch
```

```
feature: C---D---E（多个小提交）
↓ squash
main: ...---S（一个整合提交）
```

- 适用：feature 分支上有大量 WIP 小提交，不想污染 main 历史
- 结果：main 历史简洁，但丢失了 feature 内部的提交粒度

---

### 对比总结

| 策略 | 历史形状 | Merge Commit | 适用场景 |
|---|---|---|---|
| Fast-forward | 直线 | 无 | 简单、无分叉 |
| 3-way merge | 分叉图 | 有 | 最通用，协作首选 |
| Rebase | 直线 | 无 | 本地清理历史，**禁用于公共分支** |
| Squash | 直线 | 无 | 压缩 WIP 小提交 |

---

## 3. Handy Git Features

这三个工具处理的都是**本地临时状态管理**，不涉及远端推送。

---

### 3.1 `git stash` — 临时搁置未提交的改动

**场景**：在 feature 分支上改到一半，需要紧急切换到其他分支，但当前改动还不值得 commit。

`git stash` 把未提交改动打包压入临时储藏室，让工作区变干净，可以自由切换分支。

```bash
git stash              # 打包储藏，工作区恢复干净
git checkout main      # 安全切换
# ...处理完毕，切回来...
git checkout feature/xxx
git stash pop          # 取回改动，继续工作
```

stash 是个**栈**（LIFO），可以多次储藏：

```bash
git stash list                  # 查看所有 stash
git stash apply stash@{1}       # 取回指定条目但不删除
git stash drop stash@{1}        # 手动删除某条
git stash pop                   # 取回最新条目并删除
```

---

### 3.2 `git cherry-pick` — 精准摘取单个 commit

**场景**：feature 分支还没准备好合入 main，但其中某一个 commit 修了紧急 bug，main 现在就需要。

```
main:    A---B---E
              \
feature:       C---D(bugfix)---F---G
```

只把 D 摘到 main：

```bash
git checkout main
git cherry-pick <D的SHA>
```

结果：

```
main:    A---B---E---D'    ← D 的副本，SHA 不同（父节点变了）
              \
feature:       C---D---F---G   ← feature 不受影响
```

**注意**：cherry-pick 生成新 SHA。若之后整条 feature 合入 main，Git 通常能识别内容相同并跳过，但偶尔会有冲突需手动处理。

---

### 3.3 `git reset` — 撤销本地提交

> **前提**：只用于**还没 push 到远端**的提交。

三种模式：

| 命令 | 提交历史 | 暂存区 | 工作区文件 | 适用场景 |
|---|---|---|---|---|
| `git reset --soft HEAD~N` | 撤销 | 改动保留（已暂存） | 不变 | 想重写 commit message 或合并提交 |
| `git reset --mixed HEAD~N`（默认）| 撤销 | 清空（改动退为未暂存）| 不变 | 想重新 `git add` 再 commit |
| `git reset --hard HEAD~N` | 撤销 | 清空 | **文件还原，改动消失** | 彻底放弃这些提交 |

`HEAD~N` 表示往前退 N 个提交，也可以直接写 SHA。

> ⚠️ `--hard` 是破坏性操作，改动直接丢失（可用 `git reflog` 找 SHA 补救，但有风险）。

---

### 3.4 `git reset` vs `git revert` — 重要区分

| | `git reset` | `git revert` |
|---|---|---|
| 做什么 | 把 HEAD 往回移，重写历史 | 生成新 commit 撤销某次改动 |
| 历史 | 被删除 | 保留，追加一条撤销记录 |
| 适用 | 本地未推送的 commit | 已推送到远端的 commit |
| 安全性 | 对已推送内容**危险** | 安全，不重写历史 |

已经 push 到 GitHub 的提交，用 `git revert <SHA>` 而非 `git reset`。

---

## 4. 实操练习

> *待续——讲到此处时补充*

### Fork vs. Template（课堂补充）

课程用的是 **Use this template**，而非 Fork，两者有本质区别：

| | Fork | Template |
|---|---|---|
| 与原仓库的关系 | 保持连接，追踪上游历史 | **不连接**，不追踪历史 |
| 适用场景 | 想向原项目贡献代码（PR） | 只是借用代码结构，独立开发 |
| upstream 概念 | 原仓库 = upstream，fork = origin | 无 upstream 关系 |

用 template 的原因：课程要在自己的仓库里随意改 commit history，不影响原仓库，也不需要向原仓库提 PR。

**Upstream 关系链**（fork 场景下）：
```
原仓库（upstream）
    ↓ fork
你的 GitHub 仓库（origin/remote）
    ↓ clone
本地机器（local）
```

### 准备工作

1. 前往 <https://github.com/UNIVERSE-HPC/git-example>
2. 点击 **Use this template** → **Create a new repository**
3. Owner 选自己的账号，命名为 `git-example`，设为 Public
4. Clone 到本地并用 VS Code 打开：

```bash
git clone https://github.com/YOUR_USERNAME/git-example
cd git-example
code .     # 用 VS Code 打开整个项目文件夹
```

左侧文件树点击 `climate_analysis.py` 即可查看。

### 示例代码问题（climate_analysis.py）

课程使用一个故意写坏的气候分析脚本，存在以下问题：
- 命名不规范（大写函数名而非 snake_case，如 `FahrToCels`）
- 缺少 docstring
- 硬编码路径
- 函数名拼写错误（`FahrToCels` 应为 `FahrToCelsius`），导致运行时报错

### 需要创建的 Issues（按顺序）

1. `Functions should use snake case naming style`
2. `Add missing docstrings to function and module`
3. `Script fails with undefined function name error`

每个 issue 建完后在右侧 Assignees 把自己加上。

---

### Feature Branch Workflow 完整流程（以 Issue #1 为例）

#### 1. 创建 feature 分支

```bash
git checkout -b issue/1-snake-case-naming
git branch   # 确认前面有 * 的是新分支
```

命名规范：`issue/编号-简短描述`

```bash
git branch      # 只显示本地分支，* 标记当前所在分支
git branch -a   # 显示所有分支，包括远端（remotes/origin/...）
git remote -v   # 查看当前仓库连接的远端地址（fetch 和 push 各一条）
```

#### 2. 改代码

在 VS Code 里修改 `climate_analysis.py`，**只改当前 issue 对应的内容**，其他 issue 的问题留给各自的分支。

Issue #1 的改动（snake_case 命名）：

| 原来 | 改成 |
|---|---|
| `def FahrToCelsius(fahr):` | `def fahr_to_celsius(fahr):` |
| `FahrToCelsius(fahr) + 273.15` | `fahr_to_celsius(fahr) + 273.15` |
| `def FahrToKelvin(fahr):` | `def fahr_to_kelvin(fahr):` |
| `kelvin = FahrToKelvin(fahr)` | `kelvin = fahr_to_kelvin(fahr)` |

> 注意：循环里的 `FahrToCels(fahr)` 是 Issue #3 的 bug，此处**不动**。

改完后 **Ctrl+S 保存**，用 `git status` 确认文件出现在 modified 列表。

#### 3. Commit 并 Push

```bash
git add climate_analysis.py
git commit -m "Rename functions to use snake_case naming style, closes #1"
git push origin issue/1-snake-case-naming
```

commit message 里写 `closes #1`，PR 合并时会自动关闭对应 issue。

#### 4. 创建 Pull Request

GitHub 会弹出 "Compare & pull request" 横幅，点击进入，填写：
- **Title**：与 issue 标题对应
- **Description**：`Closes #1`

点 **Create pull request**。
