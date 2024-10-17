---
snip: 19
title: 合约类哈希系统调用
author: Elias Tazartes <@Eikix>
description: 引入一个 Starknet 系统调用，以获取特定地址的类哈希。
discussions-to: https://community.starknet.io/t/snip-19-get-class-hash-at-syscall
status: 草案
type: 标准跟踪
category: 核心
created: 2024-06-12
---

## 简要总结

引入一个新的系统调用 `get_class_hash_at`，旨在获取特定地址的合约类哈希。

## 摘要

Starknet 采用了一种合约类的系统，以减少已部署字节码的重复。其工作方式如下：要部署一段从未部署过的代码，必须首先 _声明_ 它。您实际上是在向 Starknet 网络“声明”一个类。从此时起，可以通过请求排序器在特定地址部署该类来实例化它。简单的用例是：您声明一个可替代代币合约，排序器只需存储一次代码；然后，任何数量的相同智能合约的部署都不会进一步膨胀链的存储。这与以太坊及以太坊虚拟机（EVM）的工作方式不同。在 EVM 世界中，如果您部署一百次 ERC20 合约，链将存储代码的副本一百次。更多信息：<https://docs.starknet.io/architecture-and-concepts/smart-contracts/contract-classes/>

虽然可以通过 [RPC 级别检查已部署合约的类哈希](https://github.com/starkware-libs/starknet-specs/blob/master/api/starknet_api_openrpc.json#L444)（`starknet_getClassHashAt` 方法），但在智能合约级别没有办法执行此操作。在 EVM 世界中，这是通过 `EXTCODEHASH` 操作码实现的。其目的是让合约在链上检查另一个合约是否是特定类的实例。这比 [SNIP-5](https://github.com/starknet-io/SNIPs/blob/main/SNIPS/snip-5.md) 更精确/强大（也更昂贵？），后者受到 ERC-165 的启发。

## 动机

在 Kakarot 中，我们利用 `replace_class_hash` 系统调用进行合约级别的升级，并使用类哈希作为检查合约版本的媒介。我们需要检查合约类哈希的能力。我们目前通过强制每个 Kakarot 账户在存储变量中存储自己的类哈希来实现这一点。

我们将受益于一个系统调用，使合约能够检查特定地址的类哈希。

### Kakarot 的简单用例：启用合约级别的版本控制和我们 Kakarot 账户合约群的自动升级

如何？

1. 在核心 EVM Kakarot 合约中设置新版本（类哈希）
2. 每个合约现在能够将其类哈希与当前正确版本（位于 Kakarot 核心 EVM 中）进行比较，并决定自我升级
3. 在调用合约之前，可以检查它是否处于正确版本。

## 理由

据我们所知，没有多种方法来实现此系统调用。在交易中调用系统调用时，排序器/区块提议者应注入被调用地址的类哈希并提交。StarknetOS - 在证明时 - 可以检查在系统调用调用时提供的类哈希是否与 Starknet 类哈希树或存储一致。

## 规范

可以将系统调用命名为与相应的 RPC 方法相同。

```rust
extern fn get_class_hash_at_syscall(
    contract_address: ContractAddress
) -> SyscallResult<ClassHash> implicits(GasBuiltin, System) nopanic;
```

替代 API：

```rust
extern fn class_hash_syscall(
    contract_address: ContractAddress
) -> SyscallResult<ClassHash> implicits(GasBuiltin, System) nopanic;
```

### 参数

contract_address: 想要知道其类哈希的合约地址。

### 返回

如果合约已部署，则返回 `ClassHash`，如果合约未部署，则返回 `Result::Error`。请注意，系统调用永远不应引发恐慌。在未部署合约的地址上调用 `get_class_hash_at_syscall` 将导致可捕获的错误（相当于返回一个哨兵值 null），而不是 CairoVM 错误。

## 参考实现

我们将需要 Starkware 团队的帮助，以准确定义实现。

## 安全考虑

据我们所知，此 SNIP 没有安全风险。特定地址的类哈希始终是已知的。此外，可能会出现与 `replace_class_hash` 系统调用相关的边缘情况。尽管如此，我们知道 `replace_class_hash` 系统调用将在调用它的交易结束时生效。这确保了调用 `get_class_hash_at` 将始终是一致的。

## 版权

通过 [MIT](../LICENSE) 放弃版权及相关权利。