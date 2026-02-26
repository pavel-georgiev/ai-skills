---
name: review-spec
description: Review a spec file for completeness, consistency, and issues
---


# Spec Reviewer

## Step 1: Read and Analyze the Spec

Read the spec file at `$ARGUMENTS` using the Read tool.

If the file doesn't exist or isn't provided, ask the user to provide a valid spec file path.

## Step 2: Initial Analysis

Analyze the spec and identify:

### Completeness Check
- Are all acceptance criteria clearly defined?
- Are edge cases and error handling specified?
- Are there clear implementation steps?
- Is the scope well-defined?
- Are dependencies listed?

### Consistency Check
- Are type definitions consistent throughout?
- Do API request/response examples match the type definitions?
- Are naming conventions consistent?
- Do implementation tasks align with the acceptance criteria?

### Technical Sanity Check
- Are the proposed solutions technically feasible?
- Are there any architectural concerns?
- Are there missing integration points?
- Are there potential race conditions or timing issues?
- Are security considerations addressed?

### Underspecified Areas
- What decisions are left ambiguous?
- What alternative approaches weren't considered?
- What assumptions need validation?

## Step 3: Report Issues

Present findings organized by category:
1. **Critical Issues** - Blockers that must be resolved
2. **Warnings** - Potential problems or ambiguities
3. **Suggestions** - Improvements that would strengthen the spec
4. **Questions** - Areas needing clarification from the user

## Step 4: Interview for Clarifications

Use the AskUserQuestion tool to gather clarifications on:
- Ambiguous requirements
- Missing technical details
- Design decisions that need user input
- Priority of identified issues

Continue interviewing until all critical issues and questions are resolved.

## Step 5: Update Spec

After gathering clarifications, ask the user:
"Would you like me to update the spec with these clarifications and fixes?"

If yes:
1. Apply the fixes and clarifications to the spec
2. Re-read the updated spec to verify changes
3. Output a summary of changes made

If no:
1. Output a summary of recommended changes for manual implementation

## Step 6: Ready for Implementation

After updating the spec, output:
1. Summary of the spec status (complete/needs work)
2. Number of tasks remaining (unchecked items)
3. Ask: "Would you like me to start implementing? I'll update the checkboxes as I progress."
