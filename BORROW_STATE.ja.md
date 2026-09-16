# Bitlang Preprocessed 借用状態仕様

この文書は **Bitlang Preprocessed** における借用状態の正規形を定義する。Bitlang ソース側の省略・推論・プリプロセッサ規則は `tomiya7688/Bitlang`、低レベル lowering は `tomiya7688/Bitlang_compiled` で管理する。

## 正規状態

借用状態が意味を持つ対象では、次のいずれかを明示する。

```text
Unborrowed
Shared_borrowed
Exclusive_borrowed
```

Bitlang Preprocessed では、この状態を暗黙の既定値や周辺コードの推測に依存させない。Bitlang ソースで明示されていたか、プリプロセッサが推論・変換したかに関係なく、最終的に解決された状態を出力する。

## 意味

- `Unborrowed`: 現在、通常アクセスを制限する有効な借用がない。
- `Shared_borrowed`: 1個以上の共有借用が有効である。共有可能な範囲では複数存在できるが、借用を無効化する変更・解放・move は制限される。
- `Exclusive_borrowed`: 排他的借用が有効である。競合する別借用や別アクセス経路からの競合操作を許可しない。

借用状態は `Owned / Borrowed` とは別軸である。`Owned / Borrowed` は所有権責任、借用状態は現在の借用状況を表す。

## 整合性

次のような状態は静的解析対象となる。

- `Shared_borrowed` または `Exclusive_borrowed` の対象を、借用を維持したまま解放・破棄・move する。
- `Exclusive_borrowed` 中に競合する参照または借用を生成する。
- 借用先が借用元より長い lifetime を持つ。
- プリプロセッサが制約を弱める状態変更を行った結果、実際の借用関係とプロパティが矛盾する。

危険性のみが示され、無効性を断定できない場合は warning としてよい。仕様違反が証明できる場合は error とする。

## プリプロセッサによる強制変更

プリプロセッサは最終状態を明示的に変更できる。

例:

```text
Exclusive_borrowed -> Unborrowed
```

ただしこれは通常の推論ではなく、意味論を変更し得る明示的 override である。最終的な Bitlang Preprocessed には override 後の状態だけを正規状態として出力する。

## Compiled への境界

`Unborrowed / Shared_borrowed / Exclusive_borrowed` は Bitlang Preprocessed の静的意味論を表すプロパティである。Bitlang Compiled に同名プロパティをそのまま保持することは要求しない。Compiled 生成前に借用整合性を検証し、その結果を低レベルの lifetime、alias、access、release 制約へ反映する。
