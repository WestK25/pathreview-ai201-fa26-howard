## Unit 7 — Issue selection

**Issue link:** [Howard PathReview issue #12](https://github.com/codepath/pathreview-ai201-fa26-howard/issues/12)

**Issue title:** Add snapshot tests for prompt templates to catch accidental changes

**Tier:** [x] Tier 1  [ ] Tier 2  [ ] Tier 3

**Problem summary:** PathReview keeps five versioned prompts that tell the model how to review a portfolio. Its snapshot test hashes all five prompts together and only checks that the result has 32 characters. A change from “with specific examples” to “without specific examples” still passed all 37 prompt tests under the same version. I want the tests to identify which released prompt changed and let a contributor add a new version with its own snapshot.

**Branch name:** `test/12-versioned-prompt-snapshots`

**Setup confirmation:** [ ] App runs locally at localhost:5173

**Cohort ledger:** [x] Issue added to cohort ledger

### Issue selection reasoning

I asked for a Tier 1 issue that I could verify with one repeatable test. The test lives in `tests/unit/test_prompt_templates.py`, and its inputs are the five strings in `rag/generator/prompt_templates.py`. I can see the exact wording that `get_template()` gives the review generator. This makes the scope manageable for me without a model call, a database fixture, or a judgment about the quality of generated feedback. Even a one-word edit can change what the model is asked to do, so the snapshot needs to cover the full text of each released version.

Before the claim, the GitHub issue was open with no comments, assignees, or related PRs, and the Howard catalog showed zero claims. My account posted the claim, and the ledger now lists my username, issue, and tier in row 32. [Claim comment](https://github.com/codepath/pathreview-ai201-fa26-howard/issues/12#issuecomment-7192135133) · [Howard ledger](https://docs.google.com/spreadsheets/d/1-39i16vRXrVwGKUHwxWfg2w_UAgsmxUYaR1oJ2djH8w/edit#gid=739942384&range=A32:H32)

### Environment setup and verification

I used one fork and one branch from upstream `main` at `14578ac9708dcbd94c3258430baddeae621c7e40`. Codex installed the Python 3.12 development dependencies in a local virtual environment and the frontend packages from the lockfile. Matching NumPy to upstream CI's 2.4.6 resolved a type-check error. On the unchanged source, `make check` passed, `make test-unit` reported 375 passed and 53 expected failures, and all 18 frontend tests passed.

Codex verified the rendered login page at `http://localhost:5173/login` in Brave. At this check my Mac had less than the 20 GB of free space listed in the setup guide. The Docker services, database migrations, and seeded login still need verification. [Setup guide](https://github.com/codepath/pathreview-ai201-fa26-howard/blob/main/docs/SETUP.md)

### My decisions and AI-assisted workflow

I delegated the technical issue selection and asked for a focused contribution that I could check without paid services. Codex compared the tracker with the ledger, traced the existing test, and ran the checks above. The course reading links redirected to CodePath sign-in, so the selection used my supplied checklist and the repository's contribution guide. My visual confirmation and personal scope review are pending.
