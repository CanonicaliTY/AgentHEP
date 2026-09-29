# 阅读工作册：AgentRivet × MadAgents

## 0. 主干与支线

| 节点 | 核心问题 | 主要原文 |
|---|---|---|
| A 物理任务 | LLM 在协助哪一步？真正的物理计算由什么执行？ | AgentRivet §1、§4；MadAgents §1、§3.2.1、§3.4 |
| B 系统设计 | 谁决定下一步？谁看见什么？谁能执行工具？ | AgentRivet §2–3、Fig.1；MadAgents §2、App.A、App.C |
| C 失败追踪 | 定义、摘要、实现、解释，错误在哪一层发生？ | AgentRivet §5–6；MadAgents §3.2.2、App.C |
| D 记忆与改进 | 修改的是代码、上下文、外部文档，还是模型参数？ | AgentRivet §2；MadAgents App.A、C、E |
| E 证据与评价 | 跑通、正确、稳定、泛化，是同一个结论吗？ | AgentRivet §4–7；MadAgents §3、App.E |
| F 研究切入 | 哪个有界问题值得用对照实验研究？ | 基于 A–E 的个人综合，不是论文已有结论 |

核心连接：A→B，B→C，C↔D，C/D→E，E→F。概念问题可以从任何案例反向追踪，不必把所有背景学完才读结果。

## 1. 全文导航

### AgentRivet

- **§1 Introduction，pp.1–2**：HEPData 与 Rivet 的角色；为什么从论文补充 routines 有价值。
- **§2 Code design and LLM orchestration，pp.2–4**：最重要的系统定义；Fig.1；共享状态和结构化输出；不在框架内编译。
- **§3 LLM prompt engineering，p.5**：不同角色的输入、职责、blockers 与 advisories；Physics Reviewer 依照 Analyst 摘要。
- **§4 Benchmark publications and MC simulations，pp.5–6**：两篇测试分析、三种模型、每种配置三次；人工修复编译问题与不修复物理问题的实验边界；与调 prompt 的两篇论文区分。
- **§5 Results，pp.6–11**：Fig.2/3 与错误来源；不要只读作者概括。
- **§6 Discussion，pp.11–12**：reviewer 也会错；编译反馈仍是未来工作；API 问题与测试时费用。
- **§7 Conclusion，p.12**：哪些是报告的成果，哪些是尚待实施的计划。
- **References，pp.13–14**：按问题选支线，不要求全部读。

### MadAgents

- **§1 Introduction，p.3**：作者明确不是在宣称新的物理发现，目标是改变事件生成的使用方式。
- **§2 Setup，pp.4–7**：LLM/tool 背景、初版 LangGraph 架构、容器环境。
- **§3.1 Software Installation，pp.8–11**：选择一次出错→检查→修复→验证的完整链条精读；其余日志扫描。
- **§3.2.1 Tailored tutorials，pp.12–16**：这里 training 主要是教用户使用 MadGraph，不是训练 LLM 权重。读懂教学与交互方式即可。
- **§3.2.2 Learning by doing，pp.17–18**：重点精读。命令的 decay nesting 与 agent 错误解释，接 Appendix C。
- **§3.2.3 Documented reweighted simulation，pp.18–21**：研究从文件/日志重建科学 provenance；不必现在掌握全部 EFT/NLO 技术。
- **§3.3 Supporting experienced users，pp.22–25**：三种模拟策略、Fig.2，以及 agent 的物理解读。第一轮要能区分不同近似与任务选择，不要求自行推导全部高阶 QCD。
- **§3.4 Autonomous event generation，pp.26–28**：重点精读。输入是 HEPTAPOD 示例部分的摘录，不是完整论文；用户也已给出工具链、输出要求、权限和模糊信息处理方式。
- **§4 Outlook，pp.29–30**：将作者总结与实际展示对应起来。
- **Appendix A Individual agents，pp.31–34**：控制器、worker 可见的上下文；计划字段；Summarizer；Table 1 工具权限。源码理解的重要桥梁。
- **Appendix B Example tutorial，pp.35–38**：生成的用户教程实例；要学习 EFT 操作时再细读。
- **Appendix C MadAgents improvements，pp.39–41**：不能跳过。证据要求与它的失败、分拆 reviewer、改为 plan-update 工具、独立上下文、并发 worker、控制器可调工具。
- **Appendix D Dataset documentation，pp.42–53**：生成的长文档。重点看 p.42、pp.45–46、pp.52–53：显式/推测版本、质量参数差异、K-factor 假设与验证边界；其余字段与安装细节按需查。
- **Appendix E Claude Code with self-improvement，pp.54–56**：不能跳过。新的实现方式，以及修改内部文档的六阶段循环。
- **References，pp.57–61**：支线路标。

## 2. 节点 A：物理工作流

### 原文支持的定位

MadAgents 协助使用 MadGraph，并可组织后续 Pythia、FastJet 或 Delphes 的流程。MadGraph/Pythia/Delphes 的例子在 §3.2.1 中被区分为 hard process/parton level、showers/hadronisation、detector simulation。§3.4 的 leptoquark 示例最终使用的是 hadron-level reconstruction，没有额外 detector simulation。

AgentRivet 从已经发表的测量提取分析定义，产生 Rivet C++ routine；作者用 Monte Carlo 事件检查该 routine 产生的分布。Rivet routine 不是事件生成器。

### 教学连接

你已有的 Z→dilepton 分析提供了连接点：你知道 selection、invariant mass、histogram 是什么意思。现在要把“分析已有事件”和“生成事件”分开，也要把生成器中的 particle-level 对象和真实 detector-level 重建对象分开。不能直接把原 lab 的 cuts/对象定义照搬到任意 Rivet 分析。

### 自测

A1. MadGraph 输出的事件与 AgentRivet 输出的 routine，分别是什么类型的对象？
A2. 自动创建一个清楚的质量峰，为什么仍不能证明所有配置和物理解读正确？
A3. 相同代码重跑得到不同 MC 事件，与 LLM 重跑生成不同代码，是哪两类不稳定性？

<details><summary>参考思路</summary>

A1：前者是物理模拟样本；后者是实现分析定义的程序。MadAgents 管理工具调用与流程，不能将 LLM 视为物理求解器本身。

A2：峰只约束了输出的一部分性质。过程、cuts、归一化、对象定义、隐含假设都可能不同。需要核对具体目标和独立证据。

A3：模拟抽样误差与 agent/代码生成的不确定性；控制一种，并不自动控制另一种。
</details>

## 3. 节点 B：控制流、信息流与工具

### AgentRivet §2–3

核心角色是 Analyst、Coder、Code Reviewer、Physics Reviewer。注意三点：

1. 顶层工作流预定义，不表示每次 LLM 输出确定。
2. Pydantic schema 检查输出结构，不会自行证明选择阈值或物理解释符合原文。
3. 当前上传版本不在框架内编译；Code Reviewer 预测可能的编译问题，不是收到真实 compiler error 后在纠错。

### MadAgents §2 / Appendix A

Orchestrator 决定调哪个角色；Planner 和 Reviewer 辅助控制；workers 实际使用相应工具。Table 1 很有用：初版 Orchestrator 没有直接工具权限，但 Reviewer 可以使用 bash 等工具。

### MadAgents Appendix C / E

不可把所有改动写回初版：

- C：增添 Physics-Expert；Reviewer 分为三类；Plan-Updater 由工具取代；上下文分离；Planner 先看环境；Orchestrator 可调用工具并派遣并发 workers。
- E：将 refined setup 放入 Claude Code，复用其编排和部分 planning/review 功能；另加 documentation self-improvement。

### 你的产出

两篇各写一张小表：角色｜输入可见内容｜能执行的动作｜输出｜下一步由谁决定。只写五六行就足够；关键是画出“看不到什么”。

### 自测

B1. Reviewer 看不到 Analyst 漏掉的定义时，怎样可能批准错误实现？这是已证实的普遍现象，还是从架构推出的待检验风险？
B2. 初版 MadAgents 与 Appendix C 对 Orchestrator 的工具权限，有什么不同？
B3. 为什么“多一个 agent”不自动意味着多一个独立证据来源？

<details><summary>参考思路</summary>

B1：代码可以与错误/不完整摘要一致。原文报告了具体遗漏案例，但把此风险推广为所有分析上的频率需要实验。

B2：初版通过结构化输出委派 worker；C 增加直接 tool calling 与并行派遣等设计。

B3：它们可能共享同样的摘要、先验误解和错误引用；角色分开不等于依据独立。
</details>

## 4. 节点 C：追踪三个失败，而不是背错误名词

### C1 AgentRivet：摘要遗漏 → 下游不实现

定位：§5.2，p.9。作者记录某模型的 Analyst 将复杂角变量概括为在 special reference frame 定义，没有传递原文给出的坐标系细节；Coder 因此没有实现这些量。

练习：写出五格链条：原文定义 → 摘要保留了什么 → Coder 做了什么 → Reviewer 能看到什么 → 最终缺失是什么。

### C2 AgentRivet：程序能运行，二维分布却归一化错误

定位：§5.2，pp.9–11 与 Fig.3。用整数 index 展示二维 bins 时，不能把 index 的宽度当成物理 bin 面积。作者用这点解释了某些与官方 routine 的整体差异。

教学式表达：若一个 bin 内的积分截面是 Δσ_i，面积是 Δx_iΔy_i，则该 bin 的平均微分截面为 Δσ_i/(Δx_iΔy_i)。这是教学推导，不是替原论文补造数据。

练习：先看 Fig.3 左右两列为什么传递不同信息，再看 notebook 中的人工小例子。两种模型给相似结果，仍可能共有同一错误。

### C3 MadAgents：命令可以执行，文字解释仍然错误

定位：§3.2.2 pp.17–18 → Appendix C pp.39–40。

原文指出 `generate p p > z h, (h > z z, z > e+ e-)` 会使 Higgs 衰变出的两个 Z 都进一步衰变，关联产生的 Z 保持不衰变。agent 的解释曾误说只有一个 Higgs 产生的 Z 衰变。

这里只追踪作者讨论的命令语义；原文同时说明了 on-shell Higgs→ZZ 的运动学限制与过程生成阶段的区别，不应把该行当作任意参数下可成功产生事件的完整设置。

接着作者增加证据要求，但又发现“找到网上例子”仍可能支撑错误解释，于是加强 reviewer 对精确上下文和直接证据的要求。

练习：区分“有引用”“引用相关”“引用直接支持这条具体命题”。证明一个配置能工作，能否证明它是唯一合法配置？

## 5. 节点 D：三种改变与 self-improvement

### 区分对象

- AgentRivet 的缓存、结构化摘要和代码草稿：保存当次工作及可复用中间产物。
- MadAgents 的历史摘要、future_note、计划状态：控制上下文与任务推进。
- Appendix E 的内部 MadGraph 文档：可以跨问题积累、修订的外部知识。

Appendix E 描述的 self-improvement 改的是文档与之后可用的上下文，不是梯度更新 LLM 权重，也不是论文声称训练出了一个新 foundation model。

### 原文六阶段

Question generation → Evaluating MadAgents → Extracting and validating claims → Grading and diagnosing → Documentation improvement → Re-evaluation / optional iteration。

每个回答中具体 claims 的核验，与“整道题答对”的判定并不相同。p.54 明确允许题目被判正确，同时带有 mistakes 标签。记录这个差别，避免将后面的“正确率”想象成每句话都正确。

p.55 的 gridpack 示例报告 13:34 到 7:37 的变化；它是一个具体示例，不是全面平均提速。另一个示例讨论文档缺失、模式区分和定义埋藏带来的错误。

### 自测

D1. 在失败问题上反复改文档，再在原问题上重测，证明了什么？要证明对新问题更好还缺什么？
D2. 文档内容完全相同，但位置与组织改变，为什么值得作为实验变量？
D3. 证据验证器也依赖 LLM 时，“核验过”有没有新的失误来源？

<details><summary>参考思路</summary>

D1：说明原案例上的回归修复/局部适应；泛化需要未参与修订的问题，以及重复与对照。

D2：正文 p.55–56 给出了重新组织定义后纠错的案例。机制是否能推广，要单独测试，不能从一个案例直接概括。

D3：可能误读命令、源码或物理推导；验证命题范围比证据范围更宽时也会错。需要独立核验的一部分样本和明确判定标准。
</details>

## 6. 节点 E：两篇论文各自展示了多少？

| 维度 | AgentRivet | MadAgents |
|---|---|---|
| 主要目标 | 论文→Rivet routine 的分析保存 | 使用、学习、管理 MadGraph 模拟流程 |
| 主体评价 | 两篇分析，三种模型，每种三次；检查代码和物理分布 | 安装、教学、文档、复杂模拟、自主任务的具体运行展示 |
| 重要范围限制 | 任务少；人工修编译错误需计入边界；无全面 review 消融 | 主体以 individual invocations 展示，不是任务集上的总体成功率；C/E 改进不能倒推成主体实验设置 |
| 外部结果检查 | 作者编译、MC 运行及后加入的官方 routine 对比 | 环境与执行结果、review；改进版追加更严格证据核验 |
| 尚不能直接推出 | 普遍可靠，或 review 在所有任务上都有效 | 大规模泛化成功率，或动态编排比固定流程更优 |

这是对论文实验设计的阅读分析，不是对软件整体能力的排名。

### 自测

E1. 同一个 analysis 跑十次，与十个不同 analysis 各跑一次，回答的是同一问题吗？
E2. 哪些步骤由 agent 完成，哪些由作者在系统外完成？
E3. 两篇使用不同任务、工具、模型、输出与检查方法，为什么不能直接用它们的运行结果比较架构优劣？

## 7. 节点 F：带去 meeting 的问题种子

这些都是待讨论的假设，不是声称文献中的空白已得到新颖性验证。

**独立证据的价值。** 在固定任务上，让 reviewer 获得原文片段或可执行检查，是否减少“代码符合摘要，却不符合原文”的情况？

**选择性验证。** 给所有步骤同样的 review 预算，与对不确定/高代价步骤集中验证，分别有什么效果？需要明确成本、漏检和错误接受的衡量方式。

**记忆的迁移。** 从失败案例修订文档，能否改善未见过的相关问题？能否避免把一个版本/模式的规则错误泛化？

**控制与工具解耦。** 不直接拿两个整套框架比赛，而选择共同子任务、相同底层模型与工具，单独改变控制方式或反馈来源。这是公平比较的实验建议，不是会前需要完成的开发任务。

## 8. 可选文献支线（只是索引）

以下来自两篇论文的参考文献；此包没有额外声称已核验或总结这些文章全文。

- 理解 action/observation loop：MadAgents [26] ReAct。
- 理解工具接口为何是研究变量：MadAgents [30] SWE-agent。
- 理解为何修改外部记忆也叫学习：MadAgents [65] Reflexion、[66] ExpeL。
- 理解自主模拟的原任务：MadAgents [51] HEPTAPOD。
- 理解更广的 HEP agent 评价：AgentRivet [15] Collider-Bench。

周四前可以只深入其中一条，也可以一条都不读。两篇主文中的具体案例更重要。

## 9. 与对话的接口

每次只带一个节点或一个卡点回来。例如：

“从 A 开始，先讲清 MadGraph/Pythia/Rivet 的分工”；“我看了 Fig.3，为什么多个模型一致还可能错”；“Appendix E 到底哪里在学习”。

每个节点的学习循环：原文一小段 → 用自己的话复述 → 一个具体例子 → 一两个自测 → 把新问题接回主图。资料给出完整地图，不意味着要一次完成所有支线。
