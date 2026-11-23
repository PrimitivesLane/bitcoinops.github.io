---
title: 'Bitcoin Optech 周报 #432'
permalink: /zh/newsletters/2026/11/20/
name: 2026-11-20-newsletter-zh
slug: 2026-11-20-newsletter-zh
type: newsletter
layout: newsletter
lang: zh
---
本周周报总结了对去中心化矿池奖励支付的分析，介绍了 Eclair 中一个已修复的拒绝服务漏洞的披露，并链接到关于陈旧链尖中继是否应包含区块数据的后续讨论。此外还包括我们的常规栏目：服务和客户端软件的近期变化、新版本和候选版本的公告，以及流行比特币基础设施软件的重大变更。

## 新闻

- **<!--framework-for-decentralized-mining-pool-payouts-->****去中心化矿池奖励支付的分析框架：** 开发者 7d5x9 在 Delving Bitcoin 上[发帖][7d5x9 dcpp]，分析去中心化[矿池][topic pooled mining]如何直接通过 coinbase 交易向矿工支付奖励，帖子将这类协议称为去中心化 coinbase 支付协议（DCPP）。为每个矿工单独设置一个输出无法扩展到很大的规模，因为挖矿固件会限制 coinbase 交易的大小，而且每个输出都会挤占原本可以容纳付费交易的空间。帖子认为，当矿池中的矿工多于输出时，任何按工作量给矿工排序的规则都会允许一种“富豪榜攻击”（rich list attack）：大矿工将算力分配给多个身份，每个身份的排名仍高于最小的矿工，从而将后者挤出 coinbase 交易。

  据 7d5x9 称，早期设计只是转移了这个问题。Braidpool 将问题从谁能获得输出，转移到了谁能控制矿池的门限密钥（见[周报 #340][news340 braidpool]）。如果用单个 coinbase 输出向支持单方退出的[联合资金池（joinpool）][topic joinpools]付款，障碍在于矿工会改变 coinbase 交易来扩展 nonce 空间，因此在找到区块之前，无法确定其 txid，也就无法签署退出交易。7d5x9 认为，在软分叉支持可重新绑定的签名（例如 [BIP448][] 提议的签名，见[周报 #397][news397 bip448]）之前，这个输出仍由单个密钥或一组门限密钥控制。在基于 Spark 风格[状态链][topic statechains]和 [Ark][topic ark] 的初步方案中，攻击者只需在每棵子树中保留一小份资金的领取权，就能阻止低开销的合作结算，增加矿工的开销；不过，帖子认为这对 Ark 的影响较小。

  帖子得出的结论是，联合资金池并不决定哪些输出进入 coinbase 交易。对此，它提出两种选择：按金额加权随机抽取（这个想法归功于 Matt Corallo），或者为指定运营者预留输出。

  曾运行一个公共 P2Pool 节点的 LearnBitcoin.com [检查][lb p2pool]了 2,545 笔 P2Pool coinbase 交易，认为即使没有攻击者，仅规模增长就会挤走小矿工，因为 P2Pool 会提高那些预计每个区块收入不足 0.001 BTC 的矿工的份额难度。对于其后继项目 P2Poolv2，jungly 根据现有固件的限制，[建议][jungly dcpp]最多使用 16 个输出。Steven Roose [回复][roose dcpp]说，他想不出任何不使用[限制条款][topic covenants]而值得探索的方案；限制条款可以让 coinbase 输出确定性地承诺一棵 Ark 交易树，向使用同一 Ark 服务器的每个矿工付款。

- **<!--disclosure-of-a-dos-vulnerability-in-eclair-->****Eclair 的一个 DoS 漏洞披露：** rorp 在 Delving Bitcoin 上[发帖][eclair vuln del]，按[负责任披露][topic responsible disclosures]流程披露了影响 Eclair [0.14.0][eclair 0.14.0] 版本的一个内存耗尽拒绝服务（DoS）漏洞，该漏洞已于 2026 年 7 月在 [0.14.1][eclair 0.14.1] 版本中修复。任何能够连接到脆弱节点的对等节点都能使其崩溃，即使双方没有通道。这个 bug 不会让攻击者直接取得资金，但反复使节点崩溃可能危及其通道资金。仍在运行 0.14.0 的用户应当升级。

  攻击利用了 Eclair 缺少一项大小检查的问题。根据 [BOLT2][]，可以通过 `start_batch` 消息将一批消息作为一次逻辑更新处理；该消息的字段包括 `batch_size`。规范要求，这个值最多为 `20`，并且必须严格大于 `1`。Eclair 检查了上限，却没有执行下限检查。

  攻击者可以先发送一条 `batch_size = 0` 的 `start_batch` 消息，再发送在[拼接][topic splicing]期间会批量发送的 `commitment_signed` 消息，以利用这个漏洞。代码只有在收到的消息数量达到 `batch_size` 时，才会停止向内部批次添加消息；但这个值为 `0`，始终小于已收到的消息数量。因此，列表会不断增长，直到内存耗尽。修复后的代码会按 BOLT2 的规定同时检查下限。

- **<!--continued-discussion-of-stale-tip-relay-->****陈旧链尖中继的后续讨论：** Antoine Poinsot 在 Bitcoin-Dev 邮件列表上关于 [BIP332][] 陈旧链尖中继的讨论中[回复][poinsot staletip]（见周报 [#417][news417 staletip] 和 [#423][news423 staletip]），建议节点最初只中继陈旧区块的区块头。他认为，仅有区块头就足以让人们观测到陈旧区块。如果同时中继区块数据，节点就能更快地重组到基于陈旧区块构建的链上，增加那些在区块竞争中落败后仍短暂继续在自己区块上挖矿的矿工的预期收入。Poinsot [认为][poinsot staletip2]，这种影响很可能很小，但不值得为此进行严密计算。

  Anthony Towns [回复][towns staletip]说，任何能从中获益的矿工都可以低成本地自行实现陈旧区块中继。Towns [估算][towns staletip2]，一个拥有全网 20% 算力的矿池，如果选择在自己的区块上挖矿，而不是在自己先看到的同高度竞争区块上挖矿，就已经能每年多获得约 16.1 BTC；如果将由此产生的单区块重组的中继延迟从五秒降至一秒，这一收益会增加到 16.36 BTC。他还认为，陈旧区块头可能引发对[自私挖矿][topic selfish mining]等行为的怀疑，而只有区块数据才能澄清这些疑虑。Poinsot [同意][poinsot staletip3]，这个估算似乎已经涵盖了对诚实挖矿激励的影响；虽然他仍认为大部分价值来自区块头，但不希望阻止完整区块的中继。

## 服务和客户端软件的变更

*在这个每月栏目中，我们会重点介绍比特币钱包和服务的有趣更新。*

- **<!--liana-v16-0-released-->****Liana v16.0 发布：** [Liana v16.0][liana v16.0] 使用 [BIP138][] 格式写入加密的[描述符][topic descriptors]备份（见[周报 #424][news424 bip138]），并将 [taproot][topic taproot] 设为新钱包的默认选项。

- **<!--coldcard-firmware-adds-codex32-backups-->****COLDCARD 固件增加 codex32 备份：** Coinkite [宣布][coldcard 5.6.3]推出 COLDCARD 固件 5.6.3 和 1.5.3Q 版本，支持生成和导入 [codex32][topic codex32] 秘密值，以及将设备的钱包拆分为二至九份 Shamir 份额，以便日后恢复。这些版本还支持 [PSBTv2][topic psbt] 交易中每个输入独立的高度和时间锁定值。

- **<!--wavelength-released-on-mainnet-->****Wavelength 在主网上发布：** Lightning Labs [宣布][wavelength mainnet]，其类似 [Ark][topic ark] 的支付工具包 Wavelength（见[周报 #415][news415 wavelength]）已在主网上以 alpha 版本提供，每个账户的限额为 300,000 sats。

- **<!--bull-bitcoin-makes-payjoin-the-default-for-exchange-orders-->****Bull Bitcoin 将 payjoin 设为交易所订单的默认选项：** Bull Bitcoin [宣布][bull payjoin blog]，通过 Bull Bitcoin Mobile 钱包在其交易所进行链上买卖时，现在默认使用 [payjoin][topic payjoin]（见[周报 #338][news338 bull]）。钱包的 [6.14.0][bull 6.14.0] 版本还增加了对发送和接收[静默支付][topic silent payments]的实验性支持。

- **<!--breez-sdk-adds-silent-payment-sending-->****Breez SDK 增加静默支付发送功能：** 基于 Spark 的 Breez SDK [0.28.0][breez 0.28.0] 版本增加了向[静默支付][topic silent payments]地址付款的支持。

- **<!--silent-payments-scanner-for-electrum-plugin-->****用于 Electrum 插件的静默支付扫描器：** Ali Sherief [发布][spscand delving]了 spscand，这是一个仅观察的[静默支付][topic silent payments]扫描器，可与[周报 #419][news419 sp electrum]介绍的 Electrum 发送插件配合使用。它可以通过用户自己的 Bitcoin Core 节点、BlindBit 服务器，或结合 P2P 对等节点提供的[致密区块过滤器][topic compact block filters]与 BlindBit 服务器来扫描支付。

- **<!--stratum-v2-releases-and-pool-adoption-->****Stratum v2 版本发布与矿池采用情况：** [Stratum v2][topic pooled mining] 参考实现发布了 [sv2-apps v0.9.0][]，增加了对 Bitcoin Core 32.x 的支持，包括通过其 IPC 接口提交找到的区块，并加强了作业声明客户端（Job Declarator Client）的健壮性（此前介绍见[周报 #384][news384 sv2]）。此外，MARA Foundation [宣布][mara sv2]，MARA 已使用 Stratum v2 挖出首个区块；Foundry 则[宣布][foundry sv2]，已开始在其矿池中测试 Stratum v2。

- **<!--coinswap-renamed-to-openswap-->****Coinswap 更名为 OpenSwap：** [OpenSwap v0.2.3][] 是此前名为 Coinswap 的 [coinswap][topic coinswap] 协议实现更名后的首个版本（见[周报 #415][news415 coinswap]）。它增加了 Electrum 服务器后端选项，并支持向第三方地址付款的交换。如果构建时启用了闪电网络支持，还可以通过 LDK Server 在链上资金和闪电网络之间进行交换（见[周报 #406][news406 ldk server]）。

## 版本和候选版本

_流行比特币基础设施项目的新版本和候选版本。请考虑升级到新版本，或帮助测试候选版本。_

- [Core Lightning 26.12rc1][] 是这一流行闪电网络节点实现下一个主版本的候选版本。这将是 26.06 之后的首个主版本，涵盖两个发布周期。它增加了在通道状态丢失后恢复 [HTLC][topic htlc] 输出的工具（见下文 Core Lightning #9730），改进了支付寻路（见周报 [#418][news418 impressions] 和 [#430][news430 xpay]），并增加了对新合作关闭协议的实验性支持（见[周报 #413][news413 simpleclose]）。发布说明还宣布，`xpay` 将在 27.03 中取代 `pay`。

## 重大的代码和文档变更

_以下是来自 [Bitcoin Core][bitcoin core repo]、[Core Lightning][core lightning repo]、[Eclair][eclair repo]、[LDK][ldk repo]、[LND][lnd repo]、[libsecp256k1][libsecp256k1 repo]、[硬件钱包接口（HWI）][hwi repo]、[Rust Bitcoin][rust bitcoin repo]、[BTCPay Server][btcpay server repo]、[BDK][bdk repo]、[比特币改进提案（BIPs）][bips repo]、[Lightning BOLTs][bolts repo]、[Lightning BLIPs][blips repo]、[Bitcoin Inquisition][bitcoin inquisition repo] 和 [BINANAs][binana repo] 的近期重大变更。_

- [Bitcoin Core #36757][] 修复了通过 [Tor 和 I2P][topic anonymity networks] 发现地址时的一处隐私泄露。此前，节点向恶意的 Tor 或 I2P 对等节点建立出站连接时，对方可以在 `version` 消息中声称，自己看到该节点位于任意一个可路由的 IP 地址。Bitcoin Core 随后可能将该地址连同自身的明网监听端口一起通告回该对等节点。一个有辨识度的端口可能帮助攻击者将这条连接与使用相同端口的明网节点关联起来（较早的 Tor 监听端口修复见[周报 #52][news52 port]）。现在，Bitcoin Core 会忽略 Tor 和 I2P 对等节点所报告的本节点地址。

- [BIPs #2356][] 将 [BIP95][] 中的占位创世参数替换为已挖出的 [testnet5][topic testnet] 创世区块参数，并将 BIP 标记为 `Deployed`。创世区块的 coinbase 包含日期 2026-11-14 和主网区块 975,744 的哈希。对这个哈希的承诺确保了 testnet5 创世区块只能在主网达到该高度之后挖出（见[周报 #431][news431 testnet5]）。

- [Core Lightning #9730][] 增加了 `lightning-hsmtool recoverhtlc` 命令，用于在本地通道状态丢失后手动恢复 [HTLC][topic htlc] 输出。对等节点发布其最新承诺交易后，恢复中的节点已经可以通过[静态通道备份][topic static channel backups]取回自己的主输出（见[周报 #421][news421 recovery]）。提供本地 `hsm_secret`、通道和 HTLC 详情，以及对等节点的 per-commitment point 后，新命令可以识别匹配的 HTLC 输出。该工具会构建已签名的交易，用于在超时后取回出站 HTLC，或用已知原像领取入站 HTLC。不过，运营者必须手动检查并广播这些交易。愿意配合的 CLN 对等节点可以使用 `dumpcommitments --point-only` 命令导出这个公开点，无需泄露秘密值。

- [LDK #5561][] 修复了对等节点在 LDK 发送对应的 `commitment_signed` 消息之前就发送 `revoke_and_ack` 时，可能导致转发资金损失的问题。此前，LDK 可能接受这条过早到达的确认消息，并推进内部承诺编号。这样，它就可能签署一笔与链上监视器记录不同的承诺交易。如果对等节点发布该交易，即使 LDK 已经支付了出站 HTLC，并且知道原像，也可能无法领取入站 HTLC。现在，LDK 会拒绝这条过早的确认消息，并强制关闭通道。这项修复已随 LDK 0.2.8 发布（见[周报 #431][news431 ldk]）。[周报 #426][news426 ldk5057]介绍过较早的一个 bug，它会在重连期间使一笔已签名的承诺交易处于未受监视的状态。

- [LDK #5562][] 和 [#5545][ldk #5545] 修复了恶意对等节点可以在其与 LDK 节点之间的通道中制造的通道 ID 冲突。第一个问题中，对等节点可以将一条已强制关闭通道的已知 ID，复用为新通道的临时 ID。这可能阻止 LDK 将转发支付的原像记录到已关闭通道的监视器中，使出站 HTLC 已支付后，入站 [HTLC][topic htlc] 仍未被领取。第二个问题中，对等节点可以根据注资交易算出某条通道未来的最终 ID，并以此作为临时 ID 提交另一项开通道请求。如果原通道在应用接受新请求之前采用了最终 ID，接受请求的操作就可能覆盖其已注资状态。现在，LDK 会拒绝冲突的 ID，并在接受时再次检查。这两项修复也已随 LDK 0.2.8 发布。Eclair 中可能导致 DoS 的相关通道 ID 冲突见[周报 #427][news427 channelid]。

- [LDK #5557][] 修复了一个漏洞：恶意对等节点可能干扰使用 Electrum 后端的节点对交易的监视。对等节点可以提议开通道，声称其注资输出使用一个已受监视的输出点，却搭配不同的锁定脚本。注册这项提议时，可能覆盖合法监视记录中保存的脚本，即使 LDK 随后拒绝该通道也无济于事。之后，LDK 会向 Electrum 查询与虚假脚本相关的交易，因而可能漏掉花费真实输出的交易，无法及时应对通道关闭。现在，LDK 会保留为同一个输出点注册的所有不同脚本，使虚假注册无法排除合法的监视。这项修复也已随 LDK 0.2.8 发布。

- [LDK #5558][] 和 [#5482][ldk #5482] 修复了两种可能让支付超过手续费预算的情况。第一个 PR 修复了使用静态 [BOLT12][topic offers] 发票进行[异步支付][topic async payments]时，未设置手续费上限的处理方式（见[周报 #321][news321 async]）。现在，如果应用省略这个设置，LDK 会使用默认上限，即支付金额的 1% 加 50 sat。此前，省略设置会完全取消上限，让恶意接收者可以提供手续费过高的[盲化路径][topic rv routing]。这项修复也已随 LDK 0.2.8 发布。第二个 PR 修复了路由为满足 HTLC 最低金额而增加付款金额时的重试处理。此前，LDK 会将增加的金额视为所请求支付的一部分，并在重试时恢复失败路径的手续费预算，使总花费可能超过上限。现在，LDK 只会重试尚欠的金额，并将任何多付金额从剩余手续费预算中扣除。

- [LND #11555][] 改变了 LND 将非 [AMP][topic amp] 发票报告为已结算的方式。虽然应用已经可以监视单个 [HTLC][topic htlc] 的结算何时在链下最终确定（见[周报 #224][news224 finality]），但 LND 此前可能在原像释放时就将发票标记为已结算。现在，释放原像时，发票会进入 `PENDING_SETTLE` 状态。等到支付中的每个 HTLC 都有了最终的链下或链上结果，如果所有部分都已收取，LND 就将发票标记为 `SETTLED`；如果有任何部分失败，则标记为 `CANCELED`。已取消的发票仍可能收到了部分款项，而且这些款项不会退还，因此应用只应将最终成功的 HTLC 金额计入到账金额。发票的 `preimage_released` 标志记录 LND 是否已承诺释放原像。AMP 支付集合仍在原像释放时结算。

- [LND #11551][] 修复了完成[多路径支付][topic multipath payments]和 [AMP][topic amp] 支付时的到期检查。此前，LND 会在每个 HTLC 到达时进行检查。如果接收者等待其余部分期间又挖出了更多区块，LND 可能在较早收到的 HTLC 已不再满足链上领取所需的[到期余量][topic cltv expiry delta]时，仍然释放原像。现在，LND 会在最后一部分到达时，根据当前区块高度重新检查每个部分。如果此前接受的某一部分现在已太接近到期，LND 就会使这组支付失败，并让发票保持待支付状态，以便再次尝试（此前关于取消单个 AMP 集合的工作见[周报 #424][news424 amp]）。

- [LND #11472][] 和 [#10842][lnd #10842] 修复了后端临时错误或单个有问题的输入可能阻止原本有效的扫出操作的问题。第一个 PR 会在交易池接纳检查遇到临时或未分类错误后，保留输入以供重试（见[周报 #373][news373 sweeper]）。它已随 LND 0.21.8-beta 和 0.20.8-beta 发布（见[周报 #431][news431 lnd]）。第二个 PR 使用不会广播的诊断交易，测试被拒绝批次的不同输入子集。如果能识别出无效或有冲突的输入，LND 就会在下一次尝试中排除它，并重试剩余输入。

- [LND #11542][] 修复了通道重连后转发 [HTLC][topic htlc] 的恢复。重放已存储的转发批次时，LND 可能先过滤掉已确认收到的 HTLC，再用剩余条目的新位置查找按原始位置索引的元数据。这可能导致未完成的 HTLC 被跳过或错误关联。此外，如果通道的 HTLC 处理器在 LND 记录批次之后、交接某个 HTLC 之前停止，可能留下一条不完整记录，导致重试被当作重复项丢弃。现在，LND 会保留原始批次索引，并在交接失败后移除不完整记录，让未完成的 HTLC 可以重试。这项修复已随 LND 0.21.8-beta 和 0.20.8-beta 发布（见[周报 #431][news431 lnd]）。

{% include snippets/recap-ad.md when="2026-11-24 17:30" %}
{% include references.md %}
{% include linkers/issues.md v=2 issues="36757,2356,9730,5561,5562,5545,5557,5558,5482,11555,11551,11472,10842,11542" %}
[7d5x9 dcpp]: https://delvingbitcoin.org/t/decentralized-coinbase-payout-protocols-dcpp-a-framework-for-discussion/3060
[news340 braidpool]: /zh/newsletters/2025/02/07/#request-for-a-covenant-design-supporting-braidpool
[news397 bip448]: /zh/newsletters/2026/03/20/#bips-1974
[lb p2pool]: https://delvingbitcoin.org/t/decentralized-coinbase-payout-protocols-dcpp-a-framework-for-discussion/3060/2
[jungly dcpp]: https://delvingbitcoin.org/t/decentralized-coinbase-payout-protocols-dcpp-a-framework-for-discussion/3060/5
[roose dcpp]: https://delvingbitcoin.org/t/decentralized-coinbase-payout-protocols-dcpp-a-framework-for-discussion/3060/4
[eclair vuln del]: https://delvingbitcoin.org/t/disclosure-memory-exhaustion-dos-in-eclair-0-14-0/3057
[eclair 0.14.0]: https://github.com/ACINQ/eclair/releases/tag/v0.14.0
[eclair 0.14.1]: https://github.com/ACINQ/eclair/releases/tag/v0.14.1
[poinsot staletip]: https://groups.google.com/g/bitcoindev/c/AwOPNxF15mU/m/qJIADCfABgAJ
[news417 staletip]: /zh/newsletters/2026/08/07/#draft-bip-for-stale-tip-relay
[news423 staletip]: /zh/newsletters/2026/09/18/#bips-2241
[poinsot staletip2]: https://groups.google.com/g/bitcoindev/c/AwOPNxF15mU/m/ORRffqwZAgAJ
[towns staletip]: https://groups.google.com/g/bitcoindev/c/AwOPNxF15mU/m/NF7UNdJoAQAJ
[towns staletip2]: https://groups.google.com/g/bitcoindev/c/AwOPNxF15mU/m/Kt4orC5bAAAJ
[poinsot staletip3]: https://groups.google.com/g/bitcoindev/c/AwOPNxF15mU/m/NqK6HjiAAQAJ
[liana v16.0]: https://github.com/wizardsardine/liana/releases/tag/v16.0
[news424 bip138]: /zh/newsletters/2026/09/25/#bips-1951
[coldcard 5.6.3]: https://blog.coinkite.com/coldcard-firmware-563-153q/
[wavelength mainnet]: https://lightning.engineering/posts/2026-10-23-wavelength-mainnet-launch/
[news415 wavelength]: /zh/newsletters/2026/07/24/#wavelength-alpha-released
[bull payjoin blog]: https://www.bullbitcoin.com/blog/payjoin-trading
[news338 bull]: /zh/newsletters/2025/01/24/#bull-bitcoin-mobile-wallet-adds-payjoin
[bull 6.14.0]: https://github.com/SatoshiPortal/bullbitcoin-mobile/releases/tag/v6.14.0
[breez 0.28.0]: https://github.com/breez/spark-sdk/releases/tag/0.28.0
[spscand delving]: https://delvingbitcoin.org/t/silent-payments-for-electrum/3078
[news419 sp electrum]: /zh/newsletters/2026/08/21/#silent-payments-sender-plugin-for-electrum
[sv2-apps v0.9.0]: https://github.com/stratum-mining/sv2-apps/releases/tag/v0.9.0
[news384 sv2]: /zh/newsletters/2025/12/12/#stratum-v2-updates
[mara sv2]: https://x.com/MARAFoundation_/status/2120520367500308580
[foundry sv2]: https://x.com/FoundryServices/status/2121279504475709625
[openswap v0.2.3]: https://github.com/citadel-foss/openswap/releases/tag/v0.2.3
[news415 coinswap]: /zh/newsletters/2026/07/24/#coinswap-v0-2-2-released
[news406 ldk server]: /zh/newsletters/2026/05/22/#ldk-server
[LDK #5561]: https://git.rust-bitcoin.org/lightningdevkit/rust-lightning/pulls/5561
[LDK #5562]: https://git.rust-bitcoin.org/lightningdevkit/rust-lightning/pulls/5562
[LDK #5545]: https://git.rust-bitcoin.org/lightningdevkit/rust-lightning/pulls/5545
[LDK #5557]: https://git.rust-bitcoin.org/lightningdevkit/rust-lightning/pulls/5557
[LDK #5558]: https://git.rust-bitcoin.org/lightningdevkit/rust-lightning/pulls/5558
[LDK #5482]: https://git.rust-bitcoin.org/lightningdevkit/rust-lightning/pulls/5482
[news52 port]: /zh/newsletters/2019/06/26/#bitcoin-core-15651
[news431 testnet5]: /en/newsletters/2026/11/13/#testnet5-genesis-block-to-be-mined
[news421 recovery]: /zh/newsletters/2026/09/04/#core-lightning-9435
[news431 ldk]: /en/newsletters/2026/11/13/#ldk-v0-2-8
[news426 ldk5057]: /zh/newsletters/2026/10/09/#ldk-5057
[news427 channelid]: /zh/newsletters/2026/10/16/#disclosure-of-two-dos-vulnerabilities-fixed-in-eclair-v0-14-1
[news321 async]: /zh/newsletters/2024/09/20/#ldk-3140
[news224 finality]: /zh/newsletters/2022/11/02/#lnd-6517
[news424 amp]: /zh/newsletters/2026/09/25/#lnd-11198
[news373 sweeper]: /zh/newsletters/2025/09/26/#lnd-10189
[news431 lnd]: /en/newsletters/2026/11/13/#lnd-v0-21-8-beta
[news418 impressions]: /zh/newsletters/2026/08/14/#core-lightning-9150
[news430 xpay]: /zh/newsletters/2026/11/06/#core-lightning-9592
[news413 simpleclose]: /zh/newsletters/2026/07/10/#core-lightning-9104
[Core Lightning 26.12rc1]: https://github.com/ElementsProject/lightning/releases/tag/v26.12rc1
