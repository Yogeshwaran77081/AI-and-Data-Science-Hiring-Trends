# DAX Measures and Calculated Columns

This document lists every DAX object in the report, with **why** it exists before **what** it does.
Table names: `AI Job Market Dataset` (main fact), `dim_country`, `DateTable`, `Fact Skill unpivoted`.

**Types:** *Measure* = evaluated in the visual's filter context. *Calculated column* = evaluated once per row and stored in the model.

---

## 1. Core KPI measures

### Total Postings (measure)
**Why:** the base count that most other measures build on.
```dax
Total Postings =
COUNTROWS('AI Job Market Dataset')
```

### Total Openings (measure)
**Why:** one posting can hold several vacancies, so openings show real hiring volume.
```dax
Total Openings =
SUM('AI Job Market Dataset'[job_openings])
```

### Avg Salary (measure)
**Why:** the central pay metric across all pages.
```dax
Avg Salary =
AVERAGE('AI Job Market Dataset'[salary])
```

### Min Salary / Max Salary (measures)
**Why:** show the salary range by company size, not just the average.
```dax
Max Salary =
MAX('AI Job Market Dataset'[salary])

Min Salary =
MIN('AI Job Market Dataset'[salary])
```

### Average Skill Count (measure)
**Why:** summarises how many of the five tracked skills a typical posting asks for (about 2.51).
```dax
Average Skill Count =
AVERAGE('AI Job Market Dataset'[Skill Count])
```

---

## 2. Hiring urgency

### High Urgency Postings (measure)
**Why:** isolates postings marked "High" so urgency can be measured against total volume.
```dax
High Urgency Postings =
CALCULATE(
    [Total Postings],
    'AI Job Market Dataset'[hiring_urgency] = "High"
)
```

### % High Urgency (measure)
**Why:** a share is comparable across countries and roles, while a raw count is not. `DIVIDE` avoids errors when the denominator is zero.
```dax
% High Urgency =
DIVIDE(
    [High Urgency Postings],
    [Total Postings]
)
```

---

## 3. Skills

### Skill Count (calculated column)
**Why:** the five skill flags are 0/1, so adding them gives the number of skills per posting, which drives the skill-count analysis.
```dax
Skill Count =
'AI Job Market Dataset'[skills_python] +
'AI Job Market Dataset'[skills_sql] +
'AI Job Market Dataset'[skills_ml] +
'AI Job Market Dataset'[skills_deep_learning] +
'AI Job Market Dataset'[skills_cloud]
```

### SkillBucket (calculated column)
**Why:** grouping skill counts into buckets makes the salary comparison readable. (The maximum count is 5, so the "6+ Skills" branch is never reached.)
```dax
SkillBucket =
SWITCH(
    TRUE(),
    'AI Job Market Dataset'[Skill Count] <= 1, "0–1 Skills",
    'AI Job Market Dataset'[Skill Count] <= 3, "2–3 Skills",
    'AI Job Market Dataset'[Skill Count] <= 5, "4–5 Skills",
    "6+ Skills"
)
```

### Skill Combo (calculated column)
**Why:** turns five flags into one readable label such as "Python + SQL + ML", so combinations can be ranked by salary.
```dax
Skill Combo =
VAR SkillsList =
{
"Python", 'AI Job Market Dataset'[skills_python],
"SQL", 'AI Job Market Dataset'[skills_sql],
"ML", 'AI Job Market Dataset'[skills_ml],
"Deep Learning", 'AI Job Market Dataset'[skills_deep_learning],
"Cloud", 'AI Job Market Dataset'[skills_cloud]
}
VAR Pairs =
GENERATESERIES(0, 4, 1) // indices for 5 pairs
VAR SkillRows =
ADDCOLUMNS(
Pairs,
"Name", SWITCH([Value],
0, "Python",
1, "SQL",
2, "ML",
3, "Deep Learning",
4, "Cloud"
),
"Flag", SWITCH([Value],
0, 'AI Job Market Dataset'[skills_python],
1, 'AI Job Market Dataset'[skills_sql],
2, 'AI Job Market Dataset'[skills_ml],
3, 'AI Job Market Dataset'[skills_deep_learning],
4, 'AI Job Market Dataset'[skills_cloud]
)
)
RETURN
CONCATENATEX(
FILTER(SkillRows, [Flag] = 1),
[Name],
" + "
)
```
*Note:* the `SkillsList` variable is not used by the final result. A simpler equivalent is on the roadmap.

### Highest Combo Salary (measure)
**Why:** finds the best average salary across all skill combinations, to show as a KPI.
```dax
Highest Combo Salary =
MAXX(
    SUMMARIZE(
        'AI Job Market Dataset',
        'AI Job Market Dataset'[Skill Combo],
        "AvgSal", AVERAGE('AI Job Market Dataset'[salary])
    ),
    [AvgSal]
)
```

### Highest Paying Skill Combo (measure)
**Why:** returns the *name* of the best-paying combination, so the headline updates automatically when filters change. `TOPN` picks the single highest row.
```dax
Highest Paying Skill Combo =
VAR T =
    TOPN(
        1,
        SUMMARIZE(
            'AI Job Market Dataset',
            'AI Job Market Dataset'[Skill Combo],
            "AvgSal", AVERAGE('AI Job Market Dataset'[salary])
        ),
        [AvgSal],
        DESC
    )
RETURN
    MAXX(T, 'AI Job Market Dataset'[Skill Combo])
```
*Sample size check:* in the full dataset every one of the 32 combinations has between 284 and 356 postings, so the ranking is not driven by tiny groups.

---

## 4. Salary buckets and experience

### SalaryBucket (calculated column)
**Why:** groups salaries into bands for distribution views.
```dax
SalaryBucket =
SWITCH(
    TRUE(),
    [salary] < 50000, "<50K",
    [salary] < 100000, "50K–100K",
    [salary] < 150000, "100K–150K",
    "150K+"
)
```

### Experience Group (calculated column)
**Why:** groups years of experience into bands.
```dax
Experience Group =
SWITCH(
    TRUE(),
    'AI Job Market Dataset'[years_experience] <= 2, "0-2",
    'AI Job Market Dataset'[years_experience] <= 5, "3-5",
    'AI Job Market Dataset'[years_experience] <= 10, "6-10",
    "Above 10"
)
```
*Note:* in this synthetic dataset `years_experience` is unrelated to `experience_level` and to salary, so the report analyses `experience_level` instead.

---

## 5. Cost-of-living adjustment

### Avg Salary (COL Adjusted) (measure)
**Why:** compares pay after accounting for how expensive each country is. Dividing by the index and multiplying by 100 restates salary as if the cost of living were 100 (the USA baseline).
```dax
Avg Salary (COL Adjusted) =
DIVIDE(
    [Avg Salary],
    AVERAGE(dim_country[avg_cost_of_living_index])
) * 100
```
*Known issue:* when visuals use the country column from the fact table, the relationship direction means `dim_country` is not filtered by country, so the index is averaged across the tech-hub tier. The fix is to use `dim_country[country]` in the visuals. See the README, Known Issues, item 1.

---

## 6. Insight text and time

### Top Hiring Role (measure)
**Why:** a text measure that writes its own insight sentence, so the headline never goes stale when filters change.
```dax
Top Hiring Role =
VAR TopRole =
    TOPN(
        1,
        SUMMARIZE(
            'AI Job Market Dataset',
            'AI Job Market Dataset'[job_title],
            "Jobs", COUNT('AI Job Market Dataset'[job_id])
        ),
        [Jobs],
        DESC
    )
RETURN
"The highest hiring demand is for " &
MAXX(TopRole, 'AI Job Market Dataset'[job_title]) &
" with " &
FORMAT(MAXX(TopRole, [Jobs]), "#,##0") &
" postings."
```

### YoY Postings Growth (measure)
**Why:** year-over-year change in postings. It is defined but not currently shown on a report page.
```dax
YoY Postings Growth =
VAR CurrentYear = MAX('DateTable'[Year])
VAR PrevYear = CurrentYear - 1
VAR CurrentPostings =
    CALCULATE(
        [Total Postings],
        'DateTable'[Year] = CurrentYear
    )
VAR PrevPostings =
    CALCULATE(
        [Total Postings],
        'DateTable'[Year] = PrevYear
    )
RETURN
DIVIDE(
    CurrentPostings - PrevPostings,
    PrevPostings
)
```

---

## 7. Interview cheat sheet

| Concept | Where it appears |
|---|---|
| `CALCULATE` with a filter | High Urgency Postings |
| Safe division with `DIVIDE` | % High Urgency, YoY Postings Growth, COL Adjusted |
| `SUMMARIZE` + `MAXX` / `TOPN` | Highest Combo Salary, Highest Paying Skill Combo, Top Hiring Role |
| `SWITCH(TRUE(), ...)` for banding | SkillBucket, SalaryBucket, Experience Group |
| Variables (`VAR`) for readability | Skill Combo, Top Hiring Role, YoY Postings Growth |
| Filter context and relationship direction | COL Adjusted (see known issue) |
| Calculated column vs measure | Skill Count (column) vs Avg Salary (measure) |
