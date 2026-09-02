# 市場理解OS まとめ案 6

# PRODUCTION / TRADING DOMAIN

## 1. 目的

ProductionはResearchの知識を使って、実際の売買候補を作り、安全に実行し、保有中もTrade Thesisの健全性を監督する。

基本原則:

> Researchは大きく、本番経路は小さくする。

Productionでは単一Hypothesisだけを唯一の根拠に固定せず、研究済みの複数Hypothesisから現在市場へ適用可能な集合を作り、一つのTrade Thesisとして扱う。

本番フロー:

```text
Approved Hypothesis / Edge Pool
+ Current Market Context
+ Market DNA
+ Feature Priority
        ↓
Applicable Hypothesis Set
        ↓
Trade Thesis
        ↓
AI Review（補助・任意）
        ↓
Signal Engine
        ↓
Pre-Trade Defense
        ↓
Execution Logic
        ↓
Order Intent
        ↓
Exchange Adapter
        ↓
Entry
        ↓
Entry Thesis固定
        ↓
Position Supervisor
        ↓
Exit / In-Trade Defense
        ↓
Trade Result / Production Evidence
```

---

# 2. AI Team / AI Meetingは採用しない

旧案の、

```text
AI Team
→ AI Meeting
→ Signal
```

はLegacy候補とする。

理由:

- AI同士が同じ誤りを共有する可能性
- データ不足時に推測を強化してしまう危険
- 多数決が正しさを保証しない
- 本番経路が重くなる
- AI停止がTrade停止へ直結しやすい

代わりにAI Reviewを補助機能として使う。

---

# 3. Hypothesis Pool

Researchを通過したHypothesis / EdgeをPoolとして保持する。

重要:

- CandidateとApprovedを混ぜない
- Retired / Weak / Hold状態を区別する
- Applicabilityを持たせる
- Market DNA / Regime / Horizonとの対応を持たせる
- Demo Forward / Live Evidenceの状態を分けて保持する

Productionが自由に未承認Hypothesisを作ったり追加したりしない。

---

# 4. Applicable Hypothesis Set

現在市場に適用可能なApproved Hypothesisを抽出する。

候補判定材料:

- Current Market DNA
- Regime
- Horizon
- Current Evidence
- Feature Priority
- Constraint
- Data Quality
- Historical / OOS
- Demo Forward
- Live Evidence
- Expiration / Staleness

出力は単なるBUY仮説一覧ではなく、役割を持つ集合とする。

役割候補:

```text
PRIMARY
= Trade Thesisの中心仮説

SUPPORTING
= 別メカニズムから中心仮説を支持

CONDITIONAL
= 仮説が成立しやすい条件・市場状態

CONTRADICTING
= 反対方向または中心仮説を弱める仮説
```

---

# 5. 仮説を多数決しない

禁止候補:

```text
3 BUY仮説
2 SELL仮説
↓
BUY
```

仮説の数はEvidenceの強さではない。

例えば、

```text
H-A = OI上昇
H-B = Leverage増加
H-C = OI急増による投機参加増加
```

は同じEvidenceや同じMechanismを言い換えている可能性がある。

そのためHypothesis Setでは、

- shared_evidence
- dependency
- redundancy
- common_cause
- research_strength
- applicability
- contradiction

を確認する。

独立した異なるMechanismが同じ方向を支持する場合と、同じEvidenceを3回数える場合を区別する。

---

# 6. Trade Thesis

複数HypothesisをそのままSignalへ投げず、一つの取引論拠へまとめる。

例:

```text
TRADE THESIS T-0042

Direction:
SHORT

Primary:
H-101 Derivatives Overheat

Supporting:
H-204 ETF / Spot Support Weakening

Conditional:
H-331 US Session High-Vol Regime

Contradicting:
H-407 Spot Buying Reversal

Expected Horizon:
30m

Expected Effect:
上昇失速 → 下落

Expected Value:
positive

Main Risk:
Spot buying reversal

Invalidation:
H-101崩壊 + H-407強化
```

Trade Thesisは「複数仮説の平均」ではなく、現在市場でどの仮説がどの役割を持っているかを固定した構造体。

---

# 7. AI Review

AIの役割:

> 市場理解OSが作ったTrade Thesis / Hypothesis Set / Evidence / Market DNA / Research結果を外部知能に独立査読させる。

AI同士は原則会話させない。

```text
Trade Thesis Package
   ├→ AI A
   ├→ AI B
   └→ AI C
```

問いの例:

- Hypothesis間に重複はあるか
- 同じEvidenceを二重評価していないか
- 最大の反証材料は何か
- Contradicting Hypothesisを過小評価していないか
- 見落としたAlternative Hypothesisはあるか
- TRADE / WAIT / REJECT
- Confidence / Uncertainty

AI Reviewが新しい未検証Hypothesisを思いついた場合、それをその場でProduction Thesisへ追加しない。

```text
AI提案
→ Research Candidate
```

として将来研究へ送る。

AI Reviewが利用不能でも本番経路は動作可能にする。

---

# 8. Signal Engine

役割:

> 現在のTrade Thesisに、実際にRiskを取るだけの期待値と適用可能性があるかを判断する。

入力候補:

- Trade Thesis
- Applicable Hypothesis Set
- Causal Edge
- Empirical Edge
- Expected Value
- Current Market Context
- Market DNA
- Similar Cases
- Feature Priority
- Constraint
- Data Quality
- Uncertainty
- AI Review

出力候補:

```text
BUY
SELL
NO_TRADE
```

全部を単純加算した総合点にしない。

役割例:

```text
Expected Value = 主判断材料
Primary Hypothesis = 中心理由
Supporting Hypothesis = 独立支持
Conditional Hypothesis = 適用条件
Contradicting Hypothesis = 反証圧力
Empirical Edge = 再現性
Market DNA = 適用可能性
Constraint = Gate
Data Quality = Trust Limit
AI Review = Advisory
```

---

# 9. CausalとEmpiricalの二系統

全Tradeを「因果仮説が完全に説明できる時だけ」に限定しない。

```text
Causal Edge ─┐
             ├→ Trade Thesis / Signal
Empirical Edge ┘
```

原因説明が不完全でもOOS / Demo Forward等で安定した期待値があるならCandidateにできる。

逆に因果説明が強くてもExpected Valueが悪ければTradeしない。

---

# 10. Pre-Trade Defense

SignalとDefenseを統合しない。

```text
Signal
= 取りたいか

Defense
= 今取ってよいか
```

確認候補:

- Constraint violation
- Data Quality
- Liquidity
- Spread
- Slippage risk
- Current DD
- Consecutive losses
- Exposure
- API / Exchange health
- Abnormal event
- Risk limit

出力:

```text
ALLOW
REDUCE
BLOCK
```

DefenseはTrade Thesisを作らない。

---

# 11. Shadow Trade

DefenseでBlockしたTrade Thesisも研究のため仮想追跡する。

```text
Trade Thesis = VALID
Signal = BUY
Defense = BLOCK
Real Trade = NONE
Shadow Trade = TRACK
```

後から、

- BlockしたThesisの何%が損失になったか
- 何%が利益機会だったか
- Defenseが厳しすぎないか

をResearchする。

---

# 12. Execution Logic

内側Executionは注文戦略を決める。

候補:

- Position Size
- Leverage
- Limit / Market
- Split Order
- Stop
- Take Profit
- Slippage tolerance
- Liquidity requirement

出力は標準Order Intent。

実際の取引所API呼び出しはExchange Adapterが行う。

Demo ForwardとLiveの比較可能性を保つため、Execution Logicは可能な限り共通化する。

---

# 13. Entry Thesis

Entryした瞬間、「なぜ入ったか」を固定保存する。

Entry ThesisはTrade ThesisのProduction Snapshot。

候補:

```text
trade_id
trace_id
trade_thesis_id
hypothesis_set_version
primary_hypothesis_id
supporting_hypothesis_ids
conditional_hypothesis_ids
contradicting_hypothesis_ids
edge_ids
expected_direction
expected_horizon
expected_effect
expected_value
key_evidence
shared_evidence_map
hypothesis_dependency_map
important_features
market_dna
feature_priority_profile
invalidation_conditions
risk_budget
constraints
entry_uncertainty
```

Entry後に都合よくHypothesisを追加・削除して理由を書き換えない。

重大な新情報が出た場合は「元のEntry Thesisが変更された」のではなく、新EventとしてSupervisor / Exit判断へ入力する。

---

# 14. Position Supervisor

目的:

> Trade中に「Trade Thesis全体がまだ成立しているか」を監督し、Exit判断へ助言を渡す。

単一Hypothesisだけを監視しない。

例:

```text
H-101 PRIMARY = WEAKENING
H-204 SUPPORTING = STRONG
H-331 CONDITIONAL = VALID
H-407 CONTRADICTING = WEAK
↓
Trade Thesis = HOLD / WATCH候補
```

別例:

```text
H-101 PRIMARY = INVALIDATED
H-204 SUPPORTING = WEAKENING
H-407 CONTRADICTING = STRONG
↓
Trade Thesis = THESIS_INVALIDATED候補
```

状態候補:

```text
HOLD
WATCH
CAUTION
THESIS_WEAKENING
THESIS_INVALIDATED
```

別枠:

```text
EMERGENCY
```

---

# 15. 足元だけを見ないための原則

Position Supervisorが短期ノイズでTradeを壊さないよう、時間軸を分離する。

```text
超短期: Orderbook / Spread / CVD
短期: OI / Funding / Liquidation
中期: Market Structure / Participant Flow
Trade Thesis時間軸: Expected Horizon
```

下位時間軸の単発異常だけで、上位のTrade Thesis全体を否定しない。

---

# 16. Hypothesis Decay / Thesis Decay

Expected Horizon内に期待したEffectが発生しないこと自体をEvidenceにする。

単一HypothesisごとのDecayだけでなくTrade Thesis全体のDecayも記録する。

```text
Expected:
30分以内に下落

45分経過
Primary Effect未発生
Supporting Evidenceも弱体化
↓
Trade Thesis Health低下
```

---

# 17. Position Supervisorは勝手に通常Exitしない

Supervisorは助言機構。

```text
Position Supervisor
→ Hypothesis Health / Trade Thesis Health
→ Exit Engine
```

通常のExitはExit Engineが、

- Trade Thesis Health
- Primary / Contradicting Hypothesis状態
- PnL
- Time
- Liquidity
- Stop
- Take Profit
- Risk

等を統合して判断する。

EMERGENCYはHard Safetyとして強制Exitを許可する候補。

---

# 18. In-Trade Defense

Supervisor = 仮説健全性助言。
In-Trade Defense = 強制安全制約。
Exit Engine = 決済判断。

責任を混ぜない。

---

# 19. Production Evidence

Real Tradeは研究件数を稼ぐための実験ではなく、現実摩擦を含む最終Evidenceとして扱う。

Liveでしか十分に観測できない候補:

- Real Fill
- Real Slippage
- Partial Fill
- Exchange latency
- Fee
- Funding cost
- Liquidity impact
- Real execution failure

Demo Forward成績とLive成績を混ぜず、差をResearchへ返す。

---

# 20. Productionで残すべき情報

Trade結果だけでなく、Trade Thesisと各Hypothesisの状態遷移を保存する。

例:

```text
Entry:
T-0042
H-101 = STRONG
H-204 = STRONG
H-407 = WEAK

10m:
H-101 = VALID
H-204 = WEAKENING
H-407 = NORMAL

20m:
H-101 = WEAKENING
H-204 = WEAK
H-407 = STRONG

Trade Thesis = THESIS_WEAKENING
↓
Exit
```

この系列をResearchで「どの仮説からTrade Thesisが壊れ始めたか」の研究材料にする。

---

# 21. Productionの禁止事項

- Research中Candidateを未承認で使う
- AI ReviewだけでTradeを決定
- Hypothesis多数決で方向を決定
- 同じEvidence由来のHypothesisを独立票として数える
- Entry後に都合のよいHypothesisを後付け
- 短期ノイズ一つでTrade Thesisを無効化
- DefenseをSignal生成器にする
- Exchange Adapterへ市場判断を持たせる
- Position Supervisorへ無制限な自動裁量を与える
- Demo ForwardとLive成績を一つの勝率へ合算する

---

# 22. 本番の最終思想

取引前:

> 複数の研究済み仮説を整理したTrade Thesisに、今Riskを取る合理性があるか。

取引中:

> Trade Thesisを構成する理由はまだ生きているか。

取引後:

> どのHypothesis・Signal・Defense・Execution・市場変化が結果へ寄与したか。

この3段階を分離し、全てResearchへ戻せるようにする。
