# FIX-004 Backup Manifest

## 0. Purpose

Edge Lifecycle と Knowledge Aging の責任重複修正前の復元基準。

## 1. Target

```text
01_DICTIONARY/STATE_DICTIONARY.md
```

## 2. Restore Point

```text
content_blob_sha: c73db3c0d08a760e5e4f01312e98cae77564ada3
previous_change_commit: b3e12e05ae41c45e6d10fbbb08b2c3dc8aa33c35
```

## 3. Planned Change

- `Edge Lifecycle` の対象を `Edge` のみに限定
- `FeatureKnowledge / FormulaKnowledge` の鮮度・劣化は `Knowledge Aging State` で管理
- Edge自身については「利用成熟度」と「Knowledge鮮度」を別State Machineとして併存可能にする

## 4. Rollback

問題があれば上記Blob SHAを基準に `STATE_DICTIONARY.md` を復元する。
