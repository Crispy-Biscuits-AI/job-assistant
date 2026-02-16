# run_daily.md — Daily Execution Prompt (Craigslist D-Pipeline)

## MODE
Daily intake + scoring + queue + packet generation.

## COMPLIANCE (MUST FOLLOW)
- Do NOT crawl Craigslist search pages.
- ONLY process Craigslist listing URLs explicitly provided in this run’s input (below).
- Do NOT access /reply or any disallowed paths.
- Do NOT send emails or submit forms; generate drafts only.
- Rate limits: max 10 listing pages per run, max 15 per day.
- Do NOT fabricate facts; only extract from listing + user truth packet/resume if provided.

## INPUT (PASTE BELOW EACH RUN)
Paste either:
A) A list of Craigslist listing URLs (one per line), OR
B) Raw Craigslist Search Alert email text that contains listing URLs.

### RUN INPUT START
PASTE_INPUT_HERE
### RUN INPUT END

## REQUIRED FILES (READ)
- config/scoring_rules.json
- data/seen_posts.json
- data/queue.json (may be overwritten for this run)
- data/applications.csv

## OUTPUT FILES (WRITE/UPDATE)
1) Update `data/seen_posts.json`
   - Extract post_id (numeric) from each URL.
   - Skip any post_id already present.
   - Append new post_ids + URLs that are processed this run.

2) Overwrite `data/queue.json`
   - Set generated_at to current local timestamp (America/Los_Angeles).
   - Fill arrays: priority, review, skip.
   - Each entry must include:
     - post_id, url, title, company, location, remote_status
     - compensation_raw, employment_type
     - score
     - matched_positive_keywords, matched_negative_keywords
     - required_skills, preferred_skills
     - instructions (subject line rules, attachments rules, etc.)
     - red_flags (array)
     - recommended_template ("email_short" or "email_default")
     - extraction_errors (array; empty if none)

3) Append to `data/applications.csv`
   - For EACH newly processed URL (even skipped), append a row with:
     - post_id
     - date_found = today’s date
     - date_prepared = today’s date if packet generated else blank
     - date_sent blank
     - company (or unknown)
     - title
     - location
     - remote_status (remote/hybrid/onsite/unknown)
     - pay_range (raw)
     - url
     - score
     - status = queued (or prepared if packet generated)
     - follow_up_date blank until sent
     - contact_method = unknown (do not access /reply)
     - instructions (short)
     - notes (short: fit summary or red flags)
     - packet_path (blank if none)

4) Generate Send Packets
   - For each PRIORITY item (and optionally REVIEW if score is within 1 point of priority threshold):
     - Create folder:
       `packets/YYYY-MM-DD_<postid>_<company_or_unknown>_<role_slug>/`
     - Write:
       - subject.txt (obey posting instructions exactly)
       - email_body.txt (tailored, concise, truthful)
       - resume_plaintext.txt (ONLY if posting asks for pasted resume; otherwise omit or leave empty with note)
       - notes.md (fit rationale, emphasis, risks)
     - Update applications.csv row:
       - status = prepared
       - date_prepared = today
       - packet_path set

## EXTRACTION STEPS (PER URL)
For each URL selected for processing (new and within cap):
1) Open listing page.
2) Extract:
   - title, location, posting date if visible
   - company if present
   - telecommuting/remote indicators
   - compensation/pay text
   - employment type if stated
   - requirements + preferred skills (as bullet lists)
   - explicit application instructions (subject line, paste resume, no attachments, etc.)
3) Red flag scan:
   - suspicious payment schemes
   - “wire money”, “gift cards”, “training fee”
   - commission-only roles masquerading as engineering
   - vague employer identity + off-platform pressure
4) Score:
   - Use scoring_rules.json (case-insensitive keyword matching across title + body).
   - Record matched keywords (exact tokens/phrases that triggered score).

## CLASSIFICATION
- Use thresholds from scoring_rules.json:
  - priority_min_score
  - review_min_score
- Anything below review_min_score → skip.

## SUMMARY OUTPUT (PRINT AT END)
Print a short run summary:
- total_urls_in_input
- new_processed
- skipped_as_seen
- priority_count
- review_count
- skip_count
- packets_generated_count
- any errors (count + brief list)

## STOP CONDITIONS
- If 10 new listing pages have been processed, stop even if input has more.
- If extraction fails on a URL, log extraction_errors and continue.

