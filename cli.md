# Proposed CLI For A Distributed SCM (With Optional Central Repo)

Distributed VCSes are extremely powerful.

However, most people who use DVCSes use them in a semi-centralized manner,
where some network-accessible repository is treated as the canonical source of
truth.

Git (and most other DVCSes, AFAIK) treat that canonical repository as a largely
informal construct.

I'm curious what a DVCS would look like if explicitly designed to provide
opt-in support for a 'canonical' repository, and to leverage it in making
programmers' lives better as much as feasible.

To think about what that design might look like, I'm trying to outline a
hypothetical CLI for such a system.


## Parent Command

For the moment, I'm assuming the parent command for all basic commands is
`vcs`.

There must be a better name for a VCS, but until I've thought of a good one I'm
going with the generic placeholder.

If run without arguments, it should trigger the `help` command.


## Help

    $ vcs help

should give you a brief overview of the system's purpose and design then point
you to the beginner's tutorial and the reference documentation.

    $ vcs help <command>

should output the documentation for that command.

`--help` should also trigger help for the passed command, as it is an ancient
convention whose absence would surprise some people.

Similarly,

    $ vcs <command> --help

should display the command's documentation.


## Commands

    $ vcs commands

should output a list of all built-in commands, with one-line summaries of each
command.


## Create New Repository

    $ vcs create <repo/path>

should create a new repository in the given directory, creating the folder if
it does not exist.


## Check Out Repository State

    $ vcs edit <repo_url> [repo_path] [commitish [filepath]]

The `edit` command is roughly analogous to Subversion's `checkout` command.

It sets up a folder for making changes to

`repo_url` is a URL to a central repository (which, it should be noted, must
declare itself as such).
