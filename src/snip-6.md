---
snip: 6
title: 标准账户接口
authors: Martín Triay <martriay@gmail.com>, Julien Niset <julien@argent.xyz>, Eric Nordelo <eric.nordelo39@gmail.com>, Sergio Garcia <sergio@argent.xyz>, Yoav Gaziel <yoav.gaziel@braavos.app>
discussions-to: https://community.starknet.io/t/snip-starknet-standard-account/95665
status: 审核
type: 标准跟踪
category: SRC
created: 2023-07-06
---

## 简要总结

账户的标准接口。

## 摘要

以下标准定义了在 Starknet 中作为账户的智能合约的标准 API。它提供了通过合约发送交易的基本功能，以及验证签名的功能，以支持账户、协议和去中心化应用之间的互操作性。

## 动机

通过原生账户抽象，Starknet 在账户管理上具有很大的灵活性，而不是在协议层面上确定其行为。不同的用例将继续为生态系统带来不同的账户实现。

拥有标准账户接口支持不同的去中心化应用、协议和标准合约（如代币），这些通常需要与账户进行可预测的交互，或者通过识别它们或期望某些可能未实现的行为来实现。

## 规范

本文档中的关键词 "MUST"、"MUST NOT"、"REQUIRED"、"SHALL"、"SHALL NOT"、"SHOULD"、"SHOULD NOT"、"RECOMMENDED"、"MAY" 和 "OPTIONAL" 应按 [RFC 2119](https://www.ietf.org/rfc/rfc2119.txt) 中的描述进行解释。

### 接口

**每个符合 SNIP-6 的账户 MUST 实现 `SRC6` 和 `SRC5`（来自 [SNIP-5](./snip-5.md)）接口**，并通过 `supports_interface` 发布这两个接口 ID：

```cairo
/// @title Represents a call to a target contract
/// @param to The target contract address
/// @param selector The target function selector
/// @param calldata The serialized function parameters
struct Call {
    to: ContractAddress,
    selector: felt252,
    calldata: Array<felt252>
}

/// @title SRC-6 Standard Account
trait ISRC6 {
    /// @notice Execute a transaction through the account
    /// @param calls The list of calls to execute
    /// @return The list of each call's serialized return value
    fn __execute__(calls: Array<Call>) -> Array<Span<felt252>>;

    /// @notice Assert whether the transaction is valid to be executed
    /// @param calls The list of calls to execute
    /// @return The string 'VALID' is represented as felt when is valid
    fn __validate__(calls: Array<Call>) -> felt252;

    /// @notice Assert whether a given signature for a given hash is valid
    /// @param hash The hash of the data
    /// @param signature The signature to validate
    /// @return The string 'VALID' is represented as felt when the signature is valid
    fn is_valid_signature(hash: felt252, signature: Array<felt252>) -> felt252;
}

/// @title SRC-5 Standard Interface Detection
trait ISRC5 {
    /// @notice Query if a contract implements an interface
    /// @param interface_id The interface identifier, as specified in SRC-5
    /// @return `true` if the contract implements `interface_id`, `false` otherwise
    fn supports_interface(interface_id: felt252) -> bool;
}
```

请注意，如果签名有效，`is_valid_signature` 的返回值 MUST 是短字符串字面量 `VALID`。

## 安全性

为了保证签名不能在其他账户或其他链上重放，哈希的数据必须对账户和链是唯一的。这对于 starknet 交易签名和 SNIP-12 签名是正确的。然而，值得注意的是，这可能不一定适用于其他类型的签名。建议钱包在签署任何数据之前，确保该数据包含账户地址和链 ID。

## 理由

（待办事项...）

## 向后兼容性

目前，多个账户使用 `bool` 作为 `is_valid_signature` 的返回值。虽然我们预计大多数账户将在未来迁移到此标准，但在此期间，我们建议使用此功能的去中心化应用和协议检查 `true` 或 `'VALID'`。

## 版权

版权及相关权利通过 [MIT](../LICENSE) 放弃。