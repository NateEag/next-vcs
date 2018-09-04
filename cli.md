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


## Tutorial

    $ vcs tutorial

should begin an interactive training session on how to use the system.

I have many thoughts on what that could look like, but the core of it is having
exercises defined as known states you're supposed to reach and checking whether
you've reached them or not (along with showing the 'ideal' solution to the
problem).

Note that this could be implemented by cherry-picking important tests for core
features from the functional test suite and running them each time the user
takes a step (or explicitly asks for a grade).


## Commands

    $ vcs commands

should output a list of all built-in commands, with one-line summaries of each
command.


## Create New Repository

    $ vcs create <repo/path>

should create a new repository in the given directory, creating the folder if
it does not exist.


## Check Out / Create Repository

    $ vcs edit <repo_path> [repo_url] [commitish [filepath]]

The `edit` command is roughly analogous to Subversion's `checkout` command. I
also do not love its name.

It configures a folder on your local filesystem for versioning content.

`repo_path` is the local directory you would like the new repository to live
in. If it does not exist, it will be created. If it exists but is not a
repository, it should suggest you try running the 'create' command to begin
versioning the directory's contents.

`repo_url` is an optional URL to a central repository (which, it should be
noted, must declare itself as such). If passed, the new checkout will treat the
repository at `repo_url` as its canonical repository. Thus, if `repo_path` does
not exist, it will be populated from this URL. If `repo_path` exists, it will
assume


## Move Commit

   $ vcs move-commits <commit IDs> [branch]

Remove specified commits from any branches they are currently included in and
append them to the target branch, which defaults to the current branch if not
given.

This is not a core command, nor have I thought through the interface carefully
(which presents several subtle difficulties, such as - how do we specify
branches, especially remote ones? How do we specify a range [or multiple
ranges] of commits?).

It's here to remind me of what I frequently want when I have to reach for the
chainsaw which is `git rebase`, or the not-quite-right tool which is `git
cherry-pick`. Often enough I've just made a commit or two on the wrong branch
without thinking, and I just want to move them.

This should, obviously, refuse to operate on shared and/or protected branches
(and there's another subtlety I haven't thought about yet - how and when we
mark branches and commits as "public history", as opposed to "work in progress
that's been auto-pushed to all available repos on the network for redundancy")
