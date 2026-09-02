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
- 仮説固定後の未来データでもEdgeが残るかをDemo Forwardで確かめる
- どのRegimeで成立・崩壊するかを見る
- FailureBoundaryを発見する
- Negative Knowledgeを蓄積する
- 複数Hypothesisの組み合わせが本当に単独仮説より強いかを研究する

ために存在する。

---

# 2. Researchへの入力

Research対象はTrade失敗だけではない。

候補:

- Causal Hypothesis
- Alternative Hypothesis
- Contradiction
- Hypothesis Set Candidate
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
- Demo Forward failure / success
- Live vs Demo divergence
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
├─ Hypothesis Set Research
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
- Demo Forward

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
- Demo Forward
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
- DNAとHypothesis Setの関係

目的:

> Market DNAが「似ている」と判断した市場が、本当にResearchやTrade上も似ていたかを検証する。

さらに、

```text
DNA-A
→ H-101 + H-204 が強い

DNA-B
→ H-101単独が強い

DNA-C
→ H-407が強くSHORT Thesisは不適用
```

のように、どの市場状態でどのHypothesis Setが適用可能かも研究対象にする。

---

# 10. Hypothesis Set Research

Productionで複数仮説を使う場合、単純多数決は禁止候補とする。

研究対象:

- 各Hypothesisの単独性能
- Hypothesis間の独立性
- Shared Evidence
- 共通Causeの二重計上
- Supporting / Conditional / Contradicting関係
- 組み合わせ時のExpected Value
- Market DNA別の組み合わせ性能
- OOS / Demo Forwardでの再現性

例:

```text
H-101単独
EV +0.10

H-101 + H-204
EV +0.18

H-101 + H-331
EV +0.13

H-101 + H-204 + H-331
EV +0.27
```

ただし、組み合わせ探索を無制限にすると過学習と組み合わせ爆発が起きる。

そのため、

- 仮説数上限
- Shared Evidence確認
- 事前登録した組み合わせ
- HistoricalだけでなくOOS / Demo Forward確認

を正式設計候補とする。

---

# 11. Experimental Framework

Random / Historical / Replay / Paper / Demo Forward / Shadow / Counterfactualを別々の巨大Layerにしない。

一つの実験基盤でmodeとして扱う。

```text
mode = RANDOM_BASELINE
mode = HISTORICAL
mode = REPLAY
mode = PAPER
mode = DEMO_FORWARD
mode = SHADOW
mode = COUNTERFACTUAL
```

## Random Baseline
戦略がRandomより本当に優れているかを比較する。仮説の種や比較基準を得る。

## Historical
過去Caseで仮説・Edge・Hypothesis Setの再現性を調べる。

## Replay
過去市場を時間順に再生し、当時知り得た情報だけで検証する。

## Paper Trade
実資金を使わず仮想取引する一般的な実験mode。

## Demo Forward
仮説やHypothesis Setを固定した後、T0以降に新しく到来する市場データだけで仮想取引する前向き検証。

## Shadow Trade
Defense等でBlockした取引、またはProduction未採用候補を仮想追跡。

## Counterfactual
実際とは別判断をした場合の結果を比較。

---

# 12. Demo Forward Trialの正式な役割

Demo ForwardはHistoricalとLiveの中間証拠。

```text
仮説作成 T0
──────────────
← Historical / Replay
→ Demo Forward
```

目的:

- 過去データから発見したEdgeの過学習を早期発見する
- Real Tradeが少なくても研究サンプルを増やす
- 複数候補をProduction前に絞る
- Position SupervisorやExit規則も実資金なしで前向き検証する

候補フロー:

```text
Hypothesis Pool
↓
Historical / OOS
↓
Demo Forward Trial
↓
Evidence Package
↓
Production Candidate
```

DemoはReal Tradeの代替ではない。

Real Tradeは実約定・手数料・Slippage・Partial Fill・API・流動性等を含む別種のEvidenceとする。

---

# 13. DemoとLiveのロジックを分岐させすぎない

比較可能性を保つため、Signal以降の判断ロジックは可能な限り共通化する。

```text
Signal
↓
Defense
↓
Execution Logic
↓
Entry Thesis
↓
Position Supervisor
↓
Exit
        │
        ├→ Demo Execution Adapter
        └→ Real Exchange Adapter
```

原則:

> 頭脳は同じ。実行先だけ違う。

Demo専用の都合の良いEntry / Exitロジックを作らない。

Demo側でも可能な限り、

- Fee
- Spread
- Slippage
- Orderbook depth
- Latency
- Funding
- Partial Fill model

を反映する。

---

# 14. Evidence Sourceを絶対に混ぜない

例えば、

```text
Random 3000件
Historical 4200件
Demo Forward 420件
Live 28件
```

を「合計7648件、勝率XX%」としてまとめない。

Hypothesis / Hypothesis Setごとに証拠源を分離する。

```text
Historical:
  n = 4200
  EV = +0.18

OOS:
  n = 900
  EV = +0.15

Random Baseline:
  n = 3000
  EV = -0.01

Demo Forward:
  n = 420
  EV = +0.16

Live:
  n = 28
  EV = +0.11
```

それぞれの意味・信頼性・市場摩擦が違うため、別Evidence Channelとして扱う。

---

# 15. Market Event単位の重複に注意

同じBTC急落で、

```text
H-101成功
H-102成功
H-103成功
```

しても、独立した3つの市場現象とは限らない。

必ずMarket Event IDへ紐付ける。

```text
EVENT-9001
├─ H-101 Trial
├─ H-102 Trial
└─ H-103 Trial
```

Evidence数を水増ししない。

---

# 16. Shadow Production

ProductionでH-101だけ採用している場合でも、未採用の有望候補をDemoで並行追跡できる。

```text
Real:
H-101 → Live

Shadow Demo:
H-102
H-103
H-104
```

一つの現在市場から複数候補を前向き研究できる。

ただしLiveの意思決定へ未承認候補を直接混ぜない。

---

# 17. Random Baselineの役割

Random Tradeは利益を狙わない。

目的:

> 研究戦略が「たまたま市場が上がった」だけではないかを確認する比較対象。

Random / DiscoveryとDemo Forwardは役割が異なる。

```text
Random
= 発見・比較

Demo Forward
= 固定した仮説の未来検証
```

---

# 18. Shadow / Counterfactualの重要性

実際に行った判断だけ研究するとSelection Biasが発生する。

DefenseでBlockしたTrade、Supervisor警告に従わなかったCase等も追跡する。

Counterfactualは未来を知っていたものとして本番評価に使わず、後知恵を利用したResearch補助とする。

---

# 19. Validation Framework

Research結果をProductionへ持っていく前に最低限次を検討する。

## Historical Validation
過去Caseで成立するか。

## OOS
研究に利用していない期間で成立するか。

## Regime Validation
Bull / Bear / Range / High Vol / Low Liquidity等で安定するか。

## Demo Forward
仮説固定後の未来データでも成立するか。

## Stress Lab
成立しそうな仮説・Edgeを意図的に壊し、限界を調べる。

全仮説に完全に同じ検証順を強制するかは本設計で決める。

---

# 20. Stress Labの位置づけ

Stress LabはResearch全体そのものではない。

成果物:

```text
StressResult
→ FailureBoundary
→ Constraint
```

Stress結果はHypothesis / Edge / Hypothesis SetのApplicabilityへ戻す。

---

# 21. Quantum / Search

Quantumは必須通過Layerにしない。

Researchが必要に応じて使う探索Tool。

用途候補:

- Feature selection
- Combination search
- Parameter search
- Hypothesis Set候補探索
- Scenario search
- Clustering補助
- Similarity optimization

探索で見つかった組み合わせをそのままProductionへ入れない。

```text
Search
→ Candidate
→ Historical / OOS
→ Demo Forward
→ Approval
```

を通す。

---

# 22. Research Priority

Research Candidateは大量発生するためQueueを持つ。

Priority要素候補:

- Impact
- Frequency
- Uncertainty
- Novelty
- Potential PnL
- Potential Loss / Tail Risk
- Reproducibility
- Data availability
- Demo Forward cost

Priority Formulaは最初から固定しない。

---

# 23. Researchの成果物

ResearchはTrade Signalそのものではなく、次を返す。

- Approved / Candidate Hypothesis
- Hypothesis Set Candidate
- Causal Edge
- Empirical Edge
- Feature Knowledge
- Formula Knowledge
- Market DNA Knowledge
- FailureBoundary
- Constraint
- Negative Knowledge
- Evidence Package
- Applicability
- Uncertainty
- Candidate for Approval

---

# 24. Causal Edge / Empirical Edge

## Causal Edge
なぜ効く可能性があるかについて、因果仮説とEvidenceが一定水準ある。

## Empirical Edge
原因説明は完全でなくても、Historical / OOS / Demo Forward / Regime等で統計的再現性がある。

因果説明が完全でないという理由だけで、再現性のあるEmpirical Edgeを捨てない。

---

# 25. ResearchからProductionへの昇格

候補フロー:

```text
Discovery / Candidate
→ Historical / Replay
→ OOS / Regime
→ Demo Forward
→ Stress / Constraint（必要時）
→ Evidence Package
→ Approval
→ Approved Knowledge / Hypothesis / Edge
→ Production
```

Research自身が本番設定を直接変更することは禁止する。

Live EvidenceはProduction後にResearchへ戻し、DemoとLiveの差も研究対象にする。
