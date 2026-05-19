---
name: scalp
description: Momentum scalping strategy -- scan for movers, confirm with orderbook, enter with tight TP/SL.
---

# Momentum Scalping

A short-term scalping strategy that identifies momentum moves and captures quick profits with tight risk management.

## Parameters

- **Take-Profit**: 1.0% from entry
- **Stop-Loss**: 0.6% from entry
- **Risk per trade**: 1-2% of available balance
- **Max concurrent positions**: 3
- **Leverage**: 10x (default)

## Workflow

### 1. Scan for Momentum

Call `get_tickers` with no symbol to retrieve all tickers. Filter for:
- **Long candidates**: `price24hPcnt` > 3% (strong upward momentum)
- **Short candidates**: `price24hPcnt` < -3% (strong downward momentum)
- **Volume filter**: prioritize symbols with above-average 24h volume

Sort candidates by absolute `price24hPcnt` to find the strongest movers.

### 2. Check Current Positions

Call `get_positions` to see how many positions are open. Do not exceed 3 concurrent scalp positions. If already at the limit, skip or close the weakest position first.

### 3. Orderbook Confirmation

For each candidate, call `get_orderbook` and analyze:
- **For longs**: bid volume should be at least 1.5x ask volume in the top 5 levels (buying pressure)
- **For shorts**: ask volume should be at least 1.5x bid volume in the top 5 levels (selling pressure)

If the orderbook does not confirm the direction, skip the symbol.

### 4. Execute the Trade

For confirmed candidates:
1. Call `get_instruments` to get lot size and tick size
2. Call `get_balance` to check available margin
3. Calculate position size (risk 1-2% of balance)
4. Call `set_leverage` to set 10x leverage
5. Call `place_order` as a market order with:
   - Take-profit at 1.0% from current price
   - Stop-loss at 0.6% from current price

### 5. Monitor and Cycle

After placing a trade:
1. Call `get_positions` to verify entry
2. Report the trade details
3. If fewer than 3 positions are open, scan for the next candidate
4. Repeat the cycle

## Notes

- This strategy works best during high-volatility periods
- Avoid scalping during low-volume hours
- If a symbol has already been scalped and closed at SL, do not re-enter the same symbol in the same session
