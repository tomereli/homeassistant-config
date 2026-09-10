# ha-ops

Operating documentation for this Home Assistant instance.

**Documentation only.** No exported YAML, no secrets, no tokens. The live
configuration lives in Home Assistant; this directory holds the knowledge needed
to change it without breaking something three screens away.

> **This repository is public.** Nothing here should contain a credential, a
> token, an external hostname or an account identifier. Entity ids and LAN
> addresses appear because the committed Lovelace storage already exposes them —
> that is not a licence to add more.

## Why this exists

Every session starts from zero. It reads the thing it was asked about, changes
it, and reports success — then the next session discovers that a counter three
views away still says the old number, or that a helper for a dead plant is still
sitting in the registry.

That is not a knowledge problem. It is a **discovery** problem: the same lookups
get re-derived every time, badly, and the parts that are easy to miss get missed
the same way every time.

So this is not a description of the system. It is the set of checks that have
already failed once.

## Layout

| Path | What it holds |
|---|---|
| `conventions/write-protocol.md` | How to write to HA at all — gates, hashes, hard rules |
| `conventions/exit-criteria.md` | What "done" means. Run these before reporting success |
| `runbooks/add-a-plant.md` | Every touchpoint when a plant enters the system |
| `runbooks/rename-an-entity.md` | The layers a name lives in, and what a rename does not update |
| `plants/entity-map.md` | Plant registry — slug, helper, sensor, album, day limit |

## Reading order for a new session

1. `conventions/write-protocol.md` — you cannot write without it
2. The runbook for what you are about to do
3. `conventions/exit-criteria.md` — before you report anything as finished

## Maintaining this

A runbook step earns its place by having been missed. When something is missed,
add the check that would have caught it — do not add a general reminder to be
careful.
