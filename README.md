# Projects with BIP322 support

**Snapshot:** 2026-09-12 17:20 CST (UTC+08:00)

This snapshot uses [BIP322 version 2.0.0](https://github.com/bitcoin/bips/blob/master/bip-0322.mediawiki), which is **Complete**. Version 1.0.0 (2026-04-15) finalized the format and added the `smp`, `ful`, and `pof` prefixes; version 2.0.0 (2026-06-04) made the message challenge mandatory for proof-of-funds signatures. An open PR is not counted as released support.


## Short conclusion

For the strict requirement—**a project exposes callable RPC commands and those commands support BIP322 message signing or verification**—the clearest current candidate is:

1. **Liana, through an open fork PR:** the PR adds BIP322 v2.0.0 signing, verification, and proof-of-funds commands to `lianad`. It remains open and is not merged into upstream Liana v15.0.

The other projects split the two capabilities:


- **Sparrow 2.5.4:** supports the completed BIP322 format in released code, but its documented RPC role is as a client connecting to Bitcoin Core; I did not find a documented public Sparrow wallet-RPC API. No open BIP322 implementation PR was found.
- **Bitcoin Core 31.1:** has JSON-RPC, but native BIP322 message-RPC work previously proposed upstream is closed rather than merged.

## Comparison

| Project (latest release checked) | RPC/API | BIP322 status | Open, unmerged work at snapshot | Strict match? |
|---|---|---|---|---|
| **Liana v15.0 / fork** | `lianad` JSON-RPC | v2.0.0 sign, verify, and proof-of-funds commands in the fork PR; not in v15.0 | [RubenWaterman/liana PR #1](https://github.com/RubenWaterman/liana/pull/1) open; upstream [issue #2205](https://github.com/wizardsardine/liana/issues/2205) open | **Closest match, unreleased fork only** |
| **bitcoin-s 1.9.12** | Application server with JSON-RPC | Open utility PR predates BIP322 completion; no completed-format or RPC integration shown | [PR #3823](https://github.com/bitcoin-s/bitcoin-s/pull/3823) open, last updated 2025-06-23 | No current strict match |
| **btcd v0.26.2** | JSON-RPC node; no wallet | Current v2-era library/PSBT work is open | [PR #2521](https://github.com/btcsuite/btcd/pull/2521) open; older [#2151](https://github.com/btcsuite/btcd/pull/2151) and draft [#2152](https://github.com/btcsuite/btcd/pull/2152) are superseded draft-era work | No wallet-level match |
| **Electrum 4.8.0** | Local authenticated JSON-RPC | Legacy compatibility case only; no BIP322 simple/full flow | [No open BIP322 PR found](https://github.com/spesmilo/electrum/pulls?q=is%3Apr+is%3Aopen+BIP322) | **No modern-format match** |
| **Sparrow 2.5.4** | RPC client for Bitcoin Core; also supports Electrum servers | Completed BIP322 format released since 2.5.1 | [No open BIP322 implementation PR found](https://github.com/sparrowwallet/sparrow/pulls?q=is%3Apr+is%3Aopen+BIP322); multisig remains requested in [issue #1715](https://github.com/sparrowwallet/sparrow/issues/1715) | Not a documented RPC-server match |
| **Bitcoin Core 31.1** | JSON-RPC | No native BIP322 message RPC identified | Earlier proposals [#16440](https://github.com/bitcoin/bitcoin/pull/16440) and [#24058](https://github.com/bitcoin/bitcoin/pull/24058) remain closed and unmerged | No |
| **BitBox02 firmware v9.26.5** | Hardware-wallet/device API | P2TR and PSBT-based signing is unreleased | [Draft PR #1977](https://github.com/BitBoxSwiss/bitbox02-firmware/pull/1977) open | No general RPC |
| **Passport2 v2.3.11** | Hardware-wallet/device API | P2TR `smp` signing is unreleased | [PR #652](https://github.com/Foundation-Devices/passport2/pull/652) open | No general RPC |
| **rust-bitcoin/bip322 0.0.12** | Rust library | Verification states, timelocks, consensus rules, and signature prefixes released; PSBT roles pending | [PR #75](https://github.com/rust-bitcoin/bip322/pull/75) open | Library only |

## Electrum

Electrum has a local, authenticated JSON-RPC server. Its command layer exposes `signmessage` and `verifymessage`.

- [Electrum command implementation](https://github.com/spesmilo/electrum/blob/master/electrum/commands.py)
- [Electrum message-signing implementation](https://github.com/spesmilo/electrum/blob/master/electrum/bitcoin.py)
- [Electrum local RPC daemon](https://github.com/spesmilo/electrum/blob/master/electrum/daemon.py)

The important compatibility point is that Electrum 4.8.0 still uses the older `Bitcoin Signed Message` convention with recoverable ECDSA signatures for `signmessage` and `verifymessage`. That is different from BIP322's modern simple/full transaction-based formats, which cover script types such as Taproot. BIP322 v2.0.0 also defines a legacy compatibility format, but Electrum does not expose BIP322-prefixed signatures or the modern BIP322 flows.

The current upstream search showed no open Electrum pull request matching BIP322:

- [Electrum open PR search for BIP322](https://github.com/spesmilo/electrum/pulls?q=is%3Apr+is%3Aopen+BIP322)

**Result:** Electrum is useful if legacy message signing plus local RPC is acceptable, but it should not be counted as an RPC+BIP322 project.

## Sparrow

Sparrow supports BIP322 in released software. Version 2.5.1 updated its implementation to the completed specification; 2.5.4 is the latest release checked. The 2.5.x line includes BIP322-related PSBT/QR/file workflows.

- [Sparrow releases](https://github.com/sparrowwallet/sparrow/releases)
- [Sparrow connection to Bitcoin Core through Core RPC](https://sparrowwallet.com/docs/connect-node.html)

Sparrow’s documented architecture is primarily that of a wallet client: it connects to a Bitcoin Core node through Bitcoin Core’s RPC or connects to an Electrum server. Based on the official documentation checked, I did not find a documented public JSON-RPC interface for controlling Sparrow’s wallet message-signing operations.

There is no open Sparrow BIP322 implementation PR in the current search. Multisig BIP322 signing is still requested in open issue #1715. Open PR #2009 only explains the unsupported multisig case in the sign-message dialog; it is a UX/limitation change rather than pending implementation:

- [Sparrow open PR search for BIP322](https://github.com/sparrowwallet/sparrow/pulls?q=is%3Apr+is%3Aopen+BIP322)
- [Sparrow PR #2009](https://github.com/sparrowwallet/sparrow/pull/2009)
- [Sparrow issue #1715](https://github.com/sparrowwallet/sparrow/issues/1715)

**Result:** Sparrow is the strongest released GUI option for BIP322, but not a confirmed public RPC-server option.

## Liana

Liana is the strongest example of the requested combination in unfinished work. Upstream v15.0 does not include it; fork PR #1 remains open and now explicitly targets BIP322 Complete v2.0.0.

The open fork PR describes BIP322 support for:

- signing messages;
- verifying signatures; and
- proof-of-funds messages.

It also proposes RPC commands such as `createsignmessage`, `finalizesignmessage`, and `verifymessage`. The upstream issue confirms that upstream `lianad` did not yet have message-signing commands at the time checked.

- [Open fork PR #1](https://github.com/RubenWaterman/liana/pull/1)
- [Upstream Liana issue #2205](https://github.com/wizardsardine/liana/issues/2205)

**Result:** This is the best candidate if the requirement specifically includes an RPC interface and the user is willing to build from an unmerged PR or maintain a fork.

## Other open PRs

These projects have meaningful unmerged BIP322 work, but do not satisfy the full RPC requirement by themselves:

### bitcoin-s

The bitcoin-s server provides JSON-RPC access to wallet, chain, and node functionality. Open [PR #3823](https://github.com/bitcoin-s/bitcoin-s/pull/3823) adds utility functions only; its three changed files are in the core library and tests, not the server RPC layer. It was last updated in June 2025, before the 2026 completion and prefix changes, so it should not be treated as current BIP322 v2.0.0 support without a rebase and conformance review.

- [bitcoin-s server documentation](https://bitcoin-s.org/docs/next/applications/server)
- [Open BIP322 utility PR #3823](https://github.com/bitcoin-s/bitcoin-s/pull/3823)

### btcd

btcd provides a JSON-RPC API/server, but the project does not include wallet functionality. Its open BIP322 PRs are lower-level library/specification work:

- [PR #2521](https://github.com/btcsuite/btcd/pull/2521): current generic BIP322 library and PSBT work, including the completed-spec test vectors;
- [PR #2151](https://github.com/btcsuite/btcd/pull/2151) and draft [#2152](https://github.com/btcsuite/btcd/pull/2152): older January 2026 draft-era work. PR #2521 says it is based loosely on #2152, so these should be treated as superseded alternatives, not three independent current implementations.

- [btcd repository](https://github.com/btcsuite/btcd)

### Hardware wallets and libraries

The following open PRs are relevant for BIP322, but they are device or library integrations rather than RPC servers:

- [BitBox02 draft PR #1977](https://github.com/BitBoxSwiss/bitbox02-firmware/pull/1977): BIP322 signing for Taproot/PSBT flows; still draft and not in firmware v9.26.5.
- [Passport2 PR #652](https://github.com/Foundation-Devices/passport2/pull/652): completed-spec `smp` signing for single-key Taproot addresses; still open and not in v2.3.11.
- [rust-bitcoin/bip322 PR #75](https://github.com/rust-bitcoin/bip322/pull/75): PSBT creation/signing/finalization roles; still open. Release 0.0.12 already includes signature variant prefixes, verification states, timelocks, and required consensus rules.

## Verification scope

- PR and issue state/draft/merge status were checked through GitHub at the snapshot time.
- Latest published versions were checked from each project's official release page or, for Electrum, its official release notes/download site.
- “No open PR found” is a repository-search result at the snapshot time, not proof that no unpublished work exists.
- Source and PR file lists were inspected to distinguish library, RPC-server, GUI, and device work. No project was built or hardware-tested for this research refresh.
