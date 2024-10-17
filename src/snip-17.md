---
snip: 17
title: 可替代代币的安全转移
author: Jack Boyuan Xu <jxu@ethsign.xyz>
discussions-to: https://community.starknet.io/t/snip-17-safe-transfer-for-fungible-tokens
status: Review
type: Standards Track
category: SRC
created: 2024-06-24
requires: SNIP-2, SNIP-5, SNIP-6, SNIP-64
---

## 摘要

本 SNIP 旨在引入对 [SNIP-2](snip-2.md) 代币转移的接口支持检查，以防止由于发送方将代币转移到空地址或未正确确认入站 [SNIP-2](snip-2.md) 代币转移的合约而导致的潜在代币损失。

## 动机

由于区块链交易的不可逆性和地址格式的反直觉性，用户错误（如地址输入错误）每天导致大量金融资产的损失。[SNIP-3](snip-3.md) 已经认识到这个问题，并内置了安全转移功能，确保接收方确认入站代币转移。没有理由认为这个机制不适用于 [SNIP-2](snip-2.md)。

## 规范

本文档中的关键字“MUST”、“MUST NOT”、“REQUIRED”、“SHALL”、“SHALL NOT”、“SHOULD”、“SHOULD NOT”、“RECOMMENDED”、“MAY”和“OPTIONAL”应按 RFC 2119 中的描述进行解释。

_"ERC-20" 与 "SNIP-2" 可互换使用，因为它在 Starknet 生态系统中广泛存在。_

[SNIP-17](snip-17.md) 接收器接口 ID 必须定义如下：

```cairo
/// Extended function selector as defined in SRC-5
const ISNIP17_RECEIVER_ID: felt252 = selector!("fn_on_erc20_received(ContractAddress,ContractAddress,u256,Span<felt252>)->felt252)");
```

符合 [SNIP-17](snip-17.md) 的 [SNIP-2](snip-2.md)/ERC-20 代币合约必须：

- 实现如下代码片段中的 `safe_transfer_from`。
- 在执行安全转移时，调用接收方的 `on_erc20_received` 或 `onERC20Received`，并将返回值与 [SNIP-17](snip-17.md) 接口 ID 进行比较。如果不匹配，则转移失败。回滚、返回 false 或执行任何其他适当的失败代币转移操作。

```cairo
#[starknet::interface]
trait ISNIP17Contract<TContractState> {
    fn safeTransferFrom(
        ref self: TContractState,
        sender: ContractAddress,
        recipient: ContractAddress,
        amount: u256,
        data: Span<felt252>
    );
    fn safe_transfer_from(
        ref self: TContractState,
        sender: ContractAddress,
        recipient: ContractAddress,
        amount: u256,
        data: Span<felt252>
    );
}
```

符合 [SNIP-17](snip-17.md) 的 [SNIP-2](snip-2.md)/ERC-20 代币接收方必须满足以下条件之一：

- 通过 [SNIP-5](snip-5.md)，自我识别为具有 [SNIP-6](snip-6.md) 接口 ID 的账户。
- 通过 [SNIP-5](snip-5.md)，确认支持 [SNIP-17](snip-17.md) 接口 ID，并实现以下 `ISNIP17Receiver` 特征。

```cairo
#[starknet::interface]
trait ISNIP17Receiver<TContractState> {
    fn on_erc20_received(
        self: @TContractState,
        operator: ContractAddress,
        from: ContractAddress,
        amount: u256,
        data: Span<felt252>
    ) -> felt252;

    fn onERC20Received(
        self: @TContractState,
        operator: ContractAddress,
        from: ContractAddress,
        amount: u256,
        data: Span<felt252>
    ) -> felt252;
}
```

## 理由

本 SNIP 借鉴了 [EIP-4524](https://eips.ethereum.org/EIPS/eip-4524)。虽然也考虑了 [EIP-1363](https://eips.ethereum.org/EIPS/eip-1363)，但最终决定保持与现有安全转移机制的一致性更为重要。

## 向后兼容性

本 SNIP 与现有的 [SNIP-2](snip-2.md)/ERC-20 合约不兼容，因为规范要求通过升级添加新功能和逻辑。

遗留的不可升级接收方与本 SNIP 不兼容，需要使用现有的不安全 `transfer` 函数转移代币。

## 参考实现

以下是基于 OpenZeppelin 的本 SNIP 示例实现的代码片段：

```cairo
fn safe_transfer_from(
    ref self: ContractState,
    sender: ContractAddress,
    recipient: ContractAddress,
    amount: u256,
    data: Span<felt252>,
) {
    self.erc20.transfer_from(sender, recipient, amount);
    if !self._check_on_erc20_received(sender, recipient, amount, data) {
        self._throw_invalid_receiver(recipient);
    }
}

fn _check_on_erc20_received(
    self: @ContractState,
    sender: ContractAddress,
    recipient: ContractAddress,
    amount: u256,
    data: Span<felt252>,
) -> bool {
    let src5_dispatcher = ISRC5Dispatcher { contract_address: recipient };
    if src5_dispatcher.supports_interface(ISNIP17_RECEIVER_ID) {
        ISNIP17ReceiverDispatcher { contract_address: recipient }
            .on_erc20_received(
                get_caller_address(), sender, amount, data
            ) == ISNIP17_RECEIVER_ID
    } else {
        src5_dispatcher.supports_interface(ISRC6_ID)
    }
}

/// SNIP-64 error standard
fn _throw_invalid_receiver(self: @ContractState, receiver: ContractAddress) {
    let data: Array<felt252> = array!['ERC20InvalidReceiver', receiver.into(),];
    panic(data);
}
```

请在 [这里](https://github.com/boyuanx/starknet-erc20-safetransfer) 查看完整的代码库。

## 安全考虑

与调用任何外部合约一样，调用此接收钩子可能会导致重入攻击。如有需要，请使用适当的设计模式和/或重入保护。

## 版权

通过 [MIT](../LICENSE) 放弃版权及相关权利。