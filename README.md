<div align="center">
  
# Repo File Sync Action

[![GitHub](https://img.shields.io/github/license/mashape/apistatus.svg)](https://github.com/Redocly/repo-file-sync-action/blob/main/LICENSE)

Keep files like Action workflows or entire directories in sync between multiple repositories.

</div>

## 👋 Introduction

With [repo-file-sync-action](https://github.com/Redocly/repo-file-sync-action) you can sync files, like workflow `.yml` files, configuration files or whole directories between repositories or branches. It works by running a GitHub Action in your main repository everytime you push something to that repo. The action will use a `sync.yml` config file to figure out which files it should sync where. If it finds a file which is out of sync it will open a pull request in the target repository with the changes.

## 🚀 Features

- Keep GitHub Actions workflow files in sync across all your repositories
- Sync any file or a whole directory to as many repositories as you want
- Easy configuration for any use case
- Create a pull request in the target repo so you have the last say on what gets merged
- Automatically label pull requests to integrate with other actions like [automerge-action](https://github.com/pascalgn/automerge-action)
- Assign users to the pull request

## 📚 Usage

Create a `.yml` file in your `.github/workflows` folder (you can find more info about the structure in the [GitHub Docs](https://docs.github.com/en/free-pro-team@latest/actions/reference/workflow-syntax-for-github-actions)):

**.github/workflows/sync.yml**

```yml
name: Sync Files
on:
  push:
    branches:
      - main
  workflow_dispatch:
jobs:
  sync:
    runs-on: ubuntu-latest
    steps:
      - name: Checkout Repository
        uses: actions/checkout@v2
      - name: Run GitHub File Sync
        uses: Redocly/repo-file-sync-action@main
        with:
          GH_PAT: ${{ secrets.GH_PAT }}
```

In order for the Action to access your repositories you have to specify a [Personal Access token](https://docs.github.com/en/free-pro-team@latest/github/authenticating-to-github/creating-a-personal-access-token) as the value for `GH_PAT`.

> **Note:** `GITHUB_TOKEN` will not work

It is recommneded to set the token as a
[Repository Secret](https://docs.github.com/en/free-pro-team@latest/actions/reference/encrypted-secrets#creating-encrypted-secrets-for-a-repository).

The last step is to create a `.yml` file in the `.github` folder of your repository and specify what file(s) to sync to which repositories:

**.github/sync.yml**

```yml
user/repository:
  - .github/workflows/test.yml
  - .github/workflows/lint.yml

user/repository2:
  - source: workflows/stale.yml
    dest: .github/workflows/stale.yml
```

More info on how to specify what files to sync where [below](#%EF%B8%8F-sync-configuration). 
<br/><br/>
 ## ⚙️ Action Inputs

Here are all the inputs [repo-file-sync-action](https://github.com/Redocly/repo-file-sync-action) takes:

| Key | Value | Required | Default |
| ------------- | ------------- | ------------- | ------------- |
| `GH_PAT` | Your [Personal Access token](https://docs.github.com/en/free-pro-team@latest/github/authenticating-to-github/creating-a-personal-access-token) | **Yes** | N/A |
| `CONFIG_PATH` | Path to the sync configuration file | **No** | .github/sync.yml |
| `PR_LABELS` | Labels which will be added to the pull request. Set to false to turn off | **No** | sync |
| `ASSIGNEES` | People to assign to the pull request | **No** | N/A |
| `COMMIT_PREFIX` | Prefix for commit message and pull request title | **No** | 🔄 |
| `COMMIT_BODY` | Commit message body. Will be appended to commit message, separated by two line returns. | **No** | '' |
| `COMMIT_EACH_FILE` | Commit each file seperately | **No** | true |
| `GIT_EMAIL` | The e-mail address used to commit the synced files | **No** | the email of the PAT used |
| `GIT_USERNAME` | The username used to commit the synced files | **No** | the username of the PAT used |
| `OVERWRITE_EXISTING_PR` | Overwrite any existing Sync PR with the new changes | **No** | true |
| `BRANCH_PREFIX` | Specify a different prefix for the new branch in the target repo | **No** | repo-sync/SOURCE_REPO_NAME |
| `TMP_DIR` | The working directory where all git operations will be done | **No** | tmp-${ Date.now().toString() } |
| `DRY_RUN` | Run everything except that nothing will be pushed | **No** | false |
| `SKIP_CLEANUP` | Skips removing the temporary directory. Useful for debugging | **No** | false |
| `SKIP_PR` | Skips creating a Pull Request and pushes directly to the default branch | **No** | false |

## 🛠️ Sync Configuration

In order to tell [repo-file-sync-action](https://github.com/Redocly/repo-file-sync-action) what files to sync where, you have to create a `sync.yml` file in the `.github` directory of your main repository (see [action-inputs](#%EF%B8%8F-action-inputs) on how to change the location).

The top-level key should be used to specify the target repository in the format `username`/`repository-name`@`branch`, after that you can list all the files you want to sync to that individual repository:

```yml
user/repo:
  - path/to/file.txt
user/repo2@develop:
  - path/to/file2.txt
```

There are multiple ways to specify which files to sync to each individual repository.

### List individual file(s)

The easiest way to sync files is the list them on a new line for each repository:

```yml
user/repo:
  - .github/workflows/build.yml
  - LICENSE
  - .gitignore
```

### Different destination path/filename(s)

Using the `dest` option you can specify a destination path in the target repo and/or change the filename for each source file:

```yml
user/repo:
  - source: workflows/build.yml
    dest: .github/workflows/build.yml
  - source: LICENSE.md
    dest: LICENSE
```

### Sync entire directories

You can also specify entire directories to sync:

```yml
user/repo:
  - source: workflows/
    dest: .github/workflows/
```

### Exclude certain files when syncing directories

Using the `exclude` key you can specify files you want to exclude when syncing entire directories (#26).

```yml
user/repo:
  - source: workflows/
    dest: .github/workflows/
    exclude: |
      node.yml
      lint.yml
```

> **Note:** the exclude file path is relative to the source path

### Don't replace existing file(s)

By default if a file already exists in the target repository, it will be replaced. You can change this behaviour by setting the `replace` option to `false`:

```yml
user/repo:
  - source: .github/workflows/lint.yml
    replace: false
```

### Sync the same files to multiple repositories

Instead of repeating yourself listing the same files for multiple repositories, you can create a group:

```yml
group:
  repos: |
    user/repo
    user/repo1
  files: 
    - source: workflows/build.yml
      dest: .github/workflows/build.yml
    - source: LICENSE.md
      dest: LICENSE
```

You can create multiple groups like this:

```yml
group:
  # first group
  - files:
      - source: workflows/build.yml
        dest: .github/workflows/build.yml
      - source: LICENSE.md
        dest: LICENSE
    repos: |
      user/repo1
      user/repo2

  # second group
  - files: 
      - source: configs/dependabot.yml
        dest: .github/dependabot.yml
    repos: |
      user/repo3
      user/repo4
```

### Syncing branches

You can also sync different branches from the same or different repositories (#51). For example, a repository named `foo/bar` with branch `main`, and `sync.yml` contents:

```yml
group:
  repos: |
    foo/bar@de
    foo/bar@es
    foo/bar@fr
  files:
    - source: .github/workflows/
      dest: .github/workflows/
```

Here all files in `.github/workflows/` will be synced from the `main` branch to the branches `de`/`es`/`fr`.

## 📖 Examples

Here are a few examples to help you get started!

### Basic Example

**.github/sync.yml**

```yml
user/repository:
  - LICENSE
  - .gitignore
```

### Sync all workflow files

This example will keep all your `.github/workflows` files in sync across multiple repositories:

**.github/sync.yml**

```yml
group:
  repos: |
    user/repo1
    user/repo2
  files:
    - source: .github/workflows/
      dest: .github/workflows/
```

### Custom labels

By default [repo-file-sync-action](https://github.com/Redocly/repo-file-sync-action) will add the `sync` label to every PR it creates. You can turn this off by setting `PR_LABELS` to false, or specify your own labels:

**.github/workflows/sync.yml**

```yml
- name: Run GitHub File Sync
  uses: Redocly/repo-file-sync-action@main
  with:
    GH_PAT: ${{ secrets.GH_PAT }}
    PR_LABELS: |
      file-sync
      automerge
```

### Assign a user to the PR

You can tell [repo-file-sync-action](https://github.com/Redocly/repo-file-sync-action) to assign users to the PR with `ASSIGNEES`:

**.github/workflows/sync.yml**

```yml
- name: Run GitHub File Sync
  uses: Redocly/repo-file-sync-action@main
  with:
    GH_PAT: ${{ secrets.GH_PAT }}
    ASSIGNEES: Redocly
```

### Custom GitHub Enterprise Host

If your target repository is hosted on a GitHub Enterprise Server you can specify a custom host name like this:

**.github/workflows/sync.yml**

```yml
https://custom.host/user/repo:
  - path/to/file.txt

# or in a group

group:
  - files:
      - source: path/to/file.txt
        dest: path/to/file.txt
    repos: |
      https://custom.host/user/repo
```

> **Note:** The key has to start with http to indicate that you want to use a custom host.

### Different branch prefix

By default all new branches created in the target repo will be in the this format: `repo-sync/SOURCE_REPO_NAME/SOURCE_BRANCH_NAME`, with the SOURCE_REPO_NAME being replaced with the name of the source repo and SOURCE_BRANCH_NAME with the name of the source branch.

If your repo name contains invalid characters, like a dot ([#32](https://github.com/Redocly/repo-file-sync-action/issues/32)), you can specify a different prefix for the branch (the text before `/SOURCE_BRANCH_NAME`):

**.github/workflows/sync.yml**

```yml
uses: Redocly/repo-file-sync-action@main
with:
    GH_PAT: ${{ secrets.GH_PAT }}
    BRANCH_PREFIX: custom-branch
```

The new branch will then be `custom-branch/SOURCE_BRANCH_NAME`.

> You can use `SOURCE_REPO_NAME` in your custom branch prefix as well and it will be replaced with the actual repo name

### Custom commit body

You can specify a custom commit body. This will be appended to the commit message, separated by two new lines. For example:

**.github/workflows/sync.yml**

```yml
- name: Run GitHub File Sync
  uses: Redocly/repo-file-sync-action@main
  with:
    GH_PAT: ${{ secrets.GH_PAT }}
    COMMIT_BODY: "Change-type: patch"
```

The above example would result in a commit message that looks something like this:
```
🔄 Synced local '<filename>' with remote '<filename>'

Change-type: patch
```


## 💻 Development

The actual source code of this library is in the `src` folder.

- run `yarn lint` or `npm run lint` to run eslint.
- run `yarn build` or `npm run build` to produce a production version of [repo-file-sync-action](https://github.com/Redocly/repo-file-sync-action) in the `dist` folder.

To run the action locally, use `npm start` command with required parameters:

```
GH_PAT=[YOUR_PERSONAL_ACCESS_TOKEN] GITHUB_REPOSITORY=[REPO_YOU_WANT_TO_SYNC] npm start                                                                                  
```

If you receive the following error during the development:


> ::warning::Source [NAME_OF_YOUR_FILE/FOLDER] not found

that means that you should put the file/folder in the root directory of this action project in the same structure that you defined it in the sync config (for ex.: `::warning::Source /docs not found` -> put `/docs` in the root dir of this repo).

## Credits

This project was originally developed by [@betahuhn](https://github.com/BetaHuhn). If you want to use the original version of this project, feel free to check out [BetaHuhn/repo-file-sync-action](https://github.com/BetaHuhn/repo-file-sync-action).

