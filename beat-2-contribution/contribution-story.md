# Contribution: Containers are reported as "Virtual Machine"

Path: `beat-2-contribution/contribution-story.md`

This file is Assignment 5, and it is graded at the path above; a copy kept anywhere else in
the repository is not read. **Keep every section header below under its exact name** — the
format criteria grade exactly that. Your course repo link is the whole submission: no tool
files and no evidence uploads are part of Assignment 5.

**Contribution Number:** [1]
**Student:** Eloy
**GitHub Username:** eloy1998
**Issue:** https://github.com/sameeralam3127/linux-vitals/issues/123
**Pull Request:** [GitHub PR link, once opened]

One repository, built across units 5 to 8, telling the whole story of your real
contribution. Fill it as you work, not at the end — the sections below ask for evidence that
is far easier to write down as it happens than to reconstruct in the last weekend.

Contribution Number stays `[1]` unless you make more than one contribution this term; it
just numbers them.

---

## Toolkit

Every story section below, from Why I Chose This Issue through Pull
Request, ends with a Toolkit block: which tool did that phase's work
and how you used it, what it produced, what difficulties or
shortcomings it presented, and what you changed or would change
because of them. Together the blocks are the thread the rubric's
Toolkit points read, and they are the raw material for your unit-9
portfolio, so keep them current as you work rather than
reconstructing them later.

A Toolkit block looks like this (copy it into each story section and
fill it there):

```text
**Toolkit**

- Tool: [which tool did this phase's work, and how you used it]
- What it produced: [described in a sentence or a short excerpt]
- Difficulties: [what fought you or fell short, or "none met"]
- What changed: [the patch, revision, or workaround, what you would
  update, or "nothing"]
```

This section also keeps one short dated entry per week of tool work,
starting below: what you built or changed that week, what broke, and
what you changed because of it. The blocks tell the story where it
happened; these entries are the toolkit's own timeline.

All four entries are graded. Keep every heading below, and fill each
one in the week it belongs to.

**Unit 5: issue-scout, a GitHub-wide scout run from my own search strategy and rubric**

I built a scout that runs three `gh search issues` queries from my fit profile, ranks the results with free filters, and grades the top five against my rubric. The first scans failed: fewest-comments-first ranked brand-new zero-comment issues from one fresh repo, and the rubric accepted a closed team repo and a task needing a paid GPU pod. I moved an age floor into the queries, added a per-repo cap and a comment floor, and tightened the rubric on repo age, issue size, competing PRs and closed repos, which made a later scan reject the bad five and accept linux-vitals #123. What still fell short is that the comment floor surfaces issues whose one comment is a claim, so I plan to drop claimed issues next.

**Unit 6: voice-guard and repo-memory file for claiming issues and building context**

I set up the voice guard with Upstream rules for claims, questions, and pings to maintainers, then installed hooks to check drafts before posting. I also copied the repo-memory template to the linux-vitals fork and filled in the repo's key facts, file structure, and test commands so Claude loads this context at the start of each session without me re-explaining the repo. The voice guard caught me stating a cause as fact instead of hypothesis in my first claim draft, which forced me to rewrite it to name the observed behavior and promise an investigation instead. Nothing broke in the memory setup—it worked immediately once I added the file to the fork—but I discovered that the hook path resolution matters: the voice guard looks for file paths relative to the original working directory, not the directory I change into mid-command.

**Unit 7: [one line on the tool work]**

[2-4 sentences: the ship-gate and your gate manifest. What it produced
on your real branch, what fought you, what changed because of it.]

**Unit 8: [one line on the tool work]**

[2-4 sentences: the PR-monitor and your playbook. What it read, what it
drafted, what you did with the draft.]

---

## Why I Chose This Issue

I ran my scout on a search for Ansible-related issues in active repos with good-first-issue labels, which surfaced linux-vitals #123 among several candidates. The issue is straightforward to understand: containers are misclassified as VMs because the platform type detection only checks `virtualization_role: guest` without examining `virtualization_type` to distinguish container types. The maintainer's response in the issue comments is detailed and welcoming, providing clear guidance on where the code lives, what tests to write, and how to run them. This issue fits my profile because it's a self-contained logic fix in Ansible YAML with a clear test pattern to follow, and the repository is actively maintained and explicitly open to newcomer contributions.

I chose this issue partly because my scout accepted it and partly because the maintainer guidance significantly reduced uncertainty about the first steps. The issue also matches what I learned from Unit 5: scope-feasible issues often have open edges (here, I need to understand exactly how Ansible virtualization_type is set in containers), and the scout's difficulty note helps me choose with eyes open. My Unit 6 toolkit entry below describes how the voice guard and scope patch shaped my approach to this choice.

**Claim comment:** https://github.com/sameeralam3127/linux-vitals/issues/123#issuecomment-6790258284

**Toolkit**

- Tool: issue-scout (search strategy: Ansible active repos + good-first-issue labels; rubric: repo age, scope-feasible, no competing work, contribution policy)
- What it produced: scanned results ranking linux-vitals among candidates; verdict: accept with straightforward difficulty note (code location identified, change is bounded)
- Difficulties: the rubric's scope check initially rejected wild issues with open edges; patching it to report difficulty instead of rejecting made the scout usable on real repos
- What changed: integrated the scope-feasible check and difficulty-note output, which shifted the scout from a yes/no gate to an informed ranking tool

---

**Toolkit**

- Tool: Linux-vitals codebase reading (GitHub issue #123 and linked maintainer guidance)
- What it produced: understanding of the problem: three output locations report platform type, all read from the same fact, which is set incorrectly for containers
- Difficulties: none met; the issue and maintainer's "Where to start" comment gave clear file locations
- What changed: nothing; the problem statement was clear and scoped


---

## Reproduction Process

**Environment Setup**

Python 3.12, Ansible 2.17.10, development clone from fork on Linux (Codespaces Ubuntu 22.04). Installation follows standard dev setup: `python3 -m venv .venv`, `source .venv/bin/activate`, `pip install -r requirements-dev.txt`, `ansible-galaxy collection install -r requirements.yml`. Environment verified functional: `pytest -q` runs ~200 tests in ~1 minute.

**Steps to Reproduce**

1. Reviewed code in `roles/vitals_scan/tasks/extended_facts.yml` lines 18-25: classification checks `virtualization_role` first and returns "Virtual Machine" if equal to 'guest', without examining `virtualization_type`.
2. Created test script simulating Ansible facts for Docker container (`role=guest, type=docker`), KVM guest (`role=guest, type=kvm`), and physical host (`role=NA, type=NA`).
3. Ran test to confirm Docker container incorrectly classified as "Virtual Machine".

**Reproduction Evidence**

- **Commit showing reproduction:** none (reproduction is via test logic, not filesystem changes)
- **Screenshots/logs:** Test output below shows bug confirmed
- **My findings:** Docker container with `virtualization_role: guest` + `virtualization_type: docker` is currently classified as "Virtual Machine" in the platform type detection logic. The misclassification appears in dashboard, JSON report, and webhook output. Both containers and KVM guests report `virtualization_role: guest`, but containers also provide `virtualization_type: docker|podman|lxc|etc.` which the code does not examine to distinguish them. Hypothesis: examining `virtualization_type` before deciding is the intended fix.

Test output:
```
✓ BUG CONFIRMED: Docker container misclassified as 'Virtual Machine'
  Facts: role=guest, type=docker
  Result: Virtual Machine (should be 'Container')
✓ KVM VMs correctly classified
✓ Physical hosts correctly classified
```

**Toolkit**

- Tool: repro-check (draft verification before posting upstream)
- What it produced: voice guard check on claim and repro drafts, caught a rule violation about stating hypotheses vs. observations
- Difficulties: initially stated root cause as fact ("The Jinja2 logic short-circuits") instead of hypothesis; voice guard stopped the draft
- What changed: rewrote to frame the observation separately from hypothesis ("Looking at the logic... I'll investigate whether examining virtualization_type...")


---

**Toolkit**

- Tool: codebase reading, maintainer guidance from issue comments
- What it produced: mapping of the four files to change (extended_facts.yml, result.yml, report.json.j2, generic_webhook_payload.json.j2) and the flow between them
- Difficulties: none met; maintainer's comment explicitly listed the files and test pattern to follow
- What changed: nothing; the approach was clear from the maintainer's guidance


---

## Testing Strategy

The repo's test suite runs three levels: unit tests (`pytest`), linting and checks (`pre-commit`), and Molecule scenarios on live containers (Ubuntu, Rocky, Fedora, openSUSE).

**For the platform type fix:**
1. Add unit test in `tests/test_discovery_facts.py` following the pattern of `linux_vitals_selinux_status` test, with fixtures for container types (docker, podman, lxc, containerd, openvz).
2. Add golden fixture test: regenerate golden findings with `LINUX_VITALS_UPDATE_GOLDEN=1 pytest tests/test_golden_findings.py` to include container platform type in the expected output.
3. Add template rendering test: verify `report.json.j2` and `generic_webhook_payload.json.j2` render the new field correctly.
4. Run full test suite: `pytest -q`, `pre-commit run --all-files`, `ansible-lint` must all pass.
5. No Molecule changes needed locally: CI runs Molecule scenarios and will verify containers are correctly identified on live containers.

**Observable checks:**
- Unit test passes with container facts classified as "Container" (or specified type)
- Golden fixture diff only adds new field, doesn't change existing platform.type
- Template tests render new field without breaking JSON/webhook structure
- PR CI shows Molecule passing on all four distributions

**Toolkit**

- Tool: CONTRIBUTING.md (development guide)
- What it produced: understanding of test structure (unit → golden → template → Molecule)
- Difficulties: none met; repo provides clear test patterns to follow
- What changed: nothing; test approach is documented

---

## Implementation Notes

**2026-11-11**

Unit 6 complete: voice guard installed with Upstream rules, repo memory file (CLAUDE.local.md) created, claim posted on issue #123. Reproduction confirmed: Docker container with `virtualization_role: guest` + `virtualization_type: docker` is misclassified as "Virtual Machine" in the platform type detection logic at `extended_facts.yml` lines 18-25. Test script demonstrates current behavior short-circuits without checking `virtualization_type` to distinguish containers from VMs. Next: implement fix in four files (extended_facts.yml, result.yml, report.json.j2, generic_webhook_payload.json.j2) and add unit tests.

**Toolkit**

- Tool: test script (`test_container_bug.py`), repro-check voice guard verification
- What it produced: executable reproduction of the bug showing Docker containers incorrectly classified
- Difficulties: voice guard rule about hypothesis vs. observation required rewording the root cause section
- What changed: clarified that observations are separate from hypotheses in repro drafts

---

## Pull Request

**PR Link:** [To be added when opened]

**PR Description:** [Will include reproduction steps, test results showing before/after, and changes to the four affected files once implementation is complete]

**Maintainer Feedback:**

- [Date]: [pending]

**Status:** Not yet started (Unit 6 analysis complete, implementation begins in Unit 7)

**Toolkit**

- Tool: (pending implementation phase)
- What it produced: (pending)
- Difficulties: (pending)
- What changed: (pending)

---

## Learnings and Reflections

**Technical Skills Gained**

Reading Ansible YAML conditionals and Jinja2 templating logic. Understanding how facts flow through role tasks and into templates. Recognizing when a conditional short-circuits and missing the next check.

**Challenges Overcome**

The voice guard catching rule violations in drafts was initially friction but became helpful: stating observations separately from hypotheses forced clarity. Repro-check scales down to small issues once I understood its scope is now for wild repos (not sandbox), and I set it up per Unit 6.

**What I'd Do Differently Next Time**

None yet; Unit 6 focused on tooling and setup. Implementation decisions will come during Unit 7.

---

## Resources Used

- GitHub issue #123 and maintainer's "Where to start" comment: identified exact files and test pattern
- `CONTRIBUTING.md` in `.github/`: development setup, test structure, PR conventions
- `CLAUDE.local.md` template: structured repo memory for future sessions

