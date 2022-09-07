# ⑂ forker

[![Release](https://img.shields.io/github/v/release/wayfair-incubator/forker?display_name=tag)](https://github.com/wayfair-incubator/forker/releases)
[![License: MIT](https://img.shields.io/badge/license-MIT-7F187F.svg)](LICENSE)
[![Code of Conduct](https://img.shields.io/badge/CoC-2.0-24B8EE.svg)](CODE_OF_CONDUCT.md)
[![Build](https://github.com/wayfair-incubator/forker/actions/workflows/build.yml/badge.svg?branch=main)](https://github.com/wayfair-incubator/forker/actions/workflows/build.yml)

[GitHub Action](https://github.com/features/actions) to automate fork creation. This action uses [octokit.js](https://github.com/octokit/octokit.js) and the [GitHub API](https://docs.github.com/en/rest) to automatically create a repository fork, either in your personal GitHub account or a GitHub organization that you administer.

If the `checkUser` option is enabled, `forker` will check the specified GitHub organization membership status for the user requesting the fork. If the user is already an organization member, `forker` will proceed to fork the repo, and then optionally grant the user `admin` permissions when using in combination with the `promoteUser` option. If the user is **not** an organization member, `forker` will exit without forking the repository, and display an error.

For legal and compliance reasons, organizations or individuals can choose to provide an optional `licenseAllowlist` to compare against the [license of the repository](https://docs.github.com/en/rest/reference/licenses) being forked. If the license key returned by the GitHub API is not found within the provided allowlist, `forker` will exit without forking the repository, and display an error.

## Inputs

### `token` (string, required)

The GitHub API [token](https://docs.github.com/en/github/authenticating-to-github/keeping-your-account-and-data-secure/creating-a-personal-access-token) you wish to use for automating fork creation. If you are using GitHub [encrypted secrets](https://docs.github.com/en/actions/reference/encrypted-secrets#using-encrypted-secrets-in-a-workflow), you should reference the variable name you have defined for your secret.

> 💡 **Tip:** Ensure the token you are using has sufficient permissions to fork repositories into your intended destination (either an organization or individual user account). In particular, the builtin `GITHUB_TOKEN` has [read-only permissions](https://docs.github.com/en/actions/reference/authentication-in-a-workflow#permissions-for-the-github_token) for repository forks, and therefore may not provide sufficient privileges for use with `forker`.

**Example:** `${{ secrets.ACCESS_TOKEN }}`

### `owner` (string, required)

The owner of the GitHub repository you wish to fork. Can be an organization or individual user account.

**Example:** `tremor-rs`

### `repo` (string, required)

The name of the GitHub repository you wish to fork.

**Example:** `tremor-runtime`

### `org` (string, optional)

The name of the destination GitHub organization where you wish to fork the specified repository.

**Example:** `wayfair-contribs`

### `user` (string, optional)

The GitHub account for the person requesting the fork.

> 💡 **Tip:** This is only required if you are managing a GitHub organization, and wish to associate a specific user with the fork request. If neither `org` nor `user` inputs are specified, `forker` will default to forking the repository into your own GitHub account. Similarly, if _only_ `user` is provided without an accompanying `org`, forker will ignore the field, since users cannot create forks on behalf of other users, only GitHub organizations.

**Example:** `lelia`

### `checkUser` (boolean, optional)

Enforces existing membership for a specified `user` in a specified GitHub `org`.

> 💡 **Tip:** If the user **is** already a GitHub `org` member, `forker` will proceed to fork the `repo`. You can optionally combine this with the `promoteUser` option to grant the user `admin` [permissions](https://docs.github.com/en/organizations/managing-access-to-your-organizations-repositories/repository-roles-for-an-organization) on the forked `repo`.
>
> 🚨 If the user is **not** a GitHub `org` member, `forker` will exit without forking the repository.

**Example:** `true`

**Default:** `false`

### `promoteUser` (boolean, optional)

Grants GitHub `org` members `admin` [permissions](https://docs.github.com/en/organizations/managing-access-to-your-organizations-repositories/repository-roles-for-an-organization) on the `repo` they wish to fork.

> 💡 **Tip:** If the requesting user only intends to make upstream contributions to the `repo` they wish to fork, it is **very likely** that they will not require elevated `admin` privileges. That said, if there is an eventual desire to truly _fork_ off and deviate substantially from the originating project, this option helps give users better control over their project and maintainership.

**Example:** `true`

**Default:** `false`

### `licenseAllowlist` (optional, string)

A newline-delimited (`"\n"`) string representing a list of allowed [license keys](https://docs.github.com/en/rest/reference/licenses) for the repository being forked. If the license key returned by the [Licenses API](https://docs.github.com/en/rest/reference/licenses) is not found within the `licenseAllowlist`, `forker` will **not** fork the repository, and instead exit with a warning.

> 💡 **Tip:** You can always reference [this directory](https://github.com/github/choosealicense.com/tree/gh-pages/_licenses) if you need a comprehensive list of license keys, beyond the commonly-used licenses returned from `GET /licenses` in the [GitHub REST API](https://docs.github.com/en/rest/reference/licenses#get-all-commonly-used-licenses).

**Example:** `"0bsd\napache-2.0\nmit"`

## Usage

### Typical

In most cases, you'll want to use the latest stable version (eg. `v0.0.5`):

```yaml
uses: wayfair-incubator/forker@v0.0.5
with:
  token: ${{ secrets.ACCESS_TOKEN }}
  repo: tremor-runtime
  owner: tremor-rs
  user: lelia
```

### Development

If you're actively developing a new feature for the action, you can always reference a specific commit SHA (eg. `a694606ff02c8ba2654865adeb7a6d2053b34afa`):

```yaml
uses: wayfair-incubator/forker@a694606ff02c8ba2654865adeb7a6d2053b34afa
with:
  token: ${{ secrets.ACCESS_TOKEN }}
  repo: tremor-runtime
  owner: tremor-rs
  user: lelia
```

### Advanced

If you are automating the creation of forks on behalf of a GitHub organization with many users, you may wish to leverage the optional `checkUser`, `promoteUser`, and `licenseAllowlist` params:

```yaml
uses: wayfair-incubator/forker@v0.0.5
with:
  token: ${{ secrets.ACCESS_TOKEN }}
  repo: tremor-runtime
  owner: tremor-rs
  org: wayfair-contribs
  user: lelia
  checkUser: true
  promoteUser: true
  licenseAllowlist: "0bsd\napache-2.0\nmit"
```

## Developing

> 💡 **Tip:** Please use [node](https://nodejs.org/en/download/releases/) v12.x or later, as well as an npm-compatible version of [typescript](https://www.npmjs.com/package/typescript).

Install the node dependencies:

```bash
npm install
```

Build the TypeScript code and package it for distribution:

```bash
npm run build && npm run package
```

Run the Jest unit tests:

> 💡 **Tip:** Before running any tests locally which require authenticating against the GitHub API, please ensure you've defined a valid token for the environment variable `INPUT_TOKEN` in your preferred shell (or shell profile), eg: `export INPUT_TOKEN="my_github_api_token_value"`. This is functionally equivalent to defining an input value for the `token` parameter in your GitHub Action's [workflow YAML](https://docs.github.com/en/actions/learn-github-actions/workflow-syntax-for-github-actions) configuration.

```bash
$ npm test

 PASS  __tests__/main.test.ts
  ✓ action throws error without required inputs (2 ms)
```

Convenience command to run all npm scripts:

```bash
npm run all
```

### Publishing

Actions are run from GitHub repos so we will checkin the packed `dist/` folder.

Then run [ncc](https://github.com/zeit/ncc) and push the results:

```bash
npm run package
git add dist
git commit -a -m "prod dependencies"
git push origin releases/v0.0.5
```

> 💡 **Tip:** We recommend using the `--license` option for `ncc`, which will create a license file for all of the production node modules used in your project.

Your action is now published! 🚀

See the [versioning documentation](https://github.com/actions/toolkit/blob/master/docs/action-versioning.md) for more details.

### Validation

You can now validate the action by referencing `./` in a workflow in your repo (see [`build.yml`](.github/workflows/build.yml))

```yaml
uses: ./
with:
  path: ./
  token: ${{ secrets.ACCESS_TOKEN }}
  ref: ${{ github.event.pull_request.head.sha }}
  repo: tremor-runtime
  owner: tremor-rs
  user: lelia
```

See the [Actions tab](https://github.com/wayfair-incubator/forker/actions) to view runs of this action!  ✅

## Contributing

Contributions are what make the open source community such an amazing place to learn, inspire, and create. Any contributions you make are **greatly appreciated**. For detailed contributing guidelines, please see [CONTRIBUTING.md](CONTRIBUTING.md).

## License

Distributed under the MIT License. See [LICENSE](LICENSE) for more information.

## Acknowledgements

This [GitHub Action](https://github.com/features/actions) was adapted from the [typescript-action](https://github.com/actions/typescript-action) template, with additional [project content](https://github.com/wayfair-incubator/oss-template) curated with 💜 by [Wayfair](https://github.com/wayfair).

For more information about Wayfair's [Open Source Program Office](https://www.linuxfoundation.org/tools/creating-an-open-source-program/), check out [**wayfair.github.io**](https://wayfair.github.io/) 🎉
