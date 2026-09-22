# SecureRealEstatePlatform

A Solidity smart contract for **fractional real-estate investment using USDT**.

The platform allows real-estate companies to register properties, divide properties into digital shares, and allow investors to purchase those shares using USDT. The contract keeps track of investor ownership, collects platform fees, manages property funds, and distributes dividends to shareholders on-chain.

The current contract is designed for **TRON-compatible USDT environments** and uses Solidity `0.5.10`.

---

## Overview

Traditional real-estate investment often requires investors to provide a large amount of capital to participate in a property.

SecureRealEstatePlatform is designed to make this process more accessible by dividing a property into a fixed number of digital shares.

For example:

```text
Property
   │
   ├── 1,000 Shares
   │
   ├── Share Price: 100 USDT
   │
   └── Total Value: 100,000 USDT
             │
             ├── Investor A → 100 shares
             ├── Investor B → 250 shares
             └── Investor C → 50 shares
