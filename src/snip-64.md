---
snip: 64
title: 可替代和不可替代代币的自定义错误
author: Jack Boyuan Xu <jxu@ethsign.xyz>
discussions-to: $DISCUSSION_LINK
status: Draft
type: Standards Track
category: SRC
created: 2023-12-13
requires: SNIP-2, SNIP-3
---

## 摘要

通过在 Cairo v1 中原生支持返回复杂错误消息，本 SNIP 定义了一组标准的结构化和可解码错误，旨在为 [SNIP-2](snip-2.md) 和 [SNIP-3](snip-3.md) 代币提供最相关和简洁的回退信息。

## 动机

模糊的错误消息是每个开发者的噩梦，而这正是我们目前在代币方面所面临的。例如，如果用户尝试转移超过其余额的 [SNIP-2](snip-2.md) 代币，当前的 OpenZeppelin 实现将返回失败原因`u256_sub Overflow`。对于普通用户甚至开发者来说，这极其模糊且无助。

标准化的错误允许每个人在测试和生产环境中期待更简洁、有用和一致的错误消息。

## 规范

本文档中的关键字“MUST”、“MUST NOT”、“REQUIRED”、“SHALL”、“SHALL NOT”、“SHOULD”、“SHOULD NOT”、“RECOMMENDED”、“MAY”和“OPTIONAL”应按 RFC 2119 中的描述进行解释。

以下错误是通过将所有值转换为`felt252`，将其附加到数组中，并将其传递给`panic`调用生成的。

由于 ERC-20 和 ERC-721 在 Starknet 生态系统中的普遍性，因此使用它们而不是 SNIP-2 和 SNIP-3。

### SNIP-2 / ERC-20 错误

- #### `['ERC20InsufficientBalance', <sender: ContractAddress>, <balance: u256>, <needed: u256>]`

  - 用于转账。
  - 表示发送者的余额不足以进行转账。
  - _`balance` MUST be less than `needed`._

- #### `['ERC20InvalidSender', <sender: ContractAddress>]`

  - 用于转账。
  - 表示一个被禁止的`sender`。
  - _推荐用于禁止从零地址或任何黑名单地址进行的转账。_
  - _MUST NOT be used for approvals_
  - _MUST NOT be used for balance or allowance requirements._

- #### `['ERC20InvalidReceiver', <receiver: ContractAddress>]`

  - 用于转账。
  - 表示一个被禁止的`receiver`。
  - _推荐用于禁止向零地址或任何黑名单地址进行的转账。_
  - _MUST NOT be used for approvals._

- #### `['ERC20InsufficientAllowance', <spender: ContractAddress>, <allowance: u256>, <needed: u256>]`

  - 用于转账。
  - 表示支出者的允许额度不足以进行转账。
  - _`allowance` MUST be less than `needed`._

- #### `['ERC20InvalidApprover', <approver: ContractAddress>]`

  - 用于批准。
  - 表示一个被禁止的`approver`。
  - _推荐用于禁止从零地址或任何黑名单地址进行的批准。_
  - _MUST NOT be used for transfers._

- #### `['ERC20InvalidSpender', <spender: ContractAddress>]`
  - 用于批准。
  - 表示一个被禁止的`spender`。
  - _推荐用于禁止向零地址或任何黑名单地址进行的批准。_
  - _MUST NOT be used for transfers._

### SNIP-3 / ERC-721 错误

- #### `['ERC721InvalidOwner', <owner: ContractAddress>]`

  - 用于余额查询。
  - 表示一个被禁止的`owner`。
  - _推荐用于不应拥有代币的地址，例如零地址或任何黑名单地址。_
  - _MUST NOT be used for transfers._

- #### `['ERC721NonexistentToken', <token_id: u256>]`

  - 表示一个未铸造或不存在的`token_id`。
  - _`token_id` MUST be non-minted or burned._

- #### `['ERC721IncorrectOwner', <sender: ContractAddress>, <token_id: u256>, <owner: ContractAddress>]`

  - 用于转账。
  - 表示提供的`owner`与`token_id`的实际`owner`不匹配。
  - _`sender` MUST NOT be `owner`._
  - _MUST NOT be used for approvals._

- #### `['ERC721InvalidSender', <sender: ContractAddress>]`

  - 用于转账。
  - 表示一个被禁止的`sender`。
  - _推荐用于不应转移代币的地址，例如零地址或任何黑名单地址。_
  - _MUST NOT be used in approvals._
  - _MUST NOT be used for ownership or approval requirements._

- #### `['ERC721InvalidReceiver', <receiver: ContractAddress>]`

  - 用于转账。
  - 表示一个被禁止的`receiver`。
  - _推荐用于不应接收代币的地址，例如零地址或任何黑名单地址。_
  - _MUST NOT be used for approvals._

- #### `['ERC721InsufficientApproval', <operator: ContractAddress>, <token_id: u256>]`

  - 用于转账。
  - _`isApprovedForAll(owner, operator)` MUST be false for the `token_id` owner and `operator`._
  - _`getApproved(token_id)` MUST NOT be `operator`._

- #### `['ERC721InvalidApprover', <approver: ContractAddress>]`

  - 用于批准。
  - 表示一个被禁止的`approver`。
  - _推荐用于不应批准代币的地址，例如零地址或任何黑名单地址。_

- #### `['ERC721InvalidOperator', <operator: ContractAddress>]`
  - 用于批准。
  - 表示一个被禁止的`operator`。
  - _推荐用于不应成为操作员的地址，例如零地址或黑名单地址。_
  - _`operator` MUST NOT be the caller._
  - _MUST NOT be used for transfers._

## 实现

该组件实现可以直接在您的代币中使用。请查看完整的代码库[这里](https://github.com/EthSign/starknet-common-error-standards) ，其中包含最小的 [SNIP-2](snip-2.md)/ERC-20 和 [SNIP-3](snip-3.md)/ERC-721 代币实现。

### SNIP-2 / ERC-20

```cairo
use starknet::ContractAddress;
#[starknet::interface]
trait IERC20Errors<TContractState> {
    fn throw_insufficient_balance(self: @TContractState, sender: ContractAddress, balance: u256, needed: u256);
    fn throw_invalid_sender(self: @TContractState, sender: ContractAddress);
    fn throw_invalid_receiver(self: @TContractState, receiver: ContractAddress);
    fn throw_insufficient_allowance(self: @TContractState, spender: ContractAddress, allowance: u256, needed: u256);
    fn throw_invalid_approver(self: @TContractState, approver: ContractAddress);
    fn throw_invalid_spender(self: @TContractState, spender: ContractAddress);
}
```

```cairo
#[starknet::component]
mod ERC20ErrorsComponent {
    use starknet::ContractAddress;
    use common_error_standards::components::ierc20_errors::IERC20Errors;

    #[storage]
    struct Storage {}

    #[event]
    #[derive(Drop, starknet::Event)]
    enum Event {}

    #[embeddable_as(ERC20ErrorsImpl)]
    impl ERC20Errors<TContractState, +HasComponent<TContractState>> of IERC20Errors<ComponentState<TContractState>> {
        fn throw_insufficient_balance(self: @ComponentState<TContractState>, sender: ContractAddress, balance: u256, needed: u256) {
            let data: Array<felt252> = array![
                'ERC20InsufficientBalance',
                sender.into(),
                balance.try_into().unwrap(),
                needed.try_into().unwrap()
            ];
            panic(data);
        }

        fn throw_invalid_sender(self: @ComponentState<TContractState>, sender: ContractAddress) {
            let data: Array<felt252> = array![
                'ERC20InvalidSender',
                sender.into(),
            ];
            panic(data);
        }

        fn throw_invalid_receiver(self: @ComponentState<TContractState>, receiver: ContractAddress) {
            let data: Array<felt252> = array![
                'ERC20InvalidReceiver',
                receiver.into(),
            ];
            panic(data);
        }

        fn throw_insufficient_allowance(self: @ComponentState<TContractState>, spender: ContractAddress, allowance: u256, needed: u256) {
            let data: Array<felt252> = array![
                'ERC20InsufficientAllowance',
                spender.into(),
                allowance.try_into().unwrap(),
                needed.try_into().unwrap(),
            ];
            panic(data);
        }

        fn throw_invalid_approver(self: @ComponentState<TContractState>, approver: ContractAddress) {
            let data: Array<felt252> = array![
                'ERC20InvalidApprover',
                approver.into(),
            ];
            panic(data);
        }

        fn throw_invalid_spender(self: @ComponentState<TContractState>, spender: ContractAddress) {
            let data: Array<felt252> = array![
                'ERC20InvalidSpender',
                spender.into(),
            ];
            panic(data);
        }
    }
}
```

### SNIP-3 / ERC-721

```cairo
use starknet::ContractAddress;
#[starknet::interface]
trait IERC721Errors<TContractState> {
    fn throw_invalid_owner(self: @TContractState, owner: ContractAddress);
    fn throw_nonexistent_token(self: @TContractState, token_id: u256);
    fn throw_incorrect_owner(self: @TContractState, sender: ContractAddress, token_id: u256, owner: ContractAddress);
    fn throw_invalid_sender(self: @TContractState, sender: ContractAddress);
    fn throw_invalid_receiver(self: @TContractState, receiver: ContractAddress);
    fn throw_insufficient_approval(self: @TContractState, operator: ContractAddress, token_id: u256);
    fn throw_invalid_approver(self: @TContractState, approver: ContractAddress);
    fn throw_invalid_operator(self: @TContractState, operator: ContractAddress);
}
```

```cairo
#[starknet::component]
mod ERC721ErrorsComponent {
    use starknet::ContractAddress;
    use common_error_standards::components::ierc721_errors::IERC721Errors;

    #[storage]
    struct Storage {}

    #[event]
    #[derive(Drop, starknet::Event)]
    enum Event {}

    #[embeddable_as(ERC721ErrorsImpl)]
    impl ERC721Errors<TContractState, +HasComponent<TContractState>> of IERC721Errors<ComponentState<TContractState>> {
        fn throw_invalid_owner(self: @ComponentState<TContractState>, owner: ContractAddress) {
            let data: Array<felt252> = array![
                'ERC721InvalidOwner',
                owner.into(),
            ];
            panic(data);
        }

        fn throw_nonexistent_token(self: @ComponentState<TContractState>, token_id: u256) {
            let data: Array<felt252> = array![
                'ERC721NonexistentToken',
                token_id.try_into().unwrap(),
            ];
            panic(data);
        }

        fn throw_incorrect_owner(self: @ComponentState<TContractState>, sender: ContractAddress, token_id: u256, owner: ContractAddress) {
            let data: Array<felt252> = array![
                'ERC721IncorrectOwner',
                sender.into(),
                token_id.try_into().unwrap(),
                owner.into(),
            ];
            panic(data);
        }

        fn throw_invalid_sender(self: @ComponentState<TContractState>, sender: ContractAddress) {
            let data: Array<felt252> = array![
                'ERC721InvalidSender',
                sender.into(),
            ];
            panic(data);
        }

        fn throw_invalid_receiver(self: @ComponentState<TContractState>, receiver: ContractAddress) {
            let data: Array<felt252> = array![
                'ERC721InvalidReceiver',
                receiver.into(),
            ];
            panic(data);
        }

        fn throw_insufficient_approval(self: @ComponentState<TContractState>, operator: ContractAddress, token_id: u256) {
            let data: Array<felt252> = array![
                'ERC721InsufficientApprovel',
                operator.into(),
                token_id.try_into().unwrap(),
            ];
            panic(data);
        }

        fn throw_invalid_approver(self: @ComponentState<TContractState>, approver: ContractAddress) {
            let data: Array<felt252> = array![
                'ERC721InvalidApprover',
                approver.into(),
            ];
            panic(data);
        }

        fn throw_invalid_operator(self: @ComponentState<TContractState>, operator: ContractAddress) {
            let data: Array<felt252> = array![
                'ERC721InvalidOperator',
                operator.into(),
            ];
            panic(data);
        }
    }

}
```

## 历史

本 SNIP 的灵感来源于：

- [EIP-6093](https://eips.ethereum.org/EIPS/eip-6093)

## 版权

版权及相关权利通过 [MIT](../LICENSE) 放弃。