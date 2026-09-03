# 市場理解OS Backup Manifest — BEFORE FIX-001

## 0. 文書情報

- 文書種別: BACKUP / RESTORE MANIFEST
- 作成日: 2026-09-03
- 対象Branch: main
- 対象変更: FIX-001 Observation / NormalizedObservation 重複解消
- 原則: このManifestは変更前状態の復元基準であり、原則上書きしない。

---

# 1. Backup対象

## ROLE_DICTIONARY.md

```text
path:
01_DICTIONARY/ROLE_DICTIONARY.md

content_blob_sha:
7cc9b6f6b39e564a57df6bd4917e67ebb768400e

last_known_creation_commit:
37e8233047a4110191318ee148853162e8d48bed
```

## OBJECT_DICTIONARY.md

```text
path:
01_DICTIONARY/OBJECT_DICTIONARY.md

content_blob_sha:
607c6d3e3a852dc5f7b2b5a0341bd09f742d9a39

last_known_creation_commit:
07b09078907a96cf42cb7af7c379e9fe335565e8
```

## STATE_DICTIONARY.md

```text
path:
01_DICTIONARY/STATE_DICTIONARY.md

content_blob_sha:
32397160bad52c5b6115140284236bee7f963770
```

STATE_DICTIONARY.md はFIX-001では変更予定なし。比較基準としてSHAのみ記録する。

---

# 2. FIX-001の変更目的

現在は `Observation` と `NormalizedObservation` が意味・Owner・生成位置で重複している。

FIX-001では次をCanonical候補とする。

```text
Collector
→ RawData
→ Normalizer
→ Observation
→ Data Quality
→ Time Series Processor
```

`Observation` を「RawDataを標準化し、研究・比較可能にした単一観測単位」として一本化する。

`NormalizedObservation` は独立Semantic Objectとして廃止し、正規化履歴は `Observation` のField / provenanceへ保持する。

---

# 3. Rollback条件

FIX-001後に責任境界・Trace・Data Contract設計へ不整合が生じた場合、本Manifestに記録したBlob SHAを変更前基準とする。

Rollback時は変更理由・対象Commit・影響範囲を別途記録する。
