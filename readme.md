# Version Control Is Not A Solved Problem

As a programmer, version control is a fundamental part of how I approach my
daily work.

Git is my current default choice for the many things it gets right, but I
believe it has many superficial flaws (which could be solved with a little
effort) and some fundamental flaws (which by definition would be harder to fix,
though in recent years it has made strides towards improving those).

It is my belief that a significantly better version control system is possible.
If the one I want is out there, I haven't found it so far.

I'm particularly curious as to whether a design is possible that fundamentally
privileges the "canonical central repo" on the surface and as the standard,
default configuration, while still supporting Git-style distributed mechanisms
under the hood.

I would also like a system that handles large and unmergeable files better than
git does, while retaining the many things git does wonderfully with source
code.

If you know of an open-source SCM that meets those criteria, *please* let me
know. I love discovering amazing new tools. PlasticSCM sounds interesting, but
I know no one who has actually used it, and it is closed source.


## Things Git Does Well

It treats commits as snapshots of the project. Most VCSes do that now, but ones
that version per-file (like CVS) are worthless for examining past project states.

It makes basic branching and merging easy. I did not understand branches until
I used Git, and now I can hardly work without them.

Since you can do everything without the server after a clone (and in a very
real sense there isn't a server), version control work becomes a fast, offline
operation, making it painless to commit early, commit often, and rewrite those
commits until you've shared them.

Repository creation and maintenance is so non-existent that I don't even think
before starting a new one.

It does a good job of preserving your data. If it's been committed, it's pretty
darn safe, and if it's been pushed to a remote, it's *very* safe.

It provides powerful tools for rewriting commits, up to and including the whole
repository. Some argue that's a bad thing, but as one who has used svndump to
edit badly botched commits that were breaking entire workflows, I think it's
very useful to have the option (and when working locally, it can expose totally
new workflows you'd never have tried otherwise).


## Things Git Does Poorly

### Big Files

It's very slow when dealing with huge files, especially a lot of them that
change frequently. I have not personally encountered this problem much, but I
have seen the beginnings of it, and know from other accounts what happens if
you try to keep game assets like 3D models in it.


### User Interface Design

Its CLI is inconsistent, confusing, and hard to remember. Highlights include:

* `git checkout` both being used for safe operations like changing branches and
  creating new branches, and also for deadly operations like discarding
  uncommitted work (idea: discarding uncommitted work should automatically
  create a backup commit containing the exact state you just discarded - the
  usual GC operations would eventually clean it up if you don't need to dig it
  out of the reflog-equivalent a day later [further idea - the default reflog
  view should show you just states you threw out, as getting those back is why
  you're usually in the reflog especially as a newbie - maybe call that 'git
  trashcan']).


### Annotating Commits With Arbitrary Structured Metadata

Git does have some support for this by way of the 'git notes' command. That's
better than a lot of VCSes do, but what little I know about them makes me
suspect they aren't great for this purpose.

* Notes are not pushed by default. That means third-party tooling to make them
  do the right thing for any uses.

* You have to tell Git whether you want notes to follow commit rewrites or not.
  Seems to me like that should be information stored in the note's own
  metadata, rather than something it's up to each user to configure correctly.

* Notes seem to be aimed at having additional information displayed in git
  log's output, so it probably takes extra work to distinguish between "this is
  meant for a human to see by default when reading logs" and "this is meant for
  tools to work with."

* You can only have one note per commit/namespace pair, which means if multiple
  systems want to store structured information in notes, they either have to
  avoid namespace collisions, or agree on a mechanism for coordinating access /
  encoding data in a single note.


### Conflict Resolution

Compared to Subversion and some other systems I've used, Git is great at
conflict resolution.

However, it could still be a lot more helpful.

WebStorm has a much better conflict resolution UI built-in than Git does. A VCS
should have a great, graphical resolver built-in, and use it when no merge tool
is explictly configured. If the user has not defined a merge tool explicitly,
then the default conflict resolution GUI should start automatically on
conflicts and have a friendly wizard walkthrough on how to resolve conflicts.
Once you're ready to turn that off, you can do so.

And of course the naming of "theirs" vs. "ours" is infuriating, as which you're
actually dealing with can get really confusing during rebases. Oh, and the
difference between the "recursive" merge strategy with the "ours" option as
opposed to the "ours" merge strategy. The UI for all this is awful.

I believe there are actually changes to support different labels depending on
the kind of merge happening, which could make things better or worse, I guess.
Couldn't find them in a quick search, though.


## Commit Identifiers

Git's hashes-for-commit-IDs are at its core, and they're a good idea.

Centralized systems, though, don't have to use hashes. At least not for commits
merged into the main line of development on the main server. I believe
Mercurial actually has support for 'optional' monotonic commit IDs on the main
central branch.

Arguably it's a bad plan to support monotonic IDs on the main line of
development when most commits and branches will not have such IDs. I'm not sure
I'd include them myself.

That said, Git should really have thought up-front about supporting a more
future-proof hash algorithm than SHA-1. Using a multihash variant for commit
IDs might have been smart, perhaps one that results in more-human-friendly
hashes (see discussion here: https://news.ycombinator.com/item?id=23577746).


### Newline Handling

Its handling of Windows and Linux newline characters is poor. At least it was
the last time I had to deal with it, and the bad ways of doing things are
almost certainly still there due to backwards compatibility.


### Rewriting History

Git doesn't do this "badly", per se. It supports rewriting unpublished history
better than most VCSes, and even enables whole-repo rewrites (useful for
purging things like huge binaries when you realize committing them in a Git
repo was a mistake).

However, its tools for performing rewrites are incomplete, confusing, badly
documented, and unsafe.

A few examples:

* There was no way to rebase a whole subtree of branches in one shot last I
  checked.

* There should be an easier way to say "I want to break a commit into multiple
  commits" than this: https://stackoverflow.com/a/6217314/1128957

* I would love a simple way to say "add this patch to an older commit in my
  private branch", something more discoverable and straightforward than
  `--fixup` / `--autosquash` (I think I want a command that looks like `git
  squash <into-ref> [<source-ref>]`, where if <source-ref> is not passed the
  staged patch is squashed into it). Magit's instant fixup behavior is pretty
  great, and is more or less what I wish git itself had built in.

* It took me quite a few readings of the rebase manpage to even grok its most
  basic usage when I first learned about it. The concept is a bit subtle, but
  the manpage is (was?) inscrutable.

* Instead of *documenting* that you shouldn't rewrite pushed history, why not
  *remember* that I pushed it and warn me if I try? Similarly, if I just
  rewrote a shared branch, it should warn me if I then try to merge the old
  commits from the remote - I almost certainly didn't mean to do that [saw this
  happen to a rebase newbie recently].

* While it warns you about the complexities of rewriting shared branches, it
  has no internal mechanism to mark branches as 'private' or 'public', and no
  way for you to know if an open branch has become a shared branch. In the
  cases where you have a de-facto central server, it could keep track of that -
  if it did, the warnings about rewriting shared history could be factored into
  the tool itself, in cases where you're rewriting history that is not known to
  be private (when you've pushed a new branch to central but not marked it as
  private, your local checkout should treat it as 'possibly shared').

* git push --force-with-lease should be the default behavior, since for history
  that's not yet canonical, it's the right thing. History that's not yet
  canonical is, of course, the 'private' branch idea I was whining about above.
  Git doesn't have them, but it's possible to sort of hack in protected
  branches by pre-receive hooks (and a private branch is almost like an
  unprotected branch, weird as that sounds). This one is basically an accident
  of history - push got --force before it got the much more reasonable
  --force-with-lease.


### Renaming Branches

Rename a local branch. If you have a remote configured, it stays untouched, and
as far as I know the only way to "rename" it is to push a new one then delete
the old one, which of course isn't actually a rename as anyone who had your old
branch checked out now cannot find it (never mind the chaos if you actually had
a merge request or code review happening on that branch).


### Preserving Branch History

It does not have any concept of 'archiving' branches; after merging, you're
expected to delete them, thereby losing useful historical information (if you
don't want to be pestered by seeing that branch in the UI all the time). I hear
Mercurial's data structures handle this differently, and in a way some people
prefer.

The same is true of tags - although there are mechanisms to only see tags
matching a certain pattern, there are no defaults in place for that, so you
drown in noise if you don't know how not to.


### Lazy-Loading History

It technically supports not retrieving history ('shallow clones' in git
parlance), but not especially cleanly, transparently, or discoverably (TODO
figure out/explain how I wish this worked). The monorepo style of working is
infeasible without this feature, in extreme edge cases like Google/FB scale.

Similarly, it has support for sparse checkouts, but as far as I know it's a
bolted on feature that doesn't have a good UI (see for instance this thread,
where the basic use case takes apenwarr to provide correct answer:
https://stackoverflow.com/a/4909267/1128957), and it probably doesn't supply
all the features people want (e.g. ACL on subdirectories). If I recall
correctly, it doesn't handle the biggest reason people want this feature, which
is to just plain *not have* the directories you didn't check out on disk
anywhere (including your VCS metadata). Again, for monorepos you need that.

Amendment - in 2.19.0, Git [started growing support for a `partial-clone`
command](https://git-scm.com/docs/partial-clone), and it's been slowly
improving that since. Looks like it might become what I want in the next year
or three.


## Ideal VCS Features

These aren't so much deficiencies in git as things it was never intended to do.

A lot of these are centered around my belief that while distributed VCSes are
really useful, in reality most teams *want* a centralized node that is the Law
and the Prophets.

The distributed stuff is mostly about programmer convenience and making it
easier to build power tools for doing stupid VCS tricks like history rewrites.


### Prescient Conflict Warnings

Git has no concept of file locking.

File locking is questionable if your format is merge-friendly and diffable.

If it is not, about the best thing you can do is say "warning; I'm working on
this."

Some hooks for interactions with remotes and an agreed-upon central locking
service would let you implement this - 'locked' files can be marked read-only
in your working copy, and we can include a mechanism for seeing who locked it
(and what repo is the source of the lock).

Obviously true locking is inherently incompatible with being distributed, but
all we really want to do here is let people know 'hey, someone else is working
on this.'

You could take it a step further and auto-lock unmergeable files as soon as the
VCS sees a change to one (it should warn you it did so and offer you the chance
to undo/unlock it). Furthermore, distributed VCSes would offer the locking
mechanisms you actually want - if you know someone left for the day and just
locked things by accident, you can leave a sticky note on their desk and ignore
the lock. The VCS can leave that note for you.

These are less "locks" and more automatic warnings if someone else starts
editing an unmergeable file. Note that gitolite has implemented this (though
not quite how I would do it, but refusing pushes for locked files is
interesting - would be nice to notify people about conflicting locks on fetches
and file modification, too. Lock messages akin to commit messages might be
interesting): http://gitolite.com/gitolite/locking.html

Insight: VCS locks aren't about controlling access to files. They're about
facilitating communication about who's doing what.

Files that can't be merged should *always* trigger a warning if two edits might
result in a conflict, but you could do higher-res warnings based on filetype
and diffs if you had language-level parsing - "you're calling a function
someone else is currently changing internals in".

That would be so sweet.


### Smarter Merging And Diffing

It's perfectly possible to teach your VCS how to merge custom file formats. Git
already supports it.

So, a really *good* VCS would ship with smart merging and diffing engines for
major programming languages (and maybe even other file formats - images, at
least, can be diffed sanely), and make it easy to write your own mergers and
differs.

Such tools have been written - the makers of PlasticSCM sell some, I think, but
I don't recall seeing general-case OSS tools for this.

This obviously should use the same language-awareness plumbing I proposed for
the smarter layer on top of the file locking system above.

Wonder if it would be possible to use [LSP
backends](https://langserver.org/) as infrastructure to magically get support
for diffing and merging arbitrary languages? Probably wouldn't quite work, but
it's the germ of an idea. With the right language-agnostic API this is possible
- whether that API already exists is the unknown for me here.
[tree-sitter](https://tree-sitter.github.io/tree-sitter/) is probably the right
basis for this, and in fact [difftastic](https://github.com/Wilfred/difftastic)
is a project that does exactly this to handle diffing.

[This paper on comparing Git's diff
algorithms](https://link.springer.com/article/10.1007/s10664-019-09772-z) has a
bibliography well worth mining - it has pointers to several papers on advanced
diffing engines, some based on language analysis and/or ASTs.

I just realized that the proper name for a tool that handles the
syntax-aware merging side of this equation is obviously `synmergey`.

...I'm so sorry. It had to be done.


### BitTorrent As Transport

So, you have a distributed VCS. Wouldn't it be nice if you could use BitTorrent
(or similar) as a transport?

Your 'central' server could act as the basic tracker, putting all the clients
on the network in touch with each other, and serving as a source guaranteed to
have every object. Lot of design work to hash out, but the core concept is
interesting, I think.

As with so many cool ideas, someone built this a long time ago:
https://github.com/cjb/GitTorrent

His spin isn't quite the one I'm thinking of, but I'd guess he's done all the
hard work to achieve what I would like.


### Branches As First-Class Citizens

While they're way closer than in many previous VCSes, Git's branches are
actually still second-class compared with, say, commits or tags, as noted
elsewhere in this doc:

- You are expected to delete them after merge (which it doesn't even have the
  grace to do for you - Bob pointed out that it really should [`git branch
  --merged` and a post-pull hook could do that])

- They [do not support shared description metadata](https://stackoverflow.com/a/8858853/1128957)

- They cannot be natively marked as 'private' vs. 'public', which would be
  really helpful in automatically deciding whether you should be allowed to
  push rewritten history to the branch or not (and perhaps also in deciding who
  should be allowed to access the branch?).

- They cannot serve as the anchor point for multiple passes at creating a
  patchseries (see commit message hacks in Gerrit for keeping all the rebased
  versions of a branch linked up)

There might be more we can do here than just those things, too, but those would
certainly be a start.


### Distributedish Task Tracking

If you kept your project's task data in the repo, as various distributed issue
tracking tools will do, you can treat the canonical repo's data as the source
of truth.

That's probably a job for an entirely different system (centralized UI that
automatically commits new submissions and relies on a diff/merge driver to make
sure it never conflicts), but I thought it was worth noting, as it could be
relevant in the VCS itself. A VCS is fundamentally a collaborative tool, so
keeping track of the work to be done could certainly be in its purview.


### Commits As Souped-Up Undo History

I use undo-tree in Emacs to have infinite undo that works sanely.

Judicious application of WIP commits and Git's 'squash' concept could let you
treat the VCS as a persistent, crash-proof, project-wide undo system
(especially if you auto-push such temporary commits to other machines in the
background). It might only make sense to auto-commit when a build runs
successfully? Or perhaps auto-commit aggressively, but only advertise the
commits with a successful build? Would need to think about marking states
explicitly as 'checkpoints', because huge undo histories make it hard to find
the last point you wanted to be at. Just namespaced tags for doing that?


### Post-Clone Hooks

Git has these, but they don't actually do what I want in practice, which is to
say "first time someone clones this repo, run this command - don't make them
think about sandbox setup."

What I'd like is an option to say, basically, "I *highly* trust this server I'm
about to clone from - let it do whatever insanity it wants." If that option's
not set, interactively ask permission to do whatever it wants when a repo
specifies a post-clone hook.

That way, workplace repositories and the like could kick off sandbox setup
immediately after the clone, instead of requiring you two run two commands.

That may sound silly and pedantic, but it would make standardizing project
setup massively easier in SOAs a la Amazon - checking out the project *gets*
you a standard instance of whatever service you need (and presumably registers
it in your local sandbox instance of the service discovery system, so your
other sandboxes know to point at it by default).


### Built-In Repo Settings

Git is, well, too distributed in how it handles certain configuration and
client options.

From a highly-trusted, centralized server, much as I want hooks to be installed
after clones/pulls/rebases/etc, I also want any number of VCS configurations to
be installed automatically.

Yes, many things can be put into files in the repo (`.gitattributes` et al),
but not all settings can - things that would wind up in `.git/`,
`~/.gitconfig'`, or similar require user intervention to get there.

Conceptually an answer to the hook problem lets you solve this, but you'd want
to be careful to make the UX nice and smooth. It would not be hard to
technically support this but do so in an annoyingly clumsy manner.


### Handling Cryptographic Signatures

Linus and some other experts disagree on how cryptographic signatures should be
handled in git.

Linus says that signing release tags is the only reasonable thing, while other
people think all commits should be signed.

I'm not sure who is right, but I wondered recently if perhaps support for
signing things besides just commits might be useful.

I could see signing a tree (folder, for those who don't speak git) as part of a
code review process.

The advantage is that you can then rebase without losing your approval if the
rebase turns out to be trivial.

OTOH, you could argue that as a downside, too - invisible semantic conflicts
are absolutely possible, as is sneaking in bad commits to the history, so maybe
reviewers should have to re-review after a rebase.

Guess if you want that policy you could require all root trees in a branch to
be approved. A different way to say that is "each individual patch in a merge
request must be approved".

And with my hypothetical semantic conflict detector, I suppose you could
automatically unapprove a rebase that might introduce a change in semantics.
That would be pretty darned cool.

...I guess really all rebases and merges should check for semantic conflicts
and warn you if they're possible (i.e., if any abstractions you've used have
undergone a change since you last merged/rebased).


### Remember, Cache, and Share Record Of Commits To Skip When Bisecting

git-bisect is an awesome tool.

Sometimes it fails to be *quite* as awesome as it could be, when you land on
commits that won't build or which break a bunch of tests. You have to manually
punt and move to another commit.

Some teams introduce heavy-duty process constraints like "No commit may be
merged that breaks the build or breaks any tests," because they love being able
to have git bisect automatically tell them when a commit under review
introduces regressions.

The VCS should support storing commit metadata about bisecting - whether a
commit breaks the build entirely, as well as a list of tests it's known to
break.

If your repo contains a build script, you could have your CI/CD pipeline notify
your VCS about commits that break the build (and should therefore be skipped in
a bisect) and tagging them thusly for you (when it has no higher-priority jobs
to do).

Humans should be able to tag commits in the same way, of course, interactively.
That metadata could be lazy-loaded from the central repo as needed during a
bisect. You could even recruit developer machines as nodes to do that
background processing to generate commit metadata.

Pretty sure this idea could use more refinement, but I think the core of it is
useful and interesting.

I think it would be feasible to teach it to detect failed tests, too.


## Git-based POC?

I think these ideas, once better defined, could be proofed out with git annex +
bup for big binaries, gittorrent, some clever aliasing,a buttload of hooks
(perhaps managed by my unfinished [githooks package
system](https://github.com/NateEag/githooks.d)), and maybe writing a few new
commands if it can't be avoided.

Could be fun.

...or complete madness. You decide.


### Git-Annex For Large Files

If you could make git-annex work transparently with the regular git commands,
and use bup as your annex backend store, you could be in a pretty good place
for dealing with large files. I'd have to actually try it to know if it's at
all true.

Unsurprisingly, I'm not the only one who's thought about this - the git-annex
maintainers are trying to adapt smudge and clean filters to do this job. It's
not clear to me if it's actually working yet, but it appears to be an area of
active research:

https://git-annex.branchable.com/todo/smudge/


## Useful Articles About VCS Usage Generally

https://mikkel.ca/blog/git-is-my-buddy-effective-solo-developer/ - some great
insights into how to use git effectively. Has some particularly useful insights
into commit hygiene and rewriting branches as you go.


## Other People's Thoughts

I am definitely not the first person to express the idea that the ideal VCS has
not yet been written. Here are a few such expressions:

http://www.chriskrycho.com/2014/next-gen-vcs.html

http://www.scopelift.co/blog/life-after-git

https://bitquabit.com/post/unorthodocs-abandon-your-dvcs-and-return-to-sanity/

https://blogs.janestreet.com/centralizing-distributed-version-control-revisited/

http://www.ianbicking.org/distributed-vs-centralized-scm.html

This HN discussion of GitHub's LFS announcement was mildly informative (mostly
it was just people yelling about "why didn't you just use git-annex?"). I'm
curious to learn what a 'Merkle tree' is, which someone mentioned in
association with rolling hashes for storing large binary files:
https://news.ycombinator.com/item?id=9343021

A lovely summary of what Git's missing for enterprise work, courtesy of someone
at Perforce:
https://web.archive.org/web/20150503044832/https://www.perforce.com/blog/140211/how-git-could-grow-enterprise-scm-system
(originally hosted at
https://www.perforce.com/blog/140211/how-git-could-grow-enterprise-scm-system
but has disappeared since)

This chat between ESR and the author of monotone was interesting:
http://www.catb.org/esr/writings/version-control/version-control.html#graydon

Similarly, this Reddit thread has a few interesting comments in it:
https://www.reddit.com/r/programming/comments/2xxb17/git_is_actually_ok_for_some_binaries/

Someone who put some decent thought into this:
http://tonsky.me/blog/reinventing-git-interface/

Someone who actually built a layer on top of git: http://gitless.com/

Legit adds a few commands to git intended to make it easier for humans to use.
https://frostming.github.io/legit/

And Microsoft announced a VFS for git, so that actual files can be fetched only
as needed:
https://blogs.msdn.microsoft.com/visualstudioalm/2017/02/03/announcing-gvfs-git-virtual-file-system/

The HN discussion of that release had many interesting comments, including
someone who said that v6 of git annex is almost invisible, needing only a 'git
annex sync' command that could probably be hidden via hook.
https://news.ycombinator.com/item?id=13559662

A lengthy thread on HN about why Fossil's author doesn't like Git:
https://news.ycombinator.com/item?id=16806114

A few different people have talked about monorepos vs. many repos. Some
articles I found thought-provoking:

* https://beza1e1.tuxen.de/monorepo_vcs.html - poor grammar but good thoughts
* https://danluu.com/monorepo/ - clear explanation of the benefits of monorepo

In principle, I think the ideal VCS would enable both monorepo and manyrepo
workflows and let you choose what's right for you. Perhaps that is not feasible
in reality, but so far I don't know of any genuine hard tradeoffs (like
security vs. accessibility, where the two really are inherently opposed to each
other). It certainly would increase expense and difficulty to support monorepo,
though, as you have to deal with a lot of complexity and scaling issues that
Git has historically just ignored.


## VCSes I Have Used

Git, CVS, Subversion, Mercurial (rarely), CA SCM (it's horrible)


## VCS UIs To Check Out

[jj](https://github.com/martinvonz/jj) looks like a really solid piece of work
in terms of making git more usable and learnable. I might just give it a try as
my daily driver, since it seems like a significant improvement on git in many
ways but appears to be backwards-compatible.

[bit](https://github.com/chriswalz/bit) is a nifty-looking layer on top of git.
I can't imagine it's as good as [magit](https://magit.vc/) (which is my daily
driver), but it looks like it has some cool ideas.


## VCSes I Am Vaguely Aware Of

monotone, darcs, bazaar, Perforce, Fossil, gitless, bitkeeper (someone on HN
said it is OSS now and has solved a lot of pain points for orgs that wind up
with a centralized repo in practice:
https://news.ycombinator.com/item?id=23582580)


## VCSes I Know Only The Names Of

A patch-based VCS a la darcs that might be interesting: https://pijul.org/

Got is an OpenBSD-specific VCS that appears to be a reimagining of Git, based
on the core datastructures: http://gameoftrees.org/


## License

Unless explicitly stated otherwise, the contents of this repo are under the
[Creative Commons Attribution 4.0
license](https://creativecommons.org/licenses/by/4.0/).
