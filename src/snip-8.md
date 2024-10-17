---
snip: 8
title: 交易 V3 结构
authors: Evyatar Oster <@evyataro>, Elin Tulchinsky <elin@starkware.co>
discussions-to: https://community.starknet.io/t/transaction-v3-snip/98228
status: 草稿
type: 标准跟踪
category: 核心
created: 2023-08-09
---

## 简单总结

本 SNIP 旨在提出一种新的交易结构，以适应 Starknet 中未来的 API 和协议变更，包括引入费用市场、Volition、Paymaster 和 Nonce 泛化。
该 SNIP 重点关注交易结构的修改和交易哈希的计算。

## 动机

本 SNIP 的动机在于尽量减少 Starknet 中交易结构的破坏性变更。为此，我们建议引入一个新的交易版本，以支持即将到来的 API 和协议变更。具体而言，我们考虑了未来的五个重大变更：

- [费用市场](https://www.starknet.io/en/roadmap/fee-market-for-transactions)：实施一种机制，使用户即使在拥堵期间也能使用网络。

- [Volition](https://community.starknet.io/t/volition-hybrid-data-availability-solution/97387)：引入一种混合状态设计，允许用户选择其首选的数据可用性模式。

- Paymaster（见 [1](https://community.starknet.io/t/starknet-account-abstraction-model-part-1/781), [2](https://community.starknet.io/t/starknet-account-abstraction-model-part-2/839)）：类似于 [EIP-4337](https://github.com/ethereum/EIPs/blob/3fd65b1a782912bfc18cb975c62c55f733c7c96e/EIPS/eip-4337.md)，通过费用抽象丰富协议，使交易发送者以外的实体能够支付交易费用。

- Nonce 泛化：交易结构旨在支持半非序列化抽象，类似于 [EIP-4337](https://eips.ethereum.org/EIPS/eip-4337#semi-abstracted-nonce-support) 中建议的方法。Nonce 字段将保持不变，包含一个单一的 felt（251 位），被解释为 64 位的顺序 nonce，这与当前实现保持一致。此外，剩余的 187 位将可用于任意通道，给予用户选择的自由。默认通道设置为 0，并将在协议完全支持 nonce 泛化之前强制执行。Nonce 在每个通道内只能是顺序的，如果用户需要并行性，可以相应地创建新通道。

- 在第一个 Invoke/Declare 交易中部署账户：根据 EIP-4337 中提出的概念，我们建议在从不存在的合约发送的 Invoke 和 Declare 交易中添加 account_deployment_data 字段，从而消除单独交易以部署账户的需要。

  - account_deployment_data 用于部署包含入口点的账户合约：`__validate__`、`__execute__`，在 Declare 的情况下，还包括 `__validate_declare__`。

通过将这些变更纳入交易结构，我们旨在平滑 Starknet 的演变，同时尽量减少干扰并保持与现有功能的兼容性。

## 规范

### API 变更：

本章将描述三种新的交易结构。

**公共字段：**

1. `version: felt` = 3

2. 与费用相关的字段：

   1. `resource_bounds: Dict[Resource, ResourceBounds]`

      1. `Resource(Enum)` 包含：

         1. L1_GAS = "L1_GAS"
         2. L2_GAS = "L2_GAS"

      2. `ResourceBounds` 包含：

         1. `max_amount: u64` - 执行期间允许使用的资源的最大数量。
         2. `max_price_per_unit: u128` - 用户愿意为资源支付的最高价格。

      3. 将以费用代币的 10^-18 为单位进行指定

   2. `tip: u64`

      1. 优先级指标决定 mempool 中交易的排序顺序。

3. 与 Volition 相关的字段：

   1. `nonce_data_availability_mode: u32`

      1. 默认 - L1DA

   2. `fee_data_availability_mode: u32`

      1. 默认 - L1DA

DA_mode 0 是 L1DA，DA_mode 1 是 L2DA。

4. 与 Paymaster 相关的字段：

   1. `paymaster_data: List[felt]`

      1. 默认值为空列表，表示没有 Paymaster
      2. 表示赞助交易的 Paymaster 地址，后跟发送给 Paymaster 的额外数据（自赞助交易为空）。

5. `nonce: felt`

6. `signature: List[felt]`

   1. 发送方提供的附加信息，用于验证交易。

**Invoke 特定字段：**

1. `sender_address: felt`

   1. 发起交易的账户地址。

2. `calldata: List[felt]`

   1. 传递给 `__validate__` 和 `__execute__` 函数的参数。

3. `account_deployment_data: List[felt]`

   1. 列表将包含 class_hash、salt 和构造函数所需的 calldata。
   2. 将来，我们可能希望使用 Invoke 而不是 deploy_account，和 EIP-4337 一样。在这种情况下，发送方地址不存在 - 排序者将尝试使用 `account_deployment_data` 中指定的 class hash 部署合约。

**Declare 特定字段：**

1. `sender_address: felt`

   1. 发起交易的账户地址。

2. `contract_class: ContractClass`

   1. [类定义](https://docs.starknet.io/documentation/architecture_and_concepts/Contracts/class-hash/#cairo1_class)。

3. `compiled_class_hash: felt`

   1. 编译类的哈希（有关更多信息，请参见 [这里](https://docs.starknet.io/documentation/starknet_versions/upcoming_versions/#what_to_expect)）。

4. `account_deployment_data: List[felt]`

   1. 列表将包含 class_hash 和构造函数所需的 calldata。
   2. 将来，我们可能希望使用 Invoke 而不是 deploy_account，和 EIP-4337 一样。在这种情况下，发送方地址不存在 - 排序者将尝试使用 `account_deployment_data` 中指定的 class hash 部署合约。

**DeployAccount 特定字段：**

1. `contract_address_salt: felt`

   1. 确定 [账户地址](https://docs.starknet.io/documentation/architecture_and_concepts/Contracts/contract-address/) 的随机盐。

2. `constructor_calldata: List[felt]`

   1. 账户构造函数的参数。

3. `class_hash: felt`

   1. 所需账户类的哈希。

### 协议变更：

定义：

`common_tx_fields = [TX_PREFIX, version, address, h(tip, resource_bounds_for_fee), h(paymaster_data), chain_id, nonce, nonce_data_availability_mode || fee_data_availability_mode]`

其中：

- TX_PREFIX 是 {“declare”, “deploy_account”, “invoke”}，相应地。
- `address` 是 Declare 和 Invoke 的 `sender_address` 或 DeployAccount 的 `contract_address`
- `chain_id` 是一个常量值，指定此交易发送的网络。见 [Chain-Id](https://docs.starknet.io/documentation/architecture_and_concepts/Blocks/transactions/#chain-id)。
- `h(tip, resource_bounds_for_fee) = h(tip, (resource||max_amount||max_price_per_unit),(resource||max_amount||max_price_per_unit)...)`，其中资源顺序为 `L1_gas`、`L2_gas`，资源名称最多为 7 个字符。
- `h` 是 [Poseidon 哈希](https://docs.starknet.io/documentation/architecture_and_concepts/Cryptography/hash-functions/#poseidon_hash)
**交易哈希计算：** 

1. 调用交易哈希计算：

`Invoke_v3_tx_hash = h(common_tx_fields, h(account_deployment_data),h(calldata))`

2. 声明交易哈希计算：
   `Declare_v3_tx_hash=h(common_tx_fields, h(account_deployment_data), class_hash, compiled_class_hash)`

   1. 其中：

      1. class_hash 是合约类的哈希。有关哈希计算的详细信息，请参见 [Class Hash](https://docs.starknet.io/documentation/architecture_and_concepts/Contracts/class-hash/#computing_the_cairo_1_class_hash)
      2. compiled_class_hash 是由 Sierra→Casm 编译器生成的 [编译类](https://docs.starknet.io/documentation/starknet_versions/upcoming_versions/#what_to_expect) 的哈希，该编译器目前在 Starknet 中使用

3. DeployAccount 交易哈希计算：
   `Deploy_account_v3_tx_hash = h(common_tx_fields, h(constructor_calldata), class_hash, contract_address_salt)`

**系统调用变更：**

1. `get_execution_info` 系统调用：

   1. 为了支持旧版本和新版本的交易，需要更新结构体 [TxInfo](https://github.com/starkware-libs/cairo/blob/90f813f487c85a20ebb65449ffc62506916504b4/corelib/src/starknet/info.cairo#L24)。所有现有字段将保留以保持向后兼容性。对于 v3 交易，成员 `max_fee` 将始终等于 0，结构体还将包含以下成员：

      1. `List[Resource]`
      2. `tip: u64`
      3. `nonce_data_availability_mode: u32`
      4. `fee_data_availability_mode: u32`
      5. `paymaster_data: List[felt]`
      6. `account_deployment_data: List[felt]`

   2. 其中 Resource 是一个新结构，包含：`resource: str`，`max_price_per_unit: u128`，`max_amount: u64`

### 向后兼容性

Starknet 在过渡阶段将支持旧的交易版本。然而，一旦在 Starknet 上实现这些功能，旧的交易版本将无法访问新功能，如费用市场、选择权和支付主。

## 安全考虑

此 SNIP 在安全性方面没有任何影响。

## 版权

通过 [MIT](../LICENSE) 放弃版权及相关权利。