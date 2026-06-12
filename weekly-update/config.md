# Skill Configuration

## Team Selection

The user selects their team at runtime. Available teams and their associated Slack handles:

| Team | Linear team name | PM Slack handle | PM Slack user ID |
|---|---|---|---|
| CF - Ocean | Ocean | @Job van Hardeveld | UCG4CU9T3 |
| CF - Forest | Forest | @Lisa Vuong | _(unknown)_ |
| CF - Mangrove | Mangrove | @Valeria | U055LV70L5C |
| CF - Android & iOS | Android & iOS | @Jillian Graham | _(unknown)_ |

The handle can be overridden by the user if they prefer an alternative.

## Team-Specific Configuration

### CF - Ocean
- **PM:** Job van Hardeveld — Slack: `UCG4CU9T3`, handle: @Job
- **Designer:** Santiago Echeverry Gonzalez — Slack: `U04RVTT2KM2`, handle: @Santi
- **Engineering Manager:** Salah Eddine Cherkaoui — Slack: `U03TNC79URY`, handle: @Salah
- **Team Slack channel:** `#cf-ocean-web` (ID: `C0512DTN0LC`)
- **Update header:** `Team Ocean :whale:` as the title
- **Additional Slack sources:** Read `#cf-ocean-web` for the current week to surface team signals (blockers, design updates, EM notes, ad-hoc tickets)

### CF - Mangrove
- **PM:** Valeria Castelli — Slack: `U055LV70L5C`, handle: @Valeria
- **Designer:** Tahnee Czerny — Slack: `U05Q5GNNFD5`, handle: @Tahnee
- **Engineering Manager:** Olga Narochnaya — Slack: `U05AFHG162F`, handle: @Olga
- **Team Slack channel:** `#cf-mangrove-web` (ID: `C055SRHRV4M`)
- **Update header:** `Team Mangrove :mangrove:` as the title
- **Additional Slack sources:** Read `#cf-mangrove-web` for the current week to surface team signals (blockers, design updates, EM notes, ad-hoc tickets)

## Linear Filter

- All issues and projects for the selected team (any assignee)
- Active, in-progress, or updated within the current week (Mon–today)

## Slack Filter

- PM user ID is team-specific (see Team-Specific Configuration above)
- Filter `product-updates` and `ab-testing-at-rebuy` by the PM's Slack user ID
- Channels:
  - `product-updates` — released features (this week, filtered by PM user ID)
  - `ab-testing-at-rebuy` — AB-test results (this week, filtered by PM user ID)
  - `product-standup` — previous week's standup update (last Mon–Fri, filtered by PM user ID) — used for plan-vs-actual comparison
  - Team-specific channel (e.g. `#cf-mangrove-web`) — read for the current week to enrich with team context
