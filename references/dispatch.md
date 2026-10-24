# 调度语义（信号词分派 + 派单流程 + 汇总）

## 调度 Agent 的职责边界

调度 Agent **只判定、派单、汇总**，不生成、不过审、不写主体产出。

## 一、按信号词分派

信号词分派表由**接入实例定义**（见 adapter-guide.md 的「链路信号词表」）。调度 Agent 读实例的信号词表，把需求映射到对应链路。

通用兜底：实例没给信号词表时，按第一目标判定。

## 二、信号不清时，按第一目标判定

- 要转化/直给结果 → 偏「硬」类链路
- 要内容记忆点 → 偏「软」类链路
- 要情感记忆点 → 偏「剧情」类链路

具体判定词由实例定义，调度 Agent 以实例为准。

## 三、仍判不准时追问

用实例给的三选一问法追问，确认后派单。

## 四、派单流程（判定后执行）

1. 生成 `task_id`（`YYYYMMDD-NNN`）。
2. 写 `workspace/brief.md`：header 填 `chain`、`producer`=调度 Agent、`consumer`=对应生成 Agent、`status`=`draft`。
3. brief 正文写清：链路类型 + 收齐的参数（字段清单由实例定义），未收齐的字段标注「待生成 Agent 追问」。
4. 把任务**交给**对应生成 Agent，附 brief.md 路径。

## 五、收产物与汇总

1. 读 `workspace/script.md`（主体产出）+ `workspace/review.md`（过审结论）。
2. **先判 review 结论**：有 🔴 → 不写 final，退回生成 Agent 重改（script.md 的 status 回 `draft`）；有 🟡 → 退回生成 Agent 回改后重交 review。
3. **全 🟢 才写** `workspace/final.md`：合并 script + review 结论，`status`=`done`，交付用户。

## 六、中途变更

- 改类型 / 参数 → 重新判定，新建 task_id，旧产物作废，稳定事实跨 task 保留。
- 用户说「按你建议来」→ 按默认偏好给推荐，标注「我替你选了 X，要改直接说」。
- 跨类需求 → 先定主链路，brief 里标注辅链路元素，主辅比例在 brief 开头声明。

## 七、分派后

调度 Agent 写完 brief.md 即退出，由生成 Agent 接管。生成 Agent 的 Step 1 不再重复判定链路，只判具体意图。
