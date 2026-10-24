# TVC 实例 · 链路生成规则（chain 目录，自包含）

## 本目录承载三链路的生成规则（生成 Agent 用）

三链路业务规则已迁移进本目录，实例**自包含**，打包分发不散架。

## 目录结构

```
chain/
├── hard-ad/skill.md          # 硬广生成 Agent（6 步工作流）
├── soft-ad/skill.md          # 软广生成 Agent
├── story-ad/skill.md         # 剧情生成 Agent
└── references/
    ├── common/               # 三链路通用
    │   ├── storyboard-format.md   # 分镜表 10 字段 + 每镜校验 7 条
    │   ├── spec-table.md          # 时长结构 / 镜头数 / 单镜时长 / 语速 / 运镜
    │   ├── prompts.md             # Kling / Veo / Runway 提示词句式
    │   ├── compliance-base.md     # 一级违禁词 + 数据引用原则
    │   └── delivery.md            # 方向确认一页纸 + 配套交付 5 项
    ├── hard/                 # 硬广专属：playbook（6 节）+ benchmark
    ├── soft/                 # 软广专属：playbook（6 节）+ benchmark
    └── story/                # 剧情专属：playbook（7 节）+ benchmark
```

## 基准目录

所有 `references/...` 引用均以 **`chain/` 为基准**。生成 Agent 读文件时，以 `chain/` 为工作目录。

## 生成 Agent 读取规则

- hard-ad 生成 Agent：读 `hard-ad/skill.md` + `references/hard/playbook.md` + `references/hard/benchmark.md` + `references/common/*`
- soft-ad 生成 Agent：读 `soft-ad/skill.md` + `references/soft/playbook.md` + `references/soft/benchmark.md` + `references/common/*`
- story-ad 生成 Agent：读 `story-ad/skill.md` + `references/story/playbook.md` + `references/story/benchmark.md` + `references/common/*`

## 与骨架协同层的衔接

- 调度 Agent：读实例根 `signal-table.md` + `param-fields.md`（不读本目录业务规则全文）。
- 生成 Agent：先读 `workspace/brief.md`（调度 Agent 收齐的参数 + 链路类型），再读本目录业务规则 + 骨架的产物契约/身份边界语义（骨架文件由编排层注入，见骨架 `references/contract.md`、`references/roles.md`）。
- 过审 Agent：读 `workspace/brief.md`（判断合规边界）+ 实例根 `compliance.md` + 骨架过审语义（骨架 `references/review.md`），以及本目录 `references/common/compliance-base.md` 和对应链路的 playbook 合规节。注意合规节节号各链路不同：hard/soft 在 playbook **第 3 节**，story 在 playbook **第 4 节**（价值观红线）。

## 已做的协同适配

三个链路的 skill.md 已从「单 agent 自审」适配为「多 agent 协同」：

- Step 5 从「自审两级制」改为「交给独立过审 Agent（不自审）」，过审读骨架 review.md + 实例 compliance.md。
- Step 6 移除了「过审风险清单」模块（归过审 Agent 出 review.md），🟢 标定改为过审 Agent 出、调度 Agent 汇总时挑。
