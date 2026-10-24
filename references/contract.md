# 产物契约（workspace 结构 + 统一 header + 交接流）

## 目的

多 Agent 协同下，agent 之间**不共享记忆**，只通过磁盘上的产物文件交接。本文件定义标准产物目录、统一 header、交接方向。

**核心原则**：靠读文件交接，不靠「记得」。

## workspace 目录结构

所有产物落在实例目录下的 `workspace/`：

```
workspace/
├── brief.md            # 调度 Agent 写入：链路判定 + 收齐的参数
├── one-pager.md        # 生成 Agent 写入：方向确认一页纸
├── script.md           # 生成 Agent 写入：主体产出（脚本/剧本/简历/分镜）
├── review.md           # 过审 Agent 写入：过审结果 + 🟢🟡🔴
└── final.md            # 调度 Agent 写入：最终交付
```

**workspace 是单任务工作区，不是归档区**：每次新任务（新 task_id）会覆盖上述 5 个文件，历史任务产物不会自动累积。需要留存历史交付的，由调度 Agent 在交付 final.md 后，把整套产物复制到 `archive/<task_id>/` 归档（可选动作，默认不做，避免磁盘膨胀）。

## 统一 header（每个产物文件顶部必附）

```markdown
---
task_id: <本次任务唯一 id，如 20261024-001>
chain: <链路标识，由实例定义；示例值仅举例，如 resume / storyboard，TVC 实例为 hard-ad / soft-ad / story-ad>
producer: <写入该文件的 agent 名>
consumer: <下一个读该文件的 agent 名>
status: draft | review | done
---
```

- `task_id`：一次任务全程不变，用于多文件关联与回查。格式 `YYYYMMDD-NNN`（当日递增）。
- `chain`：业务链路标识，由接入实例定义（见 adapter-guide.md）。
- `producer` / `consumer`：明确交接方向，防止 agent 读错文件或越权改写。
- `status` 流转：`draft`（刚写）→ `review`（待过审 / 已过审待汇总）→ `done`（定稿）。

**status 语义澄清（避免与文件名混淆）**：
- `review` 是「状态值」，不是「文件名」——它表示「该产物已完成、正在流转中」，既用于 script.md 的「待过审」，也用于 review.md 自身的「过审完成待汇总」。
- `done` 表示「定稿、不再改动」。**final.md 恒为 `done`**；script.md 与 review.md 在过审通过后由调度 Agent 汇总时统一标 `done`（或保持 `review` 亦可，但一旦标 `done` 即视为定稿，任何 agent 不得再改）。

## 交接流（谁写 → 谁读）

| 文件 | producer | consumer | 触发时机 |
|---|---|---|---|
| brief.md | 调度 Agent | 生成 Agent | 调度判定链路 + 收齐参数后 |
| one-pager.md | 生成 Agent | 用户确认 + 过审 Agent | 生成方向确认后 |
| script.md | 生成 Agent | 过审 Agent | 生成主体产出后 |
| review.md | 过审 Agent | 调度 Agent | 过审完成后 |
| final.md | 调度 Agent | 用户 | 汇总 script + review 结论后 |

## 中途变更

- 需求改类型 / 参数 → 调度 Agent 新建 task_id，旧 workspace 产物作废，可复用的事实（brief 里的稳定参数）跨 task 保留。
- 过审不通过（🔴）→ 生成 Agent 改 script.md，status 回 `draft`，重新走 review。
- 过审 🟡 → 生成 Agent 按建议回改后重交 review。

## 机械检查（写文件前自检）

每个 agent 写完产物后，先用 Grep 类手段自检（禁词、header 齐全、字段完整），再交给下游。低级错误不跨 agent 传播，是省成本关键。
