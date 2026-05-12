# IMC Prosperity 4 Round 5 Relative-Value Strategy

Python trading strategy developed for IMC Prosperity 4 Round 5, contributing to a top 1% global team score.

The strategy uses a static relative-value approach across related product families. Instead of forecasting each asset independently, it estimates a family-level fair value, compares each product’s current mid-price against its normal offset from that family mean, and trades when the residual deviation becomes large enough.

<img width="806" height="468" alt="image" src="https://github.com/user-attachments/assets/bd931e99-6a23-494d-99cc-7ac9c5f30f3e" /><img width="806" height="468" alt="image" src="https://github.com/user-attachments/assets/7e9709c0-6b98-4e44-a763-1f4d6162949a" />

## Strategy

For each product family, the model calculates:

```text
family_mid = average mid-price of related products
fair_value = family_mid + calibrated product offset
residual = current_mid - fair_value

