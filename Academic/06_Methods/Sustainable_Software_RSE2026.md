---
title: Building Better Software for Open and Reproducible Research — DH-RSE Summer School 2026
date: 2026-07-01
tags: [RSE, python, reproducibility, software-engineering, methods, FAIR]
source: "DH-RSE Summer School 2026 / Aleksandra Nenadic & Aman Goel (SSI)"
materials:
  - "D:/Kings/RSE summer school/Day 3/applying-good-research-software-practices-intro.md"
  - "D:/Kings/RSE summer school/Day 3/applying-good-research-software-practices-exercises.md"
  - "https://github.com/softwaresaved/DH-RSE-Summer-School-2026-Day3-code"
  - "https://stackoverflow.com/questions/58754860/cmd-opens-windows-store-when-i-type-python"
---

# Building Better Software for Open and Reproducible Research

## 课程结构

1. 背景与动机——可重复性危机
2. FAIR 软件原则
3. 获取和检查代码
4. 可重复软件环境（Virtual Environments）
5. 代码格式与可读性
6. 软件文档
7. 发布软件

---

## 1. 背景与动机：可重复性危机

2016 年《Nature》调查，1,500 位科学家中：
- **70%** 曾尝试重复他人实验但失败
- **52%** 连重复自己之前的研究都失败了

软件是重要原因之一：代码未公开、依赖包版本不同、参数硬编码、隐藏 bug。

**今天所有内容服务于同一目标**：让别人（或六个月后的你自己）能理解、运行、验证你的代码。

### 今日场景

继承了一位已离职博士后的代码：[NASA EVA 数据分析项目](https://github.com/softwaresaved/DH-RSE-Summer-School-2026-Day3-code)

数据：NASA 舱外活动（EVA / Spacewalk）记录，1965–2013。

代码做的事：
1. 读取 `eva_data.json`
2. 清洗并转换为 CSV
3. 计算每位宇航员累积舱外时间
4. 画累积时间折线图

这份代码**故意写坏**——任务是把它修好并发布。

---

## 2. FAIR 软件原则

FAIR 最早针对科学数据（2016），现已延伸到研究软件。

| 字母 | 含义 | 具体要求 |
|---|---|---|
| **F** Findable | 可被发现 | 有持久 DOI、有元数据、有关键词、在公开平台托管 |
| **A** Accessible | 可被获取 | 通过标准协议免费获取；元数据即使软件下线也应保留 |
| **I** Interoperable | 可互操作 | 使用标准数据格式（CSV/JSON），遵循标准 API 惯例 |
| **R** Reusable | 可被复用 | 有 License、有引用信息、有文档、有版本号 |

今天的练习几乎每一步都在服务 **R**，同时覆盖 **F** 和 **A**。

> 参考论文：[FAIR Research Software Principles, *Scientific Data* 2022](https://www.nature.com/articles/s41597-022-01710-x)

---

## 3. 获取和检查代码（Section A）

### A.1 创建副本

- 去 GitHub 点 **"Use this template"**（不是 Fork）
- **Template vs Fork 的区别**：
  - Fork 保留与原仓库的关联，适合"要向原项目提 PR"
  - Use template 创建干净副本，无原有 commit 历史，适合"以此为起点做自己的项目"

### A.2 Clone 到本地

```bash
git clone https://github.com/你的用户名/DH-RSE-Summer-School-2026-Day3-code
cd DH-RSE-Summer-School-2026-Day3-code
code .
```

先**只读**完整过一遍 `eva_data_analysis.py`，带着问题读：
- 有没有函数？
- 变量名能直接看懂意思吗？
- import 在哪里？
- 注释有没有帮助？

---

## 4. 可重复软件环境（Section B）

### 核心问题

你的机器 pandas 1.5，同事装了 pandas 2.0，API 变了，代码报错——不是代码的问题，是**环境不一样**。

**解法**：每个项目一个独立的虚拟环境（沙箱），不同项目互不干扰。

```
你的机器
├── 系统 Python（不要碰它）
├── 项目A/.venv/   ← pandas 1.5
├── 项目B/.venv/   ← pandas 2.0
└── 论文/.venv/    ← matplotlib 3.7, pandas 2.1
```

### B.1 创建虚拟环境（`venv`）

`venv` 是 Python 3.3+ 自带标准库，无需额外安装。

```bash
# 在项目根目录：
python -m venv .venv
```

**激活：**

```bash
# macOS / Linux：
source .venv/bin/activate

# Windows PowerShell：
.venv\Scripts\Activate.ps1

# Windows CMD：
.venv\Scripts\activate.bat
```

激活后提示符变为 `(.venv) $`，此时的 `python` 和 `pip` 都是沙箱内的版本。

**退出沙箱：**

```bash
deactivate
```

---

### ⚠️ Windows 专项：`python` 打开了 Microsoft Store

**根本原因**：Windows 10/11 预装了"App Execution Aliases"——`python.exe` 和 `python3.exe` 只是重定向到 Microsoft Store 的假快捷方式，不是真正的解释器。

参考：[StackOverflow #58754860](https://stackoverflow.com/questions/58754860/cmd-opens-windows-store-when-i-type-python)

**诊断：**

```powershell
where.exe python
# 如果输出含 WindowsApps，就中招了：
# C:\Users\...\AppData\Local\Microsoft\WindowsApps\python.exe
```

**解决方案（三选一）：**

**方案一（推荐）：关闭 App Execution Aliases**

1. `设置 → 应用 → 高级应用设置 → 应用执行别名`
2. 关掉 `python.exe` 和 `python3.exe`
3. 确保 Python 已从 python.org 安装，且安装时勾选了 **"Add Python to PATH"**

**方案二：用 `py` 启动器（Windows 专属）**

```powershell
py --version
py -3 -m venv .venv
```

`py` 不会被 App Execution Aliases 干扰。

**方案三：用完整路径**

```powershell
C:\Users\你的用户名\AppData\Local\Programs\Python\Python312\python.exe -m venv .venv
```

**PowerShell 执行策略报错**：

```
无法加载文件 ... Activate.ps1，因为在此系统上禁止运行脚本。
```

```powershell
# 临时解决（仅当前会话）：
Set-ExecutionPolicy -ExecutionPolicy RemoteSigned -Scope Process
.venv\Scripts\Activate.ps1
```

---

### B.2 安装依赖（`pip`）

```bash
# 安装项目依赖：
pip install pandas matplotlib

# 查看已安装包（含间接依赖）：
pip list
```

### B.3 创建 `requirements.txt`

```bash
pip freeze > requirements.txt
```

文件内容示例：

```
matplotlib==3.10.0
numpy==2.2.0
pandas==2.2.3
...
```

`==` 是精确版本锁定，确保别人装到和你完全一样的版本。

**别人重建环境只需：**

```bash
python -m venv .venv
source .venv/bin/activate
pip install -r requirements.txt
```

**注意**：`.venv/` 不应提交到 Git，在 `.gitignore` 里加：

```gitignore
.venv/
venv/
__pycache__/
*.pyc
```

**提交 requirements.txt：**

```bash
git add requirements.txt
git commit -m "Add requirements.txt to record project dependencies"
git push origin main
```

---

## 5. 代码格式与可读性（Section 1）

> 代码有两个读者：**机器**（执行）和**人**（理解）。可读性和正确性同等重要。

### 1.1 Import 语句放顶部

原代码问题：import 散落在第 1 行、第 45 行、第 78 行。

**规范：所有 import 放文件最顶部，按组分隔：**

```python
# 标准库
import json
import re

# 第三方库
import matplotlib.pyplot as plt
import pandas as pd
```

PEP 8 分组顺序：标准库 → 第三方 → 本地模块，每组间加空行。

### 1.2 PEP 8 代码风格

Python 官方风格规范：[peps.python.org/pep-0008](https://peps.python.org/pep-0008/)

**关键规则：**

```python
# ✓ 运算符两边加空格：
x = 1 + 2
duration_hours = hours + minutes / 60

# ✓ 逗号后加空格：
plt.plot(x, y, color='blue')

# ✓ 逻辑块间加空行（有节奏感）：
eva_df = read_eva_data('eva_data.json')

summary_df = summarise_by_astronaut(eva_df)

plot_cumulative_eva_time(summary_df, 'graph.png')

# ✓ 行长度不超过 79 字符，过长则换行：
result = some_long_function(
    argument_one,
    argument_two,
    argument_three,
)
```

**工具：**

```bash
pip install flake8 black

# 检查哪里不规范（只报告）：
flake8 eva_data_analysis.py

# 自动格式化（直接修改文件）：
black eva_data_analysis.py
```

`black` 是"opinionated formatter"——不给你商量余地，团队协作时反而是优点（代码风格完全一致）。

### 1.3 变量命名

原代码：`f`, `d`, `o`, `g`, `hrs`, `hrs2`, `h`, `m`, `val`——读者必须往上翻找定义才能理解。

**修复：**

| 原名 | 改名 | 原因 |
|---|---|---|
| `f` | `input_file` | 明确是输入文件 |
| `d` | `eva_df` | 明确是 EVA 数据的 DataFrame |
| `o` | `output_csv_path` | 明确是 CSV 输出路径 |
| `g` | `graph_output_path` | 明确是图片路径 |
| `hrs` | `duration_hours` | 明确是时长（小时） |
| `hrs2` | `cumulative_hours` | 明确是累积时长 |
| `h, m` | `hours, minutes` | 完整拼写 |
| `val` | `duration_in_hours` | 明确含义 |

**命名原则：**
- 变量用**名词**，函数用**动词开头**（`read_data`, `plot_graph`）
- Python 惯例：`snake_case`（下划线），不用 `camelCase`
- 名字长度与作用域成正比：临时循环变量可以短，模块级数据结构要清楚
- 不用不明显的缩写（`dur`, `hrs`, `val` → 不可以）

### 1.4 删除死代码（Dead Code）

**死代码**：定义了但从不被使用的变量或函数。

原代码的两个问题：
1. `calculate_crew_size` 函数：定义了，**从未被调用**
2. `fieldnames` 变量：定义了，**从未被使用**

处理方式：**做一个决定**——要么删掉，要么真正使用它。不能让它挂着。

> "以防万一"是版本控制要做的事，不是代码要做的事。真的需要时从 git 历史找回来。

### 1.5 重构成函数（Refactoring）

原代码：几十行**平坦脚本**，从头到尾顺序执行，没有任何函数。

**问题：**
- 无法独立测试某一步
- 任何逻辑想复用都要复制代码
- 报错只知道行号，不知道"在哪个功能里出错"
- 没有章节感，必须线性阅读

**DRY 原则（Don't Repeat Yourself）**：发现重复代码就提取成函数。

**重构步骤一：识别逻辑单元**

| 逻辑单元 | 对应函数 |
|---|---|
| 读取 JSON 数据 | `read_eva_data(filepath)` |
| 写入 CSV | `write_dataframe_to_csv(df, output_path)` |
| 解析时长字符串 | `parse_duration(duration_str)` |
| 按宇航员汇总 | `summarise_by_astronaut(eva_df)` |
| 画累积时间图 | `plot_cumulative_eva_time(df, output_path)` |

**重构后的示例：**

```python
def parse_duration(duration_str):
    """把 'H:MM' 格式转成十进制小时数。"""
    hours, minutes = duration_str.split(':')
    return float(hours) + float(minutes) / 60


def read_eva_data(input_file):
    """从 JSON 文件读取 EVA 数据，返回清洗后的 DataFrame。"""
    with open(input_file, 'r') as f:
        data = json.load(f)
    eva_df = pd.DataFrame(data)
    eva_df['duration'] = eva_df['duration'].apply(parse_duration)
    return eva_df


def write_dataframe_to_csv(df, output_path):
    """把 DataFrame 写入 CSV。"""
    df.to_csv(output_path, index=False)
    print(f'Saved to {output_path}')


def summarise_by_astronaut(eva_df):
    """按宇航员汇总累积舱外时间。"""
    summary = (
        eva_df.groupby('crew')['duration']
        .sum()
        .sort_values(ascending=False)
        .reset_index()
    )
    summary.columns = ['crew', 'total_hours']
    return summary


def plot_cumulative_eva_time(eva_df, output_path):
    """画累积舱外时间折线图并保存。"""
    eva_df_sorted = eva_df.sort_values('date')
    eva_df_sorted['cumulative_hours'] = eva_df_sorted['duration'].cumsum()
    fig, ax = plt.subplots(figsize=(10, 6))
    ax.plot(eva_df_sorted['date'], eva_df_sorted['cumulative_hours'])
    ax.set_xlabel('Date')
    ax.set_ylabel('Cumulative EVA Duration (hours)')
    ax.set_title('Cumulative Time Spent in Space on EVA Missions')
    plt.savefig(output_path)
    print(f'Graph saved to {output_path}')
```

**DRY 示例**——把重复的 CSV 写入逻辑提取成一个函数：

```python
# 重复两次（坏）：
eva_df.to_csv('eva_data.csv', index=False)
print('Saved eva_data.csv')

summary_df.to_csv('duration_by_astronaut.csv', index=False)
print('Saved duration_by_astronaut.csv')

# 提取成函数（好）：
def write_dataframe_to_csv(df, output_path):
    df.to_csv(output_path, index=False)
    print(f'Saved {output_path}')

write_dataframe_to_csv(eva_df, 'eva_data.csv')
write_dataframe_to_csv(summary_df, 'duration_by_astronaut.csv')
```

### 1.6 `main()` 函数 与 `if __name__ == "__main__":`

```python
def main():
    eva_df = read_eva_data('eva_data.json')
    write_dataframe_to_csv(eva_df, 'eva_data.csv')
    summary_df = summarise_by_astronaut(eva_df)
    write_dataframe_to_csv(summary_df, 'duration_by_astronaut.csv')
    plot_cumulative_eva_time(eva_df, 'cumulative_eva_graph.png')


if __name__ == "__main__":
    main()
```

**`if __name__ == "__main__":` 的意义：**

| 运行方式 | `__name__` 的值 | `main()` 是否执行 |
|---|---|---|
| `python eva_data_analysis.py`（直接运行） | `"__main__"` | ✓ 是 |
| `import eva_data_analysis`（被别人导入） | `"eva_data_analysis"` | ✗ 否 |

这让脚本既能直接运行，又能被作为模块导入（别人只用 `parse_duration` 时不会触发整个脚本）。

### 1.7 可选：命令行参数（argparse）

```python
import argparse

def parse_args():
    parser = argparse.ArgumentParser(
        description='Analyse NASA EVA data and generate summary statistics.'
    )
    parser.add_argument('--input', default='eva_data.json',
                        help='Path to the input JSON file')
    parser.add_argument('--output-csv', default='eva_data.csv')
    parser.add_argument('--output-graph', default='cumulative_eva_graph.png')
    return parser.parse_args()
```

```bash
# 用默认路径：
python eva_data_analysis.py

# 自定义路径：
python eva_data_analysis.py --input my_data.json --output-graph results/graph.png

# 查看帮助：
python eva_data_analysis.py --help
```

**完整文件结构（重构后）：**

```
[文件顶部 docstring]
[所有 import]
[函数定义区：parse_duration / read_eva_data / write_... / summarise_... / plot_...]
[main() 函数]
[if __name__ == "__main__": main()]
```

---

## 6. 软件文档（Section 2）

### 2.1 注释——解释"为什么"，不是"是什么"

```python
# ✗ 没用（描述显而易见的事）：
# 把 hours 和 minutes 转成十进制小时
duration_hours = float(hours) + float(minutes) / 60

# ✓ 有用（揭示非显而易见的约定）：
# NASA 原始格式为 'H:MM'（非标准 datetime），需手动解析才能做数值累加
duration_hours = float(hours) + float(minutes) / 60
```

**什么时候加注释：**
- 有非显而易见的算法决策
- 有外部数据格式约束
- 有 workaround（并注明原因）
- 代码看起来"错的"但实际有意为之

### 2.2 Docstrings——函数的说明书

Python 内置 `help()` 函数会读 docstring 并打印给用户。

**最小版本：**

```python
def parse_duration(duration_str):
    """把 'H:MM' 格式的时长字符串转成十进制小时数。"""
    ...
```

**完整版本（NumPy 风格，学术/科研常用）：**

```python
def parse_duration(duration_str):
    """
    把 NASA EVA 数据中的时长字符串转换为十进制小时数。

    Parameters
    ----------
    duration_str : str
        格式为 'H:MM' 的时长字符串，例如 '2:30' 表示 2 小时 30 分钟。

    Returns
    -------
    float
        对应的十进制小时数，例如 2.5。

    Raises
    ------
    ValueError
        如果字符串格式不是 'H:MM'（缺少冒号）会抛出此异常。

    Examples
    --------
    >>> parse_duration('2:30')
    2.5
    >>> parse_duration('0:45')
    0.75
    """
    hours, minutes = duration_str.split(':')
    return float(hours) + float(minutes) / 60
```

**文件顶部也要有 docstring：**

```python
"""
EVA Data Analysis
=================

分析 NASA 舱外活动（EVA）数据，生成统计摘要和可视化图表。

用法：
    python eva_data_analysis.py [--input FILE] [--output-csv FILE] [--output-graph FILE]
"""
```

### 2.3 README 文件

README 是仓库的"门面"，GitHub 首页自动渲染。

**结构模板：**

```markdown
# EVA Data Analysis

一句话描述项目是做什么的。

## 功能
## 依赖与安装
## 运行方式（给出具体命令 + 预期输出）
## 贡献指南
## 许可证
## 引用方式
```

参考模板：[ha0ye.github.io/CW21-README-tips/template_README.html](https://ha0ye.github.io/CW21-README-tips/template_README.html)

---

## 7. 发布软件（Section 3）

### 3.1 DOI——让代码变成可引用的研究输出

**为什么需要 DOI：**
- 审稿人/读者可通过 DOI 找到代码，不依赖可能失效的 GitHub 链接
- 有正式的引用条目，就像引用论文
- Zenodo 永久存档，即使 GitHub 账号消失 DOI 依然有效

**工具：Zenodo**（练习用 [Zenodo Sandbox](https://sandbox.zenodo.org/)）

步骤：
1. 登录 Zenodo Sandbox
2. "New Upload" → 上传从 GitHub 下载的 .zip
3. 点 "Reserve DOI"（可以提前知道 DOI，写进 README 和 CITATION.cff）
4. 填写元数据（标题、作者、描述、关键词）
5. "Publish"

> Zenodo 也可直接与 GitHub 集成：每次创建 Release 自动生成 DOI（需真实 Zenodo 账号）。

### 3.2 LICENSE 文件

**没有 LICENSE = 无法合法使用。**

根据版权法，创作作品默认"版权保留"——没有明确授权，任何人都无法合法复用你的代码，即使代码公开在 GitHub 上。

选择工具：[choosealicense.com](https://choosealicense.com/)

| License | 限制 | 适合场景 |
|---|---|---|
| **MIT** | 几乎没有，只需保留版权声明 | 最大化传播，学术代码首选 |
| Apache 2.0 | 保留版权声明 + 专利条款 | 有专利保护需求 |
| GPL-3.0 | 使用此代码的软件也必须开源 | 确保代码永远开源 |

文件：项目根目录的 `LICENSE` 或 `LICENSE.txt`。

**MIT License 开头（含版权声明）：**

```
MIT License

Copyright (c) 2026 Skylar Liu, [博士后姓名]

Permission is hereby granted...
```

> ⚠️ **机构 IP 问题**：如果在工作时间/使用机构资源写的代码，版权可能属于机构（KCL / BFI 各有 IP 政策）。发布前确认。

### 3.3 CITATION.cff——机器可读的引用信息

GitHub 识别此文件，会在仓库右侧显示 **"Cite this repository"** 按钮，自动生成 APA/BibTeX 格式引用。

```yaml
cff-version: 1.2.0
message: "If you use this software, please cite it using the metadata below."
type: software
title: "EVA Data Analysis"
authors:
  - family-names: "Liu"
    given-names: "Shihan"
    orcid: "https://orcid.org/你的ORCID"
    affiliation: "King's College London"
version: "1.0.0"
date-released: "2026-07-01"
doi: "10.5281/zenodo.xxxxxxx"
repository-code: "https://github.com/你的用户名/DH-RSE-Summer-School-2026-Day3-code"
license: MIT
```

> 还没有 ORCID？去 [orcid.org](https://orcid.org/) 注册——KCL 博士生建议都有一个。

### 3.4 GitHub Release

Release 是代码的版本快照——类似论文的"发表版"。你可以继续在 main 上开发，但 `v1.0.0` 永远指向那个时间点。

步骤：
1. 仓库页面右侧 "Releases" → "Create a new release"
2. 创建 tag：`v1.0.0`（语义化版本：主版本.次版本.补丁）
3. 填写 Release notes（这个版本做了什么）
4. "Publish release"

连接 Zenodo 后，此步骤自动触发 DOI 生成。

---

## 全景总结

```
可重复性的三个层次
│
├── 环境层（别人能装起来你的代码）
│   ├── 虚拟环境 .venv
│   └── requirements.txt
│
├── 代码层（别人能读懂并信任你的代码）
│   ├── 格式：PEP 8，imports 在顶部
│   ├── 命名：清晰的变量名和函数名
│   ├── 结构：函数化、main()、DRY
│   └── 无死代码
│
└── 发现与引用层（别人能找到并引用你的代码）
    ├── 文档：注释 + docstrings + README
    ├── 法律：LICENSE + 版权声明
    ├── 引用：CITATION.cff
    └── 标识：DOI (Zenodo) + GitHub Release
```

### 实用优先级排序

| 优先级 | 内容 | 理由 |
|---|---|---|
| 1 | `requirements.txt` + 虚拟环境 | 别人能跑起来是最基本要求 |
| 2 | `README.md` | 告诉别人这是什么、怎么用 |
| 3 | `LICENSE` | 没有这个别人合法上不能用 |
| 4 | 函数化重构 + 清晰命名 | 代码本身可读可信 |
| 5 | Docstrings | 函数级文档 |
| 6 | `CITATION.cff` + DOI | 可引用性（仅在打算公开发布时） |

---

*笔记生成于 DH-RSE Summer School 2026 Day 3，2026-07-01*
