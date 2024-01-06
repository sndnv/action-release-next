# action-release-next
GitHub action for tracking and incrementing project release versions.

### Why?
Managing versions for projects with more than one technology can be cumbersome and error-prone as each one might
have slightly different ways of defining versions (for example, sbt vs maven vs gradle).

This action aims to provide a standard way of handling version updates with a minimum amount of configuration.

The general idea would be to have this action as part of a larger set of workflows. A maintainer would manually
trigger this action/release process which will result in a new tag being created. Another workflow could then pick
up this new tag and publish the relevant artifacts automatically (a few projects that do just that can be seen below).

### Workflow
The release process has the following steps:
1) Verify that there are no uncommitted changes
2) Retrieve current version (versions in all files must match) and calculate next release version based on config
3) Update all version files with the next release version
4) Verify that the update was applied correctly
5) Create a commit and a tag for the release version update
6) Calculate and update all version files with the next snapshot version
7) Verify that the update was applied correctly
8) Create a commit for the snapshot version update
9) Push commits and tag to the target branch

### Inputs
Most inputs in the [action.yml](action.yml) file are, hopefully, easy to understand. However, the following need
a bit  more information than what fits reasonably in a few lines of YAML.

#### `next_version`
The action requires the type of the next release version to be specified via this input. The possible
options are `patch` (default), `minor` and `major`, or a specific version (for example, `1.5.3`).

#### `version_files_json`
Version file information is provided as a JSON map; the keys are paths to the version files (relative to the current
working directory) and the values are regular expressions that should match (and capture) the version number.

Example keys:
- `version.sbt` - matches a file with that name in the root/current directory
- `test-flutter/pubspec.yaml` - matches a file called `pubspec.yaml` in the `test-flutter` directory

Example regexes:
- `\s*version=(\d+\.\d+\.\d+.*)` - matches a line in a version file that (optionally) starts with some whitespace (`\s*`), 
then `version=` and then the version itself; the captured version (between `(` and `)`) must consist of 3 numbers (`\d+`)
separated by dots (`\.`), followed by some (optional) text (`.*`).
- `^ThisBuild / version := "(.*)"$` - matches a line in a version file that starts with `ThisBuild / version :=` and
captures any version string between two quotes (`"`).

> For clarity, the backslashes in the examples above are not escaped, but usually that needs to be done (by using two
> backslashes (`\\`) instead of just one).

#### `release_key`
Normally, a GitHub action cannot make changes to (protected) branches so this action uses [Deploy Keys](https://docs.github.com/en/authentication/connecting-to-github-with-ssh/managing-deploy-keys#deploy-keys)
to push to the specified target branch.

To make a deploy key usable by this action, it first needs to be created via `ssh-keygen` and the repo should be updated
accordingly (see above link for more info; do note that **the deploy key should have write permissions**). After the key is available,
a new repository secret needs to be added via the repo's `Settings` -> `Secrets and variables` -> `Actions` -> `Repository secrets`.
The content of the secret should be the private key associated with the deploy key.

> For more information about security and hardening your actions, see GitHub's [Security guides](https://docs.github.com/en/actions/security-guides).

#### Example
Example release workflow, intended to be triggered manually:
````yaml
name: Release

on:
  workflow_dispatch:
    inputs:
      next_version:
        description: 'Next release version'
        required: true
        default: 'patch'
        type: choice
        options:
          - patch
          - minor
          - major

jobs:
  validate:
    runs-on: ubuntu-latest

    if: github.ref == 'refs/heads/main' # only allow running on the main branch

    steps:
      - uses: sndnv/action-release-next@v1
        with:
          next_version: ${{ inputs.next_version }}
          target_branch: main
          release_key: ${{secrets.RELEASE_KEY}}
          version_files_json: |
            {
              "example-scala/version.sbt": "^ThisBuild / version := \"(\\d+\\.\\d+\\.\\d+.*)\"$",
              "example-flutter/pubspec.yaml": "^version: (\\d+\\.\\d+\\.\\d+.*)$",
              "example-python/setup.py": "\\s*version='(\\d+\\.\\d+\\.\\d+.*)'"
            }
````

Another example can be seen in the action's test workflow - https://github.com/sndnv/action-release-next/blob/main/.github/workflows/main.yml

### Example Projects
- https://github.com/sndnv/stasis/tree/master/.github/workflows
- https://github.com/sndnv/asciichart/tree/master/.github/workflows
- https://github.com/sndnv/fin/tree/master/.github/workflows

## Contributing

Contributions are always welcome!

Refer to the [CONTRIBUTING.md](CONTRIBUTING.md) file for more details.

## Versioning

We use [SemVer](http://semver.org/) for versioning.

## License

This project is licensed under the Apache License, Version 2.0 - see the [LICENSE](LICENSE) file for details

> Copyright 2024 https://github.com/sndnv
>
> Licensed under the Apache License, Version 2.0 (the "License");
> you may not use this file except in compliance with the License.
> You may obtain a copy of the License at
>
> http://www.apache.org/licenses/LICENSE-2.0
>
> Unless required by applicable law or agreed to in writing, software
> distributed under the License is distributed on an "AS IS" BASIS,
> WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
> See the License for the specific language governing permissions and
> limitations under the License.
