# BEFORE FIX-018A — Security Identity / Authorization Backbone

Date: 2026-09-07
Purpose: Backup manifest before FIX-018A.

## Scope

FIX-018A fixes the semantic backbone for:

```text
Principal
Authentication
Role Binding
Permission Policy
ALLOW / DENY / CONDITIONAL
Environment / Resource / Action / Scope
Security → Authority → Apply / Execute → Audit
```

FIX-018A does not implement credentials, secrets, DB IAM, Python authorization middleware, encryption, or exchange API key storage.

## Pre-change Canonical dictionary blobs

```text
01_DICTIONARY/OBJECT_DICTIONARY.md
ef9316fe620315abbd0ced94a4490b54b0939a91

01_DICTIONARY/ROLE_DICTIONARY.md
4f77d7e64728d8839e5fc586e1cb904418ff22ad

01_DICTIONARY/STATE_DICTIONARY.md
3979831c09bda1dcc0259dadb67a8da69f09f2ea
```

## Intended write strategy

To avoid replacing the very large existing dictionaries for a new cross-cutting security domain, FIX-018A creates:

```text
01_DICTIONARY/SECURITY_DICTIONARY.md
```

The existing Object / Role / State dictionaries are intentionally not rewritten in FIX-018A. Cross references can be added during the later Cross Check after FIX-019.

## Core semantics to preserve

```text
Principal ≠ Role ≠ Permission
Authentication ≠ Authorization
Permission ≠ ApprovalDecision
Permission ≠ Applied Action
Telegram / UI Channel ≠ Principal Authority
AI ≠ Approve Authority
Research Authority ≠ Production Authority
Restrict Permission ≠ Recover Permission
Default Deny + Explicit Allow
Secret Value ≠ Identity / Role / Permission
```

## Canonical relationship target

```text
External Request / Internal Service Action
↓
Principal Resolution
↓
Authentication
↓
Role Binding
↓
Permission Check
├─ DENY → Audit / Diagnostics
└─ ALLOW / CONDITIONAL satisfied
    ↓
Domain Request
    ↓
Authority Flow when required
    ↓
ApprovalDecision
    ↓
Apply Authority / Execution Authority
    ↓
StateTransitionEvent / ExecutionRecord / Object Result
    ↓
AuditEvent
```

Historical state / approval / role semantics from FIX-013–017 must remain unchanged.