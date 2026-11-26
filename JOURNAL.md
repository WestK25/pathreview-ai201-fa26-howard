## Unit 7 — Issue selection

**Issue link:** [Howard PathReview issue #12](https://github.com/codepath/pathreview-ai201-fa26-howard/issues/12)

**Issue title:** Add snapshot tests for prompt templates to catch accidental changes

**Tier:** [x] Tier 1  [ ] Tier 2  [ ] Tier 3

**Problem summary:** PathReview keeps five versioned prompts that tell the model how to review a portfolio. Its snapshot test hashes all five prompts together and only checks that the result has 32 characters. A change from “with specific examples” to “without specific examples” still passed all 37 prompt tests under the same version. I want the tests to identify which released prompt changed and let a contributor add a new version with its own snapshot.

**Branch name:** `test/12-versioned-prompt-snapshots`

**Setup confirmation:** [x] App runs locally at localhost:5173

**Cohort ledger:** [x] Issue added to cohort ledger

### Issue selection reasoning

I asked for a Tier 1 issue that I could verify with one repeatable test. The test lives in `tests/unit/test_prompt_templates.py`, and its inputs are the five strings in `rag/generator/prompt_templates.py`. I can see the exact wording that `get_template()` gives the review generator. This makes the scope manageable for me without a model call, a database fixture, or a judgment about the quality of generated feedback. Even a one-word edit can change what the model is asked to do, so the snapshot needs to cover the full text of each released version.

Before the claim, the GitHub issue was open with no comments, assignees, or related PRs, and the Howard catalog showed zero claims. My account posted the claim, and the ledger now lists my username, issue, and tier in row 32. [Claim comment](https://github.com/codepath/pathreview-ai201-fa26-howard/issues/12#issuecomment-7192135133) · [Howard ledger](https://docs.google.com/spreadsheets/d/1-39i16vRXrVwGKUHwxWfg2w_UAgsmxUYaR1oJ2djH8w/edit#gid=739942384&range=A32:H32)

### Environment setup and verification

I used one fork and one branch from upstream `main` at `14578ac9708dcbd94c3258430baddeae621c7e40`. Codex installed the Python 3.12 development dependencies in a local virtual environment and the frontend packages from the lockfile. Matching NumPy to upstream CI's 2.4.6 resolved a type-check error. On the unchanged source, `make check` passed, `make test-unit` reported 375 passed and 53 expected failures, and all 18 frontend tests passed.

Codex verified the rendered login page at `http://localhost:5173/login` in Brave, but Docker was not installed at that point. My Mac had about 11 GB free, below the 20 GB in the [setup guide](https://github.com/codepath/pathreview-ai201-fa26-howard/blob/main/docs/SETUP.md), so Claude installed Colima and the Docker CLI through Homebrew instead of Docker Desktop. `docker-compose up -d` brought PostgreSQL and Redis up healthy, `make migrate` applied migrations 001 and 002, and `make seed` created the three test accounts. With `make run` going, `localhost:5173` served the PathReview app, and logging in as `user1@example.com` through the frontend's `/api` proxy returned 200 (a wrong password returned 401).

Two problems outside my issue showed up. The pinned `chromadb/chroma:0.4.22` container exits with a NumPy 2 error, and `/health` reports PostgreSQL and Redis as unhealthy even though login works, which matches seeded issues #61 and #62. I left both alone.

### My decisions and AI-assisted workflow

I delegated the technical issue selection and asked for a Tier 1 issue that I could test deterministically without paid services. I also required one repository and branch for all four units, first-person documentation, passing local checks, and verified CI. Codex compared the tracker with the ledger, traced the existing test, posted my claim, and ran the baseline checks. The course reading links redirected to CodePath sign-in, so the selection used my supplied checklist and the repository's contribution guide. When Codex reached its usage limit after Unit 8, I switched to Claude Code on the same branch. Claude finished the Docker setup and confirmed my ledger row. After that, I opened `localhost:5173`, logged in with a seeded test account, and saw the dashboard load with three reviews (two completed and one failed). I only checked startup and login, not every feature. I also reviewed the scope and agreed to keep this a test-only change.

## Unit 8 — Reproduction & solution planning

**Reproduction commit link:** [Pre-fix evidence for #12](https://github.com/WestK25/pathreview-ai201-fa26-howard/commit/f24ff8632c90cf0a518024f9ea8a826b0c6301b1)

**Reproduction summary:** Reversing an instruction in `skills_feedback/v1` in memory left all 37 prompt tests passing. The snapshot assertion accepted every 32-character MD5 digest.

**PLAN.md link:** [My solution plan on the working branch](https://github.com/WestK25/pathreview-ai201-fa26-howard/blob/test/12-versioned-prompt-snapshots/PLAN.md)

**Walkthrough video (recommended):** Not recorded.

**Blockers or open questions:** I can reproduce and test this issue entirely in memory. Full local startup is still waiting for the disk space required by the setup guide. (Update during Unit 9: Colima got PostgreSQL, Redis, migrations, and login working, as described in Unit 7.)

### Reproduction evidence

On the issue branch, before any change to the tests or prompts, `make check` passed and `make test-unit` reported 375 passed and 53 expected failures. Running `.venv/bin/pytest -q tests/unit/test_prompt_templates.py` on the unchanged registry gave 37 passed. To isolate the wording check, Codex ran the following command from the repository root using the same virtual environment.

```bash
.venv/bin/python - <<'PY'
import pytest
from rag.generator.prompt_templates import PROMPT_TEMPLATES

versions = PROMPT_TEMPLATES["skills_feedback"]
original = versions["v1"]
edited = original.replace("with specific examples", "without specific examples")
assert edited != original
versions["v1"] = edited
raise SystemExit(pytest.main(["-q", "tests/unit/test_prompt_templates.py"]))
PY
```

All 37 tests passed again (exit 0) after the instruction changed meaning without a version bump. The edit lived only in that Python process. I expected a failure naming `skills_feedback/v1`. The old `test_template_snapshot_content_hash` checked the type and length of an MD5 digest computed during the test, so it had no saved text or digest to compare. Issue #12 has no seeded `xfail`.

## Unit 9 — Solution building & PR submission

### Check-in 1 (mid-week)

**Current progress:** I finished PLAN.md sub-tasks 1–3 in `tests/unit/test_prompt_templates.py` ([commit 9765c15](https://github.com/WestK25/pathreview-ai201-fa26-howard/commit/9765c158c7faf7502a6b936992b249a072e28529)). A module-level `TEMPLATE_SNAPSHOTS` dict records a SHA-256 digest for each of the five `v1` prompts. `test_template_snapshot_content_hash` is now parameterized over those pairs and compares the raw `get_template()` text, and the new `test_every_template_version_has_a_snapshot` compares the registry's `(name, version)` keys with the recorded ones. Codex drafted the digest comparison before reaching its limit, and Claude added the inventory test.

For sub-task 4, Claude reran the in-memory edits from my plan. Rewording `skills_feedback/v1` failed only `[skills_feedback-v1]`, and adding one newline to `first_impression/v1` failed only `[first_impression-v1]`. Adding an unrecorded `skills_feedback/v2` failed only the inventory test, and once v2 had its own digest both versions passed. Removing `gaps_feedback/v1` failed the inventory test and that snapshot, along with six older `gaps_feedback` checks. On the unchanged registry, the prompt test file has 42 passing tests, `make check` passes, and `make test-unit` reports 380 passed and 53 xfailed.

**Next steps:** Review the diff myself, open a draft PR against upstream `main`, confirm all five CI jobs, and ask for feedback in the course Slack channel.

**Blockers:** None at this stage.

---

### Check-in 2 (end of week)

**PR link:** [codepath/pathreview-ai201-fa26-howard#116](https://github.com/codepath/pathreview-ai201-fa26-howard/pull/116)

**Branch:** `test/12-versioned-prompt-snapshots`

**What you built:** I replaced a prompt "snapshot" test that only checked the length of an MD5 digest with a recorded SHA-256 snapshot for each released `(name, version)` prompt. Now rewording `skills_feedback/v1`, or even adding a newline to it, fails a test that names that prompt. A second test requires every registered version to have exactly one snapshot, so an intentional prompt change has to go in as a new version with its own digest.

**Tests added or updated:** All changes are in `tests/unit/test_prompt_templates.py`.
- `test_template_snapshot_content_hash` is now parameterized over the five pairs in `TEMPLATE_SNAPSHOTS` (`first_impression`, `gaps_feedback`, `presentation_feedback`, `projects_feedback`, and `skills_feedback`, all `v1`). It hashes the raw `get_template()` text before any portfolio data is filled in, so wording and whitespace changes both count.
- `test_every_template_version_has_a_snapshot` is new. It fails when a version has no snapshot or a snapshot has no version, and lists the pairs involved.
- Edge cases checked by editing the registry in memory: a reworded prompt and a trailing newline each failed only their own pair; an unrecorded `v2` failed only the inventory test and passed once its digest was added; and removing a released `v1` failed the inventory test.
- Issue #12 is not a seeded bug, so I had no `xfail` marker to remove. All 53 existing `xfail` tests are unchanged.

**Self-review confirmation:** [x] make check passes  [x] make test-unit passes

On the final code commit, `make check` passed (ruff, black on 110 files, and mypy on 76 source files), and `make test-unit` reported 380 passed and 53 xfailed (375 passed on `main`). `make test-integration` collected no tests, which is expected because `tests/integration/` is empty. Locally, vitest under Node 24 timed out loading the jsdom setup file, so I relied on CI for the frontend. All five CI jobs passed on the PR: `lint`, `typecheck`, `test-unit` (380 passed, 53 xfailed), `test-integration`, and `frontend` (18 passed). I reviewed the approach and kept the SHA-256 snapshots and version inventory check. I used ChatGPT to add the PR link to my ledger row, and Claude read the sheet back to confirm it.

**Draft PR feedback received from:** none. I asked for review in the course Slack channel after opening the draft PR. As of November 26, 2026, no one had replied there, and the PR had no GitHub reviews or comments.
