---
name: update-add-hooks
description: The update adds hooks to the frontmatter.
hooks:
  PostToolUse:
    - command: sh scripts/format.sh
---

# Format

Format the changed files.
