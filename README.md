# Company Research Pipeline — Fund Management (SIC 66300)

Source: `66300Fund Management.csv` (Companies House export, 5000 active UK entities under SIC 66300).

## Pipeline stages

1. **`01_raw_5000_cleaned.csv`** — the original 5000-row export, re-parsed to fix a malformed-CSV issue (unescaped commas inside company names, SIC-code lists, and addresses broke naive column splitting). This is the clean source of truth.
2. **`00_dropped_spv_entities.csv`** — 932 entities dropped as near-certain non-employers: obvious fund vehicles (GP/Feeder/Nominee/SPV/Holdco/Blocker/Bidco/Midco/Topco/roman-numeral fund series/LP suffix in the name), or companies registered *only* under SIC 66300 with no other listed activity AND incorporated 2024 or later (too new, single-purpose signal). Kept for reference, not deleted, in case a real name got mis-flagged.
3. **`02_tier0_pass_scored.csv`** — the remaining 4068 companies, each scored (`_score` column) on free signals only (no web calls): entity type (plc > ltd > guarantee/CIC types), SIC-code richness, presence of "real operating business" SIC codes alongside 66300, company age, and London registration. Higher score = more likely a real operating employer — a prioritization heuristic, not proof.
4. **`03_tier1_candidates_450.csv`** — top 450 by score. This is the pool that gets real web verification.
5. **`04_progress_tracker.csv`** — **the live working file.** One row per candidate company. Each scheduled run picks up the next batch of `check_status = pending` rows, checks for a real website + career page + open roles matching the target role categories below, and writes results back (`website_url`, `career_page_url`, `has_openings`, `matching_roles_found`, `last_checked_date`). Once `has_openings` is populated for enough companies, the best ~200-300 become the `shortlist` (flagged in that column) for weekly re-checks and eventual deep-dive (company profile + hiring contacts).

## Target role categories (job-match criteria)
- Analyst roles: IB, FP&A, credit, equity research
- Investment/portfolio roles: investment analyst, portfolio analyst, associate
- Operations/fund admin roles: fund operations, middle office, fund accounting
- Internships / graduate schemes

## Process state
- First full pass over the 450 candidates: not yet started.
- Once first pass completes, cadence switches to weekly re-checks of the confirmed shortlist only (not all 450), to catch newly posted roles.
- Deep-dive research (company profile, recruiter/hiring-contact discovery) only happens for shortlisted companies — not the full 450 or 5000.

## Notes / known limitations
- Companies House data alone cannot tell us which entities have employees. The scoring in stage 3 is a heuristic prioritization, not a guarantee — some good employers may score low (e.g. very new legitimate boutiques) and some will turn out to be false positives on the web-check pass. That's expected and handled by Tier 1.
- Registered office address is a legal filing address, not necessarily where staff actually work (many use accountants'/lawyers' addresses) — don't treat it as an office location without verification.
