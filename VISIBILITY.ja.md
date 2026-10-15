# Bitlang Explicit 可視性モデル

Status: current canonical design

Bitlang Explicitでは、source languageの `public/private` 等の綴りではなく、解決済みの可視境界を明示する。

## 1. scope_visibility 正規状態

```text
Owner_visible
File_visible
Module_visible
Module_tree_visible
Project_visible
Global_visible
Custom_scope_visibility
```

### Owner_visible

宣言の通常owner scope内だけから参照可能。

class/type memberのprivate相当などに使用する。

### File_visible

同じ物理source file由来の宣言からだけ参照可能。

Bitlang sourceの `Private module` はcanonicalにこの状態へ解決する。

### Module_visible

同じcanonical moduleからだけ参照可能。child moduleを自動的には含めない。

### Module_tree_visible

宣言moduleおよびそのdescendant module tree内から参照可能。

### Project_visible

同一Bitlang project内から参照可能。

### Global_visible

cross-project lookup/importへ参加可能。ただしExported/Unexported、dependency、import capability等の別条件を無視しない。

### Custom_scope_visibility

標準境界だけで表現できない場合に、明示的なcompile-time visibility relation expressionを付与する。

## 2. Public / Private の正規化

`Public / Private` はBitlang sourceで利用できるshorthandとして残すが、Explicit境界では具体的な正規状態へ解決する。

通常:

```text
Public
    -> Global_visible
```

`Private` は対象種別の自然なowner境界へ解決する。

少なくとも:

```text
Private module
    -> File_visible

Private class/type member
    -> Owner_visible
```

後段が `Private` の意味を対象種別から推測し直してはならない。

## 3. custom visibility expression

`Custom_scope_visibility` は任意のコードを実行する仕組みではなく、解決済みscope relationだけを評価する純粋な式である。

boolean composition:

```text
any(...)
all(...)
not(...)
```

代表的relation:

```text
same_owner
same_file
same_module
within_module(<module>)
module_descendant_of(<module>)
same_project
within_project(<project>)
project_descendant_of(<project>)
subtype_of(<type>)
named_scope(<scope-id>)
named_symbol(<symbol-id>)
adapter_domain(<adapter-id>, <domain-id>)
```

これにより、package/internal/crate/assembly/friend/pub(super)/protected-internal等をsource adapterから意味保存して正規化できる。

expressionのserializationはversioned machine-readable formatを使用する。

## 4. 他の可視性propertyとの独立

`scope_visibility` は次と独立:

```text
inheritance_visibility
    Protected / Unprotected

module_export
    Exported / Unexported
```

ただしsource languageの可視性がscopeとinheritanceのunion/intersectionを要求する場合は、誤った近似をせず `Custom_scope_visibility` へ正規化してよい。

## 5. Private module

sourceの:

```bitlang
Private module Internal {
}
```

は:

```text
scope_visibility = File_visible
```

へ正規化する。

cross-file module discoveryでは候補に含めず、外部importは「module not found」として扱う。same-file referenceは通常のmodule/name規則に従う。

## 6. backend boundary

visibilityはcompile-time semantic情報である。

必要なname resolution / access validation完了後は、reflection/runtime metadata等の別仕様が要求しない限りBitlang Low以降へvisibility policyを持ち越す必要はない。
