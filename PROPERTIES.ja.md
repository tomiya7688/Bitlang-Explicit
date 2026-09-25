# Bitlang Preprocessed プロパティ仕様

この文書は **Bitlang Preprocessed に出力される正規プロパティ集合**を定義する。

Bitlang source側でも、この文書で定義する全canonical propertyを適用可能な対象へ明示的に記述できる。sourceではさらに一部propertyの省略や、プリプロセッサ規則による補完・変換が許される。一方Bitlang Preprocessedでは、意味論上必要な状態を後段が推測し直さなくてよいように、適用可能なproperty軸を全て明示する。

## 1. 全条件明示

あるプロパティ軸が対象に適用される場合、その軸の最終状態を Preprocessed に明示する。

- source で明示されていたかどうかは問わない。
- プリプロセッサが推論した結果も明示する。
- プリプロセッサが override した場合は override 後の状態を明示する。
- 後段のコンパイラが「書かれていないので既定値」と解釈することを要求しない。
- 相反する状態を持つ軸は、一方だけを省略形にせず、双方を正規プロパティとして扱う。
- その軸自体が対象に適用されない場合まで、無意味なプロパティを強制するものではない。

### sourceとのproperty vocabulary共有

この文書にある正規propertyは、Bitlang sourceからも全て明示指定可能である。

「Preprocessedで必須」であることは「sourceでは書けない」という意味ではない。差は次の通り。

```text
Bitlang source
    -> 明示してもよい
    -> 省略してpreprocessorに解決させてもよい

Bitlang Preprocessed
    -> 適用可能な最終状態を必ず明示する
```

したがって、全propertyを明示したBitlang sourceは、source-only構文がなければPreprocessedと非常に近い形になり得る。

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

### retention domain

```text
Process_retention
Thread_retention
Task_retention
```

retention domain は、保持状態をどの実行domainで共有するかを表す。

- `Process_retention`: process / program 全体で1つの保持状態を共有する。
- `Thread_retention`: threadごとに独立した保持状態を持つ。
- `Task_retention`: task / coroutine相当の実行単位ごとに独立した保持状態を持つ。

`Static / Dynamic` および lifetime とは独立した正規property軸である。

正規適用対象は variable / field / function とする。functionではfunction-associated stateの共有domainを表す。

Bitlang sourceで省略された場合の既定値は:

```text
Process_retention
```

Bitlang Explicitでは適用可能な対象について最終状態を必ず明示する。

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

### functionに対するretention

functionに対する `Static / Dynamic` は、実行コード自体の寿命ではなく **function-associated state** の保持方式を表す。

function-associated stateには、closure environment、capture storage、first-class function object state、その他function representationが所有する状態を含める。

- `Static`: 適用可能なfunction-associated stateをstatic retentionする。
- `Dynamic`: 適用可能なfunction-associated stateは通常のowner / lifetimeに従う。

状態を持たない通常のnamed functionではruntime storage上の差が観測不能な場合があるが、propertyはsemantic contractとして明示する。

`Instance_required / Instance_unrequired` とは独立であり、function-associated stateのinitialization / finalization時期も別property/ruleで決定する。

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

## 12. initialization trigger

初期化済みかどうかの状態とは別に、自動初期化をいつ行うかを正規propertyとして明示する。

```text
Declaration_initialization
Owner_initialization
First_reach_initialization
First_use_initialization
Manual_initialization
```

- `Declaration_initialization`: 各storage instanceの通常の宣言初期化地点で初期化する。
- `Owner_initialization`: 所有object / type / module等のowner初期化時に初期化する。
- `First_reach_initialization`: そのstorage instanceについて、実行が宣言へ最初に到達した時に1回だけ初期化する。
- `First_use_initialization`: そのstorage instanceを最初に有効利用する時に1回だけ初期化する。
- `Manual_initialization`: 自動初期化を行わず、利用前に明示的な初期化操作を要求する。

この軸の正規適用対象は variable / field とする。

`Static / Dynamic` とは独立しており、`Static` だから特定の初期化時期になるとは決めない。異なるsource languageの意味論は、変換・preprocessing時に適切なinitialization triggerへ解決する。

`Initialized / Uninitialized` は現在状態、initialization triggerは初期化方針・時期を表すため、両者も別軸である。

## 13. finalization trigger

破棄・destructor/finalizer等の終了処理をいつ実行するかを、lifetime・retention・release policyとは独立した正規propertyとして明示する。

```text
Scope_end_finalization
Owner_end_finalization
Module_end_finalization
Program_end_finalization
Manual_finalization
```

- `Scope_end_finalization`: 所有するlexical/function scope終了時に自動finalizationする。
- `Owner_end_finalization`: 所有object / type / storage owner終了時に自動finalizationする。
- `Module_end_finalization`: moduleの終了・unload時に自動finalizationする。
- `Program_end_finalization`: program/process終了処理時に自動finalizationする。
- `Manual_finalization`: 自動finalizationを行わず、必要な場合は明示的finalizationを要求する。

正規の適用対象は variable / field / parameter とする。

finalization triggerは `Auto_release / Manual_release` と別軸である。finalizer/destructorは任意の終了処理を行い得る一方、release policyはrelease/freeを自動生成するかを表す。同じ対象で両軸を独立して保持する。

Bitlang sourceで省略された場合の既定値はresolved lifetimeから決定する。

```text
Local_lifetime    -> Scope_end_finalization
Function_lifetime -> Scope_end_finalization
Object_lifetime   -> Owner_end_finalization
Module_lifetime   -> Module_end_finalization
Static_lifetime   -> Program_end_finalization
```

`Manual_finalization` は暗黙既定値にせず、source / language adapter / project rule / preprocessing ruleから明示的に選択する。

## 14. nullability

```text
nullable
unnullable
```

nullability が適用される対象では必ずどちらかを明示する。

`unnullable` は省略時既定値ではなく、`nullable` と対になる正規プロパティである。

## 15. optionality

```text
Optional
Required
```

optionality が宣言上の意味を持つ対象では、presence が任意か必須かを明示する。

Bitlang source 側の `Optional<T>` 等の記法と、Preprocessed の正規プロパティ表現の対応は source -> preprocessed 変換規則で管理する。

## 16. const

```text
Const
Unconst
```

`Const` は単に `Unreassignable` または `Unwriteable` であることとは別で、値そのものを強く固定する意味を持つ。

全条件明示の原則により、const 性が意味を持つ対象では `Const / Unconst` のどちらかを明示する。

## 17. プロパティ間の直交性

可能な限り各プロパティ軸を独立して保持する。

例:

```text
Private Unprotected Unexported
Dynamic Instance_required
Readable Writeable Reassignable
Owned Unborrowed
Copyable Movable Unmoved
Auto_release Releasable Unreleased
Local_lifetime Initialized Declaration_initialization Scope_end_finalization
unnullable Required Unconst
```

ただし、独立した軸であっても全組み合わせが合法とは限らない。

例として、実際に borrow が有効な対象を `Unborrowed` とする、資源を持たない借用経路を不正に `Releasable` とする、`Released` の資源を通常アクセス可能な生資源として扱う、などの矛盾は静的解析で拒否または診断する。

## 18. 破棄安全性

release / destruction / destructor / finalizer / owner teardown は、生成元に関係なく同じ安全規則へ従う。

次のような破棄が静的に不正と証明できる場合は **error** とし、warningのまま通してはならない。

- `Released` 済み資源の再release/finalization;
- live borrow / reference をdanglingにするowner破棄;
- `Unreleasable` な経路からのrelease;
- ownership移譲なしの非owner経路からの破棄;
- lifetime / finalization trigger と矛盾する時点での破棄;
- 同一資源を複数経路から二重破棄できるcontrol flow;
- language conversion / preprocessing が上記状態を生成する変換。

source code、language adapter、macro、preprocessor function、自動cleanup、compiler loweringのどれが生成した操作であっても例外にしない。

Bitlang Preprocessedへ到達した時点でpropertyは完全明示されるが、それは「安全性検査済みだからcompilerが無条件で信頼してよい」という意味ではない。compiler/static analysisは明示されたpropertyとcontrol flowを用いて破棄安全性を検証し、不正を証明した場合はcompile errorとする。

危険性が疑われるだけで不正を証明できない場合はwarningとしてよい。ただし、その操作自体の仕様が安全性のproofを要求する場合は、proof不能を理由にerrorとすることができる。

## 19. Bitlang source との境界

Bitlang source 側の責務は次の通り。

- このPreprocessed property modelに存在する全propertyを、適用可能な対象へ利用者が直接明示できること。
- 記述を省略できる箇所では、プリプロセッサが不足情報を解決できること。
- プリプロセッサ関数がプロパティを検査・追加・変更できること。
- 危険な override に対して warning / error を出せること。

Bitlang Preprocessed 側の責務は、**それらの処理が終わった最終状態を曖昧さなく保持すること**である。

Bitlang source の書きやすさ・省略規則そのものは `tomiya7688/Bitlang` 側で管理する。
