# Best Deals Ads checkout

Self-serve checkout for print ads in Best Deals Book and Exclusively Home Improvements, built in the same style as the Best Deals Direct mailer checkout.

It's a single static page, `index.html`. Open it in a browser; there's no build step.

**Flow:** 1 · Build your ad (magazine, ad size, zones, issue months, artwork) → 2 · Checkout (business, billing address, invoice details) → 3 · Confirmation.

## Issues and premium covers
Customers can pick any months over the next 12, not only consecutive ones. The volume discount is based on how many issues they book (`CONFIG.volume`).

Back Cover, Inside Front Cover and Inside Back Cover are one advertiser per zone, per issue. To mark a cover as sold, add the month to `CONFIG.soldOut` (magazine → size → zone → `"YYYY-MM"`). That month then shows as "Taken in Zx" for anyone who picks that zone. If a customer changes size, zone or magazine and a month they picked is no longer free, that month is removed and they're told why. The `soldOut` entries in the file now are examples only.

## Demo mode
`CONFIG.demo` is `true` right now. The whole flow works, but nothing is sent: no webhook call and no email. A yellow banner tells visitors it's a demo. Set `demo: false` once the order destination, prices and inventory are real.

## Where orders go (once demo is off)
When a customer presses **Request ad space**:
- **If `CONFIG.orderEndpoint` is set** (a Zapier, Make or Google Apps Script webhook, or your CRM), the order is POSTed there as JSON: reference, magazine, size, zones, months, full pricing breakdown and customer details. From there it can create a Notion row, send an email, or start the invoice. If the POST fails, the customer sees an error with the phone number and can try again.
- **If it's empty**, the customer's email app opens with the order pre-filled to `CONFIG.salesEmail` (advertise@bestdealsbook.com), and they have to press send. This is a stopgap; set a webhook before launch.

The confirmation page shows an order summary that the customer can print or save as a PDF. A hidden honeypot field drops most spam bots.

## Before launch
- **Prices, sizes, zones:** all in `CONFIG` at the top of the `<script>`. The current values are placeholders.
- **Sold covers:** replace the example `CONFIG.soldOut` entries with real bookings.
- **Order webhook:** set `CONFIG.orderEndpoint`, and confirm that `salesEmail` is the right inbox.
- **Payment:** none is taken on the page. Once placement is confirmed, the customer is invoiced (an optional billing email can be entered at checkout).
