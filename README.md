# Commithelper

[![Build](https://github.com/jvanbruegge/commithelper/actions/workflows/ci.yml/badge.svg)](https://github.com/jvanbruegge/commithelper/actions/workflows/ci.yml)

A tool to create and lint commit messages. It has two modes `prompt` and `check`. The former asks you questions to create a well-formed commit message, while the latter lints a given commit message.

![A screenshot of a terminal showing the tool](./console.png)

## Table of contents:

-   [Usage](https://github.com/jvanbruegge/commithelper#usage)
-   [Comparison with other tools](https://github.com/jvanbruegge/commithelper#comparison-with-other-tools)
    -   [commitlint](https://github.com/jvanbruegge/commithelper#commitlint)
    -   [commitizen](https://github.com/jvanbruegge/commithelper#commitizen)
-   [Setup](https://github.com/jvanbruegge/commithelper#setup)
    -   [Setup with ghooks](https://github.com/jvanbruegge/commithelper#setup-with-ghooks)
    -   [Setup with husky](https://github.com/jvanbruegge/commithelper#setup-with-husky)
-   [Config File](https://github.com/jvanbruegge/commithelper#config-file)

## Usage

First add `commithelper` as a dev dependency to your project with `npm install commithelper --save-dev`.

```
commithelper prompt [--file <path>] [--config <path>]
```

Interactively asks the user questions to create a commit message. Use the `--file` option to specify a path to write the final message to. If omitted the message is printed to stdout. If the path already contains a file that only contains comments (lines starting with `#`) this file is overwritten, but all comment lines (starting with `#`) are kept. If the path contains a file with non-comment lines no questions are asked and program exits sucessful. This is useful when other tooling is creating the commit message or when using `git commit --amend`. Use the `--config` option to specify where to find the config file. If this is omitted, the config is read from the `commithelper` field in the `package.json`.

```
commithelper check [--file <path>] [--fix] [--config <path>]
```

Lints a commit message according to the configuration. Use `--file` to specify where to find the file containing the commit message. If omitted the message is read from stdin. Use `--fix` to try to automatically fix the commit message to pass linting. This means it will correct capitalization of the subject and correctly wrap the body of the message. Use `--config` to specify where to find the config file. If this is omitted, the config is read from the `commithelper` field in the `package.json`.

It is recommended to use [git hooks](https://git-scm.com/docs/githooks) with this tool. There are several npm packages that help you set up git hooks automatically, e.g. [husky](https://github.com/typicode/husky) or [ghooks](https://github.com/ghooks-org/ghooks).

Use the `prepare-commit-msg` hook with the `prompt` mode to interactively create a commit message and use the `commit-msg` hook for the `check` mode.

## Comparison with other tools

### commitlint

The [commitlint](https://github.com/conventional-changelog/commitlint) package and its plugins are only suited to check an existing commit message against a set of rules. While there is `@commitlint/prompt-cli`, it just states that the commits have to have the form `<type>(<scope>): <subject>` without telling you what the valid options are, so it is pretty useless.

Also it is very complicated to configure as many rules available are barely documented at all.

### commitizen

The quesions in the `prompt` mode of `commithelper` is heavily inspired by [commitizen](https://github.com/commitizen/cz-cli). But commitizen can't lint an existing commit message, so if the user is using the git client built into their editor or provides the message with `git commit -m` commitizen can't help.

Also, by default commitizen requires one of several adapters, each with their own configuration options which makes setting it up quite complicated.

## Setup

### Setup with ghooks

Install ghooks in your project with `npm install ghooks --save-dev`. Then add `config.ghooks` to `package.json` like this:

```json
{
  …
  "config": {
    "ghooks": {
      "prepare-commit-msg": "exec < /dev/tty && npm exec -- commithelper prompt --file $1",
      "commit-msg": "commithelper check --file $1 --fix",
      …
    }
  }
  …
}
```

### Setup with husky

Install husky in your project with `npm install husky --save-dev`. Then add a `prepare` script to your `package.json` that installs the husky hooks.

```json
{
  "scripts": {
    …,
    "prepare": "husky install"
  }
}
```

Then add a new executable file `.husky/prepare-commit-msg` with the content:

```sh
#!/bin/sh
. "$(dirname "$0")/_/husky.sh"

exec < /dev/tty && npm exec commithelper -- prompt --file $1
```

and a new executable file `.husky/commit-msg` with the content:

```sh
#!/bin/sh
. "$(dirname "$0")/_/husky.sh"

npm exec commithelper -- check --file $1 --fix
```

Then run `npm run prepare` to install the hooks in your project.

## Config file

You can either have a seperate json file and use the `--config` option to point `commithelper` to it, or omit `--config` and add a `commithelper` field in your `package.json`. The config object has the following fields:

| key                      | type                                       | default                                   | description                                                                                                                                                                                             |
| ------------------------ | ------------------------------------------ | ----------------------------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| **subjectLimit**         | number                                     | 100                                       | The maximum number of characters that the git subject line (first line of the commit message) is allowed to have. This is including the `type` and the `scope`                                          |
| **subjectSeperator**     | string                                     | ':'                                       | The character to seperate `type` and `scope` from the subject. E.g. with '/' the message would look like `<type>(<scope>)/<subject>`. May not be the empty string                                       |
| **typePrefix**           | string                                     | ''                                        | A string that is added before the `type`, e.g. with '[', the message would look like `[<type>(<scope>): <subject>`                                                                                      |
| **typeSuffix**           | string                                     | ''                                        | A string that is added after the `type`, e.g. with ']', the message would look like `<type>](<scope>): <subject>`                                                                                       |
| **types**                | { name: string, message: string }[]        | _default names_: ['feat', 'fix', 'chore'] | All the commit types that are allowed for this repo. `name` is what will be at the place of `<type>` in the commit message, `message` is the explanation presented to the user during `prompt` mode     |
| **scopes**               | string[]                                   | []                                        | All the scopes that are allowed for all of the commit types                                                                                                                                             |
| **scopeOverrides**       | { [type]: string[] }                       | {}                                        | Override the allowed scopes for a particular commit `type`. If this is set for a particular `type` the array defined here is used as allowed scopes instead of the array defined in the `scopes` option |
| **allowCustomScopes**    | boolean                                    | false                                     | Set this to true if the user is allowed to write out a scope that is not on the list of allowed scopes                                                                                                  |
| **bodyWrap**             | number                                     | 72                                        | The maximum line length that is allowed in the commit body, wraps the body accordingly with `--fix` or in `prompt` mode                                                                                 |
| **ticketPrefix**         | string                                     | 'ISSUES CLOSED:'                          | The string that marks the list of tickets that this commit closes. May not be the empty string                                                                                                          |
| **ticketNumberPrefix**   | string                                     | '#'                                       | A string that is in front of the ticket number. The default is useful for GitHub issues which start with `#` (e.g. `#134`). Use e.g. 'JIRA-' for other numbering systems                                |
| **ticketSeperator**      | string                                     | ','                                       | The string which is used to seperate multiple ticket numbers in the same line                                                                                                                           |
| **breakingPrefix**       | string                                     | 'BREAKING CHANGES:'                       | The string that should start the description of a breaking change. May not be the empty string                                                                                                          |
| **breakingRequiresBody** | boolean                                    | false                                     | Set this to `true` to require a commit with a breaking change to also have a body in addition to the note describing the breaking change                                                                |
| **allowBreakingChanges** | string[]                                   | ['feat', 'fix']                           | The commit `types` that are allowed to have breaking changes. Use the empty array to allow breaking changes for all types                                                                               |
| **upperCase**            | boolean                                    | false                                     | Set this to `true` to require the first letter of the `<subject>` to be upper case (requires lower case when set to `false`)                                                                            |
| **skipQuestions**        | ('body' or 'breaking' or 'issuesClosed')[] | []                                        | Do not ask the user for a body, breaking changes or closed issues                                                                                                                                       |
