# Data Dictionary

## 1. `AI_Job_Market_Dataset.csv` (main dataset)

Source: *AI & Data Science Job Market Dataset (2020–2026)* by shree0910, Kaggle. Licence: CC BY-NC-SA 4.0.
Size: **10,345 rows × 19 columns**. No missing values and no duplicate rows were found.

| Column | Type | Description | Values / range |
|---|---|---|---|
| `job_id` | Integer | Unique posting identifier | 1 – 10,345 |
| `job_title` | Text | Role advertised | AI Engineer, Business Analyst, Data Analyst, Data Engineer, Data Scientist, Machine Learning Engineer |
| `company_size` | Text | Type or size of the hiring company | Enterprise, Medium, MNC, Startup |
| `company_industry` | Text | Industry of the hiring company | Technology, E-commerce, Finance, Healthcare, Education, Retail |
| `country` | Text | Country of the posting | Australia, Canada, Germany, India, Singapore, UK, USA |
| `remote_type` | Text | Work arrangement | Remote, Hybrid, Onsite |
| `experience_level` | Text | Seniority band | Entry, Mid, Senior |
| `years_experience` | Integer | Years of experience stated | 0 – 14 (unrelated to `experience_level` in this dataset) |
| `education_level` | Text | Education requirement | Bachelor, Master, PhD |
| `skills_python` | 0/1 flag | 1 = skill listed for the posting | 0, 1 |
| `skills_sql` | 0/1 flag | 1 = skill listed | 0, 1 |
| `skills_ml` | 0/1 flag | 1 = skill listed | 0, 1 |
| `skills_deep_learning` | 0/1 flag | 1 = skill listed | 0, 1 |
| `skills_cloud` | 0/1 flag | 1 = skill listed | 0, 1 |
| `salary` | Integer | Annual salary (shown in USD in the report; the currency is assumed) | 45,083 – 204,143 |
| `job_posting_month` | Integer | Month of posting | 1 – 12 |
| `job_posting_year` | Integer | Year of posting | 2020 – 2026 |
| `hiring_urgency` | Text | Urgency of the hiring need | Low, Medium, High |
| `job_openings` | Integer | Number of vacancies in the posting | 1 – 9 |

### Columns added in Power BI

| Column | Where | Description |
|---|---|---|
| `Posting dates` | Power Query | Date built from the posting year and month |
| `Skill Count` | DAX calculated column | Number of the five skills listed (0–5) |
| `SkillBucket` | DAX calculated column | 0–1, 2–3 or 4–5 skills |
| `Skill Combo` | DAX calculated column | Readable label such as "Python + SQL + ML" |
| `SalaryBucket` | DAX calculated column | Salary band |
| `Experience Group` | DAX calculated column | Years-of-experience band |

---

## 2. `dim_country` (reference table)

> **Illustrative reference table compiled with AI assistance using approximate values. It is not sourced from an official index.** Treat any result that depends on it as indicative only. Planned improvement: replace it with a published cost-of-living index and cite it.

| country_id | country | region | avg_cost_of_living_index | tech_hub_tier | timezone_group | avg_corporate_tax_rate |
|---|---|---|---|---|---|---|
| 1 | USA | North America | 100 | Tier 1 | Americas | 21 |
| 2 | Canada | North America | 85 | Tier 2 | Americas | 26 |
| 3 | UK | Europe | 88 | Tier 1 | Europe | 25 |
| 4 | Germany | Europe | 82 | Tier 1 | Europe | 30 |
| 5 | India | Asia | 38 | Tier 1 | Asia | 25 |
| 6 | Singapore | Asia | 95 | Tier 1 | Asia | 17 |
| 7 | Australia | Oceania | 90 | Tier 2 | Asia-Pacific | 30 |

Cost-of-living index: USA = 100 baseline. Corporate tax rate is in percent.

---

## 3. Model tables in Power BI

| Table | Role | Grain |
|---|---|---|
| `AI Job Market Dataset` | Main fact table | One row per job posting |
| `Fact Skill unpivoted` | Skill-level fact table | One row per posting per skill |
| `dim_country` | Country dimension | One row per country |
| `DateTable` | Date dimension | One row per date (with Year) |
