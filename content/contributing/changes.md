---
title: "Making changes to gem5"
date: 2018-05-12T21:27:14-04:00
draft: false
weight: 20
---
It is strongly encouraged to use git branches when making changes to gem5.
Additionally, keeping changes small and concise and only have a single logical
change per commit.

Unlike our previous flow with Mercurial and patch queues, when using git, you
will be committing changes to your local branch. By using separate branches in
git, you will be able to pull in and merge changes from mainline and simply
keep up with upstream changes.

Requirements for change descriptions
------------------------------------
To help reviewers and future contributors more easily understand and track
changes, we require all change descriptions be strictly formatted.

A canonical commit message consists of three parts:

 * A short summary line describing the change. This line starts with one or
   more keywords (found in the MAINTAINERS file) separated by commas followed
   by a colon and a description of the change. This line should be no more than
   65 characters long since version control systems usually add a prefix that
   causes line-wrapping for longer lines.
 * (Optional, but highly recommended) A detailed description. This describes
   what you have done and why. If the change isn't obvious, you might want to
   motivate why it is needed. Lines need to be wrapped to 75 characters or
   less.
 * Tags describing patch metadata. You are highly recommended to use
   tags to acknowledge reviewers for their work. Gerrit will automatically add
   most tags.

Tags are an optional mechanism to store additional metadata about a patch and
acknowledge people who reported a bug or reviewed that patch. Tags are
generally appended to the end of the commit message in the order they happen.
We currently use the following tags:

 * Signed-off-by: Added by the author and the submitter (if different).
   This tag is a statement saying that you believe the patch to be correct and
   have the right to submit the patch according to the license in the affected
   files. Similarly, if you commit someone else's patch, this tells the rest
   of the world that you have have the right to forward it to the main
   repository. If you need to make any changes at all to submit the change,
   these should be described within hard brackets just before your
   Signed-off-by tag. By adding this line, the contributor certifies the
   contribution is made under the terms of the Developer Certificate of Origin
   (DCO) [https://developercertificate.org/].
 * Reviewed-by: Used to acknowledge patch reviewers. It's generally considered
   good form to add these. Added automatically.
 * Reported-by: Used to acknowledge someone for finding and reporting a bug.
 * Reviewed-on: Link to the review request corresponding to this patch. Added
   automatically.
 * Change-Id: Used by Gerrit to track changes across rebases. Added
   automatically with a commit hook by git.
 * Tested-by: Used to acknowledge people who tested a patch. Sometimes added
   automatically by review systems that integrate with CI systems.

Other than the "Signed-off-by", "Reported-by", and "Tested-by" tags, you
generally don't need to add these manually as they are added automatically by
Gerrit.

It is encouraged for the author of the patch and the submitter to add a
Signed-off-by tag to the commit message. By adding this line, the contributor
certifies the contribution is made under the terms of the Developer Certificate
of Origin (DCO) [https://developercertificate.org/].

It is imperative that you use your real name and your real email address in
both tags and in the author field of the changeset.

For significant changes, authors are encouraged to add copyright information
and their names at the beginning of the file. The main purpose of the author
names on the file is to track who is most knowledgeable about the file (e.g.,
who has contributed a significant amount of code to the file).

Note: If you do not follow these guidelines, the gerrit review site will
automatically reject your patch.
If this happens, update your changeset descriptions to match the required style
and resubmit. The following is a useful git command to update the most recent
commit (HEAD).

```
 git commit --amend
```


