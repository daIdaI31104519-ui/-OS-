# FIX-005 Backup Manifest

## 0. Purpose

Role / Object辞書内の非正式・重複Object名を正式名称へ統一する前の復元基準。

## 1. Targets

```text
01_DICTIONARY/ROLE_DICTIONARY.md
01_DICTIONARY/OBJECT_DICTIONARY.md
```

## 2. Restore Points

### ROLE_DICTIONARY.md

```text
content_blob_sha: 658c68594557eb83a2c15267e64b65322a08cb93
previous_change_commit: 505d96cc87667ee83d254396a0e7e03048a99887
```

### OBJECT_DICTIONARY.md

```text
content_blob_sha: 89a1f53af521f8fcca0843a63361a022ea7c75cd
previous_change_commit: 56ad868d9a2add99d5fe93a87497992f9785cc3c
```

## 3. Planned Change

正式Object名を次へ統一する。

```text
FeatureResult -> Feature
ResearchTrialResult -> ResearchResult
Trial Result -> ResearchResult
Historical/OOS/Regime/Demo/Stress Validation Result -> ResearchResult + evidence_source_channel / experiment_mode
Research Test Request -> ResearchCandidate
```

旧用語は必要ならMigration / Legacy Aliasとして記録するが、新規設計・新規コード・新規DB Schemaでは正式名のみを使用する。

## 4. Rollback

問題があれば上記Blob SHAを基準に各Dictionaryを復元する。
