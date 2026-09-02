# 市場理解OS まとめ案 1

## 0. 文書の位置づけ

- 状態: DRAFT SUMMARY / Canonical候補前
- 目的: これまでの市場理解OS構想を、重複・旧案・責任衝突を整理したうえで一本の全体像として保存する。
- 現在の対象: 仮想通貨FX。まずBTCUSDT等をMarket Profileで差し替え可能にする。株式・為替・商品への一般化は将来検討とし、現段階では実装対象に含めない。
- この文書群は最終設計書ではない。ここから全体像を再度絞り、本格的なPROJECT_MANUAL / DATA_FLOW / DATABASE_SCHEMA / PYTHON_ARCHITECTURE等へ昇格させる。

---

# 1. 市場理解OSの目的

市場理解OSの目的は、単純な価格予測AIを作ることではない。

基本思想は次の循環を自律化することにある。

```text
観測
→ 測定
→ 市場理解
→ 因果仮説
→ 市場状態の記憶
→ 研究
→ 優位性
→ 売買候補
→ 防御
→ 実行
→ 取引中監督
→ 結果分析
→ 再研究
```

最終判断基準は「予測が当たったか」だけではなく、以下を重視する。

1. 再現可能な期待値があるか
2. どの市場条件で成立するか
3. どこから壊れるか
4. どの程度の不確実性を持つか
5. 長期的に生存できるか
6. 成功・失敗の理由を後から追跡できるか
7. 研究成果が次の市場理解へ再利用されるか

---

# 2. 全体を5つの領域に分ける

市場理解OSを「全部が一列に並ぶ巨大レイヤー」として扱わない。

## A. OUTER / CONTROL & CONNECTION
市場理解OSを外部世界と接続し、何を・どこで・どう動かすかを管理する。

## B. MARKET UNDERSTANDING CORE
市場を観測・測定・理解し、因果仮説と市場状態を作る。

## C. RESEARCH DOMAIN
仮説・特徴量・計算式・Market DNA・失敗境界を研究し、Causal Edge / Empirical Edgeを作る。

## D. PRODUCTION / TRADING DOMAIN
研究済み知識と現在市場からSignalを作り、Defense・Execution・Position Supervisorを通して実売買する。

## E. POST-TRADE / FEEDBACK DOMAIN
取引結果を分析し、原因別にResearchへ戻す。

さらに全領域を横断して、

- Calculation / Measurement
- Data Contract
- Trace / Provenance
- Confidence / Uncertainty
- Knowledge Contract

を共通基盤として使う。

外周には、

- Python Runtime / Operations
- Monitoring
- Recovery
- Deployment
- Telegram Interface

を置く。

---

# 3. 現時点の大きな全体フロー

```text
外部市場 / 取引所 / Macro / ETF / 株式 / News / SNS / On-chain
        ↓
Outer: Market Profile / Source Adapter / Exchange Adapter / Config
        ↓
Collector
        ↓
Raw Data
        ↓
Observation / Market Event
        ↓
Normalizer
        ↓
Data Quality
        ↓
Time Series
        ↓
Calculation / Measurement 共通基盤
        ↓
Feature
        ↓
Feature Priority
        ↓
Market Intelligence
        ↓
Evidence
        ↓
Causal Engine
        ↓
Causal Hypothesis
        ↓
Market DNA
        ↓
Knowledge / Case / Memory
        ↓
Research Domain
        ↓
Causal Edge / Empirical Edge / Constraint / Knowledge
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
Entry / Entry Thesis
        ↓
Position Supervisor
        ↓
Exit / In-Trade Defense
        ↓
Trade Result
        ↓
Logger / Post-Trade Analysis
        ↓
Counterfactual / Research Router
        ↓
必要なResearchへ戻る
        ↺
```

---

# 4. 最重要設計原則

## 4.1 Researchは大きく、Live Pathは小さくする
研究では大量の検証・探索・反証を許す。本番売買では必要な検証済み結果だけを使用し、故障点と処理遅延を増やさない。

## 4.2 ResearchとProductionを分離する
研究結果は直接本番を書き換えない。

```text
Research
→ Candidate
→ Validation
→ Approval
→ Production
```

を基本とする。

## 4.3 Raw Dataを捨てない
Featureを先に作ってRawを失うことは禁止する。計算式やFeature定義を後から変更しても、Rawから再生成可能にする。

## 4.4 各責務を小さく保つ
各モジュールは本来の仕事を一つに絞る。ただし処理結果から得られる品質情報・異常・Research Candidate等の副産物は捨てない。

## 4.5 「分からない」を推測で埋めない
Market IntelligenceやCausal Engineが説明できない現象はUNEXPLAINED EVENT / Research Candidateとして保存する。

## 4.6 失敗だけでなく成功・見送りも研究する
- Unexpected Failure
- Unexpected Success
- Missed Opportunity
- Defense Block
- Supervisor Warning
- Shadow Trade

も研究対象にする。

## 4.7 全てを一つのスコアへ潰さない
Expected Value、Data Quality、Causal Support、Applicability、Constraint、AI Review等は意味が違う。単純加算した総合点だけで判断しない。

---

# 5. 旧案からの整理

以下は現段階の全体像から外す、または格下げする。

## 廃止 / Legacy候補
- AI Team
- AI Meeting
- 「負けたらTrainer」の単純再学習構造

## Research Toolへ変更
- Quantum Layer → Quantum / Search Tool

## Validationへ変更
- Stress Lab → Research内の限界・破壊条件検証

## Experimental Frameworkへ統合
- Random Baseline
- Replay
- Paper Trade
- Shadow Trade
- Counterfactual

## Knowledge DomainのViewへ整理
- Case Library
- Market Memory
- Failure Museum
- Knowledge Graph

これらは機能を捨てるのではなく、独立Layerの乱立を防ぐため責任を再配置する。

---

# 6. 現段階で追加しないもの

- 株式・為替・商品取引への本格対応
- AIによる自動資金配分
- AI同士の自律会議
- 本番取引で毎回Quantumを通す設計
- Human Market Diaryを直接学習データにする設計
- 各層が自分自身のコードやルールを自動書換えする機構

これらは必要性が実証された時にProposalとして追加する。

---

# 7. このまとめ案から本設計へ移る条件

本格的な設計MDへ移る前に、最低限次を確定する。

1. OuterとCoreの境界
2. 各Domainの入力・出力・禁止事項
3. Data Contract
4. Processing Contract
5. Trace / Event ID体系
6. Research → Approval → Productionの昇格規則
7. Market Profileによる銘柄差し替え規則
8. Python RuntimeとTelegramの責任範囲

この8点が固まった時点で、暫定まとめから正式設計へ移行する。
