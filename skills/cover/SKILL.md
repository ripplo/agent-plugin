---
name: cover
description: "Add what Ripplo could not cover. Use when the user pastes `/ripplo:cover`, alone, with a code review id, or with a list of workflows, or asks to make an uncovered workflow testable. Reads each gap, adds the API or route the app lacks so Ripplo can set up and observe that state, and reports per gap."
---

# Cover the workflows a Ripplo review could not

Input: nothing, a code review id, one or more `--blocker <id>` arguments, or one line per workflow pasted after the command (`- <workflow>: <what Ripplo cannot do>`). Output: the app changed in the working tree so Ripplo can create and observe the state each gap names, one short report per gap.

With nothing after the command, the gaps are every gap of the project connected to this repository. With an id, they are the gaps of that review. With `--blocker` arguments, the work is exactly those blockers and nothing else. All three come from the CLI. Never query Ripplo any other way. With pasted lines, those are the gaps and there is nothing to fetch. When the input ends with a `Server: <url>` line, it is not a gap: prefix every `npx ripplo` command with `RIPPLO_SERVER_URL=<url>`.

```sh
npx ripplo gaps                    # every gap of the project (JSON)
npx ripplo review <codeReviewId>   # issues and gaps of the published attempt (JSON)
npx ripplo gaps --blocker <id>     # one blocker with the workflows it affects (JSON), repeat --blocker for several
```

With `--blocker` arguments, pass them to `npx ripplo gaps` unchanged. Read the `blockers` array: each has `slug`, `kind`, `description`, and `workflows`. Each workflow has `workflow`, `behavior`, and `otherBlockers`, the blockers it still has besides this one. Remove only the blockers you were given. Do not remove a blocker listed under `otherBlockers` unless it was also given. A `ripploLimitation` blocker is never printed here, and asking for one by id fails, because no change to the app removes it. A workflow that lists one under `otherBlockers` stays uncoverable whatever you remove.

When `gaps` says several projects are connected to the repository, ask the user which one and pass it with `--project <id>`. Read the `gaps` array: `workflow` names the user journey, `behavior` says what Ripplo could not check, and `blockers` lists what the app does not give Ripplo. Each blocker has a `slug`, a `kind` (`permission`, `missingApi`, `externalSystem`, `ripploLimitation`, or `other`), and a one-sentence `description`. Several gaps can list the same blocker, and one gap can list several. `appChangeCanCover` is false when a gap lists a `ripploLimitation` blocker: Ripplo cannot cover that workflow yet, and no change to the app makes it coverable. Leave those gaps alone and name them in the report. Pasted lines carry the blocker descriptions as one text after the workflow. Sign in first if the command says so: `npx ripplo login`.

## What a gap means

Ripplo tests through the browser as a signed-in run user. Before a test it sets up the starting state and reads it back through the app's own APIs, using the run's browser session. When the app has no way to create or observe some state, Ripplo records a gap instead of inventing state. Ripplo never edits application code, so the gap stays until the app gains the capability.

Every request a run makes carries a signed `x-ripplo-run` header, also when no user is signed in. A route can accept a Ripplo run by verifying that header with `ripploRunId({ headers, secrets })` from `@ripplo/auth`, using the same secret the app's `/ripplo` handler uses. It returns the run id, or `undefined` when the header is missing, expired, or forged.

## Loop, per blocker

Work only on the gaps whose `appChangeCanCover` is true. Group them by blocker `slug` first. One change to the app removes a blocker for every gap that lists it, so start with the blockers that the most gaps list. A gap is coverable once every blocker it lists is removed.

1. **Read the blocker's `description` and `kind`.** The description says what Ripplo cannot create or observe. It never says how to fix it. `permission` means a way exists and the run user may not use it. `missingApi` means the app has no way to do it. `externalSystem` means the state lives outside the app, so the fix is usually a test fake or a guarded route that stands in for that system.
2. **Look for an existing API.** Find the app code that creates or reads that state. If an API already does it and the run user can call it with their own session, report that and change nothing. If it exists only behind a role the run user cannot hold, allow it for a verified Ripplo run.
3. **Otherwise add the smallest API that does it.** A route that creates the row or state the gap names, or one that reads it back. Guard it with `ripploRunId` and reject when it returns `undefined`. Never mount it unguarded.
4. **Keep it application code.** Test it like the rest of the app. Do not write `.ripplo/` files, bindings, or state schemas. The next Ripplo review of the pull request writes those and the workflow for each gap.

Blockers are independent. Fan out to subagents when there are many, one blocker or one shared file cluster per agent, each told to load `/ripplo:cover` first.

## Finish

Report per blocker: already possible, access extended, or added, naming the route or API and the gaps it was listed on. Then report per gap whether every blocker it lists is now removed. List each gap whose `appChangeCanCover` is false as one Ripplo cannot cover yet, tell the user to contact Ripplo at mail@ripplo.ai about it, and leave it out of the pull request section below. When you were given `--blocker` arguments, a workflow with an empty `otherBlockers`, or whose other blockers were all given too, is now coverable. Name each workflow that is still blocked and the blockers it still needs. A workflow with a `ripploLimitation` among its other blockers is never now coverable.

The next review reads the pull request description to learn which gaps to cover. Add this section to it, one line per gap, with the `workflow` exactly as the gap gives it. Include the gaps that were already possible, because no code change points the review at them.

```md
## Ripplo gaps

- <workflow>: added `POST /api/ripplo/invoices`, which creates an invoice for the run's organization.
- <workflow>: already possible, `GET /api/orders/:id` returns the refund state.
```

When the branch has no pull request yet, give the user the section to paste when they open one.

Tell the user to push. Once the push is on the pull request, offer to run `npx ripplo review start <prUrl> --depth deep`, with the `prUrl` from `npx ripplo review <codeReviewId>`, and wait for it as `/ripplo:review` describes. The review covers each listed workflow, or keeps its gap and says what still blocks it.
