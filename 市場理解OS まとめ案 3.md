# 市場理解OS まとめ案 3

# MARKET UNDERSTANDING CORE：データ・計算・特徴量・市場理解

## 1. 目的

この領域は「市場から取得した情報を、研究可能・比較可能・説明可能な状態へ変換する」ことを担当する。

大きな流れ:

```text
Collector
→ Raw Data
→ Observation / Market Event
→ Normalizer
→ Data Quality
→ Time Series
→ Calculation / Measurement
→ Feature
→ Feature Priority
→ Market Intelligence
→ Evidence
```

---

# 2. Collector

Collectorの責任は取得のみ。

取得候補:

- Price
- Volume
- Orderbook
- Trades
- CVD用約定
- Funding
- Open Interest
- Liquidation
- ETF Flow
- On-chain
- Whale
- NASDAQ / S&P500
- DXY / Yield / Macro
- News
- SNS / Sentiment
- Economic Calendar

禁止:

- BUY / SELL判断
- 「このニュースは強気」等の最終解釈
- Feature計算
- Causal判断

Collectorは同時に品質診断用Metadataを残す。

例:

```text
source
requested_at
received_at
latency
status
missing
raw_hash
schema_version
```

---

# 3. Raw Data

Raw Dataは一次証拠として必ず保存する。

原則:

- 加工前原本を残す
- 後からFormula / Featureを再計算可能にする
- SourceとTimestampを失わない
- RawをFeatureで上書きしない

Raw Dataを持つ理由は「正しいFeatureを一度で作るため」ではなく、「後から間違いを発見しても研究をやり直せるため」。

---

# 4. Observation / Market Event

Rawを研究可能な観測単位へ変換する。

例:

```text
OBSERVATION:
open_interest = 12.4B

EVENT:
open_interest increased rapidly
```

Observation = 観測値。
Market Event = 市場で発生した現象。

Market Event IDは「市場現象」を追跡するためのIDとし、OS内部処理履歴用Trace IDとは分離する。

---

# 5. Normalizer

Sourceごとの表現差を統一する。

対象:

- Timestamp
- Timezone
- Symbol
- Unit
- Decimal
- Interval
- Naming
- Currency
- Missing representation

Normalizer自身は品質評価を確定しない。

ただし以下をDiagnosticsとして残す。

- 変換失敗
- Timestamp補正量
- Unit変換履歴
- Schema mismatch
- Source差

---

# 6. Data Quality

下流が「入力をどこまで信用していいか」を判断できる情報を作る。

評価候補:

- Missing
- Duplicate
- Latency
- Staleness
- Outlier
- Timestamp drift
- Cross-source inconsistency
- API failure
- Impossible value

出力は単なるPASS / FAILだけにしない。

```text
quality_status
quality_score
issues[]
source_health
freshness
confidence_limit
```

上流のQualityが低い場合、その不確実性は下流へ伝播させる。

---

# 7. Time Series

市場データを「現在値」だけでなく時間変化として扱う。

候補:

- Current Value
- Delta
- Return
- Rate of Change
- Velocity
- Acceleration
- Persistence
- Rolling Distribution
- Percentile
- Volatility
- Lagged Values

時間窓を明示する。

例:

```text
sampling_interval = 1m
measurement_window = 15m
lookback = 7d
valid_until = ...
```

異なる時間軸を無自覚に混ぜない。

---

# 8. Calculation / Measurement Core

これは一本道の独立Layerではなく、OS全体が使う共通基盤。

管理対象:

## Formula Registry
正式な計算式とVersion。

## Feature Registry
Featureの定義・入力・時間軸・出力範囲。

## Metric Registry
Quality、Risk、Similarity等の測定定義。

## Unit Rules
USD / BTC / Ratio / Percent等の単位整合性。

## Time Window Rules
何分・何時間の値同士を比較しているか。

重要原則:

> 数学的に計算可能であることと、市場的に意味が正しいことは別。

例えばFundingとOIを単純加算してよいとは限らない。

足し算・掛け算・重み付き・条件式・非線形等の選択自体をResearch対象にする。

---

# 9. Formula Version

計算式を上書きして履歴を失わない。

例:

```text
FORM-LIQUIDITY-001 v1
FORM-LIQUIDITY-001 v2
FORM-LIQUIDITY-001 v3
```

後から、

- どのVersionでTradeしたか
- OOS性能
- Regime別性能
- Stress耐性

を追跡できるようにする。

---

# 10. Feature

Raw / Observation / Time Seriesから、市場理解に利用できる測定値を作る。

例:

- OI Acceleration
- Funding Percentile
- CVD Divergence
- Orderbook Imbalance
- Liquidation Pressure
- ETF Flow Change
- BTC-NASDAQ Correlation
- DXY Sensitivity
- Whale Pressure
- Liquidity Stress

Featureには最低限以下を紐付ける。

```text
feature_id
value
formula_version
input_refs
time_window
quality
confidence
created_at
trace_id
```

---

# 11. Feature Priority

目的:

> 「最強Feature」を固定するのではなく、「今の市場状態では何を見る価値が高いか」を決める。

考慮候補:

- Historical usefulness
- OOS usefulness
- Current Regime
- Current Event
- Time of day
- Day of week
- Horizon
- Data Quality
- Freshness
- Stability
- Feature redundancy
- Market DNA similarity

例:

```text
今日は米国大型テック決算日
BTC-NASDAQ相関が高い

Priority:
1. NASDAQ
2. Earnings Event
3. BTC-NASDAQ Correlation
4. CVD
5. OI
...
```

重要:

Feature PriorityはSignalではない。

Feature Priority → Market Intelligenceへ「どの情報を重点的に見るか」を渡す。

---

# 12. 選ばなかったFeatureも保存する

High PriorityだけでなくLow Priorityも保存する。

理由:

> 後から「低優先にした判断自体が間違っていなかったか」を検証するため。

保存候補:

```text
selected_features
low_priority_features
priority_reason
priority_version
context
confidence
```

---

# 13. Market Intelligence

責任:

> 「今、市場で何が起きているのか」を構造化して解釈する。

主な観測視点:

- Price / Structure
- Volatility
- Liquidity
- Derivatives
- Participant Flow
- ETF / Institutional Flow
- Macro
- News / Sentiment
- Abnormal Event

出力候補:

```text
Market Context
Evidence Candidate
Event
State
Confidence
Quality
Contradiction
Unexplained Event
```

禁止:

- 原因を確定する
- BUY / SELLを出す
- Causal Hypothesisを最終確定する

---

# 14. Unexplained Event

既存FeatureやEvidenceで市場変化を説明できない場合、無理に説明しない。

例:

```text
BTC +4%
ETF neutral
OI neutral
CVD neutral
Whale neutral
Macro neutral
```

この場合:

```text
UNEXPLAINED_EVENT
```

としてResearchへ送る。

研究候補:

- 未取得Source
- 未知Feature
- Regime change
- Data Quality issue
- 新しい参加者行動
- Causal structure change

「分からない」を研究資産として残す。

---

# 15. この領域の最終成果物

Market Intelligenceまでの段階で、下流へ渡すものは「売買判断」ではなく、

- Current Market Context
- Evidence
- Important Features
- Feature Priority
- Quality
- Confidence / Uncertainty
- Event IDs
- Trace IDs
- Research Candidates

とする。

この情報をCausal Engine・Market DNA・Researchが利用する。
