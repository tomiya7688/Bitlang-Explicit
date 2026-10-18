# Bitlang Explicit Property Transition 仕様

Bitlang Explicitはdeclarationのpropertyを一生固定された集合として扱わない。

## 1. 初期状態と遷移

各declarationは完全解決済みinitial property stateを持ち、その後のprogram pointでcanonical property transitionが発生し得る。

```text
initial property state
-> control flow
-> property transition
-> effective property state
```

static analysisは各operation位置のeffective stateを使用する。

## 2. Unreadable

`Unreadable` は「永久に読めない」ではなく、そのprogram pointではread禁止という意味。

例:

```text
Unreadable Writeable Uninitialized ... value
write(value, ...)
Readableへの正当なtransition
read(value)
```

ただし `Unreadable -> Readable` はinitialization、move/release復活、ownership取得等を兼ねない。他axisもreadを許可している必要がある。

## 3. transition class

property axisはtransition semanticsを明示する。

```text
flow_transitionable
structural
operation_derived
```

初期分類:

```text
Readable / Unreadable             flow_transitionable
Writeable / Unwriteable           flow_transitionable
Reassignable / Unreassignable     flow_transitionable

borrow_state                      operation_derived
move_state                        operation_derived
release_state                     operation_derived
initialization                    operation_derived

visibility/export/retention/
instance_access/lifetime/
nullability/optionality/
caller policy                     structural
```

その他axisは個別transition仕様が確定するまでfree transitionを許可しない。

## 4. control-flow merge

property stateはpath-sensitiveに追跡する。

joinで異なるstateが来た場合、より強いcapabilityへsilent mergeしない。

後続operationが要求するstateは、すべてのreachable incoming pathで成立するか、現在pathのrefinementで証明されなければならない。

## 5. canonical representation

Explicitはinitial declaration propertiesに加え、後続のproperty transition operationを保持する。

したがってfully explicitとは「declarationに一回書けば終わり」ではなく、semantic state changeも明示されていることを意味する。

## 6. lowering

transitionとeffective stateを検証した後、Bitlang Lowへ同名property metadataを保持する必要はない。low-level access/copy/move/release/control-flowへ意味を反映できれば消去可能。
