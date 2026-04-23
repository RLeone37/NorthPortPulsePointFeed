# Sarasota County CAD Dashboard

A live emergency incident dashboard pulling from the Sarasota County 911 Dispatch Reporting system. Incidents are displayed in real time with an interactive map, agency filters, and incident type filters. Designed for wall monitor display with a responsive layout that also works on mobile.

## What It Does

Fetches the public dispatch log from Sarasota County's 911 reporting site, parses the incident table, deduplicates multi-agency responses into single cards, and displays everything on an interactive dark-themed dashboard. The feed refreshes automatically every 60 seconds.

## Files

- `index.html` — The entire application. HTML, CSS, and JavaScript are contained in this single file.
- `worker.js` — A Cloudflare Worker that proxies the Sarasota County dispatch site to bypass browser CORS restrictions. Must be deployed separately on Cloudflare Workers.

## Data Source

Incident data is pulled from the Sarasota County 911 Dispatch Reporting site at `dispatchreporting.scgov.net`. This is the county's own CAD (Computer Aided Dispatch) system, making it the most direct and up-to-date public source available. Because browsers cannot fetch cross-origin pages directly, the Cloudflare Worker handles the request server-side and passes the data back to the dashboard.

## Architecture

```
index.html  →  Cloudflare Worker  →  dispatchreporting.scgov.net
```

The index.html is a static file hosted on GitHub Pages. It calls the Cloudflare Worker, which fetches the county dispatch page and returns the HTML. The dashboard then parses the table, deduplicates incidents, geocodes addresses, and renders everything.

## Features

- Live incident feed refreshing every 60 seconds
- Multi-agency deduplication — when multiple departments respond to the same call, they appear as one incident card with all agency tags shown
- Interactive Leaflet map with geocoded incident markers, bounded to Sarasota and surrounding counties (Charlotte, Manatee, DeSoto, Hardee)
- Agency filters: All, North Port, SCFD, Venice, Englewood, Nokomis, Other
- Incident type filters: All, Today, Fire, Medical, Traffic, Alarms
- Medical calls shown in the list without map markers (addresses not disclosed for privacy)
- Intersections handled in geocoding — cross-street addresses are formatted and searched correctly
- Side-by-side layout on wide screens (map left, incidents right), stacked automatically on mobile
- Command center styling with color-coded agency tags and glowing stat numbers

## Geocoding

Addresses are geocoded using Nominatim (OpenStreetMap), with results bounded to the southwest Florida region. Incidents that cannot be reliably located are listed but not plotted on the map, preventing incorrect marker placement.

## Hosting on GitHub Pages

1. Push `index.html` to your repository.
2. Go to Settings > Pages.
3. Set the source to your main branch, root folder.
4. GitHub will publish the dashboard at `https://yourusername.github.io/your-repo-name`.

## Cloudflare Worker Setup

1. Go to `workers.cloudflare.com` and create a new Worker.
2. Paste the contents of `worker.js` into the editor and deploy.
3. Copy the generated Worker URL (e.g. `https://your-worker.your-name.workers.dev`).
4. In `index.html`, set the `WORKER_URL` variable near the top of the script section to your Worker URL.

## Limitations

- The county dispatch site serves rendered HTML rather than a JSON API. Changes to the site's layout could break the parser.
- Medical incident addresses are withheld by the dispatch system for privacy and are not mapped.
- Nominatim geocoding is free but rate-limited. If an address cannot be found, no marker is shown rather than an incorrect one.
- The Cloudflare Worker free tier allows 100,000 requests per day, which is well within range for a 60-second refresh cycle.
