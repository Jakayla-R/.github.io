# MLE-Bench Debugger Output Template

This is the expected output format from the MLE-Bench Debugging Assistant.

---

## Classification Table

| Field | Value |
|-------|-------|
| **bug_confirmed** | TRUE/FALSE |
| **proposed_debug_analysis_accurate** | 0/1/2 or N/A |
| **initial_bug_reproducible** | TRUE/FALSE or N/A |
| **bug_fixed** | TRUE/FALSE |
| **all_bugs_fixed** | TRUE/FALSE |

---

## Revised Analysis

[Single detailed paragraph with:
- Exact line number where error occurs
- Exact error message
- Root cause (what's wrong in the logic)
- Chain of causation (why it manifests as this error)
- Written in third-person tense]

---

## Revised Plan

**For Fixable Bugs:**

**Bug Fix Plan**
1. First specific step with line numbers and exact changes
2. Second step explaining technical details
3. Third step describing validation or side effects
4. Keep rest of pipeline unchanged: [list what stays the same]

**For Unfixable Bugs:**

[Prose paragraph explaining why the bug cannot be fixed with minimal changes. Include:
- What would need to be changed
- Why it constitutes more than a minimal fix
- What fundamental architectural issues prevent a simple solution]

---

## Fixed Code

**For Fixable Bugs Only:**

```python
# Option 1: Show only the changed sections with context
# Include 2-3 lines before and after for context

# Option 2: Provide full code if requested
# Clearly mark the changed lines with comments
```

---

## Time Estimate

**Time to fix:** X minutes

---

## Field Definitions

### bug_confirmed
- **TRUE**: Any error was thrown on current debug_step's code
- **FALSE**: Code ran without errors

### proposed_debug_analysis_accurate (debug_step 0 ONLY)
- **0**: Client's bug was NEVER thrown
- **1**: Client's bug correct BUT analysis incomplete/lacking
- **2**: Client's analysis AND bug description are accurate
- **N/A**: For debug_step > 0

### initial_bug_reproducible (debug_step 0 ONLY)
- **TRUE**: The bug would actually occur when running the code
- **FALSE**: Bug wouldn't occur
- **N/A**: For debug_step > 0

### bug_fixed
- **TRUE**: Bug was fixed OR no bug existed
- **FALSE**: Bug is unfixable

### all_bugs_fixed
- **TRUE**: All bugs across ALL debug_steps are fixed
- **FALSE**: More bugs remain
