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

## 3. 文字列型と文字型

文字列値および文字値は大文字小文字を区別する。

```text
"A" != "a"
'A' != 'a'
```

識別子の case-insensitive 規則は、文字列および文字の内容には適用しない。

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

## 5. Bitlang Compiled への名前展開

Bitlang Preprocessed のクラスに属する値は、Bitlang Compiled へ変換する際にクラス名を先頭へ展開する。

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

ただし Bitlang Compiled は struct 型を持つため、struct をどこまで flatten するかは Bitlang Compiled 側の実装・変換規則に依存する。

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

## 6. Bitlang source との境界

Bitlang source と Bitlang Preprocessed は同じBitlangの異なる正規化状態である。

Bitlang source は省略、推論、糖衣構文、プリプロセッサ関数による変換を扱える一方、**Preprocessedに存在する全てのcanonical propertyをsourceから直接明示することもできる**。

Bitlang Preprocessed はそれらの処理後に得られる完全明示形であり、source側の省略規則や書きやすさを再定義しない。

したがって違いは「何のpropertyを表現できるか」ではなく、「適用可能なpropertyを省略したまま境界を越えられるか」である。

```text
Bitlang source
    -> preprocess / normalize
    -> Bitlang Preprocessed
    -> static analysis / compile
    -> Bitlang Compiled
```

Bitlang source 側の仕様は `tomiya7688/Bitlang`、Bitlang Compiled 側の仕様は `tomiya7688/Bitlang_compiled` をそれぞれ正本とする。

## 7. 仕様管理方針

このリポジトリを Bitlang Preprocessed の仕様・実装・テストの正本とする。

Bitlang source側の仕様とは別リポジトリで管理するが、これは同一Bitlangのsource-facing規則とfully-explicit規則の責務分離である。接続は正規化規則として明示的に定義する。

確定仕様と検討中事項を混在させず、未確定事項は未確定として明示する。
