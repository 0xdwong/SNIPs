---
snip: 14
title: 非同质化代币绑定账户
author: Darlington Nnam <darlingtonnnam@gmail.com>, Ademola Kelvin <adegbiteademola1999@gmail.com>
discussions-to: https://community.starknet.io/t/snip-72-non-fungible-tokenbound-accounts/112479
status: Draft
type: Standards Track
category: SRC
created: 2024-01-08
---

## 简要总结
一个由非同质化代币拥有的智能合约账户的接口和注册表

## 摘要

本提案定义了一个系统，将 Starknet 账户分配给所有非同质化代币（ERC-721）。这些代币绑定账户允许 NFT 拥有资产并与应用程序交互，而无需对现有智能合约或基础设施进行更改。

## 动机

ERC-721 标准促成了非同质化代币应用的爆炸性增长。一些显著的用例包括可繁殖的猫、生成艺术作品和交易流动性头寸。

然而，NFT 无法作为代理或与其他链上资产关联。这一限制使得许多现实世界的非同质化资产难以表示为 NFT。

例如：
- 在角色扮演游戏中，角色根据其采取的行动随着时间积累资产和能力。
- 由许多同质化和非同质化组件组成的汽车。
- 由多个同质化资产组成的投资组合。
- 一张打卡通行证会员卡，授予进入某个场所的权限并记录过去的互动历史。

本提案旨在利用原生账户抽象的力量，使每个 NFT 拥有与 Starknet 用户相同的权利。这包括自我保管资产、执行任意操作和控制多个独立账户的能力。通过这样做，本提案允许复杂的现实世界资产以与 Starknet 现有所有权模型相似的通用模式表示为 NFT。

这通过定义一个单例注册表来实现，该注册表为所有现有和未来的 NFT 分配唯一的、确定性的智能合约账户地址。每个账户永久绑定到单个 NFT，账户的控制权授予该 NFT 的持有者。

本提案中定义的模式不需要对现有 NFT 智能合约进行任何更改。它还与几乎所有支持 Starknet 账户的现有基础设施兼容，从链上协议到链下索引器。代币绑定账户与每个现有的链上资产标准兼容，并可以扩展以支持未来创建的新资产标准。

通过赋予每个 NFT 完整的 Starknet 账户功能，本提案为现有和未来的 NFT 启用了许多新颖的用例。

## 规范

本文档中的关键词“MUST”、“MUST NOT”、“REQUIRED”、“SHALL”、“SHALL NOT”、“SHOULD”、“SHOULD NOT”、“RECOMMENDED”、“MAY”和“OPTIONAL”应按 RFC 2119 中的描述进行解释。

由于 ERC-721 在 Starknet 生态系统中的普遍性，因此使用 ERC-721 而不是 SNIP-3。

## 概述
本提案中概述的系统有两个主要组成部分：

- 代币绑定账户的单例注册表
- 代币绑定账户实现的通用接口

<img src="../assets/snip-14/snip-14.png" width="100%" alt="ERC6551 illustration">

下图说明了 NFT、NFT 持有者、代币绑定账户和注册表之间的关系。

## 注册表
注册表是一个单例合约，作为所有代币绑定账户地址查询的入口点。它有三个功能：

- `create_account` - 为给定实现地址的 NFT 创建代币绑定账户
- `get_account` - 计算给定实现地址的 NFT 的代币绑定账户地址
- `total_deployed_accounts` - 返回某个 NFT 的已部署代币绑定账户的数量

注册表是无权限的、不可变的，并且没有所有者。注册表的完整源代码可以在[注册表实现部分](https://github.com/Starknet-Africa-Edu/TBA/blob/main/src/registry/registry.cairo)找到。

注册表必须使用`deploy_syscall`部署所有代币绑定账户，以确保每个账户地址是确定性的。每个代币绑定账户地址应从其实现地址、代币合约地址、代币 ID 和可选的盐的唯一组合中派生。

注册表必须实现以下接口：

```rust
#[starknet::interface]
trait IRegistry<TContractState> {
    /// @notice Emitted when a new tokenbound account is deployed/created
    /// @param account_address the deployed contract address of the tokenbound account
    /// @param token_contract the contract address of the NFT
    /// @param token_id the ID of the NFT
    #[derive(Drop, starknet::Event)]
    struct AccountCreated {
        account_address: ContractAddress,
        token_contract: ContractAddress,
        token_id: u256,
    }

    /// @notice deploys a new tokenbound account for an NFT
    /// @param implementation_hash the class hash of the reference account
    /// @param token_contract the contract address of the NFT
    /// @param token_id the ID of the NFT
    /// @param salt random salt for deployment
    fn create_account(ref self: TContractState, implementation_hash: felt252, token_contract: ContractAddress, token_id: u256, salt: felt252) -> ContractAddress;

    /// @notice calculates the account address for an existing tokenbound account
    /// @param implementation_hash the class hash of the reference account
    /// @param token_contract the contract address of the NFT
    /// @param token_id the ID of the NFT
    /// @param salt random salt for deployment
    fn get_account(self: @TContractState, implementation_hash: felt252, token_contract: ContractAddress, token_id: u256, salt: felt252) -> ContractAddress;

    /// @notice returns the total no. of deployed tokenbound accounts for an NFT by the registry
    /// @param token_contract the contract address of the NFT 
    /// @param token_id the ID of the NFT
    fn total_deployed_accounts(self: @TContractState, token_contract: ContractAddress, token_id: u256) -> u8;
}
```

## 账户实现
账户实现是一个有主张的、灵活的、经过审计的代币绑定账户实现。

所有代币绑定账户应通过单例注册表创建。

所有代币绑定账户实现必须实现 [SRC-5](https://github.com/starknet-io/SNIPs/blob/main/SNIPS/snip-5.md) 接口检测。

所有代币绑定账户实现必须实现以下接口：

```rust
#[starknet::interface]
trait IAccount<TContractState>{
    /// @notice Emitted exactly once when the account is initialized
    /// @param owner The owner address
    #[derive(Drop, starknet::Event)]
    struct AccountCreated {
        #[key]
        owner: ContractAddress,
    }

    /// @notice Emitted when the account executes a transaction
    /// @param hash The transaction hash
    /// @param response The data returned by the methods called
    #[derive(Drop, starknet::Event)]
    struct TransactionExecuted {
        #[key]
        hash: felt252,
        response: Span<Span<felt252>>
    }

    /// @notice Emitted when the account upgrades to a new implementation
    /// @param tokenContract the contract address of the NFT 
    /// @param tokenId the token ID of the NFT
    /// @param implementation the upgraded account class hash
    #[derive(Drop, starknet::Event)]
    struct Upgraded {
        tokenContract: ContractAddress, 
        tokenId: u256, 
        implementation: ClassHash
    }

    /// @notice used for signature validation
    /// @param hash The message hash 
    /// @param signature The signature to be validated
    fn is_valid_signature(self: @TContractState, hash:felt252, signature: Span<felt252>) -> felt252;

    /// @notice validate an account transaction
    /// @param calls an array of transactions to be executed
    fn __validate__(ref self: TContractState, calls:Array<Call>) -> felt252;

    fn __validate_declare__(self:@TContractState, class_hash:felt252) -> felt252;

    fn __validate_deploy__(self: @TContractState, class_hash:felt252, contract_address_salt:felt252) -> felt252;

    /// @notice executes a transaction
    /// @param calls an array of transactions to be executed
    fn __execute__(ref self: TContractState, calls:Array<Call>) -> Array<Span<felt252>>;

    /// @notice returns the contract address and token ID of the NFT
    fn token(self:@TContractState) -> (ContractAddress, u256);

    /// @notice gets the token bound NFT owner
    /// @param token_contract the contract address of the NFT
    /// @param token_id the token ID of the NFT
    fn owner(self: @TContractState, token_contract:ContractAddress, token_id:u256) -> ContractAddress;

    // @notice protection mechanism for selling token bound accounts. can't execute when account is locked
    // @param duration for which to lock account
    fn lock(ref self: TContractState, duration: u64);
    
    // @notice returns account lock status and time left until account unlocks
    fn is_locked(self: @TContractState) -> (bool, u64);

    // @notice check that account supports TBA interface
    // @param interface_id interface to be checked against
    fn supports_interface(self: @TContractState, interface_id: felt252) -> bool;
}   
```

## 理由
### 单例注册表
本提案指定了一个单一的、规范的注册表。它故意不指定可以由多个注册表合约实现的通用接口。这种方法使几个关键属性得以实现。

### 反事实账户
所有代币绑定账户在创建之前处于反事实状态。因此，代币绑定账户可以在合约创建之前接收资产。单例账户注册表确保所有使用`deploy_syscall`部署的代币绑定账户地址使用相同的寻址方案。

### 无信任部署
一个没有所有者的注册表确保任何代币绑定账户的唯一可信合约是实现。这保证了代币持有者可以访问存储在反事实账户中的所有资产，使用一个可信的实现。

没有规范的注册表，某些代币绑定账户可能会使用一个拥有或可升级的注册表进行部署。这可能导致存储在反事实账户中的资产丢失，并增加支持本提案的应用程序必须考虑的安全模型的范围。

### 单一入口点
一个用于查询账户地址和`AccountCreated`事件的单一入口点简化了在支持本提案的应用程序中索引代币绑定账户的复杂任务。

### 实现多样性
单例注册表允许多样化的账户实现共享一个通用的寻址方案。这为开发者提供了显著的自由度，以创新的方式实现新功能，从而可以轻松支持客户端应用程序。

### 注册表与工厂
选择“注册表”而不是“工厂”一词是为了突出合约的规范性质，并强调查询账户地址（这会定期发生）而不是创建账户（每个账户仅发生一次）的行为。

### 账户歧义
上述规范允许 NFT 拥有多个代币绑定账户。在本提案的开发过程中，考虑了其他架构，这些架构将每个 NFT 分配一个单一的代币绑定账户，使每个代币绑定账户地址成为一个明确的标识符。

然而，这些替代方案存在几个权衡。

首先，由于智能合约的无权限性质，无法强制限制每个 NFT 只有一个代币绑定账户。任何希望为每个 NFT 利用多个代币绑定账户的人都可以通过部署额外的注册表合约来实现。
第二，限制每个 NFT 只能有一个 tokenbound 账户将需要在此提案中包含一个静态、可信的账户实现。该实现不可避免地会对 tokenbound 账户的能力施加特定的限制。考虑到此提案所启用的众多未探索的用例以及多样化账户实现可能为非同质化代币生态系统带来的好处，作者认为在此提案中定义一个规范且受限的实现为时已晚。

最后，此提案旨在赋予 NFT 在链上作为代理的能力。在当前实践中，链上代理通常使用多个账户。一个常见的例子是个人使用一个“热”账户进行日常使用，而使用一个“冷”账户来存储贵重物品。如果链上代理通常使用多个账户，那么 NFT 理应继承相同的能力。

### 向后兼容性
此提案旨在与现有的非同质化代币合约实现最大程度的向后兼容。因此，它并未扩展 ERC-721 标准。

此外，此提案不要求注册表在账户创建之前执行 ERC-165 接口检查以确保与 ERC-721 的兼容性。这最大限度地提高了与 ERC-721 标准之前的非同质化代币合约（如 CryptoKitties）的兼容性。它还允许本提案中描述的系统与半同质化或同质化代币一起使用，尽管这些用例超出了提案的范围。

智能合约作者可以选择在其账户实现中强制执行 ERC-721 的接口检测。

## 参考实现

### 注册表实现
https://github.com/Starknet-Africa-Edu/TBA/blob/main/src/registry/registry.cairo

### 参考账户实现
https://github.com/Starknet-Africa-Edu/TBA/blob/main/src/account/account.cairo

## 安全考虑
### 防止欺诈
为了实现无信任的 tokenbound 账户销售，去中心化市场需要实施防止恶意账户所有者欺诈行为的保护措施。

考虑以下潜在的骗局：

- 爱丽丝拥有一个 ERC-721 代币 X，该代币拥有 tokenbound 账户 Y
- 爱丽丝向账户 Y 存入 10ETH
- 鲍勃通过去中心化市场提出以 11ETH 购买代币 X，假设他将获得存储在账户 Y 中的 10ETH 以及该代币
- 爱丽丝从 tokenbound 账户中提取 10ETH，并立即接受鲍勃的报价
- 鲍勃收到代币 X，但账户 Y 已为空

为了减轻恶意账户所有者的这种欺诈行为，我们添加了两个方法：`lock` 和 `is_locked`，去中心化市场应当实现这些方法以防止此类骗局在市场层面发生。

### 所有权循环
如果创建了所有权循环，所有在 tokenbound 账户中持有的资产可能会变得无法访问。最简单的例子是将 ERC-721 代币转移到其自己的 tokenbound 账户。如果发生这种情况，ERC-721 代币和存储在 tokenbound 账户中的所有资产将永久无法访问，因为 tokenbound 账户无法执行转移 ERC-721 代币的交易。

所有权循环可以在任何 n>0 的 tokenbound 账户图中引入。由于所需的无限搜索空间，链上防止深度>1 的循环是难以强制执行的，因此超出了本提案的范围。希望采用本提案的应用客户端和账户实现被鼓励实施措施，以限制所有权循环的可能性。

## 历史

此 SNIP 的灵感来源于：

- [EIP-6551](https://eips.ethereum.org/EIPS/eip-6551)

## 版权

通过 [MIT](../LICENSE) 放弃版权及相关权利。