# Exit criteria

Run these before reporting a change as finished. Each one exists because it was
missed.

---

## 1. Search by text, not only by reference

Referential search finds what *links* to a thing. It does not find a hardcoded
string that describes it.

For any entity added, removed or renamed, search **all** dashboards for:

- the entity id
- the display name, in both languages
- any **count** the change affects (`"13 plants"`, `"15 צמחים"`)
- the category or section it belongs to

```
ha_config_get_dashboard(mode="search", query="<entity_id>")
ha_config_get_dashboard(mode="search", query="<display name>")
ha_config_get_dashboard(mode="search", query="plants{% else %}")
```

> **The miss:** four new plants were added and correctly linked from every view
> that *navigates* to a plant. Two chips that stated the plant count as literal
> text were not found, because nothing links from a number.

---

## 2. A hardcoded number is a bug, not a value

If a counter is a literal, do not update it. Replace it with something that counts.

```jinja
{% set n = states.input_datetime
     | map(attribute='entity_id') | select('search','watered_') | list | count %}
```

Updating the literal defers the bug to the next session. That is the loop this
directory exists to break.

Same rule for hardcoded **lists**: an `expand(...)` of specific entity ids will
silently omit anything added later. Prefer a label, an area, or a naming
convention the template can select on.

---

## 3. Execute, do not assume

Every template written gets run through `ha_eval_template` and the **actual
output** reported.

`post_write_verified: true` means it saved. It says nothing about whether it is
right.

---

## 4. Orphans

Removing an entity is two steps, not one:

- **Before:** search for references. Fix or remove them first.
- **After:** delete the entity's own helpers — `input_datetime.watered_*`,
  threshold config entries, template helpers.

> **The miss:** the lipstick plant was removed from both automations, but
> `input_datetime.watered_lipstick` stayed in the registry for half a day and
> would have made every computed plant count read one too high.

---

## 5. Report what was not checked

Explicitly. A silent gap reads as completeness.

If a view was not read, if a comparison was not done, if a figure came from one
data point — say so in the same message, not when asked.

---

## 6. Verify the premise before acting on it

If the task assumes a state of the world, confirm it before writing.

> **The miss, in the other direction:** a request to file photos into albums was
> already done by an earlier session. Checking took four calls. Redoing it blind
> would have taken forty and produced duplicates.

Equally: a conclusion drawn from an absence of data is not a conclusion. A Tuya
entity writes only when its **value changes**, so a flat reading and a dead
channel look identical from `last_reported` alone. Find a second signal before
declaring something broken.
