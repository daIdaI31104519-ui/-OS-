# 市場理解OS まとめ案 6

# PRODUCTION / TRADING DOMAIN

## 1. 目的

ProductionはResearchの知識を使って、実際の売買候補を作り、安全に実行し、保有中も仮説の健全性を監督する。

基本原則:

> Researchは大きく、本番経路は小さくする。

本番フロー:

```text
Approved Knowledge / Edge
+ Current Market Context
+ Market DNA
+ Feature Priority
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
Entry Thesis
        ↓
Position Supervisor
        ↓
Exit / In-Trade Defense
        ↓
Trade Result
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
- 「多数決」が正しさを保証しない
- 本番経路が重くなる
- AI停止がTrade停止へ直結しやすい

代わりにAI Reviewを補助機能として使う。

---

# 3. AI Review

AIの役割:

> 市場理解OSが作った仮説・Evidence・Market DNA・Research結果を外部知能に独立査読させる。

AI同士は原則会話させない。

```text
Research Package
   ├→ AI A
   ├→ AI B
   └→ AI C
```

問いの例:

- この因果仮説で今Tradeする合理性はあるか
- 最大の反証材料は何か
- 見落としは何か
- TRADE / WAIT / REJECT
- Confidence

多数決だけでTradeを決めない。

意見割れはUncertainty上昇材料として扱う。

AI Reviewが利用不能でも本番経路は動作可能にする。

---

# 4. Signal Engine

役割:

> 現在の条件で期待値のある取引候補が存在するかを判断する。

入力候補:

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

重要:

全部を単純加算した謎の総合点にしない。

役割例:

```text
Expected Value = 主判断材料
Causal Support = 根拠
Empirical Edge = 再現性
Market DNA = 適用可能性
Constraint = Gate
Data Quality = Trust Limit
AI Review = Advisory
```

---

# 5. CausalとEmpiricalの二系統

全Tradeを「強い因果仮説がある時だけ」に限定しない。

```text
Causal Edge ─┐
             ├→ Signal Engine
Empirical Edge ┘
```

原因説明が不完全でもOOS等で安定した期待値があるならCandidateにできる。

逆に因果説明が強くても期待値が悪ければTradeしない。

---

# 6. Pre-Trade Defense

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

Defenseは期待値を作らない。

---

# 7. Shadow Trade

DefenseでBlockしたTradeも研究のため仮想追跡する。

```text
Signal = BUY
Defense = BLOCK
Real Trade = NONE
Shadow Trade = TRACK
```

後から、

- BlockしたTradeの何%が損失になったか
- 何%が利益機会だったか
- Defenseが厳しすぎないか

をResearchする。

---

# 8. Execution Logic

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

---

# 9. Entry Thesis

Entryした瞬間、「なぜ入ったか」を固定保存する。

これはPosition SupervisorとPost-Trade Analysisの基準になる。

候補:

```text
trade_id
trace_id
hypothesis_id
edge_ids
expected_direction
expected_horizon
expected_effect
key_evidence
important_features
market_dna
feature_priority_profile
invalidation_conditions
risk_budget
constraints
entry_confidence
```

Entry後に都合よく理由を書き換えない。

---

# 10. Position Supervisor

本番経路の重要追加機能。

目的:

> Trade中に「Entry Thesisがまだ成立しているか」を監督し、Exit判断へ助言を渡す。

単なるリアルタイム価格監視ではない。

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

# 11. 足元だけを見ないための原則

Position Supervisorが短期ノイズでTradeを壊さないよう、時間軸を分離する。

例:

```text
超短期: Orderbook / Spread / CVD
短期: OI / Funding / Liquidation
中期: Market Structure / Participant Flow
仮説時間軸: Expected Horizon
```

30分仮説を1秒の板変化だけで否定しない。

```text
Orderbook異常
→ 継続
→ CVD反転
→ Spot Flow反転
→ 元Evidence崩壊
```

のように複数Evidenceと継続性を重視する。

---

# 12. Hypothesis Decay

Expected Horizon内に期待したEffectが発生しないこと自体をEvidenceにする。

例:

```text
Hypothesis:
30分以内に下落

45分経過
Effect未発生
↓
Hypothesis Health低下
```

価格逆行だけでなく「時間内に実現しない」ことも監視する。

---

# 13. Position Supervisorは勝手に通常Exitしない

Supervisorは助言機構。

```text
Position Supervisor
→ Thesis Health / Advisory
→ Exit Engine
```

通常のExitはExit Engineが、

- Thesis Health
- PnL
- Time
- Liquidity
- Stop
- Take Profit
- Risk

等を統合して判断する。

ただしEMERGENCYはHard Safetyとして強制Exitを許可する候補。

---

# 14. In-Trade Defense

Pre-Trade Defenseと思想は同じだが、Position保有中のHard Riskを監視する。

候補:

- Exchange failure
- API failure
- Data collapse
- Liquidity disappearance
- Max loss
- Global Risk limit breach
- Constraint hard violation

Supervisor = 仮説健全性助言。
In-Trade Defense = 強制安全制約。
Exit Engine = 決済判断。

責任を混ぜない。

---

# 15. Productionで残すべき情報

Trade結果だけでなく、保有中の状態遷移を保存する。

例:

```text
Entry Thesis health = 0.84
5m = 0.87
10m = 0.73
CVD reversal
15m = 0.58
Whale reversal
20m = 0.31
Exit
```

この系列がResearchで「いつ仮説が壊れ始めたか」を研究する材料になる。

---

# 16. Productionの禁止事項

- Research中Candidateを未承認で使う
- AI ReviewだけでTradeを決定
- 短期ノイズ一つでExit
- Entry理由を保有中に書換え
- DefenseをSignal生成器にする
- Exchange Adapterへ市場判断を持たせる
- Position Supervisorへ無制限な自動裁量を与える

---

# 17. 本番の最終思想

取引前:

> 入る合理性はあるか。

取引中:

> 入った理由はまだ生きているか。

取引後:

> なぜこの結果になったか。

この3段階を分離し、全てResearchへ戻せるようにする。
