---
title: "Posting a review"
date: 2018-05-12T21:27:22-04:00
draft: false
weight: 30
---
If you have not signed up for an account on the Gerrit review site
(https://gem5-review.googlesource.com), you first have to create an account.

Setting up an account
---------------------
 1. Go to https://gem5.googlesource.com/
 2. Click "Sign In" in the upper right corner. Note: You will need a Google
 account to contribute.
 3. After signing in, click "Generate Password" and follow the instructions.

Submitting a change
-------------------

In gerrit, to submit a review request, you can simply push your git commits to
a special named branch. For more information on git push see
https://git-scm.com/docs/git-push.

There are three ways to push your changes to gerrit.

Push change to gerrit review
----------------------------

```
 git push origin HEAD:refs/for/master
```

Assuming origin is https://gem5.googlesource.com/public/gem5 and you want to
push the changeset at HEAD, this will create a new review request on top of the
master branch. More generally,

```
 git push <gem5 gerrit instance> <changeset>:refs/for/<branch>
```

See https://gerrit-review.googlesource.com/Documentation/user-upload.html for
more information.

Pushing your first change
--------------------------
The first time you push a change you may get the following error:

```
 remote: ERROR: [fb1366b] missing Change-Id in commit message footer
 ...
```

Within the error message, there is a command line you should run. For every new
clone of the git repo, you need to run the following command to automatically
insert the change id in the the commit (all on one line).

```
 curl -Lo `git rev-parse --git-dir`/hooks/commit-msg \
	https://gerrit-review.googlesource.com/tools/hooks/commit-msg ; \
 chmod +x `git rev-parse --git-dir`/hooks/commit-msg
```

If you receive the above error, simply run this command and then amend your
changeset.

```
 git commit --amend
```

Push change to gerrit as a draft
--------------------------------

```
 git push origin HEAD:refs/drafts/master
```

Push change bypassing gerrit
-----------------------------

Only maintainers can bypass gerrit review. This should very rarely be used.

```
 git push origin HEAD:refs/heads/master
```

Other gerrit push options
-------------------------

There are a number of options you can specify when uploading your changes to
gerrit (e.g., reviewers, labels). The gerrit documentation has more
information.
https://gerrit-review.googlesource.com/Documentation/user-upload.html


