# GTMS-9

## Template Change Notes

### Problem that solved by this standard

There is no standard for how change notes are added to a `template.tpl` file. Because of this, the `___NOTES___` section is often left out of date, entries lack context on the functional impact of a change, or the `Created on` line gets moved/duplicated.

`template.tpl` and `metadata.yaml` are never committed together: `template.tpl` is modified and committed first, and only afterwards is `metadata.yaml` updated (in a separate commit) with the commit hash pointing to that `template.tpl` change. This standard requires that the `___NOTES___` update be part of that same `template.tpl` commit, so the commit hash later referenced in `metadata.yaml` always already reflects the corresponding change notes entry.

### Standard description

- Every time `template.tpl` is modified in a commit that will later be referenced by a new commit hash in `metadata.yaml`, that same commit must also add a change notes entry for the change(s) to the `___NOTES___` section of `template.tpl`.
- The entry must be inserted directly below the `___NOTES___` line, above any existing entries. The `Created on` line always stays at the bottom, never moved or duplicated.
- Each entry must follow the format:
  ```
  YYYY-MM-DD - Change Notes:
    - Change description
  ```
  - Date in `YYYY-MM-DD` format.
  - Bullets use a two-space indent.
  - At most 5 bullets per entry, one per logical change. Related changes can be grouped into a single bullet to stay within the limit.
  - Each bullet must describe **what changed and why** (functional impact and/or implementation decision) — not only how (e.g. avoid bullets that only restate "updated variable X").
  - Version bumps always get their own bullet.
  - Entries are separated by one blank line.
- To add the entry, use the [`gtm-template-add-changesnotes`](skills/gtm-template-add-changesnotes/SKILL.md) LLM AI agent skill, which reviews the `git diff` of `template.tpl` and inserts the entry following this standard. Using the skill is optional — the entry can be added manually, but it must still follow the rules above.

### Example

Before:
```
___NOTES___

2026-04-09 - Change Notes:
  - Add consent-aware event enhancement: user data is now only read from/written to localStorage when consent is granted
  - Bump version to stape-gtm-1.2.0

Created on 08/15/2025, 08:58:45 AM
```

After (new entry added on 2026-10-30):
```
___NOTES___

2026-10-30 - Change Notes:
  - Add optional Meta Parameter Builder SDK integration (enabled by default) to improve _fbp and _fbc cookie coverage, including backup Click ID retrieval from in-app browsers
  - Bump version to stape-gtm-1.3.0

2026-04-09 - Change Notes:
  - Add consent-aware event enhancement: user data is now only read from/written to localStorage when consent is granted
  - Bump version to stape-gtm-1.2.0

Created on 08/15/2025, 08:58:45 AM
```

### Common Mistakes

| Mistake | Fix |
|---|---|
| Fabricating entries not present in the diff | Only write what the diff shows |
| Moving or duplicating the `Created on` line | Leave it exactly as-is, at the bottom |
| Inserting the new entry below existing entries | New entry always goes at the top |
| Bullets describing only how, not what/why | Each bullet must convey the change's purpose or effect |
| Missing version bump bullet | Always include version bumps when present in the diff |
