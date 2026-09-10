# Runbook — add a plant

Nine touchpoints. Missing any one leaves a plant that exists but is invisible, or
a counter that lies.

Read `../conventions/write-protocol.md` first. Fetch a fresh `BestPracticeKey`.

---

## 1. Immich album

```
immich_create_album(name="<Hebrew name>", description="<latin name> — bought <date>. <placement>")
immich_add_to_album(album_id=..., asset_ids=[...])
```

Keep the returned `album_id`. It is needed in three later steps.

Assets stay in `צמחיה` as well — album membership is not exclusive.

## 2. Watering helper

```
ha_config_set_helper(helper_type="input_datetime", name="Watered <Name>",
                     has_date=True, has_time=True, icon="mdi:watering-can")
```

The entity id becomes `input_datetime.watered_<slug>`. **This is the source of
truth for the plant count** — the computed chips select on `watered_`. A plant
without this helper does not exist as far as the counters are concerned.

New helpers default to **today's date**, not empty. That reads as "watered today".
Say so rather than letting it pass as a real log.

## 3. Day-based backup automation

`automation.tsmkhym_gybvy_lpy_ymym_ll_tlvt_bkhyyshn` — runs 09:30 daily, sensor
independent. This is the real safety net.

```python
config['variables']['limits']['<slug>'] = <days>
config['variables']['names']['<slug>'] = '<Hebrew name>'
```

The `<slug>` must match the `input_datetime.watered_<slug>` suffix exactly.

Skip only if the plant sits in the irrigated wall or column — day counting on
irrigated media is noise. Record *why* in the automation description.

## 4. Plant view on `plants-hub`

Append to `config['views']`. Copy the shape from an existing recent view
(`view_path="spider"` is the cleanest template):

- `custom:mushroom-title-card` — bilingual title, latin name + location subtitle
- `custom:mushroom-chips-card` — EN/עב toggle, All plants, Album, category
- `picture` — `media-source://immich/<source>|albums|<album>|<asset>|<file>|image/heic`
- `custom:mushroom-template-card` — watering tile calling `script.plant_mark_watered`
- `custom:mushroom-title-card` — Care / טיפול
- `markdown` — bilingual care text
- **`custom:navbar-card` — must be last**

Set `subview: true`.

## 5. `Indoor` view (`views[1]`) — or `Balcony` (`views[0]`)

Add a `vertical-stack` of `picture` + `mushroom-template-card` into the section
for its room. `insert()` a new section **before** the last one; the last section
holds the navbar.

## 6. `All plants` view (`views[16]`)

Add a `mushroom-template-card` to the right section grid.

## 7. `Watering queue` view (`views[20]`)

**Two** edits in this view, and the second is easy to miss:

- `cards[2]['content']` — the ranking table's `P` tuple list. It appears **twice**
  (English and Hebrew branch); `.replace()` catches both.
  Tuple shape: `('<slug>','<Hebrew name>','<sensor or empty>',<threshold>,<days>,'<view path>')`
- `cards[4]['cards']` — append a quick-log tile.

## 8. Counts

The count chips in `views[0]` and `views[1]` are computed from
`input_datetime.watered_*` and need no edit — **as long as step 2 was done**.

The "needs water" headline in both views uses a hardcoded `expand(...)` list of
binary sensors. If the new plant has a **moisture sensor**, add it there. If it
has no sensor, nothing to do.

## 9. Exit criteria

Run `../conventions/exit-criteria.md`. In particular:

```
ha_eval_template("{% set n = states.input_datetime | map(attribute='entity_id')
                    | select('search','watered_') | list | count %}{{ n }}")
```

and confirm the number matches the plants that actually exist.

---

## Placement

Do not assign an area the plant is not in. Use "טרם מוקמו / Not placed yet" as a
section and in the view text until it is physically placed. A plausible-sounding
wrong area outlives the person who wrote it.
