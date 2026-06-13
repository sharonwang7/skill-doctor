---
name: skill-doctor
description: |
  评估并优化整个技能包（SKILL.md + scripts/ + references/ + assets/）的质量。
  当用户说"review this skill"、"检查我的 SKILL.md"、"这个 skill 写得怎么样"、
  "优化/改进/重写我的 skill"、"帮我把这个 skill 拆成完整包"或"给 skill 打分"时触发。
  用户分享 skill 文件或目录并请求反馈时也触发。
version: 2.0.0
---

# Skill Doctor 🩺

评估并优化 Agent Skill。目标：产出一份按 4 个独立维度（按"层"划分）的整包诊断；
当用户要求"优化"时，把技能包修好、并在需要时补全成完整目录。

- **触发精确性** — frontmatter 能否让 agent 在对的时机被选中
- **理解精确性** — 指令层规格是否无歧义且完整
- **执行稳定性** — 资源层（文件/脚本/环境/输出）能否稳定跑出正确结果
- **经济性** — 设计是否耐复用、可移植、易维护

## 以下情况不使用
- 安全审查（检查恶意代码）→ 用 `skill-vetter`
- 仅询问 skill 安装方法

## 输入
- **skill_path**（可选）：技能包目录或 SKILL.md 路径。不提供则检查当前目录。

## 步骤
1. **加载整包**：读取 SKILL.md + `scripts/` + `references/` + `assets/` 全部文件及目录结构。找不到 SKILL.md 则告知用户并请其提供路径或粘贴内容，停止。
2. **模拟推演**：读取并遵循 [references/dry-run.md](references/dry-run.md)，做静态走查 + 白名单实跑，收集执行稳定性证据。
3. **按标准评分**：读取并遵循 [references/rubric.md](references/rubric.md)，4 维逐项打分（先引用原文/原文件，再评判）。
4. **出报告（默认·审查）**：按 [references/report-template.md](references/report-template.md) 输出评分卡 + 问题区 + 优先修复。
5. **优化模式**（仅当用户说"优化/重写/修复/改进"）：先完成步骤 1–4 让用户看到诊断；再读取并遵循 [references/scaffold-guide.md](references/scaffold-guide.md)，自动探测形态（光杆→建包+优化 / 完整目录→只优化），展示结果 → 用户确认 → 非破坏性写盘。

## 完成前验证
- [ ] 已读整包所有文件，不只 SKILL.md
- [ ] 4 维均评分且有原文/原文件引用；执行稳定性发现带 实跑/静态/核对 标签
- [ ] 总分与等级正确（触发×2 + 理解×6 + 执行×7 + 经济×5）
- [ ] 优化模式下：写盘前已展示并征得确认，默认非破坏性（可撤回）
