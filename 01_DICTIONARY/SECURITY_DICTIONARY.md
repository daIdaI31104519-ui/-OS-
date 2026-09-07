# SECURITY_DICTIONARY.md

# 市場理解OS Security Identity / Authorization Dictionary

## 0. 文書情報

- 文書種別: DICTIONARY / SECURITY SEMANTIC SOURCE OF TRUTH CANDIDATE
- 状態: CANONICAL DICTIONARY CANDIDATE
- FIX: FIX-018A
- 関連:
  - `01_DICTIONARY/ROLE_DICTIONARY.md`
  - `01_DICTIONARY/OBJECT_DICTIONARY.md`
  - `01_DICTIONARY/STATE_DICTIONARY.md`
  - `00_GOVERNANCE/GIT_RULES.md`
  - `00_GOVERNANCE/DESIGN_CHANGE_RULES.md`
- 目的: 市場理解OSで「誰が・どの責任として・何を・どこで・どこまで操作してよいか」を、Role / Approval / State / Interfaceから分離して正式化する

---

# 1. FIX-018Aの目的

FIX-013〜FIX-017までで、市場理解OSは次を分離した。

```text
REQUEST
≠ RECOMMEND
≠ APPROVE
≠ APPLY

ApprovalDecision
≠ StateTransitionEvent

Research maturity
≠ Knowledge freshness / health
≠ Production permission
≠ Current Risk permission
```

FIX-018Aではその前段として、操作主体と権限を分離する。

```text
誰なのか
↓
本当にそのIdentityなのか
↓
どのRole責任を持つのか
↓
何を許可されているのか
↓
今回の操作条件を満たすか
↓
必要ならAuthority / Approval Flowへ進む
```

FIX-018Aの正式原則:

```text
Principal
≠ Role
≠ Permission
≠ Authentication
≠ ApprovalDecision
≠ Applied Action
```

---

# 2. 最重要Security原則

## SEC-RULE-001: Default Deny

```text
Permissionが明示されていない
→ DENY
```

禁止されていないから許可、という設計を採用しない。

正式原則:

```text
DEFAULT DENY
+
EXPLICIT ALLOW
```

---

## SEC-RULE-002: Principal ≠ Role ≠ Permission

```text
Principal
= 誰か

Role
= 何の責任を持つか

Permission
= 何をしてよいか
```

Role名だけを理由に万能権限を与えない。

---

## SEC-RULE-003: Authentication ≠ Authorization

```text
Authentication
= 本当にそのPrincipalか

Authorization
= そのPrincipalが今回のActionを行ってよいか
```

Authentication成功だけでActionを許可しない。

---

## SEC-RULE-004: Permission ≠ ApprovalDecision

```text
Permission
= ApprovalDecisionを生成する資格

ApprovalDecision
= 特定Target / Version / Transitionに対して実際に下した一回のGovernance Decision
```

APPROVE Permissionを持つこと自体は、どのTransitionも承認済みという意味ではない。

---

## SEC-RULE-005: Permission ≠ Applied Action

```text
Permission
= 実行資格

Apply / Execute
= 実際の操作

StateTransitionEvent / ExecutionRecord / AuditEvent
= 実際に起きたことの履歴・証拠
```

---

## SEC-RULE-006: Research Authority ≠ Production Authority

Research Serviceが研究結果を生成できても、Production Orderを実行できることを意味しない。

```text
Research
→ ResearchResult / EvidencePackage
→ Request / Recommend
→ Approval
→ Production Promotion
→ Production
```

ResearchからProductionへ直接State / Order / Credentialを書き込まない。

---

## SEC-RULE-007: Restrict Permission ≠ Recover Permission

Safety RestrictionとPermission Expansionを別Permissionとして扱う。

例:

```text
REQUEST_RISK_RESTRICTION
≠ APPROVE_RISK_RECOVERY

PAUSE_RUNTIME
≠ RESUME_RUNTIME

PAUSE_PRODUCTION
≠ RESTORE_NORMAL_LIVE
```

Restrictive Fast PathがあってもRecoveryを同じ強さで許可しない。

---

## SEC-RULE-008: Interface / Channel ≠ Authority

```text
Telegram
Web UI
CLI
API Endpoint
```

は入力Channel / Adapterであり、Principal / Role / Approver / Apply Authorityそのものではない。

```text
Client Claims
≠ Trusted Authority
```

Role / Principal / ScopeはServer側のTrusted Bindingから解決する。

---

## SEC-RULE-009: AI ≠ Authority

AIはAdvisory / Research補助Principalになり得るが、明示的な別Governance設計なしに:

```text
APPROVE
APPLY
EXECUTE_ORDER
READ_SECRET
DELETE
```

を許可しない。

---

## SEC-RULE-010: Secret Value ≠ Identity / Role / Permission

API Key / Password / Telegram Bot Token / Exchange Secret等のSecret ValueをPrincipal / Role / Permission定義へ保存しない。

FIX-018AではSecret Value管理を実装しない。Secret / Credential GovernanceはFIX-018Bへ送る。

---

# 3. SecurityPrincipal

## SEC-OBJ-001: SecurityPrincipal

### Meaning

市場理解OSが一つのIdentityとして認識するHuman / Service / Automation / External AI等の論理主体。

`SecurityPrincipal`はCredentialでもRoleでもPermissionでもない。

### Candidate Principal Types

```text
HUMAN
SERVICE
AUTOMATION
EXTERNAL_AI
BOT_SERVICE
```

TelegramそのものをHuman Authorityとして扱わない。Telegram BotをPrincipalとして登録する場合も、そのBot PrincipalはChannel / Service権限だけを持ち、HumanのGovernance権限を継承しない。

### Candidate Fields

```yaml
principal_id:
principal_type:
display_name:
identity_provider_ref:
environment_scope_refs: []
status:
created_at:
```

### Candidate Principal Status

FIX-018Aでは複雑なPrincipal Lifecycleを作らず、最低候補だけを持つ。

```text
ACTIVE
DISABLED
```

より詳細なCredential Lifecycle / Expiry / CompromiseはFIX-018Bで扱う。

### Invariants

- PrincipalへSecret Valueを保存しない
- Principalへ万能Permissionを直接埋め込まない
- Humanだから自動管理者にしない
- Serviceだから自動内部Trustedにしない
- AIだから自動Approve Authorityにしない
- Telegram User ID等の外部IdentifierをRoleそのものとして扱わない

---

# 4. Authentication Context

## Meaning

今回のRequest / Session / Internal CallでPrincipal Identityをどのように確認したかを表すRuntime Security Context。

FIX-018Aでは独立Persistent Objectとして正式化しない。

Candidate:

```yaml
principal_ref:
authentication_method:
authenticated_at:
authentication_strength:
session_ref:
expires_at:
```

Candidate Methods:

```text
LOCAL_SESSION
TELEGRAM_ACCOUNT
SERVICE_CREDENTIAL
API_TOKEN
MFA
SIGNED_INTERNAL_CALL
```

### Invariants

- Authentication ContextはRoleを自己申告させない
- Client payload内の`role=ADMIN`等をTrusted Role Bindingとして扱わない
- Authentication成功だけでAuthorization成功とみなさない
- 重要操作のAuthentication Summary / referenceはAuditEventへ追跡可能にする

---

# 5. Role Binding

## Meaning

特定Principalが、特定Environment / Resource / Market等のScopeで、どの既存Role責任を持つかを定義するVersioned Binding。

FIX-018Aでは独立Persistent Objectへの昇格を保留し、Security / Authority Contract Candidateとする。

Candidate Structure:

```yaml
principal_ref:
role_ref:
environment_scope:
resource_scope:
market_scope:
valid_from:
valid_until:
```

Roleは`ROLE_DICTIONARY.md`を参照する。

### Formal Boundary

```text
Principal
≠ Role

Role Assignment
≠ Permission
```

同じRoleでもScopeが異なれば権限範囲は異なる。

例:

```text
Principal = HUMAN-001
Role = RISK_GOVERNANCE
Environment = PRODUCTION
Market = BTC
Resource = RISK_STATE
```

このBindingだけを理由にDatabase / Secret / Deploymentへ万能権限を与えない。

---

# 6. Permission Policy

## Meaning

PrincipalがRole Bindingを通して、どのResourceに対し、どのActionを、どのEnvironment / Scopeで許可されるかを定義するPolicy。

正式評価軸:

```text
Environment
+
Resource
+
Action
+
Scope
```

Candidate Structure:

```yaml
role_ref:
resource:
action:
environment:
scope_refs: []
decision:
conditions: []
policy_version:
```

---

# 7. Permission Decision

正式候補:

```text
ALLOW
DENY
CONDITIONAL
```

## ALLOW

明示Scope内で単独実行可能。

ただしDomain側で別のState / Data / Processing Gateが必要なら、それらを省略しない。

## DENY

当該Principal / Role / Scopeでは実行不可。

## CONDITIONAL

指定条件をすべて満たした場合のみ実行可能。

例:

```text
Authentication Strength sufficient
Required Role Binding active
Environment match
Market Scope match
Valid ApprovalDecision exists
Risk State permits
EntryThesis valid
DefenseDecision valid
```

### Invariant

`CONDITIONAL`を「だいたいALLOW」として扱わない。

```text
Condition不足
→ DENY
```

---

# 8. Action Vocabulary Candidate

FIX-013のAuthority語彙と整合させる。

```text
READ
CREATE
UPDATE
REQUEST
RECOMMEND
APPROVE
APPLY
EXECUTE
PAUSE
RECOVER
DELETE
EXPORT
ADMIN
```

重要:

```text
REQUEST
≠ RECOMMEND
≠ APPROVE
≠ APPLY
```

`WRITE`だけでAuthority差を潰さない。

---

# 9. Environment Scope Candidate

```text
RESEARCH
DEMO
PRODUCTION
OPERATIONS
```

同じActionでもEnvironmentが違えば別Permissionとして扱う。

例:

```text
EXECUTE_ORDER / DEMO
≠ EXECUTE_ORDER / PRODUCTION
```

---

# 10. Principal Permission Matrix

このMatrixはSecurity責任境界のCanonical Candidate。

凡例:

```text
ALLOW
DENY
CONDITIONAL
```

---

## 10.1 Human Principal

### Purpose

Human Governance / Emergency / Reviewの主体候補。

### ALLOW Candidate

```text
READ System Status
READ Governance / Audit views
REQUEST PAUSE
REQUEST EMERGENCY restriction
REQUEST Recovery
REQUEST Manual Override
```

### CONDITIONAL Candidate

```text
APPROVE State Transition
→ Required Role Bindingあり
→ Correct Environment / Scope
→ Authentication Strength sufficient
→ Authority Policy一致

APPROVE Recovery / Permission Expansion
→ Required Revalidation / Approval Setあり
→ Strict Pathを満たす
```

### DENY

```text
Direct DB State write
Direct RiskState write
Direct ProductionStage write
Direct Exchange Order bypassing OrderIntent
Direct Secret read by default
Bypass State Authority Flow
```

Formal:

```text
Human
≠ Direct Writer
```

---

## 10.2 Telegram / Human Interface Channel

TelegramはChannel / Adapter責任として扱う。

### ALLOW

```text
Receive authenticated command input
Map external identity to Principal Resolution input
Create RuntimeCommand / Request candidate
Return authorized READ response
```

### DENY

```text
APPROVE by channel identity alone
APPLY State directly
EXECUTE Order directly
Assign Role from message payload
Read Secret
```

Canonical Flow:

```text
Telegram Message
↓
Telegram Adapter
↓
Authentication
↓
Principal Resolution
↓
Role Binding
↓
Permission Check
↓
RuntimeCommand / Domain Request
↓
Authority Flow
```

---

## 10.3 Research Principal

Candidate ID example:

```text
PRINCIPAL-RESEARCH-001
```

### ALLOW

```text
READ RawData / Observation / Feature / MarketEvent
READ MarketContext / MarketDNA
READ Historical Knowledge within scope
CREATE ResearchCandidate
CREATE ResearchPlan
CREATE ResearchTrial
CREATE ResearchResult
CREATE Evidence / StressResult
REQUEST / RECOMMEND research lifecycle change
REQUEST / RECOMMEND Knowledge Health re-evaluation
REQUEST Production Promotion
```

### DENY

```text
APPROVE Production Promotion by default
APPLY Production Stage
APPLY Risk Recovery
CREATE live OrderIntent
EXECUTE Exchange Order
READ Production Secret
DELETE Production/Audit immutable data
```

Formal:

```text
Research
= Knowledgeを作る

Research
≠ 本番を動かす
```

---

## 10.4 Knowledge Promotion Principal

Candidate example:

```text
PRINCIPAL-KNOWLEDGE-PROMOTION-001
```

### ALLOW

```text
READ ResearchResult
READ EvidencePackage
READ Contradiction
READ KnowledgeLifecycleProfile
READ ApplicabilityProfile
RECOMMEND / APPROVE permitted Knowledge transitions
RECOMMEND / APPROVE permitted Production Promotion transitions
```

### DENY

```text
APPLY Risk State
EXECUTE Exchange Order
Modify ResearchResult evidence
Read Secret by default
```

Applyは各State MachineのApply Authorityに残す。

---

## 10.5 Risk Governance Principal

Candidate example:

```text
PRINCIPAL-RISK-001
```

### ALLOW

```text
READ Drawdown / Exposure
READ Position / Execution Health
READ Data Health / Knowledge Health / Market Novelty
REQUEST / RECOMMEND Restrictive Risk Transition
APPROVE permitted Risk Transition
```

### CONDITIONAL

```text
APPROVE Recovery / Risk Expansion
→ Strict Recovery Policy
→ Required Health confirmation
→ Required Approval Set
→ Scope一致
```

### DENY

```text
Direct Current Risk State write unless this exact Principal is separately bound as Apply Authority
Modify Hypothesis evidence
Execute Exchange Order
```

Formal:

```text
Risk Governance
= 判断責任

Risk State Controller / Apply Authority
= State書込責任
```

---

## 10.6 Production Decision Principal

Candidate example:

```text
PRINCIPAL-PRODUCTION-001
```

### ALLOW

```text
READ Approved / eligible Knowledge
READ ApplicableHypothesis inputs
READ MarketContext / MarketDNA
READ Constraint / RiskState
CREATE ApplicableHypothesisSet
CREATE TradeThesis
CREATE SignalDecision
CREATE DefenseDecision where assigned
```

### CONDITIONAL

```text
CREATE OrderIntent
→ valid TradeThesis
→ valid SignalDecision
→ valid DefenseDecision
→ valid EntryThesis
→ Current Risk permission satisfied
```

### DENY

```text
APPROVE Hypothesis Lifecycle
APPROVE own Production Promotion
APPLY Risk State
Rewrite ResearchResult
Execute Exchange order if Execution responsibility is separately assigned
Read Secret directly
```

---

## 10.7 Execution Principal

Candidate example:

```text
PRINCIPAL-EXECUTION-001
```

### ALLOW

```text
READ valid OrderIntent
READ EntryThesis
READ DefenseDecision
READ Current RiskState required for execution gate
EXECUTE authorized Exchange Order
CREATE ExecutionRecord
CREATE / feed primary execution facts for ProductionEvidence
```

### CONDITIONAL

```text
EXECUTE PRODUCTION ORDER
→ OrderIntent valid and unexpired
→ EntryThesis exists
→ DefenseDecision permits
→ Current Risk permission permits
→ Exchange / account scope matches
→ required credential reference available through FIX-018B mechanism
```

### DENY

```text
Change Trade direction
Create new Hypothesis
Approve Production Promotion
Recover RiskState
Rewrite ResearchResult
```

Formal:

```text
Production Decision
= なぜRiskを取るか

Execution
= 許可された注文をどう送るか
```

---

## 10.8 Runtime Principal

Candidate example:

```text
PRINCIPAL-RUNTIME-001
```

### ALLOW

```text
START permitted processes
PAUSE permitted processes
STOP / SAFE_SHUTDOWN
HEALTH / dependency checks
Apply Runtime State when assigned as Runtime Apply Authority
```

### DENY

```text
Approve Hypothesis
Approve Production Promotion by default
Change Market Evidence
Generate Trade direction
Read arbitrary Secret
```

Formal:

```text
Runtime
≠ Risk Governance
≠ Knowledge Promotion
```

---

## 10.9 Monitoring Principal

Candidate example:

```text
PRINCIPAL-MONITOR-001
```

### ALLOW

```text
READ Health / Latency / Source State
READ Runtime State / Resource Usage
READ Position health where required
CREATE ErrorEvent
CREATE Incident
CREATE Diagnostics
REQUEST / RECOMMEND PAUSE
REQUEST / RECOMMEND Risk Restriction
REQUEST / RECOMMEND Source Degradation
```

### DENY

```text
Direct State write
Generate ApprovalDecision by default
Execute Order
Rewrite evidence
```

---

## 10.10 Logger Principal

Candidate example:

```text
PRINCIPAL-LOGGER-001
```

### ALLOW

Custody / persistence pathに限定して:

```text
PERSIST ApprovalDecision
PERSIST StateTransitionEvent
PERSIST EntryThesis
PERSIST ProductionEvidence
PERSIST TradeResult
PERSIST AuditEvent
```

### DENY

```text
Decide RiskState
Approve Transition
Generate Trade direction
Modify persisted Immutable facts semantically
```

Formal:

```text
Logger
= Custodian

Logger
≠ Authority
```

---

## 10.11 External AI Review Principal

Candidate example:

```text
PRINCIPAL-AI-REVIEW-001
```

### ALLOW

```text
READ sanitized TradeThesis
READ sanitized MarketContext
READ sanitized Evidence / Knowledge within scope
CREATE AIReviewResult
CREATE ResearchCandidate
```

### DENY

```text
APPROVE
APPLY
EXECUTE_ORDER
READ_SECRET
DELETE
RAW_DATABASE_EXPORT
```

### CONDITIONAL

```text
READ Production-derived information
→ Sanitization Policy
→ Allowed Classification
→ Minimum necessary scope
```

Formal:

```text
AI
= Advisor / Research contributor

AI
≠ Authority
```

---

# 11. Security → Authority Integration

Canonical Flow:

```text
External Request / Internal Service Action
↓
Principal Resolution
↓
Authentication
↓
Role Binding
↓
Permission Policy Check
├─ DENY
│   ↓
│   AuditEvent / Diagnostics
│
└─ ALLOW / CONDITIONAL satisfied
    ↓
Domain Request / Action
    ↓
必要ならFIX-013 Authority Flow
    ↓
REQUEST
↓
RECOMMEND
↓
APPROVE
↓
ApprovalDecision
↓
Apply Authority
↓
StateTransitionEvent / Object Mutation / Execution
↓
AuditEvent
```

Security Permission CheckはApprovalDecisionを代替しない。

---

# 12. State Transitionとの接続

State変更では、最低概念上次を満たす。

```text
Authenticated Principal
AND valid Role Binding
AND required Permission
AND correct Environment / Scope
AND Authority Policy
AND ApprovalDecision Set when required
AND State Transition Rule
AND expected_previous_state
↓
Apply Authority
↓
StateTransitionEvent
```

Permissionを持つだけでCurrent Stateへ直接書き込まない。

---

# 13. Restrictive Fast Path

Safety Restrictive TransitionはFIX-013 / FIX-015のFast Pathを維持できる。

例:

```text
Monitoring detects hard failure
↓
REQUEST_RISK_RESTRICTION Permission確認
↓
Emergency Policy
↓
Automated Policy ApprovalDecision
↓
Risk Apply Authority
↓
EMERGENCY
```

ただし:

```text
Fast Restriction
≠ Fast Recovery
```

Recoveryは別Permission・別Approval条件を要求する。

---

# 14. Permission Matrix共通禁止事項

新規実装で次を禁止する。

```text
role == ADMIN → allow everything

is_authenticated == true → allow

Telegram command → direct state write

AI response → direct ApprovalDecision

Research service → production order

Logger → state authority

Client supplied role → trusted role

RiskState recovery using same unrestricted permission as emergency restriction

Wildcard resource=* action=* environment=* in Production by default
```

Production wildcard permissionは原則禁止候補とする。

---

# 15. Audit Boundary

重要Security操作はAudit対象候補。

例:

```text
Authentication failure
Permission denied
Role binding use for privileged action
ApprovalDecision generation
Manual Override request
Secret access（FIX-018B）
Credential rotation / revoke（FIX-018B）
Data export（FIX-018C/D）
Data deletion（FIX-018C）
Backup restore（FIX-018D）
Production order execution
```

通常の高頻度READすべてを無制限にPersistent Auditへ保存する必要はない。Audit Scope / Sampling / RetentionはSecurity / Data Contractで固定する。

---

# 16. Object追加Gate

FIX-018Aで正式追加するSecurity Objectは:

```text
SecurityPrincipal
```

候補・Contractとして留める:

```text
RoleBinding
PermissionPolicy
AuthenticationContext
```

現時点で追加しない:

```text
SecurityManager Layer
AuthorizationDecision Persistent Object
TelegramAuthority Object
AIRole Object
ResearchSecurityLayer
ProductionSecurityLayer
Credential Object
Secret Object
```

理由:

- RoleBinding / PermissionPolicyはまずVersioned Contract / Policyとして十分
- AuthenticationContextはRuntime contextとして十分
- Authorization結果は通常Processing Result / Auditで追跡可能
- Credential / SecretはFIX-018Bで独立検討する
- Securityを理由にTop-Level Layerを乱立させない

---

# 17. FIX-018A Definition of Done

```text
□ Principal ≠ Role ≠ Permissionを正式化
□ Authentication ≠ Authorizationを正式化
□ Permission ≠ ApprovalDecisionを正式化
□ Default Denyを正式化
□ ALLOW / DENY / CONDITIONALを正式化
□ Environment / Resource / Action / ScopeをPermission評価軸として正式化
□ Research Authority ≠ Production Authorityを正式化
□ Restrict Permission ≠ Recover Permissionを正式化
□ Telegram / Interface ≠ Authorityを正式化
□ AI ≠ Authorityを正式化
□ Human ≠ Direct Writerを正式化
□ Principal別Permission Matrixを定義
□ FIX-013 / FIX-015 Authority Flowへ接続
□ Secret ValueをSecurityPrincipalへ保存しない
□ Credential / Secret実装をFIX-018Bへ送る
□ 新しいSecurity巨大Layerを作らない
```

---

# 18. 後続FIXへ送る内容

## FIX-018B — Secret / Credential Governance

```text
Secret reference
Credential separation
Research credential ≠ Production credential
Exchange trade permission
Withdrawal permission prohibition candidate
Rotation
Revocation
Compromise handling
Secret Store
Runtime retrieval
```

## FIX-018C — Data Classification / Retention / Deletion

```text
PUBLIC / INTERNAL / SENSITIVE / SECRET
Retention policy
Deletion authority
DELETION_PENDING
Immutable long-term knowledge
Data export
```

## FIX-018D — Backup / Restore / External Exposure

```text
Backup encryption
Backup access
Restore authority
Restore validation
Restore ≠ Production Resume
External AI sanitization
Data minimization
```

---

# 19. 最終原則

FIX-018AのSecurity骨格:

```text
SecurityPrincipal
「誰か」
↓
Authentication Context
「本当にそのPrincipalか」
↓
Role Binding
「何の責任か」
↓
Permission Policy
「何を・どこで・どこまで許可されるか」
↓
Domain Request / Action
↓
Authority / Approval
「今回実行してよいか」
↓
Apply / Execute
「実際に操作する」
↓
StateTransitionEvent / ExecutionRecord / Object Result
↓
AuditEvent
「誰が何をしたか」
```

正式境界:

```text
Principal ≠ Role ≠ Permission
Authentication ≠ Authorization
Permission ≠ ApprovalDecision
Permission ≠ Applied Action
Research Authority ≠ Production Authority
Restrict Permission ≠ Recover Permission
Telegram / Interface ≠ Authority
AI ≠ Authority
Secret Value ≠ Identity / Role / Permission
```

市場理解OSでは、便利さのために責任境界を曖昧にせず、明示Permissionがない操作は許可しない。Productionでは特に、Research・AI・Interface・Logger・Monitoringがそれぞれの責任を越えてState / Order / Secretへ直接作用しない設計を優先する。