---
title: Ephemeral anchors

## Optional.  Shorter name to use for reference style links e.g., "foo"
## will allow using the link [topic foo][].  Not case sensitive
# shortname: foo

## Optional.  An entry will be added to the topics index for each alias
title-aliases:
  - "Pay-to-Anchor (P2A)"
  - Ephemeral dust

## Required.  At least one category to which this topic belongs.  See
## schema for options
topic-categories:
  - Contract Protocols
  - Fee Management
  - P2P Network Protocol

## Optional.  Produces a Markdown link with either "[title][]" or
## "[title](link)"
primary_sources:
    - title: Ephemeral Anchors
      link: https://gnusha.org/url/https://lists.linuxfoundation.org/pipermail/bitcoin-dev/2022-October/021036.html

    - title: Original implementation
      link: https://github.com/bitcoin/bitcoin/issues/30239

## Optional.  Each entry requires "title" and "url".  May also use "feature:
## true" to bold entry and "date"
optech_mentions:
  - title: Proposed additional rules for v3 transaction relay to assist CPFP fee bumping
    url: /en/newsletters/2022/10/05/#ephemeral-dust

  - title: Ephemeral anchors
    url: /en/newsletters/2022/10/26/#ephemeral-anchors
    feature: true

  - title: Ephemeral anchors implementation
    url: /en/newsletters/2022/12/07/#ephemeral-anchors-implementation

  - title: "Ephemeral anchors compared to `SIGHASH_GROUP`"
    url: /en/newsletters/2023/01/25/#ephemeral-anchors-compared-to-sighash-group

  - title: "Bitcoin Inquisition #23 adds part of the support for ephemeral anchors"
    url: /en/newsletters/2023/04/26/#bitcoin-inquisition-23

  - title: "LN developer discussion about multiple relay policy topics, including ephemeral anchors"
    url: /en/newsletters/2023/07/26/#reliable-transaction-confirmation

  - title: "Making ephemeral anchor spends with non-malleable txids"
    url: /en/newsletters/2023/11/15/#eliminating-malleability-from-ephemeral-anchor-spends

  - title: Discussion about ephemeral anchors for LN and v3 transaction relay proposal
    url: /en/newsletters/2024/01/10/#discussion-about-ln-anchors-and-v3-transaction-relay-proposal

  - title: Discussion about cluster mempool and a need for a CPFP carve out replacement like ephemeral anchors
    url: /en/newsletters/2024/01/17/#cpfp-carve-out-needs-to-be-removed

  - title: "Discussion about Miner Extractable Value (MEV) in non-zero ephemeral anchors"
    url: /en/newsletters/2024/01/17/#discussion-of-miner-extractable-value-mev-in-non-zero-ephemeral-anchors

  - title: Proposed changes to LN for v3 relay and ephemeral anchors
    url: /en/newsletters/2024/01/24/#proposed-changes-to-ln-for-v3-relay-and-ephemeral-anchors

  - title: Description of a replacement cycling attack against transactions using P2A
    url: /en/newsletters/2024/08/09/#replacement-cycle-attack-against-pay-to-anchor

  - title: "Bitcoin Core #30352 introduces a new standard output type, Pay-To-Anchor (P2A)"
    url: /en/newsletters/2024/08/09/#bitcoin-core-30352

  - title: "Bitcoin Core PR Review Club for #30352: Add PayToAnchor (P2A) as standard output script for spending"
    url: /en/newsletters/2024/08/09/#bitcoin-core-pr-review-club

  - title: "Guide for Wallets Employing Bitcoin Core 28.0 Policies: P2A output scripts"
    url: /en/bitcoin-core-28-wallet-integration-guide/
    date: 2024-10-10

  - title: "LN developer discussion of using P2A for version-3 LN commitments"
    url: /en/newsletters/2024/10/18/#version-3-commitment-transactions

  - title: "Bitcoin PR Review Club about ephemeral dust to improve P2A usability"
    url: /en/newsletters/2024/11/08/#bitcoin-core-pr-review-club

  - title: "Bitcoin Core #30239 makes ephemeral dust outputs standard, allowing zero-fee transaction relay"
    url: /en/newsletters/2024/11/22/#bitcoin-core-30239

  - title: "Discussion of tradeoffs in ephemeral anchor scripts related to spending trimmed HTLC value"
    url: /en/newsletters/2025/02/07/#tradeoffs-in-ln-ephemeral-anchor-scripts

  - title: "Rust Bitcoin #4111 adds support for the new P2A standard output type"
    url: /en/newsletters/2025/03/07/#rust-bitcoin-4111

## Optional.  Same format as "primary_sources" above
see_also:
  - title: V3 Transaction Relay
    link: topic v3 transaction relay

  - title: Anchor outputs
    link: topic anchor outputs

  - title: Package relay
    link: topic package relay

  - title: Fee sponsorship
    link: topic fee sponsorship

  - title: CPFP carve out
    link: topic cpfp carve out

## Optional.  Force the display (true) or non-display (false) of stub
## topic notice.  Default is to display if the page.content is below a
## threshold word count
#stub: false

## Required.  Use Markdown formatting.  Only one paragraph.  No links allowed.
## Should be less than 500 characters
excerpt: >
  **Ephemeral anchors** are a proposal to allow some transactions to be
  relayed even if they don't pay any transaction fee, provided they're
  relayed as part of a package containing a child transaction which pays
  a fee sufficient for the entire package.

---
The proposal is built on top of the [v3 transaction relay][topic v3
transaction relay] proposal.  A v3 transaction containing as little as
zero fee that has a zero-value output paying `OP_TRUE` as the entire script
would be accepted when included in a [relay package][topic package
relay] with a fee-paying child.

This allows anyone on the network to use
that output as the input to a child transaction.  This allows anyone to
create the fee-paying child, even if they don't receive any of the other
outputs from the parent transaction.  This allows ephemeral anchors to
function as [fee sponsorship][topic fee sponsorship] but without
requiring any consensus changes.

Ephemeral anchors is envisioned to be used with contract protocols such
as LN where transactions are signed by the contract participants a long
time before they are broadcast, preventing the participants from
determining an appropriate feerate to use.  Instead, any participant (or
several acting together) can use the ephemeral anchor output as the
input to a child transaction which adds fees at the time the transaction
is broadcast.  This is similar to the [anchor outputs][topic anchor
outputs] added to LN in 2021-22, based on the [CPFP carve out][topic
cpfp carve out] relay rule.

{% include references.md %}
{% include linkers/issues.md issues="" %}
