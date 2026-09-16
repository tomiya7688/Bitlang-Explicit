# Bitlang Preprocessed 言語仕様

> この文書は **Bitlang Preprocessed** の仕様であり、Bitlang 本体の仕様ではない。
> Bitlang Preprocessed は Bitlang のプリプロセッサ出力として利用される独立した言語として扱う。

## 1. 識別子と大文字小文字

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

## 2. 文字列型と文字型

文字列値および文字値は大文字小文字を区別する。

```text
"A" != "a"
'A' != 'a'
```

識別子の case-insensitive 規則は、文字列および文字の内容には適用しない。

## 3. Bitlang Compiled への名前展開

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

## 4. 仕様管理方針

このリポジトリを Bitlang Preprocessed の仕様・実装・テストの正本とする。

Bitlang 本体の仕様とは分離して管理し、両者の接続は変換規則として明示的に定義する。

今後この文書には、確定した仕様のみを追加する。検討案は確定仕様と混在させず、必要に応じて未確定事項として明示する。
