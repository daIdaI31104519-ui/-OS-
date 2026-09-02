# 市場理解OS まとめ案 4

# CAUSAL ENGINE / MARKET DNA / KNOWLEDGE DOMAIN

## 1. この領域の目的

Market Intelligenceが「今、何が起きているか」を整理した後、

- なぜ起きている可能性があるか
- その説明はどの程度支持されるか
- 現在市場は過去のどの市場に似ているか
- その知識を将来再利用できるか

を扱う。

重要:

> Causal Engineは原因を証明する装置ではない。
> Market DNAは売買Signalではない。
> Knowledgeは単なるログ置き場ではない。

---

# 2. Causal Engineの正式な役割

入力:

- Market Intelligence
- Evidence
- Market Event
- Feature / Feature Priority
- Current Context
- Historical Case
- Data Quality
- Confidence / Uncertainty

役割:

> 観測されたEffectに対して、検証可能な原因候補・代替説明・反証条件を構造化し、Researchへ渡せるCausal Hypothesisを管理する。

---

# 3. Causal Engineの研究項目

現時点での基本系列:

1. Cause Candidate
2. Effect
3. Temporal Order
4. Lag
5. Confounder
6. Alternative Hypothesis
7. Contradiction
8. Evidence Strength
9. Hypothesis Score / Reliability
10. Historical Validation要求
11. OOS要求
12. Regime Stability要求
13. Stress Labへの引き渡し条件
14. Hypothesis Lifecycle
15. Causal Hypothesis → Market DNA / Knowledgeへの反映

ただし10〜13の「実験そのもの」はCausal Engineが全部実行しない。

```text
Causal Engine
「何を検証すべきか定義」
        ↓
Research Domain
「Historical / OOS / Regime / Stressを実行」
        ↓
Research Result
        ↓
Causal Engine
「Hypothesis状態を更新」
```

これによりCausal EngineとResearchの責任重複を防ぐ。

---

# 4. Cause Candidate

Cause CandidateはCauseではない。

定義:

> Market Intelligenceが生成したEvidence / Event / Contextを根拠に、あるEffectを生じさせた可能性がある要因として一時登録される未検証の原因候補。

禁止:

- 相関だけでCause確定
- 後から起きた事象を原因扱い
- 根拠のないニュース物語化
- PredictionをCauseとして保存

Cause CandidateはResearch可能な候補として扱う。

---

# 5. Alternative / Contradictionを捨てない

採用Hypothesisだけを保存しない。

例:

```text
採用: H-021
Alternative: H-018, H-019
Contradiction: E-993, E-994
Rejected because: ...
```

理由:

後からH-021が崩れた時、以前RejectしたH-019が説明力を持つ可能性がある。

「選ばなかった説明」も研究資産。

---

# 6. Hypothesis Lifecycle

概念状態:

```text
DRAFT
→ TESTING
→ SUPPORTED
→ WEAK
→ RETIRED
```

必要に応じて、

- HOLD
- REOPENED
- CONFLICTED

等を後から設計する。

重要:

1回当たっただけでSUPPORTEDにしない。
1回外れただけでRETIREDにしない。

複数Case・OOS・Regime・Contradictionを見てLifecycleを変える。

---

# 7. Market DNA

Market DNAの役割:

> 現在の市場状態を、過去市場と比較・検索・研究できる圧縮表現へ変換する。

Market DNAは「AIの謎スコア」ではない。

候補軸:

- Trend
- Volatility
- Liquidity
- Leverage
- Derivatives
- Participant Flow
- Whale
- ETF / Institution
- Macro Linkage
- Time / Session
- News / Event Context
- Cross-market Correlation

各軸は、Raw / Feature / Formula Versionまで追跡可能にする。

---

# 8. Market DNAとFeature Priorityの違い

この2つは統合しない。

```text
Market DNA
= 今はどんな市場か

Feature Priority
= この市場では何を見る価値が高いか
```

研究が進むと、

```text
DNA-A → OI / Funding / CVDが重要
DNA-B → NASDAQ / DXY / ETFが重要
DNA-C → Orderbook / Liquidationが重要
```

という「市場状態と有効観測方法の対応」が学習できる。

---

# 9. SimilarityとNovelty

Market DNAは類似CaseだけでなくNoveltyも扱う。

例:

```text
nearest_similarity = 0.41
novelty = high
```

なら、

> 過去Caseに十分似ていない可能性

としてResearch Candidateを作る。

候補:

```text
NOVEL_REGIME_CANDIDATE
```

これによりMarket DNAを新Regime候補発見にも利用する。

---

# 10. Knowledge Domain

Knowledge系を多数の独立Layerへ分割しない。

Knowledge Domain内の主要Object候補:

```text
MarketCase
MarketDNA
CausalHypothesis
FeatureKnowledge
FormulaKnowledge
ResearchResult
Failure
FailureBoundary
Constraint
NegativeKnowledge
Relationship
```

---

# 11. Case Library / Market Memory / Failure Museum / Knowledge Graph

これらは別々に同じ情報を複製するのではなく、Knowledge Domainの異なるViewとして考える。

## Case Library
過去市場Case中心のView。

## Market Memory
再利用可能な市場経験中心のView。

## Failure Museum
失敗Case・壊れ方中心のView。

## Knowledge Graph
Object同士の関係中心のView。

同じFailureを4箇所へコピーするのではなく、一つのKnowledge Objectを複数Viewから参照する方向を基本とする。

---

# 12. Negative Knowledge

「効果がなかった」ことも保存する。

例:

```text
NK-0021
Feature: Google Trends
Target: BTCUSDT
Horizon: 30m
Result: No stable edge
OOS: Failed
Regime: All tested regimes
Reopen condition: new data / new formula / structural change
```

目的:

- 同じ無駄研究の繰り返し防止
- 条件変更時の再研究判断
- 「何が効かなかったか」の知識化

---

# 13. FailureBoundary / Constraint

Stress Lab等で得た結果を3段階で扱う。

```text
StressResult
= 実験で何が起きたか

FailureBoundary
= どこから成立しなくなったか

Constraint
= Productionで利用可能な適用・禁止条件
```

ConstraintはBUY / SELLを作らない。

役割は、

> 「このEdge / Hypothesisを今使ってよい条件か」

をGateすること。

---

# 14. 知識の昇格

Research結果をすぐProduction Knowledgeへしない。

```text
Research Result
→ Candidate Knowledge
→ Validation
→ Approval
→ Approved Knowledge
→ Production利用可能
```

Research DBとProduction利用知識の境界を持つ。

---

# 15. この領域から下流へ渡すもの

- Causal Hypothesis
- Hypothesis State
- Evidence / Contradiction
- Applicability
- Market DNA
- Similar Cases
- Novelty
- Constraint
- Approved Knowledge
- Research Candidate
- Confidence / Uncertainty
- Trace / Provenance

この段階でも最終BUY / SELLは作らない。
