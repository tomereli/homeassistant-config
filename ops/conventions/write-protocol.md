# Write protocol

Applies to every write to Home Assistant through `ha-mcp`.

## The gate

Write tools require `BestPracticeKey`. Get it from:

```
ha_get_skill_guide(skill="home-assistant-best-practices", file="SKILL.md")
```

The key is published at the top of the returned content and **rotates hourly**.
A key from earlier in a long session will be rejected — fetch a fresh one rather
than retrying the old one.

Always pass `MandatoryBPS=false` alongside it. That suppresses re-sending the
reference files inline; it does not skip the check.

## Optimistic locking

Every config write takes `config_hash` from the matching read:

| Write | Hash comes from |
|---|---|
| `ha_config_set_automation` | `ha_config_get_automation` |
| `ha_config_set_dashboard` | `ha_config_get_dashboard` |
| `ha_config_set_script` | `ha_config_get_script` |

Each successful write returns a **new** hash. Chained edits use the hash from the
previous write's response, not a re-read.

`config_hash` on a dashboard covers the **whole** dashboard even when the read was
scoped with `view_path`. A scoped read is a payload optimisation, not a narrower
lock.

## python_transform

Prefer it over full-config replacement for anything that already exists. Send the
change, not the document.

**Forbidden:** `def`, `.find()`, `import`, `while`, `try/except`, dunder access,
`eval` / `exec` / `getattr` / `setattr`.

**Available and useful:** `.replace()`, `.append()`, `.insert()`, `.pop()`,
`.remove()`, `.get()`, `in`, `str()`, `for`, comprehensions, ternaries, and `+`
concatenation.

`.replace()` replaces **all** occurrences. That is usually what you want on
bilingual templates, where the same list appears once per language branch — but
check that it is, because it will also hit branches you did not read.

### Reading efficiently

Reading a whole dashboard is expensive. Cheaper paths, in order:

1. `ha_config_get_dashboard(mode="search", query="...")` — cross-dashboard, tells
   you which views and card paths reference something. Cheapest.
2. `ha_config_get_dashboard(url_path=..., card_type=...)` — python_paths for a card type.
3. `ha_config_get_dashboard(url_path=..., view_path=...)` — one view.
4. Full config — last resort.

Searching for a **navbar-card with `include_config=true`** recovers every view path
on a dashboard in one call.

## Hard rules

**The last card in every view must remain `custom:navbar-card`.**
When inserting a section, insert *before* the section that holds the navbar — never
append to the end. Verify the navbar is still last after the write.

**Prefer native constructs over Jinja in logic positions.**
`condition: numeric_state` over `{{ states(x) | float > n }}`. `condition: state`
over `is_state()`. Native `for:` over `now() - last_changed` arithmetic. Templates
belong in `data.*`, notification bodies, `event_data` and `variables` — not in
`condition:` or `trigger:`.

**Never rename an entity without checking its consumers first.** See
`../runbooks/rename-an-entity.md`.

## Verification

`post_write_verified: true` means the write persisted. **It does not mean the
result is correct.** Any template written must be executed:

```
ha_eval_template(template="...")
```

and the actual output reported. A template that renders nothing renders silently.
