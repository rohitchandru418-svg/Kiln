# OpenHands Playbook: building Kiln Lite

## Part A: One-time setup (about 1 hour, you)

1. **Create the repo** (private) and copy this pack into its root. Commit as `chore: add build pack`.
2. **Add the PRD.** Export the Kiln PRD from Claude as Markdown (or Word, then convert) and save as `docs/prd/kiln-prd.md`. Then split by module so agents load only what they need:
   `docs/prd/m1-discovery.md`, `m2-listings-manifest.md`, `m3-playground.md`, `m4-fit-engine.md`, `m6-passport.md`, `m7-creator.md`, `m11-trust.md`, plus `appendix-a-scoring.md`. Keep the full file too.
3. **Create the issues:** `gh auth login`, then `scripts/create-issues.sh`. Create a GitHub Project and add the issues; order by the `Depends on` lists (suggested order is in `docs/backlog/kiln-lite-backlog.md`).
4. **Turn on branch protection** for `main`: require PR + passing CI, no direct pushes.
5. **Handle the human items** in `docs/architecture/decisions.md` (H1 to H7): hosting and domains, GitHub OAuth app, invited creators list, licence allow-list sign-off, product/npm name, design direction.
6. **Pick the model and budget.** Use the strongest coding model you can afford for E5, E6 and E7 (sandbox, Passport, codemods); a cheaper model is fine for E0 to E4 and E8. Set a spending cap per task.

## Part B: The loop (repeat for every issue)

1. Pick the next issue whose dependencies are merged.
2. Start a **fresh OpenHands conversation** for that issue (new session = fresh context; it also reloads AGENTS.md and the skills catalog).
3. Give it this prompt (replace the number):

   ```
   Implement GitHub issue #<N> in this repository.
   First read AGENTS.md, the issue, and every spec the issue lists under "Read first".
   Load the matching skill from .agents/skills/ if one applies.
   Follow the acceptance criteria exactly; do not add features outside the issue.
   Write the tests first or alongside. Run `pnpm check` (and integration/e2e where relevant) until green.
   If anything is ambiguous, comment "Open question:" on the issue with your recommended default and proceed with the simplest reversible choice.
   Open a pull request using the template and fill in the evidence table.
   ```
4. **Review the PR yourself** (or a teammate). Check: each acceptance criterion has real evidence; tests would fail if the feature were broken; no scope creep; no secrets; migrations match `schema.sql`.
5. Merge only when CI is green. Then start the next issue.
6. If the agent goes in circles (more than about 2 attempts on the same failure), stop, read its last output, tighten the issue (add the missing detail), and restart in a new session. Do not keep adding instructions to a long, confused session.

## Part C: Order of work

| Step | Epics | Notes |
|---|---|---|
| 1 | E0 Foundations | Do KL-001 to KL-003 first, and review these very closely: everything else builds on them |
| 2 | E1 Auth, E2 Manifest and publishing, E3 Registry | KL-020 and KL-030 unlock publishing |
| 3 | E4 Catalogue and web | Needs design direction (H7) before KL-042 and KL-043 |
| 4 | E5 Playground and E6 Passport | Can run in parallel; both execute untrusted code, so review hardest here |
| 5 | E7 CLI install | KL-070 and KL-072 are the heart of the product; build the benchmark (KL-076) early and watch the success rate |
| 6 | E8 and E9 | Admin, telemetry, security pack, load tests, docs, seed components, launch checklist |

Parallelism: two or three agent sessions at once is practical if their issues touch different packages; avoid parallel work on the same package (merge conflicts).

## Part D: Safety rules for you (important)

- **Do not publish publicly until** KL-054 (sandbox escape tests) passes, KL-090 is done, and the third-party review (H5) is complete. Lite is invite-only until then.
- Never give the agent production credentials. Use local Docker, test-mode keys, and a staging environment.
- Review anything touching authentication, tokens, tar extraction, the Passport runner, the preview origin, or signing keys line by line.
- Run the agent in an isolated environment (container or VM), not on a machine with personal data or cloud admin access.

## Part E: What this pack does NOT cover (next documents to write)

- Visual design: Figma or screen specs for KL-042 and KL-043 (H7)
- Legal text: licence terms, creator terms, privacy policy, takedown policy
- Phase 2 specs: Fit Engine AI repair loop, payments and payouts, MCP server, AI Studio, private registry

## Part F: Quick sanity checks after the first week

- `pnpm check` passes from a clean clone, in CI, in under 10 minutes.
- Preview origin test proves cookies and storage are isolated.
- A fixture component can be published, scored by Passport, listed, and installed with `kiln add` on a fixture repo.
- The Fit benchmark (KL-076) runs and records a first-pass success rate, even if low. Improve it with codemods, not with prompts.
