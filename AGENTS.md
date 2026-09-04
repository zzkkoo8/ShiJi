# AGENTS.md

本文件约束所有参与《史纪》开发的 AI/Codex Agent。

## 1. 开发原则

- **文档优先**：任何会改变玩法、历史边界、技术架构的工作，先更新对应设计文档，再实现。
- **Feature Branch**：每项功能使用独立分支；`main` 只保留已审查的稳定基线。
- **最小实现**：优先完成最小可玩闭环，不因“以后可能需要”提前建设系统。
- **官方优先**：优先 Unity 官方包、标准 C# 和成熟通用工具；新增第三方框架必须说明必要性。
- **先证据后优化**：没有 Profiler/测试证据，不得因性能猜测提前引入 DOTS/ECS、复杂对象池或自研底层框架。
- **单机优先**：MVP 不加入多人联机、账号、排行榜、商城、赛季等在线系统。

## 2. 设计基线

实现前至少阅读：

1. `docs/00-overview/GAME_VISION.md`
2. `docs/01-design/CORE_GAMEPLAY.md`
3. `docs/01-design/NARRATIVE_RULES.md`
4. `docs/03-tech/TECH_STACK.md`
5. `docs/04-production/MVP_ROADMAP.md`

若需求与上述文档冲突，不得自行选择；先提出冲突并修改设计基线。

## 3. 历史内容约束

- 不得仅凭模型记忆编造史实。
- 历史、建筑、服饰、兵器、制度、城市布局必须记录来源。
- 内容标记为三类：`史实`、`合理复原`、`游戏虚构`。
- 证据优先级：考古材料 / 博物馆与官方文保资料 > 同时代实物和图像 > 同时代文献 > 现代学术研究 > 艺术推断。
- 真实历史人物不得为了爽点随意改写核心生平；可变内容遵循 `NARRATIVE_RULES.md`。

## 4. 技术约束

MVP 默认：

- Unity 6.3 LTS
- C#
- URP
- Input System
- uGUI + TextMeshPro
- Cinemachine
- AI Navigation / NavMesh
- ScriptableObject + JSON
- MonoBehaviour + 普通 C# 类

未经过设计审查，不加入：

- DOTS / Entities
- Nakama 或其他联网后端
- 大型 Unity 第三方游戏框架
- 自研 ECS
- 自研网络协议
- 开放世界流式框架

## 5. 代码与模块

优先保持模块边界清楚，而不是追求抽象层数：

- Hero：玩家角色
- Combat：战斗与伤害
- Army：军团与单位
- Command：军令
- AI：军团/单位决策
- Economy：军资与物资
- Quest：诏令与任务
- Narrative：剧情与选择
- History：历史锚点
- Save：存档
- UI：界面

模块通过明确接口或事件交互；避免巨型 Manager、全局可写单例和循环依赖。

## 6. 工作流程

1. 阅读基线文档。
2. 明确本 Feature 的目标与验收。
3. 建 Feature 分支。
4. 先写/更新设计与测试条件。
5. 实现最小功能。
6. 运行测试或可重复验收。
7. 自审是否引入不必要复杂度。
8. 提交 PR，再合入 `main`。

完成标准不是“代码写完”，而是**目标可验证、设计文档与实现一致、没有明显过度设计**。
