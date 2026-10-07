# Build status

Tracks what has actually been built in Airtable against `proposals/proposal-analysis.md`,
separate from that document's forward-looking analysis. Update this file, not
`proposal-analysis.md`, as pieces get built. Last updated: 2026-08-30.

Target base: **WOWSA Ratifications** (`apppdxgNikq9FRbPp`). The old production base
(`appgUjmgd0K8WWp31`) that `docs/06-airtable.md` describes is untouched by any of this
work and remains the live system until a cutover decision is made.

---

## Relational structure

The target base already had the intended tables (Swims, Submissions, Ratifications,
Decision Versions, Committee Votes, Comments, Evidence, People, Rulesets) but zero
records and no linked-record relationships between any of them. Built:

- Submissions → Swim, Submissions → Applicant
- Swims → Swimmer(s), Swims → Observer(s)
- Ratifications → Submission
- Decision Versions → Ratification
- Committee Votes → Decision Version, Committee Votes → Committee Member
- Comments → Submission
- Evidence → Submission
- Supporting lookups/rollups that read across these links (swim details surfaced on
  Submissions, evidence types rolled up, vote values rolled up)

## Built and verified

| Item | State | Notes |
|---|---|---|
| Relay as a data field, not a second workflow | Confirmed, no build needed | Already true in the target schema — `Swims.Swim Type` includes Relay, `Swimmer(s)`/`Observer(s)` are multi-link so a relay team fits one record |
| Add-on instructions automation | Built, **off** | `Add-on Instructions on Purchase`, triggers on the new Orders table, 7 branches covering every combination of Record Attempt / Live Map Tracking / Blog Recap & Social Sharing plus a fallback. Email copy is new writing — no such copy existed anywhere in the current system (`docs/12-defects.md` documents this as a known gap). Needs review before activating. |
| Swim-happened detection | Built, logic verified, **off** | Daily cron. Verified against dummy Swim/Submission records with a past and future finish date — caught only the past one. Test records deleted after. Left off because only a human can switch on an Airtable automation from the UI regardless of API state. |
| Route Approval Queue | Built, published | Page 1 of the Ratifications Ops interface. One screen: distance, start/finish, the four Gate 1 flags. `Route Approval Status`/`Notes`/`Date` fields added to Ratifications to record the decision on the record itself, replacing the Google Maps screenshot + email loop. |
| Evidence Completeness scoring | Built | Formula field on Submissions, checked against the three criteria in `docs/04-forms.md` (observer log, photo/video, GPS track). Paired with a Post-Swim Evidence Queue interface page (page 2). |
| Orders table | Built, not wired live | Captures what a WooCommerce order webhook needs: line item, Solo/Relay variation, add-ons, buyer identity kept separate from swimmer identity (mirrors the old Table 1 pattern). No live webhook — needs real WooCommerce credentials to wire and test, which this build did not have. Ready for a human to wire the trigger in the Airtable UI. |

## Blocked, not built

**Google Drive folder/file collection automation.** Two real obstacles, not just a missing
value:
1. The Drive folder ID is redacted in this public repo (`[DRIVE_FOLDER_*]` placeholder) —
   real values live in `PRIVATE.md`, which was not available to this build.
2. Airtable's automation API has no native Google Drive action. It would require a
   `customScript` action — the same class of automation that broke the old order automation
   (see `docs/12-defects.md`), where a script-step automation can be read but not edited via
   the API.

`Drive Folder URL` and `Drive Folder Created` fields were added to Swims so this has
somewhere to write once a human builds it directly in the Airtable UI with the real folder ID.

## Deliberately withheld pending a decision

**Committee Decision formula.** Decision Versions already anticipated this once Committee
Votes existed — it does now. Not built, because it runs directly into the open governance
question from `proposals/proposal-analysis.md`:

> Is the rule three ratified reviews regardless of any not-ratified reviews from others, or
> is it the first three respondents to decide?

`Votes Cast` and a raw `Committee Vote Values` rollup were added as a stopgap so votes can
be read by hand until this is resolved. Build the formula only after this question has an
answer — guessing wrong here changes real ratification outcomes.

## Not attempted this run

- Introduction generation (Claude-agent piece)
- Committee dispatch + reminder automation (depends on the Committee Decision formula above)
- Certificates/social assets generation (Claude-agent-driven Canva piece)
- The distance-calculator integration as the single source for both distance figures
  (separate repo, `rosevillanuevadev/wowsa-distance-calculator` — still needs circumnavigation-route
  work with Anthony at ZeroSixZero per `proposals/proposal-analysis.md` recommendation 3)
- The offline-capable observer log form (explicitly unsolved R&D per `proposals/proposal-analysis.md`,
  not a standard build task)
- Lovable public pages, committee review view, determination/publishing panel, public status
  pages, permanent public registry — none of this run touched Lovable. This build was Airtable
  backend only.
