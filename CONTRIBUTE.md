# Distributed workflow documentation

## Branching model
For now we have these branches:
* master
* development
* feature/\<feature-name\>
* hotfix/\<fix-name\>
* document/\<documentation-name\>

Master is the release branch. Merge to master only from development. Merging needs to be done by a pull request.

Development is the development branch.

Feature branches are short-lived branches that branch off of development and get merged back to development. Merging needs to be done by a pull request. Document branches are the same but for documentation.

Hotfix branches are used to correct bugs on master and development.

#### Reasoning
This makes sense since we are developing a web application where a unit of work is usually a feature. For instance, something like a trunk/no-branches model would be more appropriate for some computational library where a unit of work is to add a function that performs some calculation, but in our case it would force us to make larger, stable commits, since otherwise going back in history to fix something or developing more than one feature in parallel would be a pain. By using feature branches we avoid this.

#### Looking ahead
Eventually, when simulator goes live, we might need a pre-production branch to do some tests before releasing, so we will could move towards Gitflow:
* https://nvie.com/posts/a-successful-git-branching-model/
* https://www.atlassian.com/git/tutorials/comparing-workflows/gitflow-workflow

## Repository model
Repositories with submodules. We decided to adopt this strategy because we can develop backend, frontend and storage separately and keep the commits for each of that code separate. Then, at least for everything backend-, frontend- and storage-related, we can cherry-pick the commits from their master branches to commit to the master branch of the top-level repository, and make a release this way.
* https://git-scm.com/book/en/v2/Git-Tools-Submodules

We use the same branching model for each repository.

## Commits
stable = can build, run and passes tests
* Commit to *master* is a release and must be stable.
* Commit to *master* is either a merge from *development* or *hotfix* branch.
* Commit to *development* is either merge from a *feature*, *document* or *hotfix* branch.
* Commit to *development* must be stable (don't commit unfinished feature or hotfix work to development, for documentation this is more relaxed).
* Commit to *feature* or *hotfix* branch has no requirements.

#### Commit message guidelines
Write your commit message in the imperative: "Fix bug" and not "Fixed bug"
or "Fixes bug."

State in one brief sentence what has been changed, but be precise, not vague: "Fix bug: set correct path for flag_tool dependency" rather than just "Fix bug".

Maybe more elaborate: https://tbaggery.com/2008/04/19/a-note-about-git-commit-messages.html

#### Further guidelines
Incremental commits for logically separate changes. Think before commiting: can I write the commit message as a single brief sentence? If not, it should probably be two or more separate commits.

## Pull requests
Pull request to merge to master needs at least two approvals.

Pull request to merge to development needs at least one approval.

This way we distribute the responsibility for integration among ourselves.

## Releases
At lease once a week, latest by Wednesday, for the duration of the course.
Tagged as vMajor.Minor.Patch where Major starts at 0, Minor at 1 and Patch at 1.

## Git housekeeping
After each release, or each second release, cleanup feature and hotfix branches that were merged in successfully. Otherwise, we keep a growing number of branches and with time  they get hard to navigate.
See link: https://railsware.com/blog/git-housekeeping-tutorial-clean-up-outdated-branches-in-local-and-remote-repositories/
Are there any good reasons not to do this?
