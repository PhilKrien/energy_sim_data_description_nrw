# FutureBeeing Energy Simulation Data – NRW Building Stock

## About

**Project:** FutureBeeing – Phase 1, Energy Menu Card  
**Institution:** FH Münster, Labor für Energiesystemmodellierung  
**Author:** Philippe Krienelke  
**Contact:** philippe.krienelke@fh-muenster.de  
**Date:** 03.06.2026  
**Version:** 1.0.0  

## Description

This repository documents the data sources, assumptions, and calculation
methods used to generate the NRW building-level dataset for the energy
simulation component of the FutureBeeing tool. The dataset covers all
residential buildings in NRW and provides annual estimates for heat demand,
electricity demand, and PV potential at the building level. The resulting data set will be published to the Open Energy Platform (OEP).

## Scope and Intended Use

- **Geographic scope:** North Rhine-Westphalia (NRW), Germany
- **Building type:** Residential buildings only
- **Spatial resolution:** Individual building level (point geometry)
- **Intended aggregation level:** District or municipal level
- **Not suitable for:** Individual building-level decision making
  
This table offers an overview of the parameters, that are saved in the database
| Column               | Type          | Nullable | Description                                                              |
|----------------------|---------------|----------|--------------------------------------------------------------------------|
| `id`                 | `bigserial`   | ❌       | Incrementally generated ID (primary key)                                 |
| `size_class`         | `varchar(18)` | ✅       | Size class of the building (ETHOS DB)                                    |
| `tabula_key`         | `varchar(18)` | ✅       | TABULA type of the building (ETHOS DB)                                   |
| `construction_year`  | `integer`     | ✅       | Construction year (ETHOS DB)                                             |
| `refurbishment_state`| `integer`     | ✅       | Refurbishment state 1–3 (ETHOS DB)                                       |
| `living_area`        | `float`       | ✅       | Conditioned living area Al [m2]                                          |
| `heat_demand_1`      | `float`       | ✅       | Annual heat demand at refurbishment state 1 [kWh/a]                      |
| `heat_demand_2`      | `float`       | ✅       | Annual heat demand at refurbishment state 2 [kWh/a]                      |
| `heat_demand_3`      | `float`       | ✅       | Annual heat demand at refurbishment state 3 [kWh/a]                      |
| `elec_demand`        | `float`       | ✅       | Annual electricity demand [kWh/a]                                        |
| `pv_potential`       | `float`       | ✅       | Annual PV potential if all roof areas used [kWh/a]                       |
| `heat_technology`    | `varchar(18)` | ✅       | Heating system (Zensus 2020, statistical/heuristic)                      |
| `geometry`           | `geometry`    | ✅       | Point location of the building                                           |

### Tabula type, refurbishment state and height

**TABULA types, construction year, size class, refurbishment state:**
License: ODC Open Database License v1.0
URL: https://zenodo.org/records/12069755

**Height and footprints:**
License: ODC Open Database License v1.0
URL: https://zenodo.org/records/11845992

The attributes `tabula_key`, `construction_year`, `size_class`, and
`refurbishment_state` are sourced from the ETHOS.BUILDA database
(Dabrock et al., 2025, doi: 10.1016/j.buildenv.2025.112782), which covers
all residential buildings in Germany.

Building heights and footprints are sourced from official government LoD2
3-D building datasets for NRW and nine other federal states, and from
OpenStreetMap for the remaining states (Dabrock et al., 2024,
doi: 10.1016/j.egyai.2024.100408)

#### Model Accuracy

`size_class` and `construction_year` are predicted by machine learning
models trained on a combination of German census grid data, building and
neighborhood morphology features (footprint area, height, roof shape,
shared walls, etc.), and socio-economic characteristics (income, unemployment
rate, education level, regional type). `refurbishment_state` is assigned
statistically based on federal state level data — no individual building
validation is possible for this attribute.
| Attribute             | Accuracy    |
|-----------------------|:-----------:|
| `size_class`          | **97.4 %**  |
| `construction_year`   | **73.9 %**  |
| `refurbishment_state` | —           |

#### Limitations

- **Size class** is highly accurate overall, but apartment blocks (AB)
  are underdetected — they are frequently misclassified as MFH.
- **Construction year** is less reliable for buildings built after 1979.
  Older buildings (pre-1979) are overrepresented in the dataset.
- **Refurbishment state** has no building-level validation. Spatial
  differences within federal states (e.g. urban vs. rural) are not captured.
- All three attributes directly determine `heat_demand_1/2/3` — errors
  propagate into the heat demand values.
- Results are **not reliable at the individual building level** and should
  be aggregated to district or municipal level for meaningful conclusions.



### Heat Demand

**TABULA – Typology Approach for Building Stock Energy Assessment**  
URL: https://webtool.building-typology.eu/#bm  
License: CC BY 4.0

**Living Area Calculation**  
URL: https://www.sciencedirect.com/science/article/pii/S0360132325002641 (Supplementary Materials S3)  
License: CC BY 4.0

The heat demand of a building is calculated from its conditioned living area
$A_l$ and a specific heat consumption value derived from the TABULA typology.
Two components are considered: space heating and domestic hot water (DHW).

#### Step 1: Conditioned Living Area

The conditioned living area $A_l$ is derived from building geometry following
EnEV 2009 and Loga et al. 2012 and how it was described in the Supplimentary Material of the ETHOS Paper:

$$h_{\text{heated}} = \max\left(h_{\text{building}} - \Delta h_{\text{roof}},\ 
h_{\text{min}}\right)$$

$$V = A_{\text{footprint}} \cdot h_{\text{heated}}$$

$$A_u = 0.32 \cdot V$$

$$A_l = \frac{A_u}{1.3}$$

where:
- $h_{\text{building}}$ is the total building height in metres
- $\Delta h_{\text{roof}} = 3.0\,\text{m}$ subtracts the non-heated roof space
- $h_{\text{min}} = 3.5\,\text{m}$ prevents unrealistically small values
- $A_{\text{footprint}}$ is the building footprint area
- The factor $0.32$ follows EnEV 2009
- The factor $1.3$ follows Loga et al. 2012

#### Step 2: Specific Heat Demand from TABULA

For each building, a `tabula_key` is assigned based on the first four
components of the `tabula_type` (country, type, year class, size class).
The TABULA dataset provides two specific energy use intensities (EUI) in
kWh/(m²·a):

- $d_{\text{heat}}$ — space heating EUI
- $d_{\text{dhw}}$ — domestic hot water EUI

Both values depend on the `tabula_key` and the `refurbishment_state`:

| `refurbishment_state` | Description |
|:---:|---|
| 1 | Existing state (no refurbishment) |
| 2 | Usual refurbishment |
| 3 | Advanced refurbishment |

#### Step 3: Total Annual Heat Demand

The total annual heat demand of a building is:

$$Q_{\text{building}} = \left(d_{\text{heat}} + d_{\text{dhw}}\right) \cdot A_l$$

where $d_{\text{heat}}$ and $d_{\text{dhw}}$ are looked up from
`tabula_heat_demand.json` based on `tabula_key` and `refurbishment_state`.

#### Limitations
- $A_l$ is estimated from building geometry and does not account for 
  internal layout or non-residential floor uses.
- TABULA EUI values are archetype averages and do not reflect individual 
  building conditions.
- DHW demand is treated as independent of occupancy; no per-person scaling 
  is applied.
- Refurbishment state is an input assumption; actual refurbishment levels 
  are not available in the building stock data.





### PV potential
URL: https://www.opengeodata.nrw.de/produkte/umwelt_klima/energie/solarkataster/photovoltaik/
License: DL-DE->Zero-2.0

The pv Potential was directly retrieved from the Solarkataster NRW dataset. It shows the maximum value in kWh/a that could be produced, if all available roof potentials were used. 

### Electricity demand
Zensus 2022 Statistisches Bundesamt
https://www.destatis.de/DE/Themen/Gesellschaft-Umwelt/Bevoelkerung/Zensus2022/_inhalt.html
License: DL-DE->Zero-2.0

The eletricity demand of a building correlates directly to the number of dwellings and the number of persons in a dwelling. In order to calculate these for every buildings, some assumptions had to be made:
1. the number of inhabitants per dwelling and the size of all dwellingsg in one building is homogenous
2. these values are dependant on the assumed tabula type
3. SFH and TH are considered to only have 1 dwelling, while MFH and AB have multiple dwellings that need to be calculated indivisually

#### Step 1: Average Dwelling Size and Inhabitants per TABULA Type

In order to determine the two values 

$$\bar{A}_{\text{tab}}$$ (average dwelling size)

$$\bar{n}_{\text{tab}}$$ (average number of inhabitants per dwelling)

for each `tabula_key`, the ETHOS dataset and the Zensus 2022 dataset were combined. The Zensus dataset 
provides these values in 100 m × 100 m rasters (chosen by the Federal 
Statistical Office for anonymization). Rasters with homogeneous TABULA type 
distributions are identified and used to derive the required averages. 
Results are stored in `tabula_stats.json`.

#### Step 2: Number of Dwellings per Building

For SFH and TH, the number of dwellings is fixed by definition:

$$N_{dw} = 1 \quad \text{for SFH, TH}$$

For MFH and AB, the number of dwellings is estimated by dividing the total 
living area $A_{living}$ by the average dwelling size for the corresponding 
TABULA type:

$$N_{dw} = \left\lfloor \frac{A_{living}}{\bar{A}_{tabula}} \right\rceil, 
\quad N_{dw} \geq 1$$

where $\lfloor \cdot \rceil$ denotes rounding to the nearest integer.

#### Step 3: Electricity Demand per Building

The annual electricity demand per dwelling $d_{dw}$ depends on the household 
size $n_{persons}$ (clipped to $[1, 5]$) and the building size class, based 
on co2online consumption data for Germany (2023–2025):

The electricity demands for different household sizes and building `size_class` were extracted from the publications down below:

1 person housholds: Jährlicher Stromverbrauch eines 1-Personen-Haushalts in Deutschland nach Gebäudetyp im Jahr 2025 (in Kilowattstunden) [Graph], co2online, 13. Mai, 2025. [Online]. Verfügbar: https://de.statista.com/statistik/daten/studie/558239/umfrage/stromverbrauch-einen-1-personen-haushalts-in-deutschland/

2 person households: Jährlicher Stromverbrauch eines 2-Personen-Haushalts in Deutschland nach Gebäudetyp im Jahr 2025 (in Kilowattstunden) [Graph], co2online, 13. Mai, 2025. [Online]. Verfügbar: https://de.statista.com/statistik/daten/studie/558277/umfrage/stromverbrauch-einen-2-personen-haushalts-in-deutschland/

3 person households: Jährlicher Stromverbrauch eines 3-Personen-Haushalts in Deutschland nach Gebäudetyp im Jahr 2025 (in Kilowattstunden) [Graph], co2online, 13. Mai, 2025. [Online]. Verfügbar: https://de.statista.com/statistik/daten/studie/558280/umfrage/stromverbrauch-einen-3-personen-haushalts-in-deutschland/

4 person households: Jährlicher Stromverbrauch eines 4-Personen-Haushalts in Deutschland nach Gebäudetyp im Jahr 2023 (in Kilowattstunden) [Graph], co2online, 4. April, 2023. [Online]. Verfügbar: https://de.statista.com/statistik/daten/studie/558288/umfrage/stromverbrauch-einen-4-personen-haushalts-in-deutschland/

5 person households: Jährlicher Stromverbrauch eines 5-Personen-Haushalts in Deutschland nach Gebäudetyp im Jahr 2025 (in Kilowattstunden) [Graph], co2online, 13. Mai, 2025. [Online]. Verfügbar: https://de.statista.com/statistik/daten/studie/558295/umfrage/stromverbrauch-einen-5-personen-haushalts-in-deutschland/

> **Note:** The 4-person household consumption value stems from a 2023 
> publication, while all others use 2025 data. No updated figure was 
> available at the time of writing.

$$d_{dw}(n, \text{type}) = 
\begin{cases} 
d_{EFH}(n) & \text{if type} \in \{\text{SFH, TH}\} \\ 
d_{MFH}(n) & \text{if type} \in \{\text{MFH, AB}}
\end{cases}$$

The total annual electricity demand of a building is then:

$$E_{building} = N_{dw} \cdot d_{dw}\!\left(\bar{n}_{tabula},\ c\right)$$

where $c$ denotes the `size_class` of the building.

$\bar{n}_{tabula}$ is the average number of inhabitants per dwelling 
for the corresponding TABULA type, rounded to the nearest integer.

#### Limitations
- Dwelling size and inhabitant count are assumed homogenous within a building.
- Consumption values are national averages and do not reflect regional 
  variation within NRW.
- Buildings with `avg_inhabitants > 5` are clipped to 5-person consumption.
- The scope of the FutureBeeing tool is on a quarter level analysis, not on building sharp decision making.

### Heating Technology

**Source:** Zensus 2022 – Statistisches Bundesamt
URL: https://www.destatis.de/DE/Themen/Gesellschaft-Umwelt/Bevoelkerung/Zensus2022/_inhalt.html
License: DL-DE->Zero-2.0

The `heat_technology` attribute is assigned heuristically based on the
heating system distribution reported in the Zensus 2022 dataset. The census
provides the number of installed heating systems per technology for 100 m ×
100 m raster cells. These shares are used to assign a heating technology to
each individual building within the corresponding raster cell.

#### Assignment Heuristic

The share of each technology within a raster cell is calculated and
translated into a number of buildings to be assigned that technology:

$$n_{\text{tech}} = \left\lfloor s_{\text{tech}} \cdot N_{\text{buildings}} \right\rfloor$$

where $s_{\text{tech}}$ is the share of a technology in the raster cell and
$N_{\text{buildings}}$ is the total number of buildings in the cell.

Technologies are assigned in the following priority order, each with a
building-type preference:

| Technology       | Preferred building type | Rationale |
|------------------|------------------------|-----------|
| `hp`             | SFH, TH first          | Heat pumps are more common in smaller buildings with outdoor space |
| `district_heat`  | MFH, AB first          | District heating is economically favored for large consumers |
| `elec`           | SFH, TH first          | Electric heating more common in smaller buildings |
| `oil`            | SFH, TH first          | Oil heating predominantly found in older detached buildings |
| `biomass`        | SFH, TH first          | Biomass heating typical for rural single-family buildings |
| `gas`            | remainder              | Default technology for all unassigned buildings |

Within MFH/AB, buildings are sorted by total heat demand descending before
assignment — larger, older buildings are assigned less efficient technologies
first.

If no technology data is available for a raster cell, all buildings in that
cell are assigned `gas` as a default.

#### Heuristic Validation

To verify that the assignment heuristics are empirically supported by the
underlying Zensus data, the dominant heating technology per raster cell was
determined from the raw Zensus 2022 counts (`idxmax` over absolute technology
counts). For each dominant technology, the mean and median share of MFH/AB
and SFH/TH buildings within the corresponding raster cells was calculated.

The table below summarizes the findings:

| Technology       | n cells | Mean MFH share | Mean SFH share | Median MFH share | Confirmed |
|------------------|--------:|:--------------:|:--------------:|:----------------:|:---------:|
| `district_heat`  |  20,824 |      0.550     |      0.450     |      0.714       | ✅        |
| `elec`           |   6,233 |      0.226     |      0.774     |      0.000       | ✅        |
| `gas`            | 342,373 |      0.225     |      0.775     |      0.000       | ✅        |
| `biomass`        |     169 |      0.165     |      0.835     |      0.000       | ✅        |
| `oil`            |  81,038 |      0.066     |      0.934     |      0.000       | ✅        |
| `hp`             |   8,933 |      0.054     |      0.946     |      0.000       | ✅        |

The results confirm the assumed preferences: cells dominated by district
heating show the highest MFH/AB share (mean 55 %, median 71 %), while cells
dominated by heat pumps and oil heating are strongly SFH/TH-dominated
(~94 % and ~93 % respectively).

#### Limitations

- Heating technologies are assigned at the raster cell level, not at the
  individual building level — within a cell, the assignment is heuristic.
- The Zensus data reflects the building stock as of 2022 and does not
  capture subsequent technology changes (e.g. heat pump installations).
- The preference rules (e.g. heat pumps → SFH/TH) reflect statistical
  tendencies and will not be accurate for every individual building.
- Due to integer rounding of technology counts, a small share of buildings
  is always assigned gas regardless of the actual gas share in the cell.


