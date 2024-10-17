---
snip: 5
title: 标准接口检测
authors: Eric Nordelo <eric.nordelo39@gmail.com>, Francisco Giordano <frangio.1@gmail.com>, Martin Triay <martriay@gmail.com>, Sergio Garcia <sergio@argent.xyz>
discussions-to: https://community.starknet.io/t/starknet-standard-interface-detection/92664/30
status: 草稿
type: 标准跟踪
category: SRC
created: 2023-05-29
---

## 简要总结

一种发布和检测智能合约实现的接口的标准方法。
灵感来自于 [ERC-165](https://eips.ethereum.org/EIPS/eip-165)。

## 摘要

这标准化了：

1. 如何识别接口。
2. 合约如何发布其实现的接口。
3. 如何检测合约是否实现了 SRC-5。
4. 如何检测合约是否实现了任何给定的接口。

## 动机

对于一些“标准接口”，如 ERC-721 代币接口，有时查询合约是否支持该接口以及如果支持，支持哪个版本的接口是有用的，以便调整与合约的交互方式。该提案标准化了接口的概念，并标准化了接口的识别（命名）。

## 规范

本文档中的关键词“MUST”、“MUST NOT”、“REQUIRED”、“SHALL”、“SHALL NOT”、“SHOULD”、“SHOULD NOT”、“RECOMMENDED”、“MAY”和“OPTIONAL”应按 [RFC 2119](https://www.ietf.org/rfc/rfc2119.txt) 中描述的方式进行解释。

### 序列化兼容性

该标准的目标之一是确保所有声明实现给定接口的合约，对于调用者的行为是相似且可预期的。在 Cairo 中，语言不强制对结构体和枚举使用编码格式，开发者需要在这些类型作为外部函数签名的一部分时实现 Serde 特性，不同的实现可能导致暴露相同接口的合约之间的不兼容。

我们定义，为了符合该标准，作为参数或外部函数返回类型使用的结构体和枚举应实现来自 `[#derive(Serde)]` 属性的默认格式，即：结构体的序列化字段的连接，以及作为变体标识符的 felt252 和枚举的序列化值的连接。其他类型必须使用语言核心库中指定的序列化格式。

### 接口

接口是一组具有具体类型参数的函数签名，通常由 `trait` 表示。这些接口旨在由符合该接口的合约作为 `external` 实现。例如：

```cairo
trait IMyContract {
    fn foo(some: u256) -> felt252;
}
```

#### 泛型类型

自 Cairo 2.0 起，我们可以定义利用泛型类型表示一组接口的特性：

```cairo
#[starknet::interface]
trait IMyContract<TContractState, TNumber> {
    fn foo(self: @TContractState, some: TNumber) -> felt252;
}
```

请注意，这些特性并不表示特定合约的实际公共接口，而是它们的一种类别。泛型 `TContractState` 类型自参数不包含在函数的公开 API 中。它在内部用于限制函数如何访问本地存储，并不是函数的公共 API 的一部分。`TNumber` 类型参数必须在实现中具体化，因为合约的外部函数中不允许使用泛型类型参数。

根据该标准，带有 `#[starknet::interface]` 属性的泛型特性表示一组接口，而每个真实接口可以表示为非泛型特性，如上所示。

### 扩展函数选择器

在 Starknet 中，函数选择器是函数名称的 `starknet_keccak`（ASCII 编码）。对于该标准，我们将扩展函数选择器定义为函数签名的 `starknet_keccak`，该签名格式如下：

```
fn_name(param1_type,param2_type,...)->output_type
```

其中 `fn_name` 是函数名称，`paramN_type` 是第 n 个函数参数的类型，`output_type` 是返回值的类型。

零参数且没有返回值的函数的签名是：

```
fn_name()
```

类型是 [corelib](https://github.com/starkware-libs/cairo/blob/main/corelib/src/lib.cairo) 中定义的类型（例如：`type felt252`）。元组、结构体和枚举被视为特殊类型。例如，`u256` 表示为 `(u128,u128)`，其中 `u128` 是一种类型，`u256` 是一种结构体。

### 特殊类型（元组、结构体和枚举）

提供这些参数到签名以获取 [扩展函数选择器](#extended-function-selector) 的定义：

#### 元组

元组的签名为：`(elem1_type,elem2_type,...)`，其中 `elemN_type` 是第 n 个元组成员的类型。

#### 结构体

具有 `n` 个字段的结构体的签名为：`(field1_type,field2_type,...)`，其中 `fieldN_type` 是第 n 个结构体字段的类型。

#### 枚举

具有 `n` 个字段的枚举的签名为：`E(variant1_type,variant2_type,...)`，其中 `variantN_type` 是第 n 个枚举变体的类型。

前导的 `E` 避免与使用元组或结构体的类似签名发生冲突。

### 示例

1. 从 Cairo 函数：

```cairo
#[derive(Drop, Serde)]
enum MyEnum {
    FirstVariant: (felt252, u256),
    SecondVariant: Array<u128>,
}

#[derive(Drop, Serde)]
struct MyStruct {
    field1: MyEnum,
    field2: felt252,
}

fn foo(param1: @MyEnum, param2: MyStruct) -> bool;
```

签名为：

```cairo
foo(@E((felt252,(u128,u128)),Array<u128>),(E((felt252,(u128,u128)),Array<u128>),felt252))->E((),())
```

### 如何识别接口

对于该标准，我们将接口标识符定义为所有 [扩展函数选择器](#extended-function-selector) 的异或。以下代码示例展示了如何计算接口标识符：

从这个 Cairo 接口：

```cairo
struct Call {
    to: ContractAddress,
    selector: felt252,
    calldata: Array<felt252>
}

trait IAccount {
    fn supports_interface(felt252) -> bool;
    fn is_valid_signature(felt252, Array<felt252>) -> bool;
    fn __execute__(Array<Call>) -> Array<Span<felt252>>;
    fn __validate__(Array<Call>) -> felt252;
    fn __validate_declare__(felt252) -> felt252;
}
```

这是计算接口 ID 的 Python 代码：

```python
# pip install cairo-lang
from starkware.starknet.public.abi import starknet_keccak

# These are the public interface function signatures
extended_function_selector_signatures_list = [
    'supports_interface(felt252)->E((),())',
    'is_valid_signature(felt252,Array<felt252>)->E((),())',
    '__execute__(Array<(ContractAddress,felt252,Array<felt252>)>)->Array<(@Array<felt252>)>',
    '__validate__(Array<(ContractAddress,felt252,Array<felt252>)>)->felt252',
    '__validate_declare__(felt252)->felt252'
]

def main():
    interface_id = 0x0
    for function_signature in extended_function_selector_signatures_list:
        function_id = starknet_keccak(function_signature.encode())
        interface_id ^= function_id
    print('IAccount ID:')
    print(hex(interface_id))


if __name__ == "__main__":
    main()
```

#### 工具

[src5-rs](https://github.com/ericnordelo/src5-rs) 是一个用于从 Cairo 特性生成 SRC5 接口 ID 的工具，使用 Cairo 源代码作为输入。

### 合约如何发布其实现的接口

符合 SRC-5 的合约应实现以下接口（称为 `ISRC5.cairo`）：

```cairo
trait ISRC5 {
    /// @notice Query if a contract implements an interface
    /// @param interface_id The interface identifier, as specified in SRC-5
    /// @return `true` if the contract implements `interface_id`, `false` otherwise
    fn supports_interface(interface_id: felt252) -> bool;
}
```

该接口的标识符为 `0x3f918d17e5ee77373b56385708f855659a07f75997f365cf87748628532a055`。您可以通过运行 `starknet_keccak('supports_interface(felt252)->E((),())')` 来计算此值。请注意，返回类型 `bool` 表示为 `E((),())`，因为它是 corelib 中定义的枚举。

因此，实现合约将具有一个 `supports_interface` 函数，该函数返回：

- 当 `interface_id` 为 `0x3f918d17e5ee77373b56385708f855659a07f75997f365cf87748628532a055`（SNIP-5 接口）时返回 `true`
- 对于该合约实现的任何其他 `interface_id` 返回 `true`
- 对于任何其他 `interface_id` 返回 `false`

该函数必须返回一个 bool。

### 如何检测合约是否实现 SRC-5

1. 源合约向目标地址发出 `call_contract_syscall`，`entrypoint_selector` 为：`0xfe80f537b66d12a00b6d3c072b44afbb716e78dde5c3f0ef116ee93d3e3283`，calldata 为一个元素的 Span，包含：`0x3f918d17e5ee77373b56385708f855659a07f75997f365cf87748628532a055`。这对应于 `contract.supports_interface(0x3f918d17e5ee77373b56385708f855659a07f75997f365cf87748628532a055)`。
2. 如果调用失败或返回 false，则目标合约不实现 SRC-5。
5. 否则，它实现了 SRC-5。

### 如何检测合约是否实现任何给定接口

1. 如果您不确定合约是否实现了 SRC-5，请使用上述程序进行确认。
2. 如果合约不实现 SRC-5，则您需要以传统方式查看合约使用了哪些方法。
3. 如果合约实现了 SRC-5，请调用 `supports_interface(interface_id)` 以确定合约是否实现了您可以使用的接口。
## 版权

通过 [MIT](../LICENSE) 放弃版权及相关权利。