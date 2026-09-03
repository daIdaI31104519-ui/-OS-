# FIX-002 Backup Manifest

## 0. Purpose

Hypothesis Lifecycle と Production Promotion Stage の混在修正前の復元基準。

## 1. Target

```text
01_DICTIONARY/OBJECT_DICTIONARY.md
```

## 2. Restore Point

```text
content_blob_sha: e799d18f35c5e609b951fd8a74b0263e01d521a1
previous_change_commit: d4db9cb19db0a549d96058cfd8b0a67ea5ed04fc
```

## 3. Planned Change

`HypothesisPoolEntry` に混在している Hypothesis Lifecycle と Production Promotion Stage を分離する。

- Hypothesis成熟度は `hypothesis_state_ref`
- Production利用段階は `production_stage`
- Production側の許可上限は `max_production_stage`
- `DEMO_FORWARD` を Hypothesis Lifecycle status として扱わない

## 4. Rollback

問題があれば上記Blob SHAを基準に `OBJECT_DICTIONARY.md` を復元する。
