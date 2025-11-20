# MLE-Bench Debugging Assistant

An expert Python debugging system specialized for MLE-bench competition code. This system analyzes buggy code, identifies root causes, determines fixability, and provides minimal fixes.

## Overview

The MLE-Bench Debugging Assistant is designed to:
- Analyze Python errors in machine learning competition code
- Provide structured bug analysis with line-by-line explanations
- Determine if bugs are fixable with minimal changes
- Generate targeted fix plans and corrected code
- Handle cascading bugs through multi-step debugging

## Key Features

### ✅ Minimal Fix Philosophy
- Changes the fewest lines possible (typically 4-8 lines)
- Adds error handling instead of restructuring code
- Preserves original data preprocessing logic
- Avoids unnecessary refactoring or optimization

### 🎯 Common Bug Pattern Recognition
- ZIP extraction issues
- Read-only filesystem errors
- Compressed file handling (7z, zip)
- KeyError and missing data robustness
- Missing library dependencies

### 📊 Structured Output
- Classification fields (TRUE/FALSE, 0/1/2)
- Detailed analysis with line numbers
- Numbered fix plans or unfixable explanations
- Fixed code with minimal changes
- Time estimates for implementation

## Quick Start

### 1. Prepare Your Input

Use the template in `templates/input-template.md`:

```markdown
Competition ID: siim-covid19-detection
Debug Step: 0

=== BUGGY CODE ===
[Your Python code]

=== ERROR OUTPUT ===
[Traceback/error message]

=== CLIENT'S ANALYSIS === (Optional)
[Initial analysis]

=== CLIENT'S PLAN === (Optional)
[Proposed fix]
```

### 2. Submit to LLM

Combine the system prompt with your input:

```
[Paste contents of docs/system-prompt.md]

---

NEW DEBUGGING REQUEST

[Paste your filled input template]
```

### 3. Review Output

The assistant will return:
- Classification table with TRUE/FALSE values
- Detailed analysis paragraph with line numbers
- Bug fix plan (numbered steps or unfixable explanation)
- Fixed code (if applicable)
- Time estimate

## Directory Structure

```
.
├── README.md                  # This file
├── docs/
│   └── system-prompt.md      # Complete system prompt for LLM
├── templates/
│   ├── input-template.md     # Input format template
│   └── output-template.md    # Expected output format
└── examples/
    └── example-session.md    # Sample debugging session
```

## Usage Examples

### Example 1: Fixable KeyError

**Input:**
- Competition: siim-covid19-detection
- Error: KeyError on dictionary lookup
- Root cause: Missing defensive checks

**Output:**
- bug_fixed: TRUE
- Plan: Add key existence check with fallback
- Time: 2-3 minutes

### Example 2: Unfixable Architecture Issue

**Input:**
- Competition: organ-segmentation
- Error: Missing mask directory
- Root cause: Expects image files, but masks are RLE strings in CSV

**Output:**
- bug_fixed: FALSE
- Explanation: Requires complete data loading redesign
- Recommendation: Beyond minimal fix scope

## Classification Fields

### bug_confirmed
- **TRUE**: Error was thrown on current code
- **FALSE**: Code ran without errors
- **Applies to**: Every debug_step

### proposed_debug_analysis_accurate (debug_step 0 only)
- **0**: Client's bug never occurred
- **1**: Bug correct but analysis incomplete
- **2**: Analysis fully accurate
- **N/A**: debug_step > 0

### initial_bug_reproducible (debug_step 0 only)
- **TRUE**: Bug would occur when running code
- **FALSE**: Bug wouldn't actually occur
- **N/A**: debug_step > 0

### bug_fixed
- **TRUE**: Bug fixed or didn't exist
- **FALSE**: Bug unfixable

### all_bugs_fixed
- **TRUE**: All bugs resolved (final step)
- **FALSE**: More bugs remain

## Common Bug Patterns

### Template A: ZIP Extraction
```
Error: FileNotFoundError
Files: train.zip, test.zip in ./data/
Fix: Extract to writable directory
Time: 5-7 minutes
```

### Template B: Read-Only Filesystem
```
Error: OSError [Errno 30]
Cause: Extracting to read-only ./data/
Fix: Change to extractall(".")
Time: 1-2 minutes
```

### Template C: 7z Files
```
Error: FileNotFoundError for .json (exists as .json.7z)
Fix: Install py7zr and extract
Time: 3-5 minutes
```

### Robustness: KeyError
```
Error: KeyError on dictionary lookup
Fix: Add key existence check
Time: 2-3 minutes
```

### Library Installation
```
Error: ModuleNotFoundError
Fix: try/except with subprocess pip install
Time: 2-3 minutes
```

## Unfixable Bug Criteria

Mark as **bug_fixed: FALSE** when:
- Requires internet access (blocked in container)
- Library installation fails despite attempts
- Architecture incompatible with data format (needs rewrite)
- Missing system-level dependencies

## Best Practices

### For Bug Analysis
1. Always include exact line numbers
2. Quote exact error messages
3. Explain causal chain from root cause to error
4. Use third-person tense
5. Be specific and direct

### For Fix Plans
1. Reference specific line numbers
2. Explain what changes and why
3. List what remains unchanged
4. Keep plans under 5 steps when possible
5. Use numbered lists for fixable bugs
6. Use prose paragraphs for unfixable bugs

### For Fixed Code
1. Show minimal context (2-3 lines before/after)
2. Mark changed lines with comments
3. Preserve original style and formatting
4. Include only necessary changes
5. Test that changes would actually work

## Tips for Users

- **Start with debug_step 0** for initial bugs
- **Increment debug_step** for cascading bugs after fixes
- **Provide full tracebacks** for accurate analysis
- **Include data context** when relevant (file structure, formats)
- **Trust minimal fixes** over perfect rewrites
- **Try robustness fixes** before marking unfixable

## Contributing

This debugging assistant system is designed to be:
- **Extensible**: Add new bug patterns as templates
- **Reusable**: Apply to any Python competition code
- **Iterative**: Handle multi-step cascading bugs
- **Practical**: Focus on what can actually be fixed

## License

This project is available for educational and research purposes.

## Support

For questions, improvements, or bug reports:
1. Review the `docs/system-prompt.md` for detailed guidelines
2. Check `examples/example-session.md` for reference
3. Use templates in `templates/` for consistent formatting
4. Submit issues or suggestions via repository issues

---

**Version**: 1.0.0
**Last Updated**: 2025-11-20
**Maintained by**: MLE-Bench Debugging Team
