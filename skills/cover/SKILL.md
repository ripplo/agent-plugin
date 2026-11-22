---
name: cover
description: "Add what Ripplo could not cover. Use when the user runs `ripplo:cover`, alone, with a code review id, with `--gap` ids, or with a list of workflows, or asks to make an uncoverable workflow testable. Reads each coverage gap, adds the API or route the app lacks so Ripplo can set up and observe that state, and reports per gap."
---

# Cover the workflows a Ripplo review could not

Input: nothing, a code review id, one or more `--gap <id>` arguments, or one line per workflow pasted after the command (`- <workflow>: <what Ripplo cannot do>`). Output: the app changed in the working tree so Ripplo can create and observe the state each coverage gap names, one short report per gap.

Two words matter here. An uncoverable workflow is a user journey Ripplo cannot test yet. A coverage gap is one thing the app does not give Ripplo, and it is what holds uncoverable workflows back. Several workflows can share one gap, and one workflow can have several. One kind of gap is different: a `ripploLimitation` gap is something Ripplo itself cannot do or judge yet. No change to the app removes it, so you never work on one.

With nothing after the command, the work is every coverage gap of the project connected to this repository. With an id, it is the gaps of the workflows that review could not cover. With `--gap` arguments, it is exactly those gaps and nothing else. All three come from the CLI. Never query Ripplo any other way. With pasted lines, each line is a workflow with what Ripplo cannot do, and there is nothing to fetch. When the input ends with a `Server: <url>` line, it is not a gap: prefix every `npx ripplo@latest` command with `RIPPLO_SERVER_URL=<url>`.

```sh
npx ripplo@latest gaps                    # every coverage gap of the project, each with its workflows (JSON)
npx ripplo@latest gaps --gap <id>         # only this gap (JSON), repeat --gap for several
npx ripplo@latest review <codeReviewId>   # issues and uncoverable workflows of the published attempt (JSON)
```

`npx ripplo@latest gaps` prints a `gaps` array. Each gap has a `slug`, a `kind` (`permission`, `missingApi`, `externalSystem`, `ripploLimitation`, or `other`), a one-sentence `description`, and `workflows`, the uncoverable workflows it holds back. Each workflow has `workflow`, `behavior` (what Ripplo could not check), and `otherGaps`, the gaps it still has besides this one. With `--gap` arguments, pass them to `npx ripplo@latest gaps` unchanged, and remove only the gaps you were given. Do not remove a gap listed under `otherGaps` unless it was also given.

The `gaps` array holds only gaps an app change can remove. A `ripploLimitation` gap appears only under a workflow's `otherGaps`, and `--gap` refuses one. Each workflow has `appChangeCanCover`, which is false when one of its gaps is a `ripploLimitation`. Such a workflow stays uncoverable whatever you remove.

`npx ripplo@latest review <codeReviewId>` prints an `uncoverable` array. Each entry has `workflow`, `behavior`, `appChangeCanCover`, and its `gaps`. Work only on the gaps of entries whose `appChangeCanCover` is true, and skip every gap whose `kind` is `ripploLimitation`.

When `gaps` says several projects are connected to the repository, ask the user which one and pass it with `--project <id>`. Sign in first if the command says so: `npx ripplo@latest login`.

## What a coverage gap means

Ripplo tests through the browser as a signed-in run user. Before a test it sets up the starting state and reads it back through the app's own APIs, using the run's browser session. When the app has no way to create or observe some state, Ripplo records the workflow as uncoverable and names the coverage gap instead of inventing state. Ripplo never edits application code, so the gap stays until the app gains the capability.

## Loop, per gap

One change to the app removes a gap for every workflow it holds back, so start with the gaps that hold back the most workflows. A workflow is coverable once every gap it has is removed.

1. **Read the gap's `description` and `kind`.** The description says what Ripplo cannot create or observe. It never says how to fix it. `permission` means a way exists and the run user may not use it. `missingApi` means the app has no way to do it. `externalSystem` means the state lives outside the app, so the fix is usually a test fake or a guarded route that stands in for that system.
2. **Look for an existing API.** Find the app code that creates or reads that state. If an API already does it and the run user can call it with their own session, report that and change nothing. If it exists only behind a role the run user cannot hold, check whether the run user should have that permission.
3. **Otherwise add the smallest API that does it.** A route that creates the row or state the gap names, or one that reads it back. Require a signed-in user with permission to perform the operation.
4. **Keep it application code.** Test it like the rest of the app. Do not write `.ripplo/` files, bindings, or state schemas. The next Ripplo review of the pull request writes those and the workflow for each uncoverable workflow.

Gaps are independent. Fan out to subagents when there are many, one gap or one shared file cluster per agent, each told to load the `ripplo:cover` skill first.

## Finish

Report per gap: already possible, access extended, or added, naming the route or API and the workflows it held back. Then report per workflow whether every gap it has is now removed. When you were given `--gap` arguments, a workflow with an empty `otherGaps`, or whose other gaps were all given too, is now coverable. Name each workflow that is still held back and the gaps it still needs. A workflow whose `appChangeCanCover` is false is never now coverable: report it as one Ripplo cannot cover yet, tell the user to contact Ripplo at mail@ripplo.ai about it, and leave it out of the pull request section below.

This skill changes the application only. It does not write Ripplo tests. The next Ripplo review of the pull request writes the tests, and it reads the pull request description to learn which workflows it can now cover. Add this section to the description so the review knows where to go:

```md
## Ripplo coverage

Now coverable:

- <workflow>: added `POST /api/ripplo/invoices`, which creates an invoice for the run's organization.
- <workflow>: already possible, `GET /api/orders/:id` returns the refund state.

Still held back:

- <workflow>: needs <description of each gap it still has>.
```

List a workflow under "Now coverable" only when every gap it has is removed, with the `workflow` exactly as the CLI gives it and the route or API that removes each gap. Include workflows that were already possible, because no code change points the review at them. List every other workflow you touched under "Still held back" with the gaps it still needs, so the review leaves it uncoverable and no one expects a test for it yet. Leave a heading out when its list is empty.

When the branch has no pull request yet, give the user the section to paste when they open one.

Tell the user to push. Once the push is on the pull request, offer to run `npx ripplo@latest review start <prUrl> --depth deep`, with the `prUrl` from `npx ripplo@latest review <codeReviewId>`, and wait for it as the `ripplo:review` skill describes. The review covers each listed workflow, or keeps it uncoverable and names the gaps that still hold it back.
