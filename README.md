# Ad Checkout

A self-serve advertising checkout page. Advertisers choose an ad size, one or more locations, and a run length, see their price instantly (with multi-month and multi-location discounts), and reserve their spot — no salesperson or rate sheet needed.

## Features

- **Ad sizes:** Front Cover, Back Cover, 2-Page Spread, Full Page, Half Page, Quarter Page
- **Locations:** per-location pricing and homes reached
- **Run length:** 1, 3, 6, or 12 months with term discounts
- **Artwork:** customer-supplied or "design it for me" add-on
- **Reserve:** collects contact info and generates a quote with a reference number to email or copy
- **Staff rate editor:** "Staff: edit rates" link in the footer (saves in that browser only)

## Editing rates permanently

All prices, locations, home counts, and discounts are **placeholders**. To change them for every visitor, edit the `DEFAULT_RATES` object near the top of the `<script>` in `index.html`:

```js
sizes:     [{ id, name, base, badge }]   // base = $ per location per month
locations: [{ id, name, homes, mult }]   // mult = location price multiplier
terms:     [{ months, disc }]            // disc = % off for that run length
multiLocation: [{ min, disc }]           // % off for N+ locations
designFee, contactEmail, contactPhone
```

Tip: adjust rates with the staff editor, click **Copy rates**, and paste the result over `DEFAULT_RATES`.

## Hosting

It's a single static `index.html` with no build step — works on GitHub Pages, or embedded in Wix via an HTML/iframe element.

## Note

Reservations are not submitted to a server; the advertiser emails or copies the generated quote. To capture leads automatically, wire the reserve form to a webhook (e.g. Zapier → Smartsheet/Pipedrive).
