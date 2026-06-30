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

## 検証結果

実際に GitHub PR で次の順にマージしました。

| PR | 向き | マージ方法 | 結果 |
| --- | --- | --- | --- |
| #1 | `stg -> prod-merge` | `Merge commit` | 成功 |
| #2 | `stg -> prod-rebase` | `Rebase and merge` | 成功 |
| #3 | `stg -> prod-merge` | `Merge commit` | 成功 |
| #4 | `stg -> prod-rebase` | `Rebase and merge` | conflict |

最終的な履歴は次の通りです。

```text
*   a4ba7c6 (prod-merge) Merge pull request #3 from aqua-ix/stg
|\  
| * 3aea223 (stg, main) main round2 part 2
| * e63cae0             main round2 part 1
* | 4ec0505             Merge pull request #1 from aqua-ix/stg
|\| 
| * c129f7c             main round1 part 2
| * ffb6a96             main round1 part 1
|/  
| * 2f03cc3 (prod-rebase) main round1 part 2
| * bcc4bfd               main round1 part 1
|/  
* f67f625 initial baseline
```

`prod-merge` は round 1 と round 2 の両方で `stg` からの PR を取り込めました。merge commit によって `stg` 側の履歴とのつながりが残るため、2 回目の昇格でも GitHub は差分を解釈できます。

一方、`prod-rebase` は round 1 の `Rebase and merge` で `stg` のコミット ID が作り直されました。そのため round 2 で同じ `stg` から再度 PR を作ると、GitHub 上では `mergeable=CONFLICTING`, `mergeStateStatus=DIRTY` になりました。

ローカルでも `prod-rebase` と `stg` の merge-base は初回コミットまで戻り、同じ round 1 の変更が別コミットとして存在する状態になります。

```text
base
main-r1-1
main-r1-2
<<<<<<< .our
=======
main-r2-1
main-r2-2
>>>>>>> .their
```

## 結論

`main -> stg -> prod` のように同じ昇格元ブランチから継続的に変更を積み上げる運用では、`prod` 側に `Rebase and merge` を使うと履歴共有が切れ、2 回目以降の昇格で conflict しやすくなります。

`stg` と `prod` の履歴を分離させないことを重視するなら、`stg -> prod` は `Merge commit` で運用する方が適切です。
