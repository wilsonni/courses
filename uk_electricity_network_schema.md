# UK Electricity Distribution Network Database Schema

## Proof-of-Concept: UK Power Networks (UKPN)

---

## 1. DNO Selection: UK Power Networks

**UK Power Networks (UKPN)** is the recommended DNO for the proof-of-concept, selected for these reasons:

| Criterion | UKPN | Nearest competitor (NGED) |
|---|---|---|
| Customers | ~8.3 million connections | ~8 million |
| Population served | ~19 million (29% of UK) | ~7.8 million |
| Licence areas | 3 (London, South East, East of England) | 4 (Midlands, South West, South Wales) |
| Open data platform | OpenDataSoft (thousands of datasets, REST API, CSV/GeoJSON/SHP) | Connected Data Portal (more limited) |
| LTDS format | Structured queryable tables | Primarily PDF |
| Smart meter data | Half-hourly at LV feeder and substation level | Emerging |
| Secondary substation coverage | Full (sites, transformers, utilisation, distribution areas) | Limited |

UKPN is the clear choice because it has the **most comprehensive, machine-readable open data** of any UK DNO, covering the full network hierarchy from Grid Supply Points down to LV feeders.

---

## 2. Datasets to Ingest

### 2.1 Network Hierarchy Datasets

| # | Dataset | UKPN Dataset ID | Key Content | Format |
|---|---|---|---|---|
| D1 | **Grid Supply Points Overview** | `ukpn-grid-supply-points-overview` | GSP names, technical limits, seasonal import/export limits, capacity utilisation | CSV, API |
| D2 | **GSP Distribution Areas** | `ukpn-grid-supply-points` | Geospatial polygons defining each GSP's supply area | GeoJSON, SHP |
| D3 | **Primary Substation Distribution Areas** | `ukpn_primary_postcode_area` | Primary substation names, firm capacity, % unutilised, supply area polygons | GeoJSON, CSV |
| D4 | **Secondary Sites** | `ukpn-secondary-sites` | All secondary substations: name, location, transformer rating (kVA), customer count, parent primary feeder | CSV, API |
| D5 | **Secondary Site Transformers** | `ukpn-secondary-site-transformers` | Transformer ratings, voltage, connectivity for each secondary site | CSV |
| D6 | **Secondary Substation Distribution Areas** | `ukpn_secondary_postcode_area` | Voronoi-based supply area polygons for each secondary substation | GeoJSON, SHP |

### 2.2 Transformer & Technical Datasets

| # | Dataset | UKPN Dataset ID | Key Content | Format |
|---|---|---|---|---|
| D7 | **LTDS Table 2a -- Transformer Data** | `ltds-table-2a` | Two-winding transformer ratings, impedances at Grid and Primary substations | CSV |
| D8 | **LTDS Table 4 -- Fault Levels** | (part of LTDS) | Fault level data at each substation | CSV |
| D9 | **Earth Potential Rise Data** | `grid-and-primary-sites-epr-data` | Fault current, ground return current, EPR classification | CSV |

### 2.3 Demand & Supply (Current / Historical)

| # | Dataset | UKPN Dataset ID | Key Content | Format |
|---|---|---|---|---|
| D10 | **LTDS Table 3a -- Observed Peak Demand** | `ltds-table-3a-load-data-observed` | Observed MW peak demand at each Grid and Primary substation (uncorrected) | CSV |
| D11 | **LTDS Table 3b -- True Peak Demand** | (part of LTDS) | Corrected MW peak demand (adds back demand served by embedded generation) | CSV |
| D12 | **Grid Transformer Power Flow** | `ukpn-grid-transformer-operational-data-half-hourly` | Half-hourly time series: voltage, current, active power (MW), reactive power (MVAr) at each SGT | CSV, API |
| D13 | **Smart Meter Consumption -- Substation** | `ukpn-smart-meter-consumption-substation` | Half-hourly aggregated import kWh at each secondary substation | CSV, API |
| D14 | **Smart Meter Consumption -- LV Feeder** | `ukpn-smart-meter-consumption-lv-feeder` | Half-hourly aggregated import kWh at each LV feeder | CSV, API |
| D15 | **Secondary Site Utilisation** | `ukpn-secondary-site-utilisation` | Annual % utilisation of each secondary substation | CSV |
| D16 | **Standard Load Profiles** | `ukpn-standard-profiles-electricity-demand` | Yearly load profiles by customer type (domestic, commercial, industrial, EV, data centre) | CSV |

### 2.4 Embedded Generation & Connected Capacity (Current Supply)

| # | Dataset | UKPN Dataset ID | Key Content | Format |
|---|---|---|---|---|
| D17 | **Embedded Capacity Register** | `ukpn-embedded-capacity-register` | All DER sites >= 1MW: location, energy source, export capacity (MW), connection status, parent primary substation | CSV, API |
| D18 | **Large Demand List** | `ukpn-large-demand-list` | Committed import projects >= 5,000 kVA (anonymised) | CSV |
| D19 | **GSP Project Status Breakdown** | `ukpn-gsp-project-status` | Connection projects in queue at each GSP by status | CSV |

### 2.5 Forecast Demand & Supply

| # | Dataset | UKPN Dataset ID | Key Content | Format |
|---|---|---|---|---|
| D20 | **DFES Network Scenario Headroom Report** | `dfes-network-headroom-report` | Demand and generation headroom (MW) at BSP and Primary level, projected to 2050 under multiple scenarios | CSV, API |
| D21 | **LTDS Infrastructure Projects** | `ukpn-ltds-infrastructure-projects` | Planned reinforcement projects: location, timescale, capacity increase | CSV |

### 2.6 Operational & Supplementary

| # | Dataset | UKPN Dataset ID | Key Content | Format |
|---|---|---|---|---|
| D22 | **Live Faults** | `ukpn-live-faults` | Near real-time power cut data (planned and unplanned outages) | CSV, API |
| D23 | **Network Statistics** | `ukpn-network-statistics` | High-level statistics per licence area from RIGs submissions | CSV |
| D24 | **Data Centre Utilisation** | `ukpn-data-centre-utilisation` | Maximum observed utilisations from operational data centres | CSV |

---

## 3. Network Hierarchy

The UK distribution network follows this tree structure:

```
TRANSMISSION (National Grid)           400kV / 275kV
       |
  [GRID SUPPLY POINT (GSP)]           400/275kV --> 132kV
       |                               Supergrid Transformers (SGTs)
       |
  [BULK SUPPLY POINT (BSP)]           132kV --> 33kV (or 66kV)
       |                               Extra High Voltage (EHV)
       |
  [PRIMARY SUBSTATION]                 33kV --> 11kV (or 6.6kV)
       |                               High Voltage (HV)
       |
  [SECONDARY SUBSTATION]              11kV --> 400V/230V
       |                               Low Voltage (LV)
       |
  [LV FEEDER]                         Underground cable / overhead line
       |
  [SERVICE CONNECTION / MPAN]          Customer meter point
```

**Key relationships:**
- 1 GSP feeds many BSPs
- 1 BSP feeds many Primary Substations
- 1 Primary Substation feeds many Secondary Substations
- 1 Secondary Substation feeds many LV Feeders
- 1 LV Feeder serves many customer MPANs

**Primary key across all datasets:** `Functional Location Code (FLOC)` -- a unique identifier assigned to every asset by UKPN.

---

## 4. Database Schema Design

### 4.1 Entity-Relationship Overview

```
 +------------------+       +------------------+       +---------------------+
 |  licence_area    |       |  grid_supply_    |       |  bulk_supply_       |
 |                  |<------+  point (GSP)     |<------+  point (BSP)        |
 |  licence_area_id |  1:N  |  gsp_id (FLOC)   |  1:N  |  bsp_id (FLOC)      |
 +------------------+       +------------------+       +---------------------+
                                                               |
                                                           1:N |
                                                               v
 +---------------------+       +------------------+       +---------------------+
 |  embedded_capacity   |  N:1  |  primary_        |       |  secondary_         |
 |  register            |------>|  substation      |<------+  substation          |
 |  ecr_id              |       |  primary_id(FLOC)|  1:N  |  secondary_id(FLOC) |
 +---------------------+       +------------------+       +---------------------+
                                       |                          |
                                       |                      1:N |
                                       |                          v
                                       |                  +---------------------+
                                       |                  |  lv_feeder          |
                                       |                  |  feeder_id          |
                                       |                  +---------------------+
                                       |
               +-----------------------+-----------------------+
               |                       |                       |
               v                       v                       v
  +-------------------+  +------------------------+  +------------------------+
  | transformer       |  | demand_observation     |  | dfes_forecast          |
  | transformer_id    |  | observation_id         |  | forecast_id            |
  | substation_floc   |  | substation_floc        |  | substation_floc        |
  +-------------------+  +------------------------+  +------------------------+
```

### 4.2 Table Definitions

#### Reference / Dimension Tables

```sql
-- ============================================================
-- LICENCE AREAS (UKPN operates 3; extensible for other DNOs)
-- ============================================================
CREATE TABLE licence_area (
    licence_area_id   VARCHAR(10)  PRIMARY KEY,   -- e.g. 'EPN', 'LPN', 'SPN'
    dno_name          VARCHAR(100) NOT NULL,       -- e.g. 'UK Power Networks'
    licence_area_name VARCHAR(200) NOT NULL,       -- e.g. 'Eastern Power Networks'
    region_desc       TEXT
);

-- Source: Manually populated from UKPN licence structure.
-- Extensibility: Add rows for NGED, NPg, ENW, SSEN, SPEN when onboarding new DNOs.


-- ============================================================
-- GRID SUPPLY POINTS (GSPs)
-- ============================================================
-- Sources: D1 (GSP Overview), D2 (GSP Distribution Areas), D12 (Grid Transformer Power Flow)
CREATE TABLE grid_supply_point (
    gsp_floc               VARCHAR(50)   PRIMARY KEY,
    gsp_name               VARCHAR(200)  NOT NULL,
    licence_area_id        VARCHAR(10)   REFERENCES licence_area(licence_area_id),

    -- Technical limits (from D1)
    winter_import_limit_mw DECIMAL(10,2),          -- MW
    summer_import_limit_mw DECIMAL(10,2),
    asset_import_limit_mw  DECIMAL(10,2),
    asset_export_limit_mw  DECIMAL(10,2),
    historical_max_mw      DECIMAL(10,2),
    historical_min_mw      DECIMAL(10,2),
    import_utilisation_pct DECIMAL(5,2),            -- %
    export_utilisation_pct DECIMAL(5,2),

    -- Geospatial (from D2)
    boundary_geojson       JSONB,                   -- GeoJSON polygon
    latitude               DECIMAL(9,6),
    longitude              DECIMAL(9,6),

    updated_at             TIMESTAMP DEFAULT NOW()
);


-- ============================================================
-- BULK SUPPLY POINTS (BSPs)
-- ============================================================
-- Sources: D20 (DFES Headroom -- reports at BSP level), LTDS network diagrams
CREATE TABLE bulk_supply_point (
    bsp_floc               VARCHAR(50)   PRIMARY KEY,
    bsp_name               VARCHAR(200)  NOT NULL,
    gsp_floc               VARCHAR(50)   REFERENCES grid_supply_point(gsp_floc),
    licence_area_id        VARCHAR(10)   REFERENCES licence_area(licence_area_id),

    voltage_kv             DECIMAL(6,1),            -- e.g. 132, 66, 33
    firm_capacity_mw       DECIMAL(10,2),

    latitude               DECIMAL(9,6),
    longitude              DECIMAL(9,6),

    updated_at             TIMESTAMP DEFAULT NOW()
);


-- ============================================================
-- PRIMARY SUBSTATIONS
-- ============================================================
-- Sources: D3 (Primary Distribution Areas), D7 (LTDS Table 2a), D10/D11 (LTDS Table 3a/3b)
CREATE TABLE primary_substation (
    primary_floc           VARCHAR(50)   PRIMARY KEY,
    primary_name           VARCHAR(200)  NOT NULL,
    bsp_floc               VARCHAR(50)   REFERENCES bulk_supply_point(bsp_floc),
    licence_area_id        VARCHAR(10)   REFERENCES licence_area(licence_area_id),

    -- Capacity (from D3)
    firm_capacity_mw       DECIMAL(10,2),
    season_of_constraint   VARCHAR(20),             -- 'Winter' or 'Summer'
    unutilised_capacity_pct DECIMAL(5,2),

    -- Location (from D3)
    boundary_geojson       JSONB,                   -- GeoJSON polygon (distribution area)
    latitude               DECIMAL(9,6),
    longitude              DECIMAL(9,6),

    updated_at             TIMESTAMP DEFAULT NOW()
);


-- ============================================================
-- SECONDARY SUBSTATIONS
-- ============================================================
-- Sources: D4 (Secondary Sites), D5 (Secondary Site Transformers),
--          D6 (Secondary Distribution Areas), D15 (Utilisation)
CREATE TABLE secondary_substation (
    secondary_floc         VARCHAR(50)   PRIMARY KEY,
    secondary_name         VARCHAR(200),
    alias                  VARCHAR(200),
    address                TEXT,
    primary_floc           VARCHAR(50)   REFERENCES primary_substation(primary_floc),
    licence_area_id        VARCHAR(10)   REFERENCES licence_area(licence_area_id),

    -- Technical (from D4, D5)
    onan_rating_kva        DECIMAL(10,2),           -- Transformer ONAN rating
    voltage_kv             DECIMAL(6,2),            -- e.g. 11.0
    indoor_outdoor         VARCHAR(10),             -- 'Indoor' or 'Outdoor'
    customer_count         INTEGER,                 -- Redacted if <= 5

    -- Utilisation (from D15)
    annual_utilisation_pct DECIMAL(5,2),
    utilisation_method     VARCHAR(50),             -- 'Monitored' or 'Estimated'

    -- Location (from D4, D6)
    boundary_geojson       JSONB,
    latitude               DECIMAL(9,6),
    longitude              DECIMAL(9,6),
    grid_ref               VARCHAR(20),

    updated_at             TIMESTAMP DEFAULT NOW()
);


-- ============================================================
-- LV FEEDERS
-- ============================================================
-- Sources: D14 (Smart Meter Consumption -- LV Feeder)
CREATE TABLE lv_feeder (
    feeder_id              VARCHAR(100)  PRIMARY KEY,  -- Composite or UKPN feeder reference
    secondary_floc         VARCHAR(50)   REFERENCES secondary_substation(secondary_floc),
    licence_area_id        VARCHAR(10)   REFERENCES licence_area(licence_area_id),

    feeder_name            VARCHAR(200),
    phase_count            INTEGER,                    -- Typically 3

    updated_at             TIMESTAMP DEFAULT NOW()
);
```

#### Transformer Detail Table

```sql
-- ============================================================
-- TRANSFORMERS (Grid, Primary, and Secondary level)
-- ============================================================
-- Sources: D7 (LTDS Table 2a), D5 (Secondary Site Transformers), D9 (EPR Data)
CREATE TABLE transformer (
    transformer_id         SERIAL        PRIMARY KEY,
    substation_floc        VARCHAR(50)   NOT NULL,     -- FK to any substation level
    substation_level       VARCHAR(20)   NOT NULL,     -- 'GSP', 'BSP', 'Primary', 'Secondary'

    -- Ratings (from D7, D5)
    rating_mva             DECIMAL(10,2),
    onan_rating_kva        DECIMAL(10,2),              -- For secondary transformers
    hv_voltage_kv          DECIMAL(6,1),
    lv_voltage_kv          DECIMAL(6,1),
    impedance_pct          DECIMAL(6,3),

    -- Fault level (from D8)
    fault_level_ka         DECIMAL(10,2),

    -- EPR (from D9)
    epr_classification     VARCHAR(20),
    ground_return_current_a DECIMAL(10,2),

    winding_type           VARCHAR(20),               -- 'Two-winding', 'Three-winding'

    updated_at             TIMESTAMP DEFAULT NOW()
);
```

#### Time-Series / Observation Tables

```sql
-- ============================================================
-- DEMAND OBSERVATIONS (current/historical usage at all levels)
-- ============================================================
-- Sources: D10 (LTDS 3a -- annual peak), D11 (LTDS 3b -- true peak),
--          D12 (Grid Transformer half-hourly), D13 (Smart Meter substation half-hourly),
--          D14 (Smart Meter LV feeder half-hourly)
CREATE TABLE demand_observation (
    observation_id         BIGSERIAL     PRIMARY KEY,
    substation_floc        VARCHAR(50)   NOT NULL,     -- FK to GSP, Primary, or Secondary
    feeder_id              VARCHAR(100),               -- FK to lv_feeder (NULL if substation-level)
    observation_level      VARCHAR(20)   NOT NULL,     -- 'GSP', 'Primary', 'Secondary', 'LV_Feeder'

    -- Time
    observation_timestamp  TIMESTAMP     NOT NULL,     -- Half-hourly for D12/D13/D14; annual date for D10/D11
    granularity            VARCHAR(20)   NOT NULL,     -- 'Half_Hourly', 'Annual_Peak'

    -- Demand values
    active_power_mw        DECIMAL(12,4),              -- MW (or kW for LV)
    reactive_power_mvar    DECIMAL(12,4),              -- MVAr
    active_energy_kwh      DECIMAL(14,4),              -- kWh (from smart meter datasets)
    reactive_energy_kvarh  DECIMAL(14,4),
    voltage_kv             DECIMAL(8,3),
    current_a              DECIMAL(10,2),

    -- Peak demand specifics (from D10, D11)
    is_observed_peak       BOOLEAN DEFAULT FALSE,      -- From LTDS Table 3a
    is_true_peak           BOOLEAN DEFAULT FALSE,      -- From LTDS Table 3b (corrected)
    peak_season            VARCHAR(10),                -- 'Winter', 'Summer'

    -- Metadata
    smart_meter_count      INTEGER,                    -- Number of meters in aggregation
    data_source            VARCHAR(50),                -- Dataset ID reference

    created_at             TIMESTAMP DEFAULT NOW()
);

-- Partition by time for performance
-- CREATE TABLE demand_observation_2024 PARTITION OF demand_observation
--     FOR VALUES FROM ('2024-01-01') TO ('2025-01-01');


-- ============================================================
-- SUPPLY OBSERVATIONS (current embedded generation output)
-- ============================================================
-- Sources: Derived from D12 (net power flow at GSP), D17 (connected capacity as baseline)
-- Note: UKPN does not publish generation output time series directly;
-- net flow at GSPs can be used to infer embedded generation contribution.
CREATE TABLE supply_observation (
    observation_id         BIGSERIAL     PRIMARY KEY,
    substation_floc        VARCHAR(50)   NOT NULL,
    observation_level      VARCHAR(20)   NOT NULL,     -- 'GSP', 'Primary'

    observation_timestamp  TIMESTAMP     NOT NULL,
    granularity            VARCHAR(20)   NOT NULL,

    -- Supply values
    net_export_mw          DECIMAL(12,4),              -- Negative = net import, Positive = net export
    generation_estimate_mw DECIMAL(12,4),              -- Estimated from net flow + demand
    embedded_capacity_mw   DECIMAL(12,4),              -- Total connected capacity at this node

    data_source            VARCHAR(50),
    created_at             TIMESTAMP DEFAULT NOW()
);
```

#### Embedded Capacity / Generation Register

```sql
-- ============================================================
-- EMBEDDED CAPACITY REGISTER (connected and queued generation)
-- ============================================================
-- Sources: D17 (Embedded Capacity Register), D18 (Large Demand List), D19 (GSP Project Status)
CREATE TABLE embedded_capacity_register (
    ecr_id                 SERIAL        PRIMARY KEY,
    site_name              VARCHAR(200),
    primary_floc           VARCHAR(50)   REFERENCES primary_substation(primary_floc),
    gsp_floc               VARCHAR(50)   REFERENCES grid_supply_point(gsp_floc),
    licence_area_id        VARCHAR(10)   REFERENCES licence_area(licence_area_id),

    -- Location
    postcode               VARCHAR(10),
    grid_reference         VARCHAR(20),
    latitude               DECIMAL(9,6),
    longitude              DECIMAL(9,6),

    -- Technical
    energy_source          VARCHAR(100),              -- 'Solar', 'Wind', 'Gas', 'Battery Storage', etc.
    technology_type        VARCHAR(100),              -- 'Photovoltaic', 'Onshore Wind', 'CCGT', 'Li-ion', etc.
    export_capacity_mw     DECIMAL(10,3),
    import_capacity_mw     DECIMAL(10,3),             -- For storage / demand sites
    connection_voltage_kv  DECIMAL(6,1),

    -- Status
    connection_status      VARCHAR(50),               -- 'Connected', 'Accepted', 'In Queue', 'Energised'
    flexible_connection    BOOLEAN,
    connection_date        DATE,

    updated_at             TIMESTAMP DEFAULT NOW()
);


-- ============================================================
-- CONNECTION QUEUE (projects awaiting connection)
-- ============================================================
-- Sources: D18 (Large Demand List), D19 (GSP Project Status)
CREATE TABLE connection_queue (
    project_id             SERIAL        PRIMARY KEY,
    gsp_floc               VARCHAR(50)   REFERENCES grid_supply_point(gsp_floc),
    primary_floc           VARCHAR(50)   REFERENCES primary_substation(primary_floc),

    project_type           VARCHAR(50),               -- 'Generation', 'Demand', 'Storage'
    capacity_mva           DECIMAL(10,2),
    status                 VARCHAR(50),               -- 'Accepted', 'Quoted', 'In Design'
    expected_connection    DATE,

    updated_at             TIMESTAMP DEFAULT NOW()
);
```

#### Forecast Tables

```sql
-- ============================================================
-- DFES FORECASTS (demand and supply headroom to 2050)
-- ============================================================
-- Sources: D20 (DFES Network Scenario Headroom Report)
CREATE TABLE dfes_forecast (
    forecast_id            BIGSERIAL     PRIMARY KEY,
    substation_floc        VARCHAR(50)   NOT NULL,
    forecast_level         VARCHAR(20)   NOT NULL,     -- 'BSP', 'Primary'

    -- Scenario
    scenario_name          VARCHAR(100)  NOT NULL,     -- 'Consumer Transformation', 'System Transformation',
                                                       -- 'Leading the Way', 'Falling Short', 'Holistic Transition'
    forecast_year          INTEGER       NOT NULL,     -- 2024..2050

    -- Headroom values (from D20)
    demand_headroom_mw     DECIMAL(10,2),              -- Spare capacity for new demand (positive = available)
    generation_headroom_mw DECIMAL(10,2),              -- Spare capacity for new generation
    firm_capacity_mw       DECIMAL(10,2),              -- Firm capacity assumed in this forecast
    forecast_peak_demand_mw DECIMAL(10,2),             -- Forecast peak demand under this scenario
    forecast_generation_mw DECIMAL(10,2),              -- Forecast connected generation

    -- Metadata
    dfes_publication_year  INTEGER,                    -- Year the DFES was published
    data_source            VARCHAR(50),

    created_at             TIMESTAMP DEFAULT NOW()
);

-- ============================================================
-- INFRASTRUCTURE PROJECTS (planned reinforcements)
-- ============================================================
-- Sources: D21 (LTDS Infrastructure Projects)
CREATE TABLE infrastructure_project (
    project_id             SERIAL        PRIMARY KEY,
    substation_floc        VARCHAR(50),               -- Primary or BSP FLOC
    licence_area_id        VARCHAR(10)   REFERENCES licence_area(licence_area_id),

    project_name           VARCHAR(300),
    project_description    TEXT,
    capacity_increase_mw   DECIMAL(10,2),
    planned_start_date     DATE,
    planned_completion_date DATE,
    project_status         VARCHAR(50),               -- 'Planned', 'In Progress', 'Completed'
    investment_driver       VARCHAR(100),              -- 'Load growth', 'Asset replacement', 'Net Zero'

    updated_at             TIMESTAMP DEFAULT NOW()
);
```

#### Load Profile Reference Table

```sql
-- ============================================================
-- STANDARD LOAD PROFILES (reference profiles by customer type)
-- ============================================================
-- Sources: D16 (Standard Profiles for Electricity Demand)
CREATE TABLE standard_load_profile (
    profile_id             SERIAL        PRIMARY KEY,
    profile_type           VARCHAR(100)  NOT NULL,     -- 'Domestic', 'Commercial', 'Industrial',
                                                       -- 'EV Charging', 'Bus Depot', 'Data Centre'
    profile_year           INTEGER       NOT NULL,
    half_hour_period       INTEGER       NOT NULL,     -- 1..17520 (48 periods * 365 days)
    day_of_year            DATE,
    time_of_day            TIME,

    demand_kw              DECIMAL(10,4),              -- Normalised demand (kW per unit)

    created_at             TIMESTAMP DEFAULT NOW()
);
```

#### Operational / Faults Table

```sql
-- ============================================================
-- NETWORK FAULTS (outage history)
-- ============================================================
-- Sources: D22 (Live Faults)
CREATE TABLE network_fault (
    fault_id               SERIAL        PRIMARY KEY,
    licence_area_id        VARCHAR(10)   REFERENCES licence_area(licence_area_id),

    fault_type             VARCHAR(50),               -- 'Planned', 'Unplanned'
    voltage_level          VARCHAR(20),
    affected_area          TEXT,
    postcode               VARCHAR(10),
    customers_affected     INTEGER,

    start_timestamp        TIMESTAMP,
    estimated_restoration  TIMESTAMP,
    actual_restoration     TIMESTAMP,

    data_source            VARCHAR(50),
    created_at             TIMESTAMP DEFAULT NOW()
);
```

---

## 5. Dataset-to-Table Mapping

This table shows exactly which UKPN dataset populates which database table:

| Dataset | Dataset ID | Target Table(s) | Join Key |
|---|---|---|---|
| D1: GSP Overview | `ukpn-grid-supply-points-overview` | `grid_supply_point` | `gsp_floc` |
| D2: GSP Distribution Areas | `ukpn-grid-supply-points` | `grid_supply_point.boundary_geojson` | `gsp_floc` |
| D3: Primary Distribution Areas | `ukpn_primary_postcode_area` | `primary_substation` | `primary_floc` |
| D4: Secondary Sites | `ukpn-secondary-sites` | `secondary_substation` | `secondary_floc` |
| D5: Secondary Site Transformers | `ukpn-secondary-site-transformers` | `transformer` (level='Secondary') | `secondary_floc` |
| D6: Secondary Distribution Areas | `ukpn_secondary_postcode_area` | `secondary_substation.boundary_geojson` | `secondary_floc` |
| D7: LTDS Table 2a | `ltds-table-2a` | `transformer` (level='GSP','Primary') | `substation_floc` |
| D8: LTDS Table 4 | (LTDS) | `transformer.fault_level_ka` | `substation_floc` |
| D9: EPR Data | `grid-and-primary-sites-epr-data` | `transformer` (EPR fields) | `substation_floc` |
| D10: LTDS Table 3a | `ltds-table-3a-load-data-observed` | `demand_observation` (annual peak) | `substation_floc` |
| D11: LTDS Table 3b | (LTDS) | `demand_observation` (true peak) | `substation_floc` |
| D12: Grid Transformer Power Flow | `ukpn-grid-transformer-operational-data-half-hourly` | `demand_observation`, `supply_observation` | `gsp_floc` |
| D13: Smart Meter -- Substation | `ukpn-smart-meter-consumption-substation` | `demand_observation` | `secondary_floc` |
| D14: Smart Meter -- LV Feeder | `ukpn-smart-meter-consumption-lv-feeder` | `demand_observation`, `lv_feeder` | `feeder_id` |
| D15: Secondary Utilisation | `ukpn-secondary-site-utilisation` | `secondary_substation` | `secondary_floc` |
| D16: Standard Load Profiles | `ukpn-standard-profiles-electricity-demand` | `standard_load_profile` | -- |
| D17: Embedded Capacity Register | `ukpn-embedded-capacity-register` | `embedded_capacity_register` | `primary_floc` |
| D18: Large Demand List | `ukpn-large-demand-list` | `connection_queue` | `gsp_floc` |
| D19: GSP Project Status | `ukpn-gsp-project-status` | `connection_queue` | `gsp_floc` |
| D20: DFES Headroom Report | `dfes-network-headroom-report` | `dfes_forecast` | `substation_floc` |
| D21: LTDS Infrastructure Projects | `ukpn-ltds-infrastructure-projects` | `infrastructure_project` | `substation_floc` |
| D22: Live Faults | `ukpn-live-faults` | `network_fault` | `licence_area_id` |
| D23: Network Statistics | `ukpn-network-statistics` | (reference / dashboard) | `licence_area_id` |
| D24: Data Centre Utilisation | `ukpn-data-centre-utilisation` | (reference / dashboard) | -- |

---

## 6. Key Relationships & Join Paths

### 6.1 Navigating the hierarchy (top-down)

```sql
-- From GSP down to secondary substations
SELECT
    g.gsp_name,
    b.bsp_name,
    p.primary_name,
    p.firm_capacity_mw,
    s.secondary_name,
    s.onan_rating_kva,
    s.customer_count
FROM grid_supply_point g
JOIN bulk_supply_point b    ON b.gsp_floc     = g.gsp_floc
JOIN primary_substation p   ON p.bsp_floc     = b.bsp_floc
JOIN secondary_substation s ON s.primary_floc = p.primary_floc
WHERE g.licence_area_id = 'LPN';   -- London
```

### 6.2 Current demand at a primary substation

```sql
-- Latest observed peak demand + utilisation
SELECT
    p.primary_name,
    p.firm_capacity_mw,
    d.active_power_mw    AS observed_peak_mw,
    d.peak_season,
    (d.active_power_mw / p.firm_capacity_mw * 100) AS utilisation_pct
FROM primary_substation p
JOIN demand_observation d ON d.substation_floc = p.primary_floc
WHERE d.is_observed_peak = TRUE
  AND d.granularity = 'Annual_Peak'
ORDER BY utilisation_pct DESC;
```

### 6.3 Embedded generation at a primary substation

```sql
-- Total connected generation capacity by energy source
SELECT
    p.primary_name,
    e.energy_source,
    COUNT(*)                    AS site_count,
    SUM(e.export_capacity_mw)  AS total_export_mw
FROM primary_substation p
JOIN embedded_capacity_register e ON e.primary_floc = p.primary_floc
WHERE e.connection_status = 'Connected'
GROUP BY p.primary_name, e.energy_source
ORDER BY total_export_mw DESC;
```

### 6.4 Forecast headroom to 2050

```sql
-- Which primary substations will be constrained by 2030?
SELECT
    p.primary_name,
    f.scenario_name,
    f.forecast_year,
    f.demand_headroom_mw,
    f.generation_headroom_mw
FROM primary_substation p
JOIN dfes_forecast f ON f.substation_floc = p.primary_floc
WHERE f.forecast_year = 2030
  AND f.demand_headroom_mw < 0    -- Negative headroom = constraint
ORDER BY f.demand_headroom_mw ASC;
```

### 6.5 Supply vs demand at a GSP (half-hourly)

```sql
-- Half-hourly net power flow at a GSP (positive = exporting to grid)
SELECT
    g.gsp_name,
    d.observation_timestamp,
    d.active_power_mw,
    s.net_export_mw,
    s.generation_estimate_mw
FROM grid_supply_point g
JOIN demand_observation d  ON d.substation_floc = g.gsp_floc
LEFT JOIN supply_observation s ON s.substation_floc = g.gsp_floc
                              AND s.observation_timestamp = d.observation_timestamp
WHERE d.granularity = 'Half_Hourly'
  AND d.observation_timestamp BETWEEN '2024-01-01' AND '2024-01-02'
ORDER BY d.observation_timestamp;
```

---

## 7. Expected Outputs

With this schema fully populated, you can produce the following analyses:

### 7.1 Current State

| Output | Description | Tables Used |
|---|---|---|
| **Full network topology map** | Interactive GIS map from GSP to secondary substation with supply areas | All hierarchy tables + `boundary_geojson` fields |
| **Capacity utilisation dashboard** | RAG status at every level showing how close each node is to its firm capacity | `primary_substation`, `secondary_substation`, `demand_observation` |
| **Demand heatmap** | Geographic heatmap of current peak demand overlaid on substation distribution areas | `primary_substation`, `demand_observation`, `boundary_geojson` |
| **Embedded generation register** | Map of all connected DER by technology type and capacity | `embedded_capacity_register`, `primary_substation` |
| **Half-hourly load curves** | Time-series demand profiles at GSP, substation, and feeder level | `demand_observation` |
| **Net import/export analysis** | Which GSPs are net exporters (embedded generation exceeds demand) | `demand_observation`, `supply_observation` |
| **Connection queue analysis** | Volume and capacity of projects awaiting connection at each GSP | `connection_queue`, `grid_supply_point` |

### 7.2 Forecast

| Output | Description | Tables Used |
|---|---|---|
| **Constraint forecast to 2050** | List of substations that will exceed firm capacity under each DFES scenario, and when | `dfes_forecast`, `primary_substation`, `bulk_supply_point` |
| **Reinforcement pipeline** | Planned infrastructure projects mapped to constrained substations | `infrastructure_project`, `dfes_forecast` |
| **EV / heat pump impact** | Forecast demand growth from electrification overlaid on available headroom | `dfes_forecast`, `standard_load_profile` |
| **Generation growth forecast** | Projected growth in embedded generation by technology and location | `dfes_forecast`, `embedded_capacity_register` |
| **Investment prioritisation** | Rank substations by severity and timing of constraint for capital planning | `dfes_forecast`, `infrastructure_project`, `primary_substation` |

### 7.3 Cross-DNO (future extensibility)

| Output | Description |
|---|---|
| **National constraint map** | Aggregate constraint data across all DNOs using the standardised schema |
| **Technology deployment comparison** | Compare EV, heat pump, solar uptake rates across DNO regions |
| **Connection queue national view** | Total pipeline of generation and demand connections nationally |

---

## 8. Extensibility for Other DNOs

The schema is designed to be **DNO-agnostic**. To onboard a new DNO:

1. **Add a row to `licence_area`** for each of the DNO's licence areas
2. **Map the DNO's field names** to the schema columns (a mapping table or ETL config per DNO)
3. **Populate the hierarchy tables** -- every DNO publishes LTDS data covering GSP, BSP, and Primary substations
4. **Ingest the Embedded Capacity Register** -- mandated format is consistent across all DNOs
5. **Load DFES forecasts** -- scenario names and granularity vary slightly but the structure is consistent

Key differences between DNOs that the schema accommodates:
- **FLOC format** varies (UKPN uses SAP FLOCs; others use different ID schemes). The `VARCHAR(50)` primary key is flexible enough for any format.
- **Data granularity** varies (some DNOs lack half-hourly substation data). The `granularity` field in observation tables handles this.
- **Secondary substation data** is sparse for non-UKPN DNOs. Secondary tables can be partially populated.
- **DFES scenario names** vary by DNO. The `scenario_name` field is free-text to accommodate this.

---

## 9. Data Access & Ingestion

### UKPN OpenDataSoft API

Base URL: `https://ukpowernetworks.opendatasoft.com/api/explore/v2.1/`

Example API calls:

```bash
# List all datasets
curl "https://ukpowernetworks.opendatasoft.com/api/explore/v2.1/catalog/datasets"

# Get GSP overview data as JSON
curl "https://ukpowernetworks.opendatasoft.com/api/explore/v2.1/catalog/datasets/ukpn-grid-supply-points-overview/records?limit=100"

# Get primary substation areas as GeoJSON
curl "https://ukpowernetworks.opendatasoft.com/api/explore/v2.1/catalog/datasets/ukpn_primary_postcode_area/exports/geojson"

# Get half-hourly grid transformer data for a date range
curl "https://ukpowernetworks.opendatasoft.com/api/explore/v2.1/catalog/datasets/ukpn-grid-transformer-operational-data-half-hourly/records?where=date_time>='2024-01-01'&limit=100"

# Export embedded capacity register as CSV
curl "https://ukpowernetworks.opendatasoft.com/api/explore/v2.1/catalog/datasets/ukpn-embedded-capacity-register/exports/csv"
```

Free registration is required for API access. Rate limits apply.

### Recommended Ingestion Order

1. `licence_area` (manual)
2. `grid_supply_point` (D1 + D2)
3. `bulk_supply_point` (from LTDS/DFES)
4. `primary_substation` (D3)
5. `secondary_substation` (D4 + D5 + D6 + D15)
6. `lv_feeder` (D14)
7. `transformer` (D7 + D5 + D9)
8. `embedded_capacity_register` (D17)
9. `connection_queue` (D18 + D19)
10. `demand_observation` (D10 + D11 + D12 + D13 + D14)
11. `supply_observation` (derived from D12)
12. `dfes_forecast` (D20)
13. `infrastructure_project` (D21)
14. `standard_load_profile` (D16)
15. `network_fault` (D22)

---

## 10. Technology Recommendations

| Component | Recommendation | Rationale |
|---|---|---|
| **Database** | PostgreSQL + PostGIS | Native geospatial support (GeoJSON, spatial queries), time-series partitioning, JSONB for flexible fields |
| **Ingestion** | Python (pandas + requests) | Simple API/CSV ingestion; psycopg2 for PostgreSQL loading |
| **Scheduling** | cron / Apache Airflow | Periodic refresh of half-hourly data and monthly ECR updates |
| **Visualisation** | QGIS / Kepler.gl / Grafana | GIS mapping, time-series dashboards |
| **API layer** | FastAPI / PostgREST | Expose the database as a REST API for downstream applications |
