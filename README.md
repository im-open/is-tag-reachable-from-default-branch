# is-tag-reachable-from-default-branch

This action takes in a tag and determines if it is reachable from the default branch. This action should be used in conjunction with the `actions/checkout` action where the `fetch-depth` arg has been set so the action has access to tags and history.

When this executes it will:

1. Checkout the default branch if it is not currently on it
   - The `ref` arg should be provided in this case, so the action knows which ref to switch back to.
   - The `ref` can be a branch, tag or SHA.
2. Retrieve the list of tags that are reachable from the default branch
3. Check if the tag is in that list of reachable tags
4. If the tag is not in the reachable list it will set the Exit Code to 1 unless the `error-if-not-reachable` input is set to false.
5. Switch back to the original branch if a `ref` arg was provided

If you are working on something other than the default branch, it may be best to run this earlier in your workflow so switching branches doesn't cause loss of changes.

## Index <!-- omit in toc -->

- [is-tag-reachable-from-default-branch](#is-tag-reachable-from-default-branch)
  - [Inputs](#inputs)
  - [Outputs](#outputs)
  - [Usage Examples](#usage-examples)
  - [Contributing](#contributing)
    - [Incrementing the Version](#incrementing-the-version)
    - [Source Code Changes](#source-code-changes)
    - [Updating the README.md](#updating-the-readmemd)
    - [Tests](#tests)
  - [Code of Conduct](#code-of-conduct)
  - [License](#license)

## Inputs

| Parameter                | Is Required | Description                                                                                                                                                                                                   |
|--------------------------|-------------|---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| `tag`                    | true        | The tag to check if it is reachable from the default branch.                                                                                                                                                  |
| `default-branch`         | false       | The default branch of the repository. Defaults to main.                                                                                                                                                       |
| `ref`                    | false       | Required when a `ref` arg was used with `actions/checkout` or if a non-default branch was selected when starting the workflow. Specify the same ref here so the action can return the repository to that ref. |
| `error-if-not-reachable` | false       | Determines whether the action should throw an error if the tag is not reachable on the default branch. Defaults to true.                                                                                      |

## Outputs

| Output      | Description                                                 |
|-------------|-------------------------------------------------------------|
| `reachable` | true or false value indicating whether the tag is reachable |

## Usage Examples

```yml
jobs:
  # Assumes the default branch is main and that the workflow is being run from the main branch
  validate-tag-to-deploy-default-branch:
    runs-on: [ubuntu-20.04]
    steps:
      - uses: actions/checkout@v7

      - name: Check if tag is reachable by main
        # You may also reference just the major or major.minor version
        uses: im-open/is-tag-reachable-from-default-branch@v2.0.0
        with:
          tag: 'latest'

  # Assumes the default branch is main and that the workflow is being run from a non-default branch
  validate-tag-to-deploy-from-non-default-workflow-branch:
    runs-on: [ubuntu-20.04]
    steps:
      - uses: actions/checkout@v7

      - name: Check if tag is reachable by main
        uses: im-open/is-tag-reachable-from-default-branch@v2.0.0
        with:
          tag: 'latest'
          ref: ${{ github.ref }}

  # Specifies the default branch is master, errors are handled outside the workflow and that
  # the action should return to v2.0.12 once it has finished checking
  validate-tag-to-deploy-advanced:
    runs-on: [ubuntu-20.04]
    steps:
      - uses: actions/checkout@v7
        with:
          ref: "v2.0.12"

      - name: Check if tag is reachable by master
        id: tag-check
        uses: im-open/is-tag-reachable-from-default-branch@v2.0.0
        with:
          tag: 'latest'                 # The tag to check
          error-if-not-reachable: false # Don't throw an error if the tag is not reachable
          default-branch: master # The default branch is master in this case, not main
          ref: 'v2.0.12'                # This is the branch/tag/sha that has been checked out

      - name: Exit if tag is not reachable
        if: ${{ steps.tag-check.outputs.reachable }} == false
        run: |
          echo "The tag ${{ github.events.inputs.tag }} is not reachable on the default branch"
          exit 1

  deploy-to-prod:
    needs: [validate-tag-to-deploy-advanced]
    runs-on: [ubuntu-20.04]
    steps:
      - uses: actions/checkout@v7
        with:
          ref: 'v2.0.12'

      - name: Prod Deploy
        run: ./deploy-to-prod
```

## Contributing

When creating PRs, please review the following guidelines:

- [ ] The action code does not contain sensitive information.
- [ ] At least one of the commit messages contains the appropriate `+semver:` keywords listed under [Incrementing the Version] for major and minor increments.
- [ ] The README.md has been updated with the latest version of the action.  See [Updating the README.md] for details.
- [ ] Any tests in the [build-and-review-pr] workflow are passing

### Incrementing the Version

This repo uses [git-version-lite] in its workflows to examine commit messages to determine whether to perform a major, minor or patch increment on merge if [source code] changes have been made.  The following table provides the fragment that should be included in a commit message to active different increment strategies.

| Increment Type | Commit Message Fragment                     |
|----------------|---------------------------------------------|
| major          | +semver:breaking                            |
| major          | +semver:major                               |
| minor          | +semver:feature                             |
| minor          | +semver:minor                               |
| patch          | *default increment type, no comment needed* |

### Source Code Changes

The files and directories that are considered source code are listed in the `files-with-code` and `dirs-with-code` arguments in both the [build-and-review-pr] and [increment-version-on-merge] workflows.  

If a PR contains source code changes, the README.md should be updated with the latest action version.  The [build-and-review-pr] workflow will ensure these steps are performed when they are required.  The workflow will provide instructions for completing these steps if the PR Author does not initially complete them.

If a PR consists solely of non-source code changes like changes to the `README.md` or workflows under `./.github/workflows`, version updates do not need to be performed.

### Updating the README.md

If changes are made to the action's [source code], the [usage examples] section of this file should be updated with the next version of the action.  Each instance of this action should be updated.  This helps users know what the latest tag is without having to navigate to the Tags page of the repository.  See [Incrementing the Version] for details on how to determine what the next version will be or consult the first workflow run for the PR which will also calculate the next version.

### Tests

The build and review PR workflow includes tests which are linked to a status check. That status check needs to succeed before a PR is merged to the default branch.  The tests do not need special permissions, so they should succeed whether they come from a branch or a fork.

## Code of Conduct

This project has adopted the [im-open's Code of Conduct](https://github.com/im-open/.github/blob/main/CODE_OF_CONDUCT.md).

## License

Copyright &copy; 2023, Extend Health, LLC. Code released under the [MIT license](LICENSE).

<!-- Links -->
[Incrementing the Version]: #incrementing-the-version
[Updating the README.md]: #updating-the-readmemd
[source code]: #source-code-changes
[usage examples]: #usage-examples
[build-and-review-pr]: ./.github/workflows/build-and-review-pr.yml
[increment-version-on-merge]: ./.github/workflows/increment-version-on-merge.yml
[git-version-lite]: https://github.com/im-open/git-version-lite
