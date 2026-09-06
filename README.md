# Crypto Tax Calculator Skill for Amadeus

[![Version](https://img.shields.io/badge/version-1.0.0-blue)](manifest.json)
[![Amadeus](https://img.shields.io/badge/AMA_Hub-Skill-00C4B3)](https://hub.ama.one/skills)

An Amadeus Agent skill for crypto tax calculations: fetch on-chain wallet transactions and CEX trade history, then compute capital gains with FIFO/LIFO lot matching.

## Features

- **On-chain wallet scanning** — Fetch native ETH transfers + ERC-20 token transfers for any EVM address via Blockscout (no API key required)
- **CEX trade history** — Pull spot trades from Binance, Coinbase, Kraken, and 100+ exchanges via CCXT
- **FIFO/LIFO lot matching** — Proper cost-basis tracking with short/long-term classification
- **Historical prices** — CoinGecko daily price lookup for accurate cost basis at transaction time
- **Income classification** — Separate staking, mining, and airdrop income from capital gains
- **Multi-chain** — Ethereum, Polygon, Arbitrum, Base, Optimism

## Installation

### Via AMA Hub (Recommended)
1. Navigate to the **Skills** tab in [AMA Hub](https://hub.ama.one/skills).
2. Click **Install from repo**.
3. Paste the raw manifest URL:
   `https://raw.githubusercontent.com/JackTheGit/amadeus-crypto-tax/main/manifest.json`

### Python Dependencies (Local standalone execution)
```bash
pip install --user ccxt requests
```

## Actions

The capabilities below correspond to the `read` actions defined in the `manifest.json`.

### `fetchWalletTransactions`

Fetch on-chain transaction history for an EVM wallet.

| Param | Type | Description |
|---|---|---|
| `address` | string | Wallet address (0x...) |
| `chain` | string | Network: ethereum, polygon, arbitrum, base, optimism |
| `fromDate` | string | Start date (YYYY-MM-DD) |
| `toDate` | string | End date (YYYY-MM-DD) |

### `fetchCexTransactions`

Fetch spot trade history from a centralized exchange.

| Param | Type | Description |
|---|---|---|
| `exchange` | string | CCXT exchange ID (e.g. binance, coinbase, kraken) |
| `apiKey` | string | The exchange API key |
| `secret` | string | The exchange API secret |
| `fromDate` | string | Start date (YYYY-MM-DD) |
| `toDate` | string | End date (YYYY-MM-DD) |

### `calculateCryptoGains`

Compute capital gains/losses with FIFO/LIFO lot matching.

| Param | Type | Description |
|---|---|---|
| `method` | string | Lot matching: FIFO or LIFO |
| `currency` | string | The fiat currency for gains reporting, e.g. USD |

## Usage (Python Scripting)

```python
from tools import fetchWalletTransactions, calculateCryptoGains

# Fetch wallet history
result = fetchWalletTransactions(
    address="0xd8dA6BF26964aF9D7eEd9e03E53415D37aA96045",
    chain="ethereum",
    fromDate="2025-01-01",
    toDate="2025-12-31"
)

# Calculate gains
tax = calculateCryptoGains(
    transactions=result["transactions"],
    method="FIFO",
    currency="USD"
)

print(tax["capital_gains_summary"])
# {
#   "total_realized_gain_loss_usd": 12500.00,
#   "short_term_gain_loss_usd": 8000.00,
#   "long_term_gain_loss_usd": 4500.00
# }

for disposal in tax["sales_detail"]:
    print(f"{disposal['sell_date'][:10]} {disposal['asset']} "
          f"gain=${disposal['gain_loss']:.2f} "
          f"{'LTCG' if disposal['long_term'] else 'STCG'}")
```

## Caveats

- **Wallet transactions are simplistic** — all outgoing ETH is classified as "SELL" (taxable disposition). Wallet-to-wallet transfers, bridge transactions, and contract interactions are NOT real sales but are classified as such.
- **No DEX swap detection** — only native ETH transfers and ERC-20 token transfers. No internal transaction tracing.
- **Unmatched disposals** — when a sell has no matching buy lots, cost basis is $0 and the full proceeds are treated as short-term gain.
- **Fees not deducted** — transaction fees are recorded but do not reduce sale proceeds.
- **ERC-20 prices are approximate** — token prices use ETH's daily price as a proxy. For accurate token pricing, provide price data externally.

## License

MIT
