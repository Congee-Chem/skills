<p align="right">
  <a href="README.md"><img src="https://img.shields.io/badge/简体中文-d97757?style=for-the-badge" alt="简体中文"></a>
  <a href="README.en.md"><img src="https://img.shields.io/badge/English-3a3f4b?style=for-the-badge" alt="English"></a>
</p>

# Claude Code 技能库

一组可被 Claude Code 调用的技能，外加一个**双语技能面板** [`index.html`](index.html)：在浏览器里浏览技能介绍、一键生成安装/清除命令。

- 所有技能与阅读版文档都提供**中英文双版**（zh / en）。
- 面板右上角切换语言，决定查看与导入的语言版本。
- 每个技能都能**装到全局或某个项目**，命令为干净覆盖。

> 技能列表与各自用途见文末 [📚 技能列表](#-技能列表)。

---

## 📁 目录结构

```
skills/
├── index.html                  # 技能面板：右上角中英切换、技能卡片、导入/清除（生成命令）
├── README.md                   # 本文件（中文）
├── README.en.md                # 英文说明
├── vibecoding/
│   ├── SKILL.md                # 技能本体·中文（带 frontmatter，供 /vibecoding 调用）
│   └── SKILL.en.md             # 技能本体·英文（同样 name: vibecoding）
└── docs/
    ├── vibecoding流程.md        # 流程纯阅读版·中文（与 SKILL.md 内容一致）
    └── vibecoding-flow.en.md    # 流程纯阅读版·英文
```

- **`vibecoding/SKILL.md` / `SKILL.en.md`** —— 真正被 Claude Code 加载执行的技能文件，两份都用 `name: vibecoding`，调用名都是 `/vibecoding`。**同一时刻只装一份**（见下方安装）。
- **`docs/*`** —— 不参与执行，纯文档，方便人快速通读这套流程。

---

## 🖥️ 用面板安装（推荐）

直接在浏览器打开 [`index.html`](index.html)：

1. 右上角切换 **中文 / EN** —— 决定要装哪个语言版本，也决定所有提示文案的语言。
2. 点技能卡片，看介绍。
3. 在介绍弹窗底部点 **导入全局配置** / **导入项目级目录** / **清除配置**。
4. 弹窗里填目标目录（留空走默认），点「生成命令」，**复制命令到终端执行**。

> 面板是静态页，浏览器不能直接动你的文件系统，所以它**生成命令**让你在终端运行。命令为**干净覆盖**（先删旧目录再拷贝），且只保留当前界面语言对应的 `SKILL.md`。

---

## 🚀 手动安装

Claude Code 从两类位置发现技能：

| 范围 | 路径 | 适用 |
|---|---|---|
| **全局**（所有项目可用） | `~/.claude/skills/<name>/SKILL.md` | 日常推荐 |
| **项目级**（仅该项目可用） | `<项目根>/.claude/skills/<name>/SKILL.md` | 想随仓库分发、团队共享 |

> 安装单位是**整个技能目录**（`vibecoding/`），不是单个文件。
> 下面命令里的 `<本仓库路径>` 替换成你 clone 本仓库的实际位置。**懒人方案**：用 [`index.html`](index.html) 面板，它会自动识别本机路径并生成可直接执行的命令。

### macOS / Linux

**中文版**（默认 `SKILL.md` 即中文，删掉多余的英文文件即可）：

```bash
# 全局安装（干净覆盖）
mkdir -p ~/.claude/skills && \
  rm -rf ~/.claude/skills/vibecoding && \
  cp -R <本仓库路径>/vibecoding ~/.claude/skills/ && \
  rm -f ~/.claude/skills/vibecoding/SKILL.en.md
```

**英文版**（用 `SKILL.en.md` 覆盖 `SKILL.md`）：

```bash
mkdir -p ~/.claude/skills && \
  rm -rf ~/.claude/skills/vibecoding && \
  cp -R <本仓库路径>/vibecoding ~/.claude/skills/ && \
  mv -f ~/.claude/skills/vibecoding/SKILL.en.md ~/.claude/skills/vibecoding/SKILL.md
```

> 项目级安装：把上面的 `~/.claude/skills` 换成 `<项目根>/.claude/skills`（先 `mkdir -p`）。

### Windows · PowerShell

```powershell
# 全局安装（中文版）
$dst = "$env:USERPROFILE\.claude\skills\vibecoding"
Remove-Item -Recurse -Force $dst -ErrorAction SilentlyContinue
New-Item -ItemType Directory -Force "$env:USERPROFILE\.claude\skills" | Out-Null
Copy-Item -Recurse -Force "C:\path\to\skills\vibecoding" "$env:USERPROFILE\.claude\skills\"
Remove-Item -Force "$dst\SKILL.en.md" -ErrorAction SilentlyContinue
# 英文版：把上面最后一行换成
# Move-Item -Force "$dst\SKILL.en.md" "$dst\SKILL.md"
```

> 想随源码联动而不复制，可用软链：macOS/Linux `ln -s <源>/vibecoding ~/.claude/skills/vibecoding`；
> Windows（管理员）`mklink /D "%USERPROFILE%\.claude\skills\vibecoding" "C:\path\to\skills\vibecoding"`（软链方式两个语言文件都会在，Claude 只认 `SKILL.md`）。

安装后**重启 / 新开一个 Claude Code 会话**，技能列表才会重新加载。

### 清除 / 卸载

```bash
rm -rf ~/.claude/skills/vibecoding          # 全局
rm -rf <项目根>/.claude/skills/vibecoding    # 项目级
```

### 验证已安装

```bash
ls ~/.claude/skills/vibecoding/SKILL.md     # macOS/Linux
dir "%USERPROFILE%\.claude\skills\vibecoding\SKILL.md"   # Windows CMD
```

在会话中输入 `/` 应能看到 `vibecoding` 出现在技能列表。

---

## 📚 技能列表

### 🚀 Vibecoding 全栈开发流程 — `/vibecoding`

人 + AI 结对，用最短可验证步长，把想法落地成**可运行、可部署**的全栈项目。按「需求澄清 → 技术栈选型 → 架构设计 → 脚手架 → 垂直切片增量实现 → 自测验证 → 评审 → 部署交付」推进；调用时自选技术栈、内置搭配合理性检查、分个人 / 团队两条轨道。

```
/vibecoding 我想做一个待办事项应用
```

<p>
  <a href="docs/vibecoding流程.md"><img src="https://img.shields.io/badge/详细介绍-简体中文-d97757?style=for-the-badge" alt="详细介绍 简体中文"></a>
  <a href="docs/vibecoding-flow.en.md"><img src="https://img.shields.io/badge/Full_docs-English-3a3f4b?style=for-the-badge" alt="Full docs English"></a>
</p>

---

## 🔧 自定义
- 改技能行为：编辑 `vibecoding/SKILL.md`（中文）或 `SKILL.en.md`（英文），改完重新按上面的命令同步到已安装位置（或用软链免同步）。
- 改调用名：修改 `SKILL.md` frontmatter 的 `name` 字段，并把目录改成同名。
- 加新技能：在仓库里新建 `<name>/SKILL.md`，并在 [`index.html`](index.html) 的 `SKILLS` 数组里加一条（含中英 `name`/`tagline`/`intro`）即可出现在面板。
