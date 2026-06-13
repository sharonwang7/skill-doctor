# 建包 / 拆包指南（优化模式的形态分支）

用户只需说"优化"。本指南规定：怎么**探测输入形态**、光杆 SKILL.md 怎么**拆成完整目录**、写盘怎么**非破坏性、可撤回**。内部拆法不必让用户操心，用户只确认结果。

---

## 一、形态探测（决定走哪条分支）

```
目标路径
  ├─ 是单个 .md 文件，或目录里只有一个 SKILL.md（无 scripts/ references/ assets/）
  │     → 光杆形态 → 「建包 + 优化」
  └─ 是目录且已有 scripts/ references/ assets/ 之一
        → 完整形态 → 「只优化，不重复建包」
```

跨平台探测（替代裸 bash，避免平台扣分）：

```powershell
# PowerShell
if (Test-Path $skillPath -PathType Container) {
  $subs = @('scripts','references','assets','examples') | Where-Object { Test-Path (Join-Path $skillPath $_) }
  if ($subs) { '完整形态' } else { '光杆形态（目录里只有 SKILL.md）' }
} elseif (Test-Path $skillPath -PathType Leaf) { '光杆形态（单文件）' }
```

```bash
# bash
if [ -d "$skill_path" ]; then
  if ls "$skill_path"/{scripts,references,assets,examples} >/dev/null 2>&1; then echo "完整形态"; else echo "光杆形态"; fi
elif [ -f "$skill_path" ]; then echo "光杆形态（单文件）"; fi
```

---

## 二、光杆 → 完整包：拆分规则

光杆 SKILL.md 的典型病：全部内联（违反正文 ≤40 行）、长规则表挤在正文、模板内容直接贴在步骤里、靠 AI 手做本该确定性的产物。按渐进式披露三层重新归位：

| 原内容（在光杆正文里） | 拆到哪里 | 引用方式 |
|---|---|---|
| 详细评分标准 / checklist / 长列表 / 风格指南 | `references/<topic>.md` | 正文写"读取并遵循 [references/xxx.md]" |
| 输出模板（报告、卡片、代码骨架） | `assets/<name>.{json,md,…}` | 正文写"按 [assets/xxx] 模板填充以下字段" |
| 需要确定性的生成/解析/计算逻辑 | `scripts/<name>.<ext>`（带 shebang + 入参） | 正文写"执行此脚本：`python scripts/xxx.py <args>`" |
| 示例输入/输出 | `examples/` | 正文写"参考 [examples/xxx]" |
| 核心流程、触发条件、完成标准 | 留在 `SKILL.md` 正文 | —— |

拆分时同步修复（对应各维度）：
- 把裸"使用X"补成带执行者的引用（理解·引用执行语义）。
- 给确定性环节生成 `scripts/` 脚本或严格模板（执行·确定性保障）——仅对"重跑会变且有害"的环节，judgment 类不加。
- 在正文/新建 `references/setup.md` 里补运行环境与所需工具声明（经济·可复用性）。
- 确保新建的每个引用都真实落盘、无悬空（执行·完整性与一致性）。

---

## 三、精简 SKILL.md 模板（拆完后的正文骨架，目标 ≤40 行）

```markdown
---
name: <hyphen-case-name>
description: |
  <一句功能定义>。当用户说 "<短语1>"、"<短语2>" 或 "<短语3>" 时触发。
version: <x.y.z>
---

# <Skill 名>

<一句话目标/产出，让 agent 知道终点。>

## 以下情况不使用
- <排除场景>

## 输入
- <参数>：<说明>

## 步骤
1. <步骤，单一目标>
2. 详细规则见，读取并遵循 [references/xxx.md]
3. 需确定性输出时，执行此脚本：`<cmd> scripts/xxx`
4. 按 [assets/xxx] 模板填充以下字段后输出：<字段>

## 完成前验证
- [ ] <自检项>
- [ ] <自检项>
```

---

## 四、写盘：非破坏性、可撤回

**展示 → 用户确认 → 才写盘。** 写盘前给两种方式，**默认非破坏性**：

```
准备写入。请选择：
  [1] 新建独立产物（默认，可撤回）
        · 建包：在 <父目录>/<skill-name>/ 下新建完整目录
        · 只优化：输出副本 SKILL.optimized.md（及 references 等同名 .optimized）
  [2] 覆盖原文件
        · 覆盖前自动对每个被覆盖文件生成 .bak 备份
```

- 选 [1]：原文件/原目录**完全不动**，用户可对比后再决定是否替换。
- 选 [2]：每个将被覆盖的文件先复制为 `<file>.bak`，再写入新内容；在结果里列出所有 `.bak` 路径，告知"如需还原，删新文件、去掉 .bak 后缀即可"。
- **任何情况下不删除、不移动**用户原有文件（孤儿文件只提示，不擅自删）。

---

## 五、只优化分支（完整形态）

已是完整目录时，**不重复建包**：只就地修复审查发现的问题文件（哪个文件有缺陷改哪个），保持原有目录结构，不新建多余层级。写盘同样走第四节的"副本（默认）/ 覆盖（带 .bak）"二选一。
