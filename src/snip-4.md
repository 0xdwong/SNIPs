---
snip: 4
title: 开罗注释标准
author: Orlando Fraser <orlandothefraser@gmail.com>
status: 草稿
type: 信息性
created: 2022-06-13
---

## 简单总结

一个用于编写开罗合约注释的标准。

灵感来源于 [Natspec](https://docs.soliditylang.org/en/v0.8.14/natspec-format.html) 注释，用于 Solidity。

## 摘要

以下标准提供了一种一致的方式来记录开罗合约及其功能。我建议最初这应该只是信息性的，然而在未来可以集成到开罗编译器中，以允许自动文档生成。

## 动机

注释标准提高了合约的可读性，使确保其正确功能变得更容易。

## 规范

所有标签都是可选的。下表解释了每个标签的目的及其使用场景。

| 标签     | 描述                                     | 上下文                                               |
|---------|-------------------------------------------|-------------------------------------------------------|
| @title  | 开罗文件的描述                           | 头部                                                |
| @author | 作者的姓名和联系方式                     | 头部                                                |
| @notice | 向最终用户解释此功能                     | 头部、存储变量、函数、事件、接口                    |
| @dev    | 向开发者解释任何额外细节                 | 头部、存储变量、函数、事件、接口                    |
| @param  | 文档化一个参数                           | 存储变量、函数、事件                                 |
| @return | 文档化一个返回变量                       | 存储变量、函数                                      |

### 头部

头部注释是可以放置在每个开罗文件顶部的注释块，紧接在导入之后。头部包含有关文件内容的一般信息，包括简要描述和作者。

### 数组

在处理数组时，@param 和 @return 标签应仅应用于指针变量，而不是长度。可以在这个单一标签中记录数组的内容。

### 示例合约
```
# SPDX-License-Identifier: MIT
%lang starknet

from starkware.cairo.common.cairo_builtins import HashBuiltin
from starkware.starknet.common.syscalls import get_caller_address

#
# @title Simple Savings Account
# @author Vitalik Buterin - vitalik.buterin@ethereum.org
# @notice A contract to track users' savings in various pots
#

# @dev Stores the balances of each pot
# @param address: Address of the user
# @param pot_index: Index of the pot
# @return amount: The amount stored
@storage_var
func balance(address : felt, pot_index : felt) -> (amount : felt):
end

# @dev Updates a user's balance in a specified pot
# @param amount: Amount to increase the balance by
# @param pot_index: Index of the pot
@external
func increase_balance{syscall_ptr : felt*, pedersen_ptr : HashBuiltin*, range_check_ptr}(
    amount : felt, pot_index : felt
):
    let (address) = get_caller_address()
    let (current) = balance.read(address, pot_index)
    balance.write(address, pot_index, current + amount)
    return ()
end
```

## 版权

通过 [MIT](../LICENSE) 放弃版权及相关权利。