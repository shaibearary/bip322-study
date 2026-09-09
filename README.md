# Projects with BIP322 support

**Snapshot:** 2026-09-02/03  


## Short conclusion

For the strict requirement—**a project exposes callable RPC commands and those commands support BIP322 message signing or verification**—the clearest current candidate is:

1. **Liana, through an open fork PR:** the PR adds BIP322 signing, verification, and proof-of-funds commands to `lianad`. It is not merged into upstream Liana.

The other projects split the two capabilities:


- **Sparrow:** supports BIP322 in released code, but its documented RPC role is as a client connecting to Bitcoin Core; I did not find a documented public Sparrow wallet-RPC API. No open BIP322 implementation PR was found.
- **Bitcoin Core:** has JSON-RPC, but native BIP322 message-RPC work previously proposed upstream is closed rather than merged.

## Comparison

| Project | RPC/API | BIP322 status | Open, unmerged work | Strict match? |
|---|---|---|---|---|
| **Liana fork** | `lianad` JSON-RPC | Sign, verify, and proof-of-funds commands in the PR | [RubenWaterman/liana PR #1](https://github.com/RubenWaterman/liana/pull/1); upstream [issue #2205](https://github.com/wizardsardine/liana/issues/2205) | **Closest match** |
| **bitcoin-s** | Application server with JSON-RPC | Open PR adds BIP322 utility functions | [PR #3823](https://github.com/bitcoin-s/bitcoin-s/pull/3823) | Partial; PR does not clearly add RPC methods |
| **btcd** | JSON-RPC API/server | BIP322 library work is incomplete/open | [PR #2521](https://github.com/btcsuite/btcd/pull/2521), [draft #2151](https://github.com/btcsuite/btcd/pull/2151), [draft #2152](https://github.com/btcsuite/btcd/pull/2152) | No wallet-level match |
| **Electrum** | Local authenticated JSON-RPC | Legacy message signing, not BIP322 | [No open BIP322 PR found](https://github.com/spesmilo/electrum/pulls?q=is%3Apr+is%3Aopen+BIP322) | **No** |
| **Sparrow** | RPC client for Bitcoin Core; also supports Electrum servers | BIP322 is in released code | [No open BIP322 implementation PR found](https://github.com/sparrowwallet/sparrow/pulls?q=is%3Apr+is%3Aopen+BIP322) | Not a documented RPC-server match |
| **Bitcoin Core** | JSON-RPC | No current native BIP322 message RPC identified | Earlier proposals [#16440](https://github.com/bitcoin/bitcoin/pull/16440) and [#24058](https://github.com/bitcoin/bitcoin/pull/24058) are closed | No |
| **BitBox02** | Hardware-wallet/device API | BIP322 P2TR/PSBT signing work | [Open draft PR #1977](https://github.com/BitBoxSwiss/bitbox02-firmware/pull/1977) | No general RPC |
| **Passport2** | Hardware-wallet/device API | Taproot BIP322 simple-message signing | [Open PR #652](https://github.com/Foundation-Devices/passport2/pull/652) | No general RPC |
| **rust-bitcoin/bip322** | Rust library | PSBT creator/signer/finalizer work | [Open PR #75](https://github.com/rust-bitcoin/bip322/pull/75) | Library only |

## Electrum

Electrum has a local, authenticated JSON-RPC server. Its command layer exposes `signmessage` and `verifymessage`.

- [Electrum command implementation](https://github.com/spesmilo/electrum/blob/master/electrum/commands.py)
- [Electrum message-signing implementation](https://github.com/spesmilo/electrum/blob/master/electrum/bitcoin.py)
- [Electrum local RPC daemon](https://github.com/spesmilo/electrum/blob/master/electrum/daemon.py)

The important compatibility point is that Electrum uses the older `Bitcoin Signed Message` convention with recoverable ECDSA signatures. That is different from BIP322, which represents message signing as a Bitcoin transaction-like proof and covers modern script types such as Taproot.

The current upstream search showed no open Electrum pull request matching BIP322:

- [Electrum open PR search for BIP322](https://github.com/spesmilo/electrum/pulls?q=is%3Apr+is%3Aopen+BIP322)

**Result:** Electrum is useful if legacy message signing plus local RPC is acceptable, but it should not be counted as an RPC+BIP322 project.

## Sparrow

Sparrow supports BIP322 in released software. Its release notes explicitly mention updating the BIP322 implementation to the completed specification, and it supports BIP322-related PSBT/QR/file workflows in the 2.5.x line.

- [Sparrow releases](https://github.com/sparrowwallet/sparrow/releases)
- [Sparrow connection to Bitcoin Core through Core RPC](https://sparrowwallet.com/docs/connect-node.html)

Sparrow’s documented architecture is primarily that of a wallet client: it connects to a Bitcoin Core node through Bitcoin Core’s RPC or connects to an Electrum server. Based on the official documentation checked, I did not find a documented public JSON-RPC interface for controlling Sparrow’s wallet message-signing operations.

There is no open Sparrow BIP322 implementation PR in the current search. One related open PR concerns explaining that multisig signing is unsupported in the sign-message dialog, but it is a UX/limitation change rather than pending BIP322 implementation:

- [Sparrow open PR search for BIP322](https://github.com/sparrowwallet/sparrow/pulls?q=is%3Apr+is%3Aopen+BIP322)
- [Sparrow PR #2009](https://github.com/sparrowwallet/sparrow/pull/2009)

**Result:** Sparrow is the strongest released GUI option for BIP322, but not a confirmed public RPC-server option.

## Liana

Liana is the strongest example of the requested combination in unfinished work.

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

The bitcoin-s server provides JSON-RPC access to wallet, chain, and node functionality. Open [PR #3823](https://github.com/bitcoin-s/bitcoin-s/pull/3823) adds BIP322 utility functions, but the PR is library/utility work; it does not clearly provide ready-made BIP322 RPC commands.

- [bitcoin-s server documentation](https://bitcoin-s.org/docs/next/applications/server)
- [Open BIP322 utility PR #3823](https://github.com/bitcoin-s/bitcoin-s/pull/3823)

### btcd

btcd provides a JSON-RPC API/server, but the project does not include wallet functionality. Its open BIP322 PRs are lower-level library/specification work:

- [PR #2521](https://github.com/btcsuite/btcd/pull/2521): generic BIP322 message-signing library work;
- [draft PR #2151](https://github.com/btcsuite/btcd/pull/2151): BIP322-related additions;
- [draft PR #2152](https://github.com/btcsuite/btcd/pull/2152): `to_spend`, `to_sign`, and witness encoding pieces, with signing and verification integration still remaining.

- [btcd repository](https://github.com/btcsuite/btcd)

### Hardware wallets and libraries

The following open PRs are relevant for BIP322, but they are device or library integrations rather than RPC servers:

- [BitBox02 PR #1977](https://github.com/BitBoxSwiss/bitbox02-firmware/pull/1977): BIP322 signing for Taproot/PSBT flows.
- [Passport2 PR #652](https://github.com/Foundation-Devices/passport2/pull/652): Taproot BIP322 simple-message signing.
- [rust-bitcoin/bip322 PR #75](https://github.com/rust-bitcoin/bip322/pull/75): PSBT creation/signing/finalization library work.
