# 市場理解OS まとめ案 2

# OUTER / CONTROL & CONNECTION

## 1. 役割

Outerは「市場を理解する場所」ではない。

役割は、

- 何を対象市場にするか
- どの取引所・データSourceへ接続するか
- どの設定でCoreを動かすか
- どの資金上限・Risk Limitを与えるか
- どのPython Processを起動・停止するか

を管理すること。

基本原則:

> 外側は市場判断をしない。内側は外部接続やOS運営をしない。

---

# 2. 現在の対象範囲

現在は仮想通貨FXに限定する。

最初の目的は「BTC専用コード」を作ることではなく、Crypto FX共通CoreにMarket Profileを差し込み、BTCUSDT / ETHUSDT / SOLUSDT等を後から簡単に切り替えられる構造にすること。

株・FX・Commodity等の別Asset Classへの一般化は後回しにする。

---

# 3. Market Profile

Market ProfileをOuterとCoreの主要境界にする。

例:

```yaml
market_id: BTC_USDT_PERP
asset: BTC
quote: USDT
market_type: crypto_perpetual
exchange: selected_exchange

timeframes:
  - 5m
  - 15m
  - 30m
  - 1h

required_sources:
  - price
  - volume
  - orderbook
  - trades
  - cvd
  - funding
  - open_interest
  - liquidation
  - etf
  - nasdaq
  - dxy
  - news
```

実装時は、Core内の複数`.py`に`BTCUSDT`を直接埋め込まない。

CoreはMarket Profileからmarket_idや設定を受け取る。

---

# 4. Source Adapter

外部データSource固有形式を、OS標準Raw / Observation形式へ変換する。

例:

```text
Bybit OI
Binance OI
その他Exchange OI
        ↓
Source Adapter
        ↓
共通形式
metric = open_interest
market_id = BTC_USDT_PERP
value = ...
timestamp = ...
source = ...
quality metadata = ...
```

Coreは可能な限り「どのAPI形式だったか」を知らない。

Provenanceとしてsource情報は残す。

---

# 5. Exchange Adapter

取引所固有APIをCoreから分離する。

```text
Execution Logic
↓
Order Intent
↓
Exchange Adapter
↓
Bybit / Binance / etc.
```

Coreが出すものは標準Order Intent。

Exchange Adapterが行うもの:

- Symbol変換
- 注文形式変換
- Tick Size対応
- Minimum Order対応
- API Authentication
- API Request
- Response変換
- 約定情報取得

禁止:

- Exchange Adapterが「BUYすべきか」を判断する
- Signal Engineが取引所APIを直接叩く

---

# 6. Outer → Core の標準入力

原則として以下に絞る。

## A. Market Configuration
対象市場・時間足・利用Source等。

## B. Market Data
標準化されたRaw / Observation / Event。

## C. Runtime Command
START / STOP / PAUSE / RESUME / RELOAD等。

## D. Global Risk Limit
利用可能資金、最大Exposure、最大DD、取引停止状態等。

---

# 7. Core → Outer の標準出力

## A. Order Intent
実行したい注文計画。

## B. System Status
RUNNING / DEGRADED / PAUSED / ERROR等。

## C. Research / Audit Event
未説明Event、未知市場候補、Research Candidate、重大矛盾等。

## D. Performance / Exposure
PnL、DD、現在Position、Exposure等。

---

# 8. Global Contextの扱い

BTC、ETH、SOLなどを後から同時運用する場合でも完全独立させない。

共通参照候補:

- Risk-On / Risk-Off
- DXY
- NASDAQ
- VIX
- Macro Event
- Global Crypto Flow
- BTC Regime

ただしOuter側に第二のMarket Intelligenceを作らない。

Outerは共通Contextを配るだけで、意味解釈はCore側のMarket Intelligenceが担当する。

---

# 9. 将来の複数銘柄運用

理想:

```text
Crypto FX Core
  ↑ Market Profile
  ├─ BTC Instance
  ├─ ETH Instance
  └─ SOL Instance
```

プログラムを銘柄ごとにコピーしない。

避ける構造:

```text
btc_os/
eth_os/
sol_os/
```

同じロジックを複製すると、修正漏れ・Version差・研究結果の不整合が発生しやすい。

---

# 10. 将来のPortfolio Control

複数Cryptoを同時運用する段階では、各Trade単体のDefenseとは別にPortfolio Riskが必要になる。

例:

- Max Crypto Exposure
- Max Correlated Exposure
- Max Single Asset
- Portfolio DD Limit
- Global Kill Switch

ただし現段階では自動資金配分AIまでは導入しない。

最初は固定上限と監視から始める。

---

# 11. Outerの禁止事項

Outerが行ってはいけない判断:

- BTCは上がる / 下がる
- この因果仮説は正しい
- このFeatureを重視すべき
- BUY / SELL
- Positionを今Exitすべき

これらはCore / Research / Productionの仕事。

---

# 12. Coreの禁止事項

Coreが行ってはいけない運用:

- API Key管理
- OS Processの再起動
- DB Backup
- Git Version更新
- 取引所接続切替
- Telegramから直接取引所操作
- 複数市場間の全体資金移動

これらはOuter / Operationsの責任とする。

---

# 13. 現段階の固定ルール

1. Coreに銘柄をハードコードしない。
2. Coreに取引所固有ロジックを混ぜない。
3. 市場差はMarket Profileで与える。
4. 外側と内側はData Contractで通信する。
5. Outerは市場を理解しない。
6. CoreはOS運営をしない。
7. 仮想通貨FXで交換可能性を証明してから他市場へ一般化する。
