# Conference Tracker — Map Maintenance Guide

This tracker now includes a filtered locations map.

The map works from static files only. No API keys are required.

## What must be maintained

There are 2 data sources that must stay aligned:

1. `conference-tracker/index.html`
   - Conference records live in the `EVENTS` array.
   - Each event has a `location` string.

2. `conference-tracker/locations.json`
   - Map coordinate index.
   - Keys are location strings.
   - Values include `lat` / `lng` and metadata.

If an event location exists in `EVENTS` but not in `locations.json`, the event still appears in list/filter/export, but it will be **Unmapped** on the map.

## Workflow when adding a conference

1. Add the event to `EVENTS` in `index.html`.
2. Check the exact `location` value.
3. If that exact location key does not exist in `locations.json`, add it.
4. Reload the page and verify the map shows the location.

## Location format rules (important)

- Use stable, human-readable location keys.
- Keep formatting consistent (examples: `Washington, DC`, `Arlington, VA`, `Paris, France`).
- Key matching is exact.
  - `Ft. Belvoir, VA` and `Fort Belvoir, VA` are different keys.
  - Extra spaces or punctuation differences break mapping.

## `locations.json` entry format

Example:

```json
"Arlington, VA": {
  "country": "US",
  "label": "Arlington County, Virginia, United States",
  "lat": 38.8769326,
  "lng": -77.0893094,
  "query": "Arlington, VA",
  "source": "nominatim",
  "state": "VA"
}
```

Recommended fields:
- `lat` (number)
- `lng` (number)
- `country` (2-letter country where possible)
- `state` (US state code when US)
- `query` (original lookup string)
- `label` (resolved label)
- `source` (`nominatim` or `manual`)

## Fast way to get coordinates

Option A (quick/manual):
- Use a map provider (OpenStreetMap/Google Maps), copy lat/lng, and add a `manual` record.

Option B (OSM Nominatim):
- Query by place name and copy best result coordinates.
- Keep source as `nominatim`.

## Validation checklist before merge

- [ ] Page loaded from HTTP(S) (not `file://`).
- [ ] New event appears in list/search/filter.
- [ ] New event appears on map at expected location.
- [ ] `Mapped locations` count looks correct.
- [ ] `Unmapped` count did not unexpectedly increase.

### Optional local consistency check

Run from repo root:

```bash
python3 - <<'PY'
import re, json
from pathlib import Path
idx = Path('conference-tracker/index.html').read_text()
locs = set(re.findall(r"location:'([^']*)'", idx))
coords = json.loads(Path('conference-tracker/locations.json').read_text())['locations']
missing = sorted([l for l in locs if l and l != 'TBD' and l not in coords])
print('Missing locations:', len(missing))
for m in missing:
    print('-', m)
PY
```

## Troubleshooting

- **Map says `Loading location index…` forever**
  - `locations.json` failed to load (path/hosting issue).

- **Map is blank under `file://`**
  - Expected. `fetch()` requires HTTP(S) in this setup.

- **Point is in wrong place**
  - Replace coordinates in `locations.json`.
  - Prefer venue-level coordinates if city-level is too coarse.

- **State click-to-zoom not working**
  - Check `us-states.geojson` exists in `conference-tracker/`.
