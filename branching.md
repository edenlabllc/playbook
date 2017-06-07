# Development process

All commits MUST go to a trunk (in our case `master`) branch. Each change increments version, creates tag and docker container. Because of this we can release to development environment frequently or even continuously.

To make versioning easier we SHOULD follow [SemVer v2.0](http://semver.org/). There are nice helpers on our CI that will help you to follow this guide:

  - for commits to master branch: tag your commits with either `[major]`, `[minor]` or a `[feature]` tags, eg: `git commit -am "Added feature A [feature]"` will bump _feature_ part of a release version.
  - for PR you need to tag only first commit and make sure PR title on GitHub contains this tag (it should be done automatically), after squash-merging single tagged commit will go into `master`.
  - If your change affects docks or some minor technical details and should not create release, simply tag it with `[ci skip]` and Travis won't start build for it.

CI won't allow you to build commits in master branch that does not increment version.

## Changelogs

CI will built changelog for releases, but all commit messages in `master` branch MUST have reasonable description.

## Going into maintenance mode

When some release went to production we SHOULD freeze it's code, to make sure we can apply minor changes (hotfixes) and release them without deploying new functionality.

To freeze release you need to create branch in format `v{major}.{feature}` from a master branch **after** CI created commit that increments version.

Example:
  ```
  $ cat mix.exs | grep '@version "'
  @version "0.4.2"
  ```

  Lets freeze code for release `v0.4`:

  ```
  $ git branch v0.4
  $ git push origin v0.4:v0.4
  ```

After release branch is created:

  - CI won't build if there are any commits in a release branch that don't have `[minor]` tag. (You cant release features to a maintenance branch).
  - CI won't build new releases in `master` branch for `0.4.X` versions.

## Hotfixing code in maintenanced releases

Ok, there are bug that needs to be fixed and delivered to production, we will start from fixing it in master branch:

  On a master branch or in PR that will be merged to it:
  ```
  $ git commit -am "Fixed issue X [minor]"
  $ git push origin master
  ```

  CI will run all tests, but won't release new version if master is still on `0.4.X` versions:
  ```
  ...
  Current branch: master
  Trunk branch: master
  Build requires maintenance?: 1
  Maintenance branch: v0.4
  [I] This build is not in a trunk or maintenance branch, new version will not be created
  ...
  ```

  Get hotfix commit hash:
  ```
  $ git log --format="%H" -n 1
  33d84d6908ba65686b066dbe1e608923f988a560
  ```

  Then checkout to a maintenance branch and cherry-pick hotfix:
  ```
  $ git checkout v0.4

  $ git pull origin v0.4

  $ git fetch origin master

  $ git cherry-pick 33d84d6908ba65686b066dbe1e608923f988a560
  [v0.4 2fe5e77c] Fixed issue X [minor]
  Date: Wed Jun 7 13:23:55 2017 +0300
  1 file changed, 1 insertion(+), 1 deletion(-)

  $ git push origin v0.4
  ```

  Done. New release v0.4.3 will be build in a maintenance branch and container can be deployed to production :thumbs_up:.

## How does it work?

You can dig into CI scripts, they are in a [separate repo](https://github.com/nebo15/ci-utils/tree/elixir) and should be included as submodule in your projects:

```
git submodule add -b elixir https://github.com/Nebo15/ci-utils.git bin/ci/release
```

After including them appropriate changes must be applied to [.travis.yaml](https://github.com/Nebo15/annon.api/blob/master/.travis.yml) and scripts that already in your `bin`.

## Trunk Branch Development?

There are lots of information about this practice across Internet, eg: [Trunk-Based Development](https://trunkbaseddevelopment.com/).

In general, our flow should look like this:

![Flow](http://paulhammant.com/images/what_is_trunk.jpg)
