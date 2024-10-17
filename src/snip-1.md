---
snip: 1
title: SNIP 目的和指南
author: Martín Triay <martriay@gmail.com>
status: 进行中
type: 元数据
created: 2022-06-03
---

## 什么是 SNIP？

SNIP 代表 StarkNet 改进提案。SNIP 是一份设计文档，向 StarkNet 社区提供信息，或描述 StarkNet 或其流程或环境的新特性。SNIP 应提供特性的简明技术规范和特性的理由。SNIP 作者负责在社区内建立共识并记录不同意见。

## SNIP 理由

我们希望 SNIP 成为提出新特性、收集社区技术意见和记录 StarkNet 设计决策的主要机制。由于 SNIP 作为文本文件保存在版本控制的仓库中，它们的修订历史是特性提案的历史记录。

对于 StarkNet 实现者来说，SNIP 是跟踪其实现进度的便捷方式。理想情况下，每个实现维护者都会列出他们已实现的 SNIP。这将为最终用户提供一种便捷的方式来了解特定实现或库的当前状态。

## SNIP 类型

SNIP 有三种类型：

- **标准跟踪 SNIP** 描述影响大多数或所有 StarkNet 实现的任何更改，例如——网络协议的更改、区块或交易有效性规则的更改、提议的应用标准/惯例，或任何影响使用 StarkNet 的应用程序互操作性的更改或添加。标准跟踪 SNIP 由两部分组成——设计文档和实现。此外，标准跟踪 SNIP 可以细分为以下类别：

  - **核心**：需要共识分叉的改进，以及不一定是共识关键但可能与 [“核心开发”讨论](https://community.starknet.io/) 相关的更改。
  - **网络**：包括对网络协议规范的提议改进。
  - **接口**：包括围绕客户端 [API/RPC](https://github.com/starkware-libs/starknet-specs) 规范和标准的改进，以及某些语言级标准，如方法名称和合约 ABI。标签“接口”与 [接口仓库] 一致，讨论应主要在该仓库中进行，然后再提交 SNIP 到 SNIP 仓库。
  - **SRC**：应用级标准和惯例，包括合约标准，如代币标准 ([SNIP-2](./snip-2.md))、URI 方案、库/包格式和钱包格式。

- **元数据 SNIP** 描述围绕 StarkNet 的过程或提议对某个过程的更改（或事件）。过程 SNIP 类似于标准跟踪 SNIP，但适用于 StarkNet 协议本身以外的领域。它们可能提议一个实现，但不涉及 StarkNet 的代码库；它们通常需要社区共识；与信息性 SNIP 不同，它们不仅仅是建议，用户通常不能自由忽视它们。示例包括程序、指南、决策过程的更改以及用于 StarkNet 开发的工具或环境的更改。任何元数据 SNIP 也被视为过程 SNIP。

- **信息性 SNIP** 描述 StarkNet 设计问题，或向 StarkNet 社区提供一般指南或信息，但不提议新特性。信息性 SNIP 不一定代表 StarkNet 社区共识或建议，因此用户和实现者可以自由忽视信息性 SNIP 或遵循其建议。

强烈建议单个 SNIP 包含一个关键提案或新想法。SNIP 越集中，成功的可能性越大。对一个客户端的更改不需要 SNIP；对多个客户端有影响的更改，或为多个应用定义标准的更改，则需要。

SNIP 必须满足某些最低标准。它必须是对提议增强的清晰和完整的描述。增强必须代表净改进。提议的实现（如适用）必须是可靠的，并且不得过度复杂化协议。

## SNIP 工作流程

### 监督 SNIP

参与该过程的各方包括您，倡导者或 _SNIP 作者_，[_SNIP 编辑_](#snip-editors) 和 [_StarkNet 核心开发者_](https://community.starknet.io/)。

在您开始撰写正式 SNIP 之前，您应该对您的想法进行审查。首先询问 StarkNet 社区，看看这个想法是否是原创，以避免在基于先前研究被拒绝的事情上浪费时间。因此，建议在 [StarkNet 社区论坛](https://community.starknet.io/) 上开启讨论线程。

一旦想法经过审查，您的下一个责任将是通过 SNIP 向审阅者和所有相关方展示该想法，邀请编辑、开发者和社区在上述渠道上提供反馈。您应该尝试评估对您的 SNIP 的兴趣是否与实施所需的工作量以及需要遵循它的各方数量相称。例如，实施核心 SNIP 所需的工作将远大于 SRC，SNIP 需要来自 StarkNet 客户端团队的足够兴趣。负面的社区反馈将被考虑在内，并可能阻止您的 SNIP 进入草案阶段。

### 核心 SNIP

对于核心 SNIP，鉴于它们需要客户端实现才能被视为 **最终**（见下文“SNIP 过程”），您需要提供客户端的实现或说服客户端实现您的 SNIP。

让客户端实现者审查您的 SNIP 的最佳方式是在社区电话中展示它。您可以通过在 [StarkNet 社区论坛](https://community.starknet.io/) 上发布评论链接您的 SNIP 来请求这样做。

AllCoreDevs 电话为客户端实现者提供了三件事。首先，讨论 SNIP 的技术优点。其次，评估其他客户端将实施的内容。第三，协调网络升级的 SNIP 实施。

这些电话通常会导致对应实施哪些 SNIP 的“粗略共识”。这种“粗略共识”基于以下假设：SNIP 不够有争议，不会导致网络分裂，并且它们在技术上是合理的。

:warning: SNIP 过程和 AllCoreDevs 电话并未设计用于解决有争议的非技术问题，但由于缺乏其他解决方式，往往会与这些问题纠缠在一起。这给客户端实现者带来了评估社区情绪的负担，从而妨碍了 SNIP 和 AllCoreDevs 电话的技术协调功能。如果您在监督一个 SNIP，您可以通过确保 [StarkNet 社区论坛](https://community.starknet.io/) 上的 SNIP 线程包含或链接尽可能多的社区讨论，并确保各利益相关者得到充分代表，从而使建立社区共识的过程更容易。

_简而言之，您作为倡导者的角色是使用下面描述的风格和格式撰写 SNIP，在适当的论坛中监督讨论，并围绕该想法建立社区共识。_

### SNIP 过程

以下是所有 SNIP 在所有轨道中的标准化过程：
![SNIP 状态图](../assets/snip-1/SNIP-process-update.jpg)

**想法** - 一个尚未草拟的想法。这不会在 SNIP 存储库中进行跟踪。

**草稿** - SNIP 开发的第一个正式跟踪阶段。当 SNIP 格式正确时，由 SNIP 编辑合并到 SNIP 存储库中。

**审查** - SNIP 作者将 SNIP 标记为准备好并请求同行评审。

**最后呼叫** - 这是 SNIP 在转为 `最终` 之前的最后审查窗口。SNIP 编辑将分配 `最后呼叫` 状态并设置审查结束日期（`last-call-deadline`），通常为 14 天后。

如果此期间导致必要的规范性更改，则会将 SNIP 恢复为 `审查`。

**最终** - 此 SNIP 代表最终标准。最终 SNIP 处于最终状态，仅应更新以纠正勘误和添加非规范性澄清。

**停滞** - 任何在 `草稿`、`审查` 或 `最后呼叫` 状态下，如果在 6 个月或更长时间内不活跃，则会被移至 `停滞`。SNIP 可以由作者或 SNIP 编辑通过将其移回 `草稿` 或其早期状态来复活。如果未被复活，提案可能会永远保持在此状态。

> _SNIP 作者会被通知其 SNIP 状态的任何算法更改_

**撤回** - SNIP 作者已撤回提议的 SNIP。此状态具有最终性，无法使用此 SNIP 编号复活。如果在以后的日期继续追求该想法，则视为新提案。

**持续** - 一种特殊状态，适用于旨在持续更新且不达到最终状态的 SNIP。最显著的包括 SNIP-1。

## 成功的 SNIP 应包含什么？

每个 SNIP 应包含以下部分：

- 前言 - RFC 822 风格的头部，包含有关 SNIP 的元数据，包括 SNIP 编号、简短描述性标题（最多 44 个字符）、描述（最多 140 个字符）和作者详细信息。无论类别如何，标题和描述都不应包含 SNIP 编号。有关详细信息，请参见 [下文](./snip-1.md#snip-header-preamble)。
- 摘要 - 摘要是一个多句（短段落）技术总结。这应该是规范部分的非常简洁且易于人类阅读的版本。有人应该能够仅通过阅读摘要来了解该规范的主要内容。
- 动机（*可选） - 动机部分对于希望更改 StarkNet 协议的 SNIP 至关重要。它应清楚地解释现有协议规范为何不足以解决 SNIP 所解决的问题。缺乏充分动机的 SNIP 提交可能会被直接拒绝。
- 规范 - 技术规范应描述任何新特性的语法和语义。规范应详细到足以允许当前 StarkNet 平台（cpp-ethereum、go-ethereum、parity、ethereumJ、ethereumjs-lib、 [以及其他](https://github.com/ethereum/wiki/wiki/Clients) ）的竞争性、互操作性实现。
- 理由 - 理由通过描述设计动机和为何做出特定设计决策来充实规范。它应描述考虑过的替代设计和相关工作，例如该特性在其他语言中的支持。理由应讨论在 SNIP 讨论中提出的重要异议或关注。
- 向后兼容性 - 所有引入向后不兼容性的 SNIP 必须包含描述这些不兼容性及其严重性的部分。SNIP 必须解释作者打算如何处理这些不兼容性。缺乏充分向后兼容性论述的 SNIP 提交可能会被直接拒绝。
- 测试用例 - 对于影响共识更改的 SNIP，实施的测试用例是强制性的。测试应作为数据内联在 SNIP 中（例如输入/预期输出对），或包含在 `../assets/snip-###/<filename>` 中。
- 参考实现 - 一个可选部分，包含参考/示例实现，供人们使用以帮助理解或实现该规范。
- 安全考虑 - 所有 SNIP 必须包含讨论与提议更改相关的安全影响/考虑的部分。包括可能对安全讨论重要的信息，表面风险，并可在提案的整个生命周期中使用。例如，包括与安全相关的设计决策、关注、重要讨论、特定于实现的指导和陷阱、威胁和风险的概述以及如何应对这些风险。缺少“安全考虑”部分的 SNIP 提交将被拒绝。没有经过审阅者认为充分的安全考虑讨论，SNIP 不能进入“最终”状态。
- 版权豁免 - 所有 SNIP 必须属于公共领域。版权豁免必须链接到许可证文件，并使用以下措辞：`Copyright and related rights waived via [MIT](../LICENSE).`

## SNIP 格式和模板

SNIP 应以 [markdown](https://github.com/adam-p/markdown-here/wiki/Markdown-Cheatsheet) 格式编写。可以遵循 [模板](https://github.com/ethereum/SNIPs/blob/master/snip-template.md)。

## SNIP 头部前言

每个 SNIP 必须以 [RFC 822](https://www.ietf.org/rfc/rfc822.txt) 风格的头部前言开始，前后各有三个连字符（`---`）。该头部也被称为 Jekyll 的 ["front matter"](https://jekyllrb.com/docs/front-matter/)。头部必须按以下顺序出现。

`snip`: _SNIP 编号_（由 SNIP 编辑确定）

`title`: _SNIP 标题是几个词，而不是完整的句子_

`description`: _描述是一个完整的（短）句子_

`author`: _作者或作者的姓名和/或用户名，或姓名和电子邮件的列表。详细信息如下。_

`discussions-to`: _指向官方讨论线程的 URL_

`status`: _草稿、审查、最后呼叫、最终、停滞、撤回、持续_

`last-call-deadline`: _最后呼叫期结束的日期_（可选字段，仅在状态为 `最后呼叫` 时需要）

`type`: _`标准跟踪`、`元` 或 `信息性` 之一_

`category`: _`核心`、`网络`、`接口` 或 `SRC` 之一_（可选字段，仅在 `标准跟踪` SNIP 时需要）

`created`: _SNIP 创建的日期_

`requires`: _SNIP 编号_（可选字段）

`withdrawal-reason`: _解释 SNIP 撤回原因的句子_（可选字段，仅在状态为 `撤回` 时需要）

允许列表的头部必须用逗号分隔元素。

要求日期的头部将始终采用 ISO 8601 格式（yyyy-mm-dd）。

#### `author` 头部

`author` 头部列出 SNIP 的作者/所有者的姓名、电子邮件地址或用户名。希望保持匿名的人可以仅使用用户名，或使用名字和用户名。`author` 头部值的格式必须是：

> Random J. User &lt;address@dom.ain&gt;

或

> Random J. User (@username)

如果包含电子邮件地址或 GitHub 用户名，并且

> Random J. User

如果未给出电子邮件地址。

不可能同时使用电子邮件和 GitHub 用户名。如果需要同时包含两者，可以将其姓名列出两次，一次使用 GitHub 用户名，一次使用电子邮件。
至少一个作者必须使用 GitHub 用户名，以便在变更请求时获得通知并有能力批准或拒绝它们。

#### `discussions-to` 头部

当 SNIP 处于草稿状态时，`discussions-to` 头部将指示讨论该 SNIP 的 URL。

首选的讨论 URL 是 [StarkNet 社区论坛](https://community.starknet.io/) 上的主题。该 URL 不能指向 GitHub 拉取请求、任何短暂的 URL，以及任何可能随着时间被锁定的 URL（即 Reddit 主题）。

#### `type` 头部

`type` 头部指定 SNIP 的类型：标准跟踪、元或信息性。如果跟踪是标准，请包括子类别（核心、网络、接口或 SRC）。

#### `category` 头部

`category` 头部指定 SNIP 的类别。这仅对标准跟踪 SNIP 是必需的。

#### `created` 头部

`created` 头部记录 SNIP 被分配编号的日期。两个头部应采用 yyyy-mm-dd 格式，例如 2001-08-14。

#### `requires` 头部

SNIPs 可能有一个 `requires` 头部，指示该 SNIP 依赖的 SNIP 编号。

## 链接到外部资源

**不应**包含指向外部资源的链接。外部资源可能会意外消失、移动或更改。

## 链接到其他 SNIPs

引用其他 SNIPs 应遵循格式 `SNIP-N`，其中 `N` 是您所引用的 SNIP 编号。每个在 SNIP 中引用的 SNIP **必须**在首次引用时附带相对 markdown 链接，并且在后续引用中 **可以**附带链接。链接 **必须**始终通过相对路径进行，以便链接在此 GitHub 存储库、该存储库的分支、主要 SNIPs 网站、主要 SNIP 网站的镜像等中有效。例如，您可以通过 `[SNIP-1](./snip-1.md)` 链接到此 SNIP。

## 辅助文件

图像、图表和辅助文件应包含在该 SNIP 的 `assets` 文件夹的子目录中，如下所示：`assets/snip-N`（其中 **N** 替换为 SNIP 编号）。在 SNIP 中链接到图像时，请使用相对链接，例如 `../assets/snip-1/image.png`。

## 转移 SNIP 所有权

有时需要将 SNIP 的所有权转移给新的负责人。一般来说，我们希望保留原作者作为转移 SNIP 的共同作者，但这实际上取决于原作者。转移所有权的一个好理由是原作者不再有时间或兴趣更新它或跟进 SNIP 过程，或者已经失联（即无法联系或未回复电子邮件）。转移所有权的一个坏理由是因为您不同意 SNIP 的方向。我们努力围绕 SNIP 建立共识，但如果这不可能，您可以随时提交一个竞争的 SNIP。

如果您有兴趣接管一个 SNIP，请发送一条消息请求接管，地址发送给原作者和 SNIP 编辑。如果原作者未能及时回复电子邮件，SNIP 编辑将做出单方面决定（这样的决定并不是不能被撤销的 :））。

## SNIP 编辑

当前的 SNIP 编辑是：

- Abdelhamid Bakhta (@abdelhamidbakhta)
- Louis Guthman (@GuthL)
- Jag Lotus (@jag-lotus)

荣誉 SNIP 编辑是：

## SNIP 编辑职责

对于每个新提交的 SNIP，编辑会执行以下操作：

- 阅读 SNIP 以检查其是否准备就绪：合理且完整。即使这些想法似乎不太可能达到最终状态，它们也必须在技术上有意义。
- 标题应准确描述内容。
- 检查 SNIP 的语言（拼写、语法、句子结构等）、标记（GitHub 风格的 Markdown）、代码风格

如果 SNIP 不准备好，编辑将其退回给作者进行修订，并附上具体说明。

一旦 SNIP 准备好进入存储库，SNIP 编辑将：

- 分配一个 SNIP 编号（通常是 PR 编号，但决定权在编辑手中）
- 合并相应的 [拉取请求](https://github.com/starknet-io/SNIPs/pulls)
- 向 SNIP 作者发送消息，告知下一步

许多 SNIP 是由具有写入权限的开发人员编写和维护的。SNIP 编辑监控 SNIP 的更改，并纠正我们看到的任何结构、语法、拼写或标记错误。

编辑不会对 SNIP 进行评判。我们仅执行行政和编辑部分。

## 风格指南

### 标题

前言中的 `title` 字段：

- 不应包含“标准”一词或其任何变体；并且
- 不应包含 SNIP 的编号。

### 描述

前言中的 `description` 字段：

- 不应包含“标准”一词或其任何变体；并且
- 不应包含 SNIP 的编号。

### SNIP 编号

引用 SNIP 编号时，应以连字符形式书写 `SNIP-X`，其中 `X` 是 SNIP 分配的编号。

### RFC 2119

鼓励 SNIPs 遵循 [RFC 2119](https://www.ietf.org/rfc/rfc2119.txt) 的术语，并在规范部分开头插入以下内容：

> 本文档中的关键字“MUST”、“MUST NOT”、“REQUIRED”、“SHALL”、“SHALL NOT”、“SHOULD”、“SHOULD NOT”、“RECOMMENDED”、“MAY”和“OPTIONAL”应按 RFC 2119 中的描述进行解释。

## 历史

本文档在很大程度上源自 [Ethereum 的 EIP-1](https://github.com/ethereum/EIPs/blob/master/EIPS/eip-1.md)，而 EIP-1 又源自 [Bitcoin 的 BIP-0001](https://github.com/bitcoin/bips)，由 Amir Taaki 编写，后者又源自 [Python 的 PEP-0001](https://www.python.org/dev/peps/)。在许多地方，文本被简单复制和修改。尽管 PEP-0001 的文本是由 Barry Warsaw、Jeremy Hylton 和 David Goodger 编写的，但他们对其在 StarkNet 改进过程中的使用不负任何责任，也不应被打扰以解答与 StarkNet 或 SNIP 相关的技术问题。请将所有评论直接发送给 SNIP 编辑。

## 版权

通过 [MIT](../LICENSE) 放弃版权及相关权利。