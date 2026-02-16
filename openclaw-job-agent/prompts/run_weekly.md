# run_weekly.md — Weekly Analytics Execution Prompt (Craigslist D-Pipeline)

## MODE
Weekly analytics + performance diagnostics + optimization recommendations.

This mode performs NO browsing and NO packet generation.
It reads historical data only.

---

## COMPLIANCE
- Do NOT open Craigslist URLs.
- Do NOT access any external websites.
- Operate ONLY on repository data files.
- Do NOT modify applications.csv.
- Generate a new weekly report file only.

---

## REQUIRED INPUT FILES (READ ONLY)

1) data/applications.csv  
2) data/queue.json (latest)  
3) config/scoring_rules.json  
4) data/weekly_reports/ (existing reports, if any)

---

## REPORT OUTPUT

Generate file:

`data/weekly_reports/YYYY-WW.md`

Where:
- YYYY = current year
- WW = ISO week number

Use America/Los_Angeles timezone.

---

## WEEK WINDOW DEFINITION

Analyze entries in applications.csv where:

- date_found is within the last 7 days (inclusive), OR
- date_sent is within the last 7 days.

If dates are missing or malformed, ignore those rows.

---

## ANALYSIS SECTIONS (MANDATORY)

# 1. Intake Metrics

Compute:

- total_listings_processed
- priority_count
- review_count
- skip_count
- avg_score
- median_score
- highest_score
- lowest_score

Also compute:

- listings_per_day (average)
- percent_priority = priority_count / total_listings_processed

---

# 2. Application Pipeline Metrics

Compute:

- prepared_count
- sent_count
- followup1_count
- followup2_count
- closed_count
- response_count (if tracked in notes/status)
- response_rate = response_count / sent_count (if sent_count > 0)

Compute average time:

- days_found_to_prepared
- days_prepared_to_sent

If missing data, state “insufficient data”.

---

# 3. Keyword Effectiveness Analysis

Using scoring_rules.json and applications.csv:

For listings with score >= priority_min_score:
- Count frequency of matched_positive_keywords
- Rank top 10 positive keywords

For skipped listings:
- Count frequency of matched_negative_keywords
- Rank top 10 negative keywords

Identify:

- High-signal keywords (frequent in Priority)
- High-noise keywords (frequent in Skip)

---

# 4. Search Stream Effectiveness

From applications.csv notes or URL patterns:

Group listings by search keyword origin if identifiable.

If search origin not stored, group by dominant matched keyword.

Compute per group:

- listings_count
- avg_score
- priority_rate
- skip_rate

Identify:

- Top-performing keyword streams
- Underperforming streams (high skip_rate)

---

# 5. Red Flag & Risk Analysis

Scan notes for:

- scam indicators
- vague employer identity
- commission-heavy roles
- crypto or speculative schemes

Count:

- total_red_flags
- red_flag_rate = red_flags / total_listings_processed

If red_flag_rate > 25%, recommend tightening filters.

---

# 6. Operational Efficiency

Compute:

- packets_generated
- packets_generated / priority_count
- unprepared_priority_count (priority but no packet)

Identify bottlenecks:

- Too many Priority listings?
- Too many Review listings?
- Too few Priority listings?

---

# 7. Recommendations (MANDATORY)

Provide structured recommendations:

### A. Keyword Adjustments
- Add keywords (based on high-signal roles)
- Remove or downweight keywords (high noise)

### B. Threshold Adjustments
- Raise/lower priority_min_score if needed
- Raise/lower review_min_score if needed

### C. Saved Search Optimization
- Recommend retiring noisy searches
- Recommend adding new signal keywords

### D. Operational Adjustments
- Adjust daily cap
- Increase/decrease review threshold
- Change follow-up cadence if response rate low

Each recommendation must include a rationale grounded in metrics.

---

## REPORT FORMAT

The weekly report must be structured, deterministic, and machine-readable.

Use the following section headers exactly:

# Weekly Job Pipeline Report — YYYY Week WW

## Summary Snapshot
## Intake Metrics
## Application Pipeline Metrics
## Keyword Effectiveness
## Search Stream Effectiveness
## Red Flags & Risk
## Operational Efficiency
## Recommendations
## Action Plan for Next Week

Under “Action Plan for Next Week”, list:

1.
2.
3.
4.
5.

---

## FAILURE HANDLING

If:

- applications.csv missing
- No entries in last 7 days
- Corrupt data rows

Then:

- Generate report with section headers
- State “No valid data for analysis”
- Do not fail execution

---

## FINAL OUTPUT BEHAVIOR

At end of execution, print:

- report_file_path
- total_rows_analyzed
- priority_count
- sent_count
- response_rate (if calculable)
- number_of_recommendations_generated

No conversational text.
No commentary.
Only structured operational output.

