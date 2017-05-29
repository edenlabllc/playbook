# Versioning

We SHOULD do our best to follow [SemVer v2.0](http://semver.org/). 

To make it easier for developers, all commits MAY be tagged either with change (`[major]`, `[minor]` or a `[feature]`) tags.

CI will decline builds that does not contain at least one commit with change tag. 
Thus in a **new feature branch** you MUST start with a meaningful commit (that will go to PR name) with a change tag.

## Pull Requests

- PR name MUST have change tag and meaningful description. 
- PR description SHOULD include references to the relative artifacts for feature implementation.
- PR comments and commit comments MAY include notes from a developer.
- PR should be squash-merged to a master branch.

## Trunk Branch

We use [trunk-based development](https://trunkbaseddevelopment.com/) with release branches. Our trunk branch is "master".

![Flow](http://paulhammant.com/images/what_is_trunk.jpg)

## Release Branches

  - When new version is released, release branch is created. It MUST be named as `vMAJOR.FEATURE` from a latest git tag, eg: `v2.3`.
  
  - After release branch is created, all code in it goest in a maintenance mode - no new features or breaking changes will be accepted (our CI will try best to check it).
  
  ```
  $ source ./bin/ci/version-info.sh
  Version information: 
   - Previous version was 0.4.0
   - There was 0 major, 1 feature and 0 changes since then
   - Next version will be 0.5.0
   - This build changes version that is in maintenance mode
  [ERROR] You can not add features or breaking changes to the version that is in maintenance mode.
  ```
  
  - When hotfix is required, checkout release branch, create hotfix branch from it and merge it back to a release branch with `[minor]` tag. New container will be built.
