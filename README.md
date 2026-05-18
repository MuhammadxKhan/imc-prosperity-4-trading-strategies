# IMC Prosperity 4 Round 5 Relative-Value Strategy

Python trading strategy developed for IMC Prosperity 4 Round 5, contributing to a top 1% global team score.

The strategy uses a static relative-value approach across related product families. Instead of forecasting each asset independently, it estimates a family-level fair value, compares each product’s current mid-price against its normal offset from that family mean, and trades when the residual deviation becomes large enough.

<img width="806" height="468" alt="image" src="https://github.com/user-attachments/assets/bd931e99-6a23-494d-99cc-7ac9c5f30f3e" />

## Strategy Overview

The Round 5 products were structured into related families of five instruments. The strategy treats the common movement of each family as mostly noise and focuses on relative mispricing between members of the same family.

For each family:

1. Compute the mid-price of every product from the best bid and best ask.
2. Skip the entire family if any product book is missing or one side of the order book is unavailable.
3. Estimate the family-level mid-price.
4. Estimate each product’s fair value as:

```python
fair_value = family_mid + product_offset
residual = current_mid - fair_value
```

5. If the residual is above the product’s threshold, the product is treated as expensive and the strategy shorts it.
6. If the residual is below the negative threshold, the product is treated as cheap and the strategy buys it.
7. If the residual is inside the threshold band, the strategy keeps the existing inventory rather than forcing a flat exit.

## Technical Highlights

- Implemented a static relative-value model across 10 product families, each containing five related instruments.
- Used calibrated product-specific fair-value offsets rather than a live EMA, after validation showed that adaptive offsets moved toward bad prices and weakened the signal.
- Applied product-specific residual thresholds to trade only instruments with positive validation contribution.
- Skipped incomplete product families to avoid biased family-average estimates when order-book data was missing.
- Converted residual signals into target long/short positions under a fixed ±10 position limit.
- Generated executable orders by crossing visible asks when buying and visible bids when selling.
