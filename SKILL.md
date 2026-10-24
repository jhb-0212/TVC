---
name: agent-pipeline
description: 通用的「生成-过审」多 Agent 协同流水线骨架。当用户需要一个内容生成类任务（写脚本/写剧本/写简历/做分镜/写文案等）被拆成多个 agent 协同完成——调度判定、生成、独立过审、汇总交付——时使用。覆盖「输入→分派→生成→过审→输出」五段，通过共享产物契约 + 统一 header 实现 agent 间靠文件交接而非记忆。业务规则由接入实例自带，本骨架只提供协同层。接入新业务见 references/adapter-guide.md。
---

# 生成-过审通用流水线（agent-pipeline）

## 概述

一套**与业务完全解耦**的多 Agent 协同骨架。任何「输入 → 生成 → 过审 → 输出」的内容类任务都能套用，区别只在接入实例的业务规则不同。

核心价值：
- **把协同层抽成骨架**：调度、产物契约、过审、交接这些共性，不用每个 skill 重搭。
- **生成与过审分离**：生成 Agent 不审自己（防止放水），过审独立、只标不改。
- **靠文件交接而非记忆**：agent 之间通过 `workspace/` 产物 + 统一 header 交接，稳定、可回查、可并行。
- **继承长剧 pipeline 的经济性设计**：模型分层 + 上下文隔离 + 机械检查优先（详见 `references/economy.md`）。

## 何时使用

- 用户要做一个内容生成任务，并希望用多 Agent 协同（而非单 agent 一把梭）
- 需要「生成 + 独立过审」的质量闭环
- 多个同类 skill（TVC、简历、分镜、文案…）想复用同一套协同骨架

## 五段流程

```
输入 brief  →  分派 dispatch  →  生成 chain  →  过审 review  →  输出 final
```

| 段 | 谁做 | 产物 |
|---|---|---|
| 输入 | 调度 Agent | workspace/brief.md |
| 分派 | 调度 Agent | brief.md 里定 chain |
| 生成 | 生成 Agent（业务实例） | workspace/one-pager.md → script.md |
| 过审 | 过审 Agent | workspace/review.md |
| 输出 | 调度 Agent | workspace/final.md |

## 四类角色与身份边界

角色定义、谁写哪个文件、越权红线 → 读 `references/roles.md`。**核心纪律**：

1. 调度 Agent 只判定/派单/汇总，不生成不过审。
2. 生成 Agent 只生成，不过审自己。
3. 过审 Agent 只标不改，发现问题写 review.md，回改由生成 Agent 执行。
4. 任何 agent 只写自己边界内的文件，读到 `status: done` 的文件不得再改。

## 产物契约

workspace 目录结构、统一 header、交接流 → 读 `references/contract.md`。**核心纪律**：

- 每个产物文件顶部必有 header：`task_id` / `chain` / `producer` / `consumer` / `status`。
- `task_id` 一次任务全程不变，跨文件关联。
- `status` 流转：`draft` → `review` → `done`。

## 执行原则（贯穿全流程，继承长剧经验）

1. **上下文隔离**：每个 agent 只喂最小必要上下文，全文不重复加载。
2. **模型分层**：便宜模型干粗活（初筛、粗写、多视角），贵模型干细活（精修、冷审）。
3. **机械检查优先**：禁词、查重、数值一致性等「检索/对比」类检查，先用 Grep 类机械手段跑，清掉低级错误后再进模型视角。这一条能拦下 60% 以上返工。
4. **校验前置**：容易返工的点提前校验，宁前慢不后返。
5. **过审独立**：生成 Agent 绝不审自己的产出，过审是独立角色。

经济性映射表、上下文隔离清单 → 读 `references/economy.md`。

## 调度语义

信号词分派、第一目标判定、派单流程、收产物汇总 → 读 `references/dispatch.md`。

## 过审语义

两级制（创作自检 + 逐镜/逐条扫描）、review 产物格式、越权红线 → 读 `references/review.md`。

## 接入新业务

如何把一个新业务（如简历、分镜、文案）套进本骨架，实例目录怎么组织 → 读 `references/adapter-guide.md`。

## 资源引用

- `references/roles.md` —— 四类角色定义 + 身份边界 + 越权红线
- `references/contract.md` —— 产物契约：workspace 结构 + 统一 header + 交接流
- `references/dispatch.md` —— 调度语义：信号词分派 + 派单流程 + 汇总
- `references/review.md` —— 过审语义：两级制 + review 产物格式 + 越权红线
- `references/economy.md` —— 模型分层 + 上下文隔离 + 机械检查优先
- `references/adapter-guide.md` —— 新业务接入指南（实例模板）
