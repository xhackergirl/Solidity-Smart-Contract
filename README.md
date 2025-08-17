# FundMe (Sepolia) — USD-denominated funding with Chainlink

Crowdfunding-style ETH contract that enforces a **\$5 USD minimum** per contribution using a **Chainlink ETH/USD** price feed. Includes a tiny `PriceConverter` library to convert ETH (wei) to USD (18 decimals) on-chain.

[![License: MIT](https://img.shields.io/badge/License-MIT-green.svg)](#license)
![Solidity](https://img.shields.io/badge/Solidity-^0.8.18-black)
![Network](https://img.shields.io/badge/Network-Sepolia-blue)

---

## What’s inside

* **`PriceConverter.sol`** — Library that:

  * Reads the **ETH/USD** price from Chainlink (Sepolia feed).
  * Scales Chainlink’s 8-decimals answer to **18 decimals** (×1e10) for ergonomic math with wei.
  * Converts any `uint256 ethAmount` (wei) to USD (18 decimals).

* **`FundMe.sol`** — Minimal funding contract that:

  * Requires each contribution to be **≥ \$5** (as `MINIMUM_USD = 5e18`).
  * Records per-funder totals and keeps a list of funders.
  * Restricts `withdraw()` to the **owner** (the deployer).
  * Transfers with `call` and resets balances first to reduce reentrancy risk.

> **Chainlink Sepolia ETH/USD feed:** `0x694AA1769357215DE4FAC081bf1f309aDC325306`
> Update this address if you deploy to other networks.

---

## How it works

1. `PriceConverter.getPrice()` calls Chainlink’s `latestRoundData()` for ETH/USD.
   Chainlink returns **8 decimals** → multiply by **1e10** → **18 decimals**.

2. `fund()` uses `using PriceConverter for uint256;`:

   ```solidity
   require(msg.value.getConversionRate() >= MINIMUM_USD, "You need to spend more ETH!");
   ```

   `getConversionRate()` returns the **USD value (18 decimals)** of `msg.value`.

3. `withdraw()` is **owner-only**, zeroes balances, clears the funders array, then sends the full balance with `call`.

---

## Quickstart (pick one)

### Hardhat (minimal)

```bash
npm i -D hardhat @nomicfoundation/hardhat-toolbox
npm i @chainlink/contracts
# add these files to contracts/, set RPC+key, then:
npx hardhat compile
# deploy script required for your setup
```

### Foundry (minimal)

```bash
forge init
forge install smartcontractkit/chainlink-brownie-contracts@v0.6.1
forge build
# forge create src/FundMe.sol:FundMe --rpc-url <SEPOLIA_RPC_URL> --private-key <KEY>
```

---

## Interact (examples)

* **Fund** (≥ \$5 in USD at current price):

  ```bash
  cast send <ADDRESS> "fund()" --value 0.01ether --rpc-url <SEPOLIA_RPC_URL> --private-key <KEY>
  ```

* **Withdraw** (owner only):

  ```bash
  cast send <ADDRESS> "withdraw()" --rpc-url <SEPOLIA_RPC_URL> --private-key <KEY>
  ```

* **Check price feed version**:

  ```bash
  cast call <ADDRESS> "getVersion()(uint256)" --rpc-url <SEPOLIA_RPC_URL>
  ```

---

## Notes & limitations

* **Network-specific feed**: The price feed address is **Sepolia-only** here. Use the correct address for other networks.
* **Staleness/round checks**: For brevity, no explicit staleness or negative answer checks are implemented. Consider validating `updatedAt`, `answeredInRound`, and `answer > 0` for production.
* **Unbounded loop in `withdraw()`**: Runs in **O(n)** over funders; can be expensive with many funders. Consider patterns that avoid unbounded loops.
* **Reentrancy**: State is reset **before** the external call. For extra safety, consider `ReentrancyGuard` or pull-payments.

---

## File structure (suggested)

```
.
├─ contracts/ or src/
│  ├─ FundMe.sol
│  └─ PriceConverter.sol
└─ README.md  ← you are here
```

---

## License

MIT © You
