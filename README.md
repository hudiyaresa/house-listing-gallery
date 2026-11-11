# House Listing Gallery

A static gallery and map of houses for sale around Jakarta and Bekasi, collected while house hunting in late 2026. It brings together three sources in one page: Threads posts, a personal shortlist, and rumah123 listings, each with photos, price, size, and location.

Live site: https://hudiyaresa.github.io/house-listing-gallery/

## Table of Contents

1. [Features](#features)
2. [Data sources](#data-sources)
3. [How the data was built](#how-the-data-was-built)
4. [Location estimates](#location-estimates)
5. [Project structure](#project-structure)
6. [Run locally](#run-locally)
7. [Notes and limitations](#notes-and-limitations)

## Features

- **Gallery**: one card per house with a photo strip, price, land and building size, bedrooms, bathrooms, certificate, notes, and a link to the original post. Click any photo to open a full-screen viewer.
- **Map**: price pins clustered by area, styled like the rumah123 map. Each popup shows a photo, the price, a short summary, and contact or source links.
- **Exact vs estimated pins**: solid pins come from the seller's own map pin. Dashed pins marked `~` are estimates (see below).
- **Filters** shared by both views: free-text search, area, price band, source, "has photos", and sort order.
- **Reference circles**: 3 km around the Jatiwaringin toll gate and 4 km around the Ministry of Forestry office in Senayan.
- Works on a phone, and follows the system light or dark theme.

## Data sources

| Source | What it holds | Price filter |
| --- | --- | --- |
| Threads | Listing posts plus offers left as replies under "looking for a house" threads | up to Rp 600M |
| Shortlist | Houses noted during the search (seller ads and links only, private chat removed) | up to Rp 600M, plus a few above |
| rumah123 | Listings from a daily rumah123 scraper, re-checked one by one so removed listings and auctions are left out | under Rp 650M |

## How the data was built

The collection scripts are not in this repo because they run against my own accounts and private chats. In short:

1. **Threads**: every shared post was reopened with Playwright to get the full text and all replies, including nested ones. Reply pages arrive from GraphQL as `text/javascript`, so every `/graphql` response is parsed, not only JSON ones.
2. **Extraction**: regular expressions pull price, land and building size, bedrooms, bathrooms, floors, certificate, and phone numbers. The price is the number closest to words like *harga*, *jual*, or *cash*; numbers after *cicilan*, *DP*, *pasaran*, or *diskon* are ignored. Emoji digits and prices written in words were fixed by hand.
3. **Gaps**: listing photos were run through Windows OCR, and seller bios were checked for WhatsApp links.
4. **rumah123**: listings under Rp 650M were opened one by one (1.5 s apart). A 404 means the listing was removed, and it is dropped. Photos load straight from the rumah123 CDN.
5. **Page**: one static `index.html` with the data embedded, [Leaflet](https://leafletjs.com/) with markercluster for the map, and Esri basemaps.

## Location estimates

Only some rumah123 listings carry a precise seller pin. For everything else, the pin is an estimate, in this order:

1. The centre of the kelurahan, computed as the median of thousands of rumah123 pins in that area
2. A short hand-made list of areas that rumah123 rarely covers (Condet, Kranji, Kalisari, and so on)
3. Geocoding the address text with Nominatim
4. The city centre, as a last resort

Estimated pins get a small fixed offset so houses in the same area do not sit on top of each other. The popup always says how the location was found.

## Project structure

```
house-listing-gallery/
├── index.html        # gallery + map, data embedded
├── assets/
│   ├── thumbs/       # 520 px thumbnails
│   └── photos/       # 1280 px photos for the full-screen viewer
└── .nojekyll
```

## Run locally

Open `index.html` in a browser, or serve the folder:

```bash
python -m http.server 8000
```

Then go to http://localhost:8000. The map tiles and rumah123 photos need an internet connection.

## Notes and limitations

- Listing text is shown as posted, in Indonesian.
- Prices and sizes come from the posts and were not verified with sellers.
- Photos belong to the people who posted them.
- This is a snapshot from November 2026. Listings may have sold since then.
