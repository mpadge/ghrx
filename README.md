<!-- badges: start -->
<!---

[![R build
status](https://github.com/mpadge/ghrx/workflows/R-CMD-check/badge.svg)](https://github.com/mpadge/ghrx/actions?query=workflow%3AR-CMD-check)
[![codecov](https://codecov.io/gh/mpadge/ghrx/branch/master/graph/badge.svg)](https://codecov.io/gh/mpadge/ghrx)
[![Project Status:
Concept](https://www.repostatus.org/badges/latest/concept.svg)](https://www.repostatus.org/#concept)

--->
<!-- badges: end -->

# ghrx

An R package to document [**g**ithub](https://github.com) issues in git
via [**r**oxygen](https://roxygen2.r-lib.org).

## Why?

### The short answer

To enable a permanent, transparent, and traceable connection between
GitHub issues and code itself. Any coders who have ever typed into a
GitHub issue something like,

> The above commit fixes this issue by ….

are creating a textual record of the intention of a modification to
their code. However, such records exist solely on GitHub, and are not
recorded at all in the actual code. `ghrx` enables the git tree to
record any and all such comments within the code itself, and to be
automatically propagated to a GitHub issue in the same way.

### A slightly longer answer

Many developers use git to manage versions of their code, and use GitHub
to host their repositories, as well as to use additional GitHub-specific
features such as issues (or pull requests). The use of GitHub
nevertheless results in a repository having two primary, and largely
separate, records:

1.  The git tree (and associated data such as records of changes to
    objects); and
2.  GitHub issues, which can and often do have a life entirely separate
    from the git tree itself.

Generally the only way to connect a git tree to GitHub issues is through
references in commit messages, such as,

    git commit -m "fixes #14"

The `-m` flags that what follows will be recorded as the commit message,
and GitHub parses any numbers prefixed with hashes (`#`) as references
to issues. GitHub also parses several synonymous verbs allowing issues
to be closed via commit messages. As explained in [their
documentation](https://guides.github.com/features/issues/#notifications)
(see just above “Search”),

> One of the more interesting ways to use GitHub Issues is to reference
> issues directly from commits… By prefacing your commits with “Fixes”,
> “Fixed”, “Fix”, “Closes”, “Closed”, or “Close” … the commit … will
> also automatically close the issue.

That statement is followed with the claim that,

> References make it possible to deeply connect the work being done with
> the bug being tracked, and are a great way to add visibility into the
> history of your project.

The assertion that this “deeply connect(s)” code and issues is perhaps
overly bold; the actual connection is quite superficial, and importantly
only exists when developers actively remember to insert appropriate
references in their commit messages. The `ghrx` package enables a far
richer and enduring record to be constructed and recorded within a code
base of relationships between that code and GitHub issues.

## How?

`ghrx` uses the [`roxygen2` package](https://roxygen2.r-lib.org) to
parse custom tags in function documentation referring to, and
controlling, GitHub issues. Control is enabled by [GitHub’s own
command-line interface](https://cli.github.com/), which must be
installed in order to use this package. The package uses the following
three primary verbs.

### `ghrx open`

The following lines illustrate how to open a new issue by recording a
bug in the code at the actual place the bug arises.

``` r
#' a_function
#'
#' This function works really well.
#'
#' @ghrx open The function `a_function` actually has a bug that mucks
#' everything up completely ...
#'
#' @param input Function input
#' @export
a_function <- function (input) {
    # ... <code>
}
```

When you update your package documentation (via
[`devtools::document()`](https://devtools.r-lib.org/reference/document.html)
or the equivalent
[`roxygen2::roxygenise()`](https://roxygen2.r-lib.org/reference/roxygenize.html)),
this `@ghrx open` tag will cause a new issue to be opened on the
package’s GitHub page, and the corresponding text inserted. The opened
issue will be a template only, allowing you to add additional
information to the issue itself before actually clicking “Open”.

This issue will be opened as soon as you update the code’s
documentation, and you’ll still need to commit and push the code changes
which include the `@ghrx open` tag. Once you’ve pushed those changes to
GitHub, the next time you run
[`devtools::document()`](https://devtools.r-lib.org/reference/document.html),
that tag will be automatically updated by replacing `@ghrx open` with
`@ghrx opened #<issue_number>`, where `opened` is not a verb used by
`ghrx`, and merely indicates that the event happened in the past. That
verb may be subsequently modified or deleted.

This process is described in more detail in the “`ghrx` Workflow”
section below. For example `@ghrx open The function ...` might become
`@ghrx opened #15 The function ...` Determining corresponding issue
numbers involves some (automated) guesswork, more so if the text
ultimately posted in the issue differs considerably from that recorded
in the function documentation, and so running
[`devtools::document()`](https://devtools.r-lib.org/reference/document.html)
will also inform you of the issue number which has been inserted, and
prompt to you check its accuracy.

From that point on, the `@ghrx opened #15` tag will remain as a
reference of the point in the code base at which that issue was opened.
Additional tags referring to an issue may be inserted at any other time
and place, such as,

``` r
#' another_function
#'
#' @ghrx #15 The bug actually is actually caused by this function
```

Code which uses `ghrx` to open an issue via a `@ghrx` tag must retain at
least one copy of a `@ghrx #<issue_number>` reference to that issue
(optionally with an extra verb in between, like
`@ghrx note #<issue_nmber>`) until the issue is closed. Attempts to
update package documentation without at least one reference to any issue
which was opened by `ghrx` and has not yet been closed will fail, and
will inform you which tags are missing and need to be re-inserted.
`ghrx` also enables the additional verbs now described.

### `ghrx comment`

As indicated at the outset, GitHub itself enables interaction from git
only through six synonymous verbs used to close issues, and through
simple references (like `git commit -m "more #15"`), which inserts
hyperlinked references in a commit to a specified issue. As also stated
above, this system only works if developers remember to insert
references. While such commit messages can and indeed should still be
used (and can be ensured by using `ghrx`-specific [pre-commit
hooks](https:pre-commit.com), as described below), `ghrx` enables issue
comments themselves to be embedded within, and directly generated by,
the code itself, and thus enables the content and history of issues to
be recorded within a git tree.

The verb used to do this is `@ghrx comment`, followed by an issue
number. The following lines added to a function’s documentation will
cause the corresponding comment to automatically appear in the thread of
the nominated issue.

``` r
#' @ghrx comment #15 This commit solves part of this issue by ...
```

As with the `open` verb, running
[`devtools::document()`](https://devtools.r-lib.org/reference/document.html)
or
[`roxygen2::roxygenise()`](https://roxygen2.r-lib.org/reference/roxygenize.html)
after committing and pushing a `@ghrx comment #<issue_number>` verb will
update that line in your code to `@ghrx commented #<issue_number>`,
where `commented` is merely a place-filler that may be modified or
removed at any later stage. `@ghrx commented` lines may be removed at
any time, as long as at least one reference to the issue number remains.

As with `@ghrx open`, simply running
[`devtools::document()`](https://devtools.r-lib.org/reference/document.html)
will cause that commit to add a comment to the specified issue before
the `@ghrx comment` has actually been committed or pushed. The
associated changes can be referenced by including `"#15"` in the commit
message, resulting in a link being inserted *below* the corresponding
issue comment (in slight contrast to the currently general approach of
committing first and commenting later).

If you forget to reference the issue in your commit message, you can run
an additional `ghrx_fix_issue_links()` function to insert any missing
references directly in all issue comments generated by `ghrx`.

### `ghrx close`

`ghrx` also includes a `close` verb (with the same synonyms as GitHub,
so any of `close`, `closes`, `closed`, `fix`, `fixes`, or `fixed` will
also work) to close issues. `@ghrx close` commands can also be combined
with `close` references in commit messages. Like all other verbs,
updating documentation after pushing a commit containing a `close` verb
will change that line in your code to `@ghrx closed #<issue_number>`.
After pushing code with a `@ghrx close` command, any references to the
nominated issue may then be removed from the code.

## The `ghrx` workflow

`ghrx` actions are triggered by custom
[`roxygen2`](https://roxygen2.r-lib.org) roclets, and so happen when you
update your package’s documentation via
[`devtools::document()`](https://devtools.r-lib.org/reference/document.html)
or
[`roxygen2::roxygenise()`](https://roxygen2.r-lib.org/reference/roxygenize.html).
These GitHub actions are thus not directly triggered by git actions,
rather `ghrx` requires developers to implement corresponding git actions
such as committing and pushing immediately after `ghrx` actions have
been triggered. The package nevertheless issues appropriate prompts,
instructions, and reminders to encourage timely and appropriate git
actions to reflect and represent `ghrx` actions. It is also possible to
enforce strict adherence to this workflow through using
[`pre-commit hooks`](https://pre-commit.com), as described below.

The primary workflow is:

1.  Add [`roxygen2`](https://roxygen2.r-lib.org)-style `#' @ghrx`
    commands to your function documentation.
2.  Trigger corresponding actions on GitHub by updating your package
    documentation. `ghrx`’s roclets will also prompt you to perform the
    corresponding git actions.
3.  Perform those git actions (generally commit and push).
4.  Update your documentation once again.

The final step is important, and will automatically modify `ghrx` verbs
in your code to equivalent past tense forms (so `@ghrx open` will become
`@ghrx opened`, `comment` will become `commented`, and `close` will
become `closed`). This means that `ghrx` will modify your code! It is up
to you as a developer to ensure that these automatic modifications are
appropriate or desirable. After your code has been modified in response
to running
[`devtools::document()`](https://devtools.r-lib.org/reference/document.html)
or
[`roxygen2::roxygenise()`](https://roxygen2.r-lib.org/reference/roxygenize.html),
we recommend an additional commit purely to record these modifications
or removals (so yes, your git tree may be replete with commit messages
of `"remove ghrx commands"`).

The only verbs recognised by `ghrx` are `open`, `comment`, and `close`.
The latter two must be given in the form
`@ghrx comment/close #<issue number>`. When not using those verbs, any
other words or phrases may be inserted between a `@ghrx` statement and a
reference to an issue number. `ghrx` will then search for the first hash
symbol (`#`), and presume the number immediately following that to be a
reference to an issue number. The following would thus be a valid `ghrx`
statement:

``` r
#' @ghrx This modification attempts to fix #15
```

### `ghrx` and `pre-commit hooks`

Tighter integration of `ghrx` and `git` can be achieved by the use of
[`pre-commit hooks`](https://pre-commit.com). Pre-commit hooks perform
specified actions on a git repository prior to each commit, and can be
used for almost any conceivable purpose. The R package,
[`precommit`](https://github.com/lorenzwalthert/precommit/) implements
several hooks useful for R package development. The `ghrx` package has
four hooks which can be used to ensure that,

1.  Any commits which add `@ghrx` commands also reference the
    corresponding issues in commit messages;
2.  Commits which include `@ghrx open/comment/close` commands must be
    followed by subsequent commits which modify those verbs to past
    tense, or remove them from the code;
3.  git commit messages which close an issue
    (`git commit -m "closes #<issue>@`) which was opened by `ghrx` must
    include a `@ghrx close` command; and conversely,
4.  Commits to which a `@gtfx close` command was added must include a
    commit message which also closes the issue.

## `ghrx` reports

Because `ghrx` records issue interactions within the code itself, the
git tree holds an integrated record of all code associated with a given
issue, and many textual explanations of modifications made in relation
to that issue. `ghrx` also includes functions to extract all of that
information in order to produce integrated overviews of GitHub issues,
including the entirely of an issue beyond just those comments generated
by `ghrx`, interwoven with all corresponding modifications to the code.
Reports can be generated at any time in an issue’s life cycle by simply
running,

``` r
ghrx_report (<issue_number>)
```
