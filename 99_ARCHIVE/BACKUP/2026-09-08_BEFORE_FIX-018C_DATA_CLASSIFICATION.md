# 2026-09-08 BEFORE FIX-018C DATA CLASSIFICATION

## Purpose

FIX-018C Data Classification semantic design を追加する直前の参照点を固定する。

このBackupは既存Dictionary本文の複製ではなく、変更前Head / Blob SHA / 予定変更範囲を固定するManifestである。

## Pre-change Head

```text
551ded977438e33cfd431508e774211611b21c8a
```

Commit:

```text
fix: formalize FIX-018B credential governance
```

## Pre-change Dictionary Blobs

```text
01_DICTIONARY/OBJECT_DICTIONARY.md
ef9316fe620315abbd0ced94a4490b54b0939a91

01_DICTIONARY/ROLE_DICTIONARY.md
4f77d7e64728d8839e5fc586e1cb904418ff22ad

01_DICTIONARY/STATE_DICTIONARY.md
3979831c09bda1dcc0259dadb67a8da69f09f2ea

01_DICTIONARY/SECURITY_DICTIONARY.md
e18f5ee2fc2630063ed3ad6a63c2e36e8864f695

01_DICTIONARY/CREDENTIAL_DICTIONARY.md
ff730b3dcbb1172e4a551db18362728a0265a7cb
```

## Planned FIX-018C Scope

新規追加のみ:

```text
01_DICTIONARY/DATA_CLASSIFICATION_DICTIONARY.md
```

今回固定する内容:

- PUBLIC / INTERNAL / SENSITIVE / SECRET の正式順序
- Classification Strategy
  - FIXED
  - SOURCE_DERIVED
  - INHERIT_MAX
  - TARGET_INHERIT
  - CONTEXT_ESCALATE
  - FIXED_MINIMUM
- Base Classification と Effective Classification の分離
- Source / Reference / Field / Runtime Context / Escalationを用いた判定
- More Restrictive / Less Restrictive の非対称ルール
- Unexpected Secret Material を通常昇格ではなくSecurity Incidentとして扱うルール
- Sanitization = Derived View Generation
- Declassification Governance
- 現行Object FamilyのDefault Classification
- 主要ObjectのClassification Matrix
- Production境界: HypothesisPoolEntryまでは原則INTERNAL、ApplicableHypothesisSet以降は原則SENSITIVE
- Classificationを Access / Export / External AI / Backup / Retention / Deletion / Audit のPolicy入力にする原則

今回まだ固定しない:

- Retention Classの正式値
- 保存年数
- Deletion Authority詳細
- Physical Delete手順
- Backup/Restore詳細
- DB Column / Index / Python implementation

## Safety

- 既存Object / Role / State / Security / Credential Dictionaryは今回変更しない。
- Secret ValueはGitへ保存しない。
- ClassificationをKnowledge Health / Storage Lifecycle / Production Stage / RiskStateと混同しない。
