# git branchless

[![PyPi](https://img.shields.io/pypi/v/git-branchless.svg)](https://pypi.org/project/git-branchless)

## Motivation

Sometimes, I am working on multiple changes to a [Git] repository.  I want
to add all of my changes to a single branch, but send them upstream in
small, reviewable chunks. Refer to [the related articles](#related-articles)
for some advantages of this workflow.

Git already supports this workflow via [git format-patch] and [git send-email],
however, many projects prefer to receive patches as pull requests.  To make
proposed changes easy to review, you'll want to submit a separate pull
request for each independent change.  With a branchless workflow, the sole
local branch typically contains multiple independent changes. To submit
those upstream as pull requests, you need to create a separate branch for
each change.  Running `git branchless` creates the desired branches without
requiring you to switch back and forth between branches. This allows you
to submit small pull requests while enjoying the benefits of a branchless
workflow. After making any changes to your worktree's branch you can easily
update the generated branches: just re-run `git branchless`.

## Installation

```sh
$ pip install --user git-revise@git+https://github.com/mystor/git-revise.git@e27bc1631f5da6041c2fa7e3d1f5a9fecfb3ef57
$ pip install --user git-branchless
```

## Usage

Create some commits with commit messages starting with `[<topic>] ` where
`<topic>` is a valid branch name.  Then run `git branchless` to create a branch
`<topic>` with the given commits.

For example, if you have created a commit history like

    $ git log --graph --oneline
    * 2708e12 (HEAD -> master) [my-awesome-feature] Initial support for feature
    * c6dd3ab [my-awesome-feature] Some more work on feature
    * 683de4b [some-unrelated-fix] Unrelated fix
    * 3eee379 Local commit without topic tag
    * 2ec4d51 (origin/master) Initial commit

Then this command will (re)create two branches:

    $ git branchless
    $ git log --graph --oneline --all
    * 2708e12 (HEAD -> master) [my-awesome-feature] Initial support for feature
    * c6dd3ab [my-awesome-feature] Some more work on feature
    * 683de4b [some-unrelated-fix] Unrelated fix
    * 3eee379 Local commit without topic tag
    | * 7645890 (my-awesome-feature) Initial support for feature
    | * e420fd6 Some more work on feature
    |/
    | * d5f4bb2 (some-unrelated-fix) Unrelated fix
    |/
    * 2ec4d51 (origin/master) Initial commit

By default, `git branchless` looks only at commits in the range
`@{upstream}..HEAD`.  It ignores commits whose subject does not start with
a topic tag.

Created branches based on the common ancestor of your branch and the upstream
branch, that is, `git merge-base @{upstream} HEAD`.

To avoid conflicts, you can specify dependencies between branches.
For example use `[child:parent1:parent2]` to base `child` off both `parent1`
and `parent2`. The order here does not matter because it will be determined
by which topic occurs first in the commit log.

By default, when dependencies are added to generated branches, the commit
message will include their topic tags. You can turn this off for all branches
with the `--trim-subject` option, or for a single dependency by adding the
`+` character before a dependency specification (like `[child:+parent]`).

If there is a merge conflict when applying a commit, you will be shown
potentially missing dependencies. You can either add the missing dependencies,
or resolve the conflict. You can tell Git to remember your conflict resolution
by enabling `git rerere` (use `git config rerere.enabled true; git config
rerere.autoUpdate true`).

Instead of the default topic tag delimiters (`[` and `]`), you can
set Git configuration values `branchless.subjectPrefixPrefix` and
`branchless.subjectPrefixSuffix`, respectively.

## Integrating Commits from Other Branches

You can use [git-branchless-pick](./git-branchless-pick) to integrate
other commit ranges into your branch:

```sh
$ git branchless-pick ..some-branch 
```

This starts an interactive rebase, prompting you to cherry-pick all
missing commits from `some-branch`, prefixing their commit subjects with
`[some-branch]`.  Old commits with such a subject are dropped, so this
allows you to quickly update to the latest upstream version of a ref that
has been force-pushed.

Here's how you would use this to cherry-pick GitHub pull requests:

```sh
$ git config --add remote.origin.fetch '+refs/pull/*/head:refs/remotes/origin/pr-*'
$ git fetch origin
$ git branchless-pick $(git merge-base origin/pr-123 HEAD)..origin/pr-123
```

## Tips

- You can use [git revise] to efficiently modify your commit messages
  to contain the `[<topic>]` tags. This command lets you edit all commit
  messages in `@{upstream}..HEAD`.

  ```sh
  $ git revise --interactive --edit
  ```

  Like `git revise`, you can use `git branchless` during an interactive rebase.

- [`git-autofixup`](https://github.com/torbiak/git-autofixup/) can eliminate
  some of the busywork involved in creating fixup commits.

## Related Articles

- In [Stacked Diffs Versus Pull Requests], Jackson Gabbard
  describes the advantages of a patch-based workflow (using [Phabricator])
  over the one-branch-per-reviewable-change model; `git-branchless` can be used
  to implement the first workflow, even when you have to use pull-requests.

- In [My unorthodox, branchless git workflow], Drew
  DeVault explains some advantages of a branchless workflow.

## Peer Projects

- [Stacked Git](https://stacked-git.github.io/) implements a similar
  workflow. It provides a comprehensive set of commands to manage commit
  metadata, whereas `git branchless` only offers one command and requires
  the user to use standard Git tools for everything else.

## Contributing

Submit feedback at <https://github.com/krobelus/git-branchless/> or at the
[public mailing list](https://lists.sr.ht/~krobelus/git-branchless) by
sending email to <mailto:~krobelus/git-branchless@lists.sr.ht>.

[Git]: <https://git-scm.com/>
[git revise]: <https://github.com/mystor/git-revise/>
[git format-patch]: <https://git-scm.com/docs/git-format-patch>
[git send-email]: <https://git-send-email.io/>
[Stacked Diffs Versus Pull Requests]: <https://jg.gg/2018/09/29/stacked-diffs-versus-pull-requests/>
[My unorthodox, branchless git workflow]: <https://drewdevault.com/2020/04/06/My-weird-branchless-git-workflow.html>
[Phabricator]: <https://www.phacility.com/>
