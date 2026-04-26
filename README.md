# Weekly Self-Report Participatory Surveillance — Codebook

This document describes the data model for a weekly self-report system for syndromic surveillance, with optional One Health extensions covering human cases, livestock, wildlife, and environmental observations. It is the input/output specification for the survey screens shown to participants and for the records produced by each weekly submission.

- **Variables:** 85
- **Sections:** 17
- **Unit of observation:** one record per participant per weekly submission (`user_id` + `report_id`).
- **Companion file:** `synthetic_data.json` — 35 synthetic weekly reports generated from a panel of 8 recurring participants over six weeks (March 14 – April 25, 2026).

## Contents

1. [Data model and conventions](#data-model-and-conventions)
2. [Missing values](#missing-values)
3. [Skip logic](#skip-logic)
4. [Variables by section](#variables-by-section)
   - [Identifiers](#identifiers)
   - [Weekly check-in](#weekly-check-in)
   - [Illness episode](#illness-episode)
   - [Care seeking](#care-seeking)
   - [Symptoms - whole body](#symptoms--whole-body)
   - [Symptoms - respiratory](#symptoms--respiratory)
   - [Symptoms - digestive](#symptoms--digestive)
   - [Symptoms - other](#symptoms--other)
   - [Symptoms - extended](#symptoms--extended)
   - [Severity](#severity)
   - [Exposure](#exposure)
   - [Auxiliary](#auxiliary)
   - [Demographics](#demographics)
   - [Vaccination](#vaccination)
   - [Environmental](#environmental)
   - [Livestock](#livestock)
   - [Wildlife](#wildlife)
5. [Value labels reference](#value-labels-reference)
6. [Synthetic dataset](#synthetic-dataset)

## Data model and conventions

Each weekly submission yields one row keyed by `report_id`. `user_id` is stable across submissions and links a participant's weekly history. Demographics and vaccination fields are collected at registration and repeated on every record for convenience; in production they would normally live in a separate participant table joined on `user_id`.

Variable types used in this codebook:

| Type | Meaning |
|---|---|
| `string`      | Free text or identifier. |
| `integer`     | Whole number with a defined range. |
| `binary`      | `0` = no / not selected, `1` = yes / selected. |
| `categorical` | Coded value from a fixed list (see value labels). |
| `date`        | ISO date in `YYYY-MM-DD`. |

## Missing values

- Numeric and categorical variables use **`-99`** for missing.
- Date and string variables use the empty/null value for missing.
- Variables not asked due to skip logic (for example, symptom fields when `health_status = 1`) are stored as null/empty rather than `-99`.

## Skip logic

- `health_status = 1` (Healthy): the entire illness episode block is skipped, including symptom onset, at-home tests, all symptom variables, all care-seeking variables, severity markers, auxiliary fields, and `diagnostic_lab_confirmation`. Demographics, vaccination, and exposure questions remain in scope.
- `health_status = 2` (Not feeling well): the full symptom and care block is asked. Within that block, `at_home_test_none` is mutually exclusive with `at_home_test_covid` and `at_home_test_flu`.
- `sym_other = 1` requires `sym_other_text` (free text).
- One Health modules (Environmental, Livestock, Wildlife) are optional and asked only when the participant elects to report an incident, regardless of `health_status`.

## Variables by section

### Identifiers

| Variable | Label | Type | Valid values | Missing | Skip logic | Notes |
|---|---|---|---|---|---|---|
| `user_id` | Persistent participant identifier | `string` | 8-char alphanumeric (e.g., U_3F9A2K) | — | — | Stable across reports; assigned at registration. |
| `report_id` | Unique weekly report identifier | `string` | 12-char UUID prefix (e.g., R_7B2C-44A1) | — | — | One per submitted survey. |
| `report_date` | Date the weekly survey was submitted | `date` | YYYY-MM-DD | — | — | System-stamped at submission. |
| `report_week` | ISO week number of report | `integer` | 1-53 | — | — | Derived from report_date. |
| `report_year` | Report year | `integer` | 2020-2030 | — | — | Derived from report_date. |

### Weekly check-in

| Variable | Label | Type | Valid values | Missing | Skip logic | Notes |
|---|---|---|---|---|---|---|
| `health_status` | How are you feeling this week? | `categorical` | 1, 2 | -99 | — | Entry point of the survey. If 1=Healthy, most symptom and care fields are skipped. |

### Illness episode

| Variable | Label | Type | Valid values | Missing | Skip logic | Notes |
|---|---|---|---|---|---|---|
| `symptom_onset_date` | Date symptoms started | `date` | YYYY-MM-DD | 1900-01-01 | Asked only if health_status = 2 | May precede report_date by several days. |
| `at_home_test_covid` | Took an at-home COVID-19 test in last 7 days | `binary` | 0, 1 | -99 | Asked only if health_status = 2 |  |
| `at_home_test_flu` | Took an at-home influenza test in last 7 days | `binary` | 0, 1 | -99 | Asked only if health_status = 2 |  |
| `at_home_test_none` | Did not take any at-home test | `binary` | 0, 1 | -99 | Asked only if health_status = 2 | Mutually exclusive with the two test variables above. |

### Care seeking

| Variable | Label | Type | Valid values | Missing | Skip logic | Notes |
|---|---|---|---|---|---|---|
| `care_no_seek` | Did not seek care | `binary` | 0, 1 | -99 | Asked only if health_status = 2 |  |
| `care_tried_unable` | Tried to seek care but was unable to | `binary` | 0, 1 | -99 | Asked only if health_status = 2 |  |
| `care_doctor` | Doctor's office | `binary` | 0, 1 | -99 | Asked only if health_status = 2 |  |
| `care_urgent` | Urgent care center | `binary` | 0, 1 | -99 | Asked only if health_status = 2 |  |
| `care_clinic` | In-store clinic / walk-in clinic | `binary` | 0, 1 | -99 | Asked only if health_status = 2 |  |
| `care_er` | Hospital emergency room | `binary` | 0, 1 | -99 | Asked only if health_status = 2 |  |
| `care_hospital` | Hospital (stayed overnight) | `binary` | 0, 1 | -99 | Asked only if health_status = 2 |  |
| `care_icu` | Hospital Intensive Care Unit (ICU) | `binary` | 0, 1 | -99 | Asked only if health_status = 2 |  |
| `care_virtual` | Virtual visit / telehealth | `binary` | 0, 1 | -99 | Asked only if health_status = 2 |  |
| `care_other` | Other care setting | `binary` | 0, 1 | -99 | Asked only if health_status = 2 |  |

### Symptoms - whole body

| Variable | Label | Type | Valid values | Missing | Skip logic | Notes |
|---|---|---|---|---|---|---|
| `sym_chills_sweats` | Chills / sweating / night sweats | `binary` | 0, 1 | -99 | Asked only if health_status = 2 |  |
| `sym_aches` | Muscle / body aches and pains | `binary` | 0, 1 | -99 | Asked only if health_status = 2 |  |
| `sym_fatigue` | Fatigue | `binary` | 0, 1 | -99 | Asked only if health_status = 2 |  |
| `sym_fever` | Fever | `binary` | 0, 1 | -99 | Asked only if health_status = 2 |  |

### Symptoms - respiratory

| Variable | Label | Type | Valid values | Missing | Skip logic | Notes |
|---|---|---|---|---|---|---|
| `sym_cough` | Cough | `binary` | 0, 1 | -99 | Asked only if health_status = 2 |  |
| `sym_chest_tight` | Chest tightness | `binary` | 0, 1 | -99 | Asked only if health_status = 2 |  |
| `sym_sore_throat` | Sore throat | `binary` | 0, 1 | -99 | Asked only if health_status = 2 |  |
| `sym_short_breath` | Shortness of breath | `binary` | 0, 1 | -99 | Asked only if health_status = 2 |  |
| `sym_loss_smell_taste` | Loss or change in smell / taste | `binary` | 0, 1 | -99 | Asked only if health_status = 2 |  |
| `sym_gasping` | Gasping for air | `binary` | 0, 1 | -99 | Asked only if health_status = 2 |  |
| `sym_runny_nose` | Runny nose | `binary` | 0, 1 | -99 | Asked only if health_status = 2 |  |
| `sym_sneezing` | Sneezing | `binary` | 0, 1 | -99 | Asked only if health_status = 2 |  |

### Symptoms - digestive

| Variable | Label | Type | Valid values | Missing | Skip logic | Notes |
|---|---|---|---|---|---|---|
| `sym_stomach_pain` | Stomach pain / cramps | `binary` | 0, 1 | -99 | Asked only if health_status = 2 |  |
| `sym_diarrhea` | Diarrhea | `binary` | 0, 1 | -99 | Asked only if health_status = 2 |  |
| `sym_vomiting` | Vomiting | `binary` | 0, 1 | -99 | Asked only if health_status = 2 |  |
| `sym_loss_appetite` | Loss of appetite | `binary` | 0, 1 | -99 | Asked only if health_status = 2 |  |
| `sym_nausea` | Nausea | `binary` | 0, 1 | -99 | Asked only if health_status = 2 |  |

### Symptoms - other

| Variable | Label | Type | Valid values | Missing | Skip logic | Notes |
|---|---|---|---|---|---|---|
| `sym_dizziness` | Dizziness | `binary` | 0, 1 | -99 | Asked only if health_status = 2 |  |
| `sym_headache` | Headache | `binary` | 0, 1 | -99 | Asked only if health_status = 2 |  |
| `sym_rash` | Rash | `binary` | 0, 1 | -99 | Asked only if health_status = 2 |  |
| `sym_pinkeye` | Pink eye / conjunctivitis | `binary` | 0, 1 | -99 | Asked only if health_status = 2 |  |
| `sym_ear_pain` | Ear pain | `binary` | 0, 1 | -99 | Asked only if health_status = 2 |  |
| `sym_other` | Other symptom (free text in sym_other_text) | `binary` | 0, 1 | -99 | Asked only if health_status = 2 |  |
| `sym_other_text` | Free-text description of other symptom | `string` | Up to 280 chars | — | Required only if sym_other = 1 |  |

### Symptoms - extended

| Variable | Label | Type | Valid values | Missing | Skip logic | Notes |
|---|---|---|---|---|---|---|
| `sym_bleeding_orifices` | Bleeding from body openings | `binary` | 0, 1 | -99 | Asked only if health_status = 2 | Used in arboviral / hemorrhagic fever surveillance modules. |
| `sym_red_eyes` | Red eyes (non-conjunctivitis) | `binary` | 0, 1 | -99 | Asked only if health_status = 2 |  |
| `sym_bloody_urine` | Discolored or bloody urine | `binary` | 0, 1 | -99 | Asked only if health_status = 2 |  |
| `sym_jaundice` | Yellow skin / yellow eyes | `binary` | 0, 1 | -99 | Asked only if health_status = 2 |  |

### Severity

| Variable | Label | Type | Valid values | Missing | Skip logic | Notes |
|---|---|---|---|---|---|---|
| `absent_work` | Absent from work this week | `binary` | 0, 1 | -99 | Asked only if health_status = 2 |  |
| `absent_school` | Absent from school this week | `binary` | 0, 1 | -99 | Asked only if health_status = 2 |  |
| `sought_healthcare` | Sought any healthcare or treatment | `binary` | 0, 1 | -99 | Asked only if health_status = 2 | Summary indicator derived from care_* variables. |

### Exposure

| Variable | Label | Type | Valid values | Missing | Skip logic | Notes |
|---|---|---|---|---|---|---|
| `exp_mass_gathering` | Attended a recent mass gathering | `binary` | 0, 1 | -99 | — | Concert, sports event, conference, religious service, etc. |
| `exp_tick_insect_bite` | Tick or insect bite in last 14 days | `binary` | 0, 1 | -99 | — |  |
| `exp_animal_bite` | Animal bite in last 14 days | `binary` | 0, 1 | -99 | — |  |
| `exp_travel_history` | Travel outside home region in last 14 days | `binary` | 0, 1 | -99 | — |  |
| `exp_live_animal_contact` | Contact with live animals | `binary` | 0, 1 | -99 | — | Includes farm, market, or wildlife exposure. |
| `exp_dead_sick_animal_contact` | Contact with dead or sick animals | `binary` | 0, 1 | -99 | — |  |
| `exp_sick_person_contact` | Contact with sick individual / confirmed case | `binary` | 0, 1 | -99 | — |  |

### Auxiliary

| Variable | Label | Type | Valid values | Missing | Skip logic | Notes |
|---|---|---|---|---|---|---|
| `digital_biomarker_signal` | Wearable-device biomarker flag (e.g., elevated resting HR) | `categorical` | 0, 1, 2 | -99 | Optional | 0=normal, 1=mild deviation, 2=marked deviation. Pulled from connected wearable when consented. |
| `photo_attached` | Photo attached to report (e.g., rash, test result) | `binary` | 0, 1 | -99 | Optional |  |
| `diagnostic_lab_confirmation` | Laboratory-confirmed diagnosis | `categorical` | 0, 1, 2, 3, 4, 5, 6 | -99 | Optional | 0=none, 1=influenza A, 2=influenza B, 3=SARS-CoV-2, 4=RSV, 5=other, 6=pending. |

### Demographics

| Variable | Label | Type | Valid values | Missing | Skip logic | Notes |
|---|---|---|---|---|---|---|
| `birth_month` | Month of birth | `integer` | 1-12 | -99 | — | Asked at registration. |
| `birth_year` | Year of birth | `integer` | 1900-2026 | -99 | — | Asked at registration. |
| `gender` | Gender identity | `categorical` | 1, 2, 3, 4, 5 | -99 | — |  |
| `race` | Race | `categorical` | 1, 2, 3, 4, 5, 6, 7 | -99 | — | Multiple selection allowed; stored as comma-separated codes. |
| `hispanic_latino` | Identifies as Hispanic or Latino/a | `categorical` | 1, 2, 3 | -99 | — |  |
| `zip_code` | Residential zip code | `string` | 5-digit US zip | — | — | Stored separately from main reports for privacy. |
| `occupation` | Occupation category | `categorical` | 1-12 | -99 | Optional |  |

### Vaccination

| Variable | Label | Type | Valid values | Missing | Skip logic | Notes |
|---|---|---|---|---|---|---|
| `flu_vaccine_2025` | Received flu vaccine on or after July 1, 2025 | `categorical` | 1, 2, 3 | -99 | — |  |
| `covid_vaccine_2025` | Received COVID-19 vaccine on or after September 1, 2025 | `categorical` | 1, 2, 3 | -99 | — |  |

### Environmental

| Variable | Label | Type | Valid values | Missing | Skip logic | Notes |
|---|---|---|---|---|---|---|
| `env_incident_date` | Date of environmental incident | `date` | YYYY-MM-DD | — | Optional module |  |
| `env_vector_location` | Location of vector spotting (lat,lon) | `string` | Decimal degrees, comma-separated | — | Optional module |  |
| `env_vector_unusual` | Unusual presence of vectors | `binary` | 0, 1 | -99 | Optional module | Vectors out of season or out of range. |
| `env_vector_density` | Estimated vector density | `categorical` | 1, 2, 3, 4 | -99 | Optional module | 1=low, 2=moderate, 3=high, 4=very high. |
| `env_flooding` | Recent flooding observed at residence | `binary` | 0, 1 | -99 | Optional module |  |
| `env_water_contamination` | Suspected water contamination | `binary` | 0, 1 | -99 | Optional module |  |

### Livestock

| Variable | Label | Type | Valid values | Missing | Skip logic | Notes |
|---|---|---|---|---|---|---|
| `ls_incident_date` | Date of livestock incident | `date` | YYYY-MM-DD | — | Optional module |  |
| `ls_incident_location` | Location of livestock incident | `string` | Free text or lat,lon | — | Optional module |  |
| `ls_sick_count` | Number of sick livestock animals | `integer` | 0-99999 | -99 | Optional module |  |
| `ls_dead_count` | Number of dead livestock animals | `integer` | 0-99999 | -99 | Optional module |  |
| `ls_species` | Livestock species affected | `string` | Free text | — | Optional module | Cattle, swine, poultry, sheep, goats, etc. |

### Wildlife

| Variable | Label | Type | Valid values | Missing | Skip logic | Notes |
|---|---|---|---|---|---|---|
| `wl_incident_date` | Date of wildlife incident | `date` | YYYY-MM-DD | — | Optional module |  |
| `wl_incident_location` | Location of wildlife incident | `string` | Free text or lat,lon | — | Optional module |  |
| `wl_species` | Wildlife species affected | `string` | Free text | — | Optional module |  |
| `wl_dead_count` | Number of dead wildlife animals | `integer` | 0-99999 | -99 | Optional module |  |

## Value labels reference

Codes for categorical variables. Binary variables (the symptoms, care-seeking fields, exposure flags, and a handful of severity and auxiliary indicators) all share the scheme `0` = No / not selected, `1` = Yes / selected, `-99` = missing, and are not repeated below.

**`health_status`** — How are you feeling this week?

| Code | Label |
|---|---|
| `1` | Healthy, thanks! |
| `2` | Not feeling well |
| `-99` | Missing / not answered |

**`at_home_test_covid`** — Took an at-home COVID-19 test in last 7 days

| Code | Label |
|---|---|
| `0` | No |
| `1` | Yes |

**`at_home_test_flu`** — Took an at-home influenza test in last 7 days

| Code | Label |
|---|---|
| `0` | No |
| `1` | Yes |

**`at_home_test_none`** — Did not take any at-home test

| Code | Label |
|---|---|
| `0` | No |
| `1` | Yes |

**`digital_biomarker_signal`** — Wearable-device biomarker flag (e.g., elevated resting HR)

| Code | Label |
|---|---|
| `0` | Normal |
| `1` | Mild deviation from baseline |
| `2` | Marked deviation from baseline |
| `-99` | Missing / no wearable connected |

**`diagnostic_lab_confirmation`** — Laboratory-confirmed diagnosis

| Code | Label |
|---|---|
| `0` |  |
| `1` | Influenza A |
| `2` | Influenza B |
| `3` | SARS-CoV-2 |
| `4` | RSV |
| `5` | Other pathogen |
| `6` | Pending |
| `-99` | Missing / not answered |

**`gender`** — Gender identity

| Code | Label |
|---|---|
| `1` | Female |
| `2` | Male |
| `3` | Non-binary / third gender |
| `4` | Other |
| `5` | Prefer not to say |
| `-99` | Missing / not answered |

**`race`** — Race

| Code | Label |
|---|---|
| `1` | American Indian or Alaska Native |
| `2` | Asian |
| `3` | Black or African American |
| `4` | Native Hawaiian or Other Pacific Islander |
| `5` | White |
| `6` | Other |
| `7` | Prefer not to say |
| `-99` | Missing / not answered |

**`hispanic_latino`** — Identifies as Hispanic or Latino/a

| Code | Label |
|---|---|
| `1` | Yes |
| `2` | No |
| `3` | Prefer not to say |
| `-99` | Missing / not answered |

**`occupation`** — Occupation category

| Code | Label |
|---|---|
| `1` | Healthcare worker |
| `2` | Education (teacher / staff / student) |
| `3` | Office / professional |
| `4` | Retail / service / hospitality |
| `5` | Agriculture / livestock / veterinary |
| `6` | Transportation / logistics |
| `7` | Manufacturing / construction |
| `8` | Public safety (police / fire / military) |
| `9` | Homemaker / caregiver |
| `10` | Retired |
| `11` | Unemployed |
| `12` | Other |
| `-99` | Missing / not answered |

**`flu_vaccine_2025`** — Received flu vaccine on or after July 1, 2025

| Code | Label |
|---|---|
| `1` | Yes |
| `2` | No, but I plan to |
| `3` | No, I'm not planning to |
| `-99` | Missing / not answered |

**`covid_vaccine_2025`** — Received COVID-19 vaccine on or after September 1, 2025

| Code | Label |
|---|---|
| `1` | Yes |
| `2` | No, but I plan to |
| `3` | No, I'm not planning to |
| `-99` | Missing / not answered |

**`env_vector_density`** — Estimated vector density

| Code | Label |
|---|---|
| `1` | Low |
| `2` | Moderate |
| `3` | High |
| `4` | Very high |
| `-99` | Missing / not answered |

## Synthetic dataset

The companion file `synthetic_data.json` contains 35 fabricated weekly reports. The data is shaped to be realistic, not random:

- A panel of 8 recurring participants reports across six consecutive weeks (roughly 80% weekly response rate).
- About 71% of reports are healthy and 29% are sick, in line with typical out-of-season participatory surveillance baselines.
- Sick reports cluster into plausible syndromes (influenza-like illness, COVID-like, common cold, gastrointestinal, allergy) rather than random symptom checkboxes.
- Care seeking and severity markers (`absent_work`, `absent_school`, `sought_healthcare`) are correlated with symptom load.
- One environmental incident and one wildlife incident are populated to demonstrate the One Health blocks. Livestock incidents are conditional on `occupation = 5` (Agriculture / livestock / veterinary).

The data is intended for testing pipelines, dashboards, and analytic code. It does not represent any real population.
