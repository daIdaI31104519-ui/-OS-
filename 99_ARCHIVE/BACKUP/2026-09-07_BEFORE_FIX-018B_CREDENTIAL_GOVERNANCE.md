# BEFORE FIX-018B — Credential Governance

Date: 2026-09-07
Purpose: Backup manifest before FIX-018B.

## Pre-change Head

```text
2cea34bf122f1856c8c8d379474af06b55d6682c
```

## Intended Scope

FIX-018B adds a dedicated credential semantic dictionary only:

```text
01_DICTIONARY/CREDENTIAL_DICTIONARY.md
```

The following existing dictionaries are intentionally NOT modified by this change:

```text
01_DICTIONARY/SECURITY_DICTIONARY.md
01_DICTIONARY/OBJECT_DICTIONARY.md
01_DICTIONARY/ROLE_DICTIONARY.md
01_DICTIONARY/STATE_DICTIONARY.md
```

No `.py`, DB schema, DATA_CONTRACT, Secret Store implementation, API key value, token value, or production credential value is part of this change.

## Pre-change Dictionary Blobs

```text
01_DICTIONARY/SECURITY_DICTIONARY.md
e18f5ee2fc2630063ed3ad6a63c2e36e8864f695

01_DICTIONARY/OBJECT_DICTIONARY.md
ef9316fe620315abbd0ced94a4490b54b0939a91

01_DICTIONARY/ROLE_DICTIONARY.md
4f77d7e64728d8839e5fc586e1cb904418ff22ad

01_DICTIONARY/STATE_DICTIONARY.md
3979831c09bda1dcc0259dadb67a8da69f09f2ea
```

## FIX-018B Intended Semantic Decisions

```text
Secret Value
≠ Secret Reference
≠ CredentialProfile
≠ Domain Permission

Research Credential
≠ Demo Credential
≠ Production Credential

Production Decision
≠ Exchange Credential Holder

Exchange Read Credential
≠ Exchange Execution Credential

Research AI Credential
≠ Production Review AI Credential

Credential ACTIVE
≠ Trade Authorized

Credential Rotation
≠ Risk Recovery
≠ Production Resume
```

## CredentialProfile Candidate Fields

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

## Initial Credential Allocation Target

For one authenticated market data provider, one exchange, Telegram, and one AI provider:

```text
Research                  1
Production Decision       0
Execution                 2
Telegram                  1
AI                        2
--------------------------------
Base Total                6
```

Demo trading, if enabled with provider credentials, should use separate Demo Read / Demo Execution credentials rather than Production credentials.

## Lifecycle Candidate

```text
ISSUED
ACTIVE
ROTATING
EXPIRED
COMPROMISED
REVOKED
```

Forbidden recovery transitions:

```text
COMPROMISED → ACTIVE
REVOKED → ACTIVE
EXPIRED → ACTIVE
```

Use a new Credential version / Secret generation instead of reviving a compromised or retired credential.

## Authority Principle

```text
REQUEST
≠ RECOMMEND
≠ APPROVE
≠ APPLY

Credential Controller
= single-writer Apply Authority

Restrictive credential transitions
= automated safety approval may be permitted

Activation / recovery / permission expansion
= validation + strict approval
```

## Secret Safety Boundary

Real secret values must never be stored in:

```text
Git
CredentialProfile
normal design DB fields
logs
ApprovalDecision payloads
AI prompts / AI input
```

This backup records design state only and contains no secret values.