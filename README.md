# Home Assistant Config

My Home Assistant configuration, automatically backed up with AI-generated commit messages.

## What is tracked

- `configuration.yaml` - Main config
- `automations.yaml` - Automations
- `scripts.yaml` - Scripts  
- `scenes.yaml` - Scenes
- `blueprints/` - Automation blueprints
- `.storage/core.device_registry` - All devices
- `.storage/core.entity_registry` - All entities
- `.storage/core.area_registry` - Rooms/areas
- `.storage/core.config_entries` - Integrations
- `.storage/lovelace_dashboards` - Dashboards

## What is NOT tracked (security)

- `secrets.yaml` - Passwords/API keys
- `.cloud/` - Nabu Casa credentials
- `.ssh/` - SSH keys
- `*.db` - Databases
- `.storage/*` - Other runtime state

## Automated Backups

A Home Assistant automation runs nightly:
1. Stages all changes
2. Gets diff and sends to Gemini AI
3. AI generates descriptive commit message
4. Commits and pushes to this repo
