---
snip: 3
title: 非同质化代币标准
author: Abdelhamid Bakhta <abdelhamid.bakhta@gmail.com>
status: 草稿
type: 标准跟踪
category: SRC
created: 2022-06-03
---

## 简要总结

非同质化代币的标准接口，也称为契约。

灵感来源于 [EIP-721](https://eips.ethereum.org/EIPS/eip-721)。

## 摘要

请参见 https://eips.ethereum.org/EIPS/eip-721#abstract。

## 动机

请参见 https://eips.ethereum.org/EIPS/eip-721#motivation。

## 规范

### 方法

**注意**：
 - 以下规范使用来自 Cairo `0.8.1`（或更高版本）的语法

### balanceOf

计算分配给所有者的所有 NFT。

分配给零地址的 NFT 被视为无效，并且此函数在查询零地址时会抛出异常。

``` cairo
    func balanceOf(owner: felt) -> (balance: Uint256):
    end
```

### ownerOf

查找 NFT 的所有者。

分配给零地址的 NFT 被视为无效，并且对它们的查询确实会抛出异常。

``` cairo
    func ownerOf(tokenId: Uint256) -> (owner: felt):
    end
```

### safeTransferFrom

将 NFT 的所有权从一个地址转移到另一个地址。

除非 `get_caller_address` 是当前所有者、授权的操作员或该 NFT 的批准地址，否则会抛出异常。

如果 `from_` 不是当前所有者，则会抛出异常。

``` cairo
    func safeTransferFrom(
            from_: felt, 
            to: felt, 
            tokenId: Uint256, 
            data_len: felt,
            data: felt*
        ):
    end
```

### transferFrom

``` cairo
    func transferFrom(from_: felt, to: felt, tokenId: Uint256):
    end
```

### approve

``` cairo
    func approve(approved: felt, tokenId: Uint256):
    end
```

### setApprovalForAll

``` cairo
    func setApprovalForAll(operator: felt, approved: felt):
    end
```

### getApproved

``` cairo
    func getApproved(tokenId: Uint256) -> (approved: felt):
    end
```

### isApprovedForAll

``` cairo
    func isApprovedForAll(owner: felt, operator: felt) -> (isApproved: felt):
    end
```

### 事件

#### Transfer

``` cairo
    @event
    func Transfer(from_: felt, to: felt, tokenId: Uint256):
    end
```

#### Approval

``` cairo
    @event
    func Approval(owner: felt, approved: felt, tokenId: Uint256):
    end
```

#### ApprovalForAll

``` cairo
    @event
    func ApprovalForAll(owner: felt, operator: felt, approved: felt):
    end
```

## 实现

#### 示例实现可在以下位置找到
- [OpenZeppelin 实现](https://github.com/OpenZeppelin/cairo-contracts/tree/main/src/openzeppelin/token/erc721)


## 历史


## 版权

版权及相关权利通过 [MIT](../LICENSE) 放弃。