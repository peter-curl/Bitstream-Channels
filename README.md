# Bitstream Channels - Stacks L2 Payment Channel Protocol

[![Clarity Version](https://img.shields.io/badge/clarity-2.0-blue)](https://docs.stacks.co/docs/clarity/)

Enterprise-grade payment channel implementation enabling Bitcoin-aligned off-chain transactions with Stacks L2 settlement. Designed for high-throughput micropayments and Bitcoin-native dispute resolution.

## Table of Contents

- [Bitstream Channels - Stacks L2 Payment Channel Protocol](#bitstream-channels---stacks-l2-payment-channel-protocol)
	- [Table of Contents](#table-of-contents)
	- [Architecture Overview](#architecture-overview)
	- [Key Features](#key-features)
		- [Layer 2 Performance](#layer-2-performance)
		- [Bitcoin Compliance](#bitcoin-compliance)
		- [Advanced Security](#advanced-security)
	- [Technical Specifications](#technical-specifications)
		- [Data Structures](#data-structures)
			- [Payment Channel Record](#payment-channel-record)
		- [Error Codes](#error-codes)
	- [Core Functionality](#core-functionality)
		- [Channel Management](#channel-management)
		- [Channel Closure](#channel-closure)
	- [Security Model](#security-model)
		- [Audit Considerations](#audit-considerations)
		- [Recovery Mechanisms](#recovery-mechanisms)
	- [Deployment \& Usage](#deployment--usage)
		- [Requirements](#requirements)

## Architecture Overview

Bitstream Channels implement a bidirectional payment channel system with:

- **STX-denominated balances** with Bitcoin-style UTXO accounting
- **Hybrid settlement**: Off-chain state updates with on-chain Bitcoin block-aligned finality
- **Dispute-resistant design**: 1-week challenge period aligned with Bitcoin block production
- **Non-custodial escrow**: Funds fully controlled by multisig cryptographic proofs

```clarity
                         +-------------------+
                         | Stacks Blockchain |
                         +-------------------+
                                ▲       ▲
                    Open/Close |       | Dispute
                               +---------+
                               | Channel |
                               | Contract|
                               +---------+
                                  ▲   ▲
                       Off-chain  |   |  On-chain
                        Updates   |   |  Settlements
                                 +-------+
                                 | Users |
                                 +-------+
```

## Key Features

### Layer 2 Performance

- 10,000+ TPS capability through off-chain state updates
- Sub-second transaction finality between channel participants

### Bitcoin Compliance

- Dispute deadlines based on Bitcoin block height
- SHA256 channel ID derivation compatible with Bitcoin scripts
- STX transfers convertible to BTC via atomic swaps

### Advanced Security

- Replay attack protection through state nonces
- Signature malleability protection
- Front-running resistant closure mechanisms

## Technical Specifications

### Data Structures

#### Payment Channel Record

```clarity
{
  channel-id: (buff 32),       // SHA256(initiator_pubkey || counterparty_pubkey || nonce)
  participant-a: principal,    // Stacks address (initiator)
  participant-b: principal     // Stacks address (counterparty)
  total-deposited: uint,       // Total STX in escrow (micro-STX)
  balance-a: uint,             // Participant A's balance (micro-STX)
  balance-b: uint,             // Participant B's balance (micro-STX)
  is-open: bool,               // Channel status flag
  dispute-deadline: uint,      // Bitcoin block height for dispute expiration
  nonce: uint                  // State version counter
}
```

### Error Codes

| Code | Constant               | Description                        |
| ---- | ---------------------- | ---------------------------------- |
| 100  | ERR-NOT-AUTHORIZED     | Unauthorized contract interaction  |
| 101  | ERR-CHANNEL-EXISTS     | Duplicate channel creation attempt |
| 102  | ERR-CHANNEL-NOT-FOUND  | Nonexistent channel access         |
| 103  | ERR-INSUFFICIENT-FUNDS | Balance validation failure         |
| 104  | ERR-INVALID-SIGNATURE  | Cryptographic proof mismatch       |
| 105  | ERR-CHANNEL-CLOSED     | Operation on terminated channel    |
| 106  | ERR-DISPUTE-PERIOD     | Premature dispute resolution       |
| 107  | ERR-INVALID-INPUT      | Parameter validation failure       |

## Core Functionality

### Channel Management

```clarity
;; Create new payment channel with initial deposit
(create-channel channel-id participant-b initial-deposit)

;; Add funds to existing channel
(fund-channel channel-id participant-b additional-funds)
```

**Parameters:**

- `channel-id`: 32-byte unique identifier (recommended: SHA256 hash of creation parameters)
- `participant-b`: Stacks address of counterparty
- `initial-deposit`: Initial STX amount in micro-STX (1 STX = 1,000,000 micro-STX)

### Channel Closure

```clarity
;; Cooperative closure with dual signatures
(close-channel-cooperative
  channel-id
  participant-b
  balance-a
  balance-b
  signature-a
  signature-b
)

;; Unilateral closure with dispute period
(initiate-unilateral-close
  channel-id
  participant-b
  proposed-balance-a
  proposed-balance-b
  signature
)

;; Finalize unilateral closure after dispute period
(resolve-unilateral-close channel-id participant-b)
```

**Security Requirements:**

- Cooperative closures require ECDSA signatures from both parties
- Unilateral closures enforce 1008-block (~1 week) dispute period
- Balance allocations must sum to total deposited STX

## Security Model

### Audit Considerations

1. **Signature Verification**: Implement full ECDSA secp256k1 verification
2. **Replay Protection**: Validate state nonces for all operations
3. **Funds Locking**: Ensure escrow balances match on-chain STX holdings
4. **Dispute Deadline**: Enforce Bitcoin block height validation

### Recovery Mechanisms

- Emergency withdrawal function (contract owner only)
- Time-locked channel expiration (1008 blocks)
- State-invalidated closed channels

## Deployment & Usage

### Requirements

- Stacks node v3.0+
- Clarinet SDK v1.5.0+
- Node.js v18+

```

```
