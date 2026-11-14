# Best Deals Ads checkout

Self-serve checkout for print ads in Best Deals Book and Exclusively Home Improvements, built in the same style as the Best Deals Direct mailer checkout.

It's a single static page, `index.html`. Open it in a browser; there's no build step.

**Flow:** 1 · Build your ad (magazine, ad size, zones, issue months, artwork) → 2 · Checkout (business, billing address, invoice details) → 3 · Confirmation.

## Pricing (2026 rate card)
Prices come from the rate card in `rates.csv`: **per issue, per magazine**, by ad size, number of zones (1, 2 or 3) and frequency (3x, 6x, 12 issues).

- **Frequency:** the rate column is set by how many different months the whole order runs in: 1–5 → 3x rate, 6–11 → 6x rate, 12+ → 12-issue rate. One or two issues are allowed at the 3x rate.
- **More than 3 zones:** the 3-zone price plus the price of the remaining zones, minus 15% (7 zones = 3 + 3 + 1, minus 15%). Set in `CONFIG.zoneBlock` / `CONFIG.extraZonesDiscount`.
- **Front Cover, Back Cover, 2-Page Full Spread:** one price per issue at any frequency, by number of zones (the card's 100K / 200K / 300K columns). The same more-than-3-zones rule is applied to them.
- **Both magazines:** each magazine is priced from the card and the two are added. There is no bundle discount.
- **Premium positions** (right-hand read, first ten pages, page 3, inside covers, centerfold): no amount is added; the checkout says "Upgrade charge TBD on invoice".
- Not used from the card: the half-zone / Charlotte County table and the 50K premium column.

## Staff portal: updating the rate card, zones and sold spots
Staff update everything at **`/staff/`** (for example `https://hamstari.github.io/adcheckout/staff/`). It isn't linked from the checkout and is hidden from search engines, but the page and the three CSV files are public.

- **Rate card:** the grid from the rate sheet. Edit any price.
- **Zones:** code, name, areas covered, homes, and which magazines mail there. Add or remove zones.
- **Sold spots:** one row per booked cover or one-per-zone position (front/back cover, inside covers, page 3, centerfold) with month, magazine and zone. It shows as **Taken** on the checkout. Double bookings are blocked.
- **Save & publish** writes `rates.csv`, `zones.csv` and `sold.csv` to `main`; the site redeploys in a couple of minutes. If two people save at once, the second is told to reload.

**Connecting the portal (one time, administrator):** until it's connected, the portal runs in demo mode (edit but not save).
1. On GitHub: Settings → Developer settings → Personal access tokens → **Fine-grained tokens** → Generate new token.
2. Repository access: **Only select repositories → adcheckout**. Permissions: **Contents: Read and write**.
3. In the portal, click **Connect to save** and paste the key.

The key can change any file in this repo, so give it only to trusted staff, set an expiry, and replace it when someone leaves.

## The request form (three steps)
**1 · Package.** Magazine, ad size, markets, issue months, placement and artwork for one ad at a time.
- Selected choices carry a "✓ Selected" label as well as color. Unavailable ones are dashed, struck through and say why.
- Markets show the zone name from `zones.csv`. When a zone has a real name (not just "Zone 1"), the zone code is shown beside it.
- Standard placement is always visible. Premium positions and the free-text placement request sit behind **Explore placement upgrades**. A chosen upgrade stays visible under that control and in the estimate. No fee is added; the estimate says "Premium placement fee to be confirmed".
- **+ Add another ad** (secondary) saves the ad and starts another. Saved ads keep their number and are never changed by editing a different ad.

**2 · Details.** Two groups: business and contact, then billing (billing email and notes are optional). Labels stay visible, phone/email/ZIP use the right keyboards and autofill, and errors appear under the field they belong to.

**3 · Confirmation.** Status, next steps (placement confirmation, DocuSign agreement, invoice, artwork) and the request summary.

**One estimate, one action.** The estimate is the only summary: each ad with size, markets, months, "price per issue × issues", Edit and Remove, an expandable monthly breakdown, and the estimated total. It sits beside the form on desktop and below it on phones, where the ad list is collapsed behind "Show estimate details". Each step has one primary button: **Continue to details**, then **Send ad request**. Nothing is pinned to the screen, so nothing can cover a field, an error or the last button inside an embed.

Entries survive going back, editing the package, a failed send and a page refresh (sessionStorage); they are cleared after a successful send.

Other rules that carry over:
- The frequency rate (3x / 6x / 12 issues) is set by the different months the whole package runs in, across all ads.
- A cover or one-per-market position already in the package can't be booked twice for the same market and month.
- **Issue deadline:** `CONFIG.issueCloseDay` (placeholder: 10) is the day of the month before an issue mails when booking closes.
- **Premium positions** are in `CONFIG.placements`. Covers can't take one; several are Full Page only.

## Issues and premium covers
Customers can pick any months over the next 12, not only consecutive ones. The number of months booked sets the frequency rate (see Pricing).

Back Cover, Inside Front Cover and Inside Back Cover are one advertiser per zone, per issue. Covers listed under **Sold covers** in the staff portal show as "Taken in Zx" for that month. If a customer changes size, zone or magazine and a month they picked is no longer free, that month is removed and they're told why.

## Preview mode and sending requests
`CONFIG.demo` is `true` right now: **preview mode**. Nothing is sent. The last step's main status is "Preview only — no request sent", with no reference number.

With `demo: false` the form is live:
- The request is POSTed as JSON (`Content-Type: text/plain`) to `CONFIG.orderEndpoint`: `requestId`, `ads[]`, `pricing`, `customer`.
- **"Request received" is shown only if the endpoint answers with a 2xx status.** The endpoint must therefore allow cross-origin requests from the form's address (CORS), or the browser reports a failure even if the request arrived.
- A reference is shown only if the endpoint's JSON reply contains `reference` (or `referenceId`, `ref`, `id`). The form never invents one.
- `requestId` is the same on a retry, so the receiving system can drop duplicates.
- While sending, the button is disabled and reads "Sending…"; extra clicks do nothing. Requests time out after 20 seconds.
- On failure the customer stays on Details with everything intact and a message to try again or call.
- With `demo: false` and no endpoint, sending fails with a "not connected" message. There is no email fallback any more, because an opened email draft can't be confirmed as sent.
- A hidden honeypot field drops most spam bots.

## Before launch
- **Prices:** the 2026 rate card is loaded. **Zone names, areas, homes and which zones each magazine mails** are still placeholders; set them in the staff portal.
- **Sold covers:** delete the example bookings in the staff portal and add real ones.
- **Staff portal:** create the access key (see above).
- **Order endpoint:** set `CONFIG.orderEndpoint` (must allow CORS and ideally return a `reference`), then set `demo: false`.
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

**On GoDaddy Website Builder:** paste the contents of `godaddy-embed.html` into an HTML section (Edit Website → Add Section → search "HTML" → Custom Code) and leave **Forced Height** empty. Its script resizes the frame to the form's height whenever a step or expandable section changes, and only accepts messages from the form's own origin. Whether the published page follows depends on GoDaddy: it wraps custom code in its own frame, and that wrapper has to grow too. If the form is cut off or leaves a gap, switch to `godaddy-embed-fixed.html` and set Forced Height to 760: a fixed box (600px on phones) that scrolls inside. Because nothing in the form is pinned, every field and the last button stay reachable in that box.

## Publishing on your site
The checkout is four plain files with no build step: `index.html`, `rates.csv`, `zones.csv` and `sold.csv`.

**Option 1: put it in a folder on your site.** If your host lets you upload files (cPanel, FTP, WordPress file manager), create a folder such as `best-deals-ads` and upload `index.html`, `rates.csv`, `zones.csv` and `sold.csv`. The checkout is then at `bestdealsnetworks.com/best-deals-ads/`. You can publish it the same way the Best Deals Direct page is published.

**Option 2: host it free, then embed it.** For site builders (Squarespace, Wix, WordPress.com, etc.):
1. Go to https://app.netlify.com/drop and drag in a folder containing `index.html`, `rates.csv`, `zones.csv` and `sold.csv`. You get a link like `https://something.netlify.app`. (Optional: in Netlify, rename the site or connect `ads.bestdealsnetworks.com`.)
2. On your site, add a page and a Code / Embed / Custom HTML block. Paste the contents of `embed.html` into it and replace `https://YOUR-HOSTED-URL/index.html` with your Netlify link. The frame resizes itself to fit each step, so there's no inner scrollbar.

**Going live:** in `index.html`, set `demo: false` (this starts sending orders) and set `orderEndpoint`. Rate changes are made in the staff portal and don't touch `index.html`.
