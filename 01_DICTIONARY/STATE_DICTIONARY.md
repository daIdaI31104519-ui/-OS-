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
誰 / どのRoleがAuthorizeした？
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

具体的Compare-And-Set / Transaction方式はData / Processing Contract / DB Schemaで固定する。

## RULE-STATE-010: transition_sequenceで順序を確定可能にする

同一Target / State Machine内でTransition履歴の順序を一意に追跡可能にする。

Timestampだけに依存して遷移順序を曖昧にしない。

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

---

# 6.1 FIX-012: Research Maturity / Knowledge Health / Production Permission / Risk Permission の完全分離

FIX-012では次の4軸を独立State Machine / State Projectionとして扱う。

```text
1. Hypothesis / Edge Lifecycle
= 研究・知識としてどこまで成熟したか

2. Knowledge Aging / Health
= その知識が現在どれだけ新鮮・再検証済み・健全か

3. Production Promotion Stage
= そのKnowledgeをProductionでどこまで使うことを許可されているか

4. Risk State
= OS / Market Instance / Portfolioとして現在どこまでRiskを取ってよいか
```

正式原則:

```text
Research maturity
≠ Knowledge freshness / health
≠ Production permission
≠ Current OS risk permission
```

一つの`status`へ圧縮しない。

例:

```text
Edge Lifecycle = APPROVED
Knowledge Aging / Health = DEGRADED
Production Promotion = PAUSED
Risk State = NORMAL
```

は合法であり、意味は:

```text
過去の研究・Approval自体は維持
現在Knowledge healthは低下
Production利用は停止
OS全体Risk環境そのものは通常
```

となる。

また:

```text
Hypothesis Lifecycle = APPROVED
Knowledge Aging / Health = CURRENT
Production Promotion = NORMAL_LIVE
Risk State = NO_NEW_ENTRY
```

も合法であり、この場合KnowledgeはProduction利用可能でもOS Safety Gateにより新規Tradeは禁止される。

StateTransitionEventでは同じKnowledgeに対して別々の履歴を持つ。

```text
HYPOTHESIS_LIFECYCLE
RESEARCHING → SUPPORTED

KNOWLEDGE_AGING
CURRENT → AGING

PRODUCTION_PROMOTION
LIMITED_LIVE → PAUSED
```

これらを一つのTransitionとして扱わない。

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

## Meaning Boundary

CausalHypothesis Lifecycleは**研究成熟度・知識としての地位**だけを表す。

次は表さない。

```text
現在Productionで使っているか
Knowledgeが古くなったか
現在Riskを取ってよいか
```

これらは別State Machineへ送る。

## DRAFT

仮説として構造化されたが、正式Researchを十分実行していない。

禁止:

- Production利用
- APPROVED扱い

## RESEARCHING

Research Plan / Trialで検証中。

## SUPPORTED

現在までのResearch Evidenceで一定の支持がある。

重要:

> SUPPORTED = Production利用許可ではない。

## WEAK

研究上、支持が弱い・再現性が不足している等の成熟度評価。

WEAKをKnowledge Agingの`AGING / STALE / DEGRADED`と混同しない。

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

研究・Knowledgeとして正式に利用対象から退役した状態。

履歴は削除しない。

## REOPENED

WEAK / CONFLICTED / RETIRED等だった仮説を、新Data・構造変化・新Formula等により再研究対象へ戻した状態。

通常:

```text
REOPENED → RESEARCHING
```

へ進む。

## FIX-012 Removed / Migrated State Semantics

```text
ACTIVE
→ Hypothesis Lifecycleから除外。
  Production利用中かどうかはProduction Promotion Stageで表す。

AGING
→ Hypothesis Lifecycleから除外。
  Knowledge Aging / Health = AGINGで表す。

SUSPENDED
→ Hypothesis Lifecycleから除外。
  Knowledge劣化はKnowledge Aging / Health、Production停止はPAUSEDで表す。
```

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
APPROVED → AGING      # Knowledge Aging側のState
APPROVED → PAUSED     # Production Promotion側のState
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

Edge Lifecycleは再現可能な優位性としての**研究・承認成熟度**だけを表す。

## FIX-012 Removed / Migrated State Semantics

```text
ACTIVE
→ Edge Lifecycleから除外。
  現在Production利用可能かはProduction Promotion Stageで表す。

DEGRADED
→ Edge Lifecycleから除外。
  現在のKnowledge health低下はKnowledge Aging / Health = DEGRADEDで表す。

SUSPENDED
→ Edge Lifecycleから除外。
  Production利用停止はProduction Promotion = PAUSEDで表す。
```

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

## COMPLETED

事前定義したCompletion Criteriaを満たした。

`良い結果だった`という意味ではない。

## STOPPED

Early Stop / Futility / Risk / Resource理由で研究終了。

## MERGED

重複Candidateへ統合された。

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

正式候補State:

```text
EDITABLE
PRE_REGISTERED
FROZEN
```

## EDITABLE

研究計画を編集可能。

## PRE_REGISTERED

結果を見る前に主要評価条件を事前登録済み。

## FROZEN

Demo Forward / Holdout等で、T0以降の評価規則を変更禁止とする固定状態。

変更が必要なら同じPlanを上書きせず、新Plan Versionを作成する。

許可される組み合わせ例:

```text
Lifecycle = READY   + Lock = PRE_REGISTERED
Lifecycle = ACTIVE  + Lock = PRE_REGISTERED
Lifecycle = ACTIVE  + Lock = FROZEN
Lifecycle = COMPLETED + Lock = FROZEN
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

---

# C. KNOWLEDGE LIFECYCLE / HEALTH

# STATE-KNW-001: Knowledge Aging / Health State

対象:

- `KnowledgeLifecycleProfile`
- Hypothesis / Edge / FeatureKnowledge / FormulaKnowledge / Constraint等の鮮度・健全性評価

FIX-012正式候補State:

```text
FRESH
CURRENT
AGING
STALE
DEGRADED
ARCHIVED
```

## Meaning Boundary

Knowledge Aging / Healthは**知識の現在の鮮度・再検証必要性・最近のEvidenceによる健全性**を表す。

研究成熟度やProduction利用許可、Risk許可は表さない。

## FRESH

最近生成・検証された。

## CURRENT

現在の利用条件で十分再検証されている。

## AGING

再検証期限が近い、またはEvidence鮮度低下。

## STALE

長期間再検証されていない。

`FALSE`とは限らないが、Production側でTrustを制限する理由になり得る。

## DEGRADED

最近のOOS / Demo / Live / Stress等で劣化Evidenceが観測された。

## ARCHIVED

現役Knowledge Health評価対象外として長期保管される状態。

## FIX-012 Removed / Migrated State Semantics

```text
SUSPENDED
→ Knowledge Aging / Healthから除外。
  Knowledge health低下はDEGRADED、Production利用停止はPAUSEDで表す。
```

重要:

```text
Knowledge = DEGRADED
```

であっても研究上の`APPROVED`履歴を自動的に消さない。

必要なProduction制限は別State Machineで行う。

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

Constraint LifecycleはConstraint自身のLifecycleであり、FIX-012のHypothesis / Edge成熟度分離とは別である。

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

## Meaning Boundary

Production Promotion Stageは**Production利用許可の段階**だけを表す。

次は表さない。

```text
研究上どこまで成熟したか
Knowledgeが古いか・劣化したか
現在OS全体がRiskを取ってよいか
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

当該Knowledge / RuleのProduction利用を一時停止。

重要:

```text
PAUSED
≠ Hypothesisが研究上Retired
≠ Knowledge healthが必ずDEGRADED
≠ OS全体RiskがEMERGENCY
```

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

再開・昇格条件はApproval / Research / Risk ContractでVersion付きに定義する。

原則:

- 昇格は段階を飛ばさない
- 降格・PAUSEDはSafetyのため段階を飛ばしてよい
- Knowledge Aging / Healthの変化とProduction Promotion遷移は別々のStateTransitionEventとして残す

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

## Meaning Boundary

Risk Stateは**現在OS / Market Instance / PortfolioとしてどこまでRiskを取ってよいか**を表す。

個別Hypothesis / EdgeのProduction Promotion Stageとは別である。

例:

```text
Hypothesis Lifecycle = APPROVED
Knowledge Aging / Health = CURRENT
Production Promotion = NORMAL_LIVE
Risk State = NO_NEW_ENTRY
```

の場合、新規Tradeは禁止。

## NORMAL

通常Policy範囲でRiskを許可。

## CAUTION

警戒状態。

## RISK_REDUCED

許可Exposure / Position Size / Frequency等を縮小。

## MICRO_ONLY

最小級Riskのみ許可。

## NO_NEW_ENTRY

新規Entry禁止。

既存Positionの安全管理は継続。

## EMERGENCY

重大Safety Event。

---

# 10. Risk State Transition原則

上方向のRisk許可回復は慎重に行う。

```text
EMERGENCY
→ NO_NEW_ENTRY
→ MICRO_ONLY
→ RISK_REDUCED
→ CAUTION
→ NORMAL
```

危険側への遷移はCritical Triggerなら即時を許可する。

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

短期Noiseで状態を高速反転させない。

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

## RUNNING

期待された機能が稼働中。

## PAUSED

Processは存在するが主要処理を意図的に停止。

## DEGRADED

一部機能低下はあるが限定稼働可能。

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

Runtime StateとHealth Stateは分離する。

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

Logical SourceとProviderを分離する。

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

Qualityは単一Stateだけでなく、Missing / Freshness / Latency / Consistency等の元Dimensionを保持する。

---

# I. STORAGE / BACKUP / MIGRATION

# STATE-STO-001: Storage Lifecycle State

候補State:

```text
HOT
WARM
COLD
ARCHIVED
DELETION_PENDING
DELETED
```

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

Software Deployment StageとHypothesis / Edge Production Promotion Stageを混同しない。

---

# 13. State Authority

Stateは誰でも自由に書き換えられるものではない。

正式Authorityは後続Contract / FIX-013 Authority Matrixで確定するが、責任候補を次とする。

| State Machine | Primary Authority |
|---|---|
| Hypothesis Lifecycle | Causal Engine + Research / Approval |
| Edge Lifecycle | Research / Approval |
| Research Candidate | Research Orchestrator |
| Research Plan Lifecycle | Research Orchestrator |
| Research Plan Lock | Research Orchestrator / Validation Framework |
| Research Trial | Experimental / Validation Framework |
| Knowledge Aging / Health | Knowledge Aging Governance |
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

FIX-013でRequest / Recommend / Approve / Applyの最終責任分離を確定する。

---

# 14. State Escalation / De-escalation原則

Safety Stateでは次を基本とする。

```text
危険側への遷移
= 速くてよい

安全側への復帰
= 検証を要求する
```

---

# 15. Hard TriggerとSoft Trigger

## Hard Trigger

即時遷移可能な重大条件。

## Soft Trigger

Persistence / 複数Evidence / Cooldown等を確認して遷移する条件。

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

---

# 17. Manual Override

人間によるState Overrideは完全禁止しないが、必ずAudit可能にする。

Manual Overrideが実際にState変更へ成功した場合、そのStateTransitionEventへ追跡情報を残す。

---

# 18. StateとMarket Eventの関係

一つのMarket Eventが複数State Transitionを起こすことがある。

すべての成功Transitionに同じMarket Event ID / Trigger Refを関連付けることで、後から影響を追跡可能にする。

---

# 19. StateとTrace / Impact Trace

State TransitionはBackward ProvenanceだけでなくForward Impactを追跡可能にする。

---

# 20. StateとEvidence Channel

Hypothesis / Edge State変更時に、Evidence Sourceを失わない。

StateはSummaryであり、元のEvidencePackage / AssessmentProfileを参照可能にする。

---

# 21. State変更とKnowledge化

重要なState Transition自体をResearch資産にできる。

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
7. 誰がStateを変更するか決まっているか？
8. 禁止遷移を定義できるか？
9. 長期維持価値があるか？
```

FIX-012以降、新State追加時は別軸の責任を侵食していないかも確認する。

---

# 23. State変更ルール

State名・意味・遷移を変更する場合、`DESIGN_CHANGE_RULES.md` に従う。

FIX-012の旧State MigrationはBackup ManifestとMigration Mappingを保持し、過去履歴を新Stateへ無言で書き換えない。

---

# 24. Python実装への変換方針

後の `PYTHON_ARCHITECTURE.md` では、本辞書を基準にEnum / State Machineへ落とす。

重要:

```text
hypothesis.status == ACTIVE
```

のような単一Mystery StatusによるProduction判定を新規実装しない。

Production判定は概念上:

```text
Lifecycle
AND Knowledge Aging / Health
AND Production Promotion Stage
AND Applicability
AND Constraint
AND Risk State
```

を別々に確認する。

---

# 25. Database実装への変換方針

Current StateとTransition Historyを分離する。

FIX-012では同一Knowledgeについて複数State Machineを別Projection /別Historyとして保持可能にする。

```text
HYPOTHESIS_LIFECYCLE
EDGE_LIFECYCLE
KNOWLEDGE_AGING
PRODUCTION_PROMOTION
```

を一つの`status` Columnへ圧縮しない。

正式Table名・Index・Transaction方式は `DATABASE_SCHEMA.md` で確定する。

---

# 26. Monitoring / Telegram表示原則

人間向け表示でも軸を潰さない。

例:

```text
Hypothesis: APPROVED
Knowledge: AGING
Production: LIMITED_LIVE
Risk: CAUTION
```

これを単一の`STATUS = ACTIVE`等へ変換しない。

---

# 27. Long-Term Governance

State Machine Versionを残し、FIX-012以前の旧State履歴も当時のVersionで解釈可能にする。

旧:

```text
Hypothesis ACTIVE / AGING / SUSPENDED
Edge ACTIVE / DEGRADED / SUSPENDED
Knowledge SUSPENDED
```

を物理削除・履歴改竄しない。

新規生成ではFIX-012後の責任分離されたStateを利用する。

---

# 28. State Dictionary Definition of Done

StateまたはState Machineを正式化するには最低限:

```text
□ State Machine名
□ 対象Object / Instance
□ State一覧
□ 各Stateの意味
□ 他State Machineとの責任境界
□ Primary Authority
□ 許可遷移
□ 禁止遷移
□ Recovery / Reopen条件
□ StateTransitionEvent保存
□ Version Rule
□ Migration Rule
```

を確認する。

---

# 29. 現段階で未確定として残す項目

以下は後続設計で決める。

```text
Risk Stateの具体的DD閾値
MICRO_LIVE / LIMITED_LIVEの資金割合
Knowledge Agingの具体的期限
Knowledge DEGRADED時のProduction Gate強度
AGING / STALE時のProduction縮小規則
Production Promotion再開条件
State Transition atomic write方式
Authority Matrixの最終一意化
```

これらはState名ではなくPolicy / Threshold / Contract / Authorityの問題である。

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

Production Promotion:
RESEARCH_ONLY → SHADOW → DEMO_FORWARD → MICRO_LIVE → LIMITED_LIVE → NORMAL_LIVE
      └──────────────────────────────────────────────────────────────→ PAUSED

Risk:
NORMAL → CAUTION → RISK_REDUCED → MICRO_ONLY → NO_NEW_ENTRY → EMERGENCY

Research Plan Lifecycle:
DRAFT → READY → ACTIVE → COMPLETED

Research Plan Lock:
EDITABLE → PRE_REGISTERED → FROZEN

Position Thesis:
HOLD → WATCH → CAUTION → THESIS_WEAKENING → THESIS_INVALIDATED

Runtime:
STOPPED → BOOTING → RUNNING / DEGRADED / ERROR → RECOVERY / STOPPING → STOPPED
```

---

# 31. 最終原則

市場理解OSではStateを「便利な文字列」として扱わない。

FIX-012以降の最重要原則:

```text
Research maturity
≠ Knowledge freshness / health
≠ Production permission
≠ Current OS risk permission
```

つまり、

```text
Hypothesis / Edge Lifecycle
+ Knowledge Aging / Health
+ Production Promotion Stage
+ Risk State
```

を独立して管理する。

一つの`ACTIVE / SUSPENDED / DEGRADED`等の曖昧なstatusへ戻さない。

これにより市場理解OSは、

```text
「研究としては承認済みだが、今は古い」
「知識は健全だが、まだDemo Forwardまで」
「KnowledgeはLive利用可能だが、OS全体Riskで新規Entry禁止」
「Edgeは過去Approvalを維持するが、最近Liveで劣化したためProduction停止」
```

をそれぞれ正確に表現できる。
