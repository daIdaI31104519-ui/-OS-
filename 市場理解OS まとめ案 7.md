# 市場理解OS まとめ案 7

# POST-TRADE / FEEDBACK / 共通契約

## 1. 目的

市場理解OSはTradeを実行して終わりではない。

Trade後に、

- 何が正しかったか
- 何が間違っていたか
- どのHypothesisが結果に寄与したか
- Trade Thesis全体は妥当だったか
- どこから間違いが始まったか
- Demo ForwardとLiveで何が違ったか
- 別判断ならどうなったか
- どの研究領域へ戻すべきか

を判定し、次の研究へ戻す。

旧来の、

```text
Trade
→ Logger
→ Trainer
```

だけの構造は採用しない。

---

# 2. Post-Trade Analysis Domain

Logger / Analyzer / Hypothesis Attribution / Failure Classification / Counterfactual / Research Routerを全部独立巨大Layerにせず、一つの分析Domainとして整理する。

```text
Trade Result / Demo Result
↓
Logger
↓
Post-Trade Analysis
│
├─ Outcome Analysis
├─ Trade Thesis Evaluation
├─ Hypothesis Attribution
├─ Feature / Formula Evaluation
├─ Signal Evaluation
├─ Defense Evaluation
├─ Execution Evaluation
├─ Position Supervisor Evaluation
├─ Demo vs Live Evaluation
├─ Counterfactual Evaluation
└─ Research Router
```

---

# 3. Logger

Loggerは事実記録を担当する。

候補:

- Trade / Trial ID
- Trace ID
- Market Event IDs
- Experiment Mode
- Evidence Source
- Trade Thesis ID
- Hypothesis Set Version
- Primary Hypothesis
- Supporting Hypotheses
- Conditional Hypotheses
- Contradicting Hypotheses
- Shared Evidence Map
- Entry Thesis
- Entry time / price
- Exit time / price
- Position size
- Fees
- Slippage
- Realized / Simulated PnL
- MAE / MFE
- Signal state
- Defense result
- Execution state
- Position Supervisor state history
- Hypothesis state history
- Exit reason
- Data Quality
- Formula / Feature Version
- Market DNA

Logger自身は「なぜ負けた」を確定しない。

---

# 4. Evidence Sourceを分離する

同じHypothesisでも証拠源を混ぜない。

最低候補:

```text
RANDOM
HISTORICAL
REPLAY
OOS
REGIME
DEMO_FORWARD
SHADOW
STRESS
LIVE
```

例えば、

```text
Historical 4200件
Demo Forward 420件
Live 28件
```

を単純に「4648件」として扱わない。

各Evidence Sourceは、

- データ生成過程
- Market friction
- Selection bias
- 実行現実性

が異なる。

---

# 5. Outcome Analysis

Trade結果を単純なWIN / LOSSだけで評価しない。

研究候補:

- Expected Valueとの差
- Expected Horizonとの差
- MAE / MFE
- Exit Timing
- Slippage
- Opportunity Cost
- Risk-adjusted outcome
- Similar Case comparison

分類例:

```text
EXPECTED_SUCCESS
UNEXPECTED_SUCCESS
EXPECTED_FAILURE
UNEXPECTED_FAILURE
MISSED_OPPORTUNITY
SYSTEM_FAILURE
```

---

# 6. Trade Thesis Evaluation

「Tradeが勝った = Trade Thesisが正しい」ではない。

例えば、Primary Hypothesisは外れていたが偶然別要因で利益になる可能性がある。

逆にTrade Thesisは妥当でもExecution failureで損失になる可能性もある。

したがって、

```text
Trade Outcome
Trade Thesis Outcome
Hypothesis Outcomes
Execution Outcome
Risk Outcome
```

を分離する。

---

# 7. Hypothesis Attribution

Trade Thesisを構成した各Hypothesisを個別評価する。

例:

```text
T-0042 LOSS

H-101 PRIMARY
→ mechanism supported

H-204 SUPPORTING
→ failed

H-331 CONDITIONAL
→ regime classification wrong

H-407 CONTRADICTING
→ actually strengthened before exit
```

これにより、

> Tradeは失敗したがH-101自体をRetireする必要はない

という判断が可能になる。

評価候補:

- expected_effect_occurred
- expected_horizon_met
- evidence_remained_valid
- contradiction_strength
- applicability_correct
- shared_evidence_issue
- dependency_issue
- contribution_to_trade_thesis

---

# 8. Hypothesis Set評価

単独仮説だけでなく組み合わせの有効性も評価する。

例:

```text
H-101単独
Demo EV +0.10

H-101 + H-204
Demo EV +0.18

Liveでは
H-101 + H-204 EV -0.02
```

この場合、

- Execution差
- Regime差
- Sample不足
- Shared Evidence過大評価
- Demo modelの甘さ

等をResearch Candidateにする。

組み合わせが良かったという理由だけで永久固定しない。

---

# 9. Demo vs Live Evaluation

非常に重要な分析。

候補状態:

```text
Historical = PASS
Demo Forward = PASS
Live = FAIL
```

なら、仮説を即Retireするのではなく、

```text
なぜLiveだけ崩れた？

Execution?
Slippage?
Spread?
Partial Fill?
Liquidity impact?
Latency?
Constraint不足?
Market regime shift?
Demo model error?
```

を研究する。

逆に、

```text
Historical = PASS
Demo Forward = FAIL
```

なら、Liveへ出す前に止められる可能性が高い。

---

# 10. Defense Evaluation

DefenseがBLOCKした取引はShadow Tradeで追跡する。

研究:

- BLOCK後に損失になった割合
- BLOCK後に利益になった割合
- Constraint別性能
- Regime別性能
- Data Quality別性能
- Trade Thesisタイプ別性能

目的:

> Defenseが安全性を上げているか、それとも機会を潰しすぎているかを測る。

---

# 11. Position Supervisor Evaluation

Supervisorの助言も研究対象にする。

単一Hypothesis警告だけでなく、Trade Thesis全体とContradicting Hypothesisの変化を追跡する。

例:

```text
PRIMARY = WEAKENING
CONTRADICTING = STRONG
Trade Thesis = THESIS_WEAKENING
↓
実際はHOLD
↓
その後 -2R
```

なら助言価値がある可能性。

逆に、警告後に元方向へ大きく進んだ場合は過敏性を疑う。

警告に従ったCaseだけで評価せず、従わなかったCaseも追う。

---

# 12. Counterfactual

反実仮想:

> 別の判断をしていたらどうなったか。

例:

```text
Real: HOLD → -1.2R
Alternative: EXIT → -0.3R
```

用途:

- Signal threshold研究
- Defense threshold研究
- Exit研究
- Supervisor研究
- Position sizing研究
- Hypothesis Set研究

Counterfactualを「未来が確実に分かった結果」と誤解しない。後知恵を利用した評価補助であり、本番で利用可能だった情報との分離が必要。

---

# 13. Market Event IDと疑似サンプル増加を防ぐ

同じMarket Eventから複数Hypothesis Trialが派生する場合、件数を独立サンプルのように水増ししない。

```text
EVENT-9001
├─ H-101 Demo Trial
├─ H-102 Demo Trial
├─ H-103 Demo Trial
└─ T-0042 Live Trade
```

この構造を保持する。

研究時には、

- Trial数
- Unique Market Event数

を必要に応じて分離する。

---

# 14. Research Router

損失・異常を一律Trainerへ送らない。

原因別Routing例:

```text
Data issue
→ Data Quality Research

Formula issue
→ Formula Research

Feature issue
→ Feature Research

Market interpretation issue
→ Market Intelligence Research

Single Hypothesis issue
→ Causal Research

Hypothesis Set issue
→ Hypothesis Set Research

DNA mismatch
→ Market DNA Research

Demo vs Live divergence
→ Experimental / Execution Research

Defense issue
→ Defense Research

Execution issue
→ Execution Research

Supervisor issue
→ Position Supervisor Research
```

成功もRouting対象。

Unexpected Successから未知Edgeを探す。

---

# 15. Processing Contract

各Moduleは本来の責務を増やさず、処理終了時に共通形式の副産物を返す。

最低候補:

```text
1. Primary Result
2. Diagnostics
3. Quality
4. Confidence / Uncertainty
5. Provenance / Trace
6. Research Candidate
```

Trial / Trade系では追加候補として、

```text
experiment_mode
evidence_source
trade_thesis_id
hypothesis_ids
market_event_ids
```

を持つ。

---

# 16. Trace / Provenance

RawからResearch Resultまで一本で追跡可能にする。

```text
TRACE-20260902-000043
```

追跡:

```text
Raw Data
→ Observation
→ Feature
→ Evidence
→ Hypothesis
→ Hypothesis Set
→ Trade Thesis
→ Signal
→ Defense
→ Execution
→ Supervisor
→ Exit
→ Outcome
→ Research Result
```

目的:

> 「この損失・Demo失敗・Live差異はどこから始まったか」を上流へ戻れること。

---

# 17. Market Event IDとTrace IDを分ける

```text
EVENT-xxxx
= 市場で発生した現象

TRACE-xxxx
= OSがその現象や判断をどう処理したか
```

同一Eventから複数Trace・Demo Trial・Hypothesis評価が派生する可能性を許す。

---

# 18. Uncertainty Propagation

値だけを下流へ渡さない。

```text
value
quality
confidence
uncertainty
```

を必要に応じて保持する。

上流Data Qualityが低いのに下流で突然99% Confidenceになる構造を避ける。

Hypothesisが3つあるからConfidenceを3倍にする等は禁止候補。

---

# 19. Research Candidate

各層はResearch Candidateを生成できる。

例:

```text
Collector
→ Source latency anomaly

Data Quality
→ recurring missing pattern

Feature
→ unstable feature

Market Intelligence
→ unexplained move

Causal
→ strong alternative hypothesis

Hypothesis Set
→ dependency / redundancy issue

Market DNA
→ novel regime candidate

Demo Forward
→ forward degradation

Live
→ demo-live divergence

Defense
→ possible over-blocking

Supervisor
→ possible warning oversensitivity
```

各Module自身が研究を実行しない。

Research Queueへ送る。

---

# 20. Research Queue

Candidate増加に備え、優先順位を管理する。

候補要素:

- Impact
- Frequency
- Uncertainty
- Novelty
- Potential Profit
- Potential Loss
- Tail Risk
- Reproducibility
- Demo / Live divergence severity

Priority計算式は後から研究する。

---

# 21. Market Journal

Researchと人間理解のために日次記録を持つ案。

## Machine Journal
正本。

候補:

- Date
- Market Profile
- Market DNA
- Feature Priority
- Major Events
- Active Hypothesis Pool
- Demo Forward Trials
- Production Trade Theses
- Signals
- Defense actions
- Trades
- Supervisor states
- Research Candidates
- Research Results
- Demo vs Live differences
- Unexpected events

## Human Market Diary
Machine Journalから日本語生成する表示用。

Human Diaryを直接Machine Learningの正本にしない。

---

# 22. Feedback Loopの完成形

```text
Discovery / Research
↓
Hypothesis Pool
↓
Historical / OOS
↓
Demo Forward
↓
Approval
↓
Applicable Hypothesis Set
↓
Trade Thesis
↓
Real Trade
↓
Logger
↓
Post-Trade Analysis
↓
Hypothesis Attribution
↓
Demo vs Live Evaluation
↓
Research Router
↓
Research
↓
Knowledge Candidate
↓
Validation
↓
Approval
↓
次のTrade
```

これにより市場理解OSを「予測して売買するBOT」ではなく「観測・研究・前向き検証・実行・比較・再研究する循環型システム」にする。
