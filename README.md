# FundMe (Sepolia) — USD-denominated funding with Chainlink

Crowdfunding-style ETH contract that enforces a **$5 USD minimum** per contribution using a **Chainlink ETH/USD** price feed. Includes a tiny `PriceConverter` library to convert ETH (wei) to USD (18 decimals) on-chain.

[![License: MIT](https://img.shields.io/badge/License-MIT-green.svg)](#license)
![Solidity](https://img.shields.io/badge/Solidity-^0.8.18-black)
![Network](https://img.shields.io/badge/Network-Sepolia-blue)

---

## ✨ What’s inside

- **`PriceConverter.sol`** — Library that:
  - Reads the **ETH/USD** price from Chainlink (Sepolia feed).
  - Scales Chainlink’s 8-decimals answer to **18 decimals** (×1e10) for ergonomic math with wei.
  - Converts any `uint256 ethAmount` (wei) to USD (18 decimals).

- **`FundMe.sol`** — Minimal funding contract that:
  - Requires each contribution to be **≥ $5** (as `MINIMUM_USD = 5e18`).
  - Records per-funder totals and keeps a list of funders.
  - Restricts `withdraw()` to the **owner** (the deployer).
  - Transfers with `call` and resets balances first to reduce reentrancy risk.

> **Chainlink Sepolia ETH/USD feed:** `0x694AA1769357215DE4FAC081bf1f309aDC325306`  
> Update this address if you deploy to other networks.

---

## 🧠 How it works

1. `PriceConverter.getPrice()` calls `latestRoundData()` on the Chainlink Aggregator.  
   Chainlink returns **8-decimal** ETH/USD → we multiply by **1e10** → **18 decimals**.

2. `fund()` uses `using PriceConverter for uint256;` so you can write:
   ```solidity
   require(msg.value.getConversionRate() >= MINIMUM_USD, "You need to spend more ETH!");
