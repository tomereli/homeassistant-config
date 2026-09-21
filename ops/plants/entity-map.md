# Plant registry

Last verified: 2026-09-21. Count computed live from `input_datetime.watered_*` — **19**.

`slug` is the suffix of `input_datetime.watered_<slug>` **and** the key in the
day-count automation's `limits` / `names` dicts. They must match.

Places below follow the room sections on `plants-hub`. They are not verified by
sight beyond what is noted.

## Indoor — hand watered, day count applies

| Plant | slug | view | days | place | moisture sensor |
|---|---|---|---|---|---|
| הויה קרנוזה | `hoya` | `hoya` | 16 | living room | `sensor.smart_soil_tester_humidity` (thr 88) |
| עץ הזית | `olive` | `olive` | 11 | living room | `sensor.temperature_humidity_sensor_humidity` (thr 75) |
| פילודנדרון דו-נוצתי | `ellies_plant` | `ellies` | 11 | living room | `sensor.ellies_plant_moisture` (thr 60) |
| סחלב | `orchid` | `orchid` | 10 | living room, by the west glass | none — bark |
| פילודנדרון בירקין | `philodendron_birkin` | `birkin` | 10 | living room library | none |
| פותוס מטבח | `kitchen_pothos` | `pothos_kitchen` | 11 | kitchen, bar rack, inner end (kokedama) | `sensor.temperature_humidity_sensor_3_humidity` (thr 70) |
| פותוס מרבל קווין | `marble_queen_pothos` | `marble-queen` | 11 | kitchen, bar rack, glass end | none |
| אכוורייה | `echeveria` | `echeveria` | 21 | kitchen counter | none |
| צמח הפיל | `elephant_bush` | `elephant` | 25 | study | `sensor.temperature_humidity_sensor_2_humidity` (thr 15) |
| פותוס חדר שינה | `bedroom_pothos` | `pothos_bedroom` | 11 | master bedroom | `sensor.temperature_humidity_sensor_4_humidity` (thr 45) — `unknown` since ~11 Sep |
| ספטיפיליום | `peace_lily` | `peace-lily` | 8 | master bedroom | `sensor.sptypylyvm_humidity` (thr 55) |
| דקל כמדוריאה | `parlour_palm` | `parlour-palm` | 9 | master bedroom library | none |
| זמיה קוקוס | `zz_plant` | `zz` | 30 | bathrooms | none |
| קלתאה אורנטה | `calathea_ornata` | `calathea-ornata` | 6 | bathrooms | none |
| קלתאה רטלסנייק | `calathea_rattlesnake` | `calathea-rattlesnake` | 6 | bathrooms | none |

## Balcony

| Plant | slug | view | days | notes |
|---|---|---|---|---|
| גינת סוקולנטים שקופה | `succulents_clear` | `succulents-clear` | 20 | against the glass, dries faster than indoors |
| גינת סוקולנטים שחורה | `succulents_black` | `succulents-black` | 21 | |
| קיר ירוק | — | `green-wall` | irrigated | |
| עמוד ירוק | — | `green-column` | irrigated | |
| צמח עכביש | `spider_plant` | `spider` | **irrigated — no day count** | column, 2nd trough from bottom, house-facing side. Sensor `sensor.smart_soil_tester_2_humidity` |
| פיקוס ננסי מגוון | `creeping_fig` | `creeping-fig` | **irrigated — no day count** | column, bottom trough, left, house-facing side |

Spider plant and creeping fig keep their `watered_*` helpers on purpose: the
count chips select on them, and a manual watering can still be logged. They are
**not** in the day-count automation and **not** in the Watering-queue ranking.

**A plant moving into the wall or column** means: remove it from the day-count
automation, remove it from the queue `P` list and quick-log tiles, move its cards
from Indoor to Balcony, and rewrite its watering tile as irrigated. Leave the
helper.

---

## Automations

| Entity | Trigger | Sensor dependent |
|---|---|---|
| `automation.tsmkhym_gybvy_lpy_ymym_ll_tlvt_bkhyyshn` | 09:30 daily | No — the real safety net. Hand-watered plants only |
| `automation.plants_daily_thirsty_roundup` | 09:00 daily | Yes |
| `automation.tsmkhym_pvsh_mydy_kshtsmkh_khvtsh_t_hsp` | threshold crossing | Yes |

`script.plant_mark_watered` takes `target_datetime` and is generic — never write a
per-plant variant. To backdate a watering, call `input_datetime.set_datetime`
directly; the script logs "now".

## Irrigation (wall + column)

- SONOFF SWV valve on the balcony, timed mode: 240 s, 1 cycle, daily at 08:00,
  about 9 L per run.
- Water leaves through **holes punched in the tubing, not drippers.** No pressure
  compensation: holes that widen take more flow, holes that scale up take less,
  and the imbalance compounds. Shortening the run lowers volume but not the
  imbalance.
- Extra pulse slots exist at 13:00 and 18:30 (`input_boolean.irrigation_pulse_2_enabled`,
  `..._pulse_3_enabled`), both **off**.
- The valve reports its own timestamps with the year 1996. Ignore them.

## Immich

- Web UI: `http://<immich-lan-host>:2283/albums/<album_id>` — LAN only
- Deep link: `immich://album?id=<album_id>`
- Media source prefix in dashboards: `media-source://immich/<source-id>|albums|…`

Every plant has its own album; every asset is also in `צמחיה`.

---

## Sensor reliability — read before trusting a number

The Tuya soil probes are not trustworthy in absolute terms.

- Two read 41% and 64% **in open air**.
- `smart_soil_tester_2` showed 86% while its pot was bone dry (6 Aug), then 24% in
  air and 92% in fresh soil (9 Sep), and sits at 100% in the column trough.
- Tuya entities are written **only when the value changes**. A flat humidity value
  is indistinguishable from a dead channel via `last_reported`. **Check the
  temperature channel of the same device**: it moves constantly, so if it is also
  frozen, data is genuinely not arriving.

**Trends are informative. Absolute values are not.** The day count is the source
of truth for hand-watered plants; the probes are a hint.

## Known naming gaps (as of 2026-09-21)

Devices were renamed by position in the Tuya app; the names did **not** reach
Home Assistant.

- `sensor.smart_soil_tester_2_humidity` is still titled "צמח שפתון Humidity"
  (a dead plant) — it is the spider plant's probe.
- The wall probes are still `קיר ירוק עליון` / `תחתון` in HA; in Tuya they
  carry position names.
- A probe named in Tuya "עמוד ירוק פיקוס שמאל תחתון" has **no HA entity
  found**. Which HA entity maps to which Tuya name is unconfirmed — do not guess.

When the wall probes were renamed in Tuya on 12 Sep ~23:28, both stopped
reporting to HA (humidity *and* temperature); they recovered on their own by
20 Sep.
