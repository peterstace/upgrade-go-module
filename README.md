# upgrade-go-module

A small utility to automate Go module upgrades.

## What does it do?

It solves two closely related user stories:

1. As a Go developer who has just released a new Go module, I want to upgrade
   that module for any Go programs that consume it, so that consumers get the
   new changes.

2. As a Go developer what writes many Go programs, I want to upgrade the
   dependencies of those programs to specific versions, so that I don't
   continue to use older versions.

3. As a Go developer, I want to upgrade multiple module dependencies at once
   in a single PR, so that I can efficiently manage related updates.

It solves these two user stories by automating a workflow that might otherwise
be executed manually:

1. Work out which locally cloned git repos contain Go programs that depend
   on a particular module.

2. For each of those repos, create a feature branch to do the upgrade in.

3. Upgrade the Go module dependency to a new version.

4. Create a PR containing the change.

## Installation

`upgrade-go-modules` is just a standalone Python script, so it's doesn't need
to be installed per se. _If_ you really want to install it though, you can add
this cloned repo to your PATH, add a symlink from a directory in your PATH to
the script, or just copy the script into a directory in your path.

It has a few runtime dependencies:

- `git`, go creating branches, pushing changes etc.
- `go`, for upgrading module dependencies.
- [`gh`](https://github.com/cli/cli) for raising PRs.

## Example Usage

```
usage: upgrade-go-module [-h] --module NAME@VERSION [--search-root SEARCH_ROOT]
                         [--target-branch TARGET_BRANCH] [--reference REFERENCE]
                         [--draft] [--label LABEL] [--upgrade-all]
                         [--always-skip-if-no-change]

Searches for local git repos containing Go modules, updates specified Go module
dependencies to particular versions, then raises a PR with those changes.

optional arguments:
  -h, --help            show this help message and exit
  --module NAME@VERSION Module to upgrade in NAME@VERSION format (can be specified multiple times)
  --search-root SEARCH_ROOT
  --target-branch TARGET_BRANCH
  --reference REFERENCE
  --draft
  --label LABEL
  --upgrade-all
  --always-skip-if-no-change
```

To invoke the script, you need to provide at least one module to upgrade using
the `--module` flag with the format `NAME@VERSION`. You can specify multiple
modules by using the `--module` flag multiple times.

Optionally, you can also provide:

- A search root, which will tell the script where to look for git repos. If not
  provided, it defaults to the current working directory.

- A draft flag, which will raise PRs in draft mode (defaults to false).

- A target branch, which is the branch that the PR should merge into (defaults
  to the default branch of the repo, usually `master` or `main`).

- A reference, which is an opaque string to include in the PR description. This
  can contain anything, but be useful for referencing (for example) a Github or
  Jira issue.

- A label, which will be added to the created PR. Can be specified multiple times.

- An upgrade-all flag, which will skip the interactive prompt for each repo.

- An always-skip-if-no-change flag, which will skip repos where only go.mod,
  go.sum, or vendor/modules.txt would change.

### Single Module Example

```
$ upgrade-go-module \
	--module github.com/peterstace/simplefeatures@v0.32.0 \
	--search-root ~/go \
	--reference https://company.atlassian.net/browse/PROJ-42 \
	--draft \
	--target-branch qa
```

### Multiple Modules Example

```
$ upgrade-go-module \
	--module github.com/peterstace/simplefeatures@v0.32.0 \
	--module github.com/stretchr/testify@v1.8.4 \
	--module github.com/google/uuid@v1.4.0 \
	--search-root ~/go \
	--reference https://company.atlassian.net/browse/PROJ-42 \
	--label dependencies \
	--upgrade-all
```

## Goals

- To automated the manual workflow involved when upgrading a Go module
  dependency.

- To help me learn Python.

## Non-goals

- Automated checking for new module versions.

- Anything that could be considered vaguely _complicated_.

## FAQ

### Why use `subprocess`?

The script uses the Python `subprocess` module for shelling out to `git`, `go`,
and `gh`. An alternative could have been to use libraries like `gitpython` or
`pygithub`. There is tradeoff though: by making use of runtime command
dependencies rather than runtime Python library dependencies, the script is
much easier for users (so setup required).

### What's wrong with existing solutions?

There are a few off the shelf solutions for automated dependency upgrades, e.g.
[renovate](https://github.com/renovatebot/renovate) and
[dependabot](https://dependabot.com/).

There's nothing wrong with them as such, they're just a bit too complicated for
what I want to achieve. If they work for you though, that's great.
