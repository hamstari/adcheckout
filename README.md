# Best Deals Ads checkout

Self-serve checkout for print ads in Best Deals Book and Exclusively Home Improvements, built in the same style as the Best Deals Direct mailer checkout.

It's a single static page, `index.html`. Open it in a browser; there's no build step.

**Flow:** 1 · Build your ad (magazine, ad size, zones, issue months, artwork) → 2 · Checkout (business, billing address, invoice details) → 3 · Confirmation.

## Issues and premium covers
Customers can pick any months over the next 12, not only consecutive ones. The volume discount is based on how many issues they book (`CONFIG.volume`).

Back Cover, Inside Front Cover and Inside Back Cover are one advertiser per zone, per issue. To mark a cover as sold, add the month to `CONFIG.soldOut` (magazine → size → zone → `"YYYY-MM"`). That month then shows as "Taken in Zx" for anyone who picks that zone. If a customer changes size, zone or magazine and a month they picked is no longer free, that month is removed and they're told why. The `soldOut` entries in the file now are examples only.

## Before launch
- **Prices, sizes, zones:** everything lives in the `CONFIG` object at the top of the `<script>`. The current prices, ad dimensions and homes per zone are placeholders.
- **Order submission:** search for `TODO` in `index.html`. The order object is built and logged, but nothing sends it anywhere yet (CRM, email or invoicing system).
- **Payment:** no payment is taken on the page. Once placement is confirmed, the customer is invoiced (an optional billing email can be entered at checkout).
