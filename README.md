# GitHub Merge Method Test

`main -> stg -> prod` の昇格フローで、GitHub PR の `Merge commit` と `Rebase and merge` の違いを確認するためのリポジトリです。

## 検証したいこと

- `stg` は `main` を fast-forward で追従する
- `prod-merge` は `stg` を `Merge commit` で取り込む
- `prod-rebase` は `stg` を `Rebase and merge` で取り込む
- 同じ `prod-merge` / `prod-rebase` ブランチに複数回積み上げたとき、履歴がどう変わるかを見る

## ブランチ構成

- `main`: 開発ブランチ
- `stg`: `main` を fast-forward で追従するステージングブランチ
- `prod-merge`: `stg` を `Merge commit` で取り込む本番想定ブランチ
- `prod-rebase`: `stg` を `Rebase and merge` で取り込む本番想定ブランチ

## 確認コマンド

```bash
git log --graph --decorate --oneline --all
```

## 検証手順

1. `main` に round 1 の変更を積む
2. `stg` を `main` に fast-forward する
3. `stg -> prod-merge` を `Merge commit` でマージする
4. `stg -> prod-rebase` を `Rebase and merge` でマージする
5. `main` に round 2 の変更を積む
6. `stg` を再度 `main` に fast-forward する
7. 同じ `prod-merge` と `prod-rebase` にもう一度 `stg` をマージする

## 観察ポイント

- `prod-merge` は merge commit が積み上がる
- `prod-rebase` は `stg` と同じ変更でも別のコミット ID が積み上がる
- 2 回目の昇格で、`prod-rebase` 側が `stg` と履歴共有し続けられるかを確認する
