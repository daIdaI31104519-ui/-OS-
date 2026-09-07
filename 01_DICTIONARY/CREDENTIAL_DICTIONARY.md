# CREDENTIAL_DICTIONARY.md

# 市場理解OS Credential / Secret Governance Dictionary

## 0. 文書情報

- 文書種別: DICTIONARY / CREDENTIAL SECURITY SEMANTIC SOURCE OF TRUTH CANDIDATE
- 状態: CANONICAL DICTIONARY CANDIDATE
- FIX: FIX-018B
- 上位Security原則:
  - `01_DICTIONARY/SECURITY_DICTIONARY.md`
- 関連:
  - `01_DICTIONARY/OBJECT_DICTIONARY.md`
  - `01_DICTIONARY/ROLE_DICTIONARY.md`
  - `01_DICTIONARY/STATE_DICTIONARY.md`
  - `00_GOVERNANCE/GIT_RULES.md`
  - `00_GOVERNANCE/DESIGN_CHANGE_RULES.md`
- 目的: API Key / Token / Service Credential等を単なる秘密文字列として扱わず、「何の資格か・誰が・どの環境で・何に対して・どこまで利用可能か」をVersion付きで管理し、Research / Production / Execution / Telegram / AI間のCredential越境を防ぐ

---

# 1. FIX-018Bの目的

FIX-018Aでは次を分離した。

```text
Principal
≠ Role
≠ Permission
≠ Authentication
≠ ApprovalDecision
≠ Applied Action
```

FIX-018Bでは外部Providerへアクセスする資格をさらに分離する。

正式原則:

```text
Secret Value
≠ Secret Reference
≠ CredentialProfile
≠ Domain Permission
≠ Provider Capability
```

Credentialを持っていること自体を、Domain Actionの許可とみなさない。

```text
Credential ACTIVE
≠ Trade Authorized
```

---

# 2. 最重要Credential原則

## CRED-RULE-001: Secret ValueをCanonical設計Objectへ保存しない

次の実値をGit / CredentialProfile / 通常設計DB / Logへ保存しない。

```text
API_KEY
API_SECRET
TOKEN
PASSWORD
PRIVATE_KEY
BOT_TOKEN
BEARER_TOKEN
```

Git上では論理参照だけを扱う。

```text
secret_ref
secret_generation_ref
```

---

## CRED-RULE-002: Secret Value ≠ Secret Reference

```text
Secret Value
= 実際の秘密情報

Secret Reference
= Secret Store等に保存された秘密情報を指す論理参照
```

例:

```text
SECRET-EXCHANGE-PROD-EXEC-001
```

はSecret Valueではない。

---

## CRED-RULE-003: CredentialProfile ≠ Secret Value

`CredentialProfile`は資格の意味・用途・Scope・Capability・Lifecycleを表す管理Objectであり、秘密値の保管Objectではない。

---

## CRED-RULE-004: Credential ≠ Domain Permission

FIX-018AのPermissionがDomain Actionの資格を管理し、CredentialProfileはProvider側資格を管理する。

```text
Domain Permission
= OS上で今回のActionを行う資格

Credential Provider Capability
= Provider側資格として何を技術的に実行可能か
```

両方必要。

---

## CRED-RULE-005: Research Credential ≠ Production Credential

```text
RESEARCH
≠ DEMO
≠ PRODUCTION
≠ OPERATIONS
```

同じProviderでもEnvironmentが異なる場合、原則別Credentialを使用する。

Research ServiceへProduction Credentialを渡さない。

---

## CRED-RULE-006: Production Decision ≠ Credential Holder

Production Thesis Builder / Signal Engine等のProduction Decision系は、原則としてExchange Execution Credentialを保持しない。

```text
Production Decision
= なぜRiskを取りたいか判断

Execution
= 有効なOrderIntentを外部Exchangeへ実行
```

Production Decision Principalの外部Execution Credential数は原則0を目標とする。

---

## CRED-RULE-007: Exchange Read ≠ Exchange Execution

Production Exchange Credentialは最低限次へ分離する。

```text
READ Credential
≠ EXECUTION Credential
```

Position / Balance / Order Reconciliation等のRead処理へCreate Order権限を与えない。

---

## CRED-RULE-008: WithdrawalはDefault Forbidden

市場理解OSの通常Production Executionに出金権限を必要としない。

正式原則:

```text
WITHDRAW
TRANSFER
MANAGE_API_KEYS
= DEFAULT FORBIDDEN
```

Provider仕様上可能な場合、Credential発行段階で無効化する。

---

## CRED-RULE-009: Research AI ≠ Production Review AI

同じAI ProviderでもCredentialを分離する。

```text
Research AI Credential
≠ Production Review AI Credential
```

Security Scope / Cost Budget / Rate Limit / Data Exposureを分離可能にする。

---

## CRED-RULE-010: Credential Rotation ≠ Risk Recovery ≠ Production Resume

Credentialを交換・再有効化しても、それだけでRiskStateやProduction Promotionを自動復旧しない。

```text
Credential Recovery
≠ Risk Recovery
≠ Production Resume
```

---

# 3. SEC-CRED-OBJ-001: CredentialProfile

## Meaning

外部Providerへアクセスする一つの論理Credential用途について、Identity / Version / Purpose / Environment / Principal Binding / Scope / Provider Capability / Secret Reference / Lifecycle / Validationを保持するVersioned Security Object。

`CredentialProfile`はSecret Valueを含まない。

## Owner

Security / Credential Governance

## Consumers

- Credential Usage Gate
- Credential Resolver
- Exchange Adapter
- Collector / Source Adapter
- Telegram Adapter
- External AI Adapter
- Runtime / Monitoring
- Security / Incident
- Audit / Governance

## Main Fields

```yaml
credential_id:
credential_version:

provider:
credential_type:
purpose:
environment:

authorized_principal_refs: []

market_scope_refs: []
account_scope_refs: []
resource_scope_refs: []

provider_capabilities: []
forbidden_provider_capabilities: []

secret_ref:
secret_generation_ref:

credential_state:
state_machine_version:
latest_transition_event_ref:

issued_at:
valid_from:
expires_at:

rotation_policy_ref:

last_validated_at:
validation_result_ref:

created_at:
```

型・Required / Nullable・Cardinality・Indexは後続Data Contract / DB / Security implementationで固定する。

---

# 4. Identity Fields

## `credential_id`

Credential用途のStable Logical ID。

例:

```text
CRED-EXCHANGE-PROD-EXEC-001
CRED-EXCHANGE-PROD-READ-001
CRED-AI-RESEARCH-001
CRED-TELEGRAM-CONTROL-001
```

Secret値がRotationしても、論理用途が同じならCredential Identityを追跡可能にする。

## `credential_version`

CredentialProfile設定内容のVersion。

例:

```text
v1 = BTC Scope only
v2 = BTC + ETH Scope
```

Principal Binding / Purpose / Capability / Scope等の意味変更を同じVersionへ無言で上書きしない。

正式原則:

```text
credential_id
= logical credential purpose identity

credential_version
= configuration generation
```

---

# 5. Provider / Type / Purpose / Environment

## `provider`

Credentialが属する外部Provider。

候補例:

```text
BINANCE
BYBIT
OPENAI
TELEGRAM
NEWS_PROVIDER_X
```

具体Provider名はMarket / Adapter Designで追加可能。

## `credential_type`

認証方式。

候補:

```text
API_KEY_PAIR
BEARER_TOKEN
BOT_TOKEN
OAUTH
SERVICE_CREDENTIAL
```

Credential TypeとProvider Capabilityを混同しない。

## `purpose`

このCredentialが何のために存在するか。

候補:

```text
MARKET_DATA_READ
ACCOUNT_READ
TRADE_EXECUTION
TELEGRAM_CONTROL
EXTERNAL_AI_RESEARCH
EXTERNAL_AI_PRODUCTION_REVIEW
```

`provider = BINANCE`だけではPurposeとして不十分。

## `environment`

正式候補:

```text
RESEARCH
DEMO
PRODUCTION
OPERATIONS
```

Environment越境を禁止する。

```text
Credential.environment = RESEARCH
→ Productionから原則利用不可
```

---

# 6. Principal Binding

## `authorized_principal_refs`

Credentialを取得・利用する候補Principalを明示する。

例:

```yaml
authorized_principal_refs:
  - PRINCIPAL-EXECUTION-001
```

ただしPrincipal Bindingだけで利用を許可しない。

正式Gate:

```text
FIX-018A Domain Permission
AND
Credential authorized_principal_refs
AND
Credential Scope
AND
Credential Lifecycle State
```

を満たす必要がある。

`authorized_principal_refs`を万能Role Listとして利用しない。

---

# 7. Scope Fields

## `market_scope_refs`

CredentialをOS上どのMarket / Instrumentへ利用可能とするか。

例:

```text
BTCUSDT
ETHUSDT
```

Provider側でMarket単位制限が不可能でもOS PolicyとしてGate可能にする。

## `account_scope_refs`

対象Account / Subaccount / Portfolio等のScope。

例:

```text
ACCOUNT-BTC-PROD-001
```

別Accountへ無制限に流用しない。

## `resource_scope_refs`

必要に応じてProvider Resource範囲を表す。

候補:

```text
SPOT
PERPETUAL
FUTURES
```

FIX-018BではSemantic候補とし、Required / Nullableは後続Contractへ送る。

---

# 8. Provider Capability

## `provider_capabilities`

Provider側でそのCredential自体が技術的に持つ能力。

Production Read例:

```text
READ_ACCOUNT
READ_BALANCE
READ_POSITION
READ_ORDER
```

Production Execution例:

```text
CREATE_ORDER
CANCEL_ORDER
READ_ORDER
READ_POSITION
```

Research Data例:

```text
READ_MARKET_DATA
```

## `forbidden_provider_capabilities`

Credential Purpose上、明示的に禁止するCapability。

Exchangeでは原則:

```text
WITHDRAW
TRANSFER
MANAGE_API_KEYS
```

を含める。

ProviderがCapability無効化をサポートしない場合も、OS側Policyで禁止を明示する。

---

# 9. Effective External Capability

外部Actionが実行可能であるためには、単一条件ではなく次のIntersectionを要求する。

```text
Effective External Capability
=
FIX-018A Domain Permission
∩ Credential authorized principal binding
∩ Credential environment
∩ Credential market/account/resource scope
∩ Provider Capability
∩ Current Credential Lifecycle State
∩ Current Security Policy
```

一つでも不成立なら外部Credential利用をDENYする。

---

# 10. Secret Reference / Secret Generation

## `secret_ref`

Secret Store等にある秘密情報を指す論理参照。

```text
secret_ref
≠ Secret Value
```

## `secret_generation_ref`

同じCredential用途でRotationされたSecret世代を識別するTrace Reference。

例:

```text
GEN-008
→ GEN-009
```

正式原則:

```text
Credential Purpose Identity
≠ Secret Generation
```

過去Audit / Executionから「どのCredential世代を利用したか」を追跡できる設計を目標とするが、Secret Value自体は保存しない。

---

# 11. Secret Safety Rules

Secret Valueを次へ出さない。

```text
Git
CredentialProfile
normal DB columns
logs
ErrorEvent message
Diagnostics free text
ApprovalDecision
StateTransitionEvent
AI prompt / AI review input
ResearchResult
Backup manifest
```

Logへ必要な場合は:

```text
credential_id
credential_version
secret_generation_ref
```

のみ記録し、Secret ValueはREDACTする。

---

# 12. Credential Lifecycle State

FIX-018B正式候補:

```text
ISSUED
ACTIVE
ROTATING
EXPIRED
COMPROMISED
REVOKED
```

Credential LifecycleはProvider資格の利用状態だけを表す。

```text
Credential Lifecycle
≠ Risk State
≠ Production Promotion Stage
≠ Runtime State
```

---

# 13. State Meaning

## ISSUED

Credential / Secretが発行済みだが、Production等で利用可能状態へまだ昇格していない。

`ISSUED`だけでCredential Usage Gateを通さない。

## ACTIVE

必要Validation / Approvalを通過し、定義されたEnvironment / Principal / Scope / Capability内で利用可能。

`ACTIVE`でもDomain Permissionがなければ外部Actionは実行不可。

## ROTATING

新Secret Generation / Credential Versionへの安全な切替中。

Rotation中の旧Primary / Candidate Generationの詳細表現はData Contractで固定する。

## EXPIRED

時間Policy等により有効期限切れ。

利用不可。

## COMPROMISED

Secret流出・侵害・不正利用が確認または十分強く疑われる状態。

Credential Usage Gateは即DENY候補。

## REVOKED

Provider / OS Governance上、正式に利用資格を失効した状態。

履歴は削除しない。

---

# 14. Credential Lifecycle原則

通常候補:

```text
ISSUED
↓ Validation + Approval
ACTIVE
├→ ROTATING
├→ EXPIRED
├→ COMPROMISED
└→ REVOKED

ROTATING
├→ new generation / version validated and activated
├→ COMPROMISED
└→ REVOKED

COMPROMISED
→ REVOKED
```

重要:

Credential Profile Version / Secret Generation切替とCredential Lifecycle Transitionを混同しない。

---

# 15. Forbidden Lifecycle Transitions

正式禁止:

```text
COMPROMISED → ACTIVE
REVOKED → ACTIVE
EXPIRED → ACTIVE
```

壊れた・失効したCredentialを再生しない。

必要なら新Credential Version / 新Secret Generationを発行する。

```text
Old Credential
COMPROMISED
→ REVOKED

New Credential / Generation
ISSUED
→ Validation
→ ACTIVE
```

---

# 16. Credential Lifecycle Authority

FIX-013 / FIX-015と同じ4責任を適用する。

```text
REQUEST
≠ RECOMMEND
≠ APPROVE
≠ APPLY
```

正式原則:

```text
Credential Lifecycle Request Authority
≠ Credential Lifecycle Recommend Authority
≠ Credential Lifecycle Approve Authority
≠ Credential Lifecycle Apply Authority
```

## Apply Authority

```text
Credential Controller
= Credential Lifecycle single-writer Apply Authority
```

これは新しい巨大Layerではなく、Credential State Machineへ実際にCurrent Stateを書き込む論理責任。

Monitoring / Human / Runtime / Security GovernanceがCredential Stateへ直接書き込まない。

---

# 17. Credential Authority Matrix

| Transition | Request Authority候補 | Recommend Authority | Approve Authority | Apply Authority |
|---|---|---|---|---|
| ISSUED → ACTIVE | Security Operator / Credential Provisioning | Security Validation | Security Governance | Credential Controller |
| ACTIVE → ROTATING | Rotation Policy / Security | Security Governance | Security Governance / Versioned Rotation Policy | Credential Controller |
| ROTATING → new ACTIVE generation/version | Credential Validation | Security Validation | Security Governance | Credential Controller |
| ACTIVE → COMPROMISED | Monitoring / Security / Human / Incident | Security Detector / Security Governance | Automated Safety Policy可 | Credential Controller |
| ROTATING → COMPROMISED | Monitoring / Security / Incident | Security Detector | Automated Safety Policy可 | Credential Controller |
| ACTIVE → REVOKED | Security / Human / Incident | Security Governance | Security Governance / Safety Policy | Credential Controller |
| COMPROMISED → REVOKED | Security / Incident | Security Governance | Automated Safety Policy可 | Credential Controller |
| ACTIVE → EXPIRED | Expiry / Time Policy | Policy Engine | Deterministic Automated Policy可 | Credential Controller |

Authority identity / exact Role IDs / IAM implementationは後続Security Contractで固定する。

---

# 18. Restrictive Fast Path

危険側TransitionはHuman待ちでSafetyを遅らせない。

候補:

```text
ACTIVE → COMPROMISED
ACTIVE → REVOKED
ROTATING → COMPROMISED
COMPROMISED → REVOKED
ACTIVE → EXPIRED
```

明示されたVersioned Security Policyにより:

```text
Automated Policy ApprovalDecision
↓
Credential Controller
↓
StateTransitionEvent
```

を許可可能。

ただしFast Pathでも:

```text
Authority Check
Policy Reference
ApprovalDecision provenance
StateTransitionEvent
Audit / Trace
```

を省略しない。

---

# 19. Activation / Permission Expansion Strict Path

次はRestrictive Transitionより厳しくする。

```text
ISSUED → ACTIVE
ROTATING → new ACTIVE generation/version
```

最低候補Gate:

```text
Credential exists
Secret Reference valid
Principal Binding valid
Environment correct
Provider Capability verified
Forbidden Capability absent
Account Scope verified
Market Scope verified
Validation passed
Required ApprovalDecision exists
Current State / expected_previous_state matches
```

一つでも不成立ならApplyしない。

---

# 20. Credential Apply-Time Validation

Credential ControllerはApply直前に最低限次を確認する。

```text
Target Credential一致
Credential Version一致
State Machine / Version一致
Current State一致
expected_previous_state一致
Requested Transition合法
Required ApprovalDecision有効
Scope一致
Expiry条件一致
Superseded Approvalでない
Current Security Policy一致
```

成功したTransitionだけ`StateTransitionEvent`を生成する。

ApprovalDecisionが存在してもApply Preconditions FailedならStateTransitionEventを生成しない。

---

# 21. Credential Usage Gate

Credentialを外部Provider Callへ使用する直前に最低候補として次を確認する。

```text
1. Principal authenticated?
2. FIX-018A Domain Permission valid?
3. Credential authorized principal binding valid?
4. Environment matches?
5. Purpose matches requested operation?
6. Requested Capability in provider_capabilities?
7. Requested Capability not in forbidden_provider_capabilities?
8. Market Scope matches?
9. Account Scope matches?
10. Resource Scope matches if required?
11. credential_state = ACTIVE?
12. valid_from reached?
13. expires_at not passed?
14. Validation / Security policy satisfied?
```

全て通過した場合のみSecret Resolverへ進む。

---

# 22. Credential Resolver Boundary

`Credential Resolver`はMechanismでありAuthorityではない。

```text
Credential Resolver
= Authorized Credential ReferenceをSecret Storeの実Secretへ一時解決するMechanism
```

正式原則:

```text
Credential Resolver
≠ Permission Authority
≠ Approver
≠ Credential Lifecycle Apply Authority
```

Resolverが独自判断でPrincipalへSecretを配布しない。

---

# 23. Secret Runtime Handling

実装段階では原則:

```text
Secret Store
↓
Credential Resolver
↓
authorized runtime process memory
↓
Provider Call
↓
possible immediate discard / minimal retention
```

を目標とする。

禁止候補:

```text
Secretを通常DBへコピー
SecretをLogへ出力
SecretをExceptionへ全文埋め込み
SecretをAIへ送信
SecretをGitへCommit
```

具体Secret Store製品 / OS Keychain / Vault等はFIX-018BではCanonical固定しない。

---

# 24. Initial Credential Allocation

前提例:

```text
1 authenticated Market Data Provider
1 Exchange
1 Telegram Bot
1 External AI Provider
```

## Research

基本:

```text
CRED-DATA-RESEARCH-READ-001
```

目的:

```text
MARKET_DATA_READ
```

Public unauthenticated APIだけならCredential 0も成立する。

Private Account DataがResearchで必要な場合でもProduction Execution Credentialを共有せず、別Read-only Credentialを検討する。

## Production Decision

原則:

```text
External Exchange Credential = 0
```

Production Thesis Builder / Signal Engine等は内部Objectを利用し、Exchangeへ直接接続しない。

## Execution

最低2つを推奨:

```text
CRED-EXCHANGE-PROD-READ-001
CRED-EXCHANGE-PROD-EXEC-001
```

Read Credential:

```text
READ_ACCOUNT
READ_BALANCE
READ_POSITION
READ_ORDER
```

Execution Credential:

```text
CREATE_ORDER
CANCEL_ORDER
READ_ORDER
必要に応じREAD_POSITION
```

両方:

```text
WITHDRAW = FORBIDDEN
TRANSFER = FORBIDDEN
MANAGE_API_KEYS = FORBIDDEN
```

## Telegram

```text
CRED-TELEGRAM-CONTROL-001
```

Provider Capability:

```text
SEND_MESSAGE
RECEIVE_UPDATE
TELEGRAM_API
```

Telegram CredentialはRisk / Production State変更権限そのものではない。

## AI

最低2つ:

```text
CRED-AI-RESEARCH-001
CRED-AI-PROD-REVIEW-001
```

Research AIとProduction Review AIのCredential / Budget / Scopeを分離する。

---

# 25. Base Credential Count

上記前提では基本:

```text
Research                  1
Production Decision       0
Execution                 2
Telegram                  1
AI                        2
--------------------------------
Base Total                6
```

Public Data ProviderだけならResearch Credentialは0になり得る。

Credential数は固定定数ではなく:

```text
Provider
× Environment Boundary
× Privilege Boundary
× Scope Boundary
```

から決める。

Service数だけを理由にCredentialを増やしたり共有したりしない。

---

# 26. Demo Credential

Demo / Testnet等でProvider Credentialが必要ならProductionと共有しない。

候補:

```text
CRED-EXCHANGE-DEMO-READ-001
CRED-EXCHANGE-DEMO-EXEC-001
```

```text
Demo Credential
≠ Production Credential
```

Demo有効時はBase Totalへ+2候補。

---

# 27. Credential Blast Radius Principle

一つのCredential流出がOS全体へ波及しないよう、Capability / Principal / Environment / Provider単位で分割する。

悪い例:

```text
ONE_API_KEY
├→ Collector
├→ Research
├→ Production
├→ Execution
└→ Analyzer
```

推奨:

```text
Market Data Credential
→ Collector / authorized Research Data service

Research AI Credential
→ Research AI

Production Review Credential
→ Production AI Review

Exchange Read Credential
→ Reconciliation / Position read

Exchange Execution Credential
→ Exchange Adapter

Telegram Credential
→ Telegram Adapter
```

---

# 28. Credential Compromise Flow

Production Exchange Execution Credentialの侵害疑い例:

```text
Suspicious Credential Activity
↓
Incident / Evidence
↓
REQUEST ACTIVE → COMPROMISED
↓
Security Detector RECOMMEND
↓
Automated Safety ApprovalDecision
↓
Credential Controller APPLY
↓
Credential = COMPROMISED
↓
StateTransitionEvent
↓
Credential Usage DENY
↓
Purpose-specific Impact Routing
```

その後:

```text
Old Credential REVOKED
↓
New Credential / Secret Generation ISSUED
↓
Validation
↓
Strict Approval
↓
ACTIVE
```

旧Compromised CredentialをACTIVEへ戻さない。

---

# 29. Purpose-Specific Incident Impact

Credential障害をUniversal OS Emergencyへ無条件変換しない。

## Exchange Execution Credential

```text
COMPROMISED
→ External order execution block
→ Incident
→ Risk Restriction Request candidate
```

RiskState変更はRisk State Controllerの責任。

```text
Credential Controller
≠ Risk State Controller
```

## Telegram Credential

```text
COMPROMISED
→ Telegram Command Channel disable / isolate
```

Telegram障害だけでResearch / Production Trading Engine全体を無条件停止しない。

```text
Telegram Failure
≠ OS Failure
```

## Research AI Credential

```text
COMPROMISED
→ Research AI usage stop
```

Production Trading本体へ無条件波及しない。

## Production AI Review Credential

AI ReviewがAdvisory / Optionalなら:

```text
Credential failure
→ AI Review unavailable / degraded
```

としてCore Production Pathを継続可能にする設計候補。

## Market Data Credential

```text
Credential failure
→ affected Source availability / Data Quality review
```

Source Lifecycle変更はSource Lifecycle Authority Flowへ送る。

---

# 30. Credential Incident ≠ Knowledge Failure

Credential侵害・失効だけを理由に:

```text
Hypothesis RETIRED
Edge RETIRED
Knowledge DEGRADED
```

へ直接変更しない。

Credential障害はSecurity / Source / Execution / Runtime / Riskの問題であり、Market Knowledgeの正否とは別。

---

# 31. Rotation Semantics

RotationをSecret Valueの無言上書きとして扱わない。

概念上:

```text
Credential Purpose
CRED-EXCHANGE-PROD-EXEC-001

Generation 8
↓ Rotation
Generation 9
```

をTrace可能にする。

候補Flow:

```text
Current Generation ACTIVE
↓
New Secret Generation issued
↓
ROTATING
↓
Capability / Scope / Authentication validation
↓
Cutover
↓
New generation / profile version ACTIVE
↓
Old generation revoked
```

Zero-downtime Rotation時のCurrent / Candidate Generation同時保持方法はData Contractで固定する。

---

# 32. Expiry

`expires_at`が設定される場合、期限切れをHuman判断だけに依存しない。

候補:

```text
Expiry Policy Trigger
↓
Automated Policy ApprovalDecision
↓
Credential Controller
↓
ACTIVE → EXPIRED
```

期限切れCredentialを自動ACTIVEへ戻さない。

---

# 33. Audit / Trace

重要Credential操作で追跡可能にする候補:

```text
credential_id
credential_version
secret_generation_ref
principal_ref
role_ref
requested_action
provider
purpose
environment
scope
permission result
credential lifecycle state
approval_decision_refs
state_transition_event_ref
incident_ref
occurred_at
```

Secret ValueはAuditへ保存しない。

---

# 34. Credential Usage and Trade Boundary

Order executionではCredential Gateの前にTrading Gateが必要。

概念上:

```text
Valid OrderIntent
+ EntryThesis
+ DefenseDecision
+ Current Risk Permission
+ Execution Domain Permission
+ Credential Usage Gate
↓
Credential Resolver
↓
Exchange Adapter
↓
Exchange
↓
ExecutionRecord
```

正式原則:

```text
Credential ACTIVE
≠ Trade Authorized
```

---

# 35. Credential and AI Data Boundary

AI CredentialがACTIVEでも、Secret / sensitive dataをAIへ送信可能という意味ではない。

```text
AI Credential Availability
≠ AI Data Exposure Permission
```

AIへ送信可能なData Classification / SanitizationはFIX-018C / Security Data Governanceで固定する。

---

# 36. Backup Boundary

CredentialProfile / Audit / Secret Reference等のMetadata Backupと実Secret Backupを分離する。

```text
Credential Metadata Backup
≠ Secret Store Backup
```

Secret Store Backup / Encryption / Restore Authorityは後続FIX-018Dへ送る。

---

# 37. Boot / Runtime Retrieval Principle

OS起動時に全Secretを全Processへ読み込まない。

原則:

```text
Need-to-use retrieval
+
Least Privilege
```

例:

```text
Research Service
→ Research Credential only

Exchange Adapter
→ Production Execution Credential only

Telegram Adapter
→ Telegram Credential only
```

一つのService侵害時のBlast Radiusを制限する。

---

# 38. Credential Object追加Gate

FIX-018Bでは正式Persistent Security Objectを原則:

```text
CredentialProfile
```

へ限定する。

次は現時点で独立Persistent Objectとして確定しない。

```text
SecretReferenceObject
CredentialResolverObject
CredentialUsageContextObject
CredentialValidationResultObject
```

扱い候補:

```text
Secret Reference
= Value / Reference

Credential Resolver
= Mechanism / Service responsibility

Credential Usage Context
= Ephemeral Runtime Security Context

Credential Validation Result
= 後続ContractでDiagnostics / Audit / 独立Object要否を再評価
```

Security Objectを理由なく増殖させない。

---

# 39. FIX-018Bで作らないもの

```text
Secret Manager Layer as universal authority
Universal Credential Admin
Secret Value Database
Credential-based Trade Decision
AI Secret Store in Git
```

Secret Store / Vault製品は実装技術でありSemantic Authorityではない。

---

# 40. Data Contractへ送る項目

後続Data / Processing / Security Contractで固定する。

```text
CredentialProfile field type
Required / Nullable
Credential ID namespace
Credential versioning rule
secret_ref type
secret_generation_ref cardinality
Principal Binding cardinality
Environment enum
Purpose enum
Provider Capability vocabulary
Forbidden Capability enforcement
Market / Account / Resource scope representation
Credential Usage Gate input/output
Credential Resolver interface
Credential State Transition atomicity
StateTransitionEvent relation
ApprovalDecision requirements
Rotation current/candidate generation representation
Expiry time semantics
Validation result representation
Audit cardinality
```

---

# 41. DB / Pythonへ送る項目

FIX-018BではTable / ClassをCanonical固定しない。

禁止例:

```python
API_KEY = "real-secret"
```

禁止例:

```python
if credential.state == "ACTIVE":
    exchange.create_order(...)
```

正しくは概念上:

```text
Domain Permission
AND Credential Principal Binding
AND Environment / Scope
AND Provider Capability
AND Credential State
AND Trading / Risk Gates
```

を確認する。

---

# 42. Security / State Integration

Credential Lifecycleは将来`STATE_DICTIONARY.md`とのCross Checkで正式State Machine ID / Authority Matrix参照を統合する。

FIX-018BではCredential専用Semantic Source of Truthとして本辞書へ状態意味とAuthorityを固定し、既存`STATE_DICTIONARY.md`を全文書換えしない。

成功Credential State Transitionは既存:

```text
OBJ-STATE-001: StateTransitionEvent
```

を利用する。

Approvalは既存:

```text
OBJ-GOV-001: ApprovalDecision
```

を利用する。

新しいCredentialStateHistory / CredentialApproval Objectを作らない。

---

# 43. FIX-018Aとの接続

FIX-018A:

```text
Principal
↓
Authentication
↓
Role Binding
↓
Permission Policy
```

FIX-018B:

```text
Credential Usage Permission
↓
CredentialProfile
↓
Credential Usage Gate
↓
Credential Resolver
↓
Secret Store
↓
External Provider
```

統合:

```text
SecurityPrincipal
↓
Authentication
↓
Role Binding
↓
Domain Permission
↓
Credential Principal / Scope / Capability Gate
↓
Credential State ACTIVE
↓
Secret Resolution
↓
External Action
```

---

# 44. FIX-018B Definition of Done

Credential設計をSemantic Levelで完了とみなす最低条件:

```text
□ CredentialProfile Meaning
□ Secret Valueとの分離
□ Secret Referenceとの分離
□ Credential Version
□ Secret Generation
□ Provider
□ Credential Type
□ Purpose
□ Environment
□ Principal Binding
□ Market / Account / Resource Scope
□ Provider Capability
□ Forbidden Capability
□ Credential Lifecycle State
□ Request Authority
□ Recommend Authority
□ Approve Authority
□ Apply Authority
□ Restrictive Fast Path
□ Activation Strict Path
□ Forbidden recovery transitions
□ Rotation semantics
□ Expiry semantics
□ Compromise handling
□ Purpose-specific incident impact
□ Risk / Runtime / Sourceとの責任境界
□ Audit / Trace
□ Base credential allocation
□ Demo separation
□ Withdrawal default forbidden
□ Data Contract TODO
□ Secret Store implementation deferred
```

---

# 45. 最終原則

市場理解OSではCredentialを「便利なAPI Key」として扱わない。

```text
CredentialProfile
= 外部Provider資格の意味・Scope・Capability・Lifecycle

Secret Value
= 外部Secret Storeで保護される秘密

Domain Permission
= PrincipalがOS上でActionを行う資格

Provider Capability
= Credential自体がProvider上で実行できる操作
```

正式原則:

```text
Secret Value
≠ Secret Reference
≠ CredentialProfile
≠ Domain Permission
≠ Provider Capability
```

さらに:

```text
Research Credential
≠ Demo Credential
≠ Production Credential

Production Decision
≠ Execution Credential Holder

Exchange Read
≠ Exchange Execution

Research AI
≠ Production Review AI

Credential ACTIVE
≠ Trade Authorized

Credential Rotation
≠ Risk Recovery
≠ Production Resume
```

Credential Lifecycleは:

```text
REQUEST
→ RECOMMEND
→ APPROVE
→ ApprovalDecision
→ Credential Controller APPLY
→ StateTransitionEvent
```

を通し、Credential Controllerをsingle-writer Apply Authorityとする。

危険側Transitionは明示されたAutomated Safety Policyで速く制限できるが、Activation / Permission ExpansionはValidation + Strict Approvalを要求する。

これにより、一つのCredential流出・誤設定・Provider障害がResearch / Production / Execution / Telegram / AI全体へ無制限に波及する構造を避ける。