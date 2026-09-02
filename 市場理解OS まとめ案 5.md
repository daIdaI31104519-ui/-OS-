# 市場理解OS まとめ案 5

# RESEARCH DOMAIN

## 1. Researchの位置づけ

Researchは市場理解OSの中で最も大きくてもよい領域の一つ。

ただし本番売買経路へ直接複雑さを持ち込まない。

基本思想:

> Researchは巨大でよい。Live Pathは小さくする。

Researchは、

- 市場の理解を深める
- Causal Hypothesisを検証する
- Feature / Formulaの有効性を調べる
- Market DNAの軸・類似性を改善する
- Edgeが本当にRandomより優れているかを確かめる
- どのRegimeで成立・崩壊するかを見る
- FailureBoundaryを発見する
- Negative Knowledgeを蓄積する

ために存在する。

---

# 2. Researchへの入力

Research対象はTrade失敗だけではない。

候補:

- Causal Hypothesis
- Alternative Hypothesis
- Contradiction
- Feature Candidate
- Formula Candidate
- Market DNA Candidate
- Unexplained Event
- Novel Regime Candidate
- Unexpected Failure
- Unexpected Success
- Missed Opportunity
- Defense Block
- Position Supervisor Warning
- Execution anomaly
- Data Quality anomaly
- Research Candidate

---

# 3. Researchを一本の直列Layerにしない

Research Domain内に複数の研究方法を持つ。

```text
Research Domain
│
├─ Observation Research
├─ Pattern Research
├─ Feature Research
├─ Formula Research
├─ Causal Research
├─ Market DNA Research
├─ Experimental Framework
├─ Validation Framework
└─ Search / Optimization Tools
```

---

# 4. Observation Research

実際の市場で自然に起きた現象を研究する。

例:

- 金曜日にBTCはどう動くか
- 米国時間だけ結果が変わるか
- FOMC前後のFunding / OI構造
- ETF強流入時のPrice / CVD / OI関係
- News Event後のLag

単一条件だけで結論を出さず、条件付き関係を調べる。

例:

```text
金曜日
↓
平均Return -0.2%

金曜日 + ETF強流入
↓
平均Return +0.3%
```

---

# 5. Pattern Research

大量Caseから「条件の組み合わせ」を探す。

研究対象例:

```text
曜日
× 時間帯
× Funding
× OI
× ETF
× CVD
× Liquidity
× Regime
```

目的は単純な相関ランキングではなく、

> どの条件下で関係が出現・反転・消失するか

を見ること。

---

# 6. Feature Research

Featureごとに、

- どのTargetで効くか
- どのHorizonで効くか
- どのMarket DNAで効くか
- どのRegimeで効くか
- どの時間帯・曜日で効くか
- Data Quality低下時にどう壊れるか
- 他Featureとの重複はあるか

を検証する。

成果はFeature Priority Registryへ反映候補として戻す。

---

# 7. Formula Research

「何を測るか」だけでなく「どう測るか」を研究する。

例:

```text
Liquidity Score A = Spread + Depth
Liquidity Score B = Spread × Depth
Liquidity Score C = nonlinear(Spread, Depth, Slippage)
```

比較対象:

- Historical
- OOS
- Regime
- Sensitivity
- Stability
- Stress

正式Formulaへ昇格する場合はFormula RegistryのVersionを更新する。

---

# 8. Causal Research

Causal Engineが定義した検証要求を実行する。

研究内容:

- Temporal Order確認
- Lag推定
- Confounder検証
- Alternative Hypothesis比較
- Contradiction収集
- Historical Case検証
- OOS
- Regime Stability
- Stress条件

Causal Researchは「原因を証明した」と安易に確定しない。

SUPPORTED / WEAK / RETIRED等のLifecycle判断材料を返す。

---

# 9. Market DNA Research

研究対象:

- DNA軸の有効性
- 軸の重複
- 重み
- Similarity計算
- Novelty計算
- Case Retrieval精度
- DNAとFeature Priorityの関係
- DNAとEdge安定性の関係

目的:

> Market DNAが「似ている」と判断した市場が、本当にResearchやTrade上も似ていたかを検証する。

---

# 10. Experimental Framework

Random / Replay / Paper / Shadow / Counterfactualを別々の巨大Layerにしない。

一つの実験基盤でmodeとして扱う。

```text
mode = RANDOM_BASELINE
mode = PAPER
mode = REPLAY
mode = SHADOW
mode = COUNTERFACTUAL
```

## Random Baseline
戦略がRandomより本当に優れているかを比較する。

## Paper Trade
実資金を使わず現在市場で仮想取引。

## Replay
過去市場を時間順に再生して当時知り得た情報だけで検証。

## Shadow Trade
Defense等でBlockした取引を仮想追跡。

## Counterfactual
実際とは別判断をした場合の結果を比較。

---

# 11. Random Baselineの役割

Random Tradeは利益を狙わない。

目的:

> 研究戦略が「たまたま市場が上がった」だけではないかを確認する比較対象。

例:

```text
Strategy PF = 1.32
Random PF = 1.01
```

と、

```text
Strategy PF = 1.04
Random PF = 1.02
```

では研究価値が違う。

---

# 12. Shadow / Counterfactualの重要性

実際に行った判断だけ研究するとSelection Biasが発生する。

例:

```text
Signal = BUY
Defense = BLOCK
Real Trade = NONE
```

Shadowでその後を追跡し、Defense Blockの正しさを研究する。

Position Supervisorについても、

```text
THESIS_WEAKENINGを出した
↓
Exitしなかった
↓
その後 +1.8R
```

なら、警告が過敏だった可能性を研究できる。

---

# 13. Validation Framework

Research結果をProductionへ持っていく前に最低限次を検討する。

## Historical Validation
過去Caseで成立するか。

## OOS
研究に利用していない期間で成立するか。

## Regime Validation
Bull / Bear / Range / High Vol / Low Liquidity等で安定するか。

## Stress Lab
成立しそうな仮説・Edgeを意図的に壊し、限界を調べる。

---

# 14. Stress Labの新しい位置づけ

Stress LabはResearch全体そのものではない。

位置:

```text
Hypothesis / Edge
→ Historical
→ OOS
→ Regime
→ Stress Lab
```

Stress対象候補:

- Flash Crash
- Liquidity collapse
- Spread expansion
- Extreme volatility
- News shock
- API latency
- Data degradation
- Correlation breakdown
- Leverage unwind

成果物:

```text
StressResult
→ FailureBoundary
→ Constraint
```

---

# 15. Quantum / Search

Quantumは必須通過Layerにしない。

Researchが必要に応じて使う探索Tool。

用途候補:

- Feature selection
- Combination search
- Parameter search
- Scenario search
- Clustering補助
- Similarity optimization

探索で見つかった結果をそのままProductionへ入れない。

```text
Quantum / Search
→ Candidate
→ Standard Validation
→ Approval
```

を通す。

---

# 16. Research Priority

Research Candidateは大量発生するため、Queueを持つ。

Priority要素候補:

- Impact
- Frequency
- Uncertainty
- Novelty
- Potential PnL
- Potential Loss / Tail Risk
- Reproducibility
- Data availability

ただしPriority Formulaは最初から固定しない。Calculation / Formula Researchで改善する。

---

# 17. Researchの成果物

ResearchはTrade Signalそのものではなく、次を返す。

- Causal Edge
- Empirical Edge
- Feature Knowledge
- Formula Knowledge
- Market DNA Knowledge
- FailureBoundary
- Constraint
- Negative Knowledge
- Research Result
- Applicability
- Uncertainty
- Candidate for Approval

---

# 18. Causal Edge / Empirical Edge

## Causal Edge
なぜ効く可能性があるかについて、因果仮説とEvidenceが一定水準ある。

## Empirical Edge
原因説明は完全でなくても、Historical / OOS / Regime等で統計的再現性がある。

因果説明が完全でないという理由だけで、再現性のあるEmpirical Edgeを捨てない。

最終的なProductionでは「因果の美しさ」ではなく、期待値・適用条件・Risk・再現性を重視する。

---

# 19. ResearchからProductionへの昇格

```text
Research
→ Candidate
→ Validation
→ Approval
→ Approved Knowledge / Edge
→ Production
```

Research自身が本番設定を直接変更することは禁止する。
