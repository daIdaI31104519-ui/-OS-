# DATA_CLASSIFICATION_DICTIONARY.md

# 市場理解OS Data Classification Dictionary

## 0. 文書情報

- 文書種別: DICTIONARY / DATA CLASSIFICATION SEMANTIC SOURCE OF TRUTH CANDIDATE
- 状態: CANONICAL DICTIONARY CANDIDATE
- FIX: FIX-018C / Data Classification Part
- 上位Security原則:
  - `01_DICTIONARY/SECURITY_DICTIONARY.md`
  - `01_DICTIONARY/CREDENTIAL_DICTIONARY.md`
- Object定義正本:
  - `01_DICTIONARY/OBJECT_DICTIONARY.md`
- 関連:
  - `01_DICTIONARY/ROLE_DICTIONARY.md`
  - `01_DICTIONARY/STATE_DICTIONARY.md`
  - `00_GOVERNANCE/GIT_RULES.md`
  - `00_GOVERNANCE/DESIGN_CHANGE_RULES.md`
- 目的: 市場理解OS内のData / Objectを人間の感覚だけで分類せず、同じ規則で `PUBLIC / INTERNAL / SENSITIVE / SECRET` へ分類し、Access / Export / External AI / Backup / Retention / Deletion / Encryption / Auditの共通Policy入力へ接続する

---

# 1. FIX-018C Data Classificationの目的

市場理解OSでは、Object名だけを見て「これはSensitiveっぽい」と判断しない。

正式な分類経路:

```text
Object Generated / Received
↓
Classification Strategy
↓
Base Classification
↓
Source / Provider Constraint
↓
Referenced Object Classification
↓
Sensitive Field Classification
↓
Runtime / Environment Context
↓
Escalation Rules
↓
Effective Classification
↓
Policy Routing
```

Classificationはラベルではなく、後続Security / Data Governance Policyの入力である。

---

# 2. 最重要分離原則

```text
Data Classification
≠ Retention Class
≠ Storage Lifecycle
≠ Knowledge Aging / Health
≠ Production Promotion Stage
≠ RiskState
```

意味:

```text
Classification
= 漏洩・公開・持出し時の取扱い厳格度

Retention
= どれくらい保持する必要があるか

Storage Lifecycle
= HOT / WARM / COLD / ARCHIVED等の配置状態

Knowledge Health
= Knowledgeが現在どの程度新鮮・健全か

Production Promotion
= Knowledgeを本番でどこまで利用可能か

RiskState
= 今この瞬間OSがどこまでRiskを取れるか
```

したがって次は成立する。

```text
TradeResult
Classification = SENSITIVE
Retention = IMMUTABLE_LONG_TERM candidate
Storage = ARCHIVED
```

```text
ResearchResult
Classification = INTERNAL
Knowledge Health = STALE
Storage = HOT
```

---

# 3. Canonical Classification Levels

Classification順序を正式に次とする。

```text
PUBLIC
<
INTERNAL
<
SENSITIVE
<
SECRET
```

`MAX()` はこの順序上で最も厳しいClassificationを返す。

---

# 4. PUBLIC

## Meaning

市場理解OS外へ出しても、機密・Account・Security・内部研究資産・Live Trading・Provider契約上の重大問題を原則生じないData。

## Important Rule

```text
Internetで閲覧可能
≠ 自動的にPUBLIC
```

Provider License / Redistribution Rule / Private Source Context等により `INTERNAL / SENSITIVE` へ上げられる。

候補例:

```text
公開再配布可能なBTC OHLCV
公開再配布可能なTicker
公開済み一般Market Reference Data
```

---

# 5. INTERNAL

## Meaning

市場理解OS内部で生成・加工・研究された知的資産、設計資産、Knowledge、計算結果等。

漏洩は望ましくないが、値そのものが認証・秘密鍵・本番Account操作権限を直接与えないData。

代表:

```text
Observation
Feature
MarketContext
Evidence
CausalHypothesis
MarketDNA
ResearchPlan
ResearchResult
Edge
FailureBoundary
Constraint
NegativeKnowledge
```

---

# 6. SENSITIVE

## Meaning

漏洩・不正持出しにより、Production、Account、Human Identity、Security、Live Position / Order、Operational State等へ具体的影響を与え得るData。

代表:

```text
ApplicableHypothesisSet
TradeThesis
SignalDecision
DefenseDecision
RiskState
EntryThesis
OrderIntent
ExecutionRecord
ProductionEvidence
Live TradeResult
SecurityPrincipal
CredentialProfile
Audit / Security Incident Data
```

---

# 7. SECRET

## Meaning

値そのものがAuthentication / Signature / Credential利用 / Secret Materialとして利用可能な秘密情報。

代表:

```text
API_KEY
API_SECRET
PASSWORD
PRIVATE_KEY
BOT_TOKEN
BEARER_TOKEN
RECOVERY_CODE
```

正式原則:

```text
CredentialProfile = SENSITIVE
Secret Value = SECRET
```

Secret ValueをCredentialProfile / Git / normal DB / Log / AI Inputへ保存しない。

---

# 8. Base Classification ≠ Effective Classification

## Base Classification

Object Typeが通常Contextで最低限どのClassificationとして扱われるか。

## Effective Classification

そのData InstanceがSource / Field / Reference / Runtime Context等を含めて実際にどのClassificationとして扱われるか。

正式原則:

```text
Base Classification
= Object Typeの通常最低基準

Effective Classification
= 今このData Instanceへ実際に適用するClassification
```

---

# 9. Effective Classification Formula

概念式:

```text
Effective Classification
=
MAX(
  Base Classification,
  Source / Provider Requirement,
  Referenced Object Requirement,
  Sensitive Field Requirement,
  Runtime / Environment Context Requirement,
  Escalation Rule Target
)
```

一つでもより厳しい要件があれば、低いClassificationへ自動的に下げない。

---

# 10. Canonical Classification Strategies

新規Objectを毎回ゼロから分類せず、原則次の6 Strategyのいずれかを選ぶ。

```text
FIXED
SOURCE_DERIVED
INHERIT_MAX
TARGET_INHERIT
CONTEXT_ESCALATE
FIXED_MINIMUM
```

Strategyが不明なObjectはDefault Allowせず、Classification Review対象とする。

---

# 11. Strategy: FIXED

## Meaning

Objectの意味上、通常Classificationが固定され、Source / Runtime Contextで容易に変化しない。

例:

```text
FormulaDefinition → INTERNAL
ResearchPlan → INTERNAL
CausalHypothesis → INTERNAL
Secret Value → SECRET
```

ただしUnexpected Secret Material等のSecurity Violationは別途Incident処理する。

---

# 12. Strategy: SOURCE_DERIVED

## Meaning

Source / Provider / License / Private-vs-Public性質がClassificationを決めるObject。

代表:

```text
RawData
```

例:

```text
Public redistributable market feed
→ PUBLIC

Paid proprietary provider feed
→ INTERNAL

Private account / order stream
→ SENSITIVE
```

`RawData = PUBLIC` と固定しない。

---

# 13. Strategy: INHERIT_MAX

## Meaning

複数Source / Input / Referenceから生成され、最も厳しいClassificationを最低限継承すべきObject。

例:

```text
MeasurementResult
Feature
EvidencePackage
Derived Analytics
```

正式原則:

```text
Derived Object
>= Highest Input Classification
```

---

# 14. Strategy: TARGET_INHERIT

## Meaning

Target / SubjectのClassificationを最低限継承すべきGovernance / Transition / Audit系Object。

代表:

```text
ApprovalDecision
StateTransitionEvent
```

例:

```text
Hypothesis ApprovalDecision
→ INTERNAL

Credential Activation ApprovalDecision
→ SENSITIVE

Risk Recovery StateTransitionEvent
→ SENSITIVE
```

概念式:

```text
Effective
=
MAX(Base, Target Classification, Referenced Sensitive Context)
```

---

# 15. Strategy: CONTEXT_ESCALATE

## Meaning

通常はINTERNAL等だが、Live Account / Position / Order / Human / Security Contextを組み合わせた時に昇格するObject。

代表:

```text
MarketDNA
ResearchResult
ResearchTrial
MarketContext
Post-Trade Analysis
Failure
```

例:

```text
MarketDNA = INTERNAL

MarketDNA + Current Live Position
→ SENSITIVE
```

---

# 16. Strategy: FIXED_MINIMUM

## Meaning

Object Type自体が最低Classificationを持ち、どのContextでもそれより下へ自動降格させない。

代表:

```text
TradeThesis
SignalDecision
DefenseDecision
RiskState
EntryThesis
OrderIntent
ExecutionRecord
ProductionEvidence
Live Position
```

例:

```text
TradeThesis
Base minimum = SENSITIVE
```

Public Market Dataだけから作られていてもPUBLIC / INTERNALへ自動降格しない。

---

# 17. More Restrictive / Less Restrictive Asymmetry

Classification変更方向を非対称に扱う。

## More Restrictive

```text
PUBLIC → INTERNAL
INTERNAL → SENSITIVE
SENSITIVE → SECRET
```

Policy-driven / automatic escalationを許可可能。

## Less Restrictive

```text
SECRET → SENSITIVE
SENSITIVE → INTERNAL
INTERNAL → PUBLIC
```

原則Explicit Reviewを要求する。

正式原則:

```text
More Restrictive
→ automatic / policy-driven possible

Less Restrictive
→ explicit review required
```

---

# 18. SECRET Declassification Rule

Secret ValueそのものをSanitizeして通常Dataへ戻す発想を採用しない。

```text
SECRET Material
→ Secret Store / Dedicated Secret Domain
```

通常Objectへ出すのはReferenceのみ。

```text
credential_id
secret_ref
secret_generation_ref
```

はSecret Valueではないが、Credential MetadataとしてSENSITIVE扱いになり得る。

原則:

```text
SECRET → lower classification
= FORBIDDEN by default
```

---

# 19. Expected SECRET ≠ Unexpected SECRET Leakage

Secret Materialには正常系と異常系を分ける。

## Expected SECRET

専用Secret Store / Secret Handling Domain上に存在する正規Secret Material。

## Unexpected SECRET

次へSecret Materialが混入した状態。

```text
ExecutionRecord
Diagnostics
Log
ResearchResult
ApprovalDecision
StateTransitionEvent
AI Input
Git
Backup Manifest
```

正式処理:

```text
Unexpected Secret Material
↓
Effective Classification = SECRET
+
Security Incident Trigger
+
Exposure Propagation Check
+
Log / Export / AI / Backup routing stop as applicable
```

単なるClassification昇格だけで終了しない。

---

# 20. Sanitization

正式原則:

```text
Sanitization
= Derived View Generation

Sanitization
≠ Source Object Mutation
```

元ObjectのClassificationを直接書き換えてResearch / AIへ渡さない。

例:

```text
ExecutionRecord (SENSITIVE)
↓
Remove / pseudonymize account_ref
Remove order_id where unnecessary
Remove credential metadata
Remove exact exposure if policy requires
↓
SanitizedExecutionView (INTERNAL candidate)
```

Source ObjectはSENSITIVEのまま保持する。

---

# 21. Sanitization Policy Values

Canonical candidate:

```text
ALLOWED
CONDITIONAL
FORBIDDEN
```

Meaning:

```text
ALLOWED
= 定義済みSanitization PolicyでDerived View作成可能

CONDITIONAL
= Context / Destination / Field除去条件を満たす場合のみ可能

FORBIDDEN
= 通常Derived ViewへのDeclassification目的のSanitizationを認めない
```

Secret Valueは原則 `FORBIDDEN`。

---

# 22. Declassification Policy

Canonical candidate:

```text
NONE_NEEDED
EXPLICIT_REVIEW
FORBIDDEN
```

`EXPLICIT_REVIEW`は元ObjectのClassificationそのものを低下させる場合に要求する。

Sanitized Derived View作成と元Object Declassificationを混同しない。

---

# 23. Classification Reason Codes

初期Reason Code候補:

```text
PUBLIC_SOURCE
PROVIDER_RESTRICTED
EXTERNAL_LICENSE_RESTRICTED
PROPRIETARY_RESEARCH
DERIVED_FROM_RESTRICTED_SOURCE
ACCOUNT_LINKED
HUMAN_IDENTIFIABLE
LIVE_POSITION
LIVE_ORDER
LIVE_EXECUTION
LIVE_TRADING_DECISION
RISK_CONTROL
SECURITY_CONTROL
CREDENTIAL_METADATA
SECRET_MATERIAL
GOVERNANCE_TARGET_SENSITIVE
INCIDENT_SECURITY_RELEVANT
```

Reason Codeは「なぜこのClassificationか」を追跡するために使い、単一Mystery Scoreへ潰さない。

---

# 24. Classification Policy Version

Classification RuleはVersion追跡可能にする。

候補Metadata:

```yaml
base_classification:
effective_classification:
classification_strategy:
classification_policy_version:
classification_reason_codes: []
source_classification_ref:
sensitivity_inherited_from_refs: []
sanitization_policy_ref:
```

これらを全Object Tableへ物理Columnとして複製するか、Envelope / Policy Evaluation Record / Metadataとして持つかは後続Data Contract / DB設計で決定する。

---

# 25. Object Family Default Classification

現行 `OBJECT_DICTIONARY.md` のFamily構成を壊さず、Family Defaultを次とする。

| Object Family | Default Strategy | Default Base |
|---|---|---|
| COMMON / CONTROL | CONTEXT_ESCALATE | INTERNAL |
| STATE / TRANSITION | TARGET_INHERIT | INTERNAL |
| GOVERNANCE / APPROVAL | TARGET_INHERIT | INTERNAL |
| OBSERVATION / DATA | INHERIT_MAX / SOURCE_DERIVED | INTERNAL candidate |
| MEASUREMENT / FEATURE | INHERIT_MAX | INTERNAL |
| MARKET UNDERSTANDING | CONTEXT_ESCALATE | INTERNAL |
| CAUSAL / DNA | CONTEXT_ESCALATE | INTERNAL |
| RESEARCH | CONTEXT_ESCALATE | INTERNAL |
| KNOWLEDGE | CONTEXT_ESCALATE | INTERNAL |
| PRODUCTION / TRADING | FIXED_MINIMUM / CONTEXT_ESCALATE | SENSITIVE candidate |
| POST-TRADE / FEEDBACK | CONTEXT_ESCALATE | INTERNAL |
| PLATFORM / OPERATIONS | CONTEXT_ESCALATE | INTERNAL / SENSITIVE depending security context |
| SECURITY METADATA | FIXED_MINIMUM | SENSITIVE |
| SECRET MATERIAL | FIXED | SECRET |

Family Defaultは個別Object Ruleがない場合のFallback候補であり、個別Object定義を上書きしない。

---

# 26. Canonical Matrix Columns

主要Object Classification Matrixは次の7列を基準とする。

| Object | Strategy | Base | Escalation | Sanitization | Declassification | Reason |
|---|---|---|---|---|---|---|

意味:

```text
Object
= Object Type

Strategy
= Classification決定方式

Base
= 通常最低Classification

Escalation
= 何を含むと上がるか

Sanitization
= Derived View作成可否

Declassification
= 元Object Classification低下Policy

Reason
= Default Classificationの主Reason
```

---

# 27. COMMON / CONTROL Matrix

| Object | Strategy | Base | Escalation | Sanitization | Declassification | Reason |
|---|---|---|---|---|---|---|
| MarketProfile | CONTEXT_ESCALATE | INTERNAL | Account / Execution / Security configuration → SENSITIVE | ALLOWED | EXPLICIT_REVIEW | internal configuration |
| SourceMetadata | CONTEXT_ESCALATE | INTERNAL | Private endpoint / account / auth-related metadata → SENSITIVE | ALLOWED | EXPLICIT_REVIEW | provenance |
| QualityProfile | INHERIT_MAX | INTERNAL | Sensitive source context → SENSITIVE | ALLOWED | EXPLICIT_REVIEW | internal quality |
| Diagnostics | CONTEXT_ESCALATE | INTERNAL | Account / Security / Incident context → SENSITIVE; Secret material → SECRET + Incident | CONDITIONAL | EXPLICIT_REVIEW | operational diagnostics |
| RuntimeCommand | FIXED_MINIMUM | SENSITIVE | Security / credential / recovery command remains SENSITIVE | CONDITIONAL | EXPLICIT_REVIEW | control intent |
| GlobalRiskLimit | FIXED_MINIMUM | SENSITIVE | no automatic lowering | CONDITIONAL | EXPLICIT_REVIEW | live risk control |

---

# 28. STATE / GOVERNANCE Matrix

| Object | Strategy | Base | Escalation | Sanitization | Declassification | Reason |
|---|---|---|---|---|---|---|
| StateTransitionEvent | TARGET_INHERIT | INTERNAL | Target / referenced context classification | CONDITIONAL | EXPLICIT_REVIEW | immutable state history |
| ApprovalDecision | TARGET_INHERIT | INTERNAL | Credential / Risk / Production recovery / sensitive target → SENSITIVE | CONDITIONAL | EXPLICIT_REVIEW | governance decision |

Rule:

```text
Hypothesis State Transition
→ INTERNAL candidate

Credential COMPROMISED Transition
→ SENSITIVE

Risk Recovery Approval
→ SENSITIVE
```

---

# 29. OBSERVATION / DATA Matrix

| Object | Strategy | Base | Escalation | Sanitization | Declassification | Reason |
|---|---|---|---|---|---|---|
| RawData | SOURCE_DERIVED | SOURCE_DERIVED | Provider restriction → INTERNAL; private account/order source → SENSITIVE | CONDITIONAL | EXPLICIT_REVIEW | primary evidence |
| Observation | INHERIT_MAX | INTERNAL | sensitive source / account-linked input → SENSITIVE | ALLOWED | EXPLICIT_REVIEW | normalized observation |
| TimeSeriesMeasurement | INHERIT_MAX | INTERNAL | sensitive input → SENSITIVE | ALLOWED | EXPLICIT_REVIEW | derived time-series measurement |
| retired NormalizedObservation alias | RETIRED | N/A | N/A | N/A | N/A | do not create new object |

`RawData`のみ単純Base固定値ではなくSource Policyを強く継承する。

---

# 30. MEASUREMENT / FEATURE Matrix

| Object | Strategy | Base | Escalation | Sanitization | Declassification | Reason |
|---|---|---|---|---|---|---|
| FormulaDefinition | FIXED | INTERNAL | Secret material is violation, not normal escalation | ALLOWED | EXPLICIT_REVIEW | proprietary calculation definition |
| MeasurementResult | INHERIT_MAX | INTERNAL | sensitive input → SENSITIVE | ALLOWED | EXPLICIT_REVIEW | derived measurement |
| Feature | INHERIT_MAX | INTERNAL | sensitive input → SENSITIVE | ALLOWED | EXPLICIT_REVIEW | internal feature asset |
| FeaturePriorityProfile | CONTEXT_ESCALATE | INTERNAL | live position/account context → SENSITIVE | ALLOWED | EXPLICIT_REVIEW | current analysis priority |

---

# 31. MARKET UNDERSTANDING Matrix

| Object | Strategy | Base | Escalation | Sanitization | Declassification | Reason |
|---|---|---|---|---|---|---|
| MarketContext | CONTEXT_ESCALATE | INTERNAL | live account / position / order context → SENSITIVE | ALLOWED | EXPLICIT_REVIEW | proprietary market interpretation |
| Evidence | INHERIT_MAX | INTERNAL | sensitive evidence source → SENSITIVE | CONDITIONAL | EXPLICIT_REVIEW | research evidence |
| Contradiction | INHERIT_MAX | INTERNAL | sensitive target/source → SENSITIVE | ALLOWED | EXPLICIT_REVIEW | internal contradiction knowledge |
| UnexplainedEvent | CONTEXT_ESCALATE | INTERNAL | private / sensitive source → SENSITIVE | ALLOWED | EXPLICIT_REVIEW | internal research asset |

Public RawDataから生成されても、OS独自解釈は原則PUBLICへ自動継承しない。

```text
Public RawData
↓
Market Intelligence proprietary interpretation
↓
INTERNAL
```

---

# 32. CAUSAL / DNA Matrix

| Object | Strategy | Base | Escalation | Sanitization | Declassification | Reason |
|---|---|---|---|---|---|---|
| Cause Candidate | CONTEXT_ESCALATE | INTERNAL | live/account sensitive context → SENSITIVE | ALLOWED | EXPLICIT_REVIEW | proprietary causal research |
| EffectDefinition | FIXED | INTERNAL | normal context no escalation | ALLOWED | EXPLICIT_REVIEW | research definition |
| CausalHypothesis | FIXED | INTERNAL | linked live account material → SENSITIVE derived context only | ALLOWED | EXPLICIT_REVIEW | proprietary hypothesis |
| HypothesisAssessmentProfile | INHERIT_MAX | INTERNAL | sensitive referenced evidence → SENSITIVE | ALLOWED | EXPLICIT_REVIEW | research assessment |
| MarketDNA | CONTEXT_ESCALATE | INTERNAL | current live position/account/exposure joined → SENSITIVE | ALLOWED | EXPLICIT_REVIEW | proprietary market representation |
| RegimeProfile | CONTEXT_ESCALATE | INTERNAL | live account/risk-specific context → SENSITIVE | ALLOWED | EXPLICIT_REVIEW | derived regime knowledge |

Rule:

```text
MarketDNA = INTERNAL

MarketDNA + Current Position / Exposure
→ separate SENSITIVE derived view
```

Source MarketDNAそのものをSENSITIVEへ無言で書き換えない。

---

# 33. RESEARCH Matrix

| Object | Strategy | Base | Escalation | Sanitization | Declassification | Reason |
|---|---|---|---|---|---|---|
| ResearchCandidate | CONTEXT_ESCALATE | INTERNAL | security/live account incident context → SENSITIVE | ALLOWED | EXPLICIT_REVIEW | research intake |
| ResearchPlan | FIXED | INTERNAL | normal context no escalation | ALLOWED | EXPLICIT_REVIEW | proprietary research design |
| ResearchTrial | CONTEXT_ESCALATE | INTERNAL | private production dataset / live account context → SENSITIVE | ALLOWED | EXPLICIT_REVIEW | experiment execution |
| ResearchResult | CONTEXT_ESCALATE | INTERNAL | live account/order/exposure evidence → SENSITIVE | ALLOWED | EXPLICIT_REVIEW | research result |
| EvidencePackage | INHERIT_MAX | INTERNAL | highest included evidence/target classification | CONDITIONAL | EXPLICIT_REVIEW | evidence bundle |
| Edge | FIXED | INTERNAL | sensitive deployment context belongs to derived Production object | ALLOWED | EXPLICIT_REVIEW | proprietary edge knowledge |
| ResearchBudget | CONTEXT_ESCALATE | INTERNAL | billing/account-identifying context → SENSITIVE | ALLOWED | EXPLICIT_REVIEW | research resource governance |

Research ObjectをProductionへ渡しただけで自動的にSENSITIVE化するのではなく、Live Account / Current Production Contextを実際に含む場合にEffective Classificationを上げる。

---

# 34. KNOWLEDGE Matrix

| Object | Strategy | Base | Escalation | Sanitization | Declassification | Reason |
|---|---|---|---|---|---|---|
| MarketCase | CONTEXT_ESCALATE | INTERNAL | live account-linked case → SENSITIVE | ALLOWED | EXPLICIT_REVIEW | reusable market knowledge |
| FeatureKnowledge | FIXED | INTERNAL | sensitive deployment detail belongs to derived view | ALLOWED | EXPLICIT_REVIEW | proprietary knowledge |
| FormulaKnowledge | FIXED | INTERNAL | sensitive deployment detail belongs to derived view | ALLOWED | EXPLICIT_REVIEW | proprietary knowledge |
| Failure | CONTEXT_ESCALATE | INTERNAL | account/security/execution incident → SENSITIVE; secret leakage → SECRET + Incident | CONDITIONAL | EXPLICIT_REVIEW | reusable failure knowledge |
| StressResult | INHERIT_MAX | INTERNAL | sensitive live input → SENSITIVE | ALLOWED | EXPLICIT_REVIEW | research stress evidence |
| FailureBoundary | FIXED | INTERNAL | sensitive live-only detail → SENSITIVE derived view | ALLOWED | EXPLICIT_REVIEW | critical safety knowledge |
| Constraint | FIXED | INTERNAL | exact live account exposure constraints → SENSITIVE derived context | ALLOWED | EXPLICIT_REVIEW | production safety knowledge |
| NegativeKnowledge | FIXED | INTERNAL | normal context no escalation | ALLOWED | EXPLICIT_REVIEW | non-effect knowledge |
| KnowledgeLifecycleProfile | TARGET_INHERIT | INTERNAL | target SENSITIVE → SENSITIVE where profile reveals sensitive target state | CONDITIONAL | EXPLICIT_REVIEW | knowledge health projection |

Knowledgeの研究価値とSensitivityを混同しない。

```text
SENSITIVE
≠ short retention
```

---

# 35. PRODUCTION Boundary

市場理解OSではProductionのClassification境界を次で固定する。

```text
Research / Knowledge
↓
HypothesisPoolEntry
= INTERNAL candidate

ApplicableHypothesisSet
↓
TradeThesis
↓
SignalDecision
↓
DefenseDecision
↓
EntryThesis
↓
OrderIntent
↓
Execution
= SENSITIVE minimum
```

意味:

```text
Knowledge
→ 何が成立し得るか

ApplicableHypothesisSet以降
→ 今この市場で本番利用しようとしている情報
```

この境界をProduction Sensitivity Boundaryとする。

---

# 36. PRODUCTION / TRADING Matrix

| Object | Strategy | Base | Escalation | Sanitization | Declassification | Reason |
|---|---|---|---|---|---|---|
| HypothesisPoolEntry | CONTEXT_ESCALATE | INTERNAL | exact live production administration details → SENSITIVE | ALLOWED | EXPLICIT_REVIEW | production eligibility registry |
| ApplicableHypothesisSet | FIXED_MINIMUM | SENSITIVE | Secret material → SECRET + Incident | CONDITIONAL | EXPLICIT_REVIEW | current production applicability |
| TradeThesis | FIXED_MINIMUM | SENSITIVE | Secret material → SECRET + Incident | CONDITIONAL | EXPLICIT_REVIEW | live trading thesis |
| AIReviewResult | INHERIT_MAX | INTERNAL | SENSITIVE review input/content → SENSITIVE | ALLOWED | EXPLICIT_REVIEW | advisory review |
| SignalDecision | FIXED_MINIMUM | SENSITIVE | Secret material → SECRET + Incident | CONDITIONAL | EXPLICIT_REVIEW | live risk-taking decision |
| DefenseDecision | FIXED_MINIMUM | SENSITIVE | Secret material → SECRET + Incident | CONDITIONAL | EXPLICIT_REVIEW | safety gate decision |
| RiskState | FIXED_MINIMUM | SENSITIVE | no automatic lowering | CONDITIONAL | EXPLICIT_REVIEW | current risk permission |
| EntryThesis | FIXED_MINIMUM | SENSITIVE | Secret material → SECRET + Incident | CONDITIONAL | EXPLICIT_REVIEW | pre-order production snapshot |
| OrderIntent | FIXED_MINIMUM | SENSITIVE | account/order routing detail remains SENSITIVE; Secret material → Incident | CONDITIONAL | EXPLICIT_REVIEW | order intent |
| ExecutionRecord | FIXED_MINIMUM | SENSITIVE | Secret material → SECRET + Incident | ALLOWED | EXPLICIT_REVIEW | account/order execution evidence |
| PositionThesisState | FIXED_MINIMUM | SENSITIVE | no automatic lowering | CONDITIONAL | EXPLICIT_REVIEW | live position thesis state |
| ExitDecision | FIXED_MINIMUM | SENSITIVE | no automatic lowering | CONDITIONAL | EXPLICIT_REVIEW | live exit decision |
| ProductionEvidence | FIXED_MINIMUM | SENSITIVE | Secret material → SECRET + Incident | ALLOWED | EXPLICIT_REVIEW | live execution evidence |
| TradeResult | CONTEXT_ESCALATE | INTERNAL | LIVE / account-linked result → SENSITIVE | ALLOWED | EXPLICIT_REVIEW | trade outcome |

`TradeResult`はResearch / SimulationとLiveを区別する。

```text
Simulation / Research TradeResult
→ INTERNAL candidate

Live account-linked TradeResult
→ SENSITIVE
```

---

# 37. Position Future Rule

FIX-019で正式Position Objectを追加する場合、初期Classification Ruleは次をDefaultとする。

```text
Position
Strategy = FIXED_MINIMUM
Base = SENSITIVE
```

理由:

```text
Current exposure
Entry / Fill linkage
Live account state
```

を表すため。

正式Object採用時にこのDictionaryへ個別行を追加する。

---

# 38. POST-TRADE / FEEDBACK Matrix

| Object | Strategy | Base | Escalation | Sanitization | Declassification | Reason |
|---|---|---|---|---|---|---|
| OutcomeAnalysisResult | CONTEXT_ESCALATE | INTERNAL | live TradeResult/account context → SENSITIVE | ALLOWED | EXPLICIT_REVIEW | post-trade research |
| TradeThesisEvaluation | CONTEXT_ESCALATE | INTERNAL | live thesis/account context → SENSITIVE | ALLOWED | EXPLICIT_REVIEW | thesis evaluation |
| HypothesisAttribution | CONTEXT_ESCALATE | INTERNAL | live account-specific attribution → SENSITIVE | ALLOWED | EXPLICIT_REVIEW | research attribution |
| DefenseEvaluation | CONTEXT_ESCALATE | INTERNAL | live safety/account details → SENSITIVE | ALLOWED | EXPLICIT_REVIEW | defense research |
| SupervisorEvaluation | CONTEXT_ESCALATE | INTERNAL | live position details → SENSITIVE | ALLOWED | EXPLICIT_REVIEW | supervisor research |
| DemoLiveDivergence | CONTEXT_ESCALATE | INTERNAL | live account/execution details → SENSITIVE | ALLOWED | EXPLICIT_REVIEW | channel divergence research |

Post-Trade Researchは元Live Dataを保持する場合SENSITIVEだが、Sanitized Derived Research ViewをINTERNALとして生成可能にする。

---

# 39. SECURITY / CREDENTIAL Matrix

| Object / Data | Strategy | Base | Escalation | Sanitization | Declassification | Reason |
|---|---|---|---|---|---|---|
| SecurityPrincipal | FIXED_MINIMUM | SENSITIVE | human/account/security metadata remains SENSITIVE | CONDITIONAL | EXPLICIT_REVIEW | identity/security metadata |
| CredentialProfile | FIXED_MINIMUM | SENSITIVE | Secret Value混入 → SECRET + Incident | LIMITED / CONDITIONAL | EXPLICIT_REVIEW | credential metadata |
| SecretReference | FIXED_MINIMUM | SENSITIVE | direct Secret Material must remain separate | CONDITIONAL | EXPLICIT_REVIEW | secret locator metadata |
| Secret Value | FIXED | SECRET | N/A | FORBIDDEN | FORBIDDEN | secret material |

CredentialProfileへSecret Valueが入った場合、それを正常なCredentialProfile Schemaとして受け入れない。

```text
CredentialProfile + Secret Value
→ Security Contract Violation
→ SECRET
→ Incident
```

---

# 40. Join / Composition Rule

単体Objectが低いClassificationでも、Join後に高くなる場合がある。

例:

```text
BTC Price = PUBLIC
+
Current Position = SENSITIVE
+
Account Exposure = SENSITIVE
↓
Live Position Analysis = SENSITIVE
```

正式原則:

```text
Join Result Classification
>= MAX(all joined material)
```

Sanitization Policyで別Derived Viewを生成しない限り、低いClassificationへ戻さない。

---

# 41. Field-level Classification

Object全体のClassificationだけでなく、必要に応じてField単位Classificationを定義可能にする。

例:

```text
ExecutionRecord
Object Effective = SENSITIVE

exchange
→ INTERNAL candidate

market
→ INTERNAL candidate

executed_price
→ INTERNAL candidate

account_ref
→ SENSITIVE

credential_ref
→ SENSITIVE

order_id
→ SENSITIVE
```

Object全体をExport / Serializeする場合は最も厳しいFieldを考慮する。

---

# 42. External AI Routing Baseline

FIX-018A/Bと接続し、ClassificationをExternal AI Policy入力にする。

初期Baseline:

```text
PUBLIC
→ policy allows candidate

INTERNAL
→ provider / purpose / policy review required

SENSITIVE
→ raw send prohibited by default; sanitized derived view only

SECRET
→ NEVER
```

`AIReviewResult`を作るために元SENSITIVE Objectを無条件送信しない。

---

# 43. Git / Log / Normal DB Baseline

## PUBLIC / INTERNAL

通常Policy下で保存可能候補。

## SENSITIVE

Explicit access / encryption / audit policyへ送る。

## SECRET

通常:

```text
Git → DENY
normal DB → DENY
Log → DENY
AI Input → DENY
ordinary backup manifest → DENY
```

Secret Store / Dedicated Secret Governanceへ送る。

---

# 44. Access Policy Routing

ClassificationはAccess Controlの代わりではない。

```text
Classification
= Data sensitivity input

FIX-018A Permission
= Principalが何をしてよいか
```

実際のRead / Export等はIntersectionで判断する。

```text
Access Allowed
=
Principal Permission
∩ Classification Policy
∩ Resource Scope
∩ Environment
∩ Current Security Policy
```

---

# 45. Classification ≠ Permission

```text
SENSITIVE
```

だから全Principalが読めない、という意味ではない。

適切なPermission / Scope / PurposeがあるPrincipalだけが読める。

逆にPUBLICでも、Provider ContractやRuntime Policy上アクセス制限される場合がある。

---

# 46. Classification ≠ Retention

次の誤設計を禁止する。

```text
SENSITIVEだから早く削除
PUBLICだから永久保存
```

Retentionは別Policy軸で決定する。

例:

```text
ApprovalDecision
Effective Classification = INTERNAL / SENSITIVE target-derived
Retention candidate = IMMUTABLE_LONG_TERM
```

```text
TradeResult LIVE
Classification = SENSITIVE
Retention candidate = IMMUTABLE_LONG_TERM
```

詳細Retention ClassはFIX-018C後半で正式化する。

---

# 47. Classification ≠ Storage Lifecycle

```text
INTERNAL + ARCHIVED
SENSITIVE + HOT
SENSITIVE + ARCHIVED
PUBLIC + COLD
```

はすべて成立し得る。

`ARCHIVED`をSensitivity意味として使わない。

---

# 48. Classification Evaluation Failure

Classification Strategy / Source Classification / Reference Classification等が解決できず、安全なEffective Classificationを決定できない場合:

```text
UNKNOWNをPUBLICとして扱わない
```

初期Safe Rule:

```text
Classification unresolved
→ treat as at least INTERNAL
→ if live/account/security context exists, treat as at least SENSITIVE
→ route to Classification Review / Diagnostics
```

Unknownを便利なDefault PUBLICにしない。

---

# 49. New Object Rule

新しいPersistent Object / important Runtime Objectを追加する場合、Object定義と同時に最低限次を決める。

```text
classification_strategy
base_classification
escalation_rules
sanitization_policy
declassification_policy
reason_codes
```

分類が未定義のままProduction / External AI / Export経路へ入れない。

---

# 50. Retired / Legacy Object Rule

Legacy / RETIRED Objectを新規Classification Matrixへ独立Objectとして復活させない。

例:

```text
NormalizedObservation
→ RETIRED / MERGED INTO Observation
```

過去Recordは当時Schema / Classification Policy Versionとして読取可能性を維持する。

---

# 51. Classification Policy Migration

将来Classification Ruleが変更されても、過去Dataを無言で全件書き換えない。

必要に応じて:

```text
old classification policy version
new classification policy version
reclassification evaluation
reason
approval / governance reference if lowering restriction
```

を追跡する。

特にLess Restrictive ReclassificationはExplicit Review対象。

---

# 52. Canonical Invariants

```text
PUBLIC < INTERNAL < SENSITIVE < SECRET
```

```text
Effective Classification
>= Base Classification unless explicit governed declassification
```

```text
Derived Object
>= Highest required source/input/reference classification
```

```text
Sanitization
= Derived View
≠ Source Mutation
```

```text
Unexpected Secret Material
= SECRET + Security Incident
```

```text
More Restrictive
→ policy-driven possible

Less Restrictive
→ explicit review
```

```text
SECRET Declassification
→ forbidden by default
```

```text
Classification
≠ Retention
≠ Storage Lifecycle
≠ Knowledge Health
≠ Production Promotion
≠ RiskState
```

```text
HypothesisPoolEntry
= INTERNAL candidate

ApplicableHypothesisSet and downstream live production decision objects
= SENSITIVE minimum
```

---

# 53. FIX-018C Data Classification Output

このDictionaryにより、Data Classificationは次の形で後続設計へ渡される。

```text
Object / Data
↓
Classification Strategy
↓
Base
↓
Source / Reference / Field / Context
↓
Effective Classification
↓
Access Policy
↓
Encryption Policy
↓
Export Policy
↓
External AI Policy
↓
Backup Policy
↓
Retention Policy
↓
Deletion Policy
↓
Audit Policy
```

---

# 54. Out of Scope / Next FIX-018C Work

今回まだ固定しない。

```text
Retention Classの正式値
各Objectの保持年数
DELETABLE / CONDITIONALLY_DELETABLE / NON_DELETABLE
Deletion Request / Approval / Apply Authority詳細
DELETION_PENDING → DELETED条件
Physical Delete / Tombstone / Archive詳細
Backup encryption / restore詳細
External export approval詳細
DB Column / Index
Python Class / Evaluator implementation
```

次のFIX-018C作業は:

```text
Data Classification
↓
Retention Class
↓
Object Retention Matrix
↓
Deletion Eligibility
↓
Deletion Authority
↓
Storage Lifecycle接続
```

の順とする。
