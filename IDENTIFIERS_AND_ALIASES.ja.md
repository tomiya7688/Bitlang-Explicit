# Bitlang Explicit Unicode Identifier / Alias 仕様

Bitlang ExplicitはUnicode識別子と1 symbolに対する複数のlookup nameを正規表現する。

## 1. Unicode identifier

source encodingはUTF-8。invalid UTF-8はerror。

通常identifierはUnicode UAX #31のXID_Start / XID_Continueを基準とし、`_` をstart/continueとして許可する。

日本語の漢字・ひらがな・カタカナをvariable/function/type/module/field/parameter等のidentifierとして使用できる。

language versionはidentifier classification/canonicalizationに使用するUnicode data versionを固定する。host language/libraryのUnicode versionへ暗黙依存しない。

## 2. Canonicalization

比較用identifierは:

```text
Unicode NFC
-> Unicode default case folding
-> canonical identifier
```

で解決する。

`_` は有意。string/character literal内容へidentifier canonicalizationを適用しない。

canonical-equivalentな綴りは同一scopeで同一identifierとして衝突する。

## 3. Named declaration aliases

named declarationは:

```text
primary_name
aliases[]
symbol_identity
```

を持てる。

最低限の対象:

```text
module
variable
field
parameter
function
type
```

例:

```text
primary_name = 実行
aliases = [execute, run]
```

`実行` / `execute` / `run` はすべて同一symbolへresolveする。

aliasはwrapper/copy/別runtime symbolを生成しない。

## 4. Collision

aliasもprimary nameと同じcanonicalizationを受ける。

同一resolution scopeで:

- alias vs 他symbol primary name collision -> error
- alias vs 他symbol alias collision -> error
- alias vs own primary canonical equality -> error
- same-symbol duplicate canonical alias -> error

diagnosticでは可能な限り元spellingを保持する。

## 5. Identity

primary canonical fully-qualified nameを宣言のstable semantic/linkage identityとする。

aliasesは同じidentityへ到達するlookup nameであり、backend symbolを複製しない。

## 6. Restriction inheritance

alias経由でもvisibility/export/import capability/call-site policy/caller restriction/ownership/accessその他の制約を完全に共有する。

aliasによるrestriction bypassは禁止。

## 7. Name-based call-site policy

`caller_function_name_*` predicateはdefaultでcaller declarationの**primary canonical name**を判定する。

call siteで使用されたaliasやcaller自身が持つaliasによって判定結果を変更しない。

alias-aware policyが必要なら将来明示的な別predicateを定義する。

## 8. Lowering boundary

Explicitではcross-unit/interface name resolutionに必要なcanonical alias setを保持できる。

すべての参照がsymbol identityへresolve済みになった後は、reflection/debug metadata等の別要求がなければBitlang Low前にalias metadataを消去できる。

## 3. Source/Explicit alias構文

Bitlang sourceではdeclaration prefix metadataとしてaliasを指定する。

```bitlang
Alias(execute, run) function 実行(...)
別名(execute, run) 関数 実行(...)
```

`Alias` / `別名` は同じlanguage token。日本語source spellingはpreprocessingで英語canonical `Alias` へ正規化する。

`Alias(...)` はproperty axisではない。property default/precedenceへ参加せず、named declarationのlookup-name metadataとして扱う。

canonical Explicitでは必要なalias metadataを:

```text
Alias(alias1, alias2, ...) <declaration>
```

の英語canonical形で保持する。

alias要素はstringではなくordinary identifier。qualified pathは指定しない。declarationに書かれたnameがprimary nameのままであり、alias追加でprimary identityは変化しない。

Explicit canonical formでは1 declarationにつきAlias clauseは最大1個。aliasなしなら省略する。
