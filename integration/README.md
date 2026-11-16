# Feeding DocuSign and Smartsheet

The checkout's Details pages ask the same questions as the **BDB / EHI Advertising Agreement** (page 4 of `BDB_Agreement_-_2025.pdf`), and every request it sends carries two ready-made blocks:

| Block | What it is | Use it for |
|---|---|---|
| `agreementTabs` | One value per field on the agreement, keyed by a DocuSign tab label. Checkboxes are `true` / `false`. | Prefilling the DocuSign template |
| `smartsheetRow` | One flat row keyed by Smartsheet column name | Adding the request to a Smartsheet |

See `sample-order.json` for a full example request (Full Page in both magazines, Zones 1 and 3, Jan–Mar, separate billing address).

## Files
- `docusign-tabs.csv`: every tab label, its type, where it sits on the agreement, and an example value. The last eight rows (payment preference, card, signatures, initial) are **not** prefilled. The advertiser fills them in DocuSign.
- `smartsheet-columns.csv`: every column, the suggested column type, and an example. Create the sheet with these exact column names so Zapier maps them in one pass.
- `sample-order.json`: what the checkout POSTs.

## How the order fills the agreement
- **Top block:** Company, DBA, LIC#, address, phone, mobile, email, main contact, sales and marketing manager come straight from the Details pages.
- **Product rows:** each ad is placed on the agreement row for its magazine and zones: Best Deals Books (Zones 1, 2, 3, 5, 6, 7), Best Deals Books Zone 4, and Exclusively Home Improvements (Zones 1, 2, 3, 5). "Both magazines" fills the BDB and EHI rows. The zone boxes are checked, Ad Size lists the size(s), Total per Mo. is the per-issue price for that row, and Monthly Rate per Zone is that total divided by the zones checked.
- **Monthly Total:** the sum of the rows. If a package has ads that run in different months, the monthly amounts vary. Notes then lists every ad and the month-by-month amounts, and always ends with the contract total and the request ID.
- **Publication Term:** 3X, 6X or 12X from the rate tier. **Issue Start** is the first month booked, and the month boxes are the months booked.
- **NEXT LEAD 360** isn't sold in the checkout, so it is left empty.
- **No card numbers pass through the checkout.** Card #, expiry, payment preference and signatures are entered by the advertiser in DocuSign.

The checkout's zones file (`zones.csv`) now uses the agreement's zone names: Pinellas, Hillsborough, Sarasota/Manatee, Charlotte, Lee/Collier, Orlando North, Orlando South.

## One-time setup

### 1. DocuSign template
1. In DocuSign: **Templates → New → Create Template**, upload page 4 (and page 5 for terms and the initial) of the agreement.
2. Add two roles: **Advertiser** (signer 1) and **Publisher** (signer 2, someone at BDB).
3. Drag a field onto each blank and set its **Data Label** to the tab label in `docusign-tabs.csv` (for example `Company`, `BDB_Zone3`, `Month_FEB`). Use Text for text and numbers, Checkbox for true/false.
4. For the Advertiser role add Payment Preference (radio), Card #, EXP DATE, Billing ZIP, Signature, Date Signed and the page 5 Initial, all required. Card # can be set to *Conceal value*. If you have DocuSign Payments, a payment field is better than typed card numbers.

### 2. Smartsheet
Create a sheet with the columns in `smartsheet-columns.csv` (Request ID as the primary column).

### 3. Zapier (Professional plan)
1. **Trigger: Webhooks by Zapier → Catch Raw Hook.** Copy the hook URL into `CONFIG.orderEndpoint` in `index.html` and set `demo: false`. Add a **Code by Zapier** step that runs `return JSON.parse(inputData.raw)` so every field becomes mappable. Catch Hook has to be raw because the form sends `text/plain`.
2. **Smartsheet → Add Row.** Map each column to `smartsheetRow.<Column name>`.
3. **DocuSign → Create Signature Request** (from template). Choose the template. Advertiser = `smartsheetRow.Main Contact` / `smartsheetRow.Email`, Publisher = your sales manager. Map each prefill tab to `agreementTabs.<Tab label>`. Checkbox tabs take `true` / `false`.
4. **Smartsheet → Update Row.** Write the envelope ID to *DocuSign Envelope ID* and set Status to *Agreement sent*.
5. A second Zap, **DocuSign → Envelope Sent / Completed**, then **Smartsheet → Find Row** (by DocuSign Envelope ID), then **Update Row**, keeps *DocuSign Status* and *Status* current (Signed, Declined).

Optional: send the envelope only after staff approve placement. In that case stop the first Zap after step 2 and start the DocuSign steps from a Smartsheet trigger (Status changed to *Approved*).

### Notes
- Zapier's catch-hook answers `{"status":"success","id":"…"}`, so the checkout shows the confirmation. The `id` isn't a request reference, so remove `j.id` from the reference list in `submitOrder` if you don't want it shown to the customer.
- Before going live, the agreement terms (renewal, cancellation and payment due dates) should match what the checkout and the confirmation page say.
