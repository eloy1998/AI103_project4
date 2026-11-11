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

## Understanding the Issue

**Problem Description**

The linux-vitals collection reports all containers as "Virtual Machine" because it treats any system with `virtualization_role: guest` as a VM, without distinguishing between actual VMs and containers (which also report as guests with a `virtualization_type` like docker, podman, lxc, containerd, etc.).

**Expected Behavior**

The system should distinguish containers from VMs. Containers should be clearly identified as containers, not grouped with virtual machines.

**Current Behavior**

Containers are misclassified as "Virtual Machine" in three places:
- Dashboard's Type column
- JSON report's `platform.type` field
- Generic webhook's `platform_type` field

**Affected Components**

- `roles/vitals_scan/tasks/discovery.yml:715-721` - the `linux_vitals_platform_type` mapping that determines platform classification

**Toolkit**

- Tool:
- What it produced:
- Difficulties:
- What changed:

---

## Reproduction Process

**Environment Setup**

[What a stranger would need to stand this up: your operating system,
the relevant versions (language, runtime, package manager, the project
itself), and how you set the project up. Then the challenges you faced
and how you solved them, or that none came up.]

**Steps to Reproduce**

1. [Step 1]
2. [Step 2]
3. [Observed result]

**Reproduction Evidence**

- **Commit showing reproduction:** [link to commit in your fork]
- **Screenshots/logs:** [if applicable]
- **My findings:** [what you discovered during reproduction; an
  honest, evidenced cannot-reproduce that led you to re-pick or ask
  the maintainer belongs here and costs you nothing]

**Toolkit**

- Tool:
- What it produced:
- Difficulties:
- What changed:

---

## Solution Approach

**Analysis**

The root cause is in `roles/vitals_scan/tasks/discovery.yml` lines 715-721 where `linux_vitals_platform_type` maps any `virtualization_role: guest` to "Virtual Machine". The code doesn't check the `virtualization_type` variable to distinguish between containers (docker, podman, lxc, containerd, systemd-nspawn, openvz, container) and actual VMs. We need to add logic to detect container types and set a new `platform.container` flag.

**Proposed Solution**

Add a new `platform.container` boolean field (true for containers, false for VMs/physical hosts) to:
- The JSON report's `platform` object
- The generic webhook payload (new `platform_container` field)
- The Ansible fact `linux_vitals_result`

This is additive-only, so `platform.type` remains unchanged, maintaining backward compatibility. The dashboard can display container information since its layout isn't part of the public interface. The `schema_version` will be incremented per compatibility.md standards.

**Implementation Plan**

1. Modify `roles/vitals_scan/tasks/discovery.yml` to add container detection logic that checks if `virtualization_type` is one of the known container types
2. Update the JSON report template to include the new `platform.container` field
3. Update the generic webhook template to include `platform_container`
4. Update documentation in `docs/report.md` (or similar) to describe the new field
5. Update/add Molecule test assertions to verify `platform.container: true`
6. Test with Molecule scenarios to ensure containers are correctly identified

**Toolkit**

- Tool:
- What it produced:
- Difficulties:
- What changed:

---

## Testing Strategy

[What observable checks prove the fix: your reproduction steps re-run
against the built change (before and after), and the repo's own test
suite or checks with the outcome shown. "Tests pass" with nothing
observable named is not a strategy.]

**Toolkit**

- Tool:
- What it produced:
- Difficulties:
- What changed:

---

## Implementation Notes

[A running log, added to as you work. Date each entry. What you
built, decisions you made when you made them, wrong turns included.]

**[Date]**

[Entry]

**[Date]**

[Entry]

**Toolkit**

- Tool:
- What it produced:
- Difficulties:
- What changed:

---

## Pull Request

**PR Link:** [GitHub PR URL when opened]

**PR Description:** [what the PR says; much of the content above
adapts into it]

**Maintainer Feedback:**

- [Date]: [feedback received, or "none yet"; silence is a normal
  state and costs you nothing]
- [Date]: [how you addressed it]

**Status:** [Awaiting review / Iterating / Merged / Closed. The
status is a fact you record, never a grade: the PR is graded as
opened.]

**Toolkit**

- Tool:
- What it produced:
- Difficulties:
- What changed:

---

## Learnings and Reflections

**Technical Skills Gained**

[What you learned technically, named concretely.]

**Challenges Overcome**

[What was hard and how you solved it; point at the real events, the
Toolkit blocks above are full of them.]

**What I'd Do Differently Next Time**

[Reflection on your process.]

---

## Resources Used

[What actually helped, not a reading list. List things this story
already shows you using: the doc you followed to set the environment
up, the thread that explained the bug, the person who answered you.
One real entry beats ten plausible ones.]

- [Link to documentation that helped]
- [Discussion, issue, or post that helped]
