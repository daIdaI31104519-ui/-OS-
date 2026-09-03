# STATE_DICTIONARY.md

# 市場理解OS State辞書・Lifecycle / State Machine 完全設計

## 0. 文書情報

- 文書種別: DICTIONARY / STATE SOURCE OF TRUTH
- 状態: CANONICAL DICTIONARY CANDIDATE
- 上位ルール:
  - `00_GOVERNANCE/GIT_RULES.md`
  - `00_GOVERNANCE/DESIGN_CHANGE_RULES.md`
- 関連辞書:
  - `01_DICTIONARY/ROLE_DICTIONARY.md`
  - `01_DICTIONARY/OBJECT_DICTIONARY.md`
- 対象: 市場理解OSでLifecycle・運用状態・安全状態・昇格状態を持つ主要Object / Role / Instance
- 目的: 同じ意味のStateを各`.md` / `.py`で別名・別定義に増殖させず、遷移条件・禁止遷移・復帰条件・履歴・Versionを一元管理する
- 最上位原則: **Stateは「現在どういう状態か」を表し、Decision / Role / Object / Evidence Strengthと混同しない**

---

# 1. State Dictionaryの目的

市場理解OSを何十年以上運用する場合、状態名が各Moduleで勝手に増えると、次の問題が起きる。

- `PAUSED` と `STOPPED` の意味がModuleごとに違う
- `SUPPORTED` と `APPROVED` が混同される
- `DEGRADED` と `ERROR` の境界が曖昧になる
- Research上の成功とProduction利用許可が同一Stateになる
- Risk低下とHypothesis劣化が同じStateとして扱われる
- 状態変更理由が保存されず、後からなぜ変化したか分からない
- 新Versionで過去Stateを再現できなくなる

そのためStateは、単なる文字列ではなくState Machineとして管理する。

本書で解決すること:

1. Stateの正式名称
2. Stateが何を意味するか
3. 誰が変更できるか
4. どこからどこへ遷移可能か
5. 何を満たせば遷移できるか
6. どの遷移を禁止するか
7. 復帰・再開条件
8. State変更履歴の保存
9. Hysteresis / Cooldown / Persistence
10. 数年・数十年後のVersion Migration

---

# 2. Stateと他概念の区別

## State

Object / Role / Instanceの現在のLifecycle・稼働・安全・昇格状況。

例:

```text
RESEARCHING
APPROVED
DEGRADED
RISK_REDUCED
THESIS_WEAKENING
```

## Decision

ある時点の判断結果。

例:

```text
BUY / SELL / NO_TRADE
ALLOW / REDUCE / BLOCK
TRADE / WAIT / REJECT
```

DecisionはStateではない。

## Evidence Strength / Assessment

支持・反証・不確実性等の評価値。

例:

```text
historical_support
contradiction_strength
uncertainty
```

評価値そのものをLifecycle Stateと混同しない。

## Experiment Mode

研究試行の方式。

例:

```text
HISTORICAL
REPLAY
DEMO_FORWARD
SHADOW
```

ModeはStateではない。

## Role

状態を生成・管理・監視する責任主体。

例:

```text
Research Orchestrator
Risk Governance
Runtime
Position Supervisor
```

---

# 3. State Machine共通Metadata

Stateを持つ主要Persistent Object / Instanceは、原則として次の情報を追跡可能にする。

```yaml
state_machine_id:
state_machine_type:
state_machine_version:
current_state:
previous_state:
state_entered_at:
state_changed_at:
trigger_refs: []
reason_codes: []
changed_by_role:
changed_by_actor:
review_due_at:
cooldown_until:
manual_override_ref:
```

全Objectへ物理的に同じFieldsを複製する必要はない。

Data Contract / DB Schemaで、

- Object本体に現在Stateを保持する
- StateTransitionEventを別保存する

等の方式を正式化する。

---

# 4. Current StateとState Historyを分離する

推奨:

```text
Current State
= 現在の高速参照用Projection

State Transition History
= 過去の変更事実を保存するImmutable履歴
```

例:

```text
H-101 current_state = ACTIVE

History:
DRAFT
→ RESEARCHING
→ SUPPORTED
→ APPROVED
→ ACTIVE
```

Current Stateだけを上書きして過去履歴を失わない。

State履歴は、将来次を説明できること。

```text
いつ変わった？
なぜ変わった？
何がTriggerだった？
どのEvidence / Error / Decisionが原因？
誰 / どのRoleが変更した？
どのState Machine Versionだった？
```

---

# 5. State変更の共通原則

## RULE-STATE-001: State変更には理由を持つ

重要Stateを理由なしで変更しない。

最低限:

```text
trigger
reason_code
changed_at
changed_by
```

を追跡可能にする。

## RULE-STATE-002: 不正な飛び越し遷移を禁止する

例:

```text
DRAFT → NORMAL_LIVE
```

のようなResearch / Validation / Approvalを飛び越した遷移は禁止。

## RULE-STATE-003: StateとScoreを分離する

単一Scoreが閾値を超えただけでStateを自動確定しない。

複数評価Dimension / Gate / Constraintを利用できるようにする。

## RULE-STATE-004: 復帰条件を持つ

停止・劣化Stateには、可能な限りRecovery / Reopen条件を定義する。

## RULE-STATE-005: UNKNOWNを許容する

不明な状態を無理にHealthy / Safe / Supportedへ分類しない。

## RULE-STATE-006: State Machine Versionを持つ

遷移ルール変更時に、過去State履歴を新Ruleで勝手に再解釈しない。

---

# 6. State分類

市場理解OSではStateを大きく次へ分ける。

```text
A. HYPOTHESIS / EDGE LIFECYCLE
B. RESEARCH LIFECYCLE
C. KNOWLEDGE LIFECYCLE
D. PRODUCTION PROMOTION / RISK
E. POSITION / TRADE THESIS
F. RUNTIME / HEALTH
G. INCIDENT / RECOVERY
H. SOURCE / DATA QUALITY
I. STORAGE / BACKUP / MIGRATION
J. DEPLOYMENT
```

Git / Designの `IDEA / PROPOSAL / CANONICAL` 等は `GIT_RULES.md` を正本とし、本書では重複定義しない。

---

# A. HYPOTHESIS / EDGE LIFECYCLE

# STATE-HYP-001: CausalHypothesis Lifecycle

対象:

- `CausalHypothesis`

正式候補State:

```text
DRAFT
RESEARCHING
SUPPORTED
WEAK
APPROVED
ACTIVE
AGING
SUSPENDED
RETIRED
REOPENED
CONFLICTED
```

## DRAFT

仮説として構造化されたが、正式Researchを十分実行していない。

禁止:

- Production利用
- APPROVED扱い

## RESEARCHING

Research Plan / Trialで検証中。

条件:

- Research Candidate / Planが存在
- 必要Dataが利用可能

## SUPPORTED

現在までのResearch Evidenceで一定の支持がある。

重要:

> SUPPORTED = Production利用許可ではない。

HistoricalだけでSUPPORTEDにするか、OOS等を必須にするかはResearch Contractで定義する。

## WEAK

一部支持は残るが、再現性・Applicability・Evidenceが弱い。

用途:

- 追加研究
- Risk縮小候補
- Production Poolから除外候補

## APPROVED

Knowledge Promotion / Approvalを通過し、Production利用候補として承認された。

重要:

> APPROVED = 常に現在市場で利用可能、ではない。

Applicability / Constraint / Risk Stateを別途確認する。

## ACTIVE

現在Production Pool等で利用可能な現役Knowledge。

## AGING

長期間再検証されていない、またはEvidenceの鮮度が低下している。

必ずしも誤りではない。

原則:

- Revalidation候補
- Risk Stage縮小候補

## SUSPENDED

現在利用停止中。

候補理由:

- Edge Health劣化
- Demo / Live divergence
- Critical Contradiction
- Structural Change
- Data Source問題

再開にはRevalidationを要求できる。

## RETIRED

現在の知識として利用終了。

原則Production利用禁止。

ただし履歴は削除しない。

## REOPENED

以前Weak / Suspended / Retired等だった仮説を、新Data・構造変化・新Formula等により再研究対象へ戻した状態。

通常:

```text
REOPENED → RESEARCHING
```

へ進む。

## CONFLICTED

支持Evidenceと反証Evidenceが強く競合し、単純なSUPPORTED / WEAKへ分類できない。

用途:

- Alternative Hypothesis比較
- Confounder再検証
- Production利用制限

---

# 7. Hypothesis基本遷移

```text
DRAFT
  ↓
RESEARCHING
  ├→ SUPPORTED
  ├→ WEAK
  └→ CONFLICTED

SUPPORTED
  ↓
APPROVED
  ↓
ACTIVE

ACTIVE
  ├→ AGING
  ├→ WEAK
  ├→ SUSPENDED
  └→ RETIRED

AGING
  ├→ RESEARCHING
  ├→ ACTIVE
  └→ SUSPENDED

WEAK / SUSPENDED / RETIRED
  ↓ 条件成立
REOPENED
  ↓
RESEARCHING
```

禁止例:

```text
DRAFT → ACTIVE
DRAFT → APPROVED
RESEARCHING → NORMAL_LIVE
RETIRED → ACTIVE  # Reopen / Research / Approvalを飛ばさない
```

---

# STATE-HYP-002: Edge Lifecycle

対象:

- `Edge`
- `FeatureKnowledge`
- `FormulaKnowledge` のProduction利用可能性

候補State:

```text
CANDIDATE
VALIDATING
SUPPORTED
APPROVED
ACTIVE
DEGRADED
SUSPENDED
RETIRED
REOPENED
```

Hypothesis Lifecycleと完全同一にせず、Edgeは再現可能な優位性の利用状態に重点を置く。

---

# B. RESEARCH LIFECYCLE

# STATE-RSCH-001: ResearchCandidate Lifecycle

対象:

- `ResearchCandidate`

正式候補State:

```text
NEW
SCREENING
QUEUED
RUNNING
PAUSED
COMPLETED
STOPPED
MERGED
REJECTED
EXPIRED
REOPENED
```

## NEW

Research入口へ生成された直後。

## SCREENING

Research Admissionで、重複・価値・Data Availability・Risk重要度・Resource Costを評価中。

## QUEUED

Admission通過後、Research Queueで実行待ち。

## RUNNING

Research Trialが実行中。

## PAUSED

一時停止。

候補:

- Data不足
- Resource Budget不足
- 他研究優先
- Source障害

## COMPLETED

事前定義したCompletion Criteriaを満たした。

`良い結果だった`という意味ではない。

## STOPPED

Early Stop / Futility / Risk / Resource理由で研究終了。

失敗結果・停止理由を保存する。

## MERGED

重複Candidateへ統合された。

元Candidateを削除せず、統合先Referenceを残す。

## REJECTED

研究する価値・実行可能性が不足していると判断。

## EXPIRED

一定期間実行されず、現在条件では研究価値が失効。

## REOPENED

新Data等により再度Research対象化。

---

# STATE-RSCH-002: ResearchPlan Lifecycle

対象:

- `ResearchPlan`

候補State:

```text
DRAFT
PRE_REGISTERED
READY
RUNNING
FROZEN
COMPLETED
SUPERSEDED
CANCELLED
```

## PRE_REGISTERED

評価指標・Entry / Continue / Stop / Completion条件を結果を見る前に固定。

## FROZEN

Demo Forward等でT0以降のRule変更を禁止するため固定された状態。

変更する場合、同Planを上書きせず新Versionを作る。

---

# STATE-RSCH-003: ResearchTrial Lifecycle

対象:

- `ResearchTrial`

候補State:

```text
PENDING
INITIALIZING
RUNNING
PAUSED
COMPLETED
EARLY_STOPPED
FAILED
INVALIDATED
CANCELLED
```

## INVALIDATED

Trial自体は実行されたが、Leakage / Data corruption / Rule violation / Future information contamination等によりEvidenceとして利用不可。

Trialを削除せず、無効理由を保存する。

---

# 8. Research Stop Rule共通原則

Research Stateを無限に `RUNNING` のままにしない。

最低限、Research Planで次を定義可能にする。

```text
ENTRY CRITERIA
CONTINUE CRITERIA
EARLY STOP CRITERIA
COMPLETION CRITERIA
REOPEN CRITERIA
```

候補Trigger:

- Unique Market Event数
- OOS deterioration
- Repeated contradiction
- No improvement / futility
- Resource budget exceeded
- Duplicate discovery
- Data source unavailable
- Structural change

---

# C. KNOWLEDGE LIFECYCLE

# STATE-KNW-001: Knowledge Aging State

対象:

- `KnowledgeLifecycleProfile`
- Hypothesis / Edge / FeatureKnowledge / FormulaKnowledge / Constraint等

候補State:

```text
FRESH
CURRENT
AGING
STALE
DEGRADED
SUSPENDED
ARCHIVED
```

## FRESH

最近生成・検証された。

## CURRENT

現在の利用条件で十分再検証されている。

## AGING

再検証期限が近い、またはEvidence鮮度低下。

## STALE

長期間再検証されていない。

`FALSE`とは限らないが、ProductionでのTrustを制限できる。

## DEGRADED

最近のDemo / Live / OOS等で性能劣化が観測された。

## SUSPENDED

Knowledge利用を一時停止。

## ARCHIVED

現役利用対象外だが歴史・研究資産として保存。

---

# STATE-KNW-002: Constraint Lifecycle

対象:

- `Constraint`

候補State:

```text
DRAFT
VALIDATING
APPROVED
ACTIVE
REVIEW_DUE
SUSPENDED
SUPERSEDED
RETIRED
```

ConstraintはProduction Safetyへ直接影響するため、未承認Stateを本番Gateへ自動適用しない。

---

# D. PRODUCTION PROMOTION / RISK

# STATE-PRD-001: Production Promotion Stage

対象:

- Hypothesis / Edge / Strategy / Rule等の本番利用段階
- `HypothesisPoolEntry.allowed_risk_stage`

正式候補Stage:

```text
RESEARCH_ONLY
SHADOW
DEMO_FORWARD
MICRO_LIVE
LIMITED_LIVE
NORMAL_LIVE
PAUSED
```

## RESEARCH_ONLY

本番利用禁止。

## SHADOW

実注文せず、Production条件下で仮想追跡。

## DEMO_FORWARD

T0以降の未来Dataで、Liveと共通ロジックを使い仮想取引検証。

## MICRO_LIVE

最小級Riskで実市場摩擦を確認するLive段階。

## LIMITED_LIVE

通常より制限されたExposure / Position / FrequencyでLive運用。

## NORMAL_LIVE

承認された通常Risk範囲で利用可能。

## PAUSED

本番利用一時停止。

重要:

> Risk割合・昇格閾値は本State Dictionaryへ固定値として埋め込まない。
> Research / Risk DesignでVersion付きPolicyとして定義する。

---

# 9. Production Promotion基本遷移

```text
RESEARCH_ONLY
↓
SHADOW
↓
DEMO_FORWARD
↓
MICRO_LIVE
↓
LIMITED_LIVE
↓
NORMAL_LIVE
```

劣化時:

```text
NORMAL_LIVE
↓
LIMITED_LIVE
↓
MICRO_LIVE
↓
PAUSED
↓
RESEARCH_ONLY / RESEARCHING
```

原則:

- 昇格は段階を飛ばさない
- 降格はSafetyのため段階を飛ばしてよい
- Critical Failureでは `NORMAL_LIVE → PAUSED` を許可

---

# STATE-RISK-001: Risk State Machine

対象:

- `RiskState`
- Market Instance / Portfolio / Account Scope

正式候補State:

```text
NORMAL
CAUTION
RISK_REDUCED
MICRO_ONLY
NO_NEW_ENTRY
EMERGENCY
```

## NORMAL

通常Policy範囲でRiskを許可。

## CAUTION

警戒状態。

候補Trigger:

- Data Quality軽度低下
- Edge Health弱化
- DD悪化開始
- Market Novelty上昇
- Execution friction増加

## RISK_REDUCED

許可Exposure / Position Size / Frequency等を縮小。

## MICRO_ONLY

最小級Riskのみ許可。

## NO_NEW_ENTRY

新規Entry禁止。

既存Positionの安全管理は継続。

## EMERGENCY

重大Safety Event。

候補:

- Critical Exchange Failure
- Position state unknown
- Severe Data corruption
- Global DD limit breach
- Execution runaway

Emergency ActionはRisk / Recovery Contractで定義する。

---

# 10. Risk State Transition原則

上方向のRisk許可回復は慎重に行う。

例:

```text
EMERGENCY
→ NO_NEW_ENTRY
→ MICRO_ONLY
→ RISK_REDUCED
→ CAUTION
→ NORMAL
```

一方、危険側への遷移は即時を許可する。

```text
NORMAL → EMERGENCY
```

もCritical Triggerなら許可。

回復条件候補:

- Root Cause resolved
- Data Health restored
- Execution reconciliation complete
- Cooldown elapsed
- Manual / Policy approval
- Edge health revalidated

---

# E. POSITION / TRADE THESIS

# STATE-POS-001: Position Thesis State

対象:

- `PositionThesisState`

正式候補State:

```text
HOLD
WATCH
CAUTION
THESIS_WEAKENING
THESIS_INVALIDATED
```

## HOLD

Trade Thesisが概ね想定範囲内。

## WATCH

重要変化が出始めており監視強化。

## CAUTION

複数Weakening条件等により警戒。

## THESIS_WEAKENING

Primary / Supporting Evidence低下、Contradiction増加、Expected Effect遅延等によりTrade Thesis健全性が明確に悪化。

## THESIS_INVALIDATED

Entry ThesisのInvalidation Conditionが成立。

重要:

- `THESIS_INVALIDATED` = Exit Decisionそのものではない
- Position Supervisorは状態を示す
- Exit Engine / In-Trade Defenseが最終Actionを決める

---

# 11. Position State Hysteresis

短期Noiseで、

```text
HOLD ↔ WATCH ↔ HOLD ↔ WATCH
```

を高速反転させない。

後続設計で次を設定可能にする。

```text
minimum_persistence_duration
minimum_evidence_count
minimum_change_magnitude
cooldown
multi_timeframe_confirmation
```

ただしHard Safety EventはHysteresisを飛ばしてCritical側へ遷移可能。

---

# F. RUNTIME / HEALTH

# STATE-RT-001: Runtime State

対象:

- Module
- Market Instance
- Runtime Process Group

正式候補State:

```text
BOOTING
RUNNING
PAUSED
DEGRADED
ERROR
RECOVERY
STOPPING
STOPPED
```

## BOOTING

Dependency / Config / Storage / Adapter等を確認し起動中。

Productionは必要DependencyがHealthyになる前に注文可能状態へ移行しない。

## RUNNING

期待された機能が稼働中。

## PAUSED

Processは存在するが主要処理を意図的に停止。

## DEGRADED

一部機能低下はあるが限定稼働可能。

例:

```text
Non-critical Source failure
→ DEGRADED
→ Quality / Confidence低下
```

## ERROR

通常処理を継続できない障害状態。

## RECOVERY

Recovery処理実行中。

## STOPPING

Graceful Shutdown中。

## STOPPED

停止完了。

---

# 12. Runtime基本遷移

```text
STOPPED
↓ START
BOOTING
├→ RUNNING
├→ DEGRADED
└→ ERROR

RUNNING
├→ PAUSED
├→ DEGRADED
├→ ERROR
└→ STOPPING

DEGRADED
├→ RUNNING
├→ ERROR
├→ PAUSED
└→ STOPPING

ERROR
├→ RECOVERY
└→ STOPPING

RECOVERY
├→ RUNNING
├→ DEGRADED
├→ ERROR
└→ STOPPING

STOPPING
→ STOPPED
```

禁止:

```text
STOPPED → RUNNING  # BOOTINGを飛ばさない
ERROR → RUNNING    # Recovery / health checkを無視しない
```

Critical EmergencyではSafe Stopを優先できる。

---

# STATE-HLT-001: System Health State

対象:

- Source
- DB
- Queue
- Adapter
- Module
- Instance

候補State:

```text
HEALTHY
DEGRADED
CRITICAL
UNKNOWN
```

## UNKNOWN

Health判定Data自体が不足。

UNKNOWNをHEALTHYへ自動変換しない。

Runtime StateとHealth Stateは分離する。

例:

```text
Runtime = RUNNING
Health = DEGRADED
```

はあり得る。

---

# G. INCIDENT / RECOVERY

# STATE-INC-001: Incident Lifecycle

対象:

- `Incident`
- 重大Error / Failure

候補State:

```text
DETECTED
CLASSIFYING
CONTAINED
MITIGATING
RECOVERING
RESOLVED
MONITORING
CLOSED
REOPENED
```

## DETECTED

障害検知。

## CLASSIFYING

Severity / Scope / Root Cause候補を評価。

## CONTAINED

影響拡大を止めた。

例:

- Source isolate
- Execution pause
- Queue stop
- Circuit breaker

## MITIGATING

暫定対策中。

## RECOVERING

正常系へ復帰処理中。

## RESOLVED

直接障害は解消。

## MONITORING

再発確認期間。

## CLOSED

復旧確認・必要Knowledge化完了。

## REOPENED

再発または未解決が判明。

---

# STATE-REC-001: Recovery Action State

候補:

```text
PENDING
RUNNING
SUCCEEDED
FAILED
ABORTED
ESCALATED
```

Retry無限ループを防止する。

後続Failure Contractで、

```text
max_retry
backoff
cooldown
circuit_breaker
manual_escalation
```

をVersion付きPolicyとして定義する。

---

# H. SOURCE / DATA QUALITY

# STATE-SRC-001: Source Lifecycle / Availability State

対象:

- Logical Source
- Provider Source

候補State:

```text
ACTIVE
DEGRADED
UNAVAILABLE
FALLBACK
DEPRECATED
RETIRED
UNKNOWN
```

## FALLBACK

Primary Source不調時に代替Providerを利用中。

Logical SourceとProviderを分離する。

例:

```text
OPEN_INTEREST = ACTIVE
Provider A = UNAVAILABLE
Provider B = FALLBACK
```

Source Provider終了でMarket理解概念そのものを失わない。

---

# STATE-DQ-001: Data Quality State

対象:

- `QualityProfile`

候補State:

```text
HEALTHY
DEGRADED
UNRELIABLE
INVALID
UNKNOWN
```

## HEALTHY

通常利用可能。

## DEGRADED

一部問題あり。Confidence / Applicabilityへ制約を伝播。

## UNRELIABLE

重要判断へ利用するには信頼性不足。

## INVALID

破損・不整合等により利用不可。

## UNKNOWN

品質評価自体が十分できない。

Qualityは単一Stateだけでなく、Missing / Freshness / Latency / Consistency等の元Dimensionを保持する。

---

# I. STORAGE / BACKUP / MIGRATION

# STATE-STO-001: Storage Lifecycle State

対象:

- Raw / Research / Operational Data Retention

候補State:

```text
HOT
WARM
COLD
ARCHIVED
DELETION_PENDING
DELETED
```

## HOT

高速参照対象。

## WARM

頻度は下がるが比較的高速に利用可能。

## COLD

低頻度参照・圧縮・低Cost保存。

## ARCHIVED

長期保存対象。

## DELETION_PENDING

Retention Policyで削除候補。削除前Validation / Legal / Knowledge Value確認を可能にする。

## DELETED

削除完了を示すMetadata状態。

重要:

Raw Dataの長期保存原則と有限Storageを、Compression / Tiering / Archiveで両立する。

---

# STATE-BKP-001: Backup Lifecycle

候補State:

```text
PENDING
RUNNING
CREATED
VERIFYING
VERIFIED
FAILED
EXPIRED
```

重要:

> Backupは `CREATED` だけでは成功扱いにしない。
> Restore可能性を確認して `VERIFIED` を区別する。

定期Restore TestはBackup Governanceで設計する。

---

# STATE-MIG-001: Schema / Data Migration Lifecycle

候補State:

```text
PLANNED
TESTING
READY
RUNNING
VERIFYING
COMPLETED
FAILED
ROLLED_BACK
SUPERSEDED
```

原則:

- Migration前Versionを追跡可能にする
- Old Schemaを即時破壊しない
- Verification前に旧Dataを削除しない
- Rollback Pathを用意する

---

# J. DEPLOYMENT

# STATE-DPL-001: Software Deployment Stage

対象:

- Code / Module / Runtime Version

候補Stage:

```text
DEVELOPMENT
UNIT_TESTED
INTEGRATION_TESTED
REPLAY_VALIDATED
PAPER_VALIDATED
CANARY
PRODUCTION
ROLLED_BACK
```

重要:

> Software Deployment StageとHypothesis / Edge Production Promotion Stageを混同しない。

例えば、

```text
Code = PRODUCTION
Hypothesis = DEMO_FORWARD
```

は成立する。

---

# 13. State Authority

Stateは誰でも自由に書き換えられるものではない。

正式Authorityは後続Contractで確定するが、責任候補を次とする。

| State Machine | Primary Authority |
|---|---|
| Hypothesis | Causal Engine + Research / Approval |
| Research Candidate | Research Orchestrator |
| Research Trial | Experimental / Validation Framework |
| Knowledge Aging | Knowledge Aging Governance |
| Production Promotion | Approval + Risk Governance |
| Risk State | Risk Governance / Defense |
| Position Thesis | Position Supervisor |
| Runtime | Runtime |
| System Health | Monitoring |
| Incident | Failure / Recovery Governance |
| Source | Source Lifecycle Governance / Adapter |
| Data Quality | Data Quality |
| Storage | Resource / Storage Governance |
| Backup | Backup Governance |
| Migration | Version / Migration Governance |
| Deployment | Deployment / Operations |

原則:

- Signal EngineがRisk Stateを勝手にNORMALへ戻さない
- RuntimeがHypothesisをAPPROVEDへ変更しない
- TelegramがState DBを直接書き換えない
- AI ReviewがProduction Promotionを直接昇格しない

---

# 14. State Escalation / De-escalation原則

Safety Stateでは次を基本とする。

```text
危険側への遷移
= 速くてよい

安全側への復帰
= 検証を要求する
```

例:

```text
NORMAL → EMERGENCY
```

は即時可能。

しかし、

```text
EMERGENCY → NORMAL
```

は原則一発復帰させない。

---

# 15. Hard TriggerとSoft Trigger

## Hard Trigger

即時遷移可能な重大条件。

候補:

- Position state unknown
- Exchange order runaway
- Critical Data corruption
- Hard Constraint violation
- Emergency Risk Limit breach

## Soft Trigger

Persistence /複数Evidence / Cooldown等を確認して遷移する条件。

候補:

- Edge Health徐々に悪化
- Contradiction増加
- Market Novelty増加
- Data freshness軽度低下

この区別により、過敏なState反転とSafety遅延の両方を防ぐ。

---

# 16. State Hysteresis共通設計

高頻度に再評価されるStateは、必要に応じて次を持つ。

```yaml
minimum_persistence_duration:
minimum_trigger_count:
minimum_change_magnitude:
cooldown_period:
recovery_confirmation_count:
hard_trigger_override:
```

対象候補:

- Position Thesis State
- Risk State
- Health State
- Edge Health
- Data Quality

---

# 17. Manual Override

人間によるState Overrideは完全禁止しないが、必ずAudit可能にする。

最低限:

```yaml
override_id:
target_state_machine:
from_state:
to_state:
requested_by:
reason:
requested_at:
expires_at:
confirmation_ref:
```

危険操作はConfirmationを要求できる。

Manual Overrideによって元Evidence / Failureを消さない。

---

# 18. StateとMarket Eventの関係

一つのMarket Eventが複数State Transitionを起こすことがある。

例:

```text
Liquidity Collapse Event
├→ Data Quality = DEGRADED
├→ Risk = RISK_REDUCED
├→ Position Thesis = WATCH
└→ Research Candidate = NEW
```

すべてに同じMarket Event ID / Trigger Refを関連付けることで、後から影響を追跡可能にする。

---

# 19. StateとTrace / Impact Trace

State TransitionはBackward ProvenanceだけでなくForward Impactを追跡可能にする。

例:

```text
Source = UNAVAILABLE
↓
Feature Quality = DEGRADED
↓
Hypothesis Applicability低下
↓
Trade Thesis影響
↓
Risk = RISK_REDUCED
```

将来のDependency / Impact Contractで正式化する。

---

# 20. StateとEvidence Channel

Hypothesis / Edge State変更時に、Evidence Sourceを失わない。

例:

```text
Historical = PASS
OOS = PASS
Demo Forward = FAIL
Live = insufficient
```

を、単純に `WEAK` 一文字だけへ圧縮しない。

StateはSummaryであり、元のEvidencePackage / AssessmentProfileを参照可能にする。

---

# 21. State変更とKnowledge化

重要なState Transition自体をResearch資産にできる。

例:

```text
NORMAL_LIVE → PAUSED
原因: Demo/Live Divergence
```

```text
RUNNING → ERROR → RECOVERY → RUNNING
原因: Source timeout
```

これらを後から、

- Failure
- Negative Knowledge
- Recovery Knowledge
- Constraint
- Research Candidate

へ変換可能にする。

---

# 22. State Definition追加Gate

新Stateを追加する前に必ず確認する。

```text
1. 既存Stateで表現できないか？
2. StateではなくDecisionではないか？
3. StateではなくAssessment Dimensionではないか？
4. StateではなくExperiment Modeではないか？
5. 既存State + reason_codeで十分ではないか？
6. 新Stateに明確な遷移条件があるか？
7. 誰がStateを変更するか決まっているか？
8. 禁止遷移を定義できるか？
9. 長期維持価値があるか？
```

満たさない場合、新Stateを増やさない。

---

# 23. State変更ルール

State名・意味・遷移を変更する場合、`DESIGN_CHANGE_RULES.md` に従う。

特にMajor変更候補:

- State削除
- State意味変更
- Lifecycle順序変更
- Production Promotion飛び越し許可
- Risk State意味変更
- Runtime State意味変更

これらは既存DB / Python Enum / Test / Monitoring / Telegram / Analyticsへ影響するため、Impact Analysisを必須候補とする。

---

# 24. Python実装への変換方針

後の `PYTHON_ARCHITECTURE.md` では、本辞書を基準にEnum / State Machineへ落とす。

例候補:

```python
class RuntimeState(str, Enum):
    BOOTING = "BOOTING"
    RUNNING = "RUNNING"
    PAUSED = "PAUSED"
    DEGRADED = "DEGRADED"
    ERROR = "ERROR"
    RECOVERY = "RECOVERY"
    STOPPING = "STOPPING"
    STOPPED = "STOPPED"
```

ただし本書ではPython class名・LibraryをCanonical固定しない。

重要なのは論理State定義であり、実装言語は将来変更可能にする。

---

# 25. Database実装への変換方針

後の `DATABASE_SCHEMA.md` では、最低限次を検討する。

```text
current_state
state_machine_version
state_changed_at
```

加えてState History用に、概念上:

```text
StateTransitionEvent
```

を持つ候補とする。

例:

```yaml
transition_id:
target_object_id:
state_machine_type:
from_state:
to_state:
trigger_refs: []
reason_codes: []
changed_at:
changed_by_role:
state_machine_version:
```

正式DB Table名は `DATABASE_SCHEMA.md` で確定する。

---

# 26. Monitoring / Telegram表示原則

Stateは人間が現在状況をすぐ理解できるよう表示可能にする。

例:

```text
BTC Instance
Runtime: RUNNING
Health: DEGRADED
Risk: RISK_REDUCED
Production Stage: LIMITED_LIVE
Position Thesis: WATCH
```

異なるState Machineを一つの `SYSTEM_STATUS = BAD` へ潰さない。

---

# 27. Long-Term Governance

何十年運用ではState体系そのものも変化する。

そのため:

- State Machine Versionを残す
- 廃止Stateを即削除しない
- Old State → New State Migration Mapを持てるようにする
- 過去Trade / Trialを当時Stateの意味で再現可能にする
- 新VersionのState意味で過去履歴を自動書き換えない

例:

```text
Runtime State Machine v1
↓
Runtime State Machine v2
```

になっても、2030年のRuntime Eventをv1定義で解釈可能にする。

---

# 28. State Dictionary Definition of Done

StateまたはState Machineを正式化するには最低限:

```text
□ State Machine名
□ 対象Object / Instance
□ State一覧
□ 各Stateの意味
□ Primary Authority
□ 入口条件
□ 出口条件
□ 許可遷移
□ 禁止遷移
□ Recovery / Reopen条件
□ Hard / Soft Trigger
□ Hysteresis要否
□ State History保存
□ Trace / Trigger Reference
□ Version Rule
□ Monitoring表示
□ Long-Term Migration方針
```

を確認する。

---

# 29. 現段階で未確定として残す項目

以下は本State Dictionaryだけで固定せず、後続設計で数値・詳細Ruleを決める。

```text
Risk Stateの具体的DD閾値
MICRO_LIVE / LIMITED_LIVEの資金割合
Research Early Stopの具体的Sample数
Knowledge Agingの具体的期限
Health Stateの数値閾値
Position Supervisor Hysteresisの具体時間
Retry回数 / Backoff時間
Storage Retention期間
Backup頻度
```

理由:

これらはState名ではなくPolicy / Threshold / Contractの問題であり、Research・Risk・Operations設計でVersion付きにした方がよい。

---

# 30. 現段階のState体系まとめ

```text
Hypothesis:
DRAFT → RESEARCHING → SUPPORTED → APPROVED → ACTIVE
                    ↘ WEAK / CONFLICTED
ACTIVE → AGING / SUSPENDED / RETIRED → REOPENED → RESEARCHING

Research Candidate:
NEW → SCREENING → QUEUED → RUNNING → COMPLETED
                    ├→ PAUSED
                    ├→ STOPPED
                    ├→ MERGED
                    ├→ REJECTED
                    └→ EXPIRED

Production Promotion:
RESEARCH_ONLY → SHADOW → DEMO_FORWARD → MICRO_LIVE → LIMITED_LIVE → NORMAL_LIVE
                                                                  ↓
                                                                PAUSED

Risk:
NORMAL → CAUTION → RISK_REDUCED → MICRO_ONLY → NO_NEW_ENTRY → EMERGENCY

Position Thesis:
HOLD → WATCH → CAUTION → THESIS_WEAKENING → THESIS_INVALIDATED

Runtime:
STOPPED → BOOTING → RUNNING
                    ├→ PAUSED
                    ├→ DEGRADED
                    └→ ERROR → RECOVERY
RUNNING / DEGRADED → STOPPING → STOPPED

Incident:
DETECTED → CLASSIFYING → CONTAINED → MITIGATING → RECOVERING → RESOLVED → MONITORING → CLOSED
```

---

# 31. 最終原則

市場理解OSではStateを「便利な文字列」として扱わない。

Stateは、

> **長期間の研究・Production・Risk・Runtimeが、今どの段階にあり、なぜそこへ移動したかを説明するための正式な履歴構造**

として扱う。

そして、

```text
State
+ Transition Rule
+ Trigger
+ History
+ Version
+ Authority
```

をセットで管理する。

これにより市場理解OSは、数年後・数十年後でも、

```text
「なぜこのHypothesisを止めた？」
「なぜこの日にRiskを下げた？」
「なぜこのProcessはDEGRADEDになった？」
「なぜこのEdgeはProductionから外れた？」
```

を再現可能にする。
