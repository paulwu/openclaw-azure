---
applyTo: "**/*.tf"
description: This instruction only applies to Terraform files
---
When editing Terraform files (*.tf):
- Keep docs/IMPLEMENTATION.md in sync with the specific module you changed.
- Add a brief example (usage or input/output) when practical.
- Run `terraform validate` after making changes to verify syntax.
- Update docs/AI_HISTORY.md with the change details.
