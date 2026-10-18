# Bitlang Preprocessed 言語仕様

> この文書は **Bitlang Preprocessed**、すなわちBitlangの完全明示された正規化後形式の仕様である。
> Bitlang sourceとは別の意味体系を持つ独立言語ではない。別リポジトリとして管理するのは処理段階・仕様責務を分離するためである。

## 1. 基本設計原則

Bitlang Preprocessed は、プリプロセッサ後のプログラムについて意味論上必要となる条件を可能な限り明示的に保持する。

Bitlang sourceでは省略・推論・糖衣構文を許可できるが、source自身もPreprocessedで使われる全正規propertyを明示的に記述できる。Bitlang Preprocessedではプリプロセッサが省略等を解決し、適用可能な最終propertyを明示した状態で出力する。

### 全条件明示の原則

- 意味論上重要な状態を暗黙の既定値に依存させない。
- プリプロセッサが推論できる情報であっても、Preprocessed 出力では明示する。
- 静的解析器および後段のコンパイラが、周辺コードや暗黙規則を推測せずに対象の性質を判断できる形を目標とする。
- 互いに反対の意味を持つ状態について、一方だけを省略形として扱わず、双方を明示的なプロパティとして表現する。

具体的な正規プロパティ集合と各軸の意味は [PROPERTIES.ja.md](PROPERTIES.ja.md) を正本とする。
借用状態の詳細は [BORROW_STATE.ja.md](BORROW_STATE.ja.md) を正本とする。

## 2. 識別子と大文字小文字

Bitlang Preprocessed では、文字列値および文字値を除き、大文字小文字を区別しない。

そのため、以下は同一の識別子として扱う。

```bitlang
MyValue
myvalue
MYVALUE
```

一方、アンダースコア (`_`) は有意な文字として扱い、区別する。

```text
MyValue == myvalue
my_value != myvalue
```

camelCase、PascalCase、snake_case などの表記スタイル自体に意味論上の制約はない。ただし大文字小文字のみが異なる名前は同一識別子であるため、同一スコープで重複定義できない。

例:

```bitlang
var int HitPoint = 100;
var int hitpoint = 200; // 同一識別子の重複定義
var int hit_point = 300; // 別識別子
```

### Unicode識別子とalias

Bitlang Explicitは日本語を含むUnicode identifierを正式に扱う。

identifierのUTF-8 validation、XID profile、NFC + Unicode default case folding、alias collision、primary symbol identityの規則は [IDENTIFIERS_AND_ALIASES.ja.md](IDENTIFIERS_AND_ALIASES.ja.md) を正本とする。

named declarationはprimary nameに加えて複数aliasを持てる。aliasは同一symbolへのlookup nameであり、別decl/runtime symbolを生成しない。

## 3. 文字列型と文字型

文字列値および文字値は大文字小文字を区別する。

```text
"A" != "a"
'A' != 'a'
```

識別子の case-insensitive 規則は、文字列および文字の内容には適用しない。

### `x` / `xx` の単位規則

Bitlangにおけるwidth suffixの単位は型familyによって変更しない。

```text
x<N>  -> N bit
xx<N> -> N byte
```

`x` を文字数、要素数、code point数など別単位へ再解釈してはならない。そうした別dimensionは別の記法を使用する。

Bitlang sourceではbyte幅を `xx<N>` で記述できるが、Bitlang Explicitでは 1 Bitlang byte = 8 bit としてcanonicalな `x<BitWidth>` へ正規化する。

例:

```text
Int2xx4
    -> Int2x32
```

したがってsource preprocessing後には:

```text
Int2xx4
Int2x32
```

は同一のcanonical type `Int2x32` になる。

`xx` はsource-level shorthandでありBitlang Explicitには残らない。

文字列の最大文字数は `xxx<N>` で表現する。

例:

```text
Strxxx3
    -> 最大3文字

Char
    -> Strxxx1
```

character countはbyte/bit幅とは独立したsemantic constraintであるため、`xxx<N>` を `x<N>` へ換算してはならない。

複数の制限は同時に表現できる。

```text
Strxxx10xx32
    -> 最大10文字
    -> 最大32byte
```

`xx<N>` のbyte幅はcanonical化時に必要に応じて `x<N*8>` へ正規化するが、`xxx<N>` のcharacter-count constraintは独立して保持する。

これはbackend storage sizeを決定する規則ではなく、semantic width / countの正規化規則である。物理storage representationはBitlang Low/backend側で別途決定する。

### 無制限 `x*` と真の0幅 `x0`

ビット幅を持つ型familyでは `x*` を**固定のビット幅上限なし**、`x0` を**0ビット**として厳密に区別する。`*` は数値0でもポインタでもなく、無制限のwidth constraintを表す記号である。演算・マクロからwidth値0が導出されても、自動的に `*` に変えてはならない。

```text
Int10x*    -> 無制限幅の任意精度整数 (符号付き、基数10)
Uint10x*   -> 無制限幅の任意精度整数 (符号なし、基数10)
Strx*      -> UTF-8符号化ビット長の上限なし
Strx0      -> 最大0ビット (空文字列のみ)
Strxx*     -> Strx* へ正規化するsource表記
Strxx0     -> Strx0 へ正規化するsource表記
Strxxx*    -> 文字数上限なし (bit上限には換算しない)
Strxxx0    -> 最大0文字 (空文字列のみ)
```

`Str` に基数はないため `Str1x*` は正式表記ではなく、`Strx*` を使用する。`Str` は制約なしの `Strx*` と同じ型であり、他の有限制約がなければ `Strxx*` や `Strxxx*` も同じsemantic typeへ正規化する。一方、別dimensionに有限幅があれば `*` はそれを消去しない。`Int` のデフォルトは `Int10x32` のままであり、`Int10x*` とは異なる型。`Bool` 等width suffixを持たない型に `x*` は付けない。

`Int/Uint` の `x*` は任意精度整数だが、有限メモリに依存するためallocation/resource failureは明示的な実行失敗として扱う。`Float<Radix>x*` は統一suffixとして予約するが、精度・丸め等の意味論が確定するまでfail-closedで拒否する。`Int/Uint/Float` の `x0` は無効な型。sourceとExplicitで制約の意味を変更してはならない。

### 制約付き文字列の型互換・警告・オーバーフロー

`Strx<N>` の `N` は「UTF-8符号化後の最大ビット数」であり、値の長さが常に `N` bitであるという意味ではない。UTF-8符号化済みの文字列のビット数は8の倍数になる。

そのため `Strx3` 型は有効であり、空文字列（0bit）だけを保持できる。このような「非空文字列を保持できない」制約付き `Str` 型を宣言した場合、コンパイラは**警告**を出すが、型宣言自体はエラーにしない。

制約はcanonicalな型identityの一部である。**異なる制約付き `Str` 型の直接代入は、値の実際の長さにかかわらずコンパイル時の型不一致エラー**とする。上限を広げる方向でも暗黙の型変換はしない。例えば `Strx8 -> Strx3` と `Strx8 -> Strx16` の直接代入はいずれもエラー。`Strxx1` は `Strx8` に正規化されるので、この2つは異なる型ではない。

異なる制約付き文字列型へ変換するには、明示的なchecked cast/変換を必要とする。変換先のbit数・byte数・文字数の上限を実際の値が超える場合は**文字列容量オーバーフローエラー**とする。違反が静的に確定する場合はコンパイルエラー、実行時にしか確定しない場合は、変換・生成・更新時のruntime checkで検出して通常のruntime error/trapにする。値の自動切り捨て・wrapは行わない。

同一の `Strx3` 型同士の通常の代入は型不一致ではない。値を生成した側が `Strx3` の容量制約を満たすことを保証する。

### 0サイズ型と0長の意味

0ビットの**意味論上のpayload**と、その値をruntimeで管理するための物理storage/descriptorは区別する。0サイズpayloadは許可できるが、それだけで「ポインタだけの型」になるわけではない。

- `Strx0` / `Strxx0` / `Strxxx0` は空文字列しか持てない有効な制約付き文字列型（空専用宣言は既定の警告対象）。文字列descriptor自体の物理サイズは0とは限らない。
- `Array<T, Length=0>` は0要素の固定長配列であり、要素アクセスは必ず範囲外。`Array<T>` のruntime長が0であることとは型契約が異なる。
- `struct Empty { }` のような0-field structは、フィールドpayloadを持たない有効な単一値の型とする。
- 0サイズ型を `N` 個含む配列には依然 `N` 個の論理要素と添字範囲がある。実メモリで隣接した別アドレスが得られるとは限らない。
- `Int/Uint/Float` の `x0` と `Boolx0` は不正な型。`Char` はちょうど1 Unicode scalar valueを要求するため、0bitに縮めて空文字列扱いにしてはならない。

ownership、borrow、lifetime、identity、release/finalizationの検証を0サイズだから省略してはならない。`Ref<T>` のnon-nullや区別が必要なobject identityを、ゼロpayloadの物理アドレスだけから推定しない。ターゲットの `sizeof` / alignment / ABI表現はBitlang Lowで別途決定する。

## 4. 正規プロパティモデル

Bitlang Preprocessed では、対象に意味を持つプロパティ軸について最終状態を明示する。

代表例には以下が含まれる。

```text
Public / Private
Protected / Unprotected
Exported / Unexported
Static / Dynamic
Instance_required / Instance_unrequired
Readable / Unreadable
Writeable / Unwriteable
Reassignable / Unreassignable
Owned / Borrowed
Copyable / Uncopyable
Movable / Unmovable
Unmoved / Moved
Auto_release / Manual_release
Releasable / Unreleasable
Unreleased / Released
Initialized / Uninitialized
Declaration_initialization / Owner_initialization / First_reach_initialization / First_use_initialization / Manual_initialization
Scope_end_finalization / Owner_end_finalization / Module_end_finalization / Program_end_finalization / Manual_finalization
nullable / unnullable
Optional / Required
Const / Unconst
```

lifetime や borrow state も明示対象である。`Static / Dynamic` は retention、`Instance_required / Instance_unrequired` は instance access requirement、initialization trigger は初期化時期、finalization trigger は終了処理時期を表す独立軸であり、互いに必要な整合性を保ちながら別々に明示する。

ここに列挙した各軸の完全な意味、独立性、状態遷移、整合性規則は [PROPERTIES.ja.md](PROPERTIES.ja.md) および [BORROW_STATE.ja.md](BORROW_STATE.ja.md) で管理する。

特に nullability は暗黙にしない。nullability が意味を持つ対象では `nullable` または `unnullable` のどちらかを必ず持ち、どちらも存在しない状態は不完全な Preprocessed として扱う。

### static保持状態の順序規則

static保持状態の初期化順序はdependency graphで決定する。

- 明示的な初期化順序制約
- initializerが利用する静的に判明した依存関係
- owner / moduleの初期化依存関係

をgraph edgeとして扱い、dependencyをdependentより先に初期化する。

graphだけで順序が決まらない場合のtie-breakは:

1. 同一declaration field内はlexical declaration order
2. 無関係なfield / module間はcanonical fully-qualified declaration name order

とする。

`First_reach_initialization` / `First_use_initialization` はlazy semanticsを維持し、trigger時に未解決dependencyを先に初期化する。`Manual_initialization` は自動初期化順から除外する。

静的に判明するinitialization cycleはerror。runtime再入でのみ発見できるcycleは、partially initialized stateを公開せずruntime errorとする。

自動finalizationの既定順序は、各retention-domain instanceにおける **実際に成功した初期化順序の逆順** とする。

explicitなfinalization order制約はこの既定順序を調整できるが、dependency / lifetime / destruction safetyと矛盾してはならない。矛盾はerrorとする。

`Thread_retention` / `Task_retention` は各domain instanceごとに独立した実初期化順・逆finalization順を保持する。

### 破棄安全性は再検証する

Bitlang Preprocessedはfully explicitであるが、生成元を信頼済みとみなして危険な破棄を許可してよいわけではない。

static analysis / compilerは、明示されたownership、borrow state、release state、lifetime、finalization trigger等を使用し、provably invalidなrelease/finalization/owner teardownをcompile errorとして拒否する。

language adapterやpreprocessorが生成した操作も手書きコードと同じ安全規則へ従う。


## 5. module scope の正規化

Bitlang Explicitでは、source側のfile/module選択を暗黙規則として再解釈しない。

Bitlang sourceの各fileはpreprocessing時に外側moduleを解決する。

- file先頭に明示moduleがある場合はそのmoduleを使用する。
- 明示moduleがない場合は、拡張子を除いたfile名からmodule名を解決する。
- namespace mountがある場合は、そのmount prefixの内側にfile moduleを置く。
- source内の `module X { ... }` は、その範囲だけを追加のnested module scopeとする。

preprocessorはfile-level module指定を、概念上file body全体をmodule scopeで囲う形へ正規化する。

例:

```bitlang
module Game;

class Player {
}
```

は後段が次のscope ownershipとして扱える形へ正規化される。

```bitlang
module Game {
    class Player {
    }
}
```

明示moduleがない `player.bit` も同様に、外側 `module player { ... }` を持つ形へ解決される。

この時点では、file名からmoduleを推測する処理、namespace mount lookup、file-headerのmodule sugarを後段へ残さない。Bitlang Explicitの後段は明示されたcanonical module ownershipだけを使用する。

物理file scopeと論理module scopeは別概念である。file単位のpreprocessing/default規則が既に解決済みであることと、module ownershipが明示されていることを混同しない。



### module visibility

canonical module scopeは `VISIBILITY.ja.md` の解決済み `scope_visibility` stateを明示する。sourceの `Public / Private` を後段へ残さない。

source `Private module` はcanonical `File_visible` へ正規化される。このfile単位の可視性について、preprocessing/name resolutionは次を保証してから後段へ渡す。

- same-file referenceだけがPrivate moduleを解決できる。
- cross-file import / qualified lookupではPrivate moduleを候補に含めない。
- import capabilityやalias生成によってPrivate moduleを外部fileへ露出させない。
- 必要な診断/source mappingのため、file-private判定が完了するまで宣言のorigin file identityを保持する。

`File_visible` module配下の宣言が `Exported` であっても、module自身がcross-file lookupへ参加しないため、外部fileからその宣言へ到達する経路にはならない。その他の標準/Custom visibilityも同じcanonical visibility engineで検証する。

### 値渡しを既定とする型transfer

Bitlang Explicitではplain `T` を常にvalue semanticsとして扱う。struct / classの種別によって暗黙のreference semanticsへ切り替えない。

```text
T       -> value
Ref<T>  -> explicit safe reference
Ptr<T>  -> explicit raw pointer
```

plain `T` parameter / assignment / returnがcopyを必要とする場合、`Copyable`でなければerror。`Uncopyable`を理由に暗黙referenceへ変更してはならない。

move可能な値のownership transferは明示されたmove semanticsとして扱い、型は依然 `T` のvalue transferである。

backendがABI上hidden pointer等を使用してもよいが、calleeからcallerの元storageへの変更が観測されるなどvalue semanticsを破壊してはならない。

完全仕様はBitlang source側 `VALUE_SEMANTICS.md` を正本とする。

## 6. Bitlang Low への名前展開

Bitlang Preprocessed のクラスに属する値は、Bitlang Low へ変換する際にクラス名を先頭へ展開する。

概念例:

```bitlang
class RRR {
    struct PPP {
        int hp;
    }

    PPP ppp;
}
```

完全に flatten する場合、概念上は次のような名前になる。

```text
rrr_ppp_hp
```

ただし Bitlang Low は struct 型を持つため、struct をどこまで flatten するかは Bitlang Low 側の実装・変換規則に依存する。

例えば struct を保持する場合は、概念上次のような形になり得る。

```text
rrr_ppp.hp
```

### 確定事項

- class に由来する所属情報は Compiled 側の名前の先頭に反映する。
- 大文字小文字の差は識別子の意味に影響しない。
- `_` は名前の構成要素として保持し、区別する。

### 未確定事項

- struct の field まで完全 flatten するか。
- struct 自体を Compiled の struct として保持する場合の最終的な命名・参照形式。

## 7. Bitlang source との境界

Bitlang source と Bitlang Preprocessed は同じBitlangの異なる正規化状態である。

Bitlang source は省略、推論、糖衣構文、プリプロセッサ関数による変換を扱える一方、**Preprocessedに存在する全てのcanonical propertyをsourceから直接明示することもできる**。

Bitlang Preprocessed はそれらの処理後に得られる完全明示形であり、source側の省略規則や書きやすさを再定義しない。

したがって違いは「何のpropertyを表現できるか」ではなく、「適用可能なpropertyを省略したまま境界を越えられるか」である。

```text
Bitlang source
    -> preprocess / normalize
    -> Bitlang Preprocessed
    -> Bitlang Lowerer
    -> Bitlang Low
```

Bitlang source 側の仕様は `tomiya7688/Bitlang`、Bitlang Low 側の仕様は `tomiya7688/Bitlang_low` をそれぞれ正本とする。

## 8. 仕様管理方針

このリポジトリを Bitlang Preprocessed の仕様・実装・テストの正本とする。

Bitlang source側の仕様とは別リポジトリで管理するが、これは同一Bitlangのsource-facing規則とfully-explicit規則の責務分離である。接続は正規化規則として明示的に定義する。

確定仕様と検討中事項を混在させず、未確定事項は未確定として明示する。
