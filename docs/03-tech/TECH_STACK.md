# TECH_STACK

## 1. 技术目标

技术选择只服务于当前产品目标：

- Windows PC + Android 共用核心代码
- 英雄直接操作
- 100～300 个可见战斗单位的 MVP
- AI 军团间接控制
- 章节化战役地图
- 单人剧情
- 尽量让 Codex/AI 能稳定理解、修改和测试

因此第一阶段优先选择**Unity 官方能力 + 普通 C#**，不提前建设“大型 RTS 引擎”。

## 2. 基础栈

| 模块 | 选择 | 原因 |
|---|---|---|
| 游戏引擎 | **Unity 6.3 LTS** | 当前 LTS，跨 Windows/Android/iOS，生态成熟 |
| 语言 | **C#** | Unity 主语言，AI/Codex 支持成熟 |
| 渲染 | **URP** | 同一渲染管线兼顾移动端和 PC |
| 输入 | **Unity Input System** | 键鼠、手柄、触摸统一 Action 层 |
| UI | **uGUI + TextMeshPro** | 成熟、资料多、AI 易维护，MVP 足够 |
| 摄像机 | **Cinemachine** | 官方成熟方案，减少自研镜头代码 |
| 寻路 | **AI Navigation / NavMesh** | MVP 足够，支持动态障碍与运行时导航 |
| 动画 | **Animator / Mecanim** | Unity 标准动画链路 |
| 配置 | **ScriptableObject** | 兵种、技能、武器、任务等静态配置 |
| 存档 | **本地 JSON** | 单机 MVP 足够，易调试、易迁移 |
| 测试 | **Unity Test Framework + 可重复场景验收** | 保证核心逻辑可回归 |
| 版本控制 | **Git + GitHub** | 代码与文档唯一事实源 |
| 大文件 | **Git LFS（资产进入后再启用）** | 管理模型、贴图、音频等二进制资源 |

Unity 6.3 LTS 官方支持期至 2027 年 12 月；项目创建时固定具体 Editor patch 版本，之后只在明确需要时升级。

官方参考：

- Unity 6 支持策略：https://unity.com/releases/unity-6/support
- Input System：https://docs.unity3d.com/6000.0/Documentation/Manual/com.unity.inputsystem.html
- AI Navigation：https://docs.unity3d.com/6000.0/Documentation/Manual/com.unity.ai.navigation.html
- Addressables：https://docs.unity3d.com/6000.0/Documentation/Manual/com.unity.addressables.html

## 3. 编程框架：不引入大型第三方框架

MVP 不采用完整第三方游戏框架、DI 容器或自研 ECS。

默认结构：

- `MonoBehaviour`：Unity 生命周期、场景对象和表现层
- 普通 C# 类：战斗计算、规则和状态
- `ScriptableObject`：静态配置
- C# interface / event：模块边界

不默认引入：

- Zenject / Extenject / VContainer
- Game Creator 等完整玩法框架
- Behavior Designer 等商业行为树
- DOTS / Entities
- 自研全局 Event Bus
- 大型 Service Locator

只有实际复杂度证明需要时再增加依赖。

## 4. 最小模块边界

```text
Hero
 ├─ Input
 ├─ Movement
 └─ Skills

Combat
 ├─ Damage
 └─ Targeting

Army
 ├─ ArmyController
 ├─ Unit
 └─ Formation(后置)

Command
 └─ Attack / Defend / Support / Retreat

AI
 ├─ ArmyStateMachine
 └─ UnitStateMachine

Economy
 └─ Supplies

Quest
 └─ Edict / Objective

Narrative
 └─ Choice / Consequence

History
 └─ HistoricalAnchor

Save
UI
```

这只是职责边界，不要求每个框都变成复杂子系统。

## 5. AI 实现方案

MVP 使用**分层有限状态机（FSM）**，不先上行为树或 Utility AI 框架。

### 军团级状态

```text
Idle
Move
Attack
Defend
Support
Retreat
```

玩家军令改变军团目标状态，军团 AI 决定具体路线和局部目标。

### 单位级状态

只实现实际需要的最少状态，例如：

```text
FollowArmy
MoveToSlot
Engage
Fallback
Dead
```

如果后续出现大量“多个目标如何权衡”的复杂决策，再局部引入评分/Utility 思路，而不是重写整个 AI。

## 6. 性能策略

目标先按 **100～300 个可见单位**验证玩法。

优化顺序：

1. Profiler 找实际瓶颈
2. 降低 AI/寻路更新频率
3. 分帧计算
4. 对象池和 LOD
5. Burst / Jobs
6. 只有确有必要才评估 DOTS/Entities

禁止因为“RTS 最终可能有上千单位”而在第一天 ECS 化。

## 7. 资源管理

MVP 场景和资产较少时使用普通 Unity 引用。

只有出现下列需求时启用 Addressables：

- 大量章节资源
- 按需加载城市/战役资产
- DLC 或资源分包
- 移动端安装包体积需要拆分

不为未来假设提前增加 Addressables 复杂度。

## 8. 平台顺序

开发与验证顺序：

```text
Windows 编辑器/PC 原型
        ↓
Android 真机操作与性能
        ↓
Windows 正式适配
        ↓
iOS（核心玩法稳定后）
```

输入层从第一天使用 Input Actions 抽象，避免后期重写 PC/手机控制逻辑。

## 9. 明确后置

以下不是当前技术依赖：

- Nakama / PlayFab / 自研后端
- Netcode / Photon / Mirror
- PvP 同步
- 云存档
- 账号系统
- 商城与支付
- DOTS/Entities
- 开放世界流式加载

如果未来增加联机，再单独做网络架构设计，不让网络约束当前单机原型。

## 10. 技术决策原则

新技术只有满足以下任一条件才引入：

1. 当前功能不用它明显难以实现；
2. 已有性能数据证明现方案不够；
3. 它能显著减少代码和长期维护成本。

否则保持现状。
