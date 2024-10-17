---
snip: 22
title: 代币化金库
description: 扩展 SNIP-2 以支持代币化、收益产生的金库
author: Nils Bundi <nbundi@proton.me>, Johannes Escherich <0xJohannes@pm.me>
discussions-to: https://community.starknet.io/t/snip-22-tokenized-vaults/114457
status: 审核
type: 标准跟踪
category: SRC
created: 2024-08-14
requires: 2
---


## 摘要

扩展 [SNIP-2](./snip-2.md) 代币标准，并增加代币化金库的基本功能，表示基础资产的份额，包括存款、取款、销毁、铸造和查看余额。灵感来源于 [EIP-4626](https://eips.ethereum.org/EIPS/eip-4626)。


## 动机

代币化金库是许多 DeFi 应用程序中广泛使用的模式，包括借贷市场、聚合器和收益代币。当前的金库实现暴露了多样化的接口。代币化金库的标准 API 将降低协议、聚合器和钱包的集成工作量，改善用户体验并提高用户的安全性。


## 规范

本文档中的关键字 "MUST"、"MUST NOT"、"REQUIRED"、"SHALL"、"SHALL NOT"、"SHOULD"、"SHOULD NOT"、"RECOMMENDED"、"MAY" 和 "OPTIONAL" 应按 [RFC 2119](https://www.ietf.org/rfc/rfc2119.txt) 中的描述进行解释。


### SNIP-2 兼容性

所有代币化金库 MUST 实现 SNIP-2 标准，包括其可选的元数据扩展。

SNIP-2 操作 `balance_of`、`transfer`、`total_supply` 等 MUST 在金库份额上操作。

如果金库份额不可转让或代币化金库实现所需的某些条件未满足，调用 `transfer` 或 `transfer_from` MAY 会回退。

SNIP-2 可选操作 `name` 和 `symbol` SHOULD 以某种方式反映基础资产的 `name` 和 `symbol`。

所有代币化金库 MUST 实现一组附加函数，以便管理金库的基础资产。


### 定义：

- _asset_: 金库管理的基础代币
- _share_: 金库资产的所有权单位，由金库的代币表示
- _fee_: 金库通过某些金库功能向用户收取的资产或份额的金额
  （例如存款/取款/铸造/销毁等）。
- _slippage_: 计算的份额价格与金库存款/取款的经济现实之间的任何差异，这些差异未被费用考虑在内。
- _caller_: 发起对代币化金库方法调用的合约


### 方法

#### asset

返回基础代币的地址。

返回的地址 MUST 代表一个 SNIP-2 代币合约。

MUST _NOT_ 回退。

```cairo
fn asset(self: @TContractState) -> ContractAddress
```

#### total_assets

返回金库“管理”的基础代币的总量。

SHOULD 包括因收益而发生的任何复利。

MUST 包括对金库中资产收取的任何费用。

MUST _NOT_ 回退。

```cairo
fn total_assets(self: @TContractState) -> u256;
```

#### convert_to_shares

模拟在当前区块中，金库将为提供的 `assets` 交换的份额数量。

MUST NOT 包括对金库中资产收取的任何费用。

MUST NOT 根据调用者显示任何变化。

MUST NOT 反映滑点或其他链上条件，在执行实际交换时。

MUST NOT 回退。

```cairo
fn convert_to_shares(self: @TContractState, assets: u256) -> u256;
```

#### convert_to_assets

模拟在当前区块中，金库将为提供的 `shares` 交换的资产数量。

MUST NOT 包括对金库中资产收取的任何费用。

MUST NOT 根据调用者显示任何变化。

MUST NOT 反映滑点或其他链上条件，在执行实际交换时。

MUST NOT 回退。

```cairo
fn convert_to_assets(self: @TContractState, shares: u256) -> u256;
```

#### max_deposit

返回可以通过 `deposit` 调用存入金库的基础代币的最大数量，针对 `receiver`。

MUST NOT 考虑调用者的可用资产余额（即调用者的 `asset` 的 `balance_of`）。

MUST 考虑全球和用户特定的限制，例如如果存款完全被禁用（即使是暂时的），它 MUST 返回 0。

如果没有对最大可存入资产数量的限制，MUST 返回 `2 ** 256 - 1`。

MUST NOT 回退。

```cairo
fn max_deposit(self: @TContractState, receiver: ContractAddress) -> u256
```

#### preview_deposit

模拟在当前区块中资产存款的效果。

MUST 返回尽可能接近且不超过在同一交易中 `deposit` 调用时铸造的金库份额的确切数量。即，如果在同一交易中调用，`deposit` 应返回与 `preview_deposit` 相同或更多的 `shares`。

MUST NOT 考虑如 `max_deposit` 返回的存款限制，并且应始终假设存款将被接受，无论用户是否有足够的代币批准等。

MUST 包括存款费用。

MUST NOT 因金库特定的用户/全球限制而回退。MAY 因其他条件而回退，这些条件也会导致 `deposit` 回退。

请注意，`convert_to_shares` 和 `preview_deposit` 之间的任何不利差异 SHOULD 被视为份额价格的滑点或其他类型的条件，这意味着存款人通过存款将会损失资产。

```cairo
fn preview_deposit(self: @TContractState, assets: u256) -> u256
```

#### deposit

通过存入确切的基础代币 `assets` 为 `receiver` 铸造 `shares` 金库份额。

MUST 触发 `Deposit` 事件。

MUST 支持在 `asset` 上的 SNIP-2 `approve` / `transfer_From` 作为存款流程。
MAY 支持额外的存款流程。

MUST 在无法存入所有 `assets` 时回退（由于达到存款限制、滑点、用户未向金库合约批准足够的基础代币等）。

```cairo
fn deposit(ref self: @TContractState, assets: u256, receiver: ContractAddress) -> u256
```

#### max_mint

返回可以通过 `mint` 调用从金库铸造的最大份额数量，针对 `receiver`。

MUST NOT 考虑调用者的可用资产余额（即调用者的 `asset` 的 `balance_of`）。

MUST 考虑全球和用户特定的限制，例如如果铸造完全被禁用（即使是暂时的），它 MUST 返回 0。

如果没有对最大可铸造份额数量的限制，MUST 返回 `2 ** 256 - 1`。

MUST NOT 回退。

```cairo
fn max_mint(self: @TContractState, receiver: ContractAddress) -> u256
```

#### preview_mint

模拟在当前区块中金库代币铸造的效果。

MUST 返回尽可能接近且不低于在同一交易中 `mint` 调用时将存入的确切资产数量。即，如果在同一交易中调用，`mint` 应返回与 `preview_mint` 相同或更少的 `assets`。

MUST NOT 考虑如 `max_mint` 返回的铸造限制，并且应始终假设铸造将被接受，无论用户是否有足够的代币批准等。

MUST 包括存款费用。

MUST NOT 因金库特定的用户/全球限制而回退。MAY 因其他条件而回退，这些条件也会导致 `mint` 回退。

请注意，`convert_to_assets` 和 `preview_mint` 之间的任何不利差异 SHOULD 被视为份额价格的滑点或其他类型的条件，这意味着存款人通过铸造将会损失资产。

```cairo
fn preview_mint(self: @TContractState, shares: u256) -> u256
```

#### mint

通过存入 `assets` 的基础代币为 `receiver` 铸造确切的 `shares` 金库份额。

MUST 触发 `Deposit` 事件。

MUST 支持在 `asset` 上的 SNIP-2 `approve` / `transfer_from` 作为铸造流程。
MAY 支持额外的铸造流程。
必须在无法铸造所有 `shares` 的情况下回滚（由于达到存款限制、滑点、用户未向金库合约批准足够的基础代币等）。

```cairo
fn mint(ref self: @TContractState, shares: u256, receiver: ContractAddress) -> u256
```

#### max_withdraw

返回可以从金库中 `owner` 余额提取的基础资产的最大金额，通过 `withdraw` 调用。

必须考虑全球和用户特定的限制，例如如果提取完全被禁用（即使是暂时的），则必须返回 0。

必须不回滚。

```cairo
fn max_withdraw(self: @TContractState, owner: ContractAddress) -> u256
```

#### preview_withdraw

模拟在当前区块下资产提取的效果。

必须返回尽可能接近且不低于在同一交易中 `withdraw` 调用时将被销毁的金库股份的确切数量。即，如果在同一交易中调用 `withdraw`，则应返回与 `preview_withdraw` 相同或更少的 `shares`。

必须不考虑诸如从 maxWithdraw 返回的提取限制，并且应始终假设提取将被接受，无论用户是否拥有足够的股份等。

必须包括提取费用。

必须不因金库特定的用户/全球限制而回滚。可能因其他条件而回滚，这些条件也会导致 `withdraw` 回滚。

请注意，`convert_to_Shares` 和 `preview_withdraw` 之间的任何不利差异应视为股份价格的滑点或其他类型的条件，这意味着存款人通过存款将损失资产。

```cairo
fn preview_withdraw(self: @TContractState, assets: u256) -> u256
```

#### withdraw

从 `owner` 销毁 `shares` 并将确切的 `assets` 基础代币发送给 `receiver`。

必须发出 `Withdraw` 事件。

必须支持一种提取流程，其中股份直接从 `owner` 销毁，且 `owner` 是 _caller_。

必须支持一种提取流程，其中股份直接从 `owner` 销毁，且 _caller_ 对 `owner` 的股份具有 SNIP-2 授权。

可能支持额外的提取流程。

如果无法提取所有 `assets`（由于达到提取限制、滑点、所有者没有足够的股份等），则必须回滚。

可能支持具有额外 _request_ 步骤的异步提取过程，该步骤在单独的方法中执行。

```cairo
fn withdraw(ref self: @TContractState, assets: u256, receiver: ContractAddress, owner: ContractAddress) -> u256
```

#### max_redeem

返回可以从金库中 `owner` 余额赎回的金库股份的最大金额，通过 `redeem` 调用。

必须返回可以从 `owner` 通过 `redeem` 转移的最大股份数量，并且不会导致回滚，这个数量必须不高于实际接受的最大值（如有必要应低估）。

必须考虑全球和用户特定的限制，例如如果赎回完全被禁用（即使是暂时的），则必须返回 0。

必须不回滚。

```cairo
fn max_redeem(self: @TContractState, owner: ContractAddress) -> u256
```

#### preview_redeem

模拟在当前区块下股份赎回的效果。

必须返回尽可能接近且不超过在同一交易中 `redeem` 调用时将被提取的确切资产数量。即，如果在同一交易中调用 `redeem`，则应返回与 `preview_redeem` 相同或更多的 `assets`。

必须不考虑诸如从 `max_redeem` 返回的赎回限制，并且应始终假设赎回将被接受，无论用户是否拥有足够的股份等。

必须包括提取费用。

必须不因金库特定的用户/全球限制而回滚。可能因其他条件而回滚，这些条件也会导致 `redeem` 回滚。

请注意，`convert_to_assets` 和 `preview_redeem` 之间的任何不利差异应视为股份价格的滑点或其他类型的条件，这意味着存款人通过赎回将损失资产。

```cairo
fn preview_redeem(self: @TContractState, shares: u256) -> u256
```

#### redeem

从 `owner` 精确销毁 `shares` 并将 `assets` 基础代币发送给 `receiver`。

必须发出 `Withdraw` 事件。

必须支持一种赎回流程，其中股份直接从 `owner` 销毁，且 `owner` 是 _caller_。

必须支持一种赎回流程，其中股份直接从 `owner` 销毁，且 _caller_ 对 `owner` 的股份具有 EIP-20 授权。

可能支持额外的赎回流程。

如果无法赎回所有 `shares`（由于达到提取限制、滑点、所有者没有足够的股份等），则必须回滚。

可能支持具有额外 _request_ 步骤的异步赎回过程，该步骤在单独的方法中执行。

```cairo
fn redeem(ref self: @TContractState, shares: u256, receiver: ContractAddress, owner: ContractAddress) -> u256
```

### 事件

#### Deposit

`sender` 已将 `assets` 兑换为 `shares`，并将这些 `shares` 转移给 `owner`。

必须在通过 `mint` 和 `deposit` 方法将代币存入金库时发出。

```cairo
#[derive(Drop, starknet::Event)]
struct Deposit {
    #[key]
    sender: ContractAddress,
    #[key]
    owner: ContractAddress,
    assets: u256,
    shares: u256
}
```

#### Withdraw

`sender` 已将 `owner` 拥有的 `shares` 兑换为 `assets`，并将这些 `assets` 转移给 `receiver`。

必须在通过 `redeem` 或 `withdraw` 函数从金库提取股份时发出。

```cairo
#[derive(Drop, starknet::Event)]
struct Withdraw {
    #[key]
    sender: ContractAddress,
    #[key]
    receiver: ContractAddress,
    #[key]
    owner: ContractAddress,
    assets: u256,
    shares: u256
}
```

## 理由

代币化金库代表了在 Starknet 及其他地方各种 DeFi 协议中的一种常见模式。如果基于共同标准，这些协议之间的互操作性将大大提高。该标准定义了一个最小 API，便于高效和安全的集成，同时不限制应用领域。

该 API 定义了 `assets` 和 `shares` 的概念，以及一组允许管理这些概念的函数。它不扩展到任何特定领域的功能，例如不同策略之间的 `rebalancing`，例如来自（收益）聚合器的已知策略。这使得该标准在应用方面保持灵活。

## 向后兼容性

该标准与 SNIP-2 标准完全向后兼容，并且与其他标准没有已知的兼容性问题。

## 参考实现

该标准的实现可以在此处找到：[Vesu v_token](https://github.com/vesuxyz/vesu-v1/blob/main/src/v_token.cairo)。

请注意，此实现需要根据您的特定用例进行调整。

## 安全考虑

代币化金库标准定义了一个标准 API，旨在使收益金库的管理和集成更加高效和安全。

该标准不管理收益来源的任何方面或特定收益来源的标准实现的细节。

因此，该标准不保证标准实现或其收益来源的安全性。

方法 `total_assets`、`convert_to_shares` 和 `convert_to_assets` 是用于显示目的的估算值，并不一定要提供其上下文所暗示的基础资产的 _确切_ 数量。

`preview` 方法尽可能接近 _真实_ 值地模拟各自的值。因此，它们可以通过改变链上条件进行操控，并不总是安全地用作价格预言机。

`convert` 方法以可能不精确的方式估算值。因此，这些估算可以以稳健的方式实现，并作为价格预言机。

因此，在集成代币化金库时，理解不同方法的用例并相应地进行集成是非常重要的。

## 版权

通过 [MIT](../LICENSE) 放弃版权及相关权利。