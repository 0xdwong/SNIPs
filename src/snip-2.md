---
snip: 2
title: 代币标准
author: Abdelhamid Bakhta <abdelhamid.bakhta@gmail.com>, Nils Bundi <nbundi@proton.me>
status: 草稿
type: 标准跟踪
category: SRC
created: 2022-06-03
---

## 简单总结

代币的标准接口。

灵感来自于 [EIP-20](https://eips.ethereum.org/EIPS/eip-20)。

## 摘要

以下标准允许在智能合约中实现代币的标准 API。

该标准提供了基本功能以转移代币，并允许代币被批准以便可以被其他链上第三方使用。

该标准使用 _蛇形命名法_ 表示法，并可选择性地使用 _驼峰命名法_ 以保持向后兼容性。

## 动机

标准接口允许 StarkNet 上的任何代币被其他应用程序重用：从钱包到去中心化交易所。

## 规范

本文档中的关键字 "MUST"、"MUST NOT"、"REQUIRED"、"SHALL"、"SHALL NOT"、"SHOULD"、"SHOULD NOT"、"RECOMMENDED"、"MAY" 和 "OPTIONAL" 应按 [RFC 2119](https://www.ietf.org/rfc/rfc2119.txt) 中的描述进行解释。

### 方法

**注意**：
 - 以下规范使用来自 Cairo `2.6.0`（或更高版本）的语法。

#### name

返回代币的名称 - 例如 `"MyToken"`。

OPTIONAL - 此方法可用于提高可用性，但接口和其他合约 MUST NOT 期望这些值存在。

```cairo
fn name(self: @TContractState) -> ByteArray
```

#### symbol

返回代币的符号。例如 "HIX"。

OPTIONAL - 此方法可用于提高可用性，但接口和其他合约 MUST NOT 期望这些值存在。

```cairo
fn symbol(self: @TContractState) -> ByteArray
```

#### decimals

返回代币使用的小数位数 - 例如 `8`，表示将代币数量除以 `100000000` 以获得其用户表示。

OPTIONAL - 此方法可用于提高可用性，但接口和其他合约 MUST NOT 期望这些值存在。

``` cairo
fn decimals(self: @TContractState) -> u8
```

#### total_supply

返回代币的总供应量。

```cairo
fn total_supply(self: @TContractState) -> u256
```

#### balance_of

返回另一个地址为 `account` 的账户余额。

``` cairo
fn balance_of(self: @TContractState, account: ContractAddress) -> u256
```

#### transfer

将 `amount` 数量的代币转移到地址 `recipient`，并 MUST 触发 `Transfer` 事件。

如果消息调用者的账户余额不足以支出，函数 SHOULD `throw`。

*注意* 0 值的转移 MUST 被视为正常转移并触发 `Transfer` 事件。

``` cairo
fn transfer(
        ref self: @TContractState, recipient: ContractAddress, amount: u256
    ) -> bool
```

#### transfer_from

将 `amount` 数量的代币从地址 `sender` 转移到地址 `recipient`，并 MUST 触发 `Transfer` 事件。

`transfer_from` 方法用于提取工作流，允许合约代表您转移代币。

这可以用于例如允许合约代表您转移代币和/或收取子货币的费用。

除非 `sender` 账户故意通过某种机制授权消息的发送者，否则函数 SHOULD `throw`。

*注意* 0 值的转移 MUST 被视为正常转移并触发 `Transfer` 事件。

``` cairo
fn transfer_from(
        ref self: @TContractState,
        sender: ContractAddress,
        recipient: ContractAddress,
        amount: u256
    ) -> bool
```

#### approve

允许 `spender` 多次从您的账户提取，最多到 `amount` 数量。如果再次调用此函数，它将用 `amount` 覆盖当前的授权。

``` cairo
fn approve(
        ref self: @TContractState, spender: ContractAddress, amount: u256
    ) -> bool
```

#### allowance

返回 `spender` 仍然被允许从 `owner` 提取的金额。

``` cairo
fn allowance(
        self: @TContractState, owner: ContractAddress, spender: ContractAddress
    ) -> u256
```

### 事件

#### Transfer

MUST 在代币转移时触发，包括零值转移。

创建新代币的代币合约 SHOULD 在代币创建时触发 Transfer 事件，`from` 地址设置为 `0x0`。

``` cairo
#[derive(Drop, PartialEq, starknet::Event)]
struct Transfer {
    #[key]
    from: ContractAddress,
    #[key]
    to: ContractAddress,
    value: u256
}
```

#### Approval

MUST 在任何成功调用 `approve(address spender, uint256 value)` 时触发。

``` cairo
#[derive(Drop, PartialEq, starknet::Event)]
struct Approval {
    #[key]
    owner: ContractAddress,
    #[key]
    spender: ContractAddress,
    value: u256
}
```

## 实现

#### 示例实现可在以下位置找到
- [OpenZeppelin 实现](https://github.com/OpenZeppelin/cairo-contracts/blob/main/src/token/erc20/erc20.cairo)

## 向后兼容性

为了向后兼容，建议 `total_supply`、`balance_of` 和 `transfer_from` 方法也使用 _驼峰命名法_ 表示法，或分别为 `totalSupply`、`balanceOf`、`transferFrom`。

这是可选的，MUST NOT 替换使用 _蛇形命名法_ 的方法。

## 安全考虑

请注意，一些智能合约系统依赖于此处定义的方法的 _驼峰命名法_。如果您的应用程序针对与这些现有系统的集成，请确保遵循“向后兼容性”部分中的建议。

## 历史

__2022-06-03__: 初始发布

__2024-05-02__: 更新发布，引入 _蛇形命名法_、_驼峰命名法_ 兼容性和 Cairo 0.12.0 语法

## 版权

版权及相关权利通过 [MIT](../LICENSE) 放弃。