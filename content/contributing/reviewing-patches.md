---
title: "Reviewing patches"
date: 2018-05-12T21:27:38-04:00
draft: false
weight: 40
---
Reviewing patches is done on our gerrit instance at
https://gem5-review.googlesource.com/.

After logging in with your Google account, you will be able to comment, review,
and push your own patches as well as review others' patches. All gem5 users are
encouraged to review patches. The only requirement to review patches is to be
polite and respectful of others.

There are multiple labels in Gerrit that can be applied to each review detailed
below.

 * Code-review: This is used by any gem5 user to review patches. When reviewing
   a patch you can give it a score of -2 to +2 with the following semantics.
   * -2: This blocks the patch. You believe that this patch should never be
     committed. This label should be very rarely used.
   * -1: You would prefer this is not merged as is
   * 0: No score
   * +1: This patch seems good, but you aren't 100% confident that it should be
     pushed.
   * +2: This is a good patch and should be pushed as is.
 * Maintainer: Currently only PMC members are maintainers. At least one
   maintainer must review your patch and give it a +1 before it can be merged.
 * Verified: This is automatically generated from the continuous integrated
   (CI) tests. Each patch must receive at least a +1 from the CI tests before
   the patch can be merged. The patch will receive a +1 if gem5 builds and
   runs, and it will receive a +2 if the stats match.
 * Style-Check: This is automatically generated and tests the patch against the
   gem5 code style (http://www.gem5.org/Coding_Style). The patch must receive a
   +1 from the style checker to be pushed.

Note: Whenever the patch creator updates the patch all reviewers must re-review
the patch. There is no longer a "Fix it, then Ship It" option.

Once you have received reviews for your patch, you will likely need to make
changes. To do this, you should update the original git changeset. Then, you
can simply push the changeset again to the same Gerrit branch to update the
review request.

Please see [governance](/contributing/governance/) and [reviewing patches](/contributing/governance/#reviewing).

```
 git push origin HEAD:refs/for/master
```

Note: If you have posted a patch and don't receive any reviews, you may need to
prod the reviewers. You can do this by adding a reply to your changeset review
on gerrit. It is expected that at least the maintainer will supply a review for
your patch.

