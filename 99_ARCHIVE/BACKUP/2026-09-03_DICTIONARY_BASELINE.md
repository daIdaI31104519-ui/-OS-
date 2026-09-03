# 市場理解OS Dictionary Backup Baseline

## 0. 文書情報

- 文書種別: BACKUP / RESTORE MANIFEST
- 作成日: 2026-09-03
- 対象Branch: main
- 目的: `ROLE_DICTIONARY.md` / `OBJECT_DICTIONARY.md` / `STATE_DICTIONARY.md` を変更する前の復元基準点を固定する
- 原則: このBackup Manifest自体は原則上書きしない。次回変更前には新しい日付・識別子のBackup Manifestを追加する。

---

# 1. Backup対象

## ROLE_DICTIONARY.md

```text
path:
01_DICTIONARY/ROLE_DICTIONARY.md

creation_commit_sha:
37e8233047a4110191318ee148853162e8d48bed

content_blob_sha:
7cc9b6f6b39e564a57df6bd4917e67ebb768400e
```

## OBJECT_DICTIONARY.md

```text
path:
01_DICTIONARY/OBJECT_DICTIONARY.md

creation_commit_sha:
07b09078907a96cf42cb7af7c379e9fe335565e8

content_blob_sha:
607c6d3e3a852dc5f7b2b5a0341bd09f742d9a39
```

## STATE_DICTIONARY.md

```text
path:
01_DICTIONARY/STATE_DICTIONARY.md

creation_commit_sha:
ff78f8a7e5755ba5c41569d419acc602be771a59

content_blob_sha:
32397160bad52c5b6115140284236bee7f963770
```

---

# 2. Restore原則

Dictionary変更で問題が発生した場合、対象ファイルをこのManifestに記録された変更前Versionへ戻せることをRollback基準とする。

Rollback時は次を確認する。

```text
1. 対象File
2. 変更前Content Blob SHA
3. 変更前Commit SHA
4. 変更後Commit
5. Rollback理由
6. 影響したRole / Object / State / Contract
```

---

# 3. 今後のBackup運用

重要Dictionary / Canonical / Contractを変更する前は、原則として先にBackup Manifestを作成する。

例:

```text
99_ARCHIVE/BACKUP/
├─ 2026-09-03_DICTIONARY_BASELINE.md
├─ YYYY-MM-DD_DICTIONARY_BEFORE_CHANGE_001.md
└─ YYYY-MM-DD_ARCHITECTURE_BEFORE_CHANGE_001.md
```

同じBackup Fileを上書きせず、時点ごとの復元点を残す。

---

# 4. BackupとArchiveの違い

```text
BACKUP
= 現在の重要Fileを特定Versionへ正確に戻すための復元基準

ARCHIVE
= Deprecated / Legacy / Retired設計を歴史資料として残す領域
```

両者を混同しない。

---

# 5. このBaselineの状態

このBackupは、Role / Object / State 3辞書の初回Cross-Consistency Reviewを行う直前の状態を固定したもの。

この時点では3辞書本体へ矛盾修正をまだ適用していない。
