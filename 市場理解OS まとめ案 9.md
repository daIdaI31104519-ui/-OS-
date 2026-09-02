# 市場理解OS まとめ案 9

# 未確定事項・固定候補・本設計前チェック

## 0. 文書の目的

このファイルは「完成した設計」ではない。

まとめ案1〜8および追加整理案を本格的な設計MDへ昇格する前に、

- 固定してよい原則
- まだ深掘りが必要な部分
- Legacyとして除外する案
- 責任衝突を再確認する部分

を管理するためのチェック文書。

---

# 1. 現時点で固定候補にしてよい原則

## FIX-CANDIDATE-001
市場理解OSの現在対象は仮想通貨FX。

BTC専用実装ではなく、Market ProfileでBTC / ETH / SOL等を差し替え可能にする。

## FIX-CANDIDATE-002
OuterとCoreを分離する。

- Outer = 接続・設定・起動・資金上限
- Core = 市場理解・研究・取引判断

## FIX-CANDIDATE-003
Raw Dataは必ず保存し、Featureだけを正本にしない。

## FIX-CANDIDATE-004
Calculation / Measurementは共通基盤型。

各層が勝手に同じ指標を別Formulaで計算しない。

## FIX-CANDIDATE-005
Feature PriorityはMarket DNAと分離する。

- DNA = 今はどんな市場か
- Priority = 今は何を見るべきか

## FIX-CANDIDATE-006
Market IntelligenceとCausal Engineを分離する。

- MI = 何が起きているか
- Causal = なぜ起きている可能性があるか

## FIX-CANDIDATE-007
ResearchとProductionを分離する。

Research結果を直接Productionへ反映しない。

## FIX-CANDIDATE-008
Researchは大きく、Live Pathは小さくする。

## FIX-CANDIDATE-009
SignalとDefenseを分離する。

- Signal = 取りたいか
- Defense = 取ってよいか

## FIX-CANDIDATE-010
Entry後にPosition Supervisorを置く。

Supervisorは「価格」ではなくEntry Thesis / Trade Thesisの健全性を監督する。

## FIX-CANDIDATE-011
Trade後は原因別にResearchへ戻す。

単純な「負けた→Trainer」は採用しない。

## FIX-CANDIDATE-012
各処理はPrimary ResultだけでなくDiagnostics / Quality / Uncertainty / Trace / Research Candidateを残す。

## FIX-CANDIDATE-013
Python Runtime / Telegram / Monitoring等を市場理解ロジックから分離する。

## FIX-CANDIDATE-014
Demo Forward TrialをExperimental Frameworkへ正式に含める。

Demo Forwardは、HistoricalとLiveの間で、仮説固定後の未来データを使う前向き検証として扱う。

## FIX-CANDIDATE-015
Random / Historical / Demo / Live Evidenceを混ぜない。

件数・勝率・EV等を単純合算しない。

## FIX-CANDIDATE-016
Productionでは単一Hypothesisだけを唯一の根拠に固定せず、Approved HypothesisからApplicable Hypothesis Setを構成できるようにする。

## FIX-CANDIDATE-017
複数Hypothesisは多数決しない。

Shared Evidence / Dependence / Redundancy / Contradictionを確認し、Trade Thesisへ整理する。

## FIX-CANDIDATE-018
Entry前にTrade Thesisを固定し、Entry後の理由後付けを禁止候補とする。

---

# 2. Legacy / 廃止候補

## LEGACY-001 AI Team
複数AIが市場判断を自律協議する構造は現段階では採用しない。

## LEGACY-002 AI Meeting
AI多数決・会議を本番判断の中心に置かない。

AIはAI Reviewとして独立査読へ変更する。

## LEGACY-003 Quantum Layer必須通過
QuantumはResearch Toolへ変更。

## LEGACY-004 Stress Lab = Research全体
Stress LabはValidation Toolの一つへ変更。

## LEGACY-005 Trainer中心Loop
Research Routerによる原因別再研究へ変更。

## LEGACY-006 Single Hypothesis Only Production
「1つの強いHypothesisだけで本番Tradeする」を固定原則にしない。

複数仮説を利用する場合も票決ではなくTrade Thesisへ構造化する。

---

# 3. 独立Layerにしない方がよい候補

以下は機能として残すが、独立Layer乱立を避ける。

## Experimental Framework
- Random Baseline
- Historical
- Replay
- Paper Trade
- Demo Forward Trial
- Shadow Trade
- Counterfactual

## Knowledge Domain Views
- Case Library
- Market Memory
- Failure Museum
- Knowledge Graph

## Post-Trade Analysis Components
- Outcome Analyzer
- Trade Thesis Evaluation
- Hypothesis Attribution
- Failure Classification
- Demo vs Live Evaluation
- Counterfactual Evaluator
- Research Router

## Hypothesis Set / Trade Thesis
新しい巨大Layerにせず、ResearchとProduction間の正式なData Object / Contractとして設計する候補。

---

# 4. 本設計前に深掘りが必要な重要項目

## TODO-001 Data Contract
各Objectの必須Field、Schema Version、Timestamp、ID、Quality、Trace等。

## TODO-002 Processing Contract
各Moduleの共通出力形式。

## TODO-003 ID体系
- Market Event ID
- Trace ID
- Observation ID
- Feature ID
- Formula ID
- Hypothesis ID
- Hypothesis Set ID
- Trade Thesis ID
- DNA ID
- Trade / Trial ID
- Research ID

の関係。

## TODO-004 Formula / Measurement Governance
Formulaの提案、検証、Version、承認、廃止手順。

## TODO-005 Feature Priority計算
Priorityを何で決めるか。単純Score化しすぎない仕組み。

## TODO-006 Causal Engine → Research Contract
どのHypothesisを、どの検証方式へ渡すか。

## TODO-007 Market DNA正式軸
Raw / Featureから各DNA軸をどう計算するか。

## TODO-008 Research Protocol
Candidate → Historical / Replay → OOS / Regime → Demo Forward → Validation → Approvalの正式状態遷移。

## TODO-009 Causal Edge / Empirical Edge
Edge Object定義、Applicability、Expiration、Version。

## TODO-010 Signal Contract
Signalが読むTrade Thesis、Gate、Expected Value、NO_TRADE理由。

## TODO-011 Defense Contract
Pre-Trade / In-Trade / Portfolio Riskの境界。

## TODO-012 Entry Thesis
Trade Thesis Snapshotとしての固定項目、Invalidation、Expected Horizon、Version。

## TODO-013 Position Supervisor
複数Hypothesisを含むTrade Thesisの観測対象、時間軸、助言閾値、Hard Safetyとの境界。

## TODO-014 Post-Trade Attribution
「各Hypothesis」「Trade Thesis」「Signal」「Defense」「Execution」「市場変化」のどこが原因だったかをどう判定するか。

## TODO-015 Research Priority
Research Candidateが大量発生した場合の優先順位。

## TODO-016 Knowledge Object
Case / Failure / Constraint / Negative KnowledgeをどうDBで持つか。

## TODO-017 Market Journal
Machine JournalのSchemaとHuman Diary生成規則。

## TODO-018 Boot Sequence
どの`.py`から起動し、どの依存関係を確認するか。

## TODO-019 Telegram Security
権限、確認操作、危険Command、Audit Log。

## TODO-020 Multi-Crypto
BTCからETH / SOLへMarket Profileだけでどこまで差し替え可能かを実証する。

## TODO-021 Demo Forward Contract
- T0定義
- 未来データのみを使う保証
- Trial開始条件
- Trial終了条件
- Fee / Spread / Slippage / Latency model
- Demo Adapter
- Live Logicとの共通範囲

を正式化する。

## TODO-022 Evidence Channel Contract
Historical / OOS / Demo / Stress / Liveをどう保存し、どう比較し、どこまで統合可能かを決める。

## TODO-023 Hypothesis Set Contract
- PRIMARY
- SUPPORTING
- CONDITIONAL
- CONTRADICTING
- Shared Evidence
- Dependence
- Redundancy

の定義。

## TODO-024 Trade Thesis Contract
Direction / Expected Effect / Expected Horizon / Expected Value / Invalidation / Main Risk / Hypothesis Set Version等を正式定義する。

## TODO-025 Hypothesis Combination Research
組み合わせ爆発と過学習を防ぐため、最大Hypothesis数、事前登録、OOS / Demo必須条件等を決める。

## TODO-026 Demo vs Live Attribution
Demo PASS / Live FAIL等の差をExecution / Slippage / Regime / Demo model / Sample不足へ分解する方法を決める。

## TODO-027 Market Event重複補正
同一EVENT内の複数Trialを独立サンプルとして数えない研究ルールを決める。

---

# 5. 今は後回しにする項目

## HOLD-001
株式・為替・Commodityへの本格対応。

## HOLD-002
AIによる自動Market選択。

## HOLD-003
AIによる自動資金配分。

## HOLD-004
AI同士の自律会議。

## HOLD-005
Productionで毎回Quantumを実行。

## HOLD-006
Human Diaryを直接学習データとして使用。

## HOLD-007
各Layerが自分自身のコードを自動修正・本番反映。

## HOLD-008
AI Reviewがその場で新しい未検証Hypothesisを生成し、現在のTrade Thesisへ直接追加すること。

---

# 6. 本設計MDへ進む際の推奨順序

まとめ案をさらに絞った後、正式設計は概ね次の順を候補とする。

```text
1. Project Constitution / Principles
2. System Boundary
3. Architecture Overview
4. Domain Responsibilities
5. Data Flow
6. Data Contract
7. Database Schema
8. Research Protocol
9. Evidence Channel / Demo Forward Protocol
10. Hypothesis Set / Trade Thesis Contract
11. Production / Risk / Execution
12. Post-Trade Attribution
13. Runtime / Boot Sequence
14. Python Architecture
15. Monitoring / Recovery / Deployment
16. Telegram Interface
17. Test Strategy
18. Implementation Roadmap
```

---

# 7. Git運用上の扱い

この「まとめ案」シリーズは、まだCanonicalそのものではない。

次段階では、

```text
まとめ案
→ Conflict / Duplication Review
→ 採用項目選定
→ Canonical Proposal
→ 人間承認
→ 正式設計MD
```

とする。

古い情報源や会話案をそのままCanonicalへコピーしない。

矛盾する旧案はLegacy / Rejected / Historicalとして参照可能な状態を残し、現在設計と混ぜない。

---

# 8. この段階のゴール

市場理解OSを一度「完全に作り切る」ことではない。

今のゴールは、

> 本格設計へ入る前に、何がCoreで、何がResearchで、何がProductionで、何がOuter / Operationsなのかを曖昧にせず、さらにResearch Evidenceがどの段階にあり、ProductionがどのHypothesis Setを根拠にTradeしたかを後から完全に説明できる状態を作ること。

この境界が固定できれば、その後の深掘りで新機能が増えても適切なDomainへ配置できる。
