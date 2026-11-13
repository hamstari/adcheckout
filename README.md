# Best Deals Ads checkout

Self-serve checkout for print ads in Best Deals Book and Exclusively Home Improvements, built in the same style as the Best Deals Direct mailer checkout.

It's a single static page, `index.html`. Open it in a browser; there's no build step.

**Flow:** 1 · Build your ad (magazine, ad size, zones, run length, artwork) → 2 · Checkout (business, billing address, invoice details) → 3 · Confirmation.

## Before launch
- **Prices, sizes, zones:** everything lives in the `CONFIG` object at the top of the `<script>`. The current prices, ad dimensions and homes per zone are placeholders.
- **Order submission:** search for `TODO` in `index.html`. The order object is built and logged, but nothing sends it anywhere yet (CRM, email or invoicing system).
- **Payment:** no payment is taken on the page. Once placement is confirmed, the customer is invoiced (an optional billing email can be entered at checkout).
