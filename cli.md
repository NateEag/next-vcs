# Proposed CLI For A Distributed SCM (With Optional Central Repo)

Distributed VCSes are extremely powerful.

However, most people who use DVCSes use them in a semi-centralized manner,
where some network-accessible repository is treated as the canonical source of
truth.

Git (and msot other DVCSes, AFAIK) treat that canonical repository as a largely
informal construct.

I'm curious what a DVCS would look like if explicitly designed to provide
opt-in support for a 'canonical' repository, and to leverage it in making
programmers' lives better as much as feasible.

To think about what that design might look like, I'm trying to outline a
hypothetical CLI for such a system.
