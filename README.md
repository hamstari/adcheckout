# Best Deals Ads checkout

Self-serve checkout for print ads in Best Deals Book and Exclusively Home Improvements, built in the same style as the Best Deals Direct mailer checkout.

It's a single static page, `index.html`. Open it in a browser; there's no build step.

**Flow:** 1 · Build your ad (magazine, ad size, zones, issue months, artwork) → 2 · Checkout (business, billing address, invoice details) → 3 · Confirmation.

## Updating rates
All prices, zones and household counts live in **`rates.csv`**, a normal spreadsheet with one row per magazine and zone:

| Magazine | Zone | Zone name | Homes | Back Cover | Inside Front Cover | Inside Back Cover | Full Page | Half Page | Quarter Page |
|---|---|---|---|---|---|---|---|---|---|
| BDB | Z1 | Zone 1 | 20000 | 1495 | 1195 | 1095 | 895 | 525 | 315 |

- **Change a price:** edit the number. `$1,495` and `1495` both work.
- **Add a zone:** add a row. It shows up on the page automatically, named from "Zone name".
- **A magazine doesn't mail a zone:** leave that row out.
- **A size isn't sold in a zone:** leave that cell blank.
- Magazine can be `BDB` / `Best Deals Book` or `EHI` / `Exclusively Home Improvements`.

**Option A, edit the file:** open `rates.csv` in Excel, Google Sheets or Smartsheet, edit it, save it as CSV, and upload it in place of the old one. On GitHub that's: open the file, then the pencil (edit) or Add file → Upload files.

**Option B, live Google Sheet:** paste the sheet in, then use File → Share → Publish to web → choose the tab → **Comma-separated values (.csv)** → Publish. Put that link in `CONFIG.ratesUrl` in `index.html`, once. After that, edits to the sheet show up on the page within about 5 minutes.

Other pricing rules: "Both magazines" adds the two rates for each zone and takes 10% off. The volume discount applies on top. Ad design is free.

If the page is opened straight from disk (not from a web server), browsers block it from reading `rates.csv`, so it uses a built-in copy of the demo rates instead.

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
- **Prices, zones, homes:** replace the demo numbers in `rates.csv`.
- **Sold covers:** replace the example `CONFIG.soldOut` entries with real bookings.
- **Order webhook:** set `CONFIG.orderEndpoint`, and confirm that `salesEmail` is the right inbox.
- **Payment:** none is taken on the page. Once placement is confirmed, the customer is invoiced (an optional billing email can be entered at checkout).
