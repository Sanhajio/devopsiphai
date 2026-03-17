# Post-Run
 
This file is read after every audit run.

---

## Step 1 — Dump Report

```bash
TIMESTAMP=$(date +%Y-%m-%d_%H%M%S)
PROJECT_NAME=$(basename "$PROJECT_PATH")
REPORT_DIR="/tmp/devopsiphai/${TIMESTAMP}"
REPORT_FILE="${REPORT_DIR}/${PROJECT_NAME}-audit-${TIMESTAMP}.md"

mkdir -p "$REPORT_DIR"
# write full audit output to $REPORT_FILE
```

Confirm to the user:
```
Report saved to: $REPORT_FILE
```

---

## Step 2 — Skill Improvement Prompt

Ask:
```
Would you like to suggest improvements to the devopsiphai skill based on this run?
[yes / no]
```

If yes, ask:
```
What should be improved?
```

Append to the run directory:
```bash
cat >> "${REPORT_DIR}/feedback.md" << FEEDBACK
# Feedback — ${TIMESTAMP}
<user feedback verbatim>
FEEDBACK
```

Confirm:
```
Feedback saved to: ${REPORT_DIR}/feedback.md
Say "improve devopsiphai based on last feedback" to apply it.
```

If no, skip silently.
