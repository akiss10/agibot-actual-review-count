---
name: agibot-actual-review-count
description: Query iGenie Studio task rows by AGIBOT Task ID/UUID and report only the actual review count, computed as pending acceptance minus pending review. Use when the user supplies AGIBOT Task IDs and asks for 实际验收条数; do not use for unrelated task metadata.
---

# AGIBOT Actual Review Count

## Outcome

For each AGIBOT Task ID/UUID the user supplies, return only its actual review count:

`实际验收条数 = 待验收 - 待审核`

Do not include the two source values, the calculation, progress percentages, or other task metadata unless the user explicitly asks for them. Use the current page values; re-query each time instead of reusing earlier results.

## Workflow

1. Use browser automation on the authenticated iGenie Studio task page.
   - Target page: `https://igeniestudio.agibot.com/data/collection/tasks?rf=1`
   - Reuse the existing browser tab and session when available.
   - If the session is logged out, ask the user to log in and continue after they confirm. Never store or persist credentials in this skill or in its output.
2. For each Task ID/UUID:
   - Enter the ID in the control labeled `Task ID/UUID`.
   - Click `搜索`.
   - Wait until a result row's Task ID cell exactly equals the requested ID. Do not read the previous row while the asynchronous refresh is still in progress.
   - In that exact row, read the elements matching:
     - `div[data-v-c4deeabb]` whose exact text matches `^待审核：<number>`.
     - `div[data-v-c4deeabb]` whose exact text matches `^待验收：<number>`.
   - Compute `待验收 - 待审核`.
3. Return one line per requested ID:

```text
<Task ID>：<actual review count> 条
```

## Page Structure Notes

- The relevant row is the `<tr>` whose second data cell (`td:nth-of-type(2)`) contains the exact Task ID.
- Several ancestors can also carry `data-v-c4deeabb`; select only leaf-like elements whose trimmed text exactly starts with `待审核：` or `待验收：`.
- A zero result is valid when both source values are equal.
- If no exact row is found, or either source field is missing, report that the count could not be read for that ID. Do not guess.
- If the difference is negative, treat the page data as inconsistent and report the anomaly rather than presenting a negative acceptance count.
