# GIT_RULES.md

# 市場理解OS Git運用・書き込み完全ルール

## 0. 文書情報

- 文書種別: GOVERNANCE / TOP-LEVEL RULE
- 状態: CANONICAL GOVERNANCE CANDIDATE
- 対象: 市場理解OSの設計・仕様・コード・運用文書を保存するGitリポジトリ
- 目的: 会話中の案、検討中Proposal、正式設計、実装仕様、コード、旧案を混同せず、何十年以上にわたり安全に変更履歴を残す
- 最上位原則: **Gitへ書くことと、会話で考えることを分離する**

---

# 1. このルールの目的

市場理解OSは長期間継続するため、Gitを単なるコード置き場として扱わない。

Gitは次の役割を持つ。

1. 現在の正式設計を保存する
2. 検討中の案を正式設計と分離して保存する
3. なぜ設計が変更されたかを追跡する
4. 旧設計・廃止案を失わない
5. 実装と設計の対応関係を追跡する
6. GPT・人間・将来の開発環境が同じルールで扱えるようにする
7. 数年後・数十年後でも「何を、なぜ、いつ変えたか」を再現可能にする

---

# 2. 最重要原則

## RULE-GIT-001: GPTは明示的な書き込み指示がない限りGitを変更しない

通常の会話、相談、深掘り、レビュー、提案だけではGitへ書き込まない。

書き込み可能な例:

```text
「Gitに保存して」
「Gitに書いて」
「この案をGitへ登録して」
「IDEA保存」
「このファイルを更新して」
「この設計をGitへ反映して」
```

書き込み不可の例:

```text
「どう思う？」
「この案あり？」
「深掘りして」
「採用寄りだね」
「こうした方が良くない？」
「設計としてどう？」
```

重要:

> **採用・良い案という会話上の評価と、Gitへ保存する許可は別。**

明示的な保存・反映指示が必要。

---

## RULE-GIT-002: GPTはGit書き込み前に対象ファイルを確認する

既存ファイルを更新する場合、原則として現在内容を先に読み、以下を確認する。

- 対象ファイルが本当に存在するか
- 現在の役割
- 現在の状態
- 既存内容との矛盾
- 上書きで失われる内容
- 最新SHA / Version

既存内容を確認せず、推測で全置換しない。

---

## RULE-GIT-003: 1回のGit変更は1つの目的に限定する

良い例:

```text
Commit A: GIT_RULES.md追加
Commit B: ROLE_DICTIONARY.md追加
Commit C: Risk State Machine更新
```

避ける例:

```text
1 Commitで
- Market DNA変更
- Telegram追加
- DB変更
- Risk変更
- Research変更
```

理由:

- 後から差分を理解しやすい
- 問題発生時に戻しやすい
- 変更理由を追跡しやすい
- 長期運用で履歴が壊れにくい

---

# 3. Git内の情報状態

市場理解OSの情報を最低限次の状態に分ける。

```text
IDEA
↓
REVIEWED
↓
PROPOSAL
↓
APPROVED
↓
CANONICAL
↓
IMPLEMENTATION_SPEC
↓
IMPLEMENTED
↓
VALIDATED
```

別系統:

```text
HOLD
REJECTED
DEPRECATED
LEGACY
ARCHIVED
```

---

# 4. 各状態の正式定義

## IDEA

思いつき・未整理案。

特徴:

- 正式設計ではない
- 矛盾していてもよい
- 実装禁止
- 将来評価対象

## REVIEWED

GPTまたは人間が、目的・利点・欠点・重複・配置候補を確認した状態。

まだ正式設計ではない。

## PROPOSAL

既存設計との関係を確認し、設計候補として構造化された状態。

最低限:

- 目的
- 役割
- 入力
- 出力
- 既存Roleとの関係
- 利点
- 欠点
- Risk
- 長期維持コスト

を整理する。

## APPROVED

ユーザーが採用を明示したProposal。

ただし、**APPROVEDになっただけでは自動Git更新しない。**

Git反映には書き込み指示が必要。

## CANONICAL

現在の正式設計。

実装は原則としてCanonicalを基準にする。

## IMPLEMENTATION_SPEC

CanonicalをPython / DB / API / Config等へ落とすための実装仕様。

## IMPLEMENTED

コードとして実装済み。

## VALIDATED

Test / Replay / Demo / Integration等で仕様に適合していることを確認済み。

---

# 5. HOLD / REJECTED / LEGACY / ARCHIVED

## HOLD

悪い案ではないが、現在は実装・正式採用しない。

例:

- 将来必要になる可能性
- データ不足
- 技術成熟待ち
- 優先順位が低い

## REJECTED

検討したが採用しない。

必ず可能なら理由を残す。

## DEPRECATED

以前は有効だったが、現在は新設計へ置換中。

## LEGACY

旧構造・旧思想として参照のため残す。

現在設計へ混ぜない。

## ARCHIVED

現役ではないが、歴史・判断理由・再検討用に保存する。

---

# 6. 原則として削除よりArchiveを優先する

何十年以上の運用では、古い設計を完全削除すると「なぜ変更したか」が失われる。

そのため原則:

```text
ACTIVE
↓
DEPRECATED
↓
LEGACY / ARCHIVED
```

とする。

完全削除を許可する候補:

- 明確な誤生成ファイル
- 秘密情報を誤ってCommitした場合
- 完全重複かつ履歴価値がない場合
- ユーザーが明示的に削除を指示した場合

秘密情報の場合は通常の削除だけでなく、履歴からの除去も別途検討する。

---

# 7. Gitへ保存する内容の分類

推奨構造:

```text
00_GOVERNANCE/
01_DICTIONARY/
02_ARCHITECTURE/
03_CONTRACTS/
04_RESEARCH/
05_PRODUCTION/
06_PLATFORM/
07_IMPLEMENTATION/
99_ARCHIVE/
```

---

# 8. 00_GOVERNANCE

最上位ルールを保存する。

候補:

```text
GIT_RULES.md
DESIGN_CHANGE_RULES.md
PROJECT_CONSTITUTION.md
DECISION_RULES.md
LONG_TERM_GOVERNANCE.md
```

原則:

> 下位設計はGovernanceへ違反してはならない。

---

# 9. 01_DICTIONARY

用語・役割・Object・Stateを一元管理する。

候補:

```text
ROLE_DICTIONARY.md
OBJECT_DICTIONARY.md
STATE_DICTIONARY.md
GLOSSARY.md
```

同じRole定義を複数ファイルへコピーしない。

---

# 10. 02_ARCHITECTURE

市場理解OS全体の構造を管理する。

候補:

```text
SYSTEM_ARCHITECTURE.md
DOMAIN_RESPONSIBILITY.md
DATA_FLOW.md
DEPENDENCY_RULES.md
```

---

# 11. 03_CONTRACTS

Role / Module間の正式な受け渡しを管理する。

候補:

```text
DATA_CONTRACT.md
PROCESSING_CONTRACT.md
ERROR_CONTRACT.md
RESEARCH_CONTRACT.md
EVIDENCE_CHANNEL_CONTRACT.md
TRADE_THESIS_CONTRACT.md
```

---

# 12. 04_RESEARCH

研究方法・Validation・Research Governanceを保存する。

正式設計と実験結果は分離する。

候補:

```text
RESEARCH_PROTOCOL.md
EXPERIMENT_PROTOCOL.md
VALIDATION_PROTOCOL.md
RESEARCH_GOVERNANCE.md
```

大量の研究結果をMarkdownへ直接無限蓄積する設計は避ける。

研究結果本体はDB / Storage側へ保存し、GitにはSchema・Protocol・重要Decisionを保存する方向を基本とする。

---

# 13. 05_PRODUCTION

本番Trade関係の正式設計。

候補:

```text
SIGNAL_DESIGN.md
DEFENSE_DESIGN.md
RISK_DESIGN.md
EXECUTION_DESIGN.md
POSITION_SUPERVISOR.md
```

Research Candidateを直接ここへ書き込まない。

---

# 14. 06_PLATFORM

市場判断ではなくOS運営に関する設計。

候補:

```text
RUNTIME.md
MONITORING.md
RECOVERY.md
STORAGE.md
BACKUP.md
SECURITY.md
```

---

# 15. 07_IMPLEMENTATION

正式設計をコードへ落とす仕様。

候補:

```text
DATABASE_SCHEMA.md
PYTHON_ARCHITECTURE.md
PROJECT_TREE.md
BOOT_SEQUENCE.md
MODULE_DEPENDENCY.md
TEST_STRATEGY.md
```

---

# 16. 99_ARCHIVE

旧案・廃止・歴史的資料。

Canonical設計と混ぜない。

---

# 17. 現在の「市場理解OS まとめ案 1〜11」の扱い

現在のまとめ案シリーズは、正式Canonicalそのものではなく、Architecture Proposal / Consolidated Draftとして扱う。

流れ:

```text
まとめ案
↓
Conflict / Duplication Review
↓
採用項目抽出
↓
Canonical Proposal
↓
ユーザー承認
↓
正式設計MD
```

まとめ案をそのまま正式コード仕様として扱わない。

---

# 18. GPTへのユーザー指示と意味

ユーザー指示を次のように解釈する。

## 「どう思う？」

- 分析のみ
- Git変更禁止

## 「深掘りして」

- 詳細検討
- Git変更禁止

## 「採用する」

- 設計上APPROVED扱い候補
- Git変更はまだ禁止

## 「保留」

- HOLD扱い
- Git変更は明示保存指示がある時のみ

## 「なかったことにする」

- 会話上の案をREJECTED候補
- 既にGitにある場合は勝手に削除しない

## 「保存して」 / 「Gitに登録して」

- Git書き込み許可
- 保存先・現在状態・既存設計との関係を確認して書く

## 「設計に入れたい」

- Proposal / Canonical候補へ昇格する意図
- **これだけではGit書き込み許可とはみなさない**

## 「実装したい」

- Canonical / Implementation Specを確認してコード化へ進む
- 設計未確定なら未確定部分を明示する
- Git書き込みは明示指示された範囲のみ

---

# 19. GPTは書き込み前に変更対象を宣言する

Git変更前に可能な限り、ユーザーへ次を短く伝える。

```text
変更するファイル:
- xxx.md
- yyy.md

変更しないファイル:
- その他
```

新規ファイルなら新規作成であることを伝える。

これにより意図しない変更を防ぐ。

---

# 20. GPTは書き込み後に変更したファイルを明示する

書き込み後は必ず、最低限次を報告する。

- 新規作成したファイル
- 更新したファイル
- 削除したファイル
- 変更しなかった重要ファイル
- 変更内容の概要

コード変更の場合は `.py`、設計変更なら `.md` 等、実際の拡張子を明示する。

---

# 21. 変更対象外ファイルを触らない

ユーザーが、

```text
GIT_RULES.mdだけ変更して
```

と指定した場合、他ファイルを「整合性のため」という理由だけで勝手に変更しない。

整合性上変更が必要と思われる場合は、

```text
追加変更候補
```

として報告し、別の明示指示を待つ。

---

# 22. Canonical変更ルール

Canonicalは通常のIdeaより変更条件を厳しくする。

最低流れ候補:

```text
問題発見 / 新Idea
↓
Proposal
↓
Impact Review
↓
Conflict Review
↓
Approval
↓
Canonical Update
↓
Dependent Spec Review
```

Canonical変更後、依存する下位文書・コードへ影響がある場合は、影響対象を列挙する。

勝手に全部更新しない。

---

# 23. 上位設計と下位設計の優先順位

優先順位:

```text
1. PROJECT_CONSTITUTION / GOVERNANCE
2. SYSTEM_ARCHITECTURE
3. DOMAIN / ROLE / OBJECT / CONTRACT
4. IMPLEMENTATION_SPEC
5. CODE
6. RUNTIME CONFIG
```

原則:

> 下位の都合で上位設計を暗黙変更しない。

コードがCanonicalと合わない場合、

```text
設計が古いのか
コードが間違っているのか
```

を先に判定する。

---

# 24. 新Layer追加ルール

新Ideaが出た時、すぐ新Layerを作らない。

最低限次を確認する。

1. 既存Roleへ入らないか
2. Submoduleで十分ではないか
3. 新Data Objectだけで解決できないか
4. Contract追加で解決できないか
5. State追加で解決できないか
6. Governance Rule追加で解決できないか
7. 本当に独立した責任・Lifecycleがあるか
8. 何十年維持する価値があるか

最後まで独立責任が残る場合のみ新Layer候補とする。

---

# 25. Role / Object / Layerを混同しない

例:

```text
Market Intelligence = Role / Domain Function
Market Context = Data Object
Feature Priority = Decision/Measurement Function
Trade Thesis = Data Object / Contract
```

Data Objectを理由なく独立Layer化しない。

---

# 26. ファイル名ルール

正式設計は内容が分かる名前を使用する。

良い例:

```text
GIT_RULES.md
ROLE_DICTIONARY.md
RESEARCH_GOVERNANCE.md
TRADE_THESIS_CONTRACT.md
```

避ける例:

```text
新しい設計.md
最終版.md
最終版2.md
new.md
test2.md
```

一時的なまとめ案を除き、「最終」「最新版」をファイル名へ多用しない。

Git履歴とVersionで管理する。

---

# 27. Versionルール

重要Schema / Contract / Formula等はVersionを持たせる。

例:

```text
schema_version: 1
contract_version: 3
formula_version: 7
```

ファイルそのものの細かい変更はGit履歴を使用する。

すべてのMarkdownファイル名に `v1`, `v2`, `v3` を付けて複製する運用は原則避ける。

必要な旧VersionはGit History / Archiveで管理する。

---

# 28. Commit Messageルール

Commit Messageは変更目的が分かるようにする。

推奨:

```text
Add Git governance rules
Define Market Intelligence role
Update Trade Thesis contract
Archive legacy AI meeting design
Fix risk state transition definition
```

避ける:

```text
update
fix
変更
test
いろいろ修正
```

日本語でも英語でもよいが、目的が特定できることを優先する。

---

# 29. Design Decisionを履歴として残す

重要変更では、後に `DECISION_LOG.md` 等へDecisionを保存する方向を採る。

形式候補:

```text
Decision ID:
Date:
Status:
Changed From:
Changed To:
Reason:
Alternatives:
Affected Files:
Affected Roles:
Migration Need:
Rollback Consideration:
```

Git Commitは「何が変わったか」を保存し、Decision Logは「なぜ変えたか」を保存する。

---

# 30. 同一内容を複数ファイルへ重複保存しない

例:

Roleの正式定義を、

```text
ROLE_DICTIONARY.md
SYSTEM_ARCHITECTURE.md
PROJECT_MANUAL.md
PYTHON_ARCHITECTURE.md
```

へ完全コピーしない。

原則:

- 正本を1つ決める
- 他文書は参照・要約する

何十年運用で定義ズレを防ぐ。

---

# 31. Single Source of Truth

重要概念は正本を明確化する。

候補:

```text
Role定義
→ ROLE_DICTIONARY.md

Object定義
→ OBJECT_DICTIONARY.md

State定義
→ STATE_DICTIONARY.md

Contract
→ 03_CONTRACTS/

全体構造
→ SYSTEM_ARCHITECTURE.md

コード構造
→ PYTHON_ARCHITECTURE.md
```

同じ内容が矛盾した場合、正本を基準に差分を解決する。

---

# 32. Gitへ保存しない情報

原則として以下をCommitしない。

- API Key
- Secret Key
- Password
- Wallet Seed / Private Key
- Authentication Token
- 個人認証情報
- 本番 `.env` の秘密値

必要なら、

```text
.env.example
```

へキー名だけ保存する。

例:

```text
EXCHANGE_API_KEY=
EXCHANGE_API_SECRET=
```

---

# 33. 大容量データをGitへ保存しない

市場理解OSは長期運用で大量データが発生する。

Gitへ直接保存しない候補:

- Raw market tick data
- 大量OHLCV
- Orderbook履歴
- 数百万Demo Trial
- DB本体
- 大型Model artifact
- Backup archive
- Log大量履歴

Gitは設計・コード・Schema・設定テンプレート・Migration・重要Decisionを中心に使う。

大量データはStorage / DB Lifecycle側で管理する。

---

# 34. Git管理対象とStorage管理対象を分ける

## Git

```text
Design
Code
Schema
Migration
Config Template
Test Code
Documentation
Decision
```

## Storage / DB

```text
Raw Data
Feature Values
Market Events
Hypothesis Evidence
Research Results
Demo Trials
Trade Results
Logs
Archive Data
```

この境界を守る。

---

# 35. コード作成前のGitルール

`.py` を新規作成・変更する前に可能な限り確認する。

1. 対応するCanonical Designは存在するか
2. 対応Roleは定義済みか
3. 入出力Contractは定義済みか
4. Stateは定義済みか
5. Error / Failure処理は定義済みか
6. Trace要件はあるか
7. Test条件はあるか
8. 新依存Libraryが必要か

未確定の場合は、コードへ暗黙仕様を埋め込まず、未確定事項として明示する。

---

# 36. コード変更時の報告

GPTがコードを変更した場合、ユーザーへ最低限次を伝える。

```text
変更:
- core/xxx.py

新規:
- contracts/yyy.py

変更なし:
- research/zzz.py

理由:
...
```

可能なら重要Function / Class単位も示す。

---

# 37. 自動コード修正禁止

市場理解OSがErrorを検知したからといって、GPT / AIが勝手にGitコードを書き換えてProductionへ反映しない。

禁止:

```text
Runtime Error
↓
AI自動修正
↓
Commit
↓
Production自動Deploy
```

正式変更は、

```text
Issue / Error
↓
Analysis
↓
Proposal / Fix
↓
Review
↓
Explicit Git Write
↓
Test
↓
Deploy
```

を基本とする。

---

# 38. ProductionとResearchのGit境界

Research用コード・設定変更がProductionを暗黙変更しないようにする。

例:

```text
Research Formula Candidate
```

を追加してもProduction Formula Registryを自動更新しない。

```text
Research
→ Candidate
→ Validation
→ Approval
→ Production Spec Update
→ Explicit Code Change
```

とする。

---

# 39. Rollback可能性

重要変更では、可能な限り戻せるようにする。

確認候補:

- DB Migrationは逆変換可能か
- Schema変更で旧データが読めるか
- Config変更を戻せるか
- Production Rule変更を戻せるか
- Adapter変更で旧Providerへ戻れるか

不可逆変更は明示する。

---

# 40. Schema Migrationを履歴管理する

何十年運用ではDB構造が必ず変わる。

Schemaを直接上書きして旧データを読めなくしない。

概念:

```text
Schema v1
↓ migration
Schema v2
↓ migration
Schema v3
```

Migration Script / Migration RuleをGit管理対象にする。

---

# 41. 依存関係変更を記録する

Python Library、API Client、DB Engine等を変更した場合、影響を記録する。

例:

```text
Dependency:
pydantic vX → vY

Affected:
contracts/
validation/
serialization/
```

何十年運用ではLibrary消滅・破壊的変更を前提にする。

---

# 42. Source / Vendor固有情報をCoreへ固定しない

Git上でも、

```text
Binance専用市場理解
```

のような密結合を避ける。

Adapter / ProfileへVendor差を隔離する。

将来Sourceが消滅してもCoreを維持できるようにする。

---

# 43. Merge / Conflict時の原則

設計文書でConflictが起きた場合、機械的に両方残してCanonicalを混ぜない。

確認順:

1. Governance
2. Canonical Architecture
3. Role / Contract正本
4. 最新Decision
5. Proposal
6. Legacy

不明ならConflict状態として残し、推測で決着しない。

---

# 44. GPTが不確かな場合

以下の場合、勝手に正式設計へ断定しない。

- 複数Canonical候補が矛盾
- 保存先が重要な意味を持ち決められない
- 既存ファイル内容と依頼が重大衝突
- 削除・破壊的変更が必要
- Security / Secretへの影響がある

ただし作業可能な安全範囲は進め、未確定部分を明示する。

---

# 45. 長期運用のためのGit原則

何十年運用では現在のGitHubや現在の開発環境が永続する保証はない。

そのため、Git Repository自体も定期的にBackup / Mirror可能な状態を維持する。

Git Hosting ProviderとGit Historyを同一視しない。

原則:

> GitHubが市場理解OSそのものではない。
> Git履歴とRepositoryを別環境へ移せるようにする。

---

# 46. Repository移行可能性

将来、

- GitHub
- Self-hosted Git
- 別Provider
- Local Mirror

へ移行しても、設計・コード・履歴が維持できる構造にする。

Provider専用機能へ過度に依存しない。

---

# 47. Git Backup

長期運用ではRepositoryもBackup対象。

候補:

```text
Primary Remote
+ Local Clone
+ Secondary Mirror
+ Periodic Archive
```

具体方式はBACKUP設計で決める。

---

# 48. 定期Architecture ReviewとGit

定期Reviewで、

- Canonicalとコードのズレ
- 未使用Proposal
- 長期HOLD
- Deprecated設計
- 壊れたLink
- 重複定義
- 古い依存
- Migration未実施

を確認する。

ただしReviewだけで勝手に大量修正しない。

Issue / Proposalとして分離する。

---

# 49. Git操作のDefinition of Done

1回のGit変更は、最低限次を確認して完了とする。

```text
□ ユーザーから明示的な書き込み指示がある
□ 対象Repositoryを確認した
□ 対象Fileを確認した
□ 既存Fileなら現在内容を読んだ
□ 変更目的が1つにまとまっている
□ 関係ないFileを触っていない
□ Secretを含んでいない
□ Canonical / Proposal状態を混同していない
□ Commit Messageが内容を表している
□ 書き込み後に対象Fileを確認した
□ ユーザーへ変更Fileを報告した
```

---

# 50. GPT Git操作の基本フロー

```text
User Request
↓
Git Write Permission確認
↓
Target Repository確認
↓
Target File検索 / 読み込み
↓
Current DesignとのConflict確認
↓
変更対象を宣言
↓
1目的でWrite
↓
再Fetchして確認
↓
変更File / 内容 / 未変更範囲を報告
```

---

# 51. 禁止事項一覧

GPT / 市場理解OSは次を行わない。

- 明示許可なしのGit書き込み
- 会話Ideaの自動Canonical化
- 未承認Research結果のProduction反映
- 1回の依頼で無関係Fileを大量更新
- 古い設計の無断削除
- API Key / SecretのCommit
- 大量Market DataのGit保存
- Errorを理由とした自動コード修正・自動Deploy
- AI提案のその場での本番実装
- Git履歴を無視した「最終版2」「最終版3」の乱立
- Conflictを推測で勝手に解消
- Codeから上位Governanceを暗黙変更

---

# 52. このルール自身の変更

`GIT_RULES.md` 自体も変更可能だが、通常ファイルより慎重に扱う。

変更候補フロー:

```text
Rule変更Proposal
↓
影響範囲確認
↓
ユーザー承認
↓
明示的Git反映指示
↓
GIT_RULES.md更新
↓
必要なら関連Governance Review
```

このファイルの変更を理由に、他ファイルを自動一括変更しない。

---

# 53. 最終原則

市場理解OSで最も避けるべきGit運用は、

> 「会話で出た案が、いつの間にか正式設計・コードになっている状態」

である。

理想は、

```text
Conversation
↓
Idea
↓
Review
↓
Proposal
↓
Approval
↓
Explicit Git Write
↓
Canonical
↓
Implementation Spec
↓
Code
↓
Test
↓
Production
```

という履歴が追跡可能なこと。

何十年後でも、

> 何を考え、なぜ採用し、何を捨て、どの設計からどのコードが生まれたのか

を復元できるGit運用を市場理解OSの正式原則とする。
