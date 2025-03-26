# Bitcoin Yield Protocol - Smart Contract Documentation

![Stacks Layer 2](https://img.shields.io/badge/Built%20on-Stacks%20L2-5546FF)

## Overview

The **Bitcoin Yield Protocol** is a decentralized staking and governance system enabling Bitcoin holders to earn yield on STX tokens while participating in protocol governance. Built on Stacks Layer 2, it combines DeFi mechanics with decentralized decision-making through a tiered staking system, dynamic rewards, and voting power allocation.

### Key Features

- **Tiered Staking**: Earn higher rewards at Tier 2 (5M staked) and Tier 3 (10M staked)
- **Lock-Up Multipliers**: Boost rewards by 25-50% with 1-2 month lock periods
- **Governance Proposals**: Create/vote on proposals using staking-based voting power
- **Emergency Circuit Breaker**: Pause functionality for critical situations
- **Cooldown Mechanism**: 24-hour waiting period for unstaking
- **Dynamic Rewards**: Yield calculated based on stake size, duration, and tier

---

## Technical Details

### Contract Basics

- **Language**: Clarity (Stacks L2 smart contract language)
- **Token**: `ANALYTICS-TOKEN` (fungible rewards token)
- **STX Pool**: All staked STX stored in `stx-pool` variable
- **Base Reward Rate**: 5% APY (adjustable via governance)
- **Minimum Stake**: 1M microSTX (1 STX = 1,000,000 µSTX)

### Core Components

| Component          | Description                                            |
| ------------------ | ------------------------------------------------------ |
| `Proposals`        | Stores governance proposals with voting data           |
| `UserPositions`    | Tracks user stakes, rewards, and tier status           |
| `StakingPositions` | Manages individual stake locks/cooldowns               |
| `TierLevels`       | Configuration for staking tiers (multipliers/benefits) |

---

## Key Functions

### User Actions

| Function           | Parameters                     | Description                                   |
| ------------------ | ------------------------------ | --------------------------------------------- |
| `stake-stx`        | `amount`, `lock-period`        | Stake STX with optional lock (0/1/2 months)   |
| `initiate-unstake` | `amount`                       | Start 24h cooldown for unstaking              |
| `complete-unstake` | -                              | Finalize unstaking after cooldown             |
| `create-proposal`  | `description`, `voting-period` | Submit governance proposal (1M+ voting power) |
| `vote-on-proposal` | `proposal-id`, `vote-for`      | Cast votes using staking power                |

### Administrative

| Function              | Description                                |
| --------------------- | ------------------------------------------ |
| `pause-contract`      | Freeze all staking operations (owner-only) |
| `resume-contract`     | Resume normal operations (owner-only)      |
| `initialize-contract` | Set up initial tiers (owner-only)          |

### Getters

| Read-Only Function   | Returns                      |
| -------------------- | ---------------------------- |
| `get-stx-pool`       | Total STX locked in protocol |
| `get-proposal-count` | Number of created proposals  |
| `get-contract-owner` | Admin address                |

---

## Staking Mechanics

### Tier System

| Tier | Minimum STX | Multiplier | Governance Features |
| ---- | ----------- | ---------- | ------------------- |
| 1    | 1M µSTX     | 1x         | Basic voting        |
| 2    | 5M µSTX     | 1.5x       | Enhanced voting     |
| 3    | 10M µSTX    | 2x         | Full privileges     |

### Lock-Up Multipliers

| Lock Period | Blocks | Multiplier |
| ----------- | ------ | ---------- |
| No Lock     | 0      | 1x         |
| 1 Month     | 4,320  | 1.25x      |
| 2 Months    | 8,640  | 1.5x       |

**Example**:  
10M STX (Tier 3) + 2-month lock = `2x * 1.5x = 3x total multiplier`

---

## Governance System

### Proposal Lifecycle

1. **Creation**: Requires ≥1M voting power
   - Max description: 256 UTF-8 characters
   - Voting period: 100-2,880 blocks (~1hr-1day)
2. **Voting**:
   - Voting power = `(STX staked) × (tier multiplier)`
   - Votes are irreversible
3. **Execution**:
   - Requires >50% approval
   - Minimum 1M votes cast

---

## Reward Calculation

Rewards accrue per block based on:  
`(staked_amount × base_rate × multiplier × blocks_staked) / 14,400,000`

**Variables**:

- `base_rate`: 500 (5% annualized)
- `multiplier`: Tier × Lock multiplier
- `blocks_staked`: Duration in blocks

**Example Calculation**:  
5M STX staked (Tier 2) with 1-month lock for 10,000 blocks  
`(5,000,000 × 500 × (1.5 × 1.25) × 10,000) / 14,400,000 = 3,906.25 µSTX`

---

## Security Features

1. **Emergency Pause**: Contract owner can freeze operations
2. **Cooldown Protection**: 24h delay on unstaking
3. **Validation Checks**:
   - Minimum stake enforcement
   - Valid lock period verification
   - Proposal parameter constraints
4. **Reentrancy Protection**: State changes before transfers

---

## Error Codes

| Code | Description               |
| ---- | ------------------------- |
| 1000 | Unauthorized access       |
| 1001 | Invalid parameters        |
| 1002 | Incorrect amount          |
| 1003 | Insufficient balance      |
| 1004 | Cooldown active           |
| 1005 | No active stake           |
| 1006 | Below minimum requirement |
| 1007 | Contract paused           |

---

## Usage Examples

### Staking with Lock

```clarity
;; Stake 5M µSTX for 1 month (4,320 blocks)
(stake-stx u5000000 u4320)
```

### Creating Proposal

```clarity
;; Create 24hr voting period proposal
(create-proposal "Increase base reward rate to 6%" u1440)
```

### Voting

```clarity
;; Vote 'Yes' on proposal 42
(vote-on-proposal u42 true)
```
