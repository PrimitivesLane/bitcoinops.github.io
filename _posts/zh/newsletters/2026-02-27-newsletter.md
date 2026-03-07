---
title: 'Bitcoin Optech Newsletter #394'
permalink: /zh/newsletters/2026/02/27/
name: 2026-02-27-newsletter-zh
slug: 2026-02-27-newsletter-zh
type: newsletter
layout: newsletter
lang: zh
---
本周的周报介绍了一项为输出脚本描述符附加补充信息的 BIP 提案。此外还包含我们的常规栏目：总结 Bitcoin Stack Exchange 上的热门问答、公告新版本和候选版本，以及描述流行比特币基础设施软件的近期变更。

## 新闻

- **<!--draft-bip-for-output-script-descriptor-annotations-->****输出脚本描述符注解的 BIP 草案**：Craig Raw 在 Bitcoin-Dev 邮件列表上[发帖][annot ml]介绍了一个新的 BIP 构想，以回应 BIP392 讨论中出现的反馈（见[周报 #387][news387 sp]）。据 Raw 介绍，诸如以区块高度表示的钱包生日等元数据，可以使[静默支付][topic silent payments]扫描更加高效。然而，元数据在技术上并非确定输出脚本所必需的，因此被认为不适合包含在[描述符][topic descriptors]中。

  Raw 的 BIP 提议以注解的形式提供有用的元数据，表示为键/值对，使用类似 URL 的查询分隔符直接附加到描述符字符串之后。带注解的描述符看起来像这样：`SCRIPT?key=value&key=value#CHECKSUM`。值得注意的是，字符 `?`、`&` 和 `=` 已在 [BIP380][] 中定义，因此校验和算法无需更新。

  在 [BIP 草案][annot draft]中，Raw 还定义了三个初始注解键，专门用于提高静默支付资金扫描效率：

  * 区块高度 `bh`：钱包首次收到资金的区块高度；

  * 间隙限制 `gl`：停止之前要派生的未使用地址数量；

  * 最大标签 `ml`：静默支付钱包要扫描的最大标签索引。

  最后，Raw 指出注解不应用于通用钱包备份流程，而仅用于在不改变描述符生成的脚本的情况下提高资金恢复效率。

## Bitcoin Stack Exchange 精选问答

*[Bitcoin Stack Exchange][bitcoin.se] 是 Optech 贡献者寻找问题答案的首选之地——或者当我们有空闲时间帮助好奇或困惑的用户时也会去那里。在这个月度栏目中，我们重点介绍自上次更新以来一些投票最高的问题和答案。*

{% comment %}<!-- https://bitcoin.stackexchange.com/search?tab=votes&q=created%3a1m..%20is%3aanswer -->{% endcomment %}
{% assign bse = "https://bitcoin.stackexchange.com/a/" %}

- [比特币 BIP324 v2 P2P 传输是否可与随机流量区分？]({{bse}}130500)
  Pieter Wuille 指出 [BIP324][] 的 [v2 加密传输][topic v2 p2p transport]协议支持流量模式整形，尽管目前没有已知软件实现该功能，并总结道"当前的实现仅能抵御涉及发送字节模式的协议签名，而非流量模式"。

- [如果矿工只广播区块头却从不提供区块会怎样？]({{bse}}130456)
  用户 bigjosh 描述了矿工在 P2P 网络上收到区块头但在收到区块内容之前可能的行为：在其上挖空块。Pieter Wuille 澄清说，实际上许多矿工通过监控其他矿池分发给矿工的工作来获知新的区块头，这种技术被称为间谍挖矿（spy mining）。

## 版本发布和候选版本

_热门比特币基础设施项目的新版本发布和候选版本。请考虑升级到新版本或帮助测试候选版本。_

- [Bitcoin Core 28.4rc1][] 是先前主要版本系列的维护版本候选发布。主要包含钱包迁移修复和移除一个不可靠的 DNS 种子。

- [Rust Bitcoin 0.33.0-beta][] 是这个用于处理比特币数据结构的库的 beta 版本。这是一次包含 300 多个提交的大型更新，引入了新的 `bitcoin-consensus-encoding` crate，添加了 P2P 网络消息编码 trait，在解码时拒绝包含重复输入或输出总额超过 `MAX_MONEY` 的交易，并提升了所有子 crate 的主版本号。

## 重大代码和文档变更

_以下是来自 [Bitcoin Core][bitcoin core repo]、[Core Lightning][core lightning repo]、[Eclair][eclair repo]、[LDK][ldk repo]、[LND][lnd repo]、[libsecp256k1][libsecp256k1 repo]、[硬件钱包接口 (HWI)][hwi repo]、[Rust Bitcoin][rust bitcoin repo]、[BTCPay Server][btcpay server repo]、[BDK][bdk repo]、[比特币改进提案 (BIPs)][bips repo]、[Lightning BOLTs][bolts repo]、[Lightning BLIPs][blips repo]、[Bitcoin Inquisition][bitcoin inquisition repo] 和 [BINANAs][binana repo] 的近期重大变更。_

- [Bitcoin Core #34568][] 对 Mining IPC 接口进行了多项破坏性变更（见[周报 #310][news310 mining]）。移除了已弃用的方法 `getCoinbaseRawTx()`、`getCoinbaseCommitment()` 和 `getWitnessCommitmentIndex()`（见[周报 #388][news388 mining]），为 `createNewBlock` 和 `checkBlock` 添加了 `context` 参数，使它们可以在单独的线程上运行而不阻塞 [Cap'n Proto][capn proto] 事件循环，并在 schema 中声明了默认选项值。`Init.makeMining` 的版本号已提升，以便旧客户端收到明确的错误而非静默地误解新 schema。线程变更是下面介绍的冷却功能的前提。

- [Bitcoin Core #34184][] 为 Mining IPC 接口的 `createNewBlock()` 方法添加了可选的冷却期。启用后，该方法在返回区块模板之前始终等待初始区块下载（IBD）完成并等待链尖赶上。这可以防止 [Stratum v2][topic pooled mining] 客户端在启动时被快速过时的模板淹没。还添加了新的 `interrupt()` 方法，以便 IPC 客户端可以干净地中止阻塞的 `createNewBlock()` 或 `waitTipChanged()` 调用。

- [Bitcoin Core #24539][] 添加了新的 `-txospenderindex` 选项，维护一个记录每个已确认输出被哪笔交易花费的索引。启用后，`gettxspendingprevout` RPC 除了现有的交易池查找外，还会返回已确认交易的 `spendingtxid` 和 `blockhash`。还为该 RPC 添加了两个新的可选参数：`mempool_only` 即使索引可用也限制只在交易池中查找，`return_spending_tx` 返回完整的花费交易。该索引不需要 `-txindex`，且与修剪不兼容。这对闪电网络和其他需要追踪花费交易链的二层协议特别有用。

- [Bitcoin Core #34329][] 添加了两个用于管理私密交易广播的新 RPC（见[周报 #388][news388 private]）：`getprivatebroadcastinfo` 返回当前在私密广播队列中的交易信息，包括选择的对等节点地址和每次广播发送的时间；`abortprivatebroadcast` 取消特定交易的广播及其待处理的连接。

- [Bitcoin Core #28792][] 完成了嵌入式 ASMap 系列 PR，将 ASMap 数据直接打包在 Bitcoin Core 二进制文件中，使启用 `-asmap` 的用户不再需要单独获取数据文件。移除构建选项 `WITH_EMBEDDED_ASMAP` 可以排除该数据。ASMap 通过在自治系统间多样化对等节点连接来提高[日蚀攻击][topic eclipse attacks]抵抗力（见周报 [#52][news52 asmap] 和 [#290][news290 asmap]）。该功能默认关闭；用户仍需指定 `-asmap` 来启用。一个新的[文档文件][github asmap-data]概述了获取数据并将其包含在 Bitcoin Core 发布版本中的流程。

- [Bitcoin Core #32138][] 移除了 `settxfee` RPC 和 `-paytxfee` 启动选项，这些选项曾允许用户为所有交易设置静态费率。两者已在 Bitcoin Core 30.0 中弃用（见[周报 #349][news349 settxfee]）。用户应改为依赖[手续费估算][topic fee estimation]或为每笔交易单独设置费率。

- [Bitcoin Core #34512][] 在 `getblock` RPC 响应的详细级别 1 及以上添加了 `coinbase_tx` 字段，包含 coinbase 交易的 `version`、`locktime`、`sequence`、`coinbase` 脚本和 `witness` 数据。输出被有意省略以保持响应紧凑。此前，访问 coinbase 属性需要详细级别 2，这会解码区块中的每笔交易。这对于监控 [BIP54][]（[共识清理][topic consensus cleanup]）coinbase locktime 要求或从 coinbase 脚本识别矿池很有用。

- [Core Lightning #8490][] 添加了新的 `payment-fronting-node` 配置选项，指定一个或多个节点作为接收支付的固定入口点。设置后，[BOLT11][] 发票中的路由提示和 [BOLT12][topic offers] 要约、发票和发票请求中的[盲化路径][topic rv routing]引入点将仅使用指定的前置节点构建。此前，CLN 会自动从节点的通道对等方中选择，可能在不同发票间暴露不同的对等方。该选项可以全局设置或按要约覆盖。

- [Eclair #3250][] 允许 `OpenChannelInterceptor` 在本地节点开启通道时未显式设置 `channel_type` 的情况下自动选择一个。此前，自动通道创建（例如 LSP 向客户端开启通道）如果未提供类型会失败。当前默认偏好[锚点通道][topic anchor outputs]，预计[简单 taproot 通道][topic simple taproot channels]将在后续 PR 中获得优先权。

- [LDK #4373][] 添加了发送[多路径支付][topic multipath payments]的支持，允许本地节点仅支付发票总额的一部分。`RecipientOnionFields` 中的新 `total_mpp_amount_msat` 字段允许声明一个大于本节点发送金额的 MPP 总额，使多个钱包或节点可以通过各自贡献部分支付来协作支付单个发票。接收者从所有贡献节点收集 HTLC，并在全额到达后领取支付。[BOLT12][topic offers] 支持留待后续。

- [BDK #2081][] 为 `SpkTxOutIndex` 和 `KeychainTxOutIndex` 添加了 `spent_txouts()` 和 `created_txouts()` 方法，给定一笔交易，返回它花费了哪些被追踪的输出以及创建了哪些新的被追踪输出。这使钱包可以轻松确定其关注的交易涉及的地址和金额。

{% include snippets/recap-ad.md when="2026-03-03 17:30" %}
{% include references.md %}
{% include linkers/issues.md v=2 issues="34568,34184,24539,34329,28792,32138,34512,8490,3250,4373,2081" %}

[annot ml]: https://groups.google.com/g/bitcoindev/c/ozjr1lF3Rkc
[news387 sp]: /zh/newsletters/2026/01/09/#draft-bip-for-silent-payment-descriptors
[annot draft]: https://github.com/craigraw/bips/blob/descriptorannotations/bip-descriptorannotations.mediawiki
[news310 mining]: /zh/newsletters/2024/07/05/#bitcoin-core-30200
[news388 mining]: /zh/newsletters/2026/01/16/#bitcoin-core-33819
[news388 private]: /zh/newsletters/2026/01/16/#bitcoin-core-29415
[capn proto]: https://capnproto.org/
[news52 asmap]: /zh/newsletters/2019/06/26/#differentiating-peers-based-on-asn-instead-of-address-prefix
[news290 asmap]: /zh/newsletters/2024/02/21/#improved-reproducible-asmap-creation-process-asmap
[github asmap-data]: https://github.com/bitcoin/bitcoin/blob/master/doc/asmap-data.md
[news349 settxfee]: /zh/newsletters/2025/04/04/#bitcoin-core-31278
[Bitcoin Core 28.4rc1]: https://bitcoincore.org/bin/bitcoin-core-28.4/test.rc1/
[Rust Bitcoin 0.33.0-beta]: https://github.com/rust-bitcoin/rust-bitcoin/releases/tag/bitcoin-0.33.0-beta
