# futurebeeing_energy_sim_data_description
Description of the resources and calculations used to create the NRW dataset for the simulations in the energy menu card of the FutureBeeing tool in Phase 1 on a building level.

This table offers an overview of the parameters, that are saved in the database
| Name | Description | Type | Nullable | Data Type | Primary Key |
|---|---|---|---|---|---|
| `id` | Incrementally generated id of this building in the DB | integer | false | bigserial | ✅ |
| `size_class` | Size class of the building determined by ETHOS DB | string | true | varchar(18) | |
| `tabula_key` | Tabula type of the building determined by the ETHOS DB | string | true | varchar(18) | |
| `construction_year` | Construction year of the building determined by the ETHOS DB | number | true | integer | |
| `refurbishment_state` | Refurbishment state of the building determined by the ETHOS DB | number | true | integer | |
| `living_area` | Living area of the building determined by own methods | number | true | float | |
| `heat_demand_1` | Heat demand per year [kWh/a] considering refurbishment state 1 | number | true | float | |
| `heat_demand_2` | Heat demand per year [kWh/a] considering refurbishment state 2 | num

### Tabula type, refurbishment state and height
Tabula types:
License: ODC Open Database License v1.0
URL: https://zenodo.org/records/12069755

Height and footprints:
License: ODC Open Database License v1.0
URL: https://zenodo.org/records/11845992



### Heat Demand
URL: https://www.sciencedirect.com/science/article/pii/S0360132325002641 Supplementary Materials S3
License: CC BY 4.0
Living area calculation S3: Download: Download Word document (80KB)

TABULA – Typology Approach for Building Stock Energy Assessment
URL: https://webtool.building-typology.eu/#bm



### PV potential
URL: https://www.opengeodata.nrw.de/produkte/umwelt_klima/energie/solarkataster/photovoltaik/
License: DL-DE->Zero-2.0

### Electricity demand


1 person housholds: Jährlicher Stromverbrauch eines 1-Personen-Haushalts in Deutschland nach Gebäudetyp im Jahr 2025 (in Kilowattstunden) [Graph], co2online, 13. Mai, 2025. [Online]. Verfügbar: https://de.statista.com/statistik/daten/studie/558239/umfrage/stromverbrauch-einen-1-personen-haushalts-in-deutschland/

2 person households: Jährlicher Stromverbrauch eines 2-Personen-Haushalts in Deutschland nach Gebäudetyp im Jahr 2025 (in Kilowattstunden) [Graph], co2online, 13. Mai, 2025. [Online]. Verfügbar: https://de.statista.com/statistik/daten/studie/558277/umfrage/stromverbrauch-einen-2-personen-haushalts-in-deutschland/

3 person households: Jährlicher Stromverbrauch eines 3-Personen-Haushalts in Deutschland nach Gebäudetyp im Jahr 2025 (in Kilowattstunden) [Graph], co2online, 13. Mai, 2025. [Online]. Verfügbar: https://de.statista.com/statistik/daten/studie/558280/umfrage/stromverbrauch-einen-3-personen-haushalts-in-deutschland/

4 person households: Jährlicher Stromverbrauch eines 4-Personen-Haushalts in Deutschland nach Gebäudetyp im Jahr 2023 (in Kilowattstunden) [Graph], co2online, 4. April, 2023. [Online]. Verfügbar: https://de.statista.com/statistik/daten/studie/558288/umfrage/stromverbrauch-einen-4-personen-haushalts-in-deutschland/

5 person households: Jährlicher Stromverbrauch eines 5-Personen-Haushalts in Deutschland nach Gebäudetyp im Jahr 2025 (in Kilowattstunden) [Graph], co2online, 13. Mai, 2025. [Online]. Verfügbar: https://de.statista.com/statistik/daten/studie/558295/umfrage/stromverbrauch-einen-5-personen-haushalts-in-deutschland/
