# Updating ad rates and sold covers

The ad checkout reads two spreadsheet files. To change prices or mark a cover as sold, edit the file in Excel and upload it again. You don't need to touch any code.

| File | What it controls |
|---|---|
| `rates.csv` | Prices, zones, zone names, homes per zone |
| `sold.csv` | Back covers and inside covers that are already booked |

## Steps
1. Download the current `rates.csv` or `sold.csv` from the website folder (or Netlify).
2. Open it in Excel and make your changes.
3. **File → Save As → CSV UTF-8 (Comma delimited) (.csv).** Keep the same file name. If Excel warns about features being lost, click **Yes / Keep current format**.
4. Upload the file in place of the old one, in the same folder as `index.html`.
5. Refresh the checkout page to check it.

## rates.csv
One row per magazine and zone. Prices are **per issue**.

| Magazine | Zone | Zone name | Homes | Back Cover | Inside Front Cover | Inside Back Cover | Full Page | Half Page | Quarter Page |
|---|---|---|---|---|---|---|---|---|---|
| BDB | Z1 | Zone 1 | 20000 | 1495 | 1195 | 1095 | 895 | 525 | 315 |
| EHI | Z1 | Zone 1 | 18000 | 1195 | 955 | 875 | 715 | 420 | 250 |

- **Change a price:** type the new number. `1495` and `$1,495` both work.
- **Add a zone:** add a row. "Zone name" is what customers see (for example *Sarasota*).
- **A magazine doesn't mail a zone:** delete that row.
- **A size isn't sold in a zone:** leave that cell empty.
- **Magazine:** `BDB` (Best Deals Book) or `EHI` (Exclusively Home Improvements).

## sold.csv
One row per booked cover. The cover then shows as **Taken** for that zone and month, so nobody else can book it.

| Magazine | Ad size | Zone | Month | Advertiser |
|---|---|---|---|---|
| BDB | Back Cover | Z1 | Jan 2027 | Sunshine Roofing |

- **Ad size:** `Back Cover`, `Inside Front Cover` or `Inside Back Cover`.
- **Month:** `Jan 2027` works. So does Excel changing it to `Jan-27` or `1/1/2027`.
- **Advertiser:** for your team only. Customers never see it.
- **Booking cancelled:** delete the row.

## Don't
- Rename the column headers.
- Rename the files.
- Save as `.xlsx`. It has to be `.csv`.

The demo rates and sold covers in these files now are examples. Replace them with real ones before going live.
