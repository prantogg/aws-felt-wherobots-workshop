# Part 2 — Map Builder AI Agent

See the root `CLAUDE.md` for the full project overview.

## Quick Reference

- **Agent**: `agent.py` — Strands Agent (Bedrock Claude) with Felt MCP tools + python_repl fallback
- **Skills**: `skills/felt-mapping/` (MCP tool docs) and `skills/aurora-postgis/` (fallback)
- **Run**: `./run.sh "your prompt"` or `./run.sh` for interactive mode

## Architecture

```
User prompt → Strands Agent → Felt MCP tools → Aurora (SQL) → Felt Map
                           ↘ python_repl (fallback for complex transforms)
```

**Primary path (Felt MCP):**
1. `list_data_sources` → find Aurora connection
2. `create_map` → new map
3. `create_layer_from_data_source` → SQL query
4. `poll_layer_processing_status` → wait
5. `generate_fsl` → AI styling
6. `update_layer_properties` → apply style
7. `render_map` → inline preview

**Fallback (python_repl):**
- MCP upload fails → `felt_python.upload_file()`
- Complex transforms → pandas/geopandas
- Debug connectivity → psycopg2

## Data (Aurora, `workshop` schema)

The CloudFormation seed loads one table, `workshop.insurance_exposure`, for
San Diego County (about 1.03M buildings). A completed Part 1 Gold run replaces
it and adds the other three, all for the City of San Diego (357,263 buildings
each). Count rows before promising a map on `cre_risk`,
`capital_markets_signals` or `energy_asset_risk`: if Part 1 was not run, only
`insurance_exposure` exists.

Every table has `risk_score`, `risk_tier`, the three factors and `score_explanation`, plus:
- `workshop.insurance_exposure` — exposure_delta, triage_priority, relative_risk_band
- `workshop.cre_risk` — acquisition_screen_flag, exposure_magnitude_index, hazard_proximity_m
- `workshop.capital_markets_signals` — disruption_signal, supply_chain_vulnerability, event_density_signal
- `workshop.energy_asset_risk` — outage_probability, wildfire_ignition_risk, weather_impact_frequency

## MCP Servers

- **Felt**: `https://felt.com/mcp` — Primary for map operations
- **Wherobots**: `https://api.cloud.wherobots.com/mcp/` — Part 1 data engineering
