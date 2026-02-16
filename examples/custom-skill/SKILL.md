---
name: daily_stock_check
description: Check stock prices and market news. Use when the user asks about stocks, market conditions, or portfolio updates.
metadata: {"openclaw": {"requires": {"bins": ["curl"]}, "os": ["darwin", "linux"]}}
---

# Daily Stock Check

## Usage

Use `curl` to fetch stock data from a public API:

```bash
curl -s "https://query1.finance.yahoo.com/v8/finance/chart/AAPL?interval=1d&range=1d" | jq '.chart.result[0].meta | {symbol, regularMarketPrice, previousClose}'
```

## Output Format

Present results as a clean summary:
- **Symbol:** ticker
- **Current Price:** latest
- **Change:** vs previous close (% and absolute)
- **Trend:** up/down emoji

## Multiple Stocks

Loop through a watchlist:
```bash
for SYMBOL in AAPL GOOGL MSFT NVDA; do
  curl -s "https://query1.finance.yahoo.com/v8/finance/chart/$SYMBOL?interval=1d&range=1d"
done
```

Present as a table (for Telegram/Discord) or bullet list (for WhatsApp).
