# 市場理解OS まとめ案 7

# POST-TRADE / FEEDBACK / 共通契約

## 1. 目的

市場理解OSはTradeを実行して終わりではない。

Trade後に、

- 何が正しかったか
- 何が間違っていたか
- どこから間違いが始まったか
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

Logger / Analyzer / Failure Classification / Counterfactual / Research Routerを全部独立巨大Layerにせず、一つの分析Domainとして整理する。

```text
Trade Result
↓
Logger
↓
Post-Trade Analysis
│
├─ Outcome Analysis
├─ Hypothesis Evaluation
├─ Feature / Formula Evaluation
├─ Signal Evaluation
├─ Defense Evaluation
├─ Execution Evaluation
├─ Position Supervisor Evaluation
├─ Counterfactual Evaluation
└─ Research Router
```

---

# 3. Logger

Loggerは事実記録を担当する。

候補:

- Trade ID
- Trace ID
- Market Event IDs
- Entry Thesis
- Entry time / price
- Exit time / price
- Position size
- Fees
- Slippage
- Realized PnL
- MAE / MFE
- Signal state
- Defense result
- Execution state
- Position Supervisor state history
- Exit reason
- Data Quality
- Formula / Feature Version
- Market DNA
- Causal Hypothesis

Logger自身は「なぜ負けた」を確定しない。

---

# 4. Outcome Analysis

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

# 5. Hypothesis Evaluation

「Tradeが勝った = 仮説が正しい」ではない。

例えば、因果仮説は間違っていたが偶然Market全体が上昇し利益になった可能性がある。

逆に仮説は正しかったがExecution failureで損失になる可能性もある。

したがって、

```text
Trade Outcome
Hypothesis Outcome
Execution Outcome
Risk Outcome
```

を分離する。

---

# 6. Defense Evaluation

DefenseがBLOCKした取引はShadow Tradeで追跡する。

研究:

- BLOCK後に損失になった割合
- BLOCK後に利益になった割合
- Constraint別性能
- Regime別性能
- Data Quality別性能

目的:

> Defenseが安全性を上げているか、それとも機会を潰しすぎているかを測る。

---

# 7. Position Supervisor Evaluation

Supervisorの助言も研究対象にする。

例:

```text
THESIS_WEAKENING発生
↓
実際はHOLD
↓
その後 +1.5R
```

なら警告過敏の可能性。

逆に、

```text
THESIS_WEAKENING発生
↓
HOLD
↓
その後 -2R
```

なら助言価値がある可能性。

警告に従ったCaseだけで評価せず、従わなかったCaseも追う。

---

# 8. Counterfactual

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

Counterfactualを「未来が確実に分かった結果」と誤解しない。あくまで後知恵を利用した評価補助であり、本番で利用可能だった情報との分離が必要。

---

# 9. Research Router

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

Causal issue
→ Causal Research

DNA mismatch
→ Market DNA Research

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

# 10. Processing Contract

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

例:

Data QualityはQuality判定だけ行うが、「特定Sourceで同じ遅延が繰り返されている」というResearch Candidateを残せる。

Collector自身がCollectorを改善することは禁止。

---

# 11. Trace / Provenance

RawからTrade結果まで一本で追跡可能にする。

例:

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
→ Market DNA
→ Signal
→ Defense
→ Execution
→ Supervisor
→ Exit
→ Outcome
→ Research Result
```

目的:

> 「この損失はどこから始まったか」を上流へ戻れること。

---

# 12. Market Event IDとTrace IDを分ける

```text
EVENT-xxxx
= 市場で発生した現象

TRACE-xxxx
= OSがその現象や判断をどう処理したか
```

同一Eventから複数Traceが派生する可能性を許す。

---

# 13. Uncertainty Propagation

値だけを下流へ渡さない。

```text
value
quality
confidence
uncertainty
```

を必要に応じて保持する。

重要:

上流Data Qualityが低いのに下流で突然99% Confidenceになる構造を避ける。

ただしConfidenceを全て単純掛け算する等の式は現時点で固定しない。Formula Research対象とする。

---

# 14. Research Candidate

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

Market DNA
→ novel regime candidate

Defense
→ possible over-blocking

Supervisor
→ possible warning oversensitivity
```

各Module自身が研究を実行しない。

Research Queueへ送る。

---

# 15. Research Queue

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

Priority計算式は後から研究する。

---

# 16. Market Journal

Researchと人間理解のために日次記録を持つ案。

## Machine Journal
正本。

候補:

- Date
- Market Profile
- Market DNA
- Feature Priority
- Major Events
- Causal Hypotheses
- Signals
- Defense actions
- Trades
- Supervisor states
- Research Candidates
- Research Results
- Unexpected events

## Human Market Diary
Machine Journalから日本語生成する表示用。

Human Diaryを直接Machine Learningの正本にしない。

将来的に品質が高くなれば市場レポートとして副次的価値を持つ可能性はあるが、初期目的は内部研究。

---

# 17. Feedback Loopの完成形

```text
Trade
↓
Logger
↓
Post-Trade Analysis
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
Production Knowledge
↓
次のTrade
```

これにより市場理解OSを「予測して売買するBOT」ではなく「観測・研究・実行・再研究する循環型システム」にする。
