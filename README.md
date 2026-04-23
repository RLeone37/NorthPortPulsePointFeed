# Sarasota County CAD Dashboard

A live emergency incident dashboard pulling from the Sarasota County 911 Dispatch Reporting system. Incidents are displayed in real time with an interactive map, agency filters, and incident type filters. Designed for wall monitor display with a responsive layout that also works on mobile.

## What It Does

Fetches the public dispatch log from Sarasota County's 911 reporting site, parses the incident table, deduplicates multi-agency responses into single cards, and displays everything on an interactive dark-themed dashboard. The feed refreshes automatically every 60 seconds.

## Files

- `index.html` — The entire application. HTML, CSS, and JavaScript are contained in this single file.
- `worker.js` — A Cloudflare Worker that proxies the Sarasota County dispatch site to bypass browser CORS restrictions. Must be deployed separately on Cloudflare Workers.

## Data Source

Incident data is pulled from the Sarasota County 911 Dispatch Reporting site at `dispatchreporting.scgov.net`. This is the county's own CAD (Computer Aided Dispatch) system and represents the most direct public source available — updated faster than third-party aggregators which receive a secondary feed from the same system.

Note: The county dispatch site covers fire and rescue incidents only. EMS and medical calls are handled through a separate system and are not published on the public feed.

## Architecture

```
index.html  →  Cloudflare Worker  →  dispatchreporting.scgov.net
```

The dashboard is a static HTML file hosted on GitHub Pages. It calls the Cloudflare Worker, which fetches the county dispatch page server-side and returns the HTML. The dashboard then parses the table, deduplicates incidents, geocodes addresses, and renders everything in the browser.

## Features

- Live incident feed refreshing every 60 seconds
- Multi-agency deduplication — when multiple departments respond to the same call, a single incident card is shown with all responding agency tags displayed
- Interactive Leaflet map with geocoded incident markers, bounded to Sarasota and surrounding counties (Charlotte, Manatee, DeSoto, Hardee)
- Agency filters: All, North Port, SCFD, Venice, Englewood, Nokomis, Other
- Incident type filters: All, Today, Fire, Marine, Traffic, Alarms
- Incidents that do not match a specific category appear in the All view
- Side-by-side layout on wide screens (map left, incidents right), stacked automatically on mobile
- Command center styling with Rajdhani and Inter fonts, color-coded agency tags, and glowing stat numbers

## Incident Categories

| Filter | Matched Keywords |
|--------|-----------------|
| Fire | FIRE, BRUSH, EXPLOSION, STRUCTURE, SMOKE, HAZMAT, BURNING, ELECTRICAL |
| Marine | MARINE, WATER, BOAT, DROWNING, FLOOD, SWIFT WATER, SURF, DIVE, VESSEL |
| Traffic | TRAFFIC, CRASH, COLLISION, ACCIDENT, MVC |
| Alarms | ALARM, CARBON, MONOXIDE |
| Other | Anything not matched above — always visible in the All view |

## Geocoding

Addresses are geocoded using Nominatim (OpenStreetMap) with results bounded to the southwest Florida region. Incidents with only a bare street name (no number, no intersection) are listed but not mapped. Incidents that cannot be reliably located are also listed only, preventing incorrect marker placement.

## Hosting on GitHub Pages

1. Push `index.html` to your repository.
2. Go to Settings > Pages.
3. Set the source to your main branch, root folder.
4. GitHub will publish the dashboard at `https://yourusername.github.io/your-repo-name`.

## Cloudflare Worker Setup

1. Go to `dash.cloudflare.com` and navigate to Workers & Pages.
2. Create a new Worker and name it `sarasota-cad-proxy`.
3. Paste the contents of `worker.js` into the editor and deploy.
4. Your Worker URL will be `https://sarasota-cad-proxy.your-username.workers.dev`.
5. In `index.html`, confirm the `WORKER_URL` variable matches your deployed Worker URL.

The Cloudflare free tier allows 100,000 requests per day, which is well within range for a 60-second refresh cycle.

## Limitations

- The county dispatch site serves rendered HTML rather than a JSON API. Changes to the site layout could break the parser.
- Nominatim geocoding is free but rate-limited. When an address cannot be reliably found within the region, no marker is shown rather than an incorrect one.
- The county site typically shows the past 24 to 48 hours of incidents. Quieter agencies such as North Port may show no results if there have been no calls in that window.
