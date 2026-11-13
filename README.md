# Best Deals Ads checkout

Self-serve checkout for print ads in Best Deals Book and Exclusively Home Improvements, built in the same style as the Best Deals Direct mailer checkout.

It's a single static page, `index.html`. Open it in a browser; there's no build step.

**Flow:** 1 · Build your ad (magazine, ad size, zones, issue months, artwork) → 2 · Checkout (business, billing address, invoice details) → 3 · Confirmation.

## Staff portal: updating rates and sold covers
Staff update everything at **`/staff/`** (for example `https://hamstari.github.io/adcheckout/staff/`). It isn't linked from the checkout and is hidden from search engines.

- **Rates:** a grid per magazine. Edit prices and homes, rename zones (the name is what customers see), untick **Mails here** if a magazine doesn't mail a zone, leave a price empty if a size isn't sold there, and add or remove zones.
- **Sold covers:** add a booking (month, magazine, cover, zone, advertiser). It shows as **Taken** on the checkout. Delete it if cancelled. Double bookings are blocked.
- **Save & publish** writes `rates.csv` and `sold.csv` in this repo. GitHub Pages then updates the live checkout within a couple of minutes. If two people save at once, the second gets a message to reload, instead of overwriting the first person's changes.

**Connecting the portal (one time, administrator):** until it's connected, the portal runs in demo mode, where you can edit but not save. To connect it:
1. On GitHub, go to Settings → Developer settings → Personal access tokens → **Fine-grained tokens** → Generate new token.
2. Under Repository access, choose **Only select repositories → adcheckout**. Under Permissions, set **Contents: Read and write**.
3. In the portal, click **Connect to save** and paste the key. Tick "Remember on this computer" so staff only do this once per computer.

The key can change files in this repo, so only give it to trusted staff. Set an expiry and replace it when someone leaves. The portal saves to `main` (set in `SETTINGS` at the top of `staff/index.html`), which is the branch GitHub Pages publishes.

Pricing rules: "Both magazines" adds the two rates for each zone and takes 10% off. The volume discount applies on top. Ad design is free.

## Premium placement
Step 5 of the checkout offers Standard placement (included), **Right-hand read**, **Front of magazine** and **Centerfold** (`CONFIG.placements` in `index.html`). No price is shown or added to the total for these: each says "Upgrade charge TBD on invoice", and staff set the charge when invoicing. Covers can't take a placement upgrade, and Centerfold is Full Page only.

**Additional placement request** is a free-text add-on, also marked "Charge TBD on invoice".

When either is chosen, the summary, confirmation and order show the total "plus placement charge TBD", and the order data has `placementChargeTBD: true`. Premium positions aren't tracked as sold the way covers are, so staff confirm them for each issue.

## Issues and premium covers
Customers can pick any months over the next 12, not only consecutive ones. The volume discount is based on how many issues they book (`CONFIG.volume`).

Back Cover, Inside Front Cover and Inside Back Cover are one advertiser per zone, per issue. Covers listed under **Sold covers** in the staff portal show as "Taken in Zx" for that month. If a customer changes size, zone or magazine and a month they picked is no longer free, that month is removed and they're told why.

## Demo mode
`CONFIG.demo` is `true` right now. The whole flow works, but nothing is sent: no webhook call and no email. A yellow banner tells visitors it's a demo. Set `demo: false` once the order destination, prices and inventory are real.

## Where orders go (once demo is off)
When a customer presses **Request ad space**:
- **If `CONFIG.orderEndpoint` is set** (a Zapier, Make or Google Apps Script webhook, or your CRM), the order is POSTed there as JSON: reference, magazine, size, zones, months, full pricing breakdown and customer details. From there it can create a Notion row, send an email, or start the invoice. If the POST fails, the customer sees an error with the phone number and can try again.
- **If it's empty**, the customer's email app opens with the order pre-filled to `CONFIG.salesEmail` (advertise@bestdealsbook.com), and they have to press send. This is a stopgap; set a webhook before launch.

The confirmation page shows an order summary that the customer can print or save as a PDF. A hidden honeypot field drops most spam bots.

## Before launch
- **Prices, zones, homes:** replace the placeholder numbers in the staff portal.
- **Sold covers:** delete the example bookings in the staff portal and add real ones.
- **Staff portal:** create the access key (see above).
- **Order webhook:** set `CONFIG.orderEndpoint`, and confirm that `salesEmail` is the right inbox.
- **Payment:** none is taken on the page. Once placement is confirmed, the customer is invoiced (an optional billing email can be entered at checkout).

## Publishing with GitHub Pages (free)
The workflow in `.github/workflows/jekyll-gh-pages.yml` publishes the `main` branch to **https://hamstari.github.io/adcheckout/** every time `main` changes (Settings → Pages → Source must be **GitHub Actions**). A deploy takes a minute or two.

Rates and sold covers are updated from the staff portal (see above), which saves to `main` and so triggers a deploy.

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

**Going live:** in `index.html`, set `demo: false` (this removes the yellow banner and starts sending orders) and set `orderEndpoint`. Rate changes are made in the staff portal and don't touch `index.html`.
