# 角色定义与身份边界

## 四类角色

| 角色 | 职责 | 写哪些文件 |
|---|---|---|
| 调度 Agent（dispatcher） | 收需求 → 判链路 → 派单 → 收产物 → 汇总交付 | brief.md、final.md |
| 生成 Agent（producer） | 执行业务生成（写脚本/剧本/简历/分镜） | one-pager.md、script.md |
| 过审 Agent（reviewer） | 独立风控，只标不改 | review.md |
| 用户（human） | 唯一人工确认点 | 确认 one-pager、最终接收 final |

## 身份边界（谁写哪个文件，强制）

| 文件 | 可写者 | 说明 |
|---|---|---|
| workspace/brief.md | 调度 Agent | 链路判定 + 收齐的参数 |
| workspace/one-pager.md | 生成 Agent | 方向确认一页纸 |
| workspace/script.md | 生成 Agent | 主体产出（脚本/剧本/简历/分镜） |
| workspace/review.md | 过审 Agent | 过审结果 + 🟢🟡🔴 |
| workspace/final.md | 调度 Agent | 最终交付（合并 script + review 结论） |

## 越权红线

1. 生成 Agent **不能写** brief.md / review.md / final.md。
2. 过审 Agent **不能改** script.md（只读），发现问题写 review.md，回改由生成 Agent 执行。
3. 调度 Agent **不直接写** script.md / review.md。
4. 任何 agent 读到 `status: done` 的文件视为定稿，不得再改；需改动必须重开 task_id。

## 为什么生成与过审必须分离

- 自己审自己会**放水**：生成时投入的立场会延续到过审，漏掉该标 🔴 的风险。
- 过审独立后，标 🟡/🔴 不带心理负担，判断更严格。
- 分离后可以**换模型冷审**：过审用与生成不同的模型，交叉挑错更有效（见 economy.md）。

## 角色与「单 agent 内路由」的区别

- 单 agent 内路由：同一个大脑读不同提示词扮不同角色，产物交接靠「记得」。
- 多 Agent 协同：每个角色是独立 agent，产物交接靠「读 workspace 文件」，不靠记忆。
- 判据：如果一个 agent 能凭记忆拿到上一个环节的全部信息，那就不需要拆分；拆分的收益来自「隔离上下文 + 独立立场 + 可回查」。
