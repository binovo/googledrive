# Build
The `google-drive` project uses _Travis CI_. \
The `.travis.yml` config file can be found in the root of the repository.


## Stages and Jobs
1. **Build**: Java Build with Unit Tests, WhiteSource
2. **Tests**: Executes the E2E tests in `alfresco-mm-test-endtoend`
3. **Release**: Release and Deployment by publishing to Nexus and AWS Staging bucket.
4. **Company Release**: Downloads the `WhiteSource` report and `distribution` and publishes to AWS Releases.


## Branches
Travis CI builds differ by branch:
* `master` / `SP/*` / `HF/*` branches:
  - regular builds which include the _Build_ and _Tests_ stages;
  - if the commit message contains the `[trigger release]` tag, the builds will also 
  include the _Release_ stage;
  - PR builds where the latest commit contains the `[trigger release]` tag will execute dry runs 
  of the release jobs (no artifacts will be published until the PR is actually merged).
* `GOOGLEDOCS-*` branches:
  - regular builds which include only the _Build_ and _Tests_ stages;
* `company_release` branch:
  - builds that include the _Company Release_ stage only.

All other branches are ignored.


## Release process steps & info
Prerequisites:
 - the `master` / `SP/*` / `HF/*` branch is green and it contains all the changes that should be 
 included in the next release.

Steps:
1. Create a new branch with the name `GOOGLEDOCS-###_release_version` from the `master` / `SP/*`/ `HF/*` 
branch.
2. Update the project version if the current POM version is not the next desired release; use a
maven command, i.e.
    ```bash
    mvn versions:set -DnewVersion=#.##.#-SNAPSHOT versions:commit
    ```
3. Update the project's dependencies (remove the `-SNAPSHOT` suffixes - only for dependencies, not
 for the local project version).
4. Create a new commit with the `[trigger release]` tag in its message. If no local changes have 
been generated by steps (2) and (3), then an empty commit should be created - e.g.
     ```bash
     git commit --allow-empty -m "GOOGLEDOCS-###: Release GD #.##.# [trigger release]"
     ```
 
     > The location of the `[trigger release]` tag in the commit message is irrelevant.

     > If for any reason your PR contains multiple commits, the commit with the `[trigger release]`
     tag should be the last (newest) one. This will trigger the Release dry runs.
5. Open a new Pull Request from the `GOOGLEDOCS-###_release_version` branch into the original
`master` / `SP/*` / `HF/*` branch and wait for a green build; the **Release** stage on the PR build
 will only execute a _Dry_Run_ of the release.
6. Once it is approved, merge the PR, preferably through the **Rebase and merge** option. If the 
**Create a merge commit** (_Merge pull request_) or **Squash and merge** options are used, you 
need to ensure that the _commit message_ contains the `[trigger release]` tag (sub-string).

## Company Release process steps & info
Prerequisites:
  - Engineering Release of the desired version has been done.

Steps:
1. Create a new `company_release` branch from the `master` / `SP/*`/ `HF/*` branch. This job uses 
the latest branch git tag to identify the version to be uploaded to the S3 release bucket.
2. Wait for a green build on the branch.
3. Delete local and remote `company_release` branch.
