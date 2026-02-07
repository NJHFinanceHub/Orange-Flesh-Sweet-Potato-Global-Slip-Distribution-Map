# 🌍 GIFT — Global OFSP Farmer Tracker

An interactive GIS tracking system that maps Orange Flesh Sweet Potato (OFSP) slip distribution to smallholder farmers worldwide. Built for the **Global Institute For Transformation (GIFT)** to showcase program impact to donors and funders.

![Status](https://img.shields.io/badge/status-active-green) ![License](https://img.shields.io/badge/license-MIT-blue)

## Features

- **Interactive Leaflet + OpenStreetMap Map** — Smooth pan/zoom with dark basemap optimized for data visualization
- **Google Sheets Integration** — Live data pull from a published Google Sheet (no backend required)
- **Privacy-First Design** — Individual farmer data is aggregated by community; donors see location + slip counts only
- **Auto-Generated Farmer IDs** — Sheet formulas automatically assign unique IDs (GIFT-0001, GIFT-0002, etc.)
- **Country Filtering** — Click country chips to isolate regions; map auto-zooms to filtered extent
- **Community Cards** — Sidebar cards with click-to-zoom navigation
- **Search** — Filter communities by name or country
- **Live Stats Dashboard** — Total communities, farmers, slips distributed, and countries at a glance
- **Responsive** — Works on desktop, tablet, and mobile
- **Embeddable** — Drop into any website via `<iframe>`
- **Zero Backend** — Static HTML deployed on GitHub Pages; data comes from Google Sheets

## Architecture

```
┌─────────────────┐      CSV/JSON       ┌──────────────────┐
│  Google Sheet    │ ──────────────────► │  index.html       │
│  (Data Source)   │   Published URL     │  (ArcGIS JS SDK)  │
└─────────────────┘                     │                    │
                                        │  ┌──────────────┐ │
                                        │  │ Parse CSV     │ │
                                        │  │ Aggregate by  │ │
                                        │  │ Community     │ │
                                        │  │ (Privacy)     │ │
                                        │  └──────┬───────┘ │
                                        │         │         │
                                        │  ┌──────▼───────┐ │
                                        │  │ ArcGIS Map   │ │
                                        │  │ + Markers    │ │
                                        │  │ + Popups     │ │
                                        │  │ + Filters    │ │
                                        │  └──────────────┘ │
                                        └──────────────────┘
                                              │
                                        GitHub Pages
                                              │
                                     ┌────────▼────────┐
                                     │  gifttransforms  │
                                     │  .org (iframe)   │
                                     └─────────────────┘
```

## Quick Start

> **Zero API keys required.** This app uses Leaflet.js + OpenStreetMap + CartoDB tiles — all 100% free with no usage limits, no accounts, no keys.

### 1. Set Up the Google Sheet

Create a Google Sheet with these exact column headers in Row 1:

| farmer_name | gender | latitude | longitude | date_distributed | slips_count | community | country |
|---|---|---|---|---|---|---|---|
| Jean-Baptiste Pierre | M | 18.4467 | -72.6342 | 2025-08-15 | 300 | Fondwa | Haiti |

**Auto-Generate Farmer IDs** — Add a `farmer_id` column with this formula in cell A2:
```
=CONCATENATE("GIFT-", TEXT(ROW()-1, "0000"))
```
Drag down to auto-generate IDs for all rows (GIFT-0001, GIFT-0002, etc.).

**Publish the Sheet:**
1. Open your Google Sheet
2. Go to **File → Share → Publish to web**
3. Select the specific sheet tab
4. Choose **Comma-separated values (.csv)**
5. Click **Publish** and copy the URL

The URL will look like:
```
https://docs.google.com/spreadsheets/d/e/2PACX-1v.../pub?gid=0&single=true&output=csv
```

### 2. Configure the App

Open `index.html` and update the `CONFIG` object at the top of the `<script>` section:

```javascript
const CONFIG = {
  googleSheetCsvUrl: 'https://docs.google.com/spreadsheets/d/e/YOUR_SHEET_ID/pub?gid=0&single=true&output=csv',
  useDemoData: false,  // Set to false once your sheet is connected
  googleSheetUrl: 'https://docs.google.com/spreadsheets/d/YOUR_SHEET_ID/edit',
};
```

### 3. Deploy

**Option A: GitHub Pages (Recommended)**

1. Push this repo to GitHub
2. Go to **Settings → Pages → Source** → Select "GitHub Actions"
3. The workflow will auto-deploy on every push to `main`
4. Your map will be live at `https://yourusername.github.io/gift-ofsp-farmer-tracker/`

**Option B: Any Static Host**

Upload `index.html` to any static hosting service (Netlify, Vercel, S3, etc.).

### 4. Embed on gifttransforms.org

Add this iframe to any page on the GIFT website:

```html
<iframe 
  src="https://yourusername.github.io/gift-ofsp-farmer-tracker/" 
  width="100%" 
  height="700" 
  frameborder="0" 
  style="border-radius: 12px; box-shadow: 0 4px 20px rgba(0,0,0,0.1);"
  title="GIFT Global OFSP Farmer Tracker"
></iframe>
```

In Duda (the website builder): use an **HTML Widget** to paste the iframe code.

## Google Sheet Template

Use `sample-data.csv` as a starting template. Import it into Google Sheets:

1. Open Google Sheets → **File → Import → Upload** → Select `sample-data.csv`
2. Add your real farmer data following the same column structure
3. GPS coordinates can be collected via smartphone (Google Maps → long-press → copy coordinates)

### Required Columns

| Column | Type | Description | Example |
|---|---|---|---|
| `farmer_name` | Text | Full name of the farmer | Jean-Baptiste Pierre |
| `gender` | Text | M or F | M |
| `latitude` | Number | GPS latitude (decimal degrees) | 18.4467 |
| `longitude` | Number | GPS longitude (decimal degrees) | -72.6342 |
| `date_distributed` | Date | YYYY-MM-DD format | 2025-08-15 |
| `slips_count` | Number | Number of OFSP slips given | 300 |
| `community` | Text | Village/community name | Fondwa |
| `country` | Text | Country name | Haiti |

### Privacy Note

The map **only displays aggregated community-level data**: location pin, community name, country, total farmer count, and total slips. Individual farmer names, genders, and exact GPS coordinates are never shown to donors. All aggregation happens client-side before rendering.

## Customization

### Adding New Countries

The color scheme auto-extends, but you can customize country colors in the `getCountryColor()` and `getCountryHex()` functions:

```javascript
function getCountryColor(country) {
  const colors = {
    'Haiti':       [232, 147, 47, 0.9],
    'Malawi':      [45, 138, 78, 0.9],
    'Philippines': [41, 128, 185, 0.9],
    'Nigeria':     [192, 57, 43, 0.9],
    'India':       [142, 68, 173, 0.9],
    // Add more countries here
    'Ethiopia':    [230, 126, 34, 0.9],
    'Brazil':      [39, 174, 96, 0.9],
  };
  return colors[country] || [128, 128, 128, 0.9];
}
```

### Changing the Basemap

Update `CONFIG.tileUrl` — all free, no key needed:
- `'https://{s}.basemaps.cartocdn.com/dark_all/{z}/{x}/{y}{r}.png'` — Dark (default)
- `'https://{s}.basemaps.cartocdn.com/light_all/{z}/{x}/{y}{r}.png'` — Light
- `'https://{s}.basemaps.cartocdn.com/rastertiles/voyager/{z}/{x}/{y}{r}.png'` — Voyager (colorful)
- `'https://{s}.tile.openstreetmap.org/{z}/{x}/{y}.png'` — Standard OSM
- `'https://{s}.tile.opentopomap.org/{z}/{x}/{y}.png'` — Topographic

## Tech Stack

- **Leaflet.js v1.9.4** — Map rendering, markers, popups (100% free, open source)
- **Leaflet.markercluster** — Smart clustering when zoomed out
- **CartoDB Dark Matter tiles** — Beautiful dark basemap (free, no key)
- **Google Sheets (Published CSV)** — Zero-cost, zero-backend data source
- **GitHub Pages** — Free static hosting with CI/CD
- **Vanilla HTML/CSS/JS** — No build step, no dependencies to manage

## File Structure

```
gift-ofsp-farmer-tracker/
├── index.html                    # Complete application (single file)
├── sample-data.csv               # Template for Google Sheet
├── README.md                     # This file
└── .github/
    └── workflows/
        └── deploy.yml            # GitHub Pages auto-deployment
```

## License

MIT — Built for the Global Institute For Transformation (GIFT).

## Links

- [GIFT Website](https://gifttransforms.org)
- [Leaflet.js Documentation](https://leafletjs.com/reference.html)
- [GIFT Donation Page](https://gifttransforms.org/donate)
