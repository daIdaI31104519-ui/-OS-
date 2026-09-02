# 市場理解OS まとめ案 8

# PYTHON RUNTIME / OPERATIONS / TELEGRAM

## 1. この領域の目的

市場理解OSの市場研究ロジックとは分離して、Pythonコード群を正しく起動・監視・停止・復旧・操作する。

役割を4系統に分ける。

```text
Python Runtime / Operations
Monitoring
Recovery / Deployment
Telegram Interface
```

これらは市場判断をしない。

---

# 2. Python Runtime / Operations

責任:

- 起動順序
- Process管理
- Scheduler
- Dependency確認
- Config読込
- Health Check
- Debug支援
- Error Handling
- Logging
- Restart
- Graceful Shutdown
- Version確認

各`.py`が勝手に別の重要Processを起動し、起動順が分散する構造を避ける。

---

# 3. Boot Sequence

最終的な実装順は本設計で確定するが、概念順序は次を候補とする。

```text
1. Environment / Config確認
2. DB / Storage確認
3. Adapter / API接続確認
4. Collector起動
5. Data Pipeline起動
6. Calculation / Feature基盤起動
7. Market Understanding Core起動
8. Research Runtime起動
9. Production Runtime起動
10. Monitoring起動
11. Telegram Interface起動
```

重要:

Productionは必要な依存関係がHealthyになる前に注文可能状態へ移行しない。

---

# 4. Runtime State

Module・Market Instanceごとに状態を持つ。

候補:

```text
BOOTING
RUNNING
PAUSED
DEGRADED
ERROR
RECOVERY
STOPPING
STOPPED
```

例:

```text
Collector = RUNNING
Data Quality = RUNNING
Research = RUNNING
Signal = PAUSED
Execution = STOPPED
```

Execution停止中に注文が流れないようRuntime Gateを持つ。

---

# 5. DEGRADED Mode

一部機能が壊れても即全停止するとは限らない。

例:

```text
ETF Collector停止
↓
OS = DEGRADED
↓
Data Quality / Confidence低下
↓
Trade可否はConstraint / Defenseで判断
```

ただしCritical SourceやExecution Safety障害ならTradeを停止する。

どの障害がDEGRADEDで、どの障害がSTOPかは本設計で明文化する。

---

# 6. Debug / Error Contract

共通Error形式を持つ候補。

```text
error_id
module
market_id
trace_id
exception_type
message
severity
input_refs
occurred_at
runtime_state
recovery_action
```

目的:

Trade異常からTraceを遡り、

```text
Trade
→ Signal
→ Hypothesis
→ Feature
→ Calculation
→ Collector
```

のどこで異常が始まったか特定できること。

---

# 7. Monitoring

Monitoringは「市場」ではなく「システム」を監視する。

対象候補:

- Collector health
- Source latency
- Data freshness
- DB health
- Queue backlog
- Research process
- Production process
- Exchange connection
- Order state
- Position state
- Memory / CPU
- Disk
- Error rate
- Telegram connection

複数銘柄化した場合:

```text
BTC Instance = HEALTHY
ETH Instance = DEGRADED
SOL Instance = STOPPED
```

のように市場Instance単位でも把握する。

---

# 8. Recovery

部分障害で全OSを破壊しない。

候補:

- Retry
- Backoff
- Source切替
- Process restart
- Queue recovery
- DB reconnect
- Safe pause
- Position safety check
- Manual escalation

例:

```text
Source A failure
→ Source A停止
→ Backup Source利用
→ Quality低下を明示
→ 必要ならProduction PAUSE
```

---

# 9. Deployment

Researchで良い結果が出ても、いきなりProduction全体へ投入しない。

候補フロー:

```text
Development
→ Unit / Integration Test
→ Replay
→ Paper
→ Canary
→ Production
```

必要に応じてRollback可能にする。

Market DNA / Formula / Feature / Defense Rule等はVersionを追跡する。

---

# 10. Telegram Interface

Telegramは市場理解OSの人間向けConsole。

責任:

1. 報告
2. Alert
3. 状態確認
4. 操作要求
5. 緊急操作
6. Market Diary閲覧

Telegram自身は市場判断をしない。

---

# 11. Telegram報告例

```text
BTC Market Status

Market DNA: ...
Top Features: ...
Causal Hypothesis: ...
Signal: WAIT
Defense: NORMAL
Position: NONE
Data Quality: HEALTHY
```

Position保有中:

```text
POSITION WARNING
BTC SHORT
Supervisor: THESIS_WEAKENING
CVD reversal
Spot buy increased
```

---

# 12. Telegram Command候補

```text
/status
/health
/positions
/risk
/research_status
/start BTC
/pause BTC
/resume BTC
/stop BTC
/daily
```

本格コマンド体系は後のTelegram設計で確定する。

---

# 13. Telegramから直接Exchangeを操作しない

禁止構造:

```text
Telegram
→ Exchange API
→ SELL
```

正しい方向:

```text
Telegram
→ Command Gateway
→ Authentication / Authorization
→ Command Validation
→ Runtime / OS
→ Defense / Execution
→ Exchange Adapter
```

Telegramは命令要求を送るだけ。

---

# 14. 危険操作の確認

例:

```text
/close_all
```

を1メッセージで即実行しない。

候補:

```text
Request
→ Current Positions表示
→ Confirmation Token
→ Confirm
→ Execution
→ Result Report
```

Kill Switch等は用途ごとに別設計する。

---

# 15. Python RuntimeとOuterの違い

```text
Outer
= 何を・どこへ接続して動かすか

Python Runtime
= Python Process群を正しく動かす

Core
= 市場を理解する

Telegram
= 人間との窓口
```

この4責任を混ぜない。

---

# 16. 現段階のコード配置イメージ

これは確定Treeではなく、本設計時の参考構造。

```text
market_understanding_os/
│
├─ control/
│  ├─ runtime/
│  ├─ monitoring/
│  ├─ recovery/
│  └─ configuration/
│
├─ adapters/
│  ├─ exchanges/
│  ├─ market_data/
│  ├─ news/
│  └─ macro/
│
├─ profiles/
│  └─ crypto/
│     ├─ BTC_USDT.yaml
│     ├─ ETH_USDT.yaml
│     └─ SOL_USDT.yaml
│
├─ core/
│  ├─ data_quality/
│  ├─ calculation/
│  ├─ feature/
│  ├─ intelligence/
│  ├─ causal/
│  ├─ market_dna/
│  ├─ research/
│  ├─ signal/
│  ├─ defense/
│  ├─ execution/
│  ├─ position_supervisor/
│  └─ analysis/
│
├─ interfaces/
│  └─ telegram/
│
└─ contracts/
   ├─ market_profile
   ├─ market_data
   ├─ processing_result
   ├─ order_intent
   ├─ risk_limit
   └─ research_event
```

実際の`.py`名・Folder TreeはPYTHON_ARCHITECTURE / PROJECT_TREE作成時に確定する。

---

# 17. 運用系の禁止事項

- RuntimeがMarket Signalを作る
- MonitoringがTrade方向を決める
- Telegramが取引所APIを直接操作
- RecoveryがResearch Ruleを勝手に変更
- Deploymentが未承認Research結果をProductionへ投入
- Error発生だけでAIが自動コード修正し本番反映

運用系はOSを安全に動かすことに集中する。
