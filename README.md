# Firmware Doc Reviewer

统一 PRD 与固件侧 PFS 评审 Skill。它按请求中的文档类型和评审视角加载 `rules/` 中对应规则，输出有证据、可追踪、可执行的结构化结论。

当前支持三类评审：

- **固件侧 PFS Review**：对固定 PFS 结构中的规格承接、PRD 双向追溯与开发准备度进行评审。
- **通用 PRD Review（产品经理视角）**：默认视角，判断 PRD 是否让研发可理解、初步拆分、估算并验收；不要求代码、协议或架构方案。
- **固件侧 PRD Review**：仅在请求明确指定固件侧、手表端、嵌入式或 firmware 时使用，判断 PRD 是否具备进入固件研发阶段的条件。

## 路由

| 优先级 | 识别方式 | 规则文件 |
|---:|---|---|
| 1 | `PFS`、`PFS Review`、“PFS 评审”、“产品功能规格书”或“功能规格说明”等固定 PFS 结构表达 | `rules/pfs-review-fw.md` |
| 2 | 明确包含“固件侧”“固件”“手表端”“嵌入式”“firmware”或等价视角；即使同时出现通用视角仍优先此项 | `rules/prd-review-fw.md` |
| 3 | 明确包含“通用 PRD”“产品经理视角”“产品侧”“PM 视角” | `rules/prd-review.md` |
| 4 | 裸 `PRD`、`PRD Review`、`产品需求文档`、`需求评审` | `rules/prd-review.md` |

未来新增明确评审视角时，在路由中增加对应规则，并置于默认通用路由之前。

## 输入与评审原则

PRD Review 的最小输入为 PRD 链接或正文，以及 PRD 已明确的目标、范围和涉及端/团队。交互稿、资源、数据口径、依赖、合规、发布计划和验收材料均可作为关联证据；PFS Review 的输入与基准 PRD 门槛见下文“固件侧 PFS Review”。

- 默认只读：不修改文档，不创建 Jira，不提交代码，不发布固件。
- 证据优先：只依据 PRD 和明确关联资料，不把推断写成事实。
- 关联资料无法访问时，记录其影响；只有实质阻断决策、研发拆分、交付或验收时才设 Block。
- 通用规则中，执行、适配、合规或项目依赖需求不因缺少独立业务 KPI 扣分或设 Block；可用问题来源、外部要求、覆盖范围或完成条件说明价值。
- 固件设备、版本、端侧归属及实现可行性核查只适用于固件侧规则；代码仓库不是任一视角的默认输入。
- 明确不适用维度直接跳过，按适用权重折算为 100 分。

## 通用 PRD Review

通用视角按“价值与范围 → 用户与流程 → 研发影响与依赖 → 风险与验收”评审。它关注新增、调整或复用边界，影响端/系统/数据/资源，跨团队与外部依赖，以及研发能否完成初步拆分与估算；不要求模块路径、接口字段、协议报文、数据库表结构或架构设计。

评分维度：

| 维度 | 分值 |
|---|---:|
| 需求价值与决策依据 | 5 |
| 目标用户与使用场景 | 10 |
| 范围、优先级与版本边界 | 15 |
| 功能规则与端到端流程 | 15 |
| 交互、内容与产品输入 | 10 |
| 研发影响与可行性边界 | 20 |
| 数据、依赖与责任边界 | 10 |
| 非功能约束与交付风险 | 10 |
| 验收标准、发布与效果验证 | 5 |

`>=80` 且无 Block 为通过；`60–79` 且无 Block 为有条件通过；`<60` 或任意 Block 为不通过。Block 仅针对阻断产品决策、研发拆分、交付或验收的缺口。

输出严格只包含 `## 1. 关键信息`，以及 `### 1.1 关键问题与关闭动作`、`### 1.2 结论依据`、`### 1.3 评分细则`；不输出第二部分。

示例：

```text
请使用 firmware-doc-reviewer 对这个 PRD 做评审。
请使用 firmware-doc-reviewer 按产品经理视角评审这个产品需求文档。
```

## 固件侧 PRD Review

固件侧规则评估固件主责项和会影响固件交付的必要跨端依赖，覆盖设备能力、端间数据契约、适用的版本/兼容策略、状态机、资源输入、可靠性、验证和依赖治理。纯其他端事项不进入固件评分。

固件侧报告保留 `关键信息` 与按适用内容输出的 `详细信息`；具体评分、Block 和模板见 [`rules/prd-review-fw.md`](rules/prd-review-fw.md)。

示例：

```text
请使用 firmware-doc-reviewer 进行固件侧 PRD Review，重点判断是否具备进入固件研发阶段的条件，并按固定模板输出。
```

## 固件侧 PFS Review

PFS Review 以 PFS `Description` 为主要规格证据，以可访问且唯一的基准 PRD 为真值来源。它支持飞书 Sheet、飞书 Docx 与 Markdown；对 Sheet 先还原合并单元格和分组继承，排除表头、说明行、空模板行和仅含下拉校验的预留行，再识别实际需求行。

`Index` 仅用于 PFS 内部定位，不假定可对应 PRD。`Feature` 是 Description 所属的功能模块上下文，可按分组继承且非必填。`PRD Index` 有值时作为优先追溯锚点；无值时根据 PRD 引用、章节、原文和 Description 的可证实语义建立追溯。

`执行程度` 中的 `SHALL/SHOULD/MAY` 是 PFS 填写者的承接声明。评审以 PRD 核验其合理性；空值或表面不一致不机械扣分或设 Block，仅在实际导致漏做、范围失控、无法拆分或无法验收时形成差异或 Block。`PFS Comments` 和 `Feature Owner` 仅在存在例外、待确认、跨团队依赖、关闭动作或责任分派时要求；其余固定列作为辅助证据，空值不机械扣分。

具体差异定义、Block、评分和固定输出模板见 [`rules/pfs-review-fw.md`](rules/pfs-review-fw.md)。

示例：

```text
请使用 firmware-doc-reviewer 对这个 PFS Sheet 做评审，并核验其与 PRD 的承接。
请使用 firmware-doc-reviewer 进行 PFS Review，重点检查 Description、执行程度和 PRD 双向追溯。
```

## 目录结构

```text
firmware-doc-reviewer/
├── SKILL.md              # 视角路由、通用边界和扩展约定
├── agents/openai.yaml    # Agent 展示信息与默认提示词
└── rules/
    ├── pfs-review-fw.md  # 固件侧 PFS Review
    ├── prd-review.md     # 通用 PRD Review：产品经理视角
    └── prd-review-fw.md  # 固件侧 PRD Review
```

## 扩展评审视角或文档类型

1. 在 `rules/` 下新增对应 Markdown 规则；
2. 在 `SKILL.md` 的路由中补充识别关键词与目标规则文件，并在默认通用路由之前安排明确视角；
3. 不为每种视角或文档类型创建独立 Skill；
4. 执行时只加载当前规则文件。

---

Owner: cs-dongqi@zepp.com
Organization: Active.Bu
