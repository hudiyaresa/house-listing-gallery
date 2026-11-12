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
- **Cluster or housing complex**: houses inside a gated cluster or a housing complex get a badge in the title, with the estate name when the listing gives one (for example "As Salam (cluster)").
- **One card per house**: rumah123 houses that were posted several times, by the same agent or by different agents, show up once. The other postings stay as "Listing 2", "Listing 3" links, and the notes list the locations those ads used.
- **Filters** shared by both views: free-text search, area (multi-select), price band, source (multi-select), "has photos", "cluster/complex only", and sort order.
- **Reference circles**: 3 km around the Jatiwaringin toll gate and 4 km around the Ministry of Forestry office in Senayan.
- Works on a phone, and follows the system light or dark theme.

## Data sources

| Source | What it holds | Price filter |
| --- | --- | --- |
| Threads | Listing posts plus offers left as replies under "looking for a house" threads | up to Rp 810M |
| Shortlist | Houses noted during the search (seller ads and links only, private chat removed) | up to Rp 810M, plus a few above |
| rumah123 | Listings from a daily rumah123 scraper, re-checked one by one so removed listings and auctions are left out, repeat postings merged | up to Rp 810M |

## How the data was built

The collection scripts are not in this repo because they run against my own accounts and private chats. In short:

1. **Threads**: every shared post was reopened with Playwright to get the full text and all replies, including nested ones. Reply pages arrive from GraphQL as `text/javascript`, so every `/graphql` response is parsed, not only JSON ones.
2. **Extraction**: regular expressions pull price, land and building size, bedrooms, bathrooms, floors, certificate, and phone numbers. The price is the number closest to words like *harga*, *jual*, or *cash*; numbers after *cicilan*, *DP*, *pasaran*, or *diskon* are ignored. Emoji digits and prices written in words were fixed by hand.
3. **Gaps**: listing photos were run through Windows OCR, and seller bios were checked for WhatsApp links.
4. **Cluster or complex**: a house counts as inside a cluster when the text says *cluster*, *klaster*, *one gate*, or *townhouse*, and inside a housing complex for *komplek*, *perumahan*, *perum*, *perumnas*, *regency*, or *residence*, or when it mentions an estate fee (IPL) or 24-hour security. Mentions that only give directions ("near perumahan X") or deny it ("bukan cluster") are skipped. rumah123 listings also use the site's own "One Gate System" and "Komplek Perumahan" tags.
5. **rumah123**: listings up to Rp 810M were opened one by one (1.5 s apart) for the full description, photos, and seller pin. A 404 means the listing was removed, and it is dropped. Photos load straight from the rumah123 CDN.
6. **Repeat postings**: two rumah123 listings count as the same house when the agent, price, land size, and building size all match and the photos or text match too, or when they share a near-identical photo (64-bit dHash, at most 6 bits apart) with the same land size and a price within 10%. A photo shared by listings of many different sizes is treated as an agent logo or brochure and ignored. The card keeps the posting with the most photos.
7. **Page**: one static `index.html` with the data embedded, [Leaflet](https://leafletjs.com/) with markercluster for the map, and Esri basemaps.

## Location estimates

Only some rumah123 listings carry a precise seller pin. For everything else, the pin is an estimate, in this order:

1. The centre of the kelurahan, computed as the median of thousands of rumah123 pins in that area
2. A short hand-made list of areas that rumah123 rarely covers (Condet, Kranji, Kalisari, and so on)
3. Geocoding the address text with Nominatim
4. The city centre, as a last resort

Some agents post one house under several districts so it shows up in more searches, for example a house in Depok listed under Kuningan or Mampang in South Jakarta. So the listing text is checked too: the "Lokasi" or "Alamat" line in the description and "di <area>" in the title. Distance hints such as "near the Jatiasih toll gate" or "5 minutes to Depok station" are ignored. If none of those places matches the ad's location and the first one is more than 3 km away (5 km when the seller set a precise pin), the place from the text is used, and the card notes what the ad said.

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
