# obsidian-knowledge-system

一个 Kimi Code / Claude Code skill：在 Obsidian Vault 中从零搭建"轻量 PARA + 知识网络"的综合个人知识库系统，并让 AI 代理能够安全地全自动维护它。

## 它能做什么

按照 `SKILL.md` 的六步流程，把一个普通 Obsidian 库升级为：

- **目录骨架**：轻量 PARA（收件箱 / 项目 / 领域 / 知识 / 资源 / 日志 / 归档）
- **知识网络**：知识卡片 + 主题索引（MOC）+ 双链
- **首页仪表盘**：基于 Obsidian Bases 的四个视图（待整理 / 进行中 / 最近更新 / 继续阅读）
- **九个核心模板**：快速记录、知识卡片、主题索引、项目主页、阅读笔记、每日/周/月日志、整理日志
- **AI 行为契约**：`整理规则.md` + `AGENTS.md`，定义指令映射、批次日志、Git 快照、反链保护和失败恢复
- **安全边界**：Git 版本管理（只跟踪配置不跟踪个人状态）、迁移先备份、失败可定向恢复、永不永久删除

## 目录结构

```text
├── SKILL.md                        # 技能主文件（六步流程 + 验收清单）
├── references/
│   └── organization-rules.md       # 整理规则完整模板（AI 自动整理的行为契约）
└── assets/
    ├── AGENTS.md                   # Vault 根目录项目指令示例
    ├── 首页.md                      # 首页（嵌入四个 Base 视图）
    ├── bases/首页视图.base          # 四个视图的 Bases 定义
    └── templates/                  # 九个核心模板
```

## 使用方式

把本仓库复制（或软链）到你的 skills 目录：

```bash
# Kimi Code
cp -r obsidian-knowledge-system ~/.kimi-code/skills/

# Claude Code
cp -r obsidian-knowledge-system ~/.claude/skills/
```

然后对 AI 说"帮我搭建 Obsidian 知识库"即可触发。

## 前提条件

- Obsidian（建议 1.15+），启用核心插件 Templates 和 Bases
- `obsidian` CLI 可用且 Obsidian 保持运行
- 本地 Git（无需远程仓库）

## 设计取舍

- **轻量 PARA + 知识网络**：纯 PARA 让知识卡片无处安放，纯双链让新项目没有落点，两者各取一半。
- **整理规则文件 > 常驻服务**：不安装后台守护进程，AI 按需触发整理，规则文件就是契约。
- **验证命令即交付标准**：每一步的"完成"都由 CLI 查询、SHA-256 校验或 Git 状态的新鲜输出背书。
- **三层备份兜底**：迁移前 ZIP 备份、源目录进废纸篓、失败时 ZIP 隔离恢复，任意一层失效即停止。

本 skill 提炼自一次真实落地的知识库搭建项目，全部流程（含反链恢复的成功/失败路径）均在真实 Vault 中验证通过。
