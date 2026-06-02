# GCF Patrol Download — Ecoscope Workflow

Ecoscope Desktop workflow that downloads EarthRanger patrol tracks and associated events,
with full event detail flattening.

---

## What this workflow produces

| Output | Format | Description |
|--------|--------|-------------|
| `patrol_trajectories.*` | Parquet | Patrol tracks as LineString geometries, one row per patrol segment |
| `patrol_events.*` | CSV or Parquet | Events linked to patrols, with `event_details` flattened to columns |

---

## Workflow steps

| Step | What happens |
|------|-------------|
| Data Source + Time Range | Pick your EarthRanger connection and date range |
| Patrol and Event Types | Filter by patrol type(s) and/or event type(s) — leave empty for all |
| Process Patrol Observations | Filter GPS points → convert to trajectory LineStrings |
| Process Patrol Events | Flatten `event_details`, extract `reported_by_name`, timezone-convert, column cleanup |
| Persist | Export trajectories (Parquet) and events (CSV) |
| Generate Maps | Optional — shows patrol tracks + event points; skip for large datasets |

---

## Key differences from the Streamlit app

| Streamlit app | This workflow |
|---------------|---------------|
| Shapefile output | Parquet (use QGIS or GeoPandas to open; preserves geometry) |
| Custom patrol time filter | `truncate_to_time_range: true` in `set_patrols_and_patrol_events_params` |
| `get_patrol_segment_events` per segment | `unpack_events_from_patrols_df_and_combined_params` (bulk, linked by `patrol_serial_number`) |
| `pd.json_normalize` for event details | `process_events_details` → `normalize_json_column` → `drop_column_prefix` |
| UUID resolution for detail_ columns | `map_to_titles: true` in `process_events_details` |
| Repeat-group flattening | Pending — see below |

---

## GCF repeat-group handling

The `custom_tasks/flatten_repeat_groups.py` function is written and working but not yet
packaged as a workflow extension. To activate it once packaged:

1. Add the extension requirement to `spec.yaml`:
   ```yaml
   - name: ecoscope-workflows-ext-gcf
     version: "..."
     channel: https://repo.prefix.dev/ecoscope-workflows-custom/
   ```
2. Uncomment the `flatten_repeat_groups` task block in `spec.yaml` (clearly marked).

Contact the wildlife-dynamics team to discuss packaging.

---

## Requirements

- Ecoscope Desktop with a configured EarthRanger data source
- `ecoscope-platform >= 2.11.14, < 2.12.0`
- `ecoscope-workflows-ext-custom 0.1.0rc6`
