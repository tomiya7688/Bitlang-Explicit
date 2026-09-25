# Bitlang Preprocessed プロパティ仕様

この文書は **Bitlang Preprocessed に出力される正規プロパティ集合**を定義する。

Bitlang source 側では、利用者が一部のプロパティを省略したり、プリプロセッサ規則によって補完・変換したりできる。一方 Bitlang Preprocessed では、意味論上必要な状態を後段が推測し直さなくてよいように、適用可能なプロパティ軸を明示する。

## 1. 全条件明示

あるプロパティ軸が対象に適用される場合、その軸の最終状態を Preprocessed に明示する。

- source で明示されていたかどうかは問わない。
- プリプロセッサが推論した結果も明示する。
- プリプロセッサが override した場合は override 後の状態を明示する。
- 後段のコンパイラが「書かれていないので既定値」と解釈することを要求しない。
- 相反する状態を持つ軸は、一方だけを省略形にせず、双方を正規プロパティとして扱う。
- その軸自体が対象に適用されない場合まで、無意味なプロパティを強制するものではない。

## 2. 可視性

### スコープ可視性

```text
Public
Private
```

### 継承可視性

```text
Protected
Unprotected
```

### module export

```text
Exported
Unexported
```

これらは独立した軸である。

## 3. 静的保持と instance access

静的保持と、instance が必要かどうかは別軸として明示する。

### retention

```text
Static
Dynamic
```

- `Static`: 対象を静的に保持する。
- `Dynamic`: 対象を静的には保持せず、適用可能な通常の非staticな lifetime / storage 関係に従う。

ここでの `Dynamic` は dynamic typing、dynamic dispatch、可変性を意味しない。`Static` の対となる retention property である。

### instance access

```text
Instance_required
Instance_unrequired
```

- `Instance_required`: 適用可能な型memberへアクセスするために、その型のinstanceを必要とする。
- `Instance_unrequired`: instanceを生成・保持していなくても、そのmemberへアクセスできる。

retention と instance access は独立しているため、意味を持つ対象では次の組み合わせを個別に表現できる。

```text
Static  + Instance_required
Static  + Instance_unrequired
Dynamic + Instance_required
Dynamic + Instance_unrequired
```

ただし、宣言種別上意味を持たない組み合わせまで合法になるわけではない。

正規の適用対象は次の通り。

- `Static / Dynamic`: variable / field / function
- `Instance_required / Instance_unrequired`: field / function

Bitlang source の `Direct` は `Instance_unrequired` へ展開されるsource-only shorthandであり、Bitlang Preprocessedでは `Direct` を残さない。

また `Static / Dynamic` は lifetime 軸そのものではない。`Static_lifetime` 等とは別プロパティとして保持し、最終状態の整合性を検証する。

## 4. 読み書き・再代入

```text
Readable
Unreadable

Writeable
Unwriteable

Reassignable
Unreassignable
```

`Readable / Unreadable` は読み取り可否、`Writeable / Unwriteable` は値または到達可能な変更可能状態への書き込み可否、`Reassignable / Unreassignable` は宣言自体を別の値・bindingへ差し替えられるかを表す。

これらは独立した軸であり、単一の `mutable / immutable` だけでまとめない。

## 5. 所有権

```text
Owned
Borrowed
```

`Owned` は対象資源の lifetime / release 責任を所有することを表す。

`Borrowed` は対象資源を所有せず、実際の owner が有効である範囲でのみ使用できることを表す。

所有権は pointer/reference 型や読み書き権限とは別軸である。

## 6. 借用状態

借用状態の正規仕様は [BORROW_STATE.ja.md](BORROW_STATE.ja.md) を正本とする。

```text
Unborrowed
Shared_borrowed
Exclusive_borrowed
```

`Owned / Borrowed` と借用状態は別軸である。

## 7. copy / move capability

```text
Copyable
Uncopyable

Movable
Unmovable
```

copy可能性とmove可能性は独立した軸であり、ownershipとも独立して表現する。

## 8. move state

```text
Unmoved
Moved
```

通常の move は source 側を `Unmoved -> Moved` に遷移させる。

`Moved` の対象は、正当な再初期化または明示的な preprocessing override が行われるまで、通常の読み取り・再move・release対象として扱えない。

プリプロセッサは明示的に `Moved -> Unmoved` 等へ変更できるが、実際の資源状態と矛盾する場合は warning または error の対象となる。

## 9. release policy / capability / state

release は少なくとも次の3軸に分離する。

### release policy

```text
Auto_release
Manual_release
```

### release capability

```text
Releasable
Unreleasable
```

### release state

```text
Unreleased
Released
```

`Auto_release / Manual_release` は解放方針、`Releasable / Unreleasable` はその宣言またはアクセス経路から解放できるか、`Unreleased / Released` は現在の実状態を表す。

`Released` 後の通常アクセス、新規参照生成、二重解放は無効である。正当な再確保・再初期化が行われた場合は `Released -> Unreleased` へ遷移できる。

## 10. lifetime

現在定義済みの lifetime property は次の通り。

```text
Local_lifetime
Function_lifetime
Object_lifetime
Module_lifetime
Static_lifetime
```

lifetime は ownership、access capability、pointer/reference 型とは別軸である。

借用先や依存値が、その参照元・owner より長い lifetime を持つ状態は静的解析対象となる。

## 11. initialization state

```text
Initialized
Uninitialized
```

`Uninitialized` の対象を値として読むことはできない。正当な初期化によって `Initialized` へ遷移する。

## 12. nullability

```text
nullable
unnullable
```

nullability が適用される対象では必ずどちらかを明示する。

`unnullable` は省略時既定値ではなく、`nullable` と対になる正規プロパティである。

## 13. optionality

```text
Optional
Required
```

optionality が宣言上の意味を持つ対象では、presence が任意か必須かを明示する。

Bitlang source 側の `Optional<T>` 等の記法と、Preprocessed の正規プロパティ表現の対応は source -> preprocessed 変換規則で管理する。

## 14. const

```text
Const
Unconst
```

`Const` は単に `Unreassignable` または `Unwriteable` であることとは別で、値そのものを強く固定する意味を持つ。

全条件明示の原則により、const 性が意味を持つ対象では `Const / Unconst` のどちらかを明示する。

## 15. プロパティ間の直交性

可能な限り各プロパティ軸を独立して保持する。

例:

```text
Private Unprotected Unexported
Dynamic Instance_required
Readable Writeable Reassignable
Owned Unborrowed
Copyable Movable Unmoved
Auto_release Releasable Unreleased
Local_lifetime Initialized
unnullable Required Unconst
```

ただし、独立した軸であっても全組み合わせが合法とは限らない。

例として、実際に borrow が有効な対象を `Unborrowed` とする、資源を持たない借用経路を不正に `Releasable` とする、`Released` の資源を通常アクセス可能な生資源として扱う、などの矛盾は静的解析で拒否または診断する。

## 16. Bitlang source との境界

Bitlang source 側の責務は次の通り。

- 利用者がプロパティを明示できること。
- 記述を省略できる箇所では、プリプロセッサが不足情報を解決できること。
- プリプロセッサ関数がプロパティを検査・追加・変更できること。
- 危険な override に対して warning / error を出せること。

Bitlang Preprocessed 側の責務は、**それらの処理が終わった最終状態を曖昧さなく保持すること**である。

Bitlang source の書きやすさ・省略規則そのものは `tomiya7688/Bitlang` 側で管理する。
