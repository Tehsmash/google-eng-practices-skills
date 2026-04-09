---
name: google-cl-description
description: >
  Helps write high-quality commit messages and pull request (PR) descriptions
  that communicate both what changed and why. Based on Google's engineering best
  practices for change descriptions. Use this skill whenever the user wants to
  write, draft, or improve a commit message, PR description, merge request (MR)
  description, or any description of a code change — even if they phrase it
  casually, e.g. "help me write a commit", "what should my PR say", "write me a
  commit message", "I need a PR description", "can you describe these changes",
  or "I'm about to push my changes". Trigger this skill even for simple requests;
  a good description is always worth getting right.
---

# Writing Good Commit & PR Descriptions

A commit message or PR description is a permanent public record of a change. It
serves two audiences: reviewers right now, and engineers years from now who'll
search version history trying to understand why the codebase looks the way it
does.

A description that only says *what* changed is half a description. The *why* is
often more valuable. Source code can reveal what the software does; it rarely
reveals the reasoning, constraints, or trade-offs that led to a decision. That
context lives in the commit message or PR description — or it's lost forever.

## Your workflow

1. **Look at the actual diff.** Run `git diff --staged` (or examine the changes)
   and read what's actually changing. Don't write from memory.

2. **Check scope.** Does the change do one thing, or multiple unrelated things?
   - If the user is still *planning*: recommend splitting before they start —
     it's much easier then.
   - If the changes are *already staged or done*: note the mixed concerns, but
     still produce the best possible description for what's there. A commit
     message that honestly describes a broad change is better than no message
     at all. You can also mention `git add -p` or `git stash` if splitting is
     feasible with modest effort.

3. **Draft the first line.** One short imperative sentence: what does this change
   do? Write it as though giving an order: "Remove the FizzBuzz size limit" not
   "Removing the FizzBuzz size limit."

4. **Draft the body.** Answer: why is this change being made? What problem does
   it solve? Were there decisions or trade-offs that aren't visible in the code?
   Is this part of a larger effort?

5. **Review before finalising.** If the change evolved during review or
   implementation, make sure the description still matches what's actually there.

---

## The first line

The first line appears alone in `git log --oneline`, email subjects, and PR
dashboards. It must stand alone.

- **Short** — aim for under ~72 characters
- **Imperative form** — "Fix", "Add", "Remove", "Refactor" — not "Fixed", "Adding"
- **Specific** — describes *this* change, not a vague category
- **Followed by a blank line** before the body

**Bad first lines** (too vague to be useful in history):
- "Fix bug"
- "Fix build"
- "Add patch"
- "Phase 1"
- "Moving code from A to B"
- "Add convenience functions"

**Good first lines** (specific, actionable):
- "Remove size limit on RPC server message freelist"
- "Construct Task with TimeKeeper to use its TimeStr and Now methods"
- "Create Python3 build rule for status.py"

---

## The body

The body should answer: *why does this change exist?* Code tells readers what
the software does; the description tells them why it was changed. Ask yourself:
if a colleague asked "why did we do this?", what would you say?

Things worth including in the body:
- The problem being solved or the goal being achieved
- Why this approach was chosen over alternatives
- Any known shortcomings or planned follow-up
- Links to bug trackers, design docs, or benchmarks — with enough inline context
  that the description still makes sense if the link goes dead
- If this is one step in a larger effort, say so

Even small changes benefit from a sentence or two of context. Put the change
in context.

---

## Good description examples

**Functionality change:**
> Remove size limit on RPC server message freelist.
>
> Servers like FizzBuzz have very large messages and would benefit from reuse.
> Make the freelist larger, and add a goroutine that frees the freelist entries
> slowly over time, so that idle servers eventually release all freelist entries.

*The first line is specific. The body explains the problem, the solution, and a
non-obvious implementation detail (the goroutine).*

---

**Refactoring:**
> Construct a Task with a TimeKeeper to use its TimeStr and Now methods.
>
> Add a Now method to Task, so the borglet() getter method can be removed (which
> was only used by OOMCandidate to call borglet's Now method). This replaces the
> methods on Borglet that delegate to a TimeKeeper.
>
> Allowing Tasks to supply Now is a step toward eliminating the dependency on
> Borglet. Eventually, collaborators that depend on getting Now from the Task
> should be changed to use a TimeKeeper directly, but this has been an
> accommodation to refactoring in small steps.
>
> Continuing the long-range goal of refactoring the Borglet Hierarchy.

*Acknowledges this is an intermediate step, explains the larger direction, and
is honest that the solution isn't ideal but is intentional.*

---

**Small change needing context:**
> Create a Python3 build rule for status.py.
>
> This allows consumers already using Python3 to depend on a rule next to the
> original status build rule. It encourages new consumers to use Python3 instead
> of Python2, and significantly simplifies some automated build file refactoring
> tools currently in progress.

*One file changed, but the body explains its significance within the larger
ecosystem.*

---

## Keeping changes focused

A focused change is easier to review, easier to revert, and easier to describe.
The ideal is **one self-contained change** per commit or PR: one part of a
feature, one bug fix, one refactoring step.

Signs a change spans multiple concerns:
- The first line needs "and" to capture everything it does
- Refactoring is mixed with behaviour changes
- Unrelated files are modified for unrelated reasons

**If the work isn't done yet**: splitting up front is almost always the right
call. Strategies:
- **Stack commits/PRs** — submit a foundation change first, build the next on top
- **Split by layer or file group** — e.g. schema changes separate from logic changes
- **Extract refactoring first** — a preparatory cleanup makes the real change
  smaller and cleaner

**If the work is already staged**: splitting is still an option (`git add -p`
can partition changes by hunk, or parts can be stashed into a separate branch),
but it's more work. If the user wants to commit as-is, write the best possible
description — be explicit in the body that the change covers multiple concerns,
and explain each one's purpose. A clear, honest description of a broad change
is far better than a vague one.

Keep related test code in the same commit as the logic it tests. Independent
test additions (adding coverage to existing code) can go in a separate commit.
