---
description: Explain a BigQuery feature, function, or concept with working examples. Pass the feature name or question.
argument-hint: "<feature or question>"
---

# BigQuery Feature Explainer

Explain the requested BigQuery feature using the `bigquery-features` skill.

## Instructions

1. **Determine the feature to explain:**
   - If an argument is provided, use it as the feature name or question.
   - If no argument is provided, look for the most recent BigQuery feature question in the current conversation.
   - If no feature is identifiable, ask the user what they want to learn about.

2. **Explain the feature** using the bigquery-features skill, covering all four aspects:
   - **What it is:** concise definition.
   - **When to use it:** concrete use cases and when to prefer it over alternatives.
   - **Working example:** complete, runnable BigQuery SQL.
   - **Common pitfalls:** gotchas, limits, performance traps.

3. **Cross-reference cost implications** using the `bigquery-optimization` skill when the feature affects cost or performance (e.g., MERGE DML quotas, BQML slot consumption, BI Engine reservations).

4. **Output format:**

```
## <Feature Name>

### What It Is
<definition>

### When to Use
<use cases>

### Example
<working SQL>

### Pitfalls
<gotchas and limits>
```
