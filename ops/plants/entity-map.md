# Plant registry

Last verified: 2026-09-09. Count computed live from `input_datetime.watered_*` — **19**.

`slug` is the suffix of `input_datetime.watered_<slug>` **and** the key in the
backup automation's `limits` / `names` dicts. They must match.

## Indoor

| Plant | slug | view path | day limit | moisture sensor |
|---|---|---|---|---|
| הויה קרנוזה | `hoya` | `hoya` | 16 | `sensor.smart_soil_tester_humidity` (thr 88) |
| עץ הזית | `olive` | `olive` | 11 | `sensor.temperature_humidity_sensor_humidity` (thr 75) |
| צמח הפיל | `elephant_bush` | `elephant` | 25 | `sensor.temperature_humidity_sensor_2_humidity` (thr 15) |
| פילודנדרון דו-נוצי | `ellies_plant` | `ellies` | 11 | `sensor.ellies_plant_moisture` (thr 60) |
| פותוס מטבח | `kitchen_pothos` | `pothos_kitchen` | 11 | `sensor.temperature_humidity_sensor_3_humidity` (thr 70) |
| פותוס חדר שינה | `bedroom_pothos` | `pothos_bedroom` | 11 | `sensor.temperature_humidity_sensor_4_humidity` (thr 45) |
| ספטיפיליום | `peace_lily` | `peace-lily` | 8 | `sensor.sptypylyvm_humidity` (thr 55) |
| סחלב | `orchid` | `orchid` | 10 | none — bark, not soil |
| זמיה קוקוס | `zz_plant` | `zz` | 30 | none |
| דקל כמדוריאה | `parlour_palm` | `parlour-palm` | 9 | none |
| קלתאה אורנטה | `calathea_ornata` | `calathea-ornata` | 6 | none |
| קלתאה רטלסנייק | `calathea_rattlesnake` | `calathea-rattlesnake` | 6 | none |
| צמח עכביש | `spider_plant` | `spider` | 8 | `sensor.smart_soil_tester_2_humidity` (thr 85) |

## Balcony

| Plant | slug | view path | day limit |
|---|---|---|---|
| גינת סוקולנטים שקופה | `succulents_clear` | `succulents-clear` | 20 |
| גינת סוקולנטים שחורה | `succulents_black` | `succulents-black` | 21 |
| קיר ירוק | — | `green-wall` | irrigated |
| עמוד ירוק | — | `green-column` | irrigated |

## Not placed yet (bought 2026-09-09)

| Plant | slug | view path | day limit | intended |
|---|---|---|---|---|
| אכוורייה | `echeveria` | `echeveria` | 21 | a succulent garden |
| פותוס מרבל קוון | `marble_queen_pothos` | `marble-queen` | 11 | bar rack, glass end |
| פילודנדרון בירקין | `philodendron_birkin` | `birkin` | 10 | bedroom library, window side |
| פיקוס ננסי מגוון | `creeping_fig` | `creeping-fig` | 5 | green column, bottom trough, shaded face |

---

## Automations

| Entity | Trigger | Sensor dependent |
|---|---|---|
| `automation.tsmkhym_gybvy_lpy_ymym_ll_tlvt_bkhyyshn` | 09:30 daily | No — the real safety net |
| `automation.plants_daily_thirsty_roundup` | 09:00 daily | Yes |
| `automation.tsmkhym_pvsh_mydy_kshtsmkh_khvtsh_t_hsp` | threshold crossing | Yes |

`script.plant_mark_watered` takes `target_datetime` and is generic — never write a
per-plant variant.

## Immich

- Web UI: `http://<immich-lan-host>:2283/albums/<album_id>` — LAN only
- Deep link: `immich://album?id=<album_id>`
- Media source prefix in dashboards: `media-source://immich/<source-id>|albums|…`

Every plant has its own album; every asset is also in `צמחיה`.

---

## Sensor reliability — read before trusting a number

The Tuya soil probes are not trustworthy in absolute terms.

- Two read 41% and 64% **in open air**.
- `smart_soil_tester_2` showed 86% while its pot was bone dry (6 Aug), and 24% in
  air / 92% in fresh soil (9 Sep).
- Tuya entities are written **only when the value changes**. A flat reading is
  indistinguishable from a dead channel via `last_reported`. Cross-check against
  the temperature channel of the same device, which moves constantly.

**Trends are informative. Absolute values are not.** Treat the day-based backup
automation as the source of truth and the probes as a hint.
