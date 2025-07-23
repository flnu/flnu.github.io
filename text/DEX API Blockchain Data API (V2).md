# DEX API | Blockchain Data API (V2)

This technical documentation explores decentralized exchange (DEX) data retrieval through blockchain query interfaces. We'll examine practical API implementations for Ethereum-based DEX analysis using Bitquery's schema framework.

## Understanding Core DEX API Endpoints

The platform provides two primary interfaces for DEX data collection:

1. **DEXTrades**: Aggregates trading activity across multiple decentralized exchanges
2. **DEXTradeByTokens**: Focuses on token-specific trading analytics

These endpoints enable developers to analyze liquidity pools, trading pairs, and market participant behavior across EVM-compatible networks.

### Network-Wide DEX Discovery

To retrieve comprehensive DEX information for an Ethereum network, implement this GraphQL query:

```graphql
query DexMarkets($network: evm_network) {
  EVM(network: $network) {
    DEXTradeByTokens {
      Trade {
        Dex {
          ProtocolFamily
        }
      }
      buyers: uniq(of: Trade_Buyer)
      sellers: uniq(of: Trade_Sender)
      count(if: {Trade: {Side: {Type: {is: buy}}}})
    }
  }
}
```

**Parameters:**
```json
{ "network": "eth" }
```

This query reveals:
- Available decentralized exchanges
- Unique buyer/seller counts
- Buy-side transaction volume

👉 [Explore crypto analytics tools](https://bit.ly/okx-bonus)

### Protocol-Specific Analytics

To analyze specific DEX protocols like Uniswap, use this enhanced query:

```graphql
query DexMarkets($network: evm_network, $market: String) {
  EVM(network: $network) {
    DEXTradeByTokens(
      orderBy: {ascendingByField: "Block_Time"}
      where: {Trade: {Dex: {ProtocolFamily: {is: $market}}}}
    ) {
      Block {
        Time(interval: {count: 1, in: hours})
      }
      trades: count
      buyers: uniq(of: Trade_Buyer)
      sellers: uniq(of: Trade_Sender)
      tokens: uniq(of: Trade_Currency_SmartContract)
    }
  }
}
```

**Parameters:**
```json
{ 
  "market": "Uniswap",
  "network": "eth"
}
```

This provides granular insights into:
- Temporal trading patterns
- Token liquidity metrics
- Market participant diversity

## Trading Pair Analysis

To examine trading pairs on specific DEX platforms:

```graphql
query DexMarkets($network: evm_network, $market: String, $time_10min_ago: DateTime, $time_1h_ago: DateTime, $time_3h_ago: DateTime) {
  EVM(network: $network) {
    DEXTradeByTokens(
      orderBy: {descendingByField: "usd"}
      where: {Trade: {Dex: {ProtocolFamily: {is: $market}}}, Block: {Time: {after: $time_3h_ago}}}
      limit: {count: 200}
    ) {
      Trade {
        Currency {
          Symbol
          Name
          SmartContract
          Fungible
        }
        Side {
          Currency {
            Symbol
            Name
            SmartContract
          }
        }
        price_usd: PriceInUSD(maximum: Block_Number)
        price_10min_ago: Price(maximum: Block_Number if: {Block: {Time: {before: $time_10min_ago}}})
      }
      usd: sum(of: Trade_AmountInUSD)
      count
    }
  }
}
```

**Parameters:**
```json
{
  "market": "Uniswap",
  "network": "eth",
  "time_10min_ago": "2024-09-22T13:21:39Z",
  "time_1h_ago": "2024-09-22T12:31:39Z",
  "time_3h_ago": "2024-09-22T10:31:39Z"
}
```

### Key Metrics Tracking:
- Price volatility over time intervals
- USD trading volume by pair
- Top trading pairs by liquidity

👉 [Access blockchain data solutions](https://bit.ly/okx-bonus)

## Trader Behavior Analysis

To identify top traders by volume:

```graphql
query DexMarkets($network: evm_network, $market: String) {
  EVM(network: $network) {
    DEXTradeByTokens(
      orderBy: {descendingByField: "volumeUsd"}
      limit: {count: 100}
      where: {Trade: {Dex: {ProtocolFamily: {is: $market}}}}
    ) {
      Trade {
        Buyer
        Dex {
          OwnerAddress
          ProtocolFamily
          ProtocolName
        }
        Currency {
          SmartContract
          Symbol
          Name
        }
      }
      volumeUsd: sum(of: Trade_Side_AmountInUSD)
    }
  }
}
```

**Parameters:**
```json
{ "market": "Uniswap", "network": "eth" }
```

### Behavioral Insights:
- Whale transaction patterns
- Protocol loyalty metrics
- Token preference hierarchies

## Real-Time Trade Monitoring

For tracking the latest trading activity:

```graphql
query LatestTrades($network: evm_network, $market: String) {
  EVM(network: $network) {
    DEXTradeByTokens(
      orderBy: {descending: Block_Time}
      limit: {count: 50}
      where: {Trade: {Dex: {ProtocolFamily: {is: $market}}}}
    ) {
      Block {
        Time
      }
      Transaction {
        Hash
      }
      Trade {
        Dex {
          OwnerAddress
          ProtocolFamily
          ProtocolName
        }
        AmountInUSD
        Price
      }
    }
  }
}
```

**Parameters:**
```json
{ "market": "Uniswap", "network": "eth" }
```

### Monitoring Applications:
- Arbitrage opportunity detection
- MEV (Miner Extractable Value) tracking
- Market sentiment analysis

## Frequently Asked Questions

**Q: How do I authenticate API requests?**  
A: Authentication uses API keys passed in request headers. Detailed instructions are available in the Bitquery documentation portal.

**Q: What rate limits apply to DEX queries?**  
A: Free-tier accounts receive 100 requests/minute with higher limits available for enterprise plans.

**Q: Can I track token pairs across multiple DEXs simultaneously?**  
A: Yes, by using the `ProtocolFamily` filter with multiple values in your query's where clause.

**Q: How does time interval aggregation affect data accuracy?**  
A: Shorter intervals provide higher granularity but may increase response times. Optimal performance balances interval duration with dataset size.

**Q: What blockchain networks does this API support?**  
A: The interface supports all EVM-compatible networks including Ethereum, Binance Smart Chain, Polygon, and Avalanche.

👉 [Discover institutional crypto solutions](https://bit.ly/okx-bonus)

## Technical Implementation Guide

For successful integration, consider these best practices:

1. **Pagination Strategy**: Use cursor-based pagination for large datasets
2. **Error Handling**: Implement exponential backoff for rate limit scenarios
3. **Schema Validation**: Utilize GraphQL fragments for query validation