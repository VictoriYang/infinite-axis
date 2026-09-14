---
title: "Astra 在 BEHAVIOR-1K 里自己改了策略：一次失败，一次成功"
date: 2026-09-14
summary: "把 GPT-6 Astra 接到 OmniGibson / BEHAVIOR-1K 的 R1Pro 上跑一个移动操作任务：抓起地面的饮料罐，放到低茶几上并松手。第一次跑了 3,630 步，罐子抓在手里、导航停在离目标 24 厘米处不收敛，判失败；Astra 自己定位到失败环节、改了两个导航参数、重跑 923 步完成任务。全过程保留两段录像、控制轨迹和独立状态审计。"
tags: ["具身智能", "BEHAVIOR-1K", "OmniGibson", "移动操作", "agent", "自我迭代", "GPT-6 Astra", "实验记录"]
cover:
  image: "lab/astra-behavior-pick-place/evidence/cover.png"
  alt: "成功尝试末帧：罐子已放在茶几上，OnTop 为真"
ShowToc: true
TocOpen: true
---

> 实验日期：2026-09-14 ｜ 仿真环境：OmniGibson / BEHAVIOR-1K（`picking_up_trash` 场景）｜ 机器人：R1Pro
>
> **完整实验页（双视频同步播放、倍速、打印复盘）：[打开 →](demo/)**

## 想验证什么

一个 agent 能把动作发出去，不等于它知道自己有没有把事做成。我想看的是另一件事：**当一次执行明明动作不断、但任务实际没有推进时，模型能不能自己判定失败、定位到是哪个环节、提出改动、再跑一次验证。**

任务本身是标准的移动操作：

> 把地面上的一个饮料罐放到低茶几上，并松手。

成功与否**在执行前就定死了**，判据是物体状态而不是程序是否跑完：

- 起始 `OnTop(can_of_soda_113, coffee_table_koagbh_0)` 为假 → 执行后为真；
- 罐子最终不在任何一个夹爪里（真的松手了，不是提着站在桌边）；
- 执行阶段对机器人和目标罐子的 `set_position_orientation` 调用数必须为 0——也就是不许直接把物体瞬移到桌上蒙混过关。

这个任务是我在 BEHAVIOR 的 `picking_up_trash` 场景里自定义的，不是该场景的官方目标（官方目标是把三个罐子扔进垃圾桶，全程为假，两次审计 JSON 里都能看到）。

## 两次执行

<div style="display:grid;grid-template-columns:1fr 1fr;gap:20px;margin:20px 0">
<figure style="margin:0">
<video controls playsinline preload="metadata" style="width:100%;border-radius:4px;background:#151c1b">
  <source src="evidence/failed_attempt_full.mp4" type="video/mp4">
</video>
<figcaption style="font-size:.85em;opacity:.75;margin-top:6px"><strong>尝试 01 · 失败</strong>｜3,630 控制步 · 92.1 秒 · 最终 OnTop = false。抓取和携物移动都发生了，靠近目标后导航不收敛，人为中止。</figcaption>
</figure>
<figure style="margin:0">
<video controls playsinline preload="metadata" poster="evidence/poster.png" style="width:100%;border-radius:4px;background:#151c1b">
  <source src="evidence/render_motion.mp4" type="video/mp4">
</video>
<figcaption style="font-size:.85em;opacity:.75;margin-top:6px"><strong>尝试 02 · 成功</strong>｜923 控制步 · 27.9 秒 · 最终 OnTop = true，双手为空。</figcaption>
</figure>
</div>

两段都是 8 fps 采样帧回放，不是实时录像，也没有按动作阶段做逐帧对齐。失败那段保留 737 张有效帧（只排除了中断时没写完的末帧 `frame_0737.png`），成功那段 223 帧。

## 它是怎么走完这一圈的

### 01 先把成功条件定死

Astra 先通过 SSH 确认服务器上已有 OmniGibson、R1Pro、CuRobo 运动规划器和空闲 GPU，然后选择「用底盘和机械臂控制器做真实运动规划」这条路，而不是走任何形式的状态直写。任务被拆成抓取、携物移动、放置、松手四段，分别检查进展。

关键在于判据的位置：**完成与否由物体在桌上的状态和夹爪持物状态共同判定，而不是「动作指令已发送」或「程序正常结束」。** 这一条是后面所有判断的支点。

### 02 第一次执行

复用现成的 `StarterSemanticActionPrimitives`，由 CuRobo 规划机械臂轨迹，底盘走 A\* 路径加位置伺服，抓持用仿真的 `sticky` 辅助约束。另外套了一层审计 wrapper，逐步记录控制指令，并定期采样关节、底盘位姿、罐子位置、抓持状态和任务谓词。

执行确实推进了一段：机械臂伸向罐子、闭合夹爪、把罐子提离地面，底盘带着罐子移动。

### 03 失败归因

卡在携物导航阶段。A\* 没找到通往采样目标的路径，原有导航器退回直线尝试；底盘实际移动了约 1.6 米，最后停在距目标约 24 厘米处，控制指令还在持续发送。

当时的到位位置容差是 8 厘米，单次导航预算 6,000 步。跑到 3,630 步人工中止，`OnTop` 仍为假，判失败。审计 JSON 里的末态很直白——罐子还在左手：

```json
"cans": { "can_of_soda_113": { "position": [2.909, 5.246, 0.758],
                               "ontop": { "coffee_table_koagbh_0": false } } },
"held": { "left": "can_of_soda_113", "right": null },
"custom_task_success": false
```

这一步里我觉得最值得记的，是它对「已确认」和「未确认」划了线：

> **已确认：**导航没有收敛。
> **尚未确认：**具体是哪一个碰撞或几何约束导致停滞。没有做接触诊断，因此不能断言「被茶几卡住」。

动作一直在发，不代表任务在推进——这是失败判定的全部理由，不是看录像看出来的观感。

### 04 调整策略

针对导航长期不收敛，只在外层 wrapper 里改了两个参数，没有动原项目：

<table>
<thead><tr><th>参数 / 判定</th><th>第一次</th><th>第二次</th></tr></thead>
<tbody>
<tr><td>底盘到位位置容差</td><td>0.08 m</td><td><strong>0.28 m</strong></td></tr>
<tr><td>单次导航最大步数</td><td>6,000</td><td><strong>700</strong></td></tr>
<tr><td>任务成功条件</td><td colspan="2">不变：OnTop 从假变真，且夹爪已松手</td></tr>
</tbody>
</table>

改动的方向是：**不再让底盘长时间追求精确到达某一个采样点，够近了就交给机械臂规划器去试放置。** 放宽的是导航到位条件，不是成功条件——机械臂仍然得真的完成抓放，最终仍由物体状态判定，没有加入任何直接移动罐子或强制置成功的操作。

这一步的性质是**假设**，不是已证实的因果结论。

### 05 复测通过

第二次 923 步：抓起罐子、调整底盘位置、执行放置、收回机械臂。这一次随机采样出了不同的导航目标，而且 A\* 找到了路径。

```json
"cans": { "can_of_soda_113": { "position": [3.342, 6.010, 0.488],
                               "ontop": { "coffee_table_koagbh_0": true } } },
"held": { "left": null, "right": null },
"pose_writes": [],
"custom_task_success": true
```

罐子的 z 从 0.065 升到 0.488（茶几台面高度），`OnTop` 由假转真，双爪为空，执行阶段被监控的位姿直写接口调用数为 0。录像与独立状态检查一致。

## 一条必须写清楚的边界

**不能把这次成功完全归因于那两个参数。** 第二次的采样导航目标和路径可用性同时发生了变化——A\* 这回找到了路。要检验参数改动的因果作用，需要固定随机种子和初始状态，做重复对照实验。这次没做。

所以这篇能支持的结论只有一条，但这条我认为是成立的：

> Astra 完整走通了「定义成功条件 → 执行 → 用状态证据判定自己失败 → 定位失败环节 → 提出并实施改动 → 复测验证」的闭环，并在一个真实规划栈上完成了移动操作任务；期间它没有用状态直写绕过任务，也没有把「动作还在发」当成「任务在推进」。

自我迭代策略的能力，验证到的是这个形状；因果强度还欠一组对照实验。

## 原始证据

两次实验独立保存，失败记录没有被覆盖。

- [失败录像 MP4](evidence/failed_attempt_full.mp4)（737 有效帧 · 92.125 秒 · 960×608）
- [成功录像 MP4](evidence/render_motion.mp4)（223 帧 · 27.875 秒 · 960×544）
- [失败审计 JSON](evidence/failure_audit_result.json)（3,630 步 · `custom_task_success: false`）
- [成功审计 JSON](evidence/audit_result.json)（923 步 · `custom_task_success: true`）
- [成功控制轨迹 JSONL](evidence/actions.jsonl)（动作序列与周期性状态观测）
- [失败视频制作记录](evidence/failure_video_info.json)（完整采样帧保留 · 末帧损坏排除说明）

完整实验页（保留双视频同步播放、倍速、打印复盘等交互）：**[打开 →](demo/)**
