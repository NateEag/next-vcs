# Version Control Is Not A Solved Problem

As a programmer, version control is a fundamental part of how I approach my
daily work.

Git is my current default choice for the many things it gets right, but I
believe it has many superficial flaws (which could be solved with a little
effort) and some fundamental flaws (which by definition would be much harder to
fix).

It is my belief that a significantly better version control system is possible.
If the one I want is out there, I haven't found it so far.

I'm particularly curious as to whether a design is possible that can handle
large and unmergeable files better than git does, while retaining the many
things git does wonderfully with source code.

I would like, therefore, to think about what such a VCS might look like.

If you know of one that sounds like a good fit, *please* let me know. I love
discovering amazing new tools. PlasticSCM sounds interesting, but I know no one
who has actually used it, and it is closed source.


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

It's very slow when dealing with huge files, especially a lot of them that
change frequently. I have not personally encountered this problem much, but I
have seen the beginnings of it, and know from other accounts what happens if
you try to keep game assets like 3D models in it.

Its CLI is inconsistent, confusing, and hard to remember.

While it supports rewriting unpublished history better than most VCSes, and
even enables whole-repo rewrites (useful for purging things like huge binaries
when you realize committing them in a Git repo was a mistake), its tools for
performing rewrites are incomplete (there was no way to rebase a whole subtree
of branches in one shot last I checked), badly documented (it took me quite a
few readings of the rebase manpage to get it when I first learned about it),
and unsafe (instead of documenting that you shouldn't rewrite pushed history,
why not *remember* that I rewrote it and warn me if I try?).

It does not have any concept of 'archiving' branches; after merging, you're
expected to delete them, thereby losing useful historical information (if you
don't want to be pestered by seeing that branch in the UI all the time). I hear
Mercurial's data structures handle this differently, and in a way some people
prefer.

Similarly, while it warns you about the complexities of rewriting shared
branches, it has no internal mechanism to mark branches as 'private' or
'public', and no way for you to know if an open branch has become a shared
branch. In the cases where you have a de-facto central server, it could keep
track of that - if it did, the warnings about rewriting shared history could be
factored into the tool itself, in cases where you're rewriting history that is
not known to be private (when you've pushed a new branch to central but not
marked it as private, your local checkout should treat it as 'possibly
shared').


The same is true of tags - although there are mechanisms to only see tags
matching a certain pattern, there are no defaults in place for that, so you
drown in noise if you don't know how not to.

It technically supports not retrieving history ('shallow clones' in git
parlance), but not especially cleanly, transparently, or discoverably (TODO
figure out/explain how I wish this worked). The monorepo style of working is
infeasible without this feature, in extreme edge cases like Google/FB scale.

Similarly, it has support for sparse checkouts, but as far as I know it's a
bolted on feature that doesn't have a good UI (see for instance this thread,
where the basic use case takes apenwarr to provide correct answer:
https://stackoverflow.com/a/4909267/1128957), and it probably doesn't supply
all the features people want (e.g. ACL on subdirectories).

It does not provide file locking. Yes, file locking is stupid if your format is
merge-friendly and diffable, but if it is not, about the best thing you can do
is say "warning; I'm working on this." Some hooks for interactions with remotes
and an agreed-upon central locking service would let you implement this -
'locked' files can just be marked read-only in your working copy. Obviously
true locking is inherently incompatible with being distributed, but all we
really want to do here is let people know 'hey, someone else is working on
this.' Furthermore, distributed VCSes would offer the locking mechanisms you
actually want - if you know someone left for the day and just locked things by
accident, you can leave a sticky note on their desk and ignore the lock. Note
that gitolite has implemented this (though not quite how I would do it, but
refusing pushes for locked files is a good idea - would be nice to notify
people about new locks on fetches, too):
http://gitolite.com/gitolite/locking.html


## Git-Annex For Large Files

If you could make git-annex work transparently with the regular git commands,
and use bup as your annex backend store, you could be in a pretty good place
for dealing with large files. I'd have to actually try it to know if it's at
all true.

Unsurprisingly, I'm not the only one who's thought about this - the git-annex
maintainers are trying to adapt smudge and clean filters to do this job. It's
not clear to me if it's actually working yet, but it appears to be an area of
active research:

https://git-annex.branchable.com/todo/smudge/


## BitTorrent As Transport

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


## Smarter Merging And Diffing

It's perfectly possible to teach your VCS how to merge custom file formats.
Pretty sure Git already supports it.

So, a really *good* VCS would ship with smart merging and diffing engines for
major programming languages (and maybe even other file formats - images, at
least, can be diffed sanely), and make it easy to write your own mergers and
differs. Such tools have been written - the makers of PlasticSCM sell some, I
think, but I don't recall seeing general-case OSS tools for this.


Diffing is another algorithm, obviously.


## Commits As Undo History

I use undo-tree in Emacs to have infinite undo that works sanely.

Judicious application of WIP commits and Git's 'squash' concept could let you
treat the VCS as a persistent, crash-proof, project-wide undo system
(especially if you auto-push such temporary commits to other machines in the
background). It might only make sense to auto-commit when a build runs
successfully? Or perhaps auto-commit aggressively, but only advertise the
commits with a successful build? Would need to think about marking states
explicitly as 'checkpoints', because huge undo histories make it hard to find
the last point you wanted to be at. Just namespaced tags for doing that?


## POC This On Top Of Git?

Starting to think these ideas, once better defined, could be proofed out with
git annex + bup for big binaries, gittorrent, some clever aliasing, actually
writing code when it can't be avoided, and a buttload of hooks, perhaps
managed by my
unfinished [githooks package system](https://github.com/NateEag/githooks.d).

Could be fun.

...or complete madness. You decide.


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
https://www.perforce.com/blog/140211/how-git-could-grow-enterprise-scm-system

This chat between ESR and the author of monotone was interesting:
http://www.catb.org/esr/writings/version-control/version-control.html#graydon

Similarly, this Reddit thread has a few interesting comments in it:
https://www.reddit.com/r/programming/comments/2xxb17/git_is_actually_ok_for_some_binaries/

Someone who put some decent thought into this:
http://tonsky.me/blog/reinventing-git-interface/

Someone who actually built a layer on top of git: http://gitless.com/

And Microsoft just announced a VFS for git, so that actual files can be fetched
only as needed:
https://blogs.msdn.microsoft.com/visualstudioalm/2017/02/03/announcing-gvfs-git-virtual-file-system/

The HN discussion of that release had many interesting comments, including
someone who said that v6 of git annex is almost invisible, needing only a 'git
annex sync' command that could probably be hidden via hook.
https://news.ycombinator.com/item?id=13559662

A lengthy thread on HN about why Fossil's author doesn't like Git:
https://news.ycombinator.com/item?id=16806114


## VCSes I Have Used

Git, CVS, Subversion, Mercurial (rarely), CA SCM (it's horrible)


## VCSes I Am Vaguely Aware Of

monotone, darcs, bazaar, BitKeeper, Perforce, Fossil


## VCSes I Know Only The Names Of

A patch-based VCS a la darcs that might be interesting: https://pijul.org/


## License

Unless explicitly stated otherwise, the contents of this repo are under the
[Creative Commons Attribution 4.0
license](https://creativecommons.org/licenses/by/4.0/).
