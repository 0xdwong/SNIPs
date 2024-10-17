---
snip: 9
title: 外部执行
authors: Argent Labs <argent.xyz>, AVNU <avnu.fi>, Braavos <braavos.app>
discussions-to: https://community.starknet.io/t/snip-outside-execution/101058
status: 草稿
type: 标准跟踪
category: SRC
created: 2023-10-11
---

## 简单总结

“外部”执行交易（也称为元交易）允许协议代表用户账户提交交易，只要他们拥有相关的签名。

## 动机

从账户合约外部的入口点发起的交易为协议提供了灵活性：

- 延迟订单：虽然已经可以预签常规交易以便后续执行，但这种方法为协议提供了更原子化的控制（例如，匹配两个限价订单），并避免了账户上的 nonce 管理问题。
- 费用补贴：由于交易的发送者支付燃气费用，因此不需要为账户提供任何燃气代币。虽然这不是主要目标，但该提案是一个事实上的解决方案，已经在为 Starknet 设计的支付者和 nonce 抽象的同时有效工作。

## 规范

应用程序可以通过构建一个表示执行的结构体来执行“外部交易”，在用户的钱包中签名，然后将其传递给账户合约上的自定义方法。

### 1. 构建 `OutsideExecution` 结构体

`OutsideExecution` 表示要代表用户账户执行的交易，由另一个合约传入。以下是 Cairo 表示，但需要在链外构建。

```rust
#[derive(Copy, Drop, Serde)]
struct OutsideExecution {
    caller: ContractAddress,
    nonce: felt252,
    execute_after: u64,
    execute_before: u64,
    calls: Span<Call>
}
```

- **caller**：可用于限制哪些调用合约可以发起此执行，尽管可以使用特殊地址 `'ANY_CALLER'` 来允许每个调用者。
- **nonce**：这与账户的常规 nonce 不同，用于防止在执行之间重用签名，只要它是唯一的，就不需要递增。
- **execute_{before,after}**：允许执行的时间戳范围。
- **calls**：账户要执行的常规调用。

### 2. 使用 SNIP-12 类型数据哈希进行签名

#### 2.1. 版本 1

使用 [domain_separator](https://github.com/starknet-io/SNIPs/blob/main/SNIPS/snip-12.md#domain-separator) 参数：
- `name` 设置为 `Account.execute_from_outside`
- `version` 设置为 `1`

并且 `OutsideExecution` 类型定义为：
```rust
  OutsideExecution: [
    { name: "caller", type: "felt" },
    { name: "nonce", type: "felt" },
    { name: "execute_after", type: "felt" },
    { name: "execute_before", type: "felt" },
    { name: "calls_len", type: "felt" },
    { name: "calls", type: "OutsideCall*" },
  ],
  OutsideCall: [
    { name: "to", type: "felt" },
    { name: "selector", type: "felt" },
    { name: "calldata_len", type: "felt" },
    { name: "calldata", type: "felt*" },
  ]
```

`OutsideExecution` 的类型哈希为：
**`H('OutsideExecution(caller:felt,nonce:felt,execute_after:felt,execute_before:felt,calls_len:felt,calls:OutsideCall*)OutsideCall(to:felt,selector:felt,calldata_len:felt,calldata:felt*)')`**

结果为 `0x11ff76fe3f640fa6f3d60bbd94a3b9d47141a2c96f87fdcfbeb2af1d03f7050`

而 `OutsideCall` 的类型哈希为：
**`H('OutsideCall(to:felt,selector:felt,calldata_len:felt,calldata:felt*)')`**

结果为 `0xf00de1fccbb286f9a020ba8821ee936b1deea42a5c485c11ccdc82c8bebb3a`

请参考此实现：[https://github.com/argentlabs/argent-contracts-starknet/blob/main/lib/outsideExecution.ts](https://github.com/argentlabs/argent-contracts-starknet/blob/main/lib/outsideExecution.ts)

#### 2.2. 版本 2

在 [domain_seperator](https://github.com/starknet-io/SNIPs/blob/main/SNIPS/snip-12.md#domain-separator) 中：
- `version` 设置为 `2`

在 `OutsideExecution` 类型定义中：

- **caller**：更改为类型 `ContractAddress`
- **execute_{before,after}**：更改为类型 `u128`
- **calls_len**：**从哈希中移除**
- **calls**：更改为 Corelib 的类型 `Call*`

因此版本 `2` 的类型定义为：
```rust
  OutsideExecution: [
    { name: "Caller", type: "ContractAddress" },
    { name: "Nonce", type: "felt" },
    { name: "Execute After", type: "u128" },
    { name: "Execute Before", type: "u128" },
    { name: "Calls", type: "Call*" },
  ],
  Call: [
    { name: "To", type: "ContractAddress" },
    { name: "Selector", type: "selector" },
    { name: "Calldata", type: "felt*" },
  ]
```
`OutsideExecution` 的类型哈希为：

**`H('"OutsideExecution"("Caller":"ContractAddress","Nonce":"felt","Execute After":"u128","Execute Before":"u128","Calls":"Call*")"Call"("To":"ContractAddress","Selector":"selector","Calldata":"felt*")')`**

结果为 `0x312b56c05a7965066ddbda31c016d8d05afc305071c0ca3cdc2192c3c2f1f0f`

而 `Call` 的类型哈希为：
**`H('"Call"("To":"ContractAddress","Selector":"selector","Calldata":"felt*")')`**

结果为 `0x3635c7f2a7ba93844c0d064e18e487f35ab90f7c39d00f186a781fc3f0c2ca9`

有关 Starknet 上链外签名的更多信息，请参见 [SNIP 12](https://github.com/starknet-io/SNIPs/blob/main/SNIPS/snip-12.md)

### 3. 将结构和签名传递给账户

检查账户是否支持此 SNIP：

```rust
let account = ISRC5Dispatcher { contract_address: acount_address };
let is_supported = account.supports_interface(SRC5_OUTSIDE_EXECUTION_INTERFACE_ID); // see below for actual value
```

调用账户上的 `execute_from_outside` 方法：

```rust
let account = IOutsideExecutionDispatcher { contract_address: acount_address };
// pre-execution logic...
let results = account.execute_from_outside(outside_execution, signature);
// post-execution logic...
```

### 对于账户构建者

#### 版本 1

此版本实现了 [SNIP-12 修订版 0](https://github.com/starknet-io/SNIPs/blob/main/SNIPS/snip-12.md#specification)。要接受带有 [domain_separator](https://github.com/starknet-io/SNIPs/blob/main/SNIPS/snip-12.md#domain-separator) 参数 `version` 设置为 `1` 的外部交易，账户合约必须实现以下接口：

```rust
/// Interface ID: 0x68cfd18b92d1907b8ba3cc324900277f5a3622099431ea85dd8089255e4181
#[derive(Copy, Drop, Serde)]
struct OutsideExecution {
    caller: ContractAddress,
    nonce: felt252,
    execute_after: u64,
    execute_before: u64,
    calls: Span<Call>
}

#[starknet::interface]
trait IOutsideExecution<TContractState> {
    /// This method allows anyone to submit a transaction on behalf of the account as long as they have the relevant signatures.
    /// This method allows reentrancy. A call to `__execute__` or `execute_from_outside` can trigger another nested transaction to `execute_from_outside` thus the implementation MUST verify that the provided `signature` matches the hash of `outside_execution` and that `nonce` was not already used.
    /// # Arguments
    /// * `outside_execution ` - The parameters of the transaction to execute.
    /// * `signature ` - A valid signature on the SNIP-12 message encoding of `outside_execution`.
    fn execute_from_outside(
        ref self: TContractState,
        outside_execution: OutsideExecution,
        signature: Array<felt252>,
    ) -> Array<Span<felt252>>;

    /// Get the status of a given nonce, true if the nonce is available to use
    fn is_valid_outside_execution_nonce(
        self: @TContractState,
        nonce: felt252
    ) -> bool;
}
```

#### 版本 2

此版本实现了 [SNIP-12 修订版 1](https://github.com/starknet-io/SNIPs/blob/main/SNIPS/snip-12.md#specification)。要接受带有 [domain_separator](https://github.com/starknet-io/SNIPs/blob/main/SNIPS/snip-12.md#domain-separator) 参数 `version` 设置为整数 `2`（而不是短字符串 `'2'` 尽管具有此类型），账户合约必须实现以下接口：

```rust
/// Interface ID: 0x1d1144bb2138366ff28d8e9ab57456b1d332ac42196230c3a602003c89872
#[derive(Copy, Drop, Serde)]
struct OutsideExecution {
    caller: ContractAddress,
    nonce: felt252,
    // note that the type here is u64 and not u128 as defined in the type hash definition
    // u64 matches the type of block_timestamp in Corelib's BlockInfo struct
    execute_after: u64,
    execute_before: u64,
    calls: Span<Call>
}

#[starknet::interface]
trait IOutsideExecution_V2<TContractState> {
    /// This method allows anyone to submit a transaction on behalf of the account as long as they have the relevant signatures.
    /// This method allows reentrancy. A call to `__execute__` or `execute_from_outside` can trigger another nested transaction to `execute_from_outside` thus the implementation MUST verify that the provided `signature` matches the hash of `outside_execution` and that `nonce` was not already used.
    /// The implementation should expect version to be set to 2 in the domain separator.
    /// # Arguments
    /// * `outside_execution ` - The parameters of the transaction to execute.
    /// * `signature ` - A valid signature on the SNIP-12 message encoding of `outside_execution`.
    fn execute_from_outside_v2(
        ref self: TContractState,
        outside_execution: OutsideExecution,
        signature: Span<felt252>,
    ) -> Array<Span<felt252>>;

    /// Get the status of a given nonce, true if the nonce is available to use
    fn is_valid_outside_execution_nonce(
        self: @TContractState,
        nonce: felt252
    ) -> bool;
}
```

**注意**：版本 2 的接口 ID 是使用与 Cairo v2.5.0 兼容的 `Call` 结构计算的，其中 `calldata` 是 `Span<felt252>`

指示性实现大纲：

```rust
fn execute_from_outside(ref self: ContractState, outside_execution: OutsideExecution, signature: Array<felt252>) -> Array<Span<felt252>> {
    // 1. Checks
    if outside_execution.caller.into() != 'ANY_CALLER' {
        assert(get_caller_address() == outside_execution.caller, 'argent/invalid-caller');
    }

    let block_timestamp = get_block_timestamp();
    assert(
        outside_execution.execute_after < block_timestamp && block_timestamp < outside_execution.execute_before,
        'argent/invalid-timestamp'
    );
    let nonce = outside_execution.nonce;
    assert(!self.outside_nonces.read(nonce), 'argent/duplicated-outside-nonce');

    let outside_tx_hash = hash_outside_execution_message(@outside_execution);

    let calls = outside_execution.calls;

    self.assert_valid_calls_and_signature(calls, outside_tx_hash, signature.span(), is_from_outside: true);

    // 2. Effects
    self.outside_nonces.write(nonce, true);

    // 3. Interactions
    let retdata = execute_multicall(calls);

    self.emit(TransactionExecuted { hash: outside_tx_hash, response: retdata.span() });
    retdata
}
```

### 版权

版权及相关权利通过 [MIT](../LICENSE) 放弃。