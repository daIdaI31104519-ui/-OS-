# ROLE_DICTIONARY.md

# 市場理解OS 役割辞書・責任境界完全設計

## 0. 文書情報

- 文書種別: DICTIONARY / ROLE SOURCE OF TRUTH
- 状態: CANONICAL DICTIONARY CANDIDATE
- 上位ルール:
  - `00_GOVERNANCE/GIT_RULES.md`
  - `00_GOVERNANCE/DESIGN_CHANGE_RULES.md`
- 対象: 市場理解OSのArchitecture / Research / Production / Platform / Governanceで使用するRole
- 目的: 各Roleの目的・責任・入力・出力・禁止事項・Failure時挙動・Researchへの戻し方を一元管理し、責任重複とLayer乱立を防ぐ
- 最上位原則: **Roleは「何を担当するか」で定義し、Object / Contract / View / Experiment Modeと混同しない**

---

# 1. Role Dictionaryの目的

市場理解OSを何十年以上運用する前提では、同じ責任を複数Moduleへ重複実装したり、新しい案のたびに新Layerを増やすと設計が崩れる。

そのため、本書をRole定義のSingle Source of Truth候補とする。

本書で解決すること:

1. 各Roleが何を担当するか
2. 各Roleが何を担当してはいけないか
3. 上流・下流の責任境界
4. Objectの所有・生成責任
5. Failure時にどこまで処理してよいか
6. Research Candidateをどこへ返すか
7. 長期運用時にRoleが肥大化しないための境界
8. 新Role追加が本当に必要か判断する基準

---

# 2. Role / Object / Contract / View / Modeの区別

## Role

処理・判断・管理の責任を持つ。

例:

```text
Collector
Market Intelligence
Causal Engine
Signal Engine
Position Supervisor
Runtime
```

## Object

Role間で生成・保存・受け渡しされる情報構造。

例:

```text
RawData
MarketEvent
Evidence
CausalHypothesis
MarketDNA
TradeThesis
OrderIntent
TradeResult
```

## Contract

Role間で何をどの形式で渡すかを定義する規則。

例:

```text
Data Contract
Processing Contract
Error Contract
Research Contract
Trade Thesis Contract
```

## View

同じKnowledge Objectを異なる目的で参照する表示・検索方法。

例:

```text
Case Library
Market Memory
Failure Museum
Knowledge Graph
```

## Experiment Mode

Research Domain内で実験方式を切り替えるためのMode。

例:

```text
RANDOM_BASELINE
HISTORICAL
REPLAY
PAPER
DEMO_FORWARD
SHADOW
COUNTERFACTUAL
```

重要:

> Object / Contract / View / Modeを、理由なく独立Top-Level Roleへ昇格させない。

---

# 3. Role定義の標準フォーマット

全Roleは原則として次を定義する。

```text
Role ID
Role Name
Category
Status
Purpose
Owns
Inputs
Outputs
Upstream
Downstream
Responsibilities
Prohibitions
Failure Behavior
Research Feedback
Long-Term Notes
```

Role詳細設計では本辞書の責任境界を変更せず、内部処理だけを深掘りする。

責任境界を変更する場合は `DESIGN_CHANGE_RULES.md` に従う。

---

# 4. Role分類

市場理解OSのRoleを大きく次へ分ける。

```text
A. GOVERNANCE / CONTROL
B. CONNECTION / ADAPTER
C. MARKET OBSERVATION / UNDERSTANDING
D. RESEARCH / KNOWLEDGE
E. PRODUCTION / TRADING
F. POST-TRADE / FEEDBACK
G. PLATFORM / OPERATIONS
```

---

# A. GOVERNANCE / CONTROL

# ROLE-GOV-001: Governance / Constitution Authority

## Category
GOVERNANCE

## Status
ACTIVE DESIGN CANDIDATE

## Purpose

市場理解OS全体が長期原則・安全原則・設計変更原則へ違反しないよう統治する。

## Owns

- Project Constitution
- Git Governance
- Design Change Governance
- Long-Term Governance Rules
- Canonical / Proposal境界

## Inputs

- Design Change Proposal
- Architecture Review
- Risk / Failure / Research Governance情報
- Migration Requirement

## Outputs

- Governance Decision
- Change Constraint
- Approval Requirement
- HOLD / REJECT / MERGE / ADOPT判断材料

## Responsibilities

- 上位設計原則を維持する
- CanonicalとProposalを混同させない
- 長期互換性を確認する
- 安易な新Layer追加を防ぐ

## Prohibitions

- Market方向判断
- BUY / SELL生成
- Research Resultの改ざん
- Production設定の直接自動変更

## Long-Term Notes

何十年運用ではコードより寿命が長い可能性があるため、実装から独立して保持する。

---

# ROLE-CTL-001: Outer Control

## Category
GOVERNANCE / CONTROL

## Purpose

市場理解OSを「何を対象に、どの設定・資金上限・接続条件で動かすか」という外側のDesired Stateを管理する。

## Owns

- Market Profile選択
- Runtime Command要求
- Global Risk Limit入力
- 接続先選択方針

## Inputs

- User / Operator Command
- Market Profile
- Global Risk Policy
- System Status

## Outputs

- RuntimeCommand
- MarketProfile / Configuration reference
- GlobalRiskLimit
- Adapter Configuration

## Upstream

- Human / Telegram Interface
- Governance

## Downstream

- Runtime
- Source Adapter
- Exchange Adapter
- Core

## Responsibilities

- どのMarket Instanceを動かすか決める
- START / PAUSE / RESUME / STOP要求を出す
- 資金・Exposure上限を渡す

## Prohibitions

- 市場解釈
- Causal判断
- Feature Priority決定
- BUY / SELL
- Processを自力で直接管理すること

## Boundary

```text
Outer = Desired State
Runtime = Actual Execution
```

---

# B. CONNECTION / ADAPTER

# ROLE-ADP-001: Source Adapter

## Category
CONNECTION / ADAPTER

## Purpose

外部Data Provider固有形式を市場理解OS標準形式へ変換する。

## Owns

- Provider固有Schema変換
- Symbol / Field mapping
- Source provenance付与

## Inputs

- Exchange / Provider API Response
- WebSocket Message
- Vendor-specific Data

## Outputs

- RawData-compatible payload
- SourceMetadata
- Diagnostics

## Responsibilities

- Provider差をCoreから隠す
- Source情報をProvenanceとして残す
- Schema mismatchをDiagnosticsへ出す

## Prohibitions

- Feature計算
- Market Interpretation
- Signal生成
- Cause判定

## Failure Behavior

変換不能・Schema mismatchを明示し、推測値で埋めない。

---

# ROLE-ADP-002: Exchange Adapter

## Category
CONNECTION / ADAPTER

## Purpose

標準OrderIntentを取引所固有注文へ変換し、注文・約定情報を標準形式へ戻す。

## Owns

- Symbol mapping
- Tick size / Minimum order対応
- API authentication interface
- Order request / response conversion

## Inputs

- OrderIntent
- Exchange Configuration

## Outputs

- ExecutionRecord
- Diagnostics

## Responsibilities

- Vendor固有API処理
- 約定情報取得
- Exchange response標準化

## Prohibitions

- BUY / SELL判断
- Position Sizeの戦略判断
- Trade Thesis生成
- Market理解

---

# C. MARKET OBSERVATION / UNDERSTANDING

# ROLE-DATA-001: Collector

## Category
MARKET OBSERVATION

## Purpose

外部世界からRaw Dataを取得する。

## Inputs

- Exchange
- Macro / ETF / News / SNS / On-chain / Other Sources

## Outputs

- RawData
- SourceMetadata
- Diagnostics

## Responsibilities

- 取得
- Timestamp / Source metadata保持
- Request / Receive latency記録

## Prohibitions

- Feature計算
- 「強気 / 弱気」最終解釈
- Cause判断
- BUY / SELL

## Failure Behavior

取得失敗を隠さずData Quality / Monitoringへ伝播する。

## Research Feedback

繰り返すSource latency / missing patternはResearchCandidate化可能。

---

# ROLE-DATA-002: Normalizer

## Category
MARKET OBSERVATION

## Purpose

Sourceごとの表現差を統一し、RawDataを比較・研究可能なObservationへ変換する。

## Inputs

- RawData

## Outputs

- Observation
- Diagnostics

## Responsibilities

- Timestamp / Timezone
- Symbol
- Unit
- Decimal
- Interval
- Naming
- Currency
- Missing representation

## Prohibitions

- 品質を最終確定する
- Feature生成
- Market判断

## Failure Behavior

変換失敗・補正量・Schema mismatchをDiagnosticsへ残す。

---

# ROLE-DATA-003: Data Quality

## Category
MARKET OBSERVATION / SAFETY

## Purpose

下流が入力をどこまで信用できるか判断できる品質情報を生成する。

## Inputs

- RawData / Observation
- SourceMetadata

## Outputs

- QualityProfile
- Diagnostics

## Responsibilities

- Missing
- Duplicate
- Latency
- Staleness
- Outlier
- Timestamp drift
- Cross-source inconsistency
- API failure
- Impossible value

## Prohibitions

- Market方向判断
- Cause判定
- Low Qualityを隠す

## Failure Behavior

自Role自体が不調な場合は下流へHigh Confidenceを許可しない。

---

# ROLE-DATA-004: Time Series Processor

## Category
MARKET OBSERVATION / MEASUREMENT

## Purpose

単一値を時間変化として扱える系列へ整形する。

## Inputs

- Observation
- QualityProfile

## Outputs

- TimeSeriesMeasurement

## Responsibilities

- Delta
- Return
- Rate of Change
- Velocity
- Acceleration
- Persistence
- Rolling values
- Lagged values

## Prohibitions

- 任意の時間窓を暗黙利用する
- Signal生成

## Long-Term Notes

sampling / measurement / lookback windowを明示し、Version追跡可能にする。

---

# ROLE-CALC-001: Calculation / Measurement Service

## Category
CROSS-CUTTING MEASUREMENT

## Purpose

OS全体で利用する計算式・測定定義を共通化する。

## Owns

- Formula Registry
- Feature Registryの計算定義部分
- Metric Registry
- Unit Rules
- Time Window Rules

## Inputs

- RawData / Observation / TimeSeriesMeasurement / Other approved inputs

## Outputs

- MeasurementResult
- FormulaDefinition reference

## Responsibilities

- 同じ概念を複数Roleで勝手に別Formula実装しないようにする
- Formula Versionを残す

## Prohibitions

- 「数学的に計算できる」だけでMarket意味を確定する
- Research未承認FormulaをProduction正式Formulaへ昇格

## Research Feedback

Formula比較・Sensitivity・OOS・StressはFormula Researchへ戻す。

---

# ROLE-FEAT-001: Feature Generator

## Category
MARKET UNDERSTANDING

## Purpose

RawData / Observation / TimeSeriesMeasurementから市場理解に利用できるVersioned Featureを生成する。

## Inputs

- Approved FormulaDefinition
- Observation / TimeSeriesMeasurement
- QualityProfile

## Outputs

- Feature
- FormulaDefinition reference
- QualityProfile reference
- Trace reference

## Responsibilities

- Feature値生成
- 入力参照保持
- Formula Version保持

## Prohibitions

- BUY / SELL
- Cause確定
- 自分のFeatureを自分でHigh Priorityにする

---

# ROLE-FEAT-002: Feature Priority

## Category
MARKET UNDERSTANDING

## Purpose

「どのFeatureが常に最強か」ではなく、「現在条件では何を見る価値が高いか」を決める。

## Inputs

- Historical / OOS usefulness
- Current Context / Event
- Time / Horizon
- Data Quality
- Feature Stability / Redundancy
- Market DNA関連情報

## Outputs

- FeaturePriorityProfile

## Responsibilities

- 注目優先順位生成
- Low Priority判断も保存

## Prohibitions

- Low Priority FeatureのRaw取得を無条件停止する
- Signal化
- Priority = Confidenceと扱う

## Research Feedback

Priority判断自体の失敗をFeature Researchへ戻す。

---

# ROLE-MI-001: Market Intelligence

## Category
MARKET UNDERSTANDING

## Purpose

「今、市場で何が起きているか」を構造化して解釈する。

## Inputs

- Feature
- FeaturePriorityProfile
- MarketEvent
- QualityProfile
- Context

## Outputs

- MarketContext
- Evidence
- Contradiction
- UnexplainedEvent
- ResearchCandidate（必要時）

## Responsibilities

- Price / Structure
- Volatility
- Liquidity
- Derivatives
- Participant Flow
- ETF / Institutional
- Macro
- News / Sentiment
- Abnormal Event

を市場状態として整理する。

## Prohibitions

- 原因を確定する
- Causal Hypothesisを最終確定する
- BUY / SELL

## Failure Behavior

説明不能なら物語を作らず `UnexplainedEvent` とする。

## Research Feedback

UnexplainedEvent / interpretation anomalyをResearchCandidateへ送る。

---

# ROLE-CAUSAL-001: Causal Engine

## Category
MARKET UNDERSTANDING / CAUSAL

## Purpose

観測されたEffectに対して、検証可能なCause Candidate・Alternative・Contradiction・検証要求を構造化する。

## Inputs

- MarketContext
- Evidence
- MarketEvent
- Feature / FeaturePriorityProfile
- MarketCase
- QualityProfile / Uncertainty

## Outputs

- CauseCandidate
- CausalHypothesis
- Contradiction
- ResearchCandidate
- HypothesisAssessmentProfile / State Transition recommendation

## Responsibilities

- Cause Candidate
- Effect
- Temporal Order
- Lag
- Confounder
- Alternative HypothesisをCausalHypothesis関係として保持
- Contradiction
- Evidence Strength profile
- Historical / OOS / Regime / Stress検証要求をResearchCandidateへ変換
- Hypothesis Lifecycle管理材料

## Prohibitions

- 相関だけでCause確定
- Historical / OOS / Stress大規模実験を自Roleで全部実行
- BUY / SELL

## Boundary

```text
Causal Engine = 何を検証すべきか定義
Research = 実際に検証する
```

---

# ROLE-DNA-001: Market DNA Builder / Manager

## Category
MARKET UNDERSTANDING / KNOWLEDGE INTERFACE

## Purpose

現在市場を過去市場と比較・検索・研究可能な圧縮状態表現へ変換する。

## Inputs

- MarketContext
- Feature
- Causal / Evidence information
- Approved DNA Formula / Axis definitions

## Outputs

- MarketDNA
- MarketCase references

## Responsibilities

- DNA axis計算
- Similarity / NoveltyをMarketDNA内の追跡可能なProfileとして生成
- Raw / Feature / Formula Versionへ追跡可能にする

## Prohibitions

- Market DNAをSignalそのものにする
- 「似ている = 同じ結果」と断定する
- Feature Priorityと同一概念化する

## Research Feedback

低Similarity / 高NoveltyはNovel Regime ResearchCandidate候補。

---

# D. RESEARCH / KNOWLEDGE

# ROLE-RSCH-001: Research Orchestrator

## Category
RESEARCH

## Purpose

ResearchCandidateを適切な研究方式へ振り分け、実験・Validation・EvidencePackage生成を管理する。

## Inputs

- ResearchCandidate
- CausalHypothesis / Edge / Feature / Formula / MarketDNA等の研究対象
- Failure / Unexpected Success
- DemoLiveDivergence

## Outputs

- ResearchPlan
- ResearchResult
- EvidencePackage
- Edge / Candidate Knowledge
- NegativeKnowledge candidate

## Responsibilities

- Research Queue
- Research Priority
- Experiment selection
- Research lifecycle
- Budget / Stop Ruleとの連携

## Prohibitions

- 未承認結果を直接Productionへ反映
- Candidateを自動でCanonical Knowledgeにする
- 無制限研究

## Long-Term Notes

Research Governanceに従い、開始・継続・停止・再開を管理する。

---

# ROLE-RSCH-002: Experimental Framework

## Category
RESEARCH SUBSYSTEM

## Purpose

複数の実験方式を共通基盤で実行する。

## Modes

- RANDOM_BASELINE
- HISTORICAL
- REPLAY
- PAPER
- DEMO_FORWARD
- SHADOW
- COUNTERFACTUAL

## Inputs

- ResearchPlan
- CausalHypothesis / ApplicableHypothesisSet candidate / other research target
- Market Data / Replay Data
- Execution Model

## Outputs

- ResearchResult
- Evidence Source Channel / Metadata reference
- Diagnostics

## Responsibilities

- Modeごとの時間・情報可用性ルールを守る
- Demo ForwardでT0以降だけを使う
- DemoとLiveで判断ロジックを可能な限り共通化する

## Prohibitions

- RandomをReal Money目的で使う
- Historical / Demo / Live結果を単純合算する
- 未来情報をReplayへ混ぜる

---

# ROLE-RSCH-003: Validation Framework

## Category
RESEARCH / VALIDATION

## Purpose

ResearchCandidate / Edge / Hypothesis SetがProduction候補になれるか検証する。

## Inputs

- ResearchResult
- CausalHypothesis / Edge candidate

## Outputs

- ResearchResult
- FailureBoundary
- Constraint candidate / reference

## Responsibilities

- OOS
- Regime Stability
- Stress Lab
- FailureBoundary抽出
- Historical / OOS / Regime / Demo / Stressの違いは別Object名ではなくResearchResultのEvidence Source Channel / Experiment Modeで保持する

## Prohibitions

- 検証成功だけで自動Production有効化
- Stress LabをResearch全体そのものとして扱う

---

# ROLE-KNOW-001: Knowledge Domain Manager

## Category
KNOWLEDGE

## Purpose

再利用可能な市場理解知識を一貫したObject群として保存・参照・Version管理する。

## Owns / Manages

- MarketCase
- MarketDNA references
- CausalHypothesis records
- FeatureKnowledge
- FormulaKnowledge
- ResearchResult
- Failure
- FailureBoundary
- Constraint
- NegativeKnowledge
- KnowledgeRelationship

## Inputs

- Approved / Candidate Knowledge
- ResearchResult
- ProductionEvidence
- Post-Trade Attribution

## Outputs

- Knowledge Object references / Query result
- MarketCase references
- Approved Knowledge references
- ResearchCandidate（Aging / Revalidation必要時）

## Responsibilities

- 同じ知識を複数DB/Viewへ無駄にコピーしない
- Version / provenance保持
- Knowledge Aging対応

## Prohibitions

- Viewごとに別の正本を作る
- 古い知識を永久真理扱いする

---

# ROLE-KNOW-002: Knowledge Promotion / Approval Gate

## Category
GOVERNANCE FUNCTION / KNOWLEDGE

## Purpose

ResearchResultをProduction利用可能Knowledgeへ昇格させる前に、Validation・状態・互換性を確認する。

## Inputs

- Candidate Knowledge / Edge
- ResearchResult / EvidencePackage
- Constraint
- Uncertainty

## Outputs

- APPROVED
- HOLD
- REJECT / RETIRE candidate
- Revalidation requirement

## Prohibitions

- ResearchResultを無検証でProductionへ通す
- AI Reviewだけで承認する

## Note

独立巨大Layerではなく、正式なApproval責任を表すGovernance Function。

---

# E. PRODUCTION / TRADING

# ROLE-AI-001: External AI Review

## Category
PRODUCTION ADVISORY

## Purpose

市場理解OSが作ったTradeThesis / Evidence / ApplicableHypothesisSetを外部知能として独立査読する。

## Inputs

- TradeThesis
- MarketDNA
- Evidence
- Contradiction
- EvidencePackage
- Constraint

## Outputs

- AIReviewResult
- ResearchCandidate（新規研究案が出た場合）

## Responsibilities

- 重複仮説指摘
- Shared Evidence二重評価指摘
- 最大反証・見落とし確認

## Prohibitions

- 最終Trade決定権
- 未検証Hypothesisをその場でProduction Thesisへ追加
- AI多数決を真実扱い

## Failure Behavior

AI Review停止でもProduction Coreが原則動作可能にする。

---

# ROLE-SIG-001: Signal Engine

## Category
PRODUCTION

## Purpose

現在のTradeThesisにRiskを取る合理性・期待値があるか判断する。

## Inputs

- TradeThesis
- ApplicableHypothesisSet reference
- Edge
- Expected Value
- MarketContext / MarketDNA
- MarketCase references
- FeaturePriorityProfile
- Constraint
- QualityProfile / Uncertainty
- AIReviewResult

## Outputs

- SignalDecision

## Responsibilities

- Expected Value中心の取引候補判断
- Causal / Empirical Edge双方を利用可能にする

## Prohibitions

- Defense責任を吸収する
- AIReviewResultだけで決定
- 全情報を意味不明な一つの総合Scoreへ潰す

---

# ROLE-DEF-001: Pre-Trade Defense

## Category
PRODUCTION / SAFETY

## Purpose

SignalDecisionが存在しても、現在Riskを取って安全かGateする。

## Inputs

- SignalDecision
- Constraint
- QualityProfile
- Liquidity / Spread / Slippage Risk
- DD / Consecutive Losses
- Exposure
- API / Exchange Health
- Abnormal Event
- GlobalRiskLimit

## Outputs

- DefenseDecision

## Responsibilities

- Hard / Safety条件確認
- Risk制約適用

## Prohibitions

- Expected Valueを作る
- TradeThesis生成
- BUY / SELL方向生成

## Research Feedback

BLOCKしたTradeThesisはShadow追跡候補。

---

# ROLE-EXEC-001: Execution Logic

## Category
PRODUCTION / EXECUTION

## Purpose

SignalDecision / DefenseDecision承認後、どの注文計画で実行するか決める。

## Inputs

- SignalDecision
- DefenseDecision
- Risk Budget
- Liquidity / Slippage information

## Outputs

- OrderIntent

## Responsibilities

- Position Size
- Leverage
- Limit / Market
- Split Order
- Stop / Take Profit
- Slippage tolerance
- Liquidity requirement

## Prohibitions

- Exchange固有APIを直接実装
- 市場仮説を作る

## Boundary

```text
Execution Logic = 標準注文計画
Exchange Adapter = 取引所固有実行
```

---

# ROLE-SUP-001: Position Supervisor

## Category
PRODUCTION / IN-TRADE INTELLIGENCE

## Purpose

Position保有中にEntryThesis / TradeThesisがまだ成立しているか監督し、Exit Engineへ助言する。

## Inputs

- EntryThesis
- Hypothesis state updates
- Current Evidence
- MarketContext
- Time / Expected Horizon

## Outputs

- PositionThesisState
- Diagnostics

## Responsibilities

- TradeThesis全体の健全性監視
- Hypothesis Decay / Thesis Decay監視
- Contradicting Hypothesis強化監視

## Prohibitions

- 単一短期Featureだけで即Thesis否定
- 通常Exitを無制限に自動実行
- Entry後に理由を書き換える

## Boundary

```text
Supervisor = Thesis健全性助言
Exit Engine = 通常Exit判断
In-Trade Defense = Hard Safety
```

---

# ROLE-DEF-002: In-Trade Defense

## Category
PRODUCTION / HARD SAFETY

## Purpose

Position保有中の強制安全制約を監視する。

## Inputs

- Position
- Exchange / API Health
- Data Health
- Liquidity
- Risk Limits
- Hard Constraint

## Outputs

- ExitDecision / safety action request
- Diagnostics

## Responsibilities

- Exchange failure
- API failure
- Data collapse
- Liquidity disappearance
- Max Loss
- Global Risk breach

## Prohibitions

- Thesisの意味解釈
- 通常の利益最大化Exit判断

---

# ROLE-EXIT-001: Exit Engine

## Category
PRODUCTION / EXIT

## Purpose

通常のPosition決済判断を行う。

## Inputs

- PositionThesisState
- Supervisor Advisory
- PnL
- Time / Horizon
- Liquidity
- Stop / Take Profit
- RiskState
- In-Trade Defense information

## Outputs

- ExitDecision

## Responsibilities

- 通常Exitの最終判断
- Supervisorの助言とRisk条件を統合

## Prohibitions

- EntryThesisを書き換える
- ResearchCandidateを直接Production Ruleへ反映

---

# F. POST-TRADE / FEEDBACK

# ROLE-LOG-001: Logger

## Category
POST-TRADE / AUDIT

## Purpose

Trade / Trial / Runtime処理の事実を追跡可能な形で記録する。

## Inputs

- Trade / Trial Events
- SignalDecision / DefenseDecision / ExecutionRecord / PositionThesisState
- MarketEvent / Trace references

## Outputs

- AuditEvent / immutable log records
- TradeResult（Trade完了時）

## Responsibilities

- 事実保存
- Version / Trace / Event ID保持
- EntryThesis保存

## Prohibitions

- 「なぜ負けた」を確定する
- ResearchResultを生成する
- ログから勝手にTrainerへ送る

---

# ROLE-ANL-001: Post-Trade Analysis

## Category
POST-TRADE / ANALYSIS

## Purpose

TradeResult / Demo等の結果を、Trade OutcomeとHypothesis / Execution / Risk Outcomeへ分解する。

## Inputs

- Logger Records
- TradeResult
- EntryThesis
- Hypothesis states
- ExecutionRecord

## Outputs

- OutcomeAnalysisResult
- TradeThesisEvaluation
- HypothesisAttribution
- DefenseEvaluation
- SupervisorEvaluation
- DemoLiveDivergence
- CounterfactualResult
- ResearchCandidate

## Responsibilities

- WIN / LOSSだけで評価しない
- Trade成功とHypothesis正しさを分離
- Demo / Live divergence分析

## Prohibitions

- 分析結果を直接Production変更へ反映

---

# ROLE-RTR-001: Research Router

## Category
POST-TRADE / RESEARCH INTERFACE

## Purpose

Post-Trade / Runtime / Market理解で発生したResearchCandidateを原因領域別に適切なResearchへ送る。

## Inputs

- ResearchCandidate
- HypothesisAttribution
- Failure / Unexpected Success

## Outputs

- ResearchRoute
- ResearchCandidate reference

## Responsibilities

例:

```text
Data issue → Data Quality Research
Formula issue → Formula Research
Feature issue → Feature Research
Market interpretation issue → MI Research
Causal issue → Causal Research
Hypothesis Set issue → Hypothesis Set Research
DNA mismatch → Market DNA Research
Execution issue → Execution Research
Supervisor issue → Supervisor Research
```

## Prohibitions

- 全失敗を一律Trainerへ送る
- Routingだけで原因を確定する

---

# G. PLATFORM / OPERATIONS

# ROLE-RT-001: Python Runtime / Operations

## Category
PLATFORM / OPERATIONS

## Purpose

Python Process群を正しい順序・依存関係・状態で起動・停止・維持する。

## Inputs

- RuntimeCommand / Desired State
- Configuration
- Dependency Status
- Health Status

## Outputs

- SystemStatus
- Diagnostics

## Responsibilities

- Boot Sequence
- Process Management
- Scheduler
- Dependency Check
- Config load
- Graceful Shutdown
- Restart execution

## Prohibitions

- Market Signal生成
- Research Rule変更
- Trade方向判断

## Boundary

```text
Outer = どうしたいか
Runtime = 実際にどう動かすか
```

---

# ROLE-MON-001: Monitoring

## Category
PLATFORM / OBSERVABILITY

## Purpose

市場ではなく市場理解OS自身のHealthを監視する。

## Inputs

- Runtime / Process metrics
- Collector / DB / Queue / Exchange health
- CPU / Memory / Disk

## Outputs

- SystemStatus / health update
- ResourceSnapshot
- ErrorEvent / Incident candidate
- Alert

## Responsibilities

- Source latency
- Data freshness
- DB health
- Queue backlog
- Memory / CPU / Disk
- Error rate
- Position / Order operational state

## Prohibitions

- Market方向判断
- Trade Signal生成

---

# ROLE-REC-001: Recovery

## Category
PLATFORM / RESILIENCE

## Purpose

障害時に部分故障を隔離し、安全に復旧する。

## Inputs

- ErrorEvent / Incident
- SystemStatus
- Recovery Policy

## Outputs

- RecoveryAction
- RuntimeCommand / safe-pause request
- Diagnostics

## Responsibilities

- Retry上限
- Backoff
- Circuit Breaker候補
- DB reconnect
- Queue recovery
- Safe pause

## Prohibitions

- 無限Restart
- Failureを隠す
- Research Rule / Trading Ruleを勝手に変更

---

# ROLE-DEPLOY-001: Deployment / Release Control

## Category
PLATFORM / RELEASE

## Purpose

設計・コード・Ruleの変更を安全な段階を通してProductionへ配置する。

## Inputs

- Validated Implementation
- Version Metadata
- Release Approval

## Outputs

- Deployment state / release record candidate
- MigrationRecord（Migration時）
- Rollback target

## Responsibilities

- Version tracking
- Release sequence
- Rollbackability

## Prohibitions

- 未承認ResearchResultの直接Production投入

---

# ROLE-IF-001: Telegram Interface

## Category
INTERFACE / HUMAN CONTROL

## Purpose

人間向けConsoleとして報告・状態確認・操作要求・緊急操作を仲介する。

## Inputs

- SystemStatus
- Market Status Summary
- Alerts
- Human Command

## Outputs

- Display / Notification
- RuntimeCommand request / authenticated command request

## Responsibilities

- `/status`
- `/health`
- `/positions`
- `/risk`
- `/research_status`
- START / PAUSE / RESUME / STOP要求

## Prohibitions

- TelegramからExchange APIを直接叩く
- 独自Market判断
- 認証なし危険操作

## Boundary

```text
Telegram
→ Command Gateway / Validation
→ Outer / Runtime / Defense / Execution
→ Exchange Adapter
```

---

# 5. Cross-Cutting Governance Functions

以下は重要だが、原則として市場処理の独立Top-Level Layerではなく、全体横断Governance Functionとして扱う。

## Resource Governance

対象:

- RAM
- Disk
- DB
- Queue
- Cache
- CPU
- API Cost

目的:

無限増殖・メモリリーク・Storage枯渇を防ぐ。

## Failure Governance

対象:

```text
Error
→ Classification
→ Retry / Isolation
→ DEGRADED / STOP
→ Recovery
→ Reoccurrence Review
```

## Risk / DD Governance

対象:

- Money DD
- Edge Health
- Execution Health
- Data Health
- Market Novelty

Risk State候補:

```text
NORMAL
CAUTION
RISK_REDUCED
MICRO_ONLY
NO_NEW_ENTRY
EMERGENCY
```

## Research Governance

対象:

- Research Admission
- Budget
- Priority
- Early Stop
- Pause
- Reopen
- Archive

## Version / Migration Governance

対象:

- Schema
- Formula
- Feature
- DNA
- Hypothesis
- Trade Thesis
- Config
- Code

## Backup / Disaster Recovery Governance

対象:

- Primary Storage
- Backup
- Restore Test
- Remote / Immutable Archive候補

## Knowledge Aging Governance

対象:

- last_validated
- current applicability
- Demo / Live degradation
- Revalidation due
- Suspend / Retire / Reopen

## Source Lifecycle Governance

対象:

- Provider変更
- API終了
- Schema変更
- Fallback Source
- Logical SourceとProvider分離

---

# 6. 明確にRoleとして扱わないもの

## Objects

```text
RawData
Observation
MarketEvent
Feature
Evidence
CausalHypothesis
MarketDNA
ResearchResult
Edge
Failure
FailureBoundary
Constraint
ApplicableHypothesisSet
TradeThesis
EntryThesis
OrderIntent
TradeResult
```

これらは処理責任ではなくData Object。

## Views

```text
Case Library
Market Memory
Failure Museum
Knowledge Graph
Human Market Diary
```

## Experiment Modes / Tools

```text
Random Baseline
Historical
Replay
Paper
Demo Forward
Shadow
Counterfactual
Stress Lab
Quantum / Search
```

- Stress Lab = Validation Tool
- Quantum / Search = Research Tool

独立必須通過Layerへしない。

## AI Team / AI Meeting

Legacy候補。

現行方針ではExternal AI Reviewへ置換する。

---

# 7. Role間の主要責任境界

## Collector vs Normalizer

```text
Collector = 取る
Normalizer = 揃える
```

## Normalizer vs Data Quality

```text
Normalizer = 形式変換
Data Quality = 信頼可能性評価
```

## Feature vs Feature Priority

```text
Feature = 測る
Priority = 今何を見るべきか決める
```

## Market Intelligence vs Causal Engine

```text
MI = 何が起きているか
Causal = なぜ起きている可能性があるか
```

## Causal Engine vs Research

```text
Causal = 何を検証すべきか
Research = 実際に検証する
```

## Market DNA vs Feature Priority

```text
DNA = 今はどんな市場か
Priority = 今は何を見る価値が高いか
```

## Research vs Production

```text
Research = Candidate / Evidence / Edgeを作る
Production = Approved KnowledgeだけでRiskを取る
```

## Signal vs Defense

```text
Signal = 取りたいか
Defense = 取ってよいか
```

## Execution Logic vs Exchange Adapter

```text
Execution = どう注文するか
Adapter = 取引所へどう送るか
```

## Supervisor vs In-Trade Defense vs Exit

```text
Supervisor = Thesisは生きているか
In-Trade Defense = Hard Safety違反か
Exit Engine = 通常決済するか
```

## Outer vs Runtime

```text
Outer = Desired State
Runtime = Actual State / Process execution
```

## Logger vs Analyzer

```text
Logger = 事実を保存
Analyzer = 意味を分析
```

## Analyzer vs Research Router

```text
Analyzer = 何が起きたか分解
Router = どの研究へ送るか
```

---

# 8. Roleが生成してよいResearch Candidate

各Roleは本来責務を超えて自分自身を研究しない。

ただしProcessing Contractの副産物としてResearchCandidateを出せる。

例:

```text
Collector → recurring source latency
Data Quality → recurring missing pattern
Feature → unstable measurement
Feature Priority → wrong priority candidate
Market Intelligence → unexplained move
Causal → strong alternative hypothesis
Market DNA → novel regime candidate
Defense → possible over-blocking
Execution → slippage anomaly
Supervisor → oversensitivity candidate
Monitoring → resource growth anomaly
```

流れ:

```text
Role
→ ResearchCandidate
→ Research Queue / Router
→ Research
```

Role自身が自分のProduction Ruleを自動書き換えない。

---

# 9. Role Failureの共通原則

各RoleはFailure時に少なくとも次を出せる設計候補とする。

```text
role_id
module
market_id
trace_id
error_id
severity
quality_impact
confidence_impact
runtime_impact
recommended_state
research_candidate
```

Failureの結果として許可される方向:

```text
RUNNING
DEGRADED
PAUSED
ERROR
RECOVERY
STOPPED
```

FailureしたRoleが推測値を作って正常処理を装わない。

---

# 10. Roleの長期運用原則

何十年以上運用するため、全Roleに次を適用する。

1. Provider名をRole責任へ埋め込まない
2. Object Schema Versionを追跡できる
3. Formula / Config Versionを追跡できる
4. Input / Output Traceを残せる
5. Role内部実装を将来別言語へ置換可能にする
6. KnowledgeとCodeを分離する
7. Resource Budgetを持てる
8. Failure / Recoveryを定義できる
9. Deprecated Roleを即削除せずMigration可能にする
10. Roleの責任変更はDecision履歴を残す

---

# 11. 新Role追加Gate

新しい案が出た時、次を順番に確認する。

```text
Q1. 既存Roleの責任に入らないか？
Q2. Existing RoleのSubmoduleで十分ではないか？
Q3. 新Object追加だけで解決しないか？
Q4. Contract変更だけで解決しないか？
Q5. View / Mode / Toolではないか？
Q6. 独立したInput / Output / Failure責任があるか？
Q7. 何十年維持する価値があるか？
```

Q1〜Q5で解決できるなら原則新Top-Level Roleを追加しない。

---

# 12. Role変更ルール

Role定義変更は `DESIGN_CHANGE_RULES.md` に従う。

特に次はMAJOR変更候補:

- Responsibility移動
- Prohibition解除
- Input / Output意味変更
- Production権限追加
- Research / Production境界変更
- Safety責任変更

Role名だけ変更して旧責任を失わないようMigration / Alias / Archiveを検討する。

---

# 13. Role Dictionary Definition of Done

Roleを正式定義済みとする最低条件:

```text
□ Role ID
□ Name
□ Category
□ Purpose
□ Inputs
□ Outputs
□ Responsibilities
□ Prohibitions
□ Upstream / Downstream
□ Failure Behavior
□ Research Feedback
□ Long-Term Notes
□ Object / Contractとの境界
□ 他Roleとの責任重複確認
```

---

# 14. 今後の詳細設計との関係

本書はRole責任の辞書。

今後、各Roleの内部アルゴリズム・State・Schema・Python Moduleを深掘りする場合でも、本書を上位参照する。

例:

```text
ROLE_DICTIONARY.md
↓
DOMAIN_RESPONSIBILITY.md
↓
DATA_CONTRACT.md
↓
各Role詳細設計
↓
PYTHON_ARCHITECTURE.md
↓
.py実装
```

下位設計が本辞書へ違反した場合、下位を修正するか、正式なDesign Changeとして本辞書を変更する。

---

# 15. 正式Object名の利用原則

RoleのInputs / OutputsでPersistent Objectを指す場合、`OBJECT_DICTIONARY.md` の正式Object名を使用する。

Legacy /曖昧表現:

```text
FeatureResult
Trial Result
ResearchTrialResult
Validation Result
Research Test Request
CausalEdge / EmpiricalEdge（別Object名としての使用）
```

新規設計での正式表現:

```text
Feature
ResearchResult
ResearchCandidate
Edge + edge_type
```

Historical / OOS / Regime / Demo / Stress等の研究結果は、別Result Object名を増やさず、`ResearchResult` の `evidence_source_channel` / `experiment_mode` / Trial参照で区別する。

---

# 16. 現段階の基本思想

市場理解OSでは、機能数を増やすことを完成度としない。

完成度を高める基準は、

> **各Roleが自分の仕事だけを正確に行い、責任境界を越えず、失敗時にも追跡でき、研究結果を安全に次へ戻せること。**

とする。

新しい案はまず「新Roleか」ではなく、

```text
既存Role
Submodule
Object
Contract
View
Experiment Mode
Governance Function
```

のどこへ置くべきか判定する。

これにより市場理解OSが長期間の設計変更で肥大化・重複・責任衝突することを防ぐ。
