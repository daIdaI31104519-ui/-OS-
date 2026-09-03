# DESIGN_CHANGE_RULES.md

# 市場理解OS 設計変更完全ルール

## 0. 文書情報

- 文書種別: GOVERNANCE / TOP-LEVEL DESIGN CHANGE RULE
- 状態: CANONICAL GOVERNANCE CANDIDATE
- 上位ルール: `00_GOVERNANCE/GIT_RULES.md`
- 対象: 市場理解OSのArchitecture / Role / Object / State / Contract / Research / Production / Platform / Implementation Design
- 目的: 新しい案・改善・廃止・統合・緊急変更を、既存設計を壊さず、何十年以上にわたり追跡・検証・Rollback可能な形で扱う
- 最上位原則: **設計変更は「思いつき」ではなく、影響範囲・責任・互換性・長期維持コストを確認してから正式化する**

---

# 1. このルールの目的

市場理解OSは長期運用を前提とするため、設計変更をその場の会話や実装都合だけで行わない。

設計変更では最低限、次を守る。

1. なぜ変更するか説明できる
2. どのRole / Object / Contract / Stateへ影響するか追跡できる
3. 既存設計との重複・責任衝突を確認する
4. 新Layerを安易に増やさない
5. ResearchとProductionの境界を壊さない
6. Data / Schema / Version互換性を確認する
7. 変更前の設計を追跡可能にする
8. 問題時にRollback可能にする
9. 実装コードがCanonical Designから逸脱しない
10. 数年後・数十年後でも変更理由を再確認できる

---

# 2. GIT_RULESとの関係

設計変更のGit操作は必ず `GIT_RULES.md` に従う。

特に以下を再確認する。

```text
会話で採用
≠
Git書き込み許可
```

```text
Design APPROVED
≠
Canonical自動更新
```

ユーザーから明示的な保存・反映指示がある場合のみGitを書き換える。

---

# 3. 設計変更の基本Lifecycle

設計変更は原則として次のLifecycleを通す。

```text
IDEA
↓
REVIEW
↓
CLASSIFY
↓
IMPACT ANALYSIS
↓
DECISION
├─ ADOPT
├─ MERGE
├─ HOLD
└─ REJECT
↓ ADOPT / MERGE
PROPOSAL
↓
APPROVAL
↓
CANONICAL CHANGE
↓
IMPLEMENTATION SPEC
↓
IMPLEMENTATION
↓
VALIDATION
↓
ACTIVE
↓
MONITOR / REVIEW
```

問題発生時:

```text
ACTIVE
↓
DEGRADED / ROLLBACK CANDIDATE
↓
ROLLBACK / FIX / RESEARCH
```

---

# 4. 設計変更の種類

変更を最低限次に分類する。

## CHANGE-TYPE-001: ADD

新しい機能・Role・Object・Contract・Rule等を追加する。

## CHANGE-TYPE-002: MODIFY

既存責任・Field・State・Flow・Rule等を変更する。

## CHANGE-TYPE-003: MERGE

重複するRole / Object / Layerを統合する。

## CHANGE-TYPE-004: SPLIT

責任が過大なRole / Domainを分離する。

## CHANGE-TYPE-005: DEPRECATE

既存設計を段階的に廃止する。

## CHANGE-TYPE-006: REMOVE

不要な設計要素を正式に除外する。

## CHANGE-TYPE-007: MIGRATE

Schema / Data / API / Storage / Runtime / Technologyを新構造へ移行する。

## CHANGE-TYPE-008: EMERGENCY

本番安全性・Data破損・Security等に関係する緊急変更。

---

# 5. 変更規模の分類

## PATCH

責任・Contract・Data意味を変えない小修正。

例:

- 説明修正
- Typo
- コメント改善
- 非機能的な命名修正

## MINOR

既存責任内の機能追加・State追加・Field追加。

例:

- Diagnostics項目追加
- 新しいResearch Candidate Type
- Optional Field追加

## MAJOR

Architecture / Responsibility / Data Contract / Production behaviorへ影響する変更。

例:

- Single HypothesisからTrade Thesisへ変更
- Market DNA計算責任の移動
- SignalとDefense境界変更
- DB Schema互換性破壊
- Production Risk Flow変更

MAJOR変更はCanonical反映前に影響分析を必須とする。

---

# 6. 新しい案を受けた時の最初の5問

新しいアイデアに対して、最初に必ず次を確認する。

```text
1. 本当に新Layer / 新Roleが必要か？
2. 既存RoleのSubmoduleで表現できないか？
3. Data Object / Contract追加だけで解決できないか？
4. 既存設計と責任・Data・Stateが重複しないか？
5. 何十年運用した時の維持コストに見合うか？
```

これを通過しない案を安易にArchitectureへ追加しない。

---

# 7. 新Layer追加Gate

新Layerを追加する場合、最低限次を満たすこと。

```text
□ 独立した明確な責任がある
□ 既存Layerへ入れると責任が崩れる
□ 独立した入力 / 出力がある
□ Lifecycleが独立している
□ Failure境界が独立している
□ Scale / Performance上分離価値がある
□ Long-Term maintenance価値がある
□ Data Contractで境界を定義可能
```

満たさない場合は優先順位として、

```text
既存Role内Submodule
↓
Shared Service
↓
Data Object
↓
Contract
↓
View
```

で解決を検討する。

---

# 8. Role変更ルール

Role変更時は `ROLE_DICTIONARY.md` の責任を基準とする。

変更候補ごとに最低限確認する。

```text
Role名
現在の目的
現在の責任
変更後の責任
入力
出力
禁止事項
上流
下流
Failure時挙動
Researchへの戻り
Long-Term Governance影響
```

Roleが「便利だから」という理由だけで別Roleの責任を取り込むことを禁止候補とする。

---

# 9. Object変更ルール

Object変更時は意味・Version・互換性を確認する。

最低限:

```text
object_id
schema_version
required_fields
optional_fields
owner_role
creator_role
consumer_roles
immutability
retention
migration_rule
```

意味が変わるFieldを同じ名前のまま再利用しない。

例:

```text
confidence v1 = model probability
confidence v2 = composite research reliability
```

のような意味変更を無言で行わない。

---

# 10. State変更ルール

Lifecycle Stateを追加・削除・統合する場合、状態名だけでなく遷移条件を定義する。

最低限:

```text
from_state
trigger
preconditions
to_state
actions
prohibited_transitions
rollback / recovery
```

例:

```text
APPROVED
↓ edge degradation
RISK_REDUCED
```

のように、誰が・何を根拠に変更するかを明示する。

---

# 11. Contract変更ルール

Role間Contract変更はMAJOR候補として扱う。

確認項目:

```text
Producer
Consumer
Current Contract Version
New Contract Version
Added Fields
Removed Fields
Changed Semantics
Backward Compatibility
Migration
Fallback
Validation
```

Production中のConsumerが旧Contractを読む可能性がある場合、互換期間またはMigrationを設計する。

---

# 12. Data Flow変更ルール

Data Flowを変更する場合、単なる矢印変更として扱わない。

最低限確認:

- 新しい依存関係
- 循環依存
- Latency
- Failure伝播
- Data Quality伝播
- Trace / Provenance
- Retry時の重複
- Ordering
- Idempotency
- Production safety

特に循環が生じる場合、Cycle単位・前回確定値・Event駆動等で明示的に解消する。

---

# 13. Research変更ルール

Research変更では「研究能力を増やす」だけでなく、Research Governanceを同時確認する。

確認項目:

```text
Research目的
Candidate生成条件
Admission条件
Budget
Stop Rule
OOS
Demo Forward
Stress
Unique Market Event Count
Overfitting Risk
Storage Growth
Negative Knowledge保存
Production昇格条件
```

研究候補を増やす変更には、研究停止条件・重複排除・計算資源上限を可能な限り同時設計する。

---

# 14. Production変更ルール

Production変更はResearch変更より厳しく扱う。

Productionへ影響する変更候補:

- Signal
- Defense
- Risk State
- Execution
- Position Size
- Leverage
- Entry / Exit
- Position Supervisor
- Exchange Adapter
- Approved Knowledge利用規則

最低限:

```text
□ Research結果がある
□ OOS / Demo等のValidationがある
□ Riskへの影響を確認
□ Failure時の挙動がある
□ Rollback可能
□ Production Versionが記録可能
```

未承認Research Candidateを直接Productionへ追加しない。

---

# 15. Research → Production昇格ルール

原則:

```text
Research
→ Candidate
→ Validation
→ Demo Forward
→ Approval
→ MICRO / LIMITED Production
→ Normal Production
```

全項目に完全同一の順序を強制するかは各Protocolで決めるが、ResearchからLiveへ無審査で飛ばさない。

Production昇格後もContinuous Evaluationを続ける。

---

# 16. Risk / DDに影響する設計変更

Risk変更は明示的にImpact Analysisする。

確認候補:

- Expected Drawdown
- Max Drawdown
- Tail Risk
- Position Exposure
- Correlation Exposure
- Slippage
- Liquidity
- Failure Mode
- Kill Switch
- Recovery State
- Edge degradation

Riskを増加させる変更は、利益期待だけを理由に採用しない。

---

# 17. Long-Term Governance確認

すべてのMINOR以上の変更で、可能な限り次を確認する。

```text
Resource
Failure
Risk
Research
Version
Migration
Storage
Backup
Security
Knowledge Aging
Source Lifecycle
Audit
```

例:

新しい高頻度Featureを追加する場合:

```text
精度だけでなく
→ Raw増加量
→ DB増加量
→ CPU
→ RAM
→ Recompute Cost
→ Archive Policy
```

まで検討する。

---

# 18. Impact Analysis

MAJOR変更では最低限次を記録する。

```text
Change ID:
Title:
Change Type:
Size:
Reason:
Current Design:
Proposed Design:
Affected Domains:
Affected Roles:
Affected Objects:
Affected Contracts:
Affected States:
Affected DB:
Affected Code:
Research Impact:
Production Impact:
Risk Impact:
Resource Impact:
Migration Required:
Backward Compatible:
Rollback Plan:
Open Questions:
```

全項目が必ず必要という意味ではないが、該当項目を無視しない。

---

# 19. Decision Status

設計案の判定は最低限次を使う。

## ADOPT

採用する。

## MERGE

既存設計へ統合する。

## HOLD

価値はあるが今は採用しない。

## REJECT

採用しない。

## RESEARCH_REQUIRED

判断材料不足。先に研究する。

## BLOCKED

依存設計・データ・技術・安全上の問題で現時点では進められない。

---

# 20. Decision Log

MAJOR変更や重要判断はDecision IDを持たせる候補とする。

例:

```text
DECISION-0042

Title:
Single Hypothesis Production → Multi-Hypothesis Trade Thesis

Status:
ADOPTED

Reason:
複数研究済み仮説を現在市場へ適用可能な集合として利用するため

Old Design:
Single Hypothesis

New Design:
Applicable Hypothesis Set → Trade Thesis

Affected:
Research / Production / Post-Trade

Rollback:
旧Single Hypothesis compatibility mode
```

将来的には `DECISION_LOG.md` を正本とする。

---

# 21. Canonical変更ルール

Canonicalは現在の正式設計。

Canonical変更前に最低限:

```text
□ Proposalが明確
□ 既存Canonical確認済み
□ Conflict確認済み
□ Duplication確認済み
□ Impact確認済み
□ Approvalあり
□ Migration / Rollback検討済み
```

Canonicalへ変更した後は、旧Canonicalを追跡不能にしない。

Git history / Archive / Decision Log等で確認可能にする。

---

# 22. 上位設計と下位設計の優先順位

原則:

```text
Governance
↓
Project Constitution
↓
Architecture
↓
Domain Responsibility
↓
Role / Object / State
↓
Contract
↓
Implementation Spec
↓
Code
```

下位設計が上位設計を勝手に上書きしない。

例:

```text
Architecture:
ResearchとProductionは分離
```

なら、コード実装が便利だからという理由でResearchがProduction DBを直接変更する構造を作らない。

上位を変更したい場合は上位Design Changeとして扱う。

---

# 23. Conflict Resolution

設計同士が矛盾した場合、最新日付だけで決めない。

確認順:

```text
1. Governance
2. Canonical Status
3. Decision Log
4. Responsibility境界
5. Contract Version
6. 最新のApproved Change
7. Historical / Legacy
```

旧資料が新しいファイルへコピーされたことで「最新扱い」になる事故を避ける。

---

# 24. Duplicate Resolution

同じ責任が複数箇所に存在した場合、原則としてSingle Source of Truthを選ぶ。

例:

```text
Role定義
→ ROLE_DICTIONARY.md

State定義
→ STATE_DICTIONARY.md

Object定義
→ OBJECT_DICTIONARY.md

Contract
→ 03_CONTRACTS
```

他文書では参照・要約に留め、独立した別定義を作らない。

---

# 25. Deprecationルール

既存設計を廃止する場合、即削除を基本としない。

```text
ACTIVE
↓
DEPRECATED
↓
LEGACY
↓
ARCHIVED
```

必要に応じて互換期間を設ける。

記録候補:

```text
deprecated_at
replacement
migration_deadline
reason
compatibility
removal_condition
```

---

# 26. Schema Migrationルール

DB / Object Schema変更ではVersionを持つ。

例:

```text
MarketDNA v1
↓ migration
MarketDNA v2
```

最低限確認:

- 旧Dataを読めるか
- Migration Scriptが必要か
- Losslessか
- Rollback可能か
- Backup取得済みか
- Validation方法

重要Knowledgeを「新Schemaに合わない」という理由だけで破棄しない。

---

# 27. Formula / Feature Version変更

FormulaやFeature定義を上書きして旧Versionを失わない。

例:

```text
FORM-LIQUIDITY-001 v1
FORM-LIQUIDITY-001 v2
```

過去Trade / Research Resultは当時使用したVersionを参照可能にする。

---

# 28. Emergency Change

次の場合は通常Proposal Flowを短縮できる候補。

- Real Money安全性
- Security Incident
- API Key漏洩
- Data Corruption
- Exchange重大障害
- Risk Limit故障
- Execution暴走
- Critical Bug

ただし緊急変更でも後から必ず記録する。

```text
EMERGENCY CHANGE
↓
Containment
↓
Safe State
↓
Fix
↓
Validation
↓
Postmortem
↓
Canonical / Decision Log更新
```

緊急時にArchitecture全体を場当たり的に改造しない。

---

# 29. Rollbackルール

重要変更には可能な限りRollback Planを持たせる。

候補:

```text
rollback_trigger
previous_version
data_compatibility
config_restore
code_restore
production_safe_state
validation_after_rollback
```

Rollback不能な変更は、その事実自体をRiskとして明示する。

---

# 30. Implementation Drift

Canonical Designと実装コードのズレを放置しない。

分類:

```text
DESIGN_AHEAD
= 設計済み・未実装

CODE_AHEAD
= コードが設計より先行

DRIFT
= 設計とコードが矛盾

SYNCED
= 一致
```

CODE_AHEADが発生した場合、コードが正しいとは自動判断しない。

```text
コード変更理由確認
↓
設計へ採用するか判断
```

を行う。

---

# 31. Test / Validationルール

設計変更のValidation方法を変更種類に応じて選ぶ。

候補:

- Static Review
- Contract Test
- Unit Test
- Integration Test
- Replay
- Historical Validation
- OOS
- Demo Forward
- Shadow
- Stress
- Canary / Micro Live

すべての変更に全Testを要求するわけではない。

Production影響が大きいほどValidationを強くする。

---

# 32. Design Definition of Done

設計変更を「完了」とする最低候補。

```text
□ 変更目的が説明できる
□ Change Type / Sizeが分かる
□ 既存責任との重複確認済み
□ 影響Role / Object / Contract確認済み
□ Failure / Risk確認済み
□ Long-Term impact確認済み
□ Migrationが必要なら定義済み
□ Rollbackが必要なら定義済み
□ Canonical / Proposal状態が明確
□ Git変更対象が明確
□ 旧設計が追跡可能
□ 次のImplementation Specへ渡せる
```

---

# 33. GPTが設計変更を提案する時の標準形式

可能な限り次の順で整理する。

```text
1. 現在設計
2. 問題
3. 変更案
4. なぜ必要か
5. 既存Roleへ入れられない理由
6. 利点
7. 欠点
8. Risk
9. Researchへの影響
10. Productionへの影響
11. Long-Term impact
12. 推奨判定
```

新案を説明するだけでGitへ自動保存しない。

---

# 34. GPTがGitへ設計変更を書く時

`GIT_RULES.md` に従い、原則として次を行う。

```text
1. 明示的書き込み指示を確認
2. 対象ファイル確認
3. 現Canonical / Proposal確認
4. Conflict / Duplication確認
5. 変更対象ファイルを宣言
6. 1目的で変更
7. Git書き込み
8. 再読込確認
9. 変更ファイルを報告
```

範囲外ファイルを勝手に更新しない。

---

# 35. 一回の設計作業は一つの主題へ絞る

例:

```text
今日はData Contract
```

なら、途中で発見したRisk / Telegram / Storage案を全て同時設計しない。

別テーマは、

```text
IDEA / TODO / Research Candidate
```

として記録候補へ回す。

これにより設計の拡散を防ぐ。

---

# 36. 設計Reviewの種類

## Conflict Review

矛盾確認。

## Duplication Review

重複責任確認。

## Boundary Review

Role境界確認。

## Dependency Review

循環・依存方向確認。

## Long-Term Review

Resource / Migration / Aging / Backup等確認。

## Production Safety Review

Real Money経路への影響確認。

## Implementation Sync Review

コードとのズレ確認。

必要なReviewだけを選択する。

---

# 37. 定期Architecture Review

何十年以上の運用では設計そのものも定期的に再評価する。

候補:

```text
短期:
重大Change後のReview

中期:
Research / Production / Riskの整合確認

長期:
Architecture / Technology / Storage / Migration / Security Review
```

周期は別途正式設計する。

「昔Canonicalだったから永久に正しい」とは扱わない。

---

# 38. 変更してはいけないものを明確にする

明示的Approvalなしに次を変更しない候補。

- Project Constitution
- Research / Production分離原則
- Real Money Risk上限思想
- Raw Data保全原則
- Trace / Provenance原則
- Canonical Data Contract
- Security / Secret Policy
- Git書き込みルール

これらは上位Governance Changeとして扱う。

---

# 39. 現在のまとめ案1〜11との関係

`市場理解OS まとめ案 1〜11.md` はArchitecture Proposal / Consolidated Draft。

今後は、まとめ案から正式設計を作る際に本ルールを適用する。

```text
まとめ案
↓
Role / Object / State抽出
↓
Conflict / Duplication Review
↓
Proposal
↓
Approval
↓
Canonical
```

まとめ案を直接コード仕様として扱わない。

---

# 40. この文書自身の変更

`DESIGN_CHANGE_RULES.md` 自体も設計変更ルールの対象。

変更時は最低限:

```text
変更理由
変更内容
影響
旧ルールとの互換性
GIT_RULESとの整合
```

を確認する。

この文書を変更した結果、過去のDesign Decisionが意味不明にならないようにする。

---

# 41. 最終原則

市場理解OSでは、

> **良いアイデアを速く追加することより、良いアイデアを正しい場所へ、追跡可能・検証可能・Rollback可能な形で追加することを優先する。**

何十年以上の運用では「設計を変えないこと」ではなく、

> **安全に変え続けられること**

を設計品質とする。
