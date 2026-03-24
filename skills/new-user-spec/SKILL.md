---
description: Create user specification through adaptive interview (delegates to user-spec-planning via Codex shim)
---

# Instructions

Resolve and execute the target skill via shim:

```powershell
pwsh -File shared/scripts/dispatch-skill.ps1 -SkillAlias new-user-spec -AsPrompt
```