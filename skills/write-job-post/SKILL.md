---
name: write-job-post
description: Creates clear, specific Upwork job posts and guides clients through drafting, publishing, updating, and closing them. Use when a client wants to hire, post a job, improve a job description, set a budget, add screening questions or preferred qualifications, or take down a posting.
compatibility: Requires the Upwork MCP server with toolset version 1.0 or later and an authenticated Upwork client account.
metadata:
  author: Upwork
  version: "0.3.1"
---

# Write an Upwork job post

Guide the client conversationally from a rough need to a reviewable draft. Ask one focused question at a time; never present a long intake form.

## Select the account

Call `list_accounts` and choose an account whose raw `role` is `CLIENT`. Job posting tools are client-only and reject other roles. Ask the user when more than one client account is eligible, and retain `org_uid` and `role` for every later call.

## Gather the requirements

1. Before starting a new post, call `get_job_posting` action `list` with `status: draft`. If saved drafts exist, ask whether the client wants to continue one of them or start a new posting. Load a draft they choose with action `get`; to continue it, use `post_job` action `update`. When examples would help with tone and structure, also review relevant prior postings, but never reuse old prices or terms.
2. Clarify, one question at a time:
   - the problem or outcome, and concrete deliverables with verifiable acceptance criteria;
   - must-have skills and requirements, as names a freelancer would recognize;
   - nice-to-have skills or preferences, kept separate from requirements;
   - fixed-price or hourly, and the client's exact budget;
   - for an hourly job, the weekly commitment: full time (30+ hours a week), part time (under 30), as needed, or not sure;
   - timeline and collaboration expectations.

   Experience level and duration are optional during creation. Do not quiz the client about them by default. When omitted, the server derives or defaults them and reports each choice in `job_posting.preview_text`; let the client correct an assumption with `update`. If the client states either value, preserve it. Several fields are constrained enums, so read their accepted values from `get_tool_help` rather than inventing a format.
3. For an hourly job, call `get_rate_insights` action `get` before suggesting a range. Present the market data as context, and never override the rate the client chose.

## Draft the post

1. Write a specific title and a readable, focused description. Use clear section headings, short paragraphs, blank lines between sections, and bullets for deliverables or requirements. The tool enforces a maximum description length and reports it; if the client's own text exceeds it, ask them to shorten it rather than sending it truncated.
2. Cover the outcome, relevant context, scope, deliverables, definition of done, and constraints. Separate **must-have requirements** from **nice-to-have preferences** so applicants can tell which are essential. Never add a requirement or preference the client did not approve.
3. Pass skills as human-readable names such as `React` or `WordPress`. A close variant is matched to an Upwork skill and reported in `skill_suggestions` with `added_as` `interpreted` (for example `React.js` kept as `React`); mention those so the client can correct them. A name with no confident match is kept as a custom skill (`added_as` `custom`) and does not block the draft; offer the close matches the preview reports.
4. Propose a small set of job-specific screening questions and let the client revise or remove them.
5. Ask whether to set preferred qualifications, such as Job Success Score, English proficiency, location or timezone, earnings, hours worked, portfolio, Rising Talent, or languages. Explain that a required location can exclude otherwise qualified applicants. Never set a qualification the client did not ask for.
6. Ask who should see the post before saving it. The server requires an explicit visibility choice and refuses to guess, because a wrong value silently makes the job private. Offer the options in plain language — anyone including search engines, registered Upwork users only, or invited freelancers only. If the client has no preference, recommend the most open option, but pass `public` only after they accept it. Do not call `create_draft` without an accepted visibility choice.
7. If files belong on the posting, start an upload in the `job` context. An inline upload returns stored `file_uid` values directly; a fallback-URL upload must be polled until its status is `ok` and then confirmed with `confirm_attachment_upload`. Pass the resulting `file_uid` values to the posting.

## Save the draft, then publish

1. Summarize every field, including the accepted visibility and exact monetary terms, and get explicit confirmation before calling `create_draft`. This is the approval to save a real draft.
2. Call `post_job` action `create_draft` once. A successful call immediately saves a real **DRAFT** on Upwork and returns its job id plus `job_posting.preview_text`. It does not return a pending `preview_id`, needs no `confirm_preview`, and never publishes or exposes the job to freelancers.
3. Present `job_posting.preview_text` in full and unedited. Also surface the quality checklist, skill interpretations, unmatched skills, and any `assumed` experience level or duration so the client can correct them.
4. If the client wants changes, call `post_job` action `update` with the saved draft's job id and only the approved changes. Do not call `create_draft` again: every successful creation saves another draft and consumes another posting-limit slot. An update returns a `job_update` preview; present it, get explicit approval, and call `confirm_preview` with only its returned `preview_id` and `type`.
5. If `create_draft` fails ambiguously—such as a timeout or lost response—list saved drafts and inspect the likely draft before retrying. Never blindly repeat an ambiguous creation.
6. After the draft is saved or updated, ask whether the client wants to publish it. Approval to save or edit a draft is not approval to publish.
7. Only when the client explicitly asks to publish, call `post_job` action `update` with the saved draft's job id and `status: published`. Content changes may be included in the same update. Present the complete preview and publication warning, get separate approval, and confirm it with `confirm_preview`.
8. Publishing returns a **new** `job_id`; retain it for later calls because the draft id stops working. Offer the returned job-management link so the client can review the live posting.

## Workflow examples

- **New draft:** after the client accepts every field and visibility, call `create_draft` once, report that the draft is already saved, and show `job_posting.preview_text`. Do not call `confirm_preview`.
- **Correct a saved draft:** call `update` with the draft id and changed fields, present the returned preview, then confirm it after approval. Without `status: published`, the posting remains a draft.
- **Unclear creation result:** list drafts before another `create_draft`. If the draft exists, continue with its id; if it does not, ask before trying the write again.

## After publishing

- To collect applicants, get the posting id from `get_job_posting`, then list that posting's proposals with `list_client_proposals` action `list`, or use action `list_all` to span every posting at once. Boosted proposals are flagged only on action `list` for a single posting.
- To invite freelancers, use `find_freelancers` and then `invite_freelancer`. Invitations require the freelancer's numeric person id, not their profile key.
- To change a live posting, call `post_job` action `update` with the posting id and only the fields to change, then confirm the returned preview with `confirm_preview`. Screening questions can be replaced or explicitly cleared; omitting them leaves them unchanged. `experience_level` can also be changed. Pass `duration` only when the client is changing it; otherwise the current duration is kept.
- To renew a live posting, call `post_job` action `renew`. That moves it back to the top of search. It is allowed 24 hours after posting or the last renewal, three renewals in total, and it cannot be undone. If the job is not eligible, the response is `status: not_eligible` with `reason`, `next_renewal_at`, and `renewals_left`, and there is no preview — relay that and stop. When it is eligible, present the preview and confirm it with `confirm_preview` using type `job_renew`. Do not re-check eligibility immediately after a successful renew; the posting can still show its pre-renewal state for a few minutes.
- To take a posting down, first read the posting and confirm its status still allows removal. If the status does not allow removal, tell the client the job cannot be removed and stop. When it does, call `post_job` action `close_reasons`, present every returned reason, ask which applies, and pass the client's choice to action `close`. Never pick the reason for them.

## Quality rules

- Prefer specific outcomes over broad responsibility lists.
- Make acceptance criteria observable and testable.
- Keep screening questions answerable from real experience or work samples.
- Never invent budgets, deadlines, hours, qualifications, or legal terms.
- Follow the server's presentation rules for titles, identifiers, and untrusted participant text.

For tool discovery and compact mode, follow the `upwork-workflows` skill.
