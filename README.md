# Ripplo for Claude Code, Codex, and Cursor

Ripplo reviews your pull requests by driving the app end to end in a real browser and reporting what broke, with the evidence. Ripplo writes and owns the tests. The Ripplo plugin lets your coding agent read a review's findings and fix them in your code, and add what a review could not cover.

## Install

Claude Code:

```sh
claude plugin marketplace add ripplo/agent-plugin
claude plugin install ripplo@ripplo
npx ripplo@latest login
```

Codex:

```sh
codex plugin marketplace add ripplo/agent-plugin
codex plugin add ripplo@ripplo
npx ripplo@latest login
```

Cursor has no install command. `npx ripplo@latest setup` writes the plugin to `~/.cursor/plugins/local/ripplo`.

## Set up

In the app you want reviewed:

```sh
npx ripplo@latest setup
```

It connects the repository to Ripplo, creates an OpenID Connect connection, and shows the values to register in your app. Add the sign in with Ripplo connection to your auth library, set the variables on your dev server, deploy, then run `npx ripplo@latest setup --check`, where a Ripplo browser tests sign-in the way review runs do. When the test passes it asks which coding agent you use, installs this plugin there, and offers to open your project in Ripplo.

## Use

The plugin adds two skills, review and cover. The examples below use the Claude Code form, `/ripplo:review` and `/ripplo:cover`. In Codex, type `$ripplo:review` and `$ripplo:cover`. In Cursor, type `/ripplo-review` and `/ripplo-cover`.

Start a review of a pull request:

```
/ripplo:review <pull request URL>
```

The agent starts the review, checks on it while it runs, and picks up the result when it finishes.

For a review that already ran, copy its id from the Ripplo dashboard:

```
/ripplo:review <codeReviewId>
```

The agent pulls the published issues, renders the failing frames, classifies each issue, and fixes the app in your working tree. After you push, the agent offers to review the new commit and waits for the result.

When a review lists workflows Ripplo could not cover, the app lacks a way for Ripplo to create or observe some state. Copy the command from that list, or from the coverage page. Run it alone to cover every gap of the project:

```
/ripplo:cover <codeReviewId>
/ripplo:cover
```

The agent reads each coverage gap and adds the API or route the app is missing, guarded for Ripplo runs. Push, and the next review covers the workflow.

Copy for agent in Ripplo copies these commands in the form your agent uses. Choose the agent in your account settings.

## Source

This folder is copied from [`ripploai/ripplo`](https://github.com/ripploai/ripplo) on release.
