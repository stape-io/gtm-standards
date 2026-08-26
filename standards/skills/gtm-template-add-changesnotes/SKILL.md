---
name: gtm-template-add-changesnotes
description: Use when asked to add or update a changelog entry in a GTM template .tpl file's ___NOTES___ section after reviewing changes made to the template.
---

# GTM Template: Add Change Notes

## Overview

Review what changed in a `.tpl` file using `git diff`, then insert a new dated changelog entry at the top of the `___NOTES___` section.

## Steps

1. **Get today's date** in `YYYY-MM-DD` format.

2. **Identify the diff** — run these in order, use whichever returns output:
   ```bash
   git diff HEAD -- template.tpl          # unstaged changes
   git diff --cached -- template.tpl      # staged changes
   git diff HEAD~1 HEAD -- template.tpl   # last commit
   ```

3. **Write bullet points** — one per logical change, **at most 5 total**. Each bullet describes **what changed and why**: the functional impact, the implementation decision, or both when both matter. Avoid bullets that only repeat how (e.g. "updated variable X") — say what effect it has. Version bumps always get their own bullet. When needed to stay within the limit, group related changes into a single bullet: low-level items (e.g. helper imports that exist solely to support a feature) belong under that feature's bullet, and thematically related items (e.g. multiple help text improvements, several parameter moves) can share a single bullet under a common theme.

4. **Insert the new entry** directly below the `___NOTES___` line, above any existing entries. The `Created on` line always stays at the bottom.

## Format

```
___NOTES___

YYYY-MM-DD - Change Notes:
  - Change description

[previous entry if any]

Created on MM/DD/YYYY, H:MM:SS AM/PM
```

Each entry separated by one blank line. Two-space indent on bullets.

## Placement Rule

```
___NOTES___
                          ← blank line
[NEW ENTRY HERE]          ← insert here
                          ← blank line
[older entry, if any]
                          ← blank line
Created on ...            ← never move this
```

## Example

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

## Common Mistakes

| Mistake | Fix |
|---|---|
| Fabricating entries not in the diff | Only write what the diff shows — never invent or copy from examples |
| Moving or duplicating `Created on` | Leave it exactly as-is at the bottom |
| Inserting below existing entries | New entry always goes at the top |
| Bullets describing only how, not what/why | Each bullet must convey the change's purpose or effect |
| Missing version bump bullet | Always include version bumps when present in the diff |
