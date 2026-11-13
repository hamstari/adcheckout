# Best Deals Ads checkout

Self-serve checkout for print ads in Best Deals Book and Exclusively Home Improvements, built in the same style as the Best Deals Direct mailer checkout.

It's a single static page, `index.html`. Open it in a browser; there's no build step.

**Flow:** 1 · Build your ad (magazine, ad size, zones, issue months, artwork) → 2 · Checkout (business, billing address, invoice details) → 3 · Confirmation.

## Updating rates and sold covers (Google Sheet)
Rates, zones, household counts and sold covers come from a spreadsheet. Nothing is hard-coded in the page.

**One-time setup**
1. In Google Drive: New → File upload → `Best Deals Ads - Rates.xlsx`, then open it with Google Sheets (or File → Save as Google Sheets).
2. Share → General access → **Anyone with the link** → **Viewer**. Copy the link.
3. Paste the link into `googleSheetUrl` near the top of `CONFIG` in `index.html`.

**After that** your team just edits the sheet, and changes show the next time someone loads the checkout.

- **Rates tab:** one row per magazine (BDB / EHI) and zone, with prices per issue for each ad size. Add a row to add a zone ("Zone name" is what customers see). Delete a row if a magazine doesn't mail that zone. Leave a cell blank if a size isn't sold there.
- **Sold covers tab:** one row per booked Back Cover, Inside Front Cover or Inside Back Cover, with its magazine, zone and month (for example `Jan 2027`). That cover then shows as "Taken" for that month and zone. Delete the row if the booking is cancelled.
- Don't rename the two tabs or their column headers. The "How to use" tab repeats these steps.

**Without a Google Sheet:** if `googleSheetUrl` is empty, the page reads `rates.csv` and `sold.csv` next to it (same columns). If neither can be loaded (for example, the page is opened straight from disk), it uses built-in copies of the demo data.

Other pricing rules: "Both magazines" adds the two rates for each zone and takes 10% off. The volume discount applies on top. Ad design is free.

## Issues and premium covers
Customers can pick any months over the next 12, not only consecutive ones. The volume discount is based on how many issues they book (`CONFIG.volume`).

Back Cover, Inside Front Cover and Inside Back Cover are one advertiser per zone, per issue. Covers listed on the **Sold covers** tab show as "Taken in Zx" for that month. If a customer changes size, zone or magazine and a month they picked is no longer free, that month is removed and they're told why.

## Demo mode
`CONFIG.demo` is `true` right now. The whole flow works, but nothing is sent: no webhook call and no email. A yellow banner tells visitors it's a demo. Set `demo: false` once the order destination, prices and inventory are real.

## Where orders go (once demo is off)
When a customer presses **Request ad space**:
- **If `CONFIG.orderEndpoint` is set** (a Zapier, Make or Google Apps Script webhook, or your CRM), the order is POSTed there as JSON: reference, magazine, size, zones, months, full pricing breakdown and customer details. From there it can create a Notion row, send an email, or start the invoice. If the POST fails, the customer sees an error with the phone number and can try again.
- **If it's empty**, the customer's email app opens with the order pre-filled to `CONFIG.salesEmail` (advertise@bestdealsbook.com), and they have to press send. This is a stopgap; set a webhook before launch.

The confirmation page shows an order summary that the customer can print or save as a PDF. A hidden honeypot field drops most spam bots.

## Before launch
- **Prices, zones, homes:** replace the demo numbers on the Rates tab.
- **Sold covers:** replace the example rows with real bookings.
- **Google Sheet:** do the one-time setup above.
- **Order webhook:** set `CONFIG.orderEndpoint`, and confirm that `salesEmail` is the right inbox.
- **Payment:** none is taken on the page. Once placement is confirmed, the customer is invoiced (an optional billing email can be entered at checkout).

## Publishing on your site
The checkout is plain files with no build step: `index.html`, plus `rates.csv` and `sold.csv` if you're not using the Google Sheet.

**Option 1: put it in a folder on your site.** If your host lets you upload files (cPanel, FTP, WordPress file manager), create a folder such as `best-deals-ads` and upload `index.html`, `rates.csv` and `sold.csv`. The checkout is then at `bestdealsnetworks.com/best-deals-ads/`. You can publish it the same way the Best Deals Direct page is published.

**Option 2: host it free, then embed it.** For site builders (Squarespace, Wix, WordPress.com, etc.):
1. Go to https://app.netlify.com/drop and drag in a folder containing `index.html`, `rates.csv` and `sold.csv`. You get a link like `https://something.netlify.app`. (Optional: in Netlify, rename the site or connect `ads.bestdealsnetworks.com`.)
2. On your site, add a page and a Code / Embed / Custom HTML block. Paste the contents of `embed.html` into it and replace `https://YOUR-HOSTED-URL/index.html` with your Netlify link. The frame resizes itself to fit each step, so there's no inner scrollbar.

**Going live:** in `index.html`, set `demo: false` (this removes the yellow banner and starts sending orders), set `googleSheetUrl`, and set `orderEndpoint`. Re-upload `index.html` after each change.
