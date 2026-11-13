# Best Deals Ads checkout

Self-serve checkout for print ads in Best Deals Book and Exclusively Home Improvements, built in the same style as the Best Deals Direct mailer checkout.

It's a single static page, `index.html`. Open it in a browser; there's no build step.

**Flow:** 1 · Build your ad (magazine, ad size, zones, issue months, artwork) → 2 · Checkout (business, billing address, invoice details) → 3 · Confirmation.

## Updating rates and sold covers
Rates, zones, household counts and sold covers come from two CSV files next to the page: `rates.csv` and `sold.csv`. The team edits them in Excel and re-uploads them. **See [UPDATING-RATES.md](UPDATING-RATES.md)** for the steps and column rules.

If the files can't be loaded (for example, the page is opened straight from disk), the page uses built-in copies of the demo data.

Pricing rules: "Both magazines" adds the two rates for each zone and takes 10% off. The volume discount applies on top. Ad design is free.

## Issues and premium covers
Customers can pick any months over the next 12, not only consecutive ones. The volume discount is based on how many issues they book (`CONFIG.volume`).

Back Cover, Inside Front Cover and Inside Back Cover are one advertiser per zone, per issue. Covers listed in `sold.csv` show as "Taken in Zx" for that month. If a customer changes size, zone or magazine and a month they picked is no longer free, that month is removed and they're told why.

## Demo mode
`CONFIG.demo` is `true` right now. The whole flow works, but nothing is sent: no webhook call and no email. A yellow banner tells visitors it's a demo. Set `demo: false` once the order destination, prices and inventory are real.

## Where orders go (once demo is off)
When a customer presses **Request ad space**:
- **If `CONFIG.orderEndpoint` is set** (a Zapier, Make or Google Apps Script webhook, or your CRM), the order is POSTed there as JSON: reference, magazine, size, zones, months, full pricing breakdown and customer details. From there it can create a Notion row, send an email, or start the invoice. If the POST fails, the customer sees an error with the phone number and can try again.
- **If it's empty**, the customer's email app opens with the order pre-filled to `CONFIG.salesEmail` (advertise@bestdealsbook.com), and they have to press send. This is a stopgap; set a webhook before launch.

The confirmation page shows an order summary that the customer can print or save as a PDF. A hidden honeypot field drops most spam bots.

## Before launch
- **Prices, zones, homes:** replace the demo numbers in `rates.csv`.
- **Sold covers:** replace the example rows in `sold.csv` with real bookings.
- **Order webhook:** set `CONFIG.orderEndpoint`, and confirm that `salesEmail` is the right inbox.
- **Payment:** none is taken on the page. Once placement is confirmed, the customer is invoiced (an optional billing email can be entered at checkout).

## Publishing with GitHub Pages (free)
This repo is public, so GitHub can host the checkout for free:
1. On GitHub, go to the repo's **Settings → Pages**.
2. Under **Build and deployment**, set Source to **Deploy from a branch**, pick the branch (for example `main` once this is merged) and folder **/ (root)**, then **Save**.
3. After about a minute it's live at **https://hamstari.github.io/adcheckout/**.

To update rates, edit `rates.csv` / `sold.csv` on GitHub (open the file → pencil icon, or Add file → Upload files), then **Commit changes**. The site updates within about a minute.

**Optional: your own address.** In Settings → Pages → Custom domain, enter `ads.bestdealsnetworks.com`. Then in Wix go to Domains → your domain → Manage DNS records, add a **CNAME** with host `ads` pointing to `hamstari.github.io`, and tick **Enforce HTTPS** in GitHub once it's available.

**On Wix (keeps visitors on your site):**
1. In the Wix Editor, open the page (for example "Advertise"), then **Add Elements (+) → Embed Code → Embed a Site**.
2. Click **Change Website Address**, paste `https://hamstari.github.io/adcheckout/` (or `https://ads.bestdealsnetworks.com`), and **Update**.
3. Stretch the box to full width. Make it about **900px tall on desktop**.
4. Switch to the **mobile editor** and set the box to about **500–550px tall**, so it fits on a phone screen with the total bar visible.

The checkout scrolls inside the box. On desktop the summary and total stay pinned, and on phones a bar with the total and the Continue button stays at the bottom of the box. Visitors never leave bestdealsnetworks.com.

**On GoDaddy Website Builder:** paste the contents of `godaddy-embed.html` into an HTML section (Edit Website → Add Section → search "HTML" → Custom Code). The box is 900px tall on desktop and 560px on phones, and the checkout scrolls inside it.

## Publishing on your site
The checkout is three plain files with no build step: `index.html`, `rates.csv` and `sold.csv`.

**Option 1: put it in a folder on your site.** If your host lets you upload files (cPanel, FTP, WordPress file manager), create a folder such as `best-deals-ads` and upload `index.html`, `rates.csv` and `sold.csv`. The checkout is then at `bestdealsnetworks.com/best-deals-ads/`. You can publish it the same way the Best Deals Direct page is published.

**Option 2: host it free, then embed it.** For site builders (Squarespace, Wix, WordPress.com, etc.):
1. Go to https://app.netlify.com/drop and drag in a folder containing `index.html`, `rates.csv` and `sold.csv`. You get a link like `https://something.netlify.app`. (Optional: in Netlify, rename the site or connect `ads.bestdealsnetworks.com`.)
2. On your site, add a page and a Code / Embed / Custom HTML block. Paste the contents of `embed.html` into it and replace `https://YOUR-HOSTED-URL/index.html` with your Netlify link. The frame resizes itself to fit each step, so there's no inner scrollbar.

**Going live:** in `index.html`, set `demo: false` (this removes the yellow banner and starts sending orders) and set `orderEndpoint`. Rate changes only need `rates.csv` / `sold.csv` re-uploaded, not `index.html`.
