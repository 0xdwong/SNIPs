---
snip: 7
title: 添加 Starknet Appchain 信息 RPC 端点
author: Abdelhamid Bakhta <@abdelhamidbakhta>
discussions-to: https://community.starknet.io/t/addition-of-starknet-appchain-info-rpc-endpoint/97884
status: Draft
type: Standards Track
category: Interface
created: 2023-07-27
---

## 简要总结

本提案建议实现一个新的 RPC 端点，以检索与 Starknet appchains 相关的信息。

## 摘要

随着 Starknet appchains 在 Starknet 生态系统中变得越来越重要，方便地检索特定 appchain 的数据已成为一项关键需求。本提案概述了一个新 RPC 端点的设计和理由，该端点可以提供此类信息。该端点将被钱包和其他应用程序利用，以显示有关 appchain 的相关细节并促进与其的交互。

## 动机

Starknet Stack 的引入使得独立于主 Starknet 链的 Starknet appchains 的创建成为可能，并作为应用程序部署和操作的平台。需要注意的是，Starknet appchains 并不完全是 Layer 2 (L2) 链，可能与公共 Starknet 主网不完全兼容。

我们需要明确希望公开关于这些 appchains 的哪种数据，这将使所有 Starknet Stack 工具（如钱包、区块浏览器和索引器）之间的无缝集成成为可能。

向更模块化设计的日益转变需要对 appchain 进行分类并披露相关信息。本提案旨在解决这一需求。总体而言，appchain 可以根据以下标准进行分类：

- 其作为 Starknet Appchain 的身份（与公共 Starknet 主网和测试网相对）
- 其选择的结算层（例如，Starknet 主网、以太坊等）
- 它支持的数据可用性解决方案类型（例如，Starknet 主网、以太坊（calldata / blob 交易）、Celestia、Avail 等）
- 其费用机制，即它是否使用自定义代币或 Starknet 主网代币？是否支持自定义费用市场？

## 规范

### RPC 端点

提议的端点名称：`starknet_appchainInfo`。

OpenRPC 规范：

```json
{
   "name":"starknet_appchainInfo",
   "summary":"Returns information about the Starknet Appchain",
   "params":[],
   "result":{
      "name":"result",
      "description":"Details about the Appchain, if relevant",
      "schema":{
         "$ref":"#/components/schemas/APPCHAIN_INFO"
      }
   }
}
```

`APPCHAIN_INFO` 结果的模式如下：

```json
{
   "APPCHAIN_INFO":{
      "title":"Appchain Information",
      "type":"object",
      "properties":{
         "is_appchain":{
            "title":"Is Appchain",
            "type":"boolean"
         },
         "chain_id":{
            "title":"Chain ID",
            "$ref":"#/components/schemas/CHAIN_ID"
         },
         "fee_token_address":{
            "title":"Fee Token Address",
            "description":"The address of the fee token",
            "$ref":"#/components/schemas/FELT"
         },
         "settlement_layer":{
            "title":"Settlement Layer",
            "description":"Details about the settlement layer",
            "$ref":"#/components/schemas/SETTLEMENT_LAYER"
         },
         "data_availability_mode":{
            "title":"Data Availability Layer",
            "description":"Details about the Data Availability",
            "$ref":"#/components/schemas/DATA_AVAILABILITY"
         }
      }
   }
}
```

`SETTLEMENT_LAYER` 和 `DATA_AVAILABILITY` 的模式将在规范开发过程的后期阶段定义。

## 版权

通过 [MIT](../LICENSE) 放弃版权及相关权利。