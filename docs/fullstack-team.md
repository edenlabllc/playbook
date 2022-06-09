# In Fullstack Developers We Trust
Forget about back or front. Being a team of fullstack developers is beneficial for multiple reasons:
big individual impact by being able to deliver a whole story, back to front, end-to-end
more likely to be able to fix any bug
common understanding and greater respect for each other
smaller dependency to one's individual skills. We can all take over one's work.
In a nutshell, being a fullstack developer will give you freedom and responsibility.

## Coding standards

High level guidelines:

- Be consistent.
- Establish clear boundaries, design code [like it will be removed tomorrow].
- Prefer to write simple code which is easy to understand.
- Prefer to write efficient code (especially when it is slow and should run on scale).
- Follow language-specific conventions.
- Name things unambiguously, use descriptive names.
- Strongly prefer to detect errors.
- [Fail fast] and noisely, [fail politely].
- Don't violate a guideline without a good reason.
- A reason is good when you can convince a teammate.

[like it will be removed tomorrow]: https://blog.rstankov.com/design-code-for-removal
[Fail fast]: http://stratus3d.com/blog/2020/01/20/applying-the-let-it-crash-philosophy-outside-erlang
[fail politely]: https://joearms.github.io/published/2013-04-28-Fail-fast-noisely-and-politely.html

## Style Guide

We write code in a consistent style that emphasizes cleanliness and team communication.

**High level guidelines:**

* Be consistent.
* Don't rewrite existing code to follow this guide.
* Don't violate a guideline without a good reason.
* A reason is good when you can convince a teammate.

## Branch organization

The next branch types are in use in this repo:

### Mainline

The _mainline branch_ (usually named `main`) always reflects the current state of the product. For changes pushed into mainline branch will be performed automated checks and if they pass versions of subprojects that was changed will be incremented (according to documented earlier semver types of changes) and their new releases will be cut. The mainline branch is continuously deployed (usually to the `dev` environment).

### Feature branches

_Feature branches_ are opened by developers when they begin work on a feature, continue working on that feature until they are done, and then integrate with mainline. Feature branches are usually named by pattern `feat/<domain>/<feature-name>`.

It's extremely important that your new feature branch is created from the mainline branch.

Changes you make on a feature branch don't affect the mainline branch, so you're free to experiment and commit changes, safe in the knowledge that your branch won't be merged until it's ready to be reviewed by someone you're collaborating with. If you work on more than one feature at the same time, you should open a separate branch for each one.

During the lifespan of the feature development, you should watch the mainline branch to see if there have been commits since the feature branch was created. Feature branch should be in sync with mainline before merging back (we usually rebasing feature branches on top of the fresh mainline); this can be done at various times during the feature development or at the end of it, but time to handle merge conflicts (if any) should be accounted for.

_Bugfix branches_ or _Test branches_ are just another flavor of the feature branches. They differ only by semantical meaning and naming pattern: bugfix branches are usually named as `fix/<domain>/<bug-name>` for _Bugfix branches_ or `test/<domain>/<test-name>` for _Test branches_.

## Commit guidelines

When committing changes we follow next rules:

- We follow [good commit message guidelines] in our commit messages.

- Each commit should be a logically separate changeset. We split commits per issue, also we make changes to each subproject in separate commits.

- We group related changes (e.g. which address the same issue in the same subproject or which fix the issue discovered during code review) into single commits. Squash trivial commits into one or use [fixup commits] or [`git absorb`] utility for such changes.

- In order to preserve [linear history] we don't use merge commits using [rebase workflow] instead.

[good commit message guidelines]: https://tbaggery.com/2008/04/19/a-note-about-git-commit-messages.html
[fixup commits]: https://thoughtbot.com/blog/autosquashing-git-commits
[`git absorb`]: https://github.com/tummychow/git-absorb
[linear history]: https://www.bitsnbites.eu/a-tidy-linear-git-history
[rebase workflow]: https://thoughtbot.com/blog/git-interactive-rebase-squash-amend-rewriting-history

## Documenting changes

We are documenting changes in projects in changelog files via `rush change` utility. Please refer to the [changelog creation guide] for more info.

[changelog creation guide]: https://rushjs.io/pages/best_practices/change_logs

## Proposing changes

Changes should be made through opening pull requests and should be reviewed by members of @edenlabllc/dev-frontend or @edenlabllc/_**our_sub_team**_ team.

### Steps to propose a pull request

1. Create a pull request to propose changes to a repository.
2. Fill pull request template form
3. Ask @edenlabllc/dev-frontend or @edenlabllc/_**our_sub_team**_ team to review your PR

## Code review

A guide for reviewing code and having your code reviewed.

### General rules

- PR shouldn't be large. Thousands of lines can't be easily reviewed and merged with confidence. Keep is as simple as possible.
- Name your pull requests using the same rules as for commit messages. Because if you squash and merge your pull request, it will create commit from pull request title.
- Review rules is regulated by [Code of conduct](https://github.com/orgs/edenlabllc/teams/dev-frontend/discussions/3)
- Please do not create a Pull Request without creating an issue first.

### Having your code reviewed

- Be grateful for the reviewer's suggestions. ("Good call. I'll make that change.")
- A common axiom is "Don't take it personally. The review is of the code, not you." We used to include this, but now prefer to say what we mean: Be aware of [how hard it is to convey emotion online] and how easy it is to misinterpret feedback. If a review seems aggressive or angry or otherwise personal, consider if it is intended to be read that way and ask the person for clarification of intent, in person if possible.
- Keeping the previous point in mind: assume the best intention from the reviewer's comments.
- Explain why the code exists. ("It's like that because of these reasons. Would
  it be more clear if I rename this class/file/method/variable?")
- Extract some changes and refactorings into future tickets/stories.
- Push commits based on earlier rounds of feedback as isolated commits to the branch. Do not squash until the branch is ready to merge. Reviewers should be able to read individual updates based on their earlier feedback.
- Seek to understand the reviewer's perspective.
- Try to respond to every comment.
- Wait to merge the branch until continuous integration tells you the test suite is green in the branch.
- Merge once you feel confident in the code and its impact on the project.
- Final editorial control rests with the pull request author.

[how hard it is to convey emotion online]: https://www.fastcodesign.com/3036748/why-its-so-hard-to-detect-emotion-in-emails-and-texts

### Reviewing code

Understand why the change is necessary (fixes a bug, improves the user
experience, refactors the existing code). Then:

- Communicate which ideas you feel strongly about and those you don't.
- Identify ways to simplify the code while still solving the problem.
- If discussions turn too philosophical or academic, move the discussion offline
  to a regular Friday afternoon technique discussion. In the meantime, let the
  author make the final decision on alternative implementations.
- Offer alternative implementations, but assume the author already considered
  them. ("What do you think about using a custom validator here?")
- Seek to understand the author's perspective.
- Remember that you are here to provide feedback, not to be a gatekeeper.

## Testing
A key factor to maintainability and scaling
-- --

### Why Do We Test Our Code?
Most of the benefits of testing appears with time, but it has benefits even in the short term:
* To make sure that the code you just wrote behaves as expected
* To provide some specifications about the correct behaviour of your code
* To give you confidence when refactoring existing code
* To make sure that developers will see when they brake existing behaviour

### What Do We Test?
Testing the back end is just as important as testing the front end.

Here is the minimal expectations we have regarding specs into the backend:
* specs for API endpoints
* specs for service methods

For the frontend:
* specs for new utils, helpers and macros.
* e2e specs

### Tools for testing
In general, we use the same tools across different projects:

For the backend: 
* Jest

For the frontend: 
* Jest
* Cypress
* Cucumber

For the mobile: 
* WebdriverIO
* Appium
* Cucumber

### Continuous Integration

Martin Fowler has an [extensive description] of Continuous Integration. The basics are:

* We have a test suite that each developer runs on their own machine.
* When they commit their code to a shared version control repository, the tests are run again, "integrated" with code from other developers.

This helps ensure there's nothing specific to the developer's machine making the tests pass. The code in version control needs to run cleanly in production later so before the code is allowed to be deployed to production, it is run on a CI service.

As default CI service we use **Github Actions**.

[extensive description]: https://tbot.io/mfowler-ci
