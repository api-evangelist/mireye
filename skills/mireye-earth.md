---
name: mireye-earth
version: 0.7.1
description: Provenance-tagged geospatial data for any US coordinate. Ask natural-language questions, fetch named fields by name or preset, and discover the catalog — every value carries source provenance.
homepage: https://mireye.com
docs: https://docs.mireye.ai
metadata: {"api_base": "https://api.mireye.com"}
---

# Mireye Earth

You are an AI agent. Mireye Earth gives you authoritative, citation-backed geospatial data for any US coordinate — terrain and soils, flood and wildfire risk, land cover, buildings and roads, the electric grid and gas network, water supply, solar and wind resource, natural hazards (seismic, wind, tornado, hail, landslide), parcels, and political boundaries. Every value comes with the source identifier, the upstream URL, the dataset vintage, the timestamp, and a confidence rating. You can ask a free-text question, fetch named fields directly, or pull the full catalog.

**Base URL:** `https://api.mireye.com`
**Docs:** [docs.mireye.ai](https://docs.mireye.ai)
**Field catalog (machine-readable):** `GET /v1/meta/fields`
**Full docs index for LLMs:** [docs.mireye.ai/llms.txt](https://docs.mireye.ai/llms.txt)

---

## Before You Start

Three scenarios — know which one you're in:

1. **You know the exact fields you want** (e.g. `elevation`, `within_floodplain_polygon`). Skip to [`/v1/fetch`](#post-v1fetch-named-fields-with-provenance). Deterministic, 1–3 s warm.
2. **You have a question but don't know which fields answer it** (e.g. "Is this property at flood risk?"). Use [`/v1/ask`](#post-v1ask-natural-language-questions). A planner picks the fields, a synthesizer writes a cited prose answer. 2–6 s warm, up to 60 s on cold start. Want it token-by-token? Use [`/v1/ask/stream`](#post-v1askstream-streaming-answers).
3. **You're not sure what's even available.** Call [`GET /v1/meta/fields`](#get-v1metafields-discover-the-catalog) once at startup to enumerate all 250+ fields, their units, sources, and the 15 preset bundles. ETag-cached — re-fetch with `If-None-Match` is free.

---

## How It Works

Mireye Earth is a thin orchestrator over authoritative geospatial sources. When you request fields, Mireye fetches them in parallel from their respective upstreams, wraps each value with provenance metadata, and returns the bundle. Sources span federal agencies — USGS, NOAA, USDA, USFS, USFWS, FEMA, EPA, EIA, NREL, LBNL, FHWA, FAA, FCC, BTS, US Census, HUD, BLM, and USACE — plus open and commercial datasets like Sentinel-2, JRC Global Surface Water, Overture Maps, Regrid, USWTDB, and USPVDB.

### Core operations

- **`POST /v1/ask`** — natural-language Q&A. You send a coordinate + question; a planner LLM picks the relevant catalog fields, Mireye fetches them, a synthesizer LLM writes a cited prose answer. Use this when you don't know the schema in advance. `POST /v1/ask/stream` streams the same answer over Server-Sent Events.
- **`POST /v1/fetch`** — structured field access. You name the fields (or a preset bundle); Mireye returns each one with `value`, `unit`, `source`, `source_url`, `confidence`, `dataset_vintage`, `fetched_at`, `ttl_seconds`, `status`, and `notes`. Use this when you know what you want.
- **`GET /v1/meta/fields`** — the machine-readable catalog of every field and preset. Public, no auth, ETag-cached.

All three share the same 175-field catalog and the same provenance shape on every value.

There's also a **Sites** surface (`/v1/sites`, `/v1/ask-site`) for registering a named location once and asking repeated questions against it — see [Sites](#sites-persistent-locations) below.

### Resource hierarchy

```
Catalog (GET /v1/meta/fields)
├── 175 named fields across 7 layers
│   ├── Terrain           (elevation, slope, soils, hydrology, wetlands, coast, floodplain)
│   ├── Land Cover        (LCMS, land use, NLCD canopy, NDVI, USDA CDL)
│   ├── Built Environment (buildings, roads, bridges, turbines, solar facilities, opportunity zones)
│   ├── Utilities & Energy (power plants, transmission, substations, gas, water, broadband, interconnection queue, prices)
│   ├── Climate & Resource (solar GHI/DNI/PV yield, wind speed/density/CF, temperature, humidity, drought, snow)
│   ├── Hazards           (seismic, design wind, tornado, hail, lightning, landslide, dams, brownfields, superfund, air quality)
│   └── Parcels & Boundaries (divisions, Census, PAD-US protected areas, critical habitat, easements, Regrid parcels)
└── 14 presets that expand to curated field bundles
    └── terrain, flood_risk, wildfire_underwrite, land_cover, site_selection,
        building_lookup, utilities, boundaries, natural_hazard, grid_interconnect,
        data_center_siting, solar_siting, wind_siting, storage_siting
```

### Coverage

US only. Accepted envelope: `lat ∈ [18, 72]`, `lng ∈ [-180, -65]`. Covers CONUS, Alaska, Hawaii, and US territories. Out-of-bounds requests return `400 coord_out_of_bounds`. Pull the live envelope from `GET /v1/meta/fields` under `us_envelope` if you want to validate client-side.

---

## Quick Start

### Step 1: Pick a coordinate

Anywhere in the US. For this guide we'll use lower Manhattan:

```
lat = 40.7128
lng = -74.0060
```

### Step 2: Ask a question

```bash
curl -s https://api.mireye.com/v1/ask \
  -H 'authorization: Bearer YOUR_MIREYE_TOKEN' \
  -H 'content-type: application/json' \
  -d '{
    "lat": 40.7128,
    "lng": -74.0060,
    "question": "Is this property in a flood zone?"
  }' | jq
```

**Response (abridged):**

```json
{
  "answer": "This Manhattan address is not currently in a designated 100-year floodplain per FEMA NFHL data, but it sits 412 m from the East River shoreline at an elevation of only 13.15 m.",
  "confidence": "high",
  "citations": [
    {
      "source": "FEMA_NFHL",
      "source_url": "https://hazards.fema.gov/femaportal/wps/portal/NFHLWMS",
      "fields": ["within_floodplain_polygon"],
      "fetched_at": "2026-06-24T22:00:00Z",
      "confidence": "high"
    }
  ],
  "fields_used": ["within_floodplain_polygon", "elevation", "coast_distance_m"]
}
```

The `answer` is prose. The `citations` array is the audit trail — one entry per source used. `fields_used` lists the catalog field names the answer depends on; use it to re-verify deterministically via `/v1/fetch`.

### Step 3: Fetch raw fields directly

When you know the field names, skip the planner and ask for them by name:

```bash
curl -s https://api.mireye.com/v1/fetch \
  -H 'authorization: Bearer YOUR_MIREYE_TOKEN' \
  -H 'content-type: application/json' \
  -d '{
    "lat": 40.7128,
    "lng": -74.0060,
    "fields": ["elevation", "coast_distance_m", "within_floodplain_polygon"]
  }' | jq
```

Each field comes back as a self-contained record with its value, unit, source, source URL, confidence, dataset vintage, fetch timestamp, TTL, and status.

### Step 4: Use a preset

When you want a curated bundle for a common workflow, pass a preset name instead of (or in addition to) a `fields` array:

```bash
curl -s https://api.mireye.com/v1/fetch \
  -H 'authorization: Bearer YOUR_MIREYE_TOKEN' \
  -H 'content-type: application/json' \
  -d '{"lat": 29.7604, "lng": -95.3698, "preset": "flood_risk"}' | jq '.fields | keys'
```

Fourteen presets ship today: `terrain`, `flood_risk`, `wildfire_underwrite`, `land_cover`, `site_selection`, `building_lookup`, `utilities`, `boundaries`, `natural_hazard`, `grid_interconnect`, `data_center_siting`, `solar_siting`, `wind_siting`, `storage_siting`. See [Presets](#presets) below for full expansions.

### Step 5: Discover the catalog

Once, at startup, fetch the machine-readable catalog (no auth required):

```bash
curl -s https://api.mireye.com/v1/meta/fields | jq
```

Cache the response body and the `ETag` header. On subsequent boots, send `If-None-Match: <etag>` — a `304 Not Modified` means your cached copy is still valid. The catalog only changes on schema-version bumps (field renames, type changes, new fields), so a single fetch at startup is usually enough.

You're done. The rest of this document is reference.

---

## Rules

### Coordinate scope

- **US only.** Mireye is built on US-focused source datasets. Coordinates outside `lat ∈ [18, 72]`, `lng ∈ [-180, -65]` return `400 coord_out_of_bounds`. If a human gives you a non-US location, tell them Mireye doesn't cover it — don't fabricate.
- **Coordinates are in decimal degrees, WGS84.** Latitude before longitude. `+` for north and east, `-` for south and west.

### Provenance and confidence

- **Every value carries provenance.** Never present a Mireye value to a human without surfacing at least the `source` and ideally the `source_url`, `dataset_vintage`, and `fetched_at`. That's the whole point of using Mireye instead of guessing.
- **Respect confidence ratings.** Each value is `high`, `medium`, `low`, or `unknown` (lowercase). For regulatory, underwriting, or audit workflows, filter to `high` before quoting a value as definitive. `low` should be flagged for human review.
- **Read the per-field `status`.** `/v1/fetch` stamps each field with `status: "ok"` (a real value is present) or `status: "absent"` (the source confirmed there's *no* value at that coordinate — e.g., asking for a crop class on a city block). An `absent` field is **not a failure**: `value` is `null`, `confidence` is `unknown`, and `notes` explains why. Don't retry — the answer is "no data exists here." Fields that genuinely *failed* to fetch never appear under `fields`; they go to `partial_failures`.
- **Check `partial_failures` on every `/v1/fetch` response.** A 200 can still contain failed fields. If `retryable: true`, the source had a transient issue — retry. If `retryable: false`, the source returned a permanent error — don't. (`/v1/ask` has no `partial_failures` array — its prose answer flags any field it couldn't get.)

### Honesty about data

- **Don't fabricate values Mireye didn't return.** If a field is `absent`, missing, or in `partial_failures`, say so explicitly. The cited prose answer from `/v1/ask` already does this — preserve that behavior when summarizing for a human.
- **Don't drop citations.** When you summarize a Mireye result back to a human, keep at least the source names. The citation chain is the trust contract.

---

## Authentication

`/v1/meta/fields` is public so agents can inspect the catalog without a token. `/v1/ask`, `/v1/ask/stream`, `/v1/fetch`, and the Sites endpoints require a Mireye bearer token:

```
Authorization: Bearer YOUR_MIREYE_TOKEN
```

Three ways to get one:

- **Local MCP (Claude Desktop, Cursor, custom agents):** run `mireye-mcp login` once — a device flow prints a verification URL and code, and stores a token locally. Or set `MIREYE_BEARER_TOKEN`.
- **Hosted MCP (Claude Code):** point your client at `https://api.mireye.com/mcp` and complete the browser OAuth 2.1 + PKCE flow — no manual token handling.
- **Direct HTTP:** create an API token from the Mireye account settings page and send it as a bearer header.

**401 vs 403:** `401` means no token, or the token is expired/revoked — re-authenticate. `403` means the token is valid but the account is not permitted by backend policy — that's an account issue, not a bad key.

### Request correlation

Every response includes an `X-Request-ID` header. If you supply one yourself, Mireye echoes it back unchanged — useful for correlating your application logs against Mireye's server logs. When reporting a `500 internal` error, always include the `X-Request-ID` value, the request body, and the approximate UTC timestamp.

---

## API Reference

### `POST /v1/ask` — natural-language questions

Send a US coordinate and a free-text question. A planner model picks relevant fields from the catalog, fetches them in parallel, and a synthesizer model writes a cited prose answer.

```bash
curl -X POST https://api.mireye.com/v1/ask \
  -H 'authorization: Bearer YOUR_MIREYE_TOKEN' \
  -H 'content-type: application/json' \
  -d '{
    "lat": 40.7128,
    "lng": -74.0060,
    "question": "Is this property in a flood zone?",
    "include_trace": false
  }'
```

| Field           | Type    | Required | Description                                                                                |
| --------------- | ------- | -------- | ------------------------------------------------------------------------------------------ |
| `lat`           | number  | Yes      | Latitude in `[18, 72]`.                                                                    |
| `lng`           | number  | Yes      | Longitude in `[-180, -65]`.                                                                |
| `question`      | string  | Yes      | Free-text geospatial question. Max 2000 chars.                                             |
| `include_trace` | boolean | No       | When `true`, includes a `trace` object with planner reasoning and the model names used.     |

**Response (200) — key fields:**

```json
{
  "lat": 40.7128,
  "lng": -74.006,
  "question": "Is this property in a flood zone?",
  "answered_at": "2026-06-24T22:00:00Z",
  "answer": "...prose answer with implicit citations...",
  "confidence": "high",
  "citations": [
    {
      "source": "USGS_3DEP",
      "source_url": "https://epqs.nationalmap.gov/",
      "fields": ["elevation"],
      "fetched_at": "2026-06-24T22:14:01.882Z",
      "confidence": "high"
    }
  ],
  "fields_used": ["elevation", "within_floodplain_polygon", "coast_distance_m"]
}
```

- `answer` — prose. Cite-aware; references sources by name and explicitly flags any field it couldn't get. Note `/v1/ask` does **not** return a `partial_failures` array (that's a `/v1/fetch` field) — gaps surface in the prose answer instead.
- `confidence` — `high | medium | low | unknown`. Reflects the *weakest* citation used. Any `medium` or `low` field pulls the overall down.
- `citations` — one entry per source, with the list of `fields` that source contributed.
- `fields_used` — flat array of catalog field names. Use this to re-verify the answer deterministically via `/v1/fetch`.
- `trace` (only with `include_trace: true`) — planner reasoning, the planner & synthesizer model names, requested fields, and any preset expansion. Useful when debugging "why did it pick those fields?"

**Latency:** 2–6 s warm, up to 60 s on cold start.

### `POST /v1/ask/stream` — streaming answers

Same request body as `/v1/ask` (`lat`, `lng`, `question`, `include_trace`), but the answer streams back as **Server-Sent Events** so you can render tokens as they arrive. Events carry incremental `answer` text plus the accumulating `citations`; a final event delivers the complete record (`answer`, `confidence`, `citations`, `fields_used`, `answered_at`). Use this to drive a responsive UI; use `/v1/ask` when you just want the final JSON.

```bash
curl -N -X POST https://api.mireye.com/v1/ask/stream \
  -H 'authorization: Bearer YOUR_MIREYE_TOKEN' \
  -H 'content-type: application/json' \
  -d '{"lat": 40.7128, "lng": -74.0060, "question": "Wildfire risk here?"}'
```

### `POST /v1/fetch` — named fields with provenance

Request specific catalog fields (or a preset bundle). Returns each one with full provenance. Failed fields go to `partial_failures` — successful (and validly absent) ones are still returned in a 200 response.

```bash
# Named fields
curl -X POST https://api.mireye.com/v1/fetch \
  -H 'authorization: Bearer YOUR_MIREYE_TOKEN' \
  -H 'content-type: application/json' \
  -d '{
    "lat": 40.7128,
    "lng": -74.0060,
    "fields": ["elevation", "coast_distance_m", "within_floodplain_polygon"]
  }'

# Preset
curl -X POST https://api.mireye.com/v1/fetch \
  -H 'authorization: Bearer YOUR_MIREYE_TOKEN' \
  -H 'content-type: application/json' \
  -d '{"lat": 29.7604, "lng": -95.3698, "preset": "flood_risk"}'

# Preset + extra named fields (deduplicated, max 50 total after expansion)
curl -X POST https://api.mireye.com/v1/fetch \
  -H 'authorization: Bearer YOUR_MIREYE_TOKEN' \
  -H 'content-type: application/json' \
  -d '{
    "lat": 29.7604,
    "lng": -95.3698,
    "preset": "flood_risk",
    "fields": ["tree_canopy_pct"]
  }'
```

| Field    | Type     | Required                         | Description                                                                                                    |
| -------- | -------- | -------------------------------- | -------------------------------------------------------------------------------------------------------------- |
| `lat`    | number   | Yes                              | Latitude in `[18, 72]`.                                                                                        |
| `lng`    | number   | Yes                              | Longitude in `[-180, -65]`.                                                                                    |
| `fields` | string[] | One of `fields`/`preset`         | Named catalog fields. See [Field catalog](#field-catalog) or call `/v1/meta/fields` for the live list.         |
| `preset` | string   | One of `fields`/`preset`         | Preset name. Expands server-side, unioned with `fields`, deduplicated. Max 50 fields after expansion.          |

**Response (200) — per-field shape:**

```json
{
  "lat": 40.7128,
  "lng": -74.006,
  "fetched_at": "2026-06-24T22:15:11.420Z",
  "fields": {
    "elevation": {
      "value": 13.15,
      "unit": "meters",
      "source": "USGS_3DEP",
      "source_url": "https://epqs.nationalmap.gov/",
      "confidence": "high",
      "dataset_vintage": "2023",
      "fetched_at": "2026-06-24T22:15:10.110Z",
      "ttl_seconds": 31536000,
      "status": "ok",
      "notes": null
    }
  },
  "partial_failures": []
}
```

- `value` — typed per the catalog (`float`, `int`, `bool`, or `string`); `null` when `status` is `absent`.
- `unit` — SI string like `"meters"`, `"percent"`, or `null` for dimensionless and enum fields.
- `source`, `source_url` — source identifier and canonical upstream URL (always HTTPS). Use `source_url` to independently re-fetch and verify any value.
- `confidence` — `high | medium | low | unknown`.
- `dataset_vintage` — the edition/year of the upstream dataset the value came from (e.g. `"2024"`), or `null` when the source doesn't expose one. Distinct from `fetched_at`.
- `fetched_at` — when *Mireye* hit the upstream source. Authoritative "as-of" time for the value.
- `ttl_seconds` — how long the value is considered fresh, derived from the upstream update cadence. Use this for your own cache TTL. Ranges from ~7 days (Sentinel-2 NDVI) to ~1 year (USGS elevation, Census boundaries).
- `status` — `"ok"` (real value) or `"absent"` (source confirmed no data here; a valid semantic null, not an error). Failed fields never appear here — they go to `partial_failures`.
- `notes` — human-readable caveats (e.g., cloud-screening for NDVI, geometric-overlap confidence for building joins, why a field is absent), or `null`.

**Partial failures:**

```json
{
  "partial_failures": [
    {
      "field": "lcms_class",
      "source": "USFS_LCMS",
      "error": "TimeoutError: source timed out after 10s",
      "retryable": true
    }
  ]
}
```

Check `partial_failures` on every call. `retryable: true` → transient upstream blip, retry. `retryable: false` → permanent (e.g., no coverage), don't.

**Latency:** 1–3 s warm.

### Presets

Fourteen presets ship today. Pass the name as `preset` on `/v1/fetch`. The live expansions are always in `GET /v1/meta/fields` under `presets`.

| Preset                | What it's for | Fields |
| --------------------- | ------------- | ------ |
| `terrain`             | Topography & soils basics | `elevation`, `slope_degrees`, `aspect_cardinal`, `coast_distance_m`, `soil_drainage_class`, `bedrock_depth_cm` |
| `flood_risk`          | Floodplain, wetlands, water | `elevation`, `coast_distance_m`, `within_floodplain_polygon`, `intersects_nhd_area`, `intersects_wetland`, `wetland_type`, `wetland_subtype`, `wetland_acres`, `nearest_wetland_distance_m`, `wetlands_within_100m_count`, `wetlands_within_500m_count`, `surface_water_permanence_pct`, `nearest_waterbody_name` |
| `wildfire_underwrite` | Wildfire fuel & terrain | `lcms_class`, `tree_canopy_pct`, `ndvi_current`, `ndvi_change_5y`, `slope_degrees`, `elevation` |
| `land_cover`          | Land use & crops | `lcms_class`, `land_use_class`, `tree_canopy_pct`, `cdl_class`, `dominant_crop_5y` |
| `site_selection`      | General development screen | `elevation`, `slope_degrees`, `lcms_class`, `within_floodplain_polygon`, `intersects_wetland`, `wetland_type`, `nearest_wetland_distance_m`, `wetlands_within_100m_count`, `nearest_major_road_distance_m`, `nearest_transmission_line_distance_m`, `nearest_transmission_line_voltage_kv`, `nearest_transmission_line_voltage_class`, `nearest_transmission_line_voltage_basis`, `max_transmission_line_voltage_kv_within_radius`, `max_transmission_line_voltage_class_within_radius`, `intersects_conservation_easement`, `intersects_protected_area`, `protected_area_gap_status`, `intersects_critical_habitat`, `critical_habitat_status`, `parcel_id`, `parcel_area_m2`, `parcel_geometry_wkt`, `parcel_boundary_geojson` |
| `building_lookup`     | Primary structure attributes | `primary_building_overture_class`, `primary_building_height_m`, `primary_building_num_floors`, `primary_building_footprint_sqm` |
| `utilities`           | Power, transmission, gas | `nearest_power_plant_name`, `nearest_power_plant_distance_m`, `nearest_power_plant_primary_fuel`, `nearest_power_plant_capacity_mw`, `nearest_transmission_line_distance_m`, `nearest_transmission_line_voltage_kv`, `nearest_transmission_line_voltage_class`, `nearest_transmission_line_voltage_basis`, `nearest_transmission_line_status`, `nearest_transmission_line_owner`, `max_transmission_line_voltage_kv_within_radius`, `max_transmission_line_voltage_class_within_radius`, `transmission_lines_within_radius_count`, `nearest_gas_pipeline_distance_m` |
| `boundaries`          | Jurisdiction & Census | `political_region`, `political_county`, `political_locality`, `tract_geoid` |
| `natural_hazard`      | Multi-peril hazard screen | `seismic_pga_2pct_50yr_g`, `seismic_design_category`, `design_wind_speed_mph`, `wildfire_annual_frequency`, `tornado_annual_frequency`, `hail_annual_frequency`, `lightning_annual_flash_days`, `landslide_susceptibility_index`, `soil_shrink_swell_class`, `within_floodplain_polygon`, `slope_degrees`, `nearest_dam_distance_m`, `nearest_dam_hazard_potential`, `high_hazard_dams_within_10km` |
| `grid_interconnect`   | Transmission & interconnection | `nearest_substation_distance_m`, `nearest_substation_max_voltage_kv`, `nearest_substation_status`, `electric_utility_service_territory`, `interconnection_queue_active_capacity_county_mw`, `wind_least_cost_interconnect_distance_m`, `nearest_proposed_generator_distance_m`, `egrid_subregion`, `nearest_transmission_line_distance_m`, `nearest_power_plant_distance_m`, `nearest_power_plant_capacity_mw` |
| `data_center_siting`  | Power, water, cooling, fiber, risk (49 fields) | `nearest_substation_distance_m`, `nearest_substation_max_voltage_kv`, `nearest_substation_status`, `electric_utility_service_territory`, `avg_retail_electricity_price_industrial_usd_per_kwh`, `egrid_subregion`, `egrid_co2_output_rate_kg_per_mwh`, `interconnection_queue_active_capacity_county_mw`, `design_wet_bulb_temperature_0_4pct_degc`, `mean_annual_dry_bulb_temperature_degc`, `mean_annual_relative_humidity_pct`, `days_above_32c_annual_count`, `surface_water_supply_use_index_huc12`, `public_water_system_population_served`, `huc12_thermoelectric_consumptive_use_m3_per_day`, `nearest_groundwater_well_depth_to_water_m`, `fiber_provider_count`, `fiber_broadband_available`, `mobile_5g_coverage_class`, `nearest_urban_area_distance_m`, `soil_shrink_swell_class`, `surface_management_agency`, `nearest_gas_pipeline_distance_m`, `surface_water_permanence_pct`, `nearest_dam_distance_m`, `nearest_dam_hazard_potential`, `high_hazard_dams_within_10km`, `nearest_hazardous_facility_distance_m`, `nearest_hazardous_facility_name`, `housing_units_within_1km`, `housing_units_density_per_km2`, `natural_gas_citygate_price_usd_per_mcf`, `natural_gas_industrial_price_usd_per_mcf`, `in_shale_play`, `nearest_shale_play_name`, `sedimentary_basin_name`, `nearest_gas_compressor_distance_m`, `nearest_gas_storage_distance_m`, `nearest_lng_terminal_distance_m`, `drought_category`, `in_opportunity_zone`, `opportunity_zone_tract_geoid`, `in_air_quality_nonattainment`, `air_quality_nonattainment_pollutants`, `air_quality_worst_classification`, `nearest_brownfield_distance_m`, `brownfields_within_radius_count`, `nearest_superfund_distance_m`, `superfund_sites_within_radius_count` |
| `solar_siting`        | PV resource & land | `ghi_annual_kwh_m2_day`, `dni_annual_kwh_m2_day`, `pv_capacity_factor_pct`, `pv_specific_yield_kwh_per_kw`, `optimal_fixed_tilt_degrees`, `surface_albedo_annual`, `mean_annual_snow_cover_days`, `mean_annual_dry_bulb_temperature_degc`, `days_above_32c_annual_count`, `nearest_utility_solar_facility_distance_m`, `nearest_utility_solar_facility_capacity_mw`, `prime_farmland_classification`, `blm_solar_application_land_status`, `surface_management_agency`, `nearest_repowering_site_distance_m`, `slope_degrees`, `aspect_degrees`, `aspect_cardinal`, `parcel_zoning`, `is_cultivated`, `tree_canopy_pct`, `housing_units_within_1km`, `housing_units_density_per_km2` |
| `wind_siting`         | Wind resource & land | `mean_wind_speed_100m_ms`, `mean_wind_speed_120m_ms`, `mean_wind_speed_160m_ms`, `wind_power_density_100m_wm2`, `prevailing_wind_direction_100m_cardinal`, `weibull_k_100m`, `wind_capacity_factor_pct`, `wind_least_cost_interconnect_distance_m`, `nearest_wind_turbine_distance_m`, `nearest_wind_turbine_hub_height_m`, `nearest_wind_turbine_total_height_m`, `nearest_wind_project_capacity_mw`, `special_use_airspace_type`, `golden_eagle_nest_density_index`, `prime_farmland_classification`, `surface_management_agency`, `nearest_repowering_site_distance_m`, `slope_degrees`, `elevation`, `nearest_airport_distance_m`, `bedrock_depth_cm`, `housing_units_within_1km`, `housing_units_density_per_km2` |
| `storage_siting`      | Battery storage siting | `nearest_substation_distance_m`, `nearest_substation_max_voltage_kv`, `nearest_substation_status`, `electric_utility_service_territory`, `avg_retail_electricity_price_industrial_usd_per_kwh`, `egrid_co2_output_rate_kg_per_mwh`, `interconnection_queue_active_capacity_county_mw`, `wind_least_cost_interconnect_distance_m`, `nearest_proposed_generator_distance_m`, `nearest_utility_solar_facility_distance_m`, `surface_management_agency`, `prime_farmland_classification`, `nearest_transmission_line_distance_m`, `nearest_power_plant_distance_m`, `parcel_zoning` |

You can combine a preset with extra named `fields` in the same request — Mireye unions and deduplicates them. The total after expansion cannot exceed 50, so the large `data_center_siting` preset (49 fields) leaves little room for extras; split into multiple calls if you need more.

### `GET /v1/meta/fields` — discover the catalog

Returns the complete machine-readable catalog: all 250+ fields (each with `name`, `type`, `unit`, `description`, `interpretation_hints`, `layer`, `source`, `source_url`, `ttl_seconds`, `lifecycle`, `nullable`, `null_meaning`, and `presets` membership), the 15 preset expansions, and the US envelope. Public (no auth), served from memory, sub-50 ms, ETag-cached.

```bash
curl -i https://api.mireye.com/v1/meta/fields
```

Subsequent requests should use the ETag:

```bash
curl -i https://api.mireye.com/v1/meta/fields \
  -H 'if-none-match: "abc123..."'
# 304 Not Modified means your cached body is still valid
```

**Response headers:**

| Header          | Value                                                                |
| --------------- | -------------------------------------------------------------------- |
| `ETag`          | SHA-256 of the body as a quoted string.                              |
| `Cache-Control` | `public, max-age=3600` — safe to cache up to one hour.               |

**Response body (abridged):**

```json
{
  "version": "0.7.0",
  "us_envelope": { "lat_min": 18, "lat_max": 72, "lng_min": -180, "lng_max": -65 },
  "fields": [
    {
      "name": "elevation",
      "type": "float",
      "unit": "meters",
      "description": "Ground elevation above the NAVD88 vertical datum at the queried point. Sourced from USGS 3DEP / EPQS at ~10m native resolution.",
      "interpretation_hints": "Below 10m within 5km of the coast → storm-surge exposure relevant. Above 3000m → alpine permitting, snow loads. Combine with coast_distance_m and within_floodplain_polygon for flood-zone reasoning.",
      "layer": "terrain",
      "source": "USGS_3DEP",
      "source_url": "https://epqs.nationalmap.gov/",
      "ttl_seconds": 31536000,
      "lifecycle": "stable",
      "nullable": false,
      "null_meaning": null,
      "presets": ["terrain", "flood_risk", "site_selection", "wildfire_underwrite", "..."]
    }
  ],
  "presets": {
    "flood_risk": ["elevation", "coast_distance_m", "within_floodplain_polygon", "..."]
  }
}
```

For long-running clients: fetch once at startup, store the ETag, send `If-None-Match` on every subsequent request. The catalog only changes on schema-version bumps.

### Field catalog

250+ fields across 7 layers. Fields in the same layer fetch in parallel — asking for more fields from the same layer adds no extra latency. The live catalog at `GET /v1/meta/fields` is the authoritative source; the names below are a snapshot.

**Layer 1 — Terrain** (USGS 3DEP / NHDPlus HR / WBD, USDA SSURGO / STATSGO, USFWS NWI, NOAA CUSP, JRC GSW, FEMA NFHL)

`elevation`, `slope_degrees`, `aspect_degrees`, `aspect_cardinal`, `coast_distance_m`, `soil_drainage_class`, `soil_map_unit_name`, `bedrock_depth_cm`, `prime_farmland_classification`, `soil_shrink_swell_class`, `intersects_nhd_area`, `nearest_flowline_name`, `nearest_waterbody_name`, `huc_12_name`, `within_floodplain_polygon`, `surface_water_permanence_pct`, `intersects_wetland`, `wetland_type`, `wetland_subtype`, `wetland_acres`, `nearest_wetland_distance_m`, `wetlands_within_100m_count`, `wetlands_within_500m_count`

**Layer 2 — Land Cover** (USFS LCMS, NLCD TCC, Sentinel-2, USDA CDL)

`lcms_class`, `land_use_class`, `tree_canopy_pct`, `ndvi_current`, `ndvi_change_5y`, `cdl_class`, `is_cultivated`, `dominant_crop_5y`

**Layer 3 — Built Environment** (Overture Buildings & Transportation, FHWA NBI, USWTDB, USPVDB, EPA Repowering, HUD Opportunity Zones)

`primary_building_height_m`, `primary_building_num_floors`, `primary_building_footprint_sqm`, `primary_building_overture_class`, `nearest_major_road_name`, `nearest_major_road_distance_m`, `nearest_bridge_name`, `nearest_wind_turbine_distance_m`, `nearest_wind_turbine_hub_height_m`, `nearest_wind_turbine_total_height_m`, `nearest_wind_project_capacity_mw`, `nearest_utility_solar_facility_distance_m`, `nearest_utility_solar_facility_capacity_mw`, `nearest_repowering_site_distance_m`, `in_opportunity_zone`, `opportunity_zone_tract_geoid`

**Layer 4 — Utilities & Energy** (EIA Atlas / 860M / power / prices / gas / shale, EPA eGRID / SDWIS / CWS, LBNL Queued Up, FAA NASR, FCC ASR / BDC, BTS NTAD / Ports, US Census urban, USGS NWIS / thermoelectric / sedimentary basins / IWAA)

`nearest_power_plant_name`, `nearest_power_plant_distance_m`, `nearest_power_plant_primary_fuel`, `nearest_power_plant_capacity_mw`, `nearest_transmission_line_distance_m`, `nearest_transmission_line_voltage_kv`, `nearest_transmission_line_voltage_class`, `nearest_transmission_line_voltage_basis`, `nearest_transmission_line_status`, `nearest_transmission_line_owner`, `max_transmission_line_voltage_kv_within_radius`, `max_transmission_line_voltage_class_within_radius`, `transmission_lines_within_radius_count`, `nearest_substation_distance_m`, `nearest_substation_max_voltage_kv`, `nearest_substation_status`, `electric_utility_service_territory`, `egrid_subregion`, `egrid_co2_output_rate_kg_per_mwh`, `avg_retail_electricity_price_industrial_usd_per_kwh`, `interconnection_queue_active_capacity_county_mw`, `nearest_proposed_generator_distance_m`, `nearest_gas_pipeline_distance_m`, `nearest_gas_compressor_distance_m`, `nearest_gas_storage_distance_m`, `nearest_lng_terminal_distance_m`, `natural_gas_citygate_price_usd_per_mcf`, `natural_gas_industrial_price_usd_per_mcf`, `in_shale_play`, `nearest_shale_play_name`, `sedimentary_basin_name`, `nearest_public_water_system_name`, `public_water_system_population_served`, `surface_water_supply_use_index_huc12`, `huc12_thermoelectric_consumptive_use_m3_per_day`, `nearest_groundwater_well_depth_to_water_m`, `fiber_provider_count`, `fiber_broadband_available`, `mobile_5g_coverage_class`, `nearest_urban_area_distance_m`, `nearest_rail_line_distance_m`, `nearest_airport_name`, `nearest_airport_distance_m`, `nearest_port_name`, `nearest_antenna_structure_distance_m`, `nearest_antenna_structure_height_m`, `nearest_antenna_structure_type`, `nearest_antenna_structure_owner`, `antenna_structures_within_500m_count`, `antenna_structures_within_2km_count`

**Layer 5 — Climate & Resource** (NREL NSRDB / PVWatts / Wind Toolkit / reV / Solar Resource, NOAA NCEI nClimGrid & Normals / SNODAS / ASCE wind, US Drought Monitor)

`ghi_annual_kwh_m2_day`, `dni_annual_kwh_m2_day`, `pv_capacity_factor_pct`, `pv_specific_yield_kwh_per_kw`, `optimal_fixed_tilt_degrees`, `surface_albedo_annual`, `mean_wind_speed_100m_ms`, `mean_wind_speed_120m_ms`, `mean_wind_speed_160m_ms`, `wind_power_density_100m_wm2`, `prevailing_wind_direction_100m_cardinal`, `weibull_k_100m`, `wind_capacity_factor_pct`, `wind_least_cost_interconnect_distance_m`, `design_wet_bulb_temperature_0_4pct_degc`, `mean_annual_dry_bulb_temperature_degc`, `mean_annual_relative_humidity_pct`, `days_above_32c_annual_count`, `mean_annual_snow_cover_days`, `drought_category`

**Layer 6 — Hazards** (USGS NSHM seismic / landslide / DesignMaps ASCE7, FEMA NRI, NOAA ASCE wind vectors, USACE NID dams, EPA FRS RMP / SEMS / ACRES / Green Book, US Census)

`seismic_design_category`, `seismic_pga_2pct_50yr_g`, `design_wind_speed_mph`, `wildfire_annual_frequency`, `tornado_annual_frequency`, `hail_annual_frequency`, `lightning_annual_flash_days`, `landslide_susceptibility_index`, `nearest_dam_distance_m`, `nearest_dam_hazard_potential`, `high_hazard_dams_within_10km`, `nearest_hazardous_facility_distance_m`, `nearest_hazardous_facility_name`, `nearest_brownfield_distance_m`, `brownfields_within_radius_count`, `nearest_superfund_distance_m`, `superfund_sites_within_radius_count`, `in_air_quality_nonattainment`, `air_quality_nonattainment_pollutants`, `air_quality_worst_classification`, `housing_units_within_1km`, `housing_units_density_per_km2`

**Layer 7 — Parcels & Boundaries** (Overture Divisions, US Census TIGERweb & Geocoder, USGS PAD-US, Regrid, USFWS Critical Habitat & Golden Eagle, BLM SMA & Solar PEIS, FAA SUA)

`political_region`, `political_county`, `political_locality`, `tract_geoid`, `parcel_id`, `parcel_apn`, `parcel_owner`, `parcel_address`, `parcel_zoning`, `parcel_area_m2`, `parcel_geometry_wkt`, `parcel_boundary_geojson`, `parcel_data_source`, `parcel_match_type`, `parcel_match_distance_m`, `parcel_match_radius_m`, `intersects_conservation_easement`, `easement_holder`, `easement_type`, `easement_purpose`, `easement_acres`, `easement_year_established`, `intersects_protected_area`, `protected_area_name`, `protected_area_gap_status`, `protected_area_designation`, `protected_area_manager`, `protected_area_public_access`, `intersects_critical_habitat`, `critical_habitat_status`, `critical_habitat_species`, `critical_habitat_listing_status`, `surface_management_agency`, `blm_solar_application_land_status`, `special_use_airspace_type`, `golden_eagle_nest_density_index`

The live catalog at `GET /v1/meta/fields` is the authoritative source. Common naming mistakes:

- `elevation_m` → `elevation` (no unit suffix in the name)
- `flood_zone` → `within_floodplain_polygon` (use the full predicate name)
- `slope` → `slope_degrees` (unit suffix *is* required when ambiguous)
- `solar` / `ghi` → `ghi_annual_kwh_m2_day`, `dni_annual_kwh_m2_day` (resource fields carry the full unit)
- `wind_speed` → `mean_wind_speed_100m_ms` (hub height + unit are part of the name)
- `substation` → `nearest_substation_distance_m` / `nearest_substation_max_voltage_kv`
- `primary_building` → `primary_building_overture_class`, `primary_building_height_m`, `primary_building_num_floors`, `primary_building_footprint_sqm`
- `conservation_easement` → `intersects_conservation_easement` or `easement_holder`

### Sites — persistent locations

For workflows that ask many questions about the *same* place, register it once and reference it by ID instead of re-sending the geometry each time.

- **`POST /v1/sites`** — register a site (point or area) and get back a `site_id`.
- **`GET /v1/sites/{site_id}`** — retrieve the stored site and its computed dossier.
- **`POST /v1/ask-site`** — natural-language Q&A scoped to a registered site: `{ "site_id": "...", "question": "..." }`. Same cited-answer shape as `/v1/ask`.

All three require a bearer token; sites are scoped to your account. Use the point queries (`/v1/ask`, `/v1/fetch`) for one-off lookups and Sites when you're building a persistent dossier.

### Error responses

| Status | Error code            | Meaning                                                                                                      |
| ------ | --------------------- | ------------------------------------------------------------------------------------------------------------ |
| 400    | `coord_out_of_bounds` | Coordinate outside `lat ∈ [18, 72]`, `lng ∈ [-180, -65]`.                                                    |
| 400    | `fields_unknown`      | One or more field names not in the catalog. Response includes `fields_unknown: [...]` listing the offenders. |
| 400    | `fields_too_many`     | More than 50 fields after preset expansion. Split into multiple requests.                                    |
| 400    | `no_fields_requested` | `/v1/fetch` called without `fields` or `preset`.                                                             |
| 401    | `unauthorized`        | Missing, expired, or revoked bearer token. Re-authenticate.                                                  |
| 403    | `forbidden`           | Valid token, but the account is not permitted by backend policy.                                             |
| 422    | (validation)          | Request body failed schema validation (e.g., `lat`/`lng` wrong type, `question` empty).                      |
| 500    | `internal`            | Orchestrator crash. Response includes `request_id`; quote it when reporting.                                 |

Shape:

```json
{ "error": "fields_unknown", "message": "Unknown field names: ['flood_zone']", "fields_unknown": ["flood_zone"] }
```

Partial failures inside a `/v1/fetch` 200 response are *not* errors — they're a `partial_failures` array entry. Always check it. (`/v1/ask` reports gaps in its prose answer instead, not a `partial_failures` array.)

---

## MCP integration

If you're running inside an MCP-aware host (Claude Desktop, Claude Code, Cursor, custom agent), you can use the Mireye Earth MCP server instead of calling HTTP directly. It exposes the same two operations as native tools: `mireye_ask` and `mireye_fetch` (prefixed so they don't collide with generic `ask`/`fetch` tools from other MCP servers). There are two ways to connect.

### Hosted remote endpoint (recommended for Claude Code)

Mireye runs a hosted MCP server at **`https://api.mireye.com/mcp`** over Streamable HTTP with native OAuth 2.1 + PKCE. This is the simplest path for Claude Code — no local install, browser sign-in:

```bash
claude mcp remove mireye-earth -s user   # only if an old stdio entry exists
claude mcp add --transport http --scope user mireye-earth https://api.mireye.com/mcp
```

Restart Claude Code, run `/mcp`, and complete the browser OAuth flow.

### Local stdio package (Claude Desktop, Cursor, custom agents)

The local adapter ships as its own slim PyPI package — **`mireye-mcp`** — with only `httpx` and the `mcp` SDK as dependencies. No GDAL, no native builds. `uvx` fetches it on demand:

```bash
uvx mireye-mcp
```

Authenticate once with the device flow (or set `MIREYE_BEARER_TOKEN` for non-interactive hosts):

```bash
mireye-mcp login      # prints a verification URL + code; approve in your account page
mireye-mcp status     # inspect stored credentials
mireye-mcp logout     # clear (add --revoke to also revoke server-side)
```

Add to your host's MCP config (e.g. `~/Library/Application Support/Claude/claude_desktop_config.json` on macOS, or `~/.cursor/mcp.json` for Cursor):

```json
{
  "mcpServers": {
    "mireye-earth": {
      "command": "uvx",
      "args": ["mireye-mcp"]
    }
  }
}
```

Restart the host. The `mireye_ask` and `mireye_fetch` tools appear under the plug menu. Point at a self-hosted deployment with `"env": { "MIREYE_BASE_URL": "https://your-deploy.example.com" }`. Stored credentials are bound to the `MIREYE_BASE_URL` they were created against.

### Tools, resources, and prompts

MCP keeps tools, resources, and prompts in three separate namespaces. To *do* something (fetch data, ask a question), call a **tool** via `tools/call`. The prompts below are starting templates a user invokes; they're not the call surface.

- **Tools** (the call surface — `tools/call`): `mireye_ask`, `mireye_fetch`.
- **Catalog resources** (read these instead of a `list_fields` tool): `mireye://catalog/fields`, `mireye://catalog/presets`, `mireye://catalog/us-envelope`, `mireye://field/{name}`, `mireye://preset/{name}`. Backed by `GET /v1/meta/fields` with a 1-hour ETag-aware cache.
- **Workflow prompts** (these are MCP *prompts*, fetched via `prompts/get` — not tools): `mireye_ask`, `mireye_fetch`, `mireye_site_report`, `mireye_flood_check`, `mireye_wildfire_underwrite`, `mireye_pick_fields`. The `mireye_ask` / `mireye_fetch` names appear in both lists — same name, different MCP primitive. Claude Code surfaces prompts as slash commands of the form `/mcp__mireye-earth__<prompt>`; the model still calls the underlying *tool* to actually run the request.

The server is also published to the [Official MCP Registry](https://registry.modelcontextprotocol.io) as **`com.mireye/earth`**, carrying both the PyPI distribution and the hosted remote — so registry-aware clients can discover and install it without manual config.

See [docs.mireye.ai/mcp/installation](https://docs.mireye.ai/mcp/installation) and [docs.mireye.ai/mcp/tools](https://docs.mireye.ai/mcp/tools) for full setup.

---

## Critical Gotchas

Read these once. They'll save you.

1. **US only.** Coordinates outside the envelope return `400`. Don't try to "approximate" by snapping to the nearest US point — tell the human Mireye doesn't cover that location.
2. **All data endpoints need a bearer token.** Only `GET /v1/meta/fields` is public. `/v1/ask`, `/v1/ask/stream`, `/v1/fetch`, and Sites require `Authorization: Bearer …`.
3. **`/v1/ask` cold start can be 60 s.** Warm is 2–6 s. If you're driving a UI, set a generous timeout (e.g. 90 s) and surface a loading state — or use `/v1/ask/stream` to render tokens as they arrive.
4. **`/v1/fetch` partial failures live inside 200 responses.** Always read its `partial_failures` array — don't assume success just because the HTTP status is 200. (`/v1/ask` has no such array; it flags missing fields in the prose answer.)
5. **`status: "absent"` ≠ failure.** It means "the source has no value here." `value` is `null`, `confidence` is `unknown`. Don't retry. Show it to the human as "no data at this location." Truly failed fields are in `partial_failures`, not under `fields`.
6. **Confidence is lowercase and bucketed.** `high | medium | low | unknown`. For regulatory/audit work, gate on `high`. For screening, `medium` is usually fine. Flag `low` for human review.
7. **Per-field provenance is the product.** Never strip `source` / `source_url` / `dataset_vintage` / `fetched_at` when summarizing for a human. The citation chain is what separates Mireye from a hallucination.
8. **Cache the catalog, don't re-fetch.** `GET /v1/meta/fields` is sub-50 ms but ETag-cached for a reason — pull once at startup and reuse.
9. **`/v1/ask` picks fields nondeterministically.** Different runs of the same question can pick different fields. If you need determinism, capture `fields_used` from one run and replay via `/v1/fetch`.
10. **The `data_center_siting` preset is 49 fields.** It nearly fills the 50-field-per-request ceiling on its own — don't expect to union many extra fields onto it in one call.

---

## Ideas — What You Can Do With Mireye

- **Data center siting.** The `data_center_siting` preset pulls power (substation distance, voltage, interconnection-queue headroom), electricity & gas prices, eGRID emissions, cooling (design wet-bulb, humidity, hot-day counts), water supply, fiber & 5G availability, and hazards/contamination in one call — the full first-pass screen for a hyperscale site.
- **Renewable energy siting.** `solar_siting` and `wind_siting` combine NREL resource data (GHI/DNI, PV yield, wind speed at hub height, capacity factor) with land constraints (slope, farmland, BLM status, protected areas, eagle-nest density) and the least-cost interconnect distance. `storage_siting` and `grid_interconnect` cover battery and transmission screens.
- **Property insurance underwriting.** Pull `flood_risk`, `wildfire_underwrite`, or `natural_hazard` for a coordinate, gate on `high` confidence, store the citation chain as part of the underwriting record. See [docs.mireye.ai/use-cases/insurance](https://docs.mireye.ai/use-cases/insurance).
- **Mortgage & title due diligence.** `flood_risk` + `boundaries` + `intersects_conservation_easement` + `easement_holder` for floodplain status, jurisdiction, and recorded easements with source citations. See [docs.mireye.ai/use-cases/lending](https://docs.mireye.ai/use-cases/lending).
- **Agent reasoning grounded in source data.** Wire Mireye into an agent that has to make claims about physical locations. Cited, reproducible, auditable. See [docs.mireye.ai/use-cases/agents](https://docs.mireye.ai/use-cases/agents).
- **Portfolio screening.** Batch-call `/v1/fetch` over many coordinates, filter on `confidence == "high"`, route everything else to a human-review queue.
- **Re-verification pipelines.** Ask `/v1/ask` once, capture `fields_used`, re-run `/v1/fetch` on those exact field names months later, diff. Detect changes (e.g., a parcel newly added to a floodplain) without writing dataset-specific code.

---

## Learn More

- **Quickstart:** [docs.mireye.ai/quickstart](https://docs.mireye.ai/quickstart)
- **Introduction:** [docs.mireye.ai/introduction](https://docs.mireye.ai/introduction)
- **Authentication:** [docs.mireye.ai/authentication](https://docs.mireye.ai/authentication)
- **Full API reference:** [/v1/ask](https://docs.mireye.ai/api-reference/ask), [/v1/fetch](https://docs.mireye.ai/api-reference/fetch), [/v1/meta/fields](https://docs.mireye.ai/api-reference/meta-fields)
- **Field catalog:** [docs.mireye.ai/api-reference/field-catalog](https://docs.mireye.ai/api-reference/field-catalog)
- **Errors:** [docs.mireye.ai/api-reference/errors](https://docs.mireye.ai/api-reference/errors)
- **MCP setup:** [docs.mireye.ai/mcp/installation](https://docs.mireye.ai/mcp/installation), [docs.mireye.ai/mcp/tools](https://docs.mireye.ai/mcp/tools), [docs.mireye.ai/mcp/troubleshooting](https://docs.mireye.ai/mcp/troubleshooting)
- **Use cases:** [building agents](https://docs.mireye.ai/use-cases/agents), [insurance](https://docs.mireye.ai/use-cases/insurance), [lending](https://docs.mireye.ai/use-cases/lending)
- **Full docs index for LLMs:** [docs.mireye.ai/llms.txt](https://docs.mireye.ai/llms.txt)
- **OpenAPI spec:** [api.mireye.com/v1/openapi.json](https://api.mireye.com/v1/openapi.json)
- **MCP source:** [github.com/Mireye-Labs/mireye-earth-mcp](https://github.com/Mireye-Labs/mireye-earth-mcp)
