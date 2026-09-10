# Runbook — rename an entity

A name lives in more layers than the registry. Renaming one layer looks like it
worked and breaks the others silently.

---

## The layers

| Layer | Renamed by | Propagates? |
|---|---|---|
| Device name at source (Tuya, Zigbee) | The vendor app | Friendly name only |
| Entity friendly name | `ha_set_entity(name=...)` | — |
| Entity id | `ha_set_entity(new_entity_id=...)` | **No** — breaks consumers |
| Config Entry title | Not renameable via options flow | — |
| Config Entry `options` (source entity ids) | Not touched by a registry rename | **No** |
| References in automations, scripts, dashboards | Nothing | **No** |

**Entity ids are frozen at creation.** Renaming the device in Tuya updates the
friendly name and leaves the slug behind. That is why
`binary_sensor.tsyts_khdr_bvdh_kitchen_pothos_needs_water` reads "study room" in
its id and "kitchen pothos" in its name. Cosmetic — but never read location from a
slug.

---

## Order of operations

**1. Find every consumer, first.**

```
ha_search(query="<exact entity_id>", search_types=["automation","script","scene","helper","dashboard"])
ha_config_get_dashboard(mode="search", query="<exact entity_id>")
```

Search the **exact entity id**, not a name fragment. `ha_search` with the exact id
reports automations and scenes that reference it even when their config body could
not be read.

**2. Rename.**

```
ha_set_entity(entity_id=..., new_entity_id=..., name=...)
```

**3. Update every consumer found in step 1**, in the same session. There is no
grace period — the automation is broken between step 2 and step 3.

**4. Search again** by display name and by any hardcoded count. See
`../conventions/exit-criteria.md` §1.

**5. Delete orphaned helpers** belonging to the old identity.

---

## Config Entry trap

Threshold, template, group, utility_meter and other flow-based helpers store the
**source entity id inside their config entry options**. A registry rename does not
update it. The helper keeps working against the old id until the old id stops
existing, and then goes `unknown` with no error.

To inspect:

```
ha_get_integration(entry_id=..., include_options=True)
```

Flow-based helpers **cannot be renamed** through their options flow. To change the
title, delete and recreate — and accept that its own entity id changes too.

---

## Do not guess physical mappings

Which key on which panel, which room a transliterated slug refers to, which pot a
probe is actually sitting in — none of these are derivable from the config.

Ask, or verify by measurement. A probe's identity can be confirmed by pulling it
and watching a **live** channel move — but check which channels are actually
reporting first, because a flat value and a dead channel are indistinguishable in
`last_reported` when the integration writes only on change.
