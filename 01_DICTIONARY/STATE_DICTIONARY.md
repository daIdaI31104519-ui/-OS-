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

## StateTransitionEvent

Stateそのものではなく、State Machineで実際に成功・適用された一回の遷移事実を保存するImmutable Object。

正式Object定義は:

```text
01_DICTIONARY/OBJECT_DICTIONARY.md
OBJ-STATE-001: StateTransitionEvent
```

を参照する。

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
latest_transition_event_ref:
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

- Object本体またはCurrent State Projectionに現在Stateを保持する
- 成功したState変更ごとに`StateTransitionEvent`をImmutable保存する

方式を正式化する。

`current_state`等のProjectionは高速参照用であり、過去Transition Historyの代替ではない。

---

# 4. Current StateとState Historyを分離する

FIX-010で次を正式化する。

```text
Current State
= 現在の高速参照用Projection / Cache

StateTransitionEvent
= 過去に実際に成功・適用されたState変更事実を保存するImmutable履歴
```

例:

```text
H-101 current_state = APPROVED
latest_transition_event_ref = STE-0003

History:
STE-0001 DRAFT → RESEARCHING
STE-0002 RESEARCHING → SUPPORTED
STE-0003 SUPPORTED → APPROVED
```

Current Stateだけを上書きして過去履歴を失わない。

State履歴は、将来次を説明できること。

```text
いつ変わった？
なぜ変わった？
何がTriggerだった？
どのEvidence / Error / Decisionが原因？
誰がRequestした？
誰 / どのRoleがRecommendした？
誰 / どのRoleがApproveした？
誰 / どのRoleが実際にApplyした？
どのState Machine Versionだった？
Manual Overrideだった？
一つ前のTransitionは何だった？
```

原則として、正当な`StateTransitionEvent`を`transition_sequence`順にReplayすることでCurrent Stateを再構築可能な設計を目標とする。

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

`trigger_refs` と `reason_codes` は意味を分ける。

```text
trigger_refs
= 何が遷移の起点になったか

reason_codes
= なぜその遷移を適用したか
```

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

成功Transitionは当時の`state_machine_version`をStateTransitionEventへ保存する。

## RULE-STATE-007: 成功TransitionだけをStateTransitionEvent化する

Authority Check / Transition Rule Check / Precondition Checkで拒否されたRequestは、成功State Transitionではない。

```text
Rejected / Failed Transition Attempt
≠ StateTransitionEvent
```

拒否・失敗はAuditEvent / Diagnostics / ApprovalDecision候補で追跡する。

## RULE-STATE-008: StateTransitionEventはImmutable

過去のTransition判断が後から誤りと判明しても、既存Eventを削除・書き換えない。

必要なら新しい合法Transition、Correction / Superseded関係、Reopen等で履歴を追加する。

## RULE-STATE-009: Concurrent / Stale State Writeを防止する

Transition適用時には、少なくとも概念上:

```text
expected_previous_state
```

と実Current Stateの一致を検証できるようにする。

例:

```text
Process A reads ACTIVE
Process B reads ACTIVE

A applies ACTIVE → SUSPENDED
B later tries ACTIVE → RETIRED
```

BはCurrentが既にSUSPENDEDなら古いACTIVE前提のTransitionとして拒否できること。

具体的Compare-And-Set / Transaction方式はData / Processing Contract / DB Schemaで固定する。

## RULE-STATE-010: transition_sequenceで順序を確定可能にする

同一Target / State Machine内でTransition履歴の順序を一意に追跡可能にする。

Timestampだけに依存して遷移順序を曖昧にしない。

## RULE-STATE-011: Authority責任を分離する

FIX-013以降、State変更責任を次へ分離する。

```text
REQUEST
↓
RECOMMEND
↓
APPROVE
↓
APPLY
↓
StateTransitionEvent
```

```text
Request Authority
≠ Recommend Authority
≠ Approve Authority
≠ Apply Authority
```

4責任を必ず別人・別Processへ割り当てる必要はないが、意味と権限を混同しない。

## RULE-STATE-012: Applyは原則Single Writer

```text
1 State Machine
= 原則1 Apply Authority / single-writer responsibility
```

複数Roleが同じCurrent Stateを無制限に直接更新してはならない。

## RULE-STATE-013: RestrictionとRecoveryを非対称にする

```text
Safety Restriction / 危険側への遷移
= 明示されたEmergency AuthorityによるFast Pathを許可可能

Recovery / Permission Expansion / Risk Expansion
= Strict Approval / Revalidationを要求する
```

Restrictive Authorityが単独で安全側・権限拡大側へ自動復帰させない。

---

# 6. State分類

市場理解OSではStateを大きく次へ分ける。

```text
A. HYPOTHESIS / EDGE LIFECYCLE
B. RESEARCH LIFECYCLE
C. KNOWLEDGE LIFECYCLE / HEALTH
D. PRODUCTION PROMOTION / RISK
E. POSITION / TRADE THESIS
F. RUNTIME / HEALTH
G. INCIDENT / RECOVERY
H. SOURCE / DATA QUALITY
I. STORAGE / BACKUP / MIGRATION
J. DEPLOYMENT
```

Git / Designの `IDEA / PROPOSAL / CANONICAL` 等は `GIT_RULES.md` を正本とし、本書では重複定義しない。

すべての主要State Machineは、成功遷移履歴に共通`StateTransitionEvent`を利用できる。個別の`HypothesisStateHistory` / `RiskStateHistory` / `RuntimeStateHistory`等を理由なく増殖させない。

## FIX-012 4軸分離原則

```text
Hypothesis / Edge Lifecycle
= 研究・知識としてどこまで成熟したか

Knowledge Aging / Health
= その知識が現在どれだけ新鮮・再検証済み・健全か

Production Promotion Stage
= そのKnowledgeをProductionでどこまで利用してよいか

Risk State
= OS / Market Instance / Portfolioとして現在どこまでRiskを取ってよいか
```

正式原則:

```text
Research maturity
≠ Knowledge freshness / health
≠ Production permission
≠ Current OS risk permission
```

例:

```text
Edge Lifecycle = APPROVED
Knowledge Aging / Health = DEGRADED
Production Promotion = PAUSED
Risk State = NORMAL
```

は成立する。

---

# A. HYPOTHESIS / EDGE LIFECYCLE

# STATE-HYP-001: CausalHypothesis Lifecycle

対象:

- `CausalHypothesis`

FIX-012正式候補State:

```text
DRAFT
RESEARCHING
SUPPORTED
WEAK
CONFLICTED
APPROVED
RETIRED
REOPENED
```

CausalHypothesis Lifecycleは研究成熟度・知識としての地位だけを表す。

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

研究上、一部支持は残るが再現性・Evidenceが弱い。

Knowledge Aging / Healthの`AGING / STALE / DEGRADED`とは別。

## CONFLICTED

支持Evidenceと反証Evidenceが強く競合し、単純なSUPPORTED / WEAKへ分類できない。

## APPROVED

Knowledge Promotion / Approvalを通過し、Production利用候補となり得る研究上の地位。

重要:

```text
APPROVED
≠ CURRENT
≠ NORMAL_LIVE
≠ Risk NORMAL
```

## RETIRED

研究・Knowledgeとして正式に利用対象から退役。

履歴は削除しない。

## REOPENED

WEAK / CONFLICTED / RETIRED等だった仮説を、新Data・構造変化・新Formula等により再研究対象へ戻した状態。

通常:

```text
REOPENED → RESEARCHING
```

へ進む。

## FIX-012 Migration

```text
ACTIVE
→ Hypothesis Lifecycleから除外。Production利用中かはProduction Promotion Stageで表す。

AGING
→ Hypothesis Lifecycleから除外。Knowledge Aging / Health = AGINGで表す。

SUSPENDED
→ Hypothesis Lifecycleから除外。Knowledge劣化はKnowledge Aging / Health、Production停止はPAUSEDで表す。
```

過去のStateTransitionEventを削除・書き換えず、旧State Machine Versionとして解釈する。

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
  ├→ APPROVED
  ├→ WEAK
  └→ CONFLICTED

APPROVED
  └→ RETIRED

WEAK / CONFLICTED / RETIRED
  ↓ 条件成立
REOPENED
  ↓
RESEARCHING
```

禁止例:

```text
DRAFT → APPROVED
RESEARCHING → NORMAL_LIVE
APPROVED → AGING   # Knowledge Aging側
APPROVED → PAUSED  # Production Promotion側
```

---

# STATE-HYP-002: Edge Lifecycle

対象:

- `Edge`

FIX-012正式候補State:

```text
CANDIDATE
VALIDATING
SUPPORTED
APPROVED
RETIRED
REOPENED
```

Edge Lifecycleは再現可能な優位性としての研究・承認成熟度だけを表す。

重要:

- `FeatureKnowledge` / `FormulaKnowledge` をEdge Lifecycleへ入れない
- Feature / Formulaの知識鮮度・再検証必要性は `STATE-KNW-001` で管理する
- Edgeの現在Production利用段階は `STATE-PRD-001` で管理する

例:

```text
Edge Lifecycle = APPROVED
Knowledge Aging / Health = AGING
Production Promotion = LIMITED_LIVE
```

は成立する。

基本遷移候補:

```text
CANDIDATE
→ VALIDATING
→ SUPPORTED
→ APPROVED
→ RETIRED

RETIRED
→ REOPENED
→ VALIDATING
```

FIX-012 Migration:

```text
ACTIVE
→ Production Promotion Stageへ責任移動

DEGRADED
→ Knowledge Aging / Health = DEGRADEDへ責任移動

SUSPENDED
→ Production停止はPAUSED、Knowledge劣化はKnowledge Aging / Healthへ責任分離
```

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

正式候補State:

```text
DRAFT
READY
ACTIVE
COMPLETED
SUPERSEDED
CANCELLED
```

## DRAFT

研究計画を作成・編集している段階。

## READY

必要な定義・評価基準・Data Scope等が揃い、Trial開始可能。

## ACTIVE

このPlanに基づくResearch Trialが1件以上実行中、または研究全体が継続中。

重要:

> `ACTIVE` はPlanが編集可能という意味ではない。Planの編集可否は別の `ResearchPlan Lock State` で表す。

## COMPLETED

PlanのCompletion Criteriaを満たし、計画として完了。

## SUPERSEDED

新VersionのResearchPlanへ置換済み。

## CANCELLED

実行前または途中で計画を正式に取消。

### 基本遷移

```text
DRAFT
→ READY
→ ACTIVE
→ COMPLETED

DRAFT / READY / ACTIVE
→ CANCELLED

DRAFT / READY / ACTIVE / COMPLETED
→ SUPERSEDED
```

---

# STATE-RSCH-002-L: ResearchPlan Lock State

対象:

- `ResearchPlan`

目的:

> PlanのLifecycleとは別に、評価規則・Data Scope・Metric・Stop Rule等を変更してよいかを表す。

正式候補State:

```text
EDITABLE
PRE_REGISTERED
FROZEN
```

## EDITABLE

研究計画を編集可能。

## PRE_REGISTERED

結果を見る前に主要評価条件を事前登録済み。変更には理由・Version更新を要求できる。

## FROZEN

Demo Forward / Holdout等で、T0以降の評価規則を変更禁止とする固定状態。

変更が必要なら同じPlanを上書きせず、新Plan Versionを作成する。

### 許可される組み合わせ例

```text
Lifecycle = READY   + Lock = PRE_REGISTERED
Lifecycle = ACTIVE  + Lock = PRE_REGISTERED
Lifecycle = ACTIVE  + Lock = FROZEN
Lifecycle = COMPLETED + Lock = FROZEN
```

重要:

> `ACTIVE` と `FROZEN` は矛盾しない。前者は研究計画の進行状態、後者は編集可否を表す。

禁止:

```text
FROZENだからLifecycleもFROZENと扱う
ACTIVEだから評価Ruleを自由に変更する
```

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

# C. KNOWLEDGE LIFECYCLE / HEALTH

# STATE-KNW-001: Knowledge Aging / Health State

対象:

- `KnowledgeLifecycleProfile`
- Hypothesis / Edge / FeatureKnowledge / FormulaKnowledge / Constraint等

FIX-012正式候補State:

```text
FRESH
CURRENT
AGING
STALE
DEGRADED
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

## ARCHIVED

現役利用対象外だが歴史・研究資産として保存。

FIX-012 Migration:

```text
SUSPENDED
→ Knowledge Aging / Healthから除外。
  Knowledge health低下はDEGRADED、Production利用停止はProduction Promotion = PAUSEDで表す。
```

重要:

Knowledge `DEGRADED`であっても、Hypothesis / Edgeの研究上のAPPROVED履歴を自動的に消さない。

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
- `HypothesisPoolEntry.production_stage`

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

Production Promotion StageはProduction利用許可だけを表す。

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

当該Knowledge / RuleのProduction利用一時停止。

重要:

```text
PAUSED
≠ Hypothesis / Edge RETIRED
≠ Knowledge DEGRADED
≠ Risk EMERGENCY
```

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

劣化・Safety・再検証必要時:

```text
NORMAL_LIVE / LIMITED_LIVE / MICRO_LIVE / DEMO_FORWARD / SHADOW
→ PAUSED
```

原則:

- 昇格は段階を飛ばさない
- 降格はSafetyのため段階を飛ばしてよい
- Critical Failureでは `NORMAL_LIVE → PAUSED` を許可
- Knowledge Aging / Healthの遷移とProduction Promotionの遷移は別StateTransitionEventとして残す

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

Risk Stateは現在OS / Market Instance / PortfolioとしてどこまでRiskを取ってよいかを表し、個別Hypothesis / EdgeのProduction Promotion Stageとは別。

## NORMAL

通常Policy範囲でRiskを許可。

## CAUTION

警戒状態。

候補Trigger:

- Data Quality軽度低下
- Knowledge Health弱化
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
- Knowledge health revalidated

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
STOPPED → RUNNING
ERROR → RUNNING
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

# 13. FIX-013 State Authority Matrix

Stateは誰でも自由に書き換えられるものではない。

FIX-013では、State変更のAuthorityを次の4責任へ正式分離する。

```text
REQUEST
↓
RECOMMEND
↓
APPROVE
↓
APPLY
↓
StateTransitionEvent
```

## 13.1 Authority Meaning

```text
REQUEST
= State変更の必要性を要求する。Current Stateは変更しない。

RECOMMEND
= Evidence / Validation / Domain専門判断に基づき、Transitionを推奨する。Current Stateは変更しない。

APPROVE
= Governance / Policy上、そのTransitionを実施してよいと許可する。Current Stateへ直接書き込まない。

APPLY
= Current State / expected_previous_state / Transition Rule / Authority / Approvalを検証し、実際にCurrent State変更を適用する。
```

正式原則:

```text
Request Authority
≠ Recommend Authority
≠ Approve Authority
≠ Apply Authority
```

## 13.2 Single-Writer Principle

```text
1 State Machine
= 原則1 Apply Authority / single-writer responsibility
```

複数Roleが同じCurrent Stateへ無制限に直接書き込む構造を禁止する。

`State Controller`という名前は、新しいTop-Level Layerを意味せず、既存Domain内のApply responsibilityを表す論理名。

## 13.3 Authority Matrix

| State Machine | Request Authority候補 | Recommend Authority | Approve Authority | Apply Authority |
|---|---|---|---|---|
| Hypothesis Lifecycle | Causal Engine / Research / Post-Trade | Research / Validation | Knowledge Promotion / Approval Gate | Hypothesis State Controller |
| Edge Lifecycle | Research / Post-Trade | Validation Framework | Knowledge Promotion / Approval Gate | Edge State Controller |
| Knowledge Aging / Health | Research / Post-Trade / Monitoring / Knowledge Domain | Knowledge Aging Governance | Knowledge Aging Governance | Knowledge Lifecycle Controller |
| Production Promotion | Research / Knowledge / Risk / Post-Trade | Validation / Knowledge Promotion | Knowledge Promotion + Risk Governance | Production Promotion Controller |
| Risk State | Monitoring / Defense / Runtime / Authorized Human | Defense / Risk Governance | Risk Governance | Risk State Controller |
| Research Candidate | Authorized Domain Roles / Research Router | Research Router | Research Orchestrator | Research Candidate Controller |
| Research Plan Lifecycle | Research | Research Orchestrator | Research Orchestrator | Research Plan Controller |
| Research Plan Lock | Research / Validation | Validation Framework | Research Orchestrator / Validation Framework | Research Plan Lock Controller |
| Research Trial | Research / Validation | Experimental / Validation Framework | Experimental / Validation Framework | Research Trial Controller |
| Constraint Lifecycle | Research / Validation / Risk | Validation / Knowledge Promotion | Knowledge Promotion / Risk Governance | Constraint State Controller |
| Position Thesis | Market / Evidence inputs / Supervisor | Position Supervisor | Position Supervisor Policy | Position Thesis Controller / Position Supervisor |
| Runtime | Human / Monitoring / Recovery / Outer Control | Runtime / Monitoring / Recovery | Runtime Authority | Runtime Controller |
| System Health | Monitoring probes / subsystem diagnostics | Monitoring | Monitoring Policy | Health Controller |
| Incident | Monitoring / Runtime / Recovery | Failure / Recovery Governance | Failure / Recovery Governance | Incident Controller |
| Recovery Action | Incident / Runtime / Human | Recovery | Recovery Policy / Authority | Recovery Controller |
| Source Lifecycle | Adapter / Monitoring | Source Lifecycle Governance | Source Lifecycle Governance | Source Lifecycle Controller |
| Data Quality | Data Quality checks / Monitoring | Data Quality | Data Quality Policy | Data Quality Controller |
| Storage Lifecycle | Resource / Storage Monitoring | Storage Governance | Storage Governance | Storage Controller |
| Backup Lifecycle | Operations / Scheduler | Backup Governance | Backup Governance | Backup Controller |
| Migration Lifecycle | Version / Migration Governance | Migration Validation | Migration Governance | Migration Controller |
| Deployment Stage | Development / Operations | Validation / CI | Deployment / Release Control | Deployment Controller |

このMatrixはSemantic Responsibilityの正本候補であり、実装上のClass名・Process名・IAM設定は後続Contract / Python / Security設計で決める。

## 13.4 Shared State Transition Engine

共通`State Transition Engine`は実装可能だが、Authorityではない。

```text
State Transition Engine
= Transition Rule Check
+ expected_previous_state / CAS
+ Approval / Authorization validation
+ Atomic Apply
+ StateTransitionEvent persist
+ Current Projection update

State Authority
= 誰がRequest / Recommend / Approve / Applyできるか
```

共通Engineが全State MachineのApprove権限を持つ設計は禁止する。

## 13.5 Safety Restrictive Fast Path

次のような危険側・制限側Transitionは、Policyで明示されたEmergency AuthorityによるFast Pathを許可できる。

```text
Risk NORMAL → EMERGENCY
Risk NORMAL → NO_NEW_ENTRY
Production NORMAL_LIVE → PAUSED
Runtime RUNNING → PAUSED / STOPPING
Source ACTIVE → UNAVAILABLE
Data Quality HEALTHY → INVALID
```

Fast Pathでも:

```text
Authority Check
Transition Rule Check
StateTransitionEvent
Audit / Trace
```

を省略しない。

## 13.6 Recovery / Permission Expansion Strict Path

次のような安全側・権限拡大側TransitionはRestrictive Fast Pathより厳しく扱う。

```text
Risk EMERGENCY → NORMAL
Risk NO_NEW_ENTRY → NORMAL
Production PAUSED → NORMAL_LIVE
Runtime ERROR / RECOVERY → RUNNING
Data Quality INVALID → HEALTHY
Source UNAVAILABLE → ACTIVE
```

候補要件:

```text
Root Cause resolved
Revalidation / Health verification
Cooldown / persistence
Required Approval
Current State再確認
```

Restrictive Authorityが単独でRisk / Permissionを元へ戻してはならない。

## 13.7 Human / Telegram

Human / TelegramはState DBへ直接書き込まない。

```text
Human / Telegram
→ RuntimeCommand / Manual Override Request
→ Authentication / Authorization
→ Authority Flow
→ APPLY
→ StateTransitionEvent
```

`/stop` / `/emergency` / `/no-entry`等のSafety Restrictionは、Policyで許可されたEmergency Fast Pathへ接続可能。

`/risk-normal` / `/normal-live`等のRisk Expansion / Permission ExpansionはStrict Approvalを要求する。

## 13.8 AI / Logger / Post-Trade

External AI Reviewは原則:

```text
REQUEST / RECOMMEND
```

まで。AI単独でAPPROVE / APPLYしない。

LoggerはStateTransitionEvent / AuditEventのCustodianであり、State Authorityではない。

Post-Trade AnalysisはState変更材料を分析しRequest / Recommendation候補を出せるが、Current Stateへ直接書き込まない。

## 13.9 ApprovalDecisionとの境界

FIX-013では`APPROVE`責任を正式化するが、独立`ApprovalDecision` Objectはまだ作らない。

```text
ApprovalDecision Object
→ FIX-015で正式化
```

FIX-013時点では`authorization_ref / recommendation_ref`等から後続Objectへ接続可能にする。

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

どちらの成功TransitionもStateTransitionEventとして記録し、回復の方が慎重であることを履歴から確認できるようにする。

FIX-013以降、この非対称性はAuthorityにも適用する。

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

- Knowledge Aging / Health徐々に悪化
- Contradiction増加
- Market Novelty増加
- Data freshness軽度低下

この区別により、過敏なState反転とSafety遅延の両方を防ぐ。

StateTransitionEventにはHard / Soft判定の根拠となったTrigger / Reasonを参照可能にする。

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
- Knowledge Aging / Health
- Data Quality

Hysteresis Ruleを変更しても過去Transitionを新Ruleで再解釈せず、State Machine / Policy Versionから当時の条件を追跡できるようにする。

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

Manual Overrideが実際にState変更へ成功した場合、そのStateTransitionEventには:

```yaml
manual_override_ref:
authorization_ref:
requested_by_actor:
```

を残し、自動遷移と人間Overrideを区別する。

Override要求が拒否された場合は成功StateTransitionEventを生成しない。

FIX-013以降、Manual Overrideも原則Apply Authorityを経由する。Emergency Restrictive Overrideだけは明示Policyに従うFast Pathを許可できる。

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

すべての成功Transitionに同じMarket Event ID / Trigger Refを関連付けることで、後から影響を追跡可能にする。

ただし一つのMarketEventから複数StateTransitionEventが生成され得るため、MarketEventとStateTransitionEventを同一Objectにしない。

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

各成功State Transitionに`trace_id / trigger_refs / previous_transition_event_ref`等を持たせ、後から順序と影響を追えるようにする。

FIX-013以降、重要TransitionではRequest / Recommendation / Approval / Apply provenanceも追跡可能にする。

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

FIX-012では、最近のDemo / Live劣化を研究成熟度へ直接混ぜず、Knowledge Aging / HealthのTransitionとして表現できる。

StateTransitionEventの`trigger_refs`から、State変更を起こしたEvidencePackage / ResearchResult / ProductionEvidence等へ遡れることを目標とする。

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

これらのStateTransitionEventを後から、

- Failure
- Negative Knowledge
- Recovery Knowledge
- Constraint
- Research Candidate

へ変換可能にする。

State履歴そのものを消さず、Knowledgeは履歴を参照する。

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
7. 誰がRequest / Recommend / Approve / Applyできるか決まっているか？
8. 禁止遷移を定義できるか？
9. 長期維持価値があるか？
10. FIX-012の別State軸の責任を侵食していないか？
```

満たさない場合、新Stateを増やさない。

State History Objectも同様に個別Objectを増やさず、原則共通`StateTransitionEvent`を利用する。

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
- Apply Authority変更
- Emergency Fast Path権限変更

これらは既存DB / Python Enum / Test / Monitoring / Telegram / Analytics / StateTransitionEvent解釈へ影響するため、Impact Analysisを必須候補とする。

State Machine Version変更時も過去StateTransitionEventを新Versionの意味で無言に書き換えない。

FIX-012の旧StateはMigration Mappingを残し、過去履歴を物理削除しない。

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

FIX-012以降、次のような万能Status判定を新規実装しない。

```python
if hypothesis.status == "ACTIVE":
    trade()
```

Production可否は概念上:

```text
Lifecycle
AND Knowledge Aging / Health
AND Production Promotion Stage
AND Applicability
AND Constraint
AND Risk State
```

を別々に確認する。

FIX-013以降、各ModuleがCurrent Stateを直接更新する実装を避け、Apply Authority / State Transition Engine経由へ統一する。

---

# 25. Database実装への変換方針

後の `DATABASE_SCHEMA.md` では、Current StateとTransition Historyを分離して検討する。

Current Projection候補:

```text
state_machine_id
state_machine_type
current_state
previous_state
state_machine_version
state_changed_at
latest_transition_event_ref
```

Transition Historyは正式Semantic Object:

```text
OBJ-STATE-001: StateTransitionEvent
```

を保存する。

論理Fields:

```yaml
state_transition_event_id:
target_object_ref:
state_machine_id:
state_machine_type:
state_machine_version:
from_state:
to_state:
expected_previous_state:
transition_sequence:
trigger_refs: []
reason_codes: []
requested_by_role:
recommended_by_role:
recommendation_ref:
authorized_by_role:
applied_by_role:
requested_by_actor:
transitioned_at:
effective_at:
created_at:
manual_override_ref:
authorization_ref:
previous_transition_event_ref:
trace_id:
```

FIX-012では同一Knowledgeについて:

```text
HYPOTHESIS_LIFECYCLE / EDGE_LIFECYCLE
KNOWLEDGE_AGING
PRODUCTION_PROMOTION
```

を一つの`status`列へ圧縮しない。

FIX-013ではDBへ直接書き込めるRole / Serviceを無制限に増やさず、State MachineごとのApply Authority / write pathを一意にできる設計を優先する。

正式Table名・Index・Transaction方式は `DATABASE_SCHEMA.md` で確定する。

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

FIX-012対象Knowledgeでは必要に応じて:

```text
Hypothesis: APPROVED
Knowledge: AGING
Production: LIMITED_LIVE
Risk: CAUTION
```

のように軸を分けて表示する。

異なるState Machineを一つの `SYSTEM_STATUS = BAD` や `status = ACTIVE` へ潰さない。

必要時にはCurrent Stateだけでなく、直近`StateTransitionEvent`の変更時刻・Reason Code・Authority provenanceを表示可能にする。

---

# 27. Long-Term Governance

何十年運用ではState体系そのものも変化する。

そのため:

- State Machine Versionを残す
- StateTransitionEventへ当時のState Machine Versionを残す
- 廃止Stateを即削除しない
- Old State → New State Migration Mapを持てるようにする
- 過去Trade / Trialを当時Stateの意味で再現可能にする
- 新VersionのState意味で過去履歴を自動書き換えない
- Current Projectionが壊れてもTransition Historyから再構築可能な設計を維持する
- Authority Matrix / Apply Authority変更もVersion / Audit対象にする

FIX-012以前の:

```text
Hypothesis ACTIVE / AGING / SUSPENDED
Edge ACTIVE / DEGRADED / SUSPENDED
Knowledge SUSPENDED
```

も旧State Machine Versionとして履歴を保存し、新定義へ無言で書き換えない。

---

# 28. State Dictionary Definition of Done

StateまたはState Machineを正式化するには最低限:

```text
□ State Machine名
□ 対象Object / Instance
□ State一覧
□ 各Stateの意味
□ 他State Machineとの責任境界
□ Request Authority
□ Recommend Authority
□ Approve Authority
□ Apply Authority
□ Emergency / Restrictive Authority要否
□ 入口条件
□ 出口条件
□ 許可遷移
□ 禁止遷移
□ Recovery / Reopen条件
□ Hard / Soft Trigger
□ Hysteresis要否
□ StateTransitionEvent保存
□ transition_sequence / Ordering rule
□ Trace / Trigger Reference
□ Version Rule
□ Migration Rule
□ Manual Override trace
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
Knowledge DEGRADED時のProduction Gate強度
AGING / STALE時のProduction縮小規則
Production Promotion再開条件
Health Stateの数値閾値
Position Supervisor Hysteresisの具体時間
Retry回数 / Backoff時間
Storage Retention期間
Backup頻度
StateTransitionEventの物理Table / Index / Partition
State Transition atomic write方式
State Transition Engineの実装方式
Authority delegation / secondary authorityの具体的IAM表現
Emergency Authorityの認証方式
ApprovalDecision Object / schema（FIX-015）
```

理由:

これらはState名そのものではなくPolicy / Threshold / Contract / Storage / Security / Approval Objectの問題であり、Research・Risk・Operations・Data Contract・DB・Security設計でVersion付きにした方がよい。

---

# 30. 現段階のState体系まとめ

```text
Hypothesis Lifecycle:
DRAFT → RESEARCHING → SUPPORTED → APPROVED → RETIRED
                    ├→ WEAK
                    └→ CONFLICTED
WEAK / CONFLICTED / RETIRED → REOPENED → RESEARCHING

Edge Lifecycle:
CANDIDATE → VALIDATING → SUPPORTED → APPROVED → RETIRED
RETIRED → REOPENED → VALIDATING

Knowledge Aging / Health:
FRESH → CURRENT → AGING → STALE
                    └────→ DEGRADED
                         → ARCHIVED

Research Candidate:
NEW → SCREENING → QUEUED → RUNNING → COMPLETED
                    ├→ PAUSED
                    ├→ STOPPED
                    ├→ MERGED
                    ├→ REJECTED
                    └→ EXPIRED

Research Plan Lifecycle:
DRAFT → READY → ACTIVE → COMPLETED
       └──────────────→ SUPERSEDED / CANCELLED

Research Plan Lock:
EDITABLE → PRE_REGISTERED → FROZEN

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

すべての成功Transitionは共通`StateTransitionEvent`として履歴化可能にする。

Authorityは全State Machineで概念上:

```text
REQUEST → RECOMMEND → APPROVE → APPLY → StateTransitionEvent
```

へ分離する。

---

# 31. 最終原則

市場理解OSではStateを「便利な文字列」として扱わない。

FIX-010では:

```text
Current State
= 現在値の高速Projection

StateTransitionEvent
= 実際に成功・適用されたState変更のImmutable Historical Fact
```

を分離する。

FIX-012ではさらに:

```text
Research maturity
≠ Knowledge freshness / health
≠ Production permission
≠ Current OS risk permission
```

を正式に分離する。

FIX-013ではさらに:

```text
REQUEST
≠ RECOMMEND
≠ APPROVE
≠ APPLY
```

を正式に分離し、Current Stateへの書込みは原則State MachineごとのApply Authorityへ一本化する。

さらに:

```text
Safety Restriction
= Fast Path可能

Recovery / Risk Expansion / Permission Expansion
= Strict Path
```

とする。

これにより市場理解OSは、数年後・数十年後でも、

```text
「誰がState変更を要求した？」
「誰がEvidenceを見て推奨した？」
「誰が承認した？」
「誰が実際に書き込んだ？」
「なぜEmergencyへ落とした？」
「なぜ復帰を許可した？」
```

まで再現可能にする。
