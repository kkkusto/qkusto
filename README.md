1. Job Overview
Job IDs: Focus on 711 and 712, both generating four files each.

Types of jobs:

711 → Weekly job

712 → Monthly job

Data Source: Both jobs use almost the same set of tables; the main difference is the data period considered.

📌 2. Dependency Management
711/712 jobs have 10 dependencies each.

The team analyzed these dependencies by looking into predecessor and successor jobs, which initially resulted in 100+ jobs.

But all 10 dependencies are linked to migrated table loads (no need to backtrack to the source).

The team decided: Find the equivalent RAW PHASE 2 jobs and handle dependencies only at that level.

No need to re-create the entire chain from source — just maintain the dependency at the migrated table level.

📌 3. Data Freshness
These dependencies exist to ensure the latest data is pulled after all relevant data loads complete.

Without the dependencies, there's a risk of missing a day’s data.

For 711: uses 10 tables — the same-day load completion is mandatory to avoid missing data.

Since all these are migrated tables, only the equivalent RAW PHASE 2 dependencies are needed.

No need to go back to the source data.

📌 4. Confirmation
Speaker 3 confirms: 711 and 712 are loading the migrated tables — no need to go further back.

📌 5. Filtering
Speaker 2 shows that all tables used in the extract are Target tables.

All relevant load jobs for these tables are already in the dependencies (400 series jobs).

The 700 series jobs are the open batch jobs with dependencies attached.

Analysis revealed: No further backtracking needed.

Conclusion: Only 700 series dependencies need to be created.

📌 6. Additional Use Cases
Use case: Pros and Complaints

Complaints:

21 jobs → 19 complaints-related

Weekly and monthly files are generated.

Each table has one job to unload data and generate extracts.

In total: 44 jobs → 42 simple jobs, 2 complicated ones.

All jobs use 1-2 tables to unload data and share externally.

Dependencies:

Minimum 2–3 dependencies per job, always includes 736 (Clause job dependency).

400 series jobs are mostly related to data load jobs of migrated tables.

Recommendation:

Only manage 700 series dependencies.

Ignore 400 series dependencies.

RAW PHASE 2 team will map the equivalent load jobs.

Speaker 2 will provide:

List of tables for which RAW PHASE 2 equivalent jobs are needed.

The team will map the dependencies accordingly.

📌 7. Status
34 jobs related to Pros and Complaints.

44 jobs total in this use case.

Speaker 2 will share details about Outbound jobs (ABOP).

ABOP outbound details shared.

ABOP inbound still in progress.

Dependencies to be clarified and finalized.

📌 8. Migration Challenges
Some tables are generated during global preprocessing of migrated jobs.

The target tables exist in RAW PHASE 2, but intermediate tables are missing.

RAW PHASE 2 team to provide information on where these intermediate tables exist.

Column level mismatch identified between equivalent tables in RAW PHASE 2:

Expectation: Table structure in RAW PHASE 2 should match the original.

Some exceptions exist where additional columns are missing — impacts web service feeds.

Need to raise this with RAW PHASE 2 team.

📌 9. Tactical vs. Non-Tactical Tables
Speaker 2 raises an observation:

Tactical and non-tactical tables have the same structure but may differ in data retention.

Could potentially reuse non-tactical tables instead of developing new tactical tables to save time.

Approval needed for this approach.
