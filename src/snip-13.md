---
snip: 13
title: 索引 ERC20 中的 `Transfer` 和 `Approval` 事件
author: Natan Granit <natan@starkware.co>
status: 草案
type: 标准跟踪
created: 2024-05-05
---

## 摘要

Starknet 中的事件由两个 felt 数组组成，`keys` 和 `data`，前者类似于以太坊中的主题。与以太坊类似，Starknet 的 json-rpc 允许通过 `starknet_getEvents` 方法对事件键进行过滤。

在此 SNIP 中，我们建议更新 StarkGate 的 ERC20（包括 ETH、STRK、USDC [和其他](https://github.com/starknet-io/starknet-addresses/blob/master/bridged_tokens/mainnet.json)）以索引 `Transfer` 和 `Approval` 事件中的更多字段，以便允许对发送者或接收者进行过滤。

## 动机

例如，在 Starknet 上运行的交易所等 Dapp 需要跟踪特定地址的转账。目前，Starknet 的 json-rpc 仅允许在特定区块范围内接收给定 ERC20 的所有 `Transfer` 或 `Approval` 事件。此 SNIP 将使得可以过滤这些事件，允许在 `Transfer` 事件中按 `from` 或 `to` 进行过滤，在 `Approval` 事件中按 `owner` 或 `spender` 进行过滤。

在以太坊和其他 EVM 链上已经是这种情况。由于早期 Cairo 迭代的限制，事件只有一个与事件名称对应的键。这导致只能对所有转账事件进行过滤，这远非理想。

## 向后兼容性

**此更改不向后兼容**。所有监听 ERC20 转账和批准事件的 DAPP 都必须调整其事件解码，以便在 `keys` 数组中查找字段，而不是在 `data` 数组中。

## 规范

Starknet 的 json-rpc [`starknet_getEvents` 方法](https://github.com/starkware-libs/starknet-specs/blob/76bdde23c7dae370a3340e40f7ca2ef2520e75b9/api/starknet_api_openrpc.json#L798) 接受一个 `EventFilter` 对象，其中包含一个嵌套的键列表以进行匹配。例如，如果用户发送了一个包含 $\big[[k_1, k_2], [\;], [k_3]\big]$ 的事件过滤器，则节点应返回第一个键为 $k_1$ 或 $k_2$，第三个键为 $k_3$，第二个键不受限制，可以取任何值的事件。此功能由各种 Starknet SDK 支持，例如，参见以下 [starknet.js 教程](https://www.starknetjs.com/docs/guides/events#without-transaction-hash) 以了解如何过滤事件。

目前，所有 StarkGate 的 ERC20 中的 `Transfer` 和 `Approval` 事件如下：

```rust
    /// Emitted when tokens are moved from address `from` to address `to`.
    #[derive(Copy, Drop, PartialEq, starknet::Event)]
    struct Transfer {
        // #[key] - Not indexed, to maintain backward compatibility.
        from: ContractAddress,
        // #[key] - Not indexed, to maintain backward compatibility.
        to: ContractAddress,
        value: u256
    }

    /// Emitted when the allowance of a `spender` for an `owner` is set by a call
    /// to [approve](approve). `value` is the new allowance.
    #[derive(Copy, Drop, PartialEq, starknet::Event)]
    struct Approval {
        // #[key] - Not indexed, to maintain backward compatibility.
        owner: ContractAddress,
        // #[key] - Not indexed, to maintain backward compatibility.
        spender: ContractAddress,
        value: u256
    }
```
此 SNIP 基本上建议取消注释上述 `#[key]` 注释：

```rust
    /// Emitted when tokens are moved from address `from` to address `to`.
    #[derive(Drop, PartialEq, starknet::Event)]
    struct Transfer {
        #[key]
        from: ContractAddress,
        #[key]
        to: ContractAddress,
        value: u256
    }

    /// Emitted when the allowance of a `spender` for an `owner` is set by a call
    /// to `approve`. `value` is the new allowance.
    #[derive(Drop, PartialEq, starknet::Event)]
    struct Approval {
        #[key]
        owner: ContractAddress,
        #[key]
        spender: ContractAddress,
        value: u256
    }
```
也就是说，从 0x1 转账到 0x2 的 100 个代币，现在发出的事件为：

`keys`: [selector(“Transfer”)]

`data`: [0x1, 0x2, 100, 0]

数据数组中的前两个 felt 是 `from` 和 `to` 的值，最后两个 felt 是金额的 u256 的低 128 位和高 128 位。

如果此 SNIP 被接受，发出的事件将更改为：

`keys`: [selector(“Transfer”), 0x1, 0x2]

`data`: [100, 0]

其中 `selector(x)` 是 `x` 的 [sn_keccak](https://docs.starknet.io/documentation/architecture_and_concepts/Cryptography/hash-functions/#starknet_keccak) 。

## 安全考虑

未更改其代码以不同方式解析事件的 Dapps 在 ERC20 合约升级后可能会出现故障。

我们考虑了此 SNIP 中建议的更改是否可能被利用以造成更大损害。我们分析的场景是：未更改其代码的交易所是否可能被误导认为已向其账户进行了转账，从而在交易所上记入一个账户，而实际上并没有发生这样的交易。

我们声称这是不可能的。目前，DAPP 从 `data` 数组的第一个和第二个成员中获取 `from` 和 `to` 值。更改后，`data` 数组的第一个和第二个成员将分别是 `amount_low` 和 `amount_high`。由于 `amount_low` 和 `amount_high` 都被强制为 128 位数字，因此包含 124 个前导零，这些数字无法与 Starknet 上的账户地址发生冲突，后者[必然](https://docs.starknet.io/documentation/architecture_and_concepts/Smart_Contracts/contract-address/)是哈希计算的结果。

## 版权

通过 [MIT](../LICENSE) 放弃版权及相关权利。