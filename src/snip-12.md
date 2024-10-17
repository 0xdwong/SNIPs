---
snip: 12
title: 链外签名（类似于 EIP712）
authors: Gaëtan A. <@gaetbout>, Sergio sgc <@sgc-code>, Julien Niset <@juniset>
discussions-to: https://community.starknet.io/t/snip-off-chain-signatures-a-la-eip712/98029
status: Review
type: Standards Track
category: SRC
created: 2023-11-10
---


## 摘要

与 EIP712 一样，这是一个用于哈希和签名类型结构化数据的标准，而不仅仅是十六进制（或 felt）值在 Starknet 中。

其目的不是定义您应该如何设计协议。

## 动机

盲目地签署一些随机的十六进制值并不是很用户友好，但更重要的是，这非常危险。用户需要通过展示他可以理解的值来理解他即将签署的内容。

本文档旨在创建一个与现有 Dapp、钱包和智能合约兼容的标准，同时增加一些额外功能以表达新类型，以帮助更好地显示。本文档整合了一些之前在 Starknet 中创建链外签名的努力（其中一些文档不够完善）。

以下是一个 NFT 销售订单的示例，以及钱包今天能够显示的内容与在此规范改进后可以做到的内容的对比

![wallet rev 0 vs rev 1](../assets/snip-12/wallet-example.png)

## 规范

受 EIP-712 启发，我们可以将链外消息的编码定义为：

```jsx
signed_data = encode(PREFIX_MESSAGE, Enc[domain_separator], account, Enc[message])
```

`hash_array(array)`  
对于修订 `0`：将使用 `pedersen` 函数作为哈希函数。见：  
https://docs.starknet.io/documentation/architecture_and_concepts/Cryptography/hash-functions/#pedersen_array_hash

对于修订 `1`：将使用 `poseidon` 函数作为哈希函数。见：  
https://docs.starknet.io/documentation/architecture_and_concepts/Cryptography/hash-functions/#poseidon_array_hash

`starknet_keccak(str)`  
作为对 str 的 starknet_keccak 哈希。见：  
https://docs.starknet.io/documentation/architecture_and_concepts/Cryptography/hash-functions/#starknet_keccak

`serialise(x)`  
作为 cairo 将值转换为 felt 的方式

`escape(name)`  
对于修订 `0`：返回与输入相同的内容。  
对于修订 `1`：带有任何转义的双引号名称。遵循 JSON 对象的规范。见：  
https://www.json.org/json-en.html

### 前缀消息

`PREFIX_MESSAGE` **必须是** `StarkNet Message`。  
这旨在区分将来使用的链外发送消息和将直接发送到排序器进行链上处理的交易。

### 域分隔符

`domain_separator` 定义为以下对象。

```js
"StarknetDomain": [
  { "name": "name", "type": "shortstring" }, 
  { "name": "version", "type": "shortstring" },
  { "name": "chainId", "type": "shortstring" },
  { "name": "revision", "type": "shortstring" }
]
```

该对象确保基于以下内容的消息唯一性：

- **name**：Dapp 的名称，如果您的合约需要执行多个链外签名，甚至可以包含函数名称。
- **version**：您的合约使用的 Dapp 版本。防止同一 Dapp 的两个版本生成相同的哈希。通常，如果您更新合约并且哈希行为发生变化，则应更新此字段。
- **chainId**：Dapp 使用的链 ID 表示为短字符串。防止从一个网络到另一个网络的重放攻击。
- **revision (可选)**：要使用的规范修订版。如果省略该值，则默认为 `0`。
    - 修订 `0`：表示在发布此 SNIP 之前的事实规范。目的是帮助向后兼容。不推荐使用。
    - 修订 `1`：将是规范的初始版本。请注意，对于此修订，字段中的值应为整数 `1` 而不是短字符串 `"1"`，尽管被定义为短字符串。此例外是为了支持 Braavos 钱包实现中的不一致性。见下文的 [示例](#json-example)。

在修订 `0` 中引入，在修订 `1` 中更改  
在修订 `0` 中，字段 `name`、`version` 和 `chainId` 的类型为 `felt`。  
从修订 `1` 开始，这些字段使用 `shortstring` 类型。

在修订 `0` 中，域对象命名为 `StarkNetDomain`。  
从修订 `1` 开始，域对象命名为 `StarknetDomain`。  
当使用旧版本钱包的用户仅支持修订 `0` 时，如果收到使用修订 `1` 的签名请求，则会出现问题。过时的钱包在不知道修订 `1` 的情况下，会以不同的方式计算哈希，因此会生成无效签名。  
这就是我们将 `StarkNetDomain` 更改为 `StarknetDomain` 的原因。  
如果 dapp 请求钱包使用 `StarknetDomain` 签署某些内容，则应失败，因为它期望 `StarkNetDomain`。

### 账户

`account` 是被签名的账户合约的合约地址。  
这防止两个账户为相同消息生成相同的哈希。

### 消息

是要签名的交易消息，表示为一个对象。

## 如何处理每种类型

 `type_hash(x) = starknet_keccak(encode_type(x))`

**注意** `type_hash` 对于给定对象/枚举是常量，在智能合约中运行交易时不需要计算。

### 类型识别

有三种类型：
- 基本类型：在此规范中为给定修订定义。例：felt、ClassHash、timestamp、u128
- 预设类型：在规范中定义的结构体。例：TokenAmount、NftId、u256。它们也依赖于所使用的修订版
- 用户定义类型：请求的 "types" 字段中的类型。它们还包括域分隔符（例：`StarknetDomain`）

用户定义类型必须遵循一些规则，如果不满足，则请求必须被拒绝：
- 域分隔符必须严格遵循 "域分隔符" 部分中定义的格式
- 名称不能为空
- 名称不能与基本类型匹配，如 felt、ClassHash、timestamp、u128
- 名称不能与预设类型匹配，如 TokenAmount、NftId、u256
- 名称不能以 * 结尾
- 名称不能用括号括起来
- 名称不能包含逗号 (,) 字符（因为它在枚举类型中用作分隔符）
- 不能定义重复的类型
- 所有枚举变体类型必须用括号括起来，其他类型不能用括号括起来
- 类型必须是基本类型、预设类型或用户定义类型，其他类型不允许
- 所有定义的类型必须被其他类型引用（不能有悬空类型）

### 当 X 是一个对象时

#### **编码**

`Enc[x] = hash_array(type_hash(MyObject), Enc[param1], Enc[param2], ..., Enc[paramN])`

示例：

```js
"My Object": [
  { "name": "Param 1", "type": "u128" },
  { "name": "Param 2", "type": "u128*" },
  { "name": "Param 3", "type": "selector" },
  { "name": "Param 4", "type": "Other Object" },
  { "name": "Param 5", "type": "merkletree" },
  // ...
  { "name": "Param N", "type": "u128" }
]
```

#### **encode_type**

 `escape(name) || "(" || escape(param1_name) || ":" || escape(param1_type) || "," || ... || escape(paramN_name) || ":"|| escape(paramN_type) || ")"` 

如果对象引用其他对象/枚举，这些对象也可以引用其他对象/枚举，则收集引用的对象/枚举，按名称排序，并附加到编码中。

如果我们回到之前使用的示例，我们有：  
`type_hash(MyObject) = starknet_keccak('"My Object"("Param 1":"u128","Param 2":"u128*","Param 3":"selector","Param 4":"Other Object","Param 5":"merkletree",...,"Param N":"u128")"Other Object"("Param 1":"u128"...)')`

### 当 X 是一个数组时

在修订 `0` 中引入

#### **编码**

`Enc[X=(x0, x1, ..., xN)] = hash_array([Enc[x0], Enc[x1], ... Enc[xN]])`

#### **encode_type**

类型为 `InnerType` 的数组必须编码为 `InnerType*`。  
内部类型可以是本规范中支持的任何其他类型。
### 当 X 是一个 felt

在修订版 `0` 中引入  
通常不推荐使用，因为很难以用户友好的方式显示。通常可以使用更具体的类型

**编码** `Enc[x] = serialise(x)`，**编码类型** `felt`

### 当 X 是一个 bool

在修订版 `0` 中引入

#### **编码**

`Enc[x] =` 

`0` 表示假  
`1` 表示真

**编码类型：** `bool` 

### 当 x 是一个字符串

在修订版 `0` 中引入，在修订版 `1` 中更改

在修订版 `0` 中，这表示最多 31 个 ASCII 字符的字符串。  
从修订版 `1` 开始，这种类型将表示任意大小的字符串。  
如果只需要 31 个字符，类型“shortstring”可能更合适  
**编码** `Enc[x] = hash_array(serialise(x))`，**编码类型** `string`

### 当 X 是一个选择器

在修订版 `0` 中引入

这表示智能合约函数的名称。

**编码** `Enc[x] = starknet_keccak(x)`，**编码类型** `selector`

### 当 X 是一个 merkle 树

在修订版 `0` 中引入

这种类型允许钱包签署大量数据，但只签署其 merkle 树的根，从而使链上验证更便宜。但仍然能够向用户显示所有数据

#### **编码**

`Enc[X=(x0, x1, ..., xN)] = calculate_merkle_tree_root(x0, x1, ..., xN)`

X 是我们将作为 merkle 树签署的相同类型的项的列表。

用于 merkle 树的哈希函数将是：  
对于修订版 `0`：`pedersen`  
对于修订版 `1`：`poseidon`  

**编码类型** `merkletree`

在钱包层面，仅提供 merkle 树根而不包含任何数据是不安全的。钱包还需要接收数据，这就是需要额外参数的原因。  
参数 `contains` 需要被指定，它将指代一个对象类型，用于表示叶子作为一个对象：

```js
// ...
"Example": [
  { "name": "Contract Addresses", "type": "merkletree", "contains": "Leaf" },
],
"Leaf": [
  { "name": "Contract Address", "type": "ContractAddress" }
]
// ...
```

钱包将从 Dapp 接收一组叶子，以便可以向用户显示这些叶子。然后它应该对所有叶子进行哈希，并确保根是相同的：

```js
// ...
"Contract Addresses": [
  {
    "Contract Address": "0x...123"
  },
  // ...
  {
    "Contract Address": "0x..beaf"
  }
]
// ...
```

为了计算 Merkle 根，钱包将每个叶子编码为一个单一的 felt（使用本文档中使用的相同编码）。 

在验证链外签名时，只需向合约提供树的根。验证 Merkle 证明将需要验证链外签名以及验证证明。

### 当 X 是一个 u128

在修订版 `1` 中引入

使用最多 128 位的无符号整数

**编码** `Enc[x] = serialise(x)`，**编码类型** `u128`

### 当 X 是一个 i128

在修订版 `1` 中引入

使用最多 128 位的带符号整数（包括符号）

**编码** `Enc[x] = serialise(x)`，**编码类型** `i128`

### 当 X 是一个 ContractAddress

在修订版 `1` 中引入

表示一个 starknet 合约地址。参见：  
https://docs.starknet.io/documentation/architecture_and_concepts/Smart_Contracts/contract-address/

**编码** `Enc[x] = serialise(x)`，**编码类型** `ContractAddress`

### 当 X 是一个 ClassHash

在修订版 `1` 中引入

表示一个 Starknet 类哈希。参见：  
https://docs.starknet.io/documentation/architecture_and_concepts/Smart_Contracts/class-hash/

**编码** `Enc[x] = serialise(x)`，**编码类型** `ClassHash`

### 当 X 是一个时间戳

在修订版 `1` 中引入

将被视为一个 `u128`，表示以秒为单位的时间戳。该类型的目的是允许钱包相应地格式化值

**编码** `Enc[x] = serialise(x)`，**编码类型** `timestamp`

### 当 X 是一个 u256

在修订版 `1` 中引入

将被编码为以下对象，分割低/高 128 位。该类型不需要在 `types` 部分声明。

```js
"u256": [
  { "name": "low", "type": "u128" },
  { "name": "high", "type": "u128" }
]
```

### 当 X 是一个 Token Amount

在修订版 `1` 中引入

将被编码为以下对象。该类型不需要在 `types` 部分声明。

这允许钱包将代币与金额分组以便更好地显示。钱包能够显示正确的小数、法定货币值、图标等…

```js
"TokenAmount": [
  { "name": "token_address", "type": "ContractAddress" },
  { "name": "amount", "type": "u256" }
]
```

### 当 X 是一个 Nft ID

在修订版 `1` 中引入

将被编码为以下对象。该类型不需要在 `types` 部分声明。

这允许钱包将代币 ID 与合约地址分组以便更好地显示。钱包将能够显示正确的代币信息、图像和其他属性）

```js
"NftId": [
  { "name": "collection_address", "type": "ContractAddress" },
  { "name": "token_id", "type": "u256" }
]
```

### 当 x 是一个 shortstring

在修订版 `1` 中引入

如果您使用的是修订版 `0`，则应使用类型“string”

该类型仅允许最多 31 个 ASCII 字符。

最终该规范应允许更长的字符串，但我们在等待规范在 Cairo 语言上最终确定（理想情况下在修订版 1 中解决）

**编码** `Enc[x] = serialise(x)`，**编码类型** `shortstring`

### 当 X 是一个枚举

在修订版 `1` 中引入

示例：

```js
{
  "types": {
    // ...
    "Example": [
      { "name": "some_enum", "type": "enum", "contains": "My Enum" },
    ],
    "My Enum": [
      { "name": "Variant 1", "type": "()" }
      { "name": "Variant 2", "type": "(u128, u128*)," }
      // ...
      { "name": "Variant N", "type": "(u128)" }
    ]
  },
  // ...
  "message": {
    // ...
    "Some Enum": { "Variant 2": [32, [12, 32]] }
    "Some Other Enum": { "Variant 1": [] }
  }
}

```

#### **编码**

`Enc[enum] = hash_array(type_hash(enum), variant_index, Enc[chosen_variant_parameter1],..., Enc[chosen_variant_parameterN])`  

#### **编码类型**

`escape(enum_name) || "(" || escape(variant_1_name) || "(" || escape(param1_type) || "," || ... || escape(paramN_type) || ")," || ... || escape(variant_n_name) || "(" || ... || ")" || ")"`

如果枚举引用其他对象/枚举，且这些对象/枚举也可以引用其他对象/枚举，则收集引用的对象/枚举，按名称排序，并附加到编码中。 

如果我们回到之前使用的示例，我们有：  
`type_hash(MyEnum) = starknet_keccak('"My Enum"("Variant 1"(),"Variant 2"("u128","u128*"),...,"Variant N"("u128"))')`

### 当 X 是其他类型

请求应被视为无效

### JSON 示例

```js
{
  "types": {
    "StarknetDomain": [
      { "name": "name", "type": "shortstring" },
      { "name": "version", "type": "shortstring" },
      { "name": "chainId", "type": "shortstring" },
      { "name": "revision", "type": "shortstring" } 
    ],
    "Example Message": [
      { "name": "Name", "type": "string" },
      { "name": "Some Array", "type": "u128*" },
      { "name": "Some Object", "type": "My Object" }
    ],
    "My Object": [
      { "name": "Some Selector", "type": "selector" },
      { "name": "Some Contract Address", "type": "ContractAddress" },
    ],
  },
  "primaryType": "Example Message",
  "domain": {
    "name": "Starknet Example",
    "version": "1",
    "chainId": "SN_MAIN",
    "revision" : 1
  },
  "message": {
    "Name": "some name"
    "Some Array": [1, 2, 3, 4],
    "Some Object": {
      "Some Selector": "transfer",
      "Some Contract Address": "0x0123"
    },
  }
}
```
**注意：** 字段 `revision` 的值是整数 `1`，尽管该字段的类型是 `shortstring`

## 实现

在这里找到一个示例库以获取更详细的示例。  
请注意，此实现使用 Pedersen 作为哈希函数。  
https://github.com/argentlabs/starknet-off-chain-signature

## 参考

1. https://github.com/argentlabs/argent-x/discussions/14
2. https://www.starknetjs.com/docs/guides/signature/#sign-and-verify-following-eip712
3. https://eips.ethereum.org/EIPS/eip-712
4. https://github.com/0xs34n/starknet.js/blob/develop/__mocks__/typedDataExample.json
5. [https://github.com/0xs34n/starknet.js/blob/develop/src/utils/typedData.ts](https://github.com/0xs34n/starknet.js/blob/develop/src/types/typedData.ts)

## 安全考虑

此 SNIP 在安全性方面没有任何影响。

## 版权

版权及相关权利通过 [MIT](../LICENSE) 放弃。