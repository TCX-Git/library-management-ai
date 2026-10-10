# 风险分析文档：图书管理系统

> **状态**：草稿（未经人工审查）
> **说明**：本内容由 Agent 生成，未经人工审查，不得直接作为实现依据。
> **依据**：`specs/00-project-brief.md`、`specs/01-clarifying-questions.md`、`specs/constitution.md`、`specs/02-requirements.md`、`specs/03-use-cases.md`、`specs/04-use-case-model.puml`、`specs/05-domain-model.md`、`specs/06-domain-class-diagram.puml`、`specs/07-architecture.md`、`specs/08-package-diagram.puml`、`specs/13-database-design.md`
> **说明**：`specs/09-design-model.md`、`specs/10-sequence-borrow-book.puml`、`specs/11-sequence-return-book.puml`、`specs/12-sequence-reserve-book.puml`、`specs/14-api-spec.md`、`specs/15-test-plan.md` 当前均为空文件，相关风险已在本分析中识别。

---

## 一、风险评估方法

### 1.1 等级定义

| 维度 | 取值 | 说明 |
| --- | --- | --- |
| **可能性（P）** | 高 / 中 / 低 | 风险发生的概率。已确认为空文件或已存在明确冲突的，可能性记为"高"或"确定"。 |
| **影响（I）** | 高 / 中 / 低 | 风险发生后对项目目标（可运行、可追溯、可验收）的破坏程度。 |
| **风险等级** | 严重 / 高 / 中 / 低 | 由"可能性 × 影响"综合判定。 |

### 1.2 判定矩阵

| 可能性＼影响 | 高 | 中 | 低 |
| --- | --- | --- | --- |
| **高** | 严重 | 高 | 中 |
| **中** | 高 | 中 | 低 |
| **低** | 中 | 低 | 低 |

### 1.3 风险编号约定

| 前缀 | 风险类别 |
| --- | --- |
| R-REQ | 需求风险 |
| R-MOD | 建模风险 |
| R-ARC | 架构风险 |
| R-DB | 数据库风险 |
| R-PERM | 权限风险 |
| R-AGENT | Agent 使用风险 |

---

## 二、需求风险

| 编号 | 风险描述 | 可能性 | 影响 | 等级 | 证据来源 |
| --- | --- | --- | --- | --- | --- |
| R-REQ-01 | 待确认问题 P-01~P-07（共 7 项）全部未关闭，涉及办理借阅证边界、管理员角色、杂志/论文上限、续借起算、自助办理、登录态、预约欠费联动等，影响后续设计、编码与测试依据 | 高 | 高 | **严重** | `02-requirements.md` 第 8 章；`03-use-cases.md` 多处"待确认" |
| R-REQ-02 | 澄清回答 A5 与 A4 内容重复（疑似复制粘贴错误），导致"办理借阅证与注册账号关系"未得到真实澄清，用例边界不确定 | 中 | 中 | **中** | `01-clarifying-questions.md` A4/A5 |
| R-REQ-03 | 管理员角色划分口径三方冲突：A1（单一 Admin）vs A3（职责分离）vs 宪法第 9 条（区分图书管理员/系统管理员），权限设计、数据库表拆分、用例图 actor 均受牵连 | 高 | 高 | **严重** | `02-requirements.md` P-02；`constitution.md` 第 9 条；`04-use-case-model.puml`；`13-database-design.md` |
| R-REQ-04 | 杂志/论文的借阅数量是否另设独立上限未明确（P-03），借阅规则种子数据与 BR 实现不确定 | 中 | 中 | **中** | `02-requirements.md` P-03；`13-database-design.md` 第十一章 |
| R-REQ-05 | 需求文档状态为"草稿（未经人工审查）"，尚未进入 baseline，按宪法第 4 条不能作为实现唯一依据 | 高 | 高 | **严重** | `02-requirements.md` 头部；`constitution.md` 第 4 条 |
| R-REQ-06 | 读者自助办理借阅/归还/续借的范围未明确（P-05），影响用例主参与者与权限边界 | 中 | 中 | **中** | `02-requirements.md` P-05 |
| R-REQ-07 | NFR-001 性能指标标注"待定"，无法作为性能验收依据 | 低 | 低 | **低** | `02-requirements.md` NFR-001 |

---

## 三、建模风险

| 编号 | 风险描述 | 可能性 | 影响 | 等级 | 证据来源 |
| --- | --- | --- | --- | --- | --- |
| R-MOD-01 | "图书标题（Title）"概念映射不一致：需求文档有独立 Title 概念（6.3 节），领域模型将书目信息并入 `LibraryItem` 不单列 Title 类，数据库又使用 `book_titles` 表，三处口径不统一，实现时实体/表映射易出错 | 高 | 中 | **高** | `02-requirements.md` 6.3；`05-domain-model.md` 1.2；`13-database-design.md` 5.5 |
| R-MOD-02 | 读者类型鉴别存在冗余：`Reader.readerType`（5 类）与 `StudentReader.studentType`（4 类）取值重叠，领域模型/类图仍保留两套，虽数据库已简化合并，但文档层面未完全收敛 | 中 | 低 | **低** | `05-domain-model.md` 3.1；`06-domain-class-diagram.puml`；`13-database-design.md` 3.1 |
| R-MOD-03 | 用例图 actor 命名与拆分不一致：`04-use-case-model.puml` 使用 Librarian/SystemAdmin/Student/Teacher，`03-use-cases.md` 内嵌用例图使用 Admin/SuperAdmin/学生读者/教师读者，且 04 将 Student、Teacher 泛化自 Reader，与需求的 StudentReader/TeacherReader 表述存在偏差 | 高 | 中 | **高** | `03-use-cases.md` 2.1；`04-use-case-model.puml` |
| R-MOD-04 | 借出物"类型 × 语种"维度组合建模未统一：中文图书与外文图书罚款单价不同（BR-008），但 `LibraryItem` 的 `item_type` 仅 Book/Magazine/Thesis，语种由 `language` 单列，`FineRule` 规则键如何细分未定 | 高 | 中 | **高** | `02-requirements.md` BR-008；`13-database-design.md` 5.10、第八章 |
| R-MOD-05 | 设计模型（09）与三个核心时序图（10/11/12）均为空，借阅/归还/预约核心流程缺乏详细设计与时序支撑，无法验证架构落位与用例实现 | 高（确定为空） | 高 | **严重** | `09-design-model.md`、`10/11/12-sequence-*.puml` 空文件 |

---

## 四、架构风险

| 编号 | 风险描述 | 可能性 | 影响 | 等级 | 证据来源 |
| --- | --- | --- | --- | --- | --- |
| R-ARC-01 | API 规格（14）与测试计划（15）均为空，接口契约与测试策略缺失，无法据此开展编码与测试验收 | 高（确定为空） | 高 | **严重** | `14-api-spec.md`、`15-test-plan.md` 空文件 |
| R-ARC-02 | 登录态技术选型（Session 或 Token）未确定（P-06），鉴权实现与安全设计存在不确定性 | 中 | 中 | **中** | `02-requirements.md` P-06；`07-architecture.md` 第四章 |
| R-ARC-03 | 并发控制方案仅给出"乐观锁或悲观锁"二选一，未最终确定，`Copy` 是否加 `version` 字段、锁策略如何落地待定 | 中 | 中 | **中** | `07-architecture.md` 6.2 |
| R-ARC-04 | 预约 7 天保留期自动取消、站内通知的定时/异步机制未设计，BR-014/UC-14 的"过期自动取消"缺乏实现方案 | 中 | 高 | **高** | `02-requirements.md` BR-014；`03-use-cases.md` UC-14 |
| R-ARC-05 | 权限矩阵（第四章 4.3）与用例图 actor、数据库管理员表拆分均受 P-02 未决影响，存在返工风险（与 R-REQ-03 同源） | 高 | 高 | **严重** | `07-architecture.md` 4.1 注、4.3；`13-database-design.md` |
| R-ARC-06 | 领域层 `Repository` 接口的归属表述存在歧义（"定义于领域层或基础设施层接口包"），依赖倒置边界不够清晰 | 低 | 低 | **低** | `07-architecture.md` 2.3、2.4 |

---

## 五、数据库风险

| 编号 | 风险描述 | 可能性 | 影响 | 等级 | 证据来源 |
| --- | --- | --- | --- | --- | --- |
| R-DB-01 | `fine_rules.item_type` 语种细分取值（如 `Book_Chinese`/`Book_Foreign`）未确认，罚款规则表数据与 BR-008 的映射不确定 | 高 | 中 | **高** | `13-database-design.md` 5.10、第八章、第十一章 |
| R-DB-02 | `librarians`/`system_admins` 是否拆分依赖 P-02 未决，管理员表结构存在返工风险 | 高 | 中 | **高** | `13-database-design.md` 第二章说明 1、第十一章 |
| R-DB-03 | 单表继承导致 `book_titles` 大量可空列（如 `isbn` 仅 Book 非空），数据完整性依赖应用层校验，存在脏数据风险 | 中 | 中 | **中** | `13-database-design.md` 3.1、5.5 |
| R-DB-04 | SQLite 默认不强制外键，需 `PRAGMA foreign_keys = ON` 显式开启，若遗漏将导致引用完整性失效 | 中 | 高 | **高** | `13-database-design.md` 第六章 |
| R-DB-05 | 罚款金额小数精度：应用层 `Decimal` 计算与 SQLite `REAL` 存储的一致性需严格把控，否则出现浮点误差 | 低 | 中 | **低** | `13-database-design.md` 1.4、5.10 |
| R-DB-06 | `borrow_policies` 种子数据对杂志/论文的 `max_count` 取值依赖 P-03 未决，初始规则数据可能不完整 | 中 | 低 | **低** | `13-database-design.md` 第九章、第十一章 |
| R-DB-07 | 表名 `book_titles` 与领域类 `LibraryItem` 语义不完全对应（"标题" vs "借出物"），与 R-MOD-01 同源，易造成命名误解 | 低 | 低 | **低** | `13-database-design.md` 5.5；`05-domain-model.md` 1.2 |

---

## 六、权限风险

| 编号 | 风险描述 | 可能性 | 影响 | 等级 | 证据来源 |
| --- | --- | --- | --- | --- | --- |
| R-PERM-01 | RBAC 角色口径冲突（P-02）导致权限矩阵、用例图 actor、数据库管理员表三处不一致，越权防护设计存在偏差 | 高 | 高 | **严重** | `02-requirements.md` 2.4；`07-architecture.md` 4.3；`04-use-case-model.puml` |
| R-PERM-02 | 读者自助办理范围未明确（P-05），借阅/归还/续借的权限边界（仅管理员柜台 vs 读者自助）不确定 | 中 | 高 | **高** | `02-requirements.md` P-05；`03-use-cases.md` 1.2 |
| R-PERM-03 | 密码加密存储的具体算法与盐值策略未明确，NFR-003 安全要求落实存在风险 | 中 | 中 | **中** | `02-requirements.md` NFR-003；`07-architecture.md` 2.4 |
| R-PERM-04 | 数据级鉴权（读者仅能查询本人借阅信息）依赖应用服务层实现，但缺乏明确设计细节，存在横向越权（IDOR）风险 | 中 | 高 | **高** | `07-architecture.md` 4.2；`02-requirements.md` FR-013 |
| R-PERM-05 | 操作日志/审计的覆盖范围与存储方案仅架构层一句话提及，未细化，NFR-003 审计要求不完整 | 低 | 低 | **低** | `07-architecture.md` 4.2、2.4 |

---

## 七、Agent 使用风险

| 编号 | 风险描述 | 可能性 | 影响 | 等级 | 证据来源 |
| --- | --- | --- | --- | --- | --- |
| R-AGENT-01 | 澄清回答出现复制粘贴错误（A5 = A4），表明 Agent 生成内容未经充分校验即被当作下游依据，错误会向需求/设计/实现逐层传播 | 高 | 中 | **高** | `01-clarifying-questions.md` A4/A5 |
| R-AGENT-02 | 多份文档由 Agent 独立生成，已出现跨文档口径偏差（Title 概念、管理员角色、actor 命名、借出物语种维度），一致性存在系统性风险 | 高 | 高 | **严重** | 多个文档交叉比对（见 R-MOD-01/03/04） |
| R-AGENT-03 | 所有文档均标注"未经人工审查"，且无审查记录/日志，宪法第 6 条"人工审查优先"尚未执行，baseline 未建立 | 高 | 高 | **严重** | 各文档头部状态；`constitution.md` 第 6 条 |
| R-AGENT-04 | 空文件（09/10/11/12/14/15）表明后续仍需 Agent 生成大量内容，若一次性批量生成，易引入新的一致性问题且难以逐项校验 | 高 | 中 | **高** | `09~15` 空文件 |
| R-AGENT-05 | 当前无 baseline 状态记录，Agent 后续可能误改已定稿文档，违反宪法第 4 条第 3 款"禁止擅自修改 baseline" | 中 | 中 | **中** | `constitution.md` 第 4 条 |
| R-AGENT-06 | 历史数据归档/清理（BR-016 可选按年归档）等边界行为若在 Agent 生成设计时被遗漏，将造成需求与实现脱节 | 低 | 低 | **低** | `02-requirements.md` BR-016 |

---

## 八、风险汇总与优先级建议

### 8.1 汇总统计

| 等级 | 数量 | 风险编号 |
| --- | --- | --- |
| 严重 | 9 | R-REQ-01、R-REQ-03、R-REQ-05、R-MOD-05、R-ARC-01、R-ARC-05、R-PERM-01、R-AGENT-02、R-AGENT-03 |
| 高 | 11 | R-MOD-01、R-MOD-03、R-MOD-04、R-ARC-04、R-DB-01、R-DB-02、R-DB-04、R-PERM-02、R-PERM-04、R-AGENT-01、R-AGENT-04 |
| 中 | 8 | R-REQ-02、R-REQ-04、R-REQ-06、R-ARC-02、R-ARC-03、R-DB-03、R-PERM-03、R-AGENT-05 |
| 低 | 8 | R-REQ-07、R-MOD-02、R-ARC-06、R-DB-05、R-DB-06、R-DB-07、R-PERM-05、R-AGENT-06 |

> 注：汇总表中"严重"档按风险条目逐条统计，跨类别同源风险（如 R-REQ-03 与 R-ARC-05、R-PERM-01 同源于 P-02）建议合并处置。

### 8.2 处置优先级建议

1. **立即处置（阻塞性，进入编码前必须解决）**：
   - 关闭 P-01~P-07 全部待确认问题，尤其 P-02（管理员角色口径）——它同时牵动权限矩阵、数据库表拆分、用例图 actor（R-REQ-03 / R-ARC-05 / R-PERM-01）。
   - 对需求、用例、领域模型、类图、架构、数据库等已生成文档进行人工审查并定稿 baseline（R-REQ-05 / R-AGENT-03）。
   - 统一 Title 概念、actor 命名、借出物语种维度三处跨文档口径（R-MOD-01 / R-MOD-03 / R-MOD-04）。

2. **编码前补全（缺失交付物）**：
   - 补全设计模型（09）、三个时序图（10/11/12）、API 规格（14）、测试计划（15）（R-MOD-05 / R-ARC-01）。

3. **编码中重点关注**：
   - SQLite 外键 PRAGMA 开启、金额 Decimal 精度、预约过期自动取消定时机制、数据级鉴权（R-DB-04 / R-DB-05 / R-ARC-04 / R-PERM-04）。

4. **过程管控**：
   - 建立 Agent 生成内容的逐份人工审查与 baseline 台账，避免批量生成引入新偏差（R-AGENT-01 / R-AGENT-02 / R-AGENT-04 / R-AGENT-05）。