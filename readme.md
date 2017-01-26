# Version Control Is Not A Solved Problem

As a programmer, version control is a fundamental part of how I approach my
daily work.

Git is my current default choice for the many things it gets right, but it has
many superficial flaws and some fundamental ones.

It is my belief that a significantly better version control system is possible.
If the one I want is out there, I haven't found it so far.

I would like, therefore, to think about what such a VCS might look like.

If you know of one that meets my criteria, *please* let me know. I love
discovering amazing new tools.


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


## Other People's Thoughts

I am definitely not the first person to express the idea that the ideal VCS has
not yet been written. Here are a few such expressions:

http://www.chriskrycho.com/2014/next-gen-vcs.html

http://www.scopelift.co/blog/life-after-git

This chat between ESR and the author of monotone was interesting:
http://www.catb.org/esr/writings/version-control/version-control.html#graydon

Similarly, this Reddit thread has a few interesting comments in it:
https://www.reddit.com/r/programming/comments/2xxb17/git_is_actually_ok_for_some_binaries/

## VCSes I Have Used

Git, CVS, Subversion, Mercurial (rarely), CA SCM (it's horrible)


## VCSes I Am Vaguely Aware Of

monotone, darcs, bazaar, BitKeeper, Perforce, Fossil


## VCSes I Know Only The Names Of

A patch-based VCS a la darcs that might be interesting: https://pijul.org/


## License

Unless explicitly stated otherwise, the contents of this repo are under the
[Creative Comments Attribution 4.0 license](https://creativecommons.org/licenses/by/4.0/).
