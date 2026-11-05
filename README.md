# Generator Uplatnica

**Free, browser-based generator for Croatian HUB-3A payment slips (opće uplatnice) with a scannable PDF417 barcode.**

No installation, no sign-up, no ads, no backend. Everything runs in your browser: your data never leaves your machine.

🌐 **Live app:** https://chesssparks.github.io/GeneratorUplatnica/

---

## Table of contents

- [What it does](#what-it-does)
- [Features](#features)
- [How it works](#how-it-works)
- [Usage](#usage)
  - [Single slip](#single-slip)
  - [Bulk generation from CSV](#bulk-generation-from-csv)
- [CSV format](#csv-format)
- [PDF417 barcode format](#pdf417-barcode-format)
- [Tech stack](#tech-stack)
- [Project structure](#project-structure)
- [Getting started (development)](#getting-started-development)
- [Build and deployment](#build-and-deployment)
- [Chrome extension build](#chrome-extension-build)
- [API](#api)
- [Support](#support)

---

## What it does

In Croatia, bills and transfers are commonly paid with the **HUB-3A** form (the standard "opća uplatnica"). Modern banking apps can scan the **PDF417 barcode** printed on the slip and fill in every field automatically: IBAN, amount, reference number, description.

Generator Uplatnica lets anyone (individuals, associations, clubs, small businesses) create correctly formatted HUB-3A slips with a valid barcode. You can make one at a time or hundreds at once from a CSV file. Typical uses:

- Membership fees for associations and sports clubs (članarine)
- Invoices for freelancers and small businesses
- Rent, shared costs, tuition, or any recurring payment request

## Features

| Feature | Description |
|---|---|
| **Single slip** | Fill in a form and get a print-ready HUB-3A slip with a live preview. |
| **PDF417 barcode** | Generated to the HUB-3 standard (`HRVHUB30`), so mobile banking apps can scan it. |
| **CSV import** | Drag and drop a CSV file to generate many slips at once. |
| **CSV template** | Download a ready-made template with example rows (UTF-8 with BOM, so Excel shows Croatian characters correctly). |
| **ZIP download** | Export every slip from a CSV as an individual PNG, named after the payer, in one `.zip` archive. |
| **Print / PDF** | Print directly or "Save as PDF" through the browser's print dialog. |
| **Privacy-first** | 100% client-side: no server, no accounts, no tracking of entered data. |
| **SEO-ready** | Meta tags, Open Graph, Twitter cards, JSON-LD structured data, `sitemap.xml` and `robots.txt`. |

## How it works

```
 ┌──────────────┐      ┌──────────────┐      ┌──────────────────────┐
 │  SlipForm    │      │  CsvUpload   │      │                      │
 │ (one slip)   │      │ (PapaParse)  │      │                      │
 └──────┬───────┘      └──────┬───────┘      │                      │
        │ slip data           │ slip data[]  │                      │
        └──────────┬──────────┘              │                      │
                   ▼                         │                      │
            ┌─────────────┐                  │                      │
            │   App.vue   │──────────────────┤   Print (window.print)
            └──────┬──────┘                  │   ZIP (html2canvas + JSZip)
                   ▼                         │                      │
            ┌──────────────┐  bwip-js        │                      │
            │ PaymentSlip  │ ─────────► PDF417 barcode on <canvas>   │
            └──────────────┘                 └──────────────────────┘
```

1. Data comes either from the **form** (`SlipForm.vue`) or from a **CSV file** (`CsvUpload.vue`, parsed with PapaParse).
2. Each record is rendered by **`PaymentSlip.vue`**: the fields are laid over the HUB-3A background image (`src/assets/uplatnica.png`), and a HUB-3 payload string is encoded into a PDF417 barcode with **bwip-js**.
3. Output:
   - **Print / PDF:** `window.print()` with print-specific CSS.
   - **ZIP:** every slip is rasterised with **html2canvas** (2× scale) and packed into a ZIP with **JSZip**. These heavier libraries are lazy-loaded only when needed.

## Usage

### Single slip

1. Open the **Jedna uplatnica** (single slip) tab.
2. Fill in payer (platitelj) and recipient (primatelj) details.
3. Enter the amount. The amount field works like a cash register: type digits and they fill from the right (`1`, `5`, `0`, `0` gives `15,00`).
4. Choose the payment model (e.g. `HR00`, `HR01`, `HR99`) and enter the reference number (poziv na broj).
5. Click **Generate**, then print or save as PDF.

Required fields: **recipient IBAN**, **amount > 0**, and **payment description**.

### Bulk generation from CSV

1. Open the **CSV uvoz** (CSV import) tab.
2. Click **Download template** to get `predlozak-uplatnica.csv`.
3. Fill it in with Excel, LibreOffice or Google Sheets, then save it as CSV (UTF-8).
4. Drag the file onto the upload area, or click to browse.
5. All slips are rendered. **Print** them, or **Download ZIP** to get one PNG per payer.

## CSV format

The first row must be a header with these exact column names (order does not matter):

| Column | Meaning | Max length in barcode |
|---|---|---|
| `imePlatitelja` | Payer name | 30 |
| `adresaPlatitelja` | Payer street address | 27 |
| `postanskiBrojIMjestoPlatitelja` | Payer postal code and city | 27 |
| `imePrimatelja` | Recipient name | 25 |
| `adresaPrimatelja` | Recipient street address | 25 |
| `postanskiBrojIMjestoPrimatelja` | Recipient postal code and city | 27 |
| `ibanPrimatelja` | Recipient IBAN (spaces are stripped) | 21 |
| `iznosTransakcije` | Amount in EUR, e.g. `100,00` or `100.00` | 15 digits (cents) |
| `modelPlacanja` | Payment model, e.g. `HR00` (defaults to `HR99` if empty) | 4 |
| `pozivNaBroj` | Reference number | 22 |
| `sifraNamjene` | Purpose code, e.g. `SALA`, `OTHR` | 4 |
| `opisPlacanja` | Payment description | 35 |

Example:

```csv
imePlatitelja,adresaPlatitelja,postanskiBrojIMjestoPlatitelja,imePrimatelja,adresaPrimatelja,postanskiBrojIMjestoPrimatelja,ibanPrimatelja,iznosTransakcije,modelPlacanja,pozivNaBroj,sifraNamjene,opisPlacanja
Ivan Ivić,Ilica 1,10000 Zagreb,Udruga XYZ,Trg J.J.Strossmayera 1,31000 Osijek,HR1210010051863000160,"100,00",HR00,2026-001,SALA,Članarina 2026
```

> **Tip:** Amounts with a decimal comma (`100,00`) must be quoted in a comma-separated file. The downloadable template already does this.

Text longer than these limits is truncated in the barcode, as the HUB-3 specification requires. The printed fields are not cut off in the data, but very long text may not fit visually.

## PDF417 barcode format

The barcode encodes a newline-separated payload according to the **HUB-3 standard** (Hrvatska udruga banaka):

```
HRVHUB30                 ← header
EUR                      ← currency
000000000010000          ← amount in cents, zero-padded to 15 digits
Ivan Ivić                ← payer name
Ilica 1                  ← payer address
10000 Zagreb             ← payer postal code & city
Udruga XYZ               ← recipient name
Trg J.J.Strossmayera 1   ← recipient address
31000 Osijek             ← recipient postal code & city
HR1210010051863000160    ← recipient IBAN
HR00                     ← payment model
2026-001                 ← reference number
SALA                     ← purpose code
Članarina 2026           ← description
```

This is built in `buildBarcodeText()` in `src/components/PaymentSlip.vue` and rendered with `bwip-js` (`bcid: 'pdf417'`).

## Tech stack

- **[Vue 3](https://vuejs.org/)** (Composition API via `setup()`) on **Vue CLI 5**
- **[bwip-js](https://github.com/metafloor/bwip-js)**: PDF417 barcode rendering
- **[PapaParse](https://www.papaparse.com/)**: CSV parsing
- **[html2canvas](https://html2canvas.hertzen.com/)**: rendering slips to PNG
- **[JSZip](https://stuk.github.io/jszip/)**: building the ZIP archive
- **GitHub Actions + GitHub Pages**: CI/CD and hosting

## Project structure

```
GeneratorUplatnica/
├── .github/workflows/deploy.yml   # Build & deploy to GitHub Pages on push to main
├── public/
│   ├── index.html                 # SEO meta tags, Open Graph, JSON-LD
│   ├── robots.txt
│   └── sitemap.xml
├── src/
│   ├── main.js                    # App entry point
│   ├── App.vue                    # Layout, tabs, hero, print & ZIP logic
│   ├── assets/
│   │   └── uplatnica.png          # HUB-3A slip background
│   └── components/
│       ├── SlipForm.vue           # Single-slip form + validation
│       ├── CsvUpload.vue          # CSV drag & drop, parsing, template download
│       └── PaymentSlip.vue        # Slip rendering + PDF417 barcode
├── vue.config.js                  # publicPath for Pages / extension builds
├── babel.config.js
└── package.json
```

## Getting started (development)

**Requirements:** Node.js 18+ and npm.

```bash
# Clone
git clone https://github.com/ChessSparks/GeneratorUplatnica.git
cd GeneratorUplatnica

# Install dependencies
npm install

# Start the dev server with hot reload (http://localhost:8080)
npm run serve

# Lint and fix files
npm run lint
```

## Build and deployment

```bash
npm run build
```

The production build goes to `dist/` with `publicPath` set to `/GeneratorUplatnica/` (see `vue.config.js`), which matches the GitHub Pages URL.

**Automatic deployment:** every push to `main` triggers `.github/workflows/deploy.yml`, which installs dependencies, builds the app, and publishes `dist/` to the `gh-pages` branch with `peaceiris/actions-gh-pages`. You can also start it by hand from the Actions tab (`workflow_dispatch`).

## Chrome extension build

```bash
npm run build:extension
```

This builds with relative asset paths (`publicPath: './'`) and writes the output to `../ChromeExtensionApp/app`, a sibling folder that holds the Chrome extension wrapper. That folder must exist next to this repository.

## API

A HUB-3A **REST API** is available **on request**, for integrating slip generation into ERPs, webshops, accounting software and similar systems:

```
POST /api/v1/uplatnica            # create a slip from JSON
GET  /api/v1/uplatnica/:id.png    # get it as PNG
GET  /api/v1/uplatnica/:id.pdf    # get it as PDF
POST /api/v1/batch                # generate many slips in one call

Authorization: Bearer <api-key>
```

It includes Bearer authentication, rate limiting and SLA support. To ask about access, use the **API** tab in the app or email **nevendavidovic407@gmail.com**.

## Support

The project is free and ad-free. If it saves you time, you can support development:

☕ **[Buy me a coffee](https://buymeacoffee.com/sparkschess)**

Bug reports and suggestions are welcome through [GitHub Issues](https://github.com/ChessSparks/GeneratorUplatnica/issues).
