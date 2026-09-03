# FIX-003 Backup Manifest

## 0. Purpose

ResearchPlan Lifecycle と Freeze / Lock状態の分離修正前の復元基準。

## 1. Target

```text
01_DICTIONARY/STATE_DICTIONARY.md
```

## 2. Restore Point

```text
content_blob_sha: 32397160bad52c5b6115140284236bee7f963770
creation_commit_sha: ff78f8a7e5755ba5c41569d419acc602be771a59
```

## 3. Planned Change

`ResearchPlan` の `RUNNING` と `FROZEN` が同時成立し得る問題を解消する。

- Plan Lifecycle: DRAFT / READY / ACTIVE / COMPLETED / SUPERSEDED / CANCELLED
- Plan Lock State: EDITABLE / PRE_REGISTERED / FROZEN
- FreezeはLifecycleではなく編集可否・事前登録状態として扱う

## 4. Rollback

問題があれば上記Blob SHAを基準に `STATE_DICTIONARY.md` を復元する。
