# Build
The `share` project uses _Travis CI_. \
The `.travis.yml` config file can be found in the root of the repository.


## Stages and Jobs
1. **Build**: Java Build with Unit Tests, Push a new image to quay.io, WhiteSource, Source Clear Scan (SCA)
2. **Tests**: Executes the E2E tests in `alfresco-tas-share-test`
3. **Release**: Release by publishing to Nexus.
4. **Company Release**: Publish docker image for share to Docker Hub.

## Branches
Travis CI builds differ by branch:
* `master` / `SP/*` branches:
  - regular builds which include the _Build_ and _Tests_ stages;
  - if the commit message contains the `[release]` key word, the builds will also 
  include the _Release_ stage;
  - if the commit message contains the `[trigger company release]` as commit message, the builds will also 
  include the _Company Release_ stage;
* `APPS-*` branches:
  - regular builds which include only the _Build_ and _Tests_ stages;

All other branches are ignored.

## Release process steps & info
Prerequisites:
 - the `master` / `SP/*` / `HF/*` branch is green and it contains all the changes that should be 
 included in the next release.

Steps:
1. Create a new branch with the name `APPS-###_release_version` from the `master` / `SP/*`/ `HF/*` 
branch.
2. Update the global env variables in travis.yml
    ```bash
    env:
   global:
    # Release version has to start with real version (7.0.0-....) for the docker image to build successfully.
    - RELEASE_VERSION=7.0.0-A1
    - DEVELOPMENT_VERSION=7.0.0-SNAPSHOT
     ```
3. Update the project's dependencies (remove the `-SNAPSHOT` suffixes - only for dependencies, not
 for the local project version).
4. Update the version properties in pom.xml from root
 ```bash
        <version.major>7</version.major>
        <version.minor>0</version.minor>
        <version.revision>0</version.revision>

        <!-- Version of Alfresco Platform to depend on -->
        <alfresco.platform.version>7.0.0-A3</alfresco.platform.version>
        <alfresco.content-services.version>7.0.0-A3</alfresco.content-services.version>
 ```

5. Create a new commit with the `[release]` tag in its message. If no local changes have 
been generated by steps (2) and (3), then an empty commit should be created - e.g.
     ```bash
     git commit --allow-empty -m "APPS-###: Release share #.##.# [release]"
     ```
     > The location of the `[release]` tag in the commit message is irrelevant.

     > If for any reason your PR contains multiple commits, the commit with the `[release]`
     tag should be the last (newest) one. This will trigger the Release dry runs.
6. Open a new Pull Request from the `share-###_release_version` branch into the original
`master` / `SP/*` / `HF/*` branch and wait for a green build; 
7. Once it is approved, merge the PR, preferably through the **Rebase and merge** option. If the 
**Create a merge commit** (_Merge pull request_) or **Squash and merge** options are used, you 
need to ensure that the _commit message_ contains the `[release]` tag (sub-string).

## Company Release process steps & info
Prerequisites:
  - Engineering Release of the desired version has been done.

Steps:
1. Create a new branch from the `master` / `SP/*`/ `HF/*` branch. This job uses 
the latest branch git tag to identify the version.
2. Wait for a green build on the branch.
3. Create a new commit with the `[trigger company release]` tag in its message. 
    ```bash
     git commit --allow-empty -m "APPS-###:  #.##.# [trigger company release]"
     ``` 
4. Open a new Pull Request from the new branch into the original.
5. Once it is approved, merge the PR, you need to ensure that the _commit message_ contains the [trigger company release] tag (sub-string).


Note : These instructions are only for Branches in this repo and are not available for forks.


