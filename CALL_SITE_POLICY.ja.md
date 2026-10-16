# Bitlang Explicit Call-Site Policy

Status: current canonical design

Bitlang Explicitでは、callable declarationに対して「どのcallerからのcallを許可するか」をcompile-time policyとして完全明示できる。

## 1. 正規property axis

```text
Call_site_unrestricted
Custom_call_site_policy
```

canonical targetは `function`。method / constructor相当のcallableもnormalization後はこの契約へ従う。

`Call_site_unrestricted` は追加のcaller制限なし。

`Custom_call_site_policy` はversioned machine-readable call-policy expressionを持つ。

## 2. caller function name

代表的なpredicate:

```text
caller_function_name_exact(<name>)
caller_function_name_prefix(<prefix>)
caller_function_name_suffix(<suffix>)
caller_function_name_glob(<glob>)
```

name matchはBitlangのcanonical identifierに対して行う。

- 大文字小文字は識別子identityへ影響しない。
- `_` は有意。
- diagnostic用の元source spellingではなくcaller declarationのprimary canonical function nameを判定する。
- aliasはdefaultのcaller name判定へ影響しない。primary nameが `main` のfunctionへ `log_main` aliasを追加しても `log_*` policyを満たさない。

glob:

- `*` = 0文字以上
- `?` = ちょうど1文字
- regex/runtime scriptは使用しない

## 3. その他のcaller relation

```text
caller_function(<canonical-symbol>)
caller_owner_type(<canonical-type>)
caller_module(<canonical-module>)
caller_project(<project-id>)
caller_adapter_domain(<adapter-id>, <domain-id>)
```

composition:

```text
any(...)
all(...)
not(...)
```

## 4. indirect call

function valueへ変換してもpolicyを失わない。

indirect callのreachable targetに制限付きfunctionが含まれる場合、call siteが全targetのpolicyを満たすことをcompilerが証明できなければならない。

証明不能をsilentに許可しない。

## 5. direct caller semantics

call policyはruntime call stackではなく、invocationのdirect semantic callerを評価する。

allowed callerの内部から一般wrapperを呼び、そのwrapperがrestricted functionを呼んだ場合、restricted functionから見たcallerはwrapperである。

transitive permissionが必要なら、wrapper自体をallowed relationへ明示する。

## 6. 他のrestrictionとの独立

visibility / export / import capability / caller-class count restrictionとは独立。

最終的なcallは全てのapplicable restrictionを満たす必要がある。

## 7. backend

検証後はBitlang Lowへcall-site policyを残す必要はない。Assam Core / VMにcaller-name runtime checkを追加しない。
