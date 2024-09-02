# git-filter-repo demo
Demo of git-filter-repo.

This repo demonstrates how to use [git-filter-repo](https://github.com/newren/git-filter-repo) to remove sensitive data such as passwords or API keys from a git repo.

In addition to removing sensitive data from individual files, we'll also ensure data are removed from git commit history.

- [Initial State](#initial-state)
- [Initial Clean-up](#initial-clean-up)
- [Rewrite History with git-filter-repo](#rewrite-history-with-git-filter-repo)
  - [Removing Files](#removing-files)
  - [Remove Secrets from Files](#remove-secrets-from-files)
- [Pushing Changes to the Remote](#pushing-changes-to-the-remote)

## Initial State

Before trying to remove any files this is the state of the repo.

On branch, `main`:

```
├── README.md
└── files
    └── config.txt
```

Contents of `files/config.txt`:

```
# some basic configuration

SOURCE_DIR=~/src
GIT_REPO=git-filter-repo
GIT_USER=git
GIT_TOKEN=***REMOVED***
GIT_BIN=/usr/bin/git
```

Oh no! It looks like the value for `GIT_TOKEN` is an access token which should not have been committed to the repo. We'll need to fix that. The file `config.txt` was merged to `main` from branch `config`. We'll want to ensure the `GIT_TOKEN` value is removed from the merged file in `main`, any branches, and all commits.

On branch, `newbranch`:

```
├── README.md
└── files
    ├── config.txt
    └── secrets.txt
```

Contents of `files/secrets.txt`:

```
This file potentially contains some secret information that has
been committed to an unmerged branch.

Let's assume it was realized it shouldn't have been committed
before it was merged to `main`. But it still needs to be deleted.

Oh look, here's some secret info:

LgouCi4KICAgICAgICAgVE9PIE1BTlkgU0VDUkVUUwouCi4KLgouCi4KM3QjLUQt
QE1AJW1nLjdiMmpWN21yLHh0Kkw+QWpxNXN2THQ4WHhdcyx3QjEpUEIsQXpCZzZR
eHk9b2VQQm5dPTUjNykyMGFdJWVOaV5iXVlMYUA5QWpLXVdRdk04clkzSy0yIQo=

Let's hope this secret doesn't fall into the wrong hands.
```

It turns out that this entire file shouldn't have been committed. Even though the branch `newbranch` is not yet merged to `main`, all traces of the file need to be removed.

## Initial Clean-up

As a first step, we'll commit changes to the repo using normal git commands to remove the offending data.

First, the `GIT_TOKEN` is removed from `files/config.txt` in https://github.com/mmercurio/git-filter-repo-demo/pull/2 and merged to `main` .

Afterwards, we can confirm `GIT_TOKEN` has been removed on `main`:

```shell
$ cat files/config.txt
# some basic configuration

SOURCE_DIR=~/src
GIT_REPO=git-filter-repo
GIT_USER=git
GIT_BIN=/usr/bin/git
```

Second, the entire file `files/secrets.txt` is removed from branch `newbranch` by pushing a new commit to remove the file:

```shell
$ git checkout newbranch
Switched to branch 'newbranch'
Your branch is up to date with 'origin/newbranch'.

$ git rm files/secrets.txt
rm 'files/secrets.txt'

$ git commit -m "rm secrets.txt"
[newbranch bd98c34] rm secrets.txt
 1 file changed, 13 deletions(-)
 delete mode 100644 files/secrets.txt

$ git push
Enumerating objects: 5, done.
Counting objects: 100% (5/5), done.
Delta compression using up to 8 threads
Compressing objects: 100% (2/2), done.
Writing objects: 100% (3/3), 546 bytes | 546.00 KiB/s, done.
Total 3 (delta 0), reused 0 (delta 0), pack-reused 0 (from 0)
To github.com:mmercurio/git-filter-repo.git
   3ad3cb8..bd98c34  newbranch -> newbranch
```

After pushing these changes `newbranch` now looks like:

```
├── README.md
└── files
    └── config.txt
```

## Rewrite History with git-filter-repo

### Removing Files

First, we'll remove the file `files/secrets.txt` and all evidence from all commits.

In the example repo presented above, we have a single file in one directory that needs to be removed, so we can precisely specify the exact path using the `--path` option. We'll also use the `--invert-paths` option which says to only keep the paths *not selected* (i.e., remove the specified paths):

```shell
git filter-repo --path files/secrets.txt --invert-paths
```

If we needed to remove all files named `secrets.txt` across many directories in the repo, we could use the `--use-base-name` option, which treats the path name as a base filename without regard to any directory and would match every file in the repo with the specified path name:

```shell
git filter-repo --use-base-name --path secrets.txt --invert-paths
```

In our case, there is only one file to remove, so either command will work. There are many options and ways to select files, including various globbing and regex matching. See [git-filter-repo(1) Manual Page](https://htmlpreview.github.io/?https://github.com/newren/git-filter-repo/blob/docs/html/git-filter-repo.html) for details. Note, however, if the file to be removed was moved or renamed in past commits, it you may need to get more creative. The manual page and project page on GitHub includes more information.

First, let's confirm what the repo looks like before attempting to rewrite history. The `secrets.txt` file we want to remove was added and then removed on an unmerged branch, `newbranch`:

```shell
$ git log newbranch
commit bd98c34ecf60119679eea8ed1623d50d410d7ea4 (origin/newbranch, newbranch)
Author: Michael Mercurio <mmercurio@users.noreply.github.com>
Date:   Fri Aug 30 23:09:29 2024 -0400

    rm secrets.txt

commit 3ad3cb8ecb71476768fc2c829090b892d67e5ec5
Author: Michael Mercurio <mmercurio@users.noreply.github.com>
Date:   Fri Aug 30 22:12:50 2024 -0400

    Add secrets

commit bc1207502e7bb63c9d2ec73522227317d539a3b7
Merge: 16a38f2 32bcd88
Author: Michael Mercurio <mmercurio@users.noreply.github.com>
Date:   Fri Aug 30 22:08:56 2024 -0400

    Merge pull request #1 from mmercurio/config

    Add config

commit 32bcd8865621d2e3ced357801b76aeb5127e336a (origin/config)
Author: Michael Mercurio <mmercurio@users.noreply.github.com>
Date:   Fri Aug 30 22:04:58 2024 -0400

    add config

commit 16a38f2c5a98020de06128d29968db5d6a70d192
Author: Michael Mercurio <mmercurio@users.noreply.github.com>
Date:   Fri Aug 30 21:25:03 2024 -0400

    Initial commit
```

We can see the contents of the file by examining the commit which adds the secrets:

```shell
$ git show 3ad3cb8ecb71476768fc2c829090b892d67e5ec5
commit 3ad3cb8ecb71476768fc2c829090b892d67e5ec5
Author: Michael Mercurio <mmercurio@users.noreply.github.com>
Date:   Fri Aug 30 22:12:50 2024 -0400

    Add secrets

diff --git a/files/secrets.txt b/files/secrets.txt
new file mode 100644
index 0000000..be995c0
--- /dev/null
+++ b/files/secrets.txt
@@ -0,0 +1,13 @@
+This file potentially contains some secret information that has
+been commited to an unmerged branch.
+
+Let's assume it was realized it shouldn't have been commited
+before it was merged to `main`. But it still needs to be deleted.
+
+Oh look, here's some secret info:
+
+LgouCi4KICAgICAgICAgVE9PIE1BTlkgU0VDUkVUUwouCi4KLgouCi4KM3QjLUQt
+QE1AJW1nLjdiMmpWN21yLHh0Kkw+QWpxNXN2THQ4WHhdcyx3QjEpUEIsQXpCZzZR
+eHk9b2VQQm5dPTUjNykyMGFdJWVOaV5iXVlMYUA5QWpLXVdRdk04clkzSy0yIQo=
+
+Let's hope this secret doesn't fall into the wrong hands.
```

Or, directly access the file by checking out the commit:

```shell
$ git checkout 3ad3cb8ecb71476768fc2c829090b892d67e5ec5
HEAD is now at 3ad3cb8 Add secrets

$  cat files/secrets.txt
This file potentially contains some secret information that has
been commited to an unmerged branch.

Let's assume it was realized it shouldn't have been commited
before it was merged to `main`. But it still needs to be deleted.

Oh look, here's some secret info:

LgouCi4KICAgICAgICAgVE9PIE1BTlkgU0VDUkVUUwouCi4KLgouCi4KM3QjLUQt
QE1AJW1nLjdiMmpWN21yLHh0Kkw+QWpxNXN2THQ4WHhdcyx3QjEpUEIsQXpCZzZR
eHk9b2VQQm5dPTUjNykyMGFdJWVOaV5iXVlMYUA5QWpLXVdRdk04clkzSy0yIQo=

Let's hope this secret doesn't fall into the wrong hands.
```

Before executing `git filter-repo` command, it must be run on a freshly cloned repo (otherwise `--force` option will be needed). In the example below, we'll perform a fresh clone, rewrite history to remove `files/secrets.txt` from existence, and then verify all evidence of the file was removed.

Clone the repo into a new location:

```shell
$ mkdir fresh_clone
$ cd fresh_clone
$ git clone git@github.com:mmercurio/git-filter-repo-demo.git
Cloning into 'git-filter-repo-demo'...
remote: Enumerating objects: 24, done.
remote: Counting objects: 100% (24/24), done.
remote: Compressing objects: 100% (18/18), done.
remote: Total 24 (delta 7), reused 13 (delta 1), pack-reused 0 (from 0)
Receiving objects: 100% (24/24), 6.57 KiB | 6.57 MiB/s, done.
Resolving deltas: 100% (7/7), done.
$ cd git-filter-repo-demo
```

Using `git filter-repo` to remove the file:

```shell
$ git filter-repo --path files/secrets.txt --invert-paths
Parsed 9 commits
New history written in 0.07 seconds; now repacking/cleaning...
Repacking your repo and cleaning out old unneeded objects
HEAD is now at fdf227a Merge pull request #2 from mmercurio/rm_git_token
Enumerating objects: 19, done.
Counting objects: 100% (19/19), done.
Delta compression using up to 10 threads
Compressing objects: 100% (12/12), done.
Writing objects: 100% (19/19), done.
Total 19 (delta 3), reused 12 (delta 2), pack-reused 0 (from 0)
Completely finished after 0.14 seconds.
```

If you received a message an error message this:

```
Aborting: Refusing to destructively overwrite repo history since
this does not look like a fresh clone.
  (expected at most one entry in the reflog for HEAD)
Please operate on a fresh clone instead.  If you want to proceed
anyway, use --force.
```

and you followed the command above to start with a fresh clone, you may need to add the `--force` option, which is safe *provided you started from a fresh clone*. For more information on this, see [FRESHCLONE](https://htmlpreview.github.io/?https://github.com/newren/git-filter-repo/blob/docs/html/git-filter-repo.html#FRESHCLONE) in the git-filter-repo(1) Manual Page.

Let's repeat the steps above in to verify all traces of the files are removed:

```shell
$ git log newbranch
commit f313f198d5ccd4a461998df0c53445f95652c190 (HEAD -> newbranch)
Merge: a65551a 27db42c
Author: Michael Mercurio <mmercurio@users.noreply.github.com>
Date:   Fri Aug 30 22:08:56 2024 -0400

    Merge pull request #1 from mmercurio/config

    Add config

commit 27db42c8bc027ff7ef9e177495d86df93940b24a (config)
Author: Michael Mercurio <mmercurio@users.noreply.github.com>
Date:   Fri Aug 30 22:04:58 2024 -0400

    add config

commit a65551a6c805459cc766a9ea2be35c50bafc174a
Author: Michael Mercurio <mmercurio@users.noreply.github.com>
Date:   Fri Aug 30 21:25:03 2024 -0400

    Initial commit
```

Notice that the commits to add and remove `secrets.txt` are no longer in the history and also the other commits have been rewritten with updated commit hashes.

### Remove Secrets from Files

In this example, we'll erase from hisotyr the secret `GIT_ACCESS_TOKEN` value which was added to and then removed from the file `files/config.txt`.

Even though the token was removed from the file and commited to the repo, we're able to examine the history to see that the access token value is still available:

```shell
$ git log
commit f0eb21d91af73249709734a2f4da79012cf632d3 (origin/main, origin/HEAD, main)
Merge: bc12075 e38dab6
Author: Michael Mercurio <mmercurio@users.noreply.github.com>
Date:   Fri Aug 30 23:04:17 2024 -0400

    Merge pull request #2 from mmercurio/rm_git_token

    rm GIT_TOKEN

commit e38dab6ad68a8e5a71592ca83436c6becc2c6740 (origin/rm_git_token)
Author: Michael Mercurio <mmercurio@users.noreply.github.com>
Date:   Fri Aug 30 23:02:45 2024 -0400

    rm GIT_TOKEN
```

And the secret value removed can be examined:

```shell
$ git -P show e38dab6ad68a8e5a71592ca83436c6becc2c6740
commit e38dab6ad68a8e5a71592ca83436c6becc2c6740 (origin/rm_git_token)
Author: Michael Mercurio <mmercurio@users.noreply.github.com>
Date:   Fri Aug 30 23:02:45 2024 -0400

    rm GIT_TOKEN

diff --git a/files/config.txt b/files/config.txt
index 001748a..68de0ea 100644
--- a/files/config.txt
+++ b/files/config.txt
@@ -3,5 +3,4 @@
 SOURCE_DIR=~/src
 GIT_REPO=git-filter-repo
 GIT_USER=git
-GIT_TOKEN=***REMOVED***
 GIT_BIN=/usr/bin/git
```

The first step is to add the secret value to replace to a file such as `passwords.txt`:

```shell
echo '***REMOVED***' > ../passwords.txt
```

Next, we enter the `git filter-repo` command to replace all instances of the secrets in the repo:

```shell
$ git filter-repo --replace-text ../passwords.txt
Parsed 10 commits
New history written in 0.07 seconds; now repacking/cleaning...
Repacking your repo and cleaning out old unneeded objects
HEAD is now at 9dc4c1f Merge pull request #2 from mmercurio/rm_git_token
Enumerating objects: 27, done.
Counting objects: 100% (27/27), done.
Delta compression using up to 10 threads
Compressing objects: 100% (21/21), done.
Writing objects: 100% (27/27), done.
Total 27 (delta 6), reused 6 (delta 0), pack-reused 0 (from 0)
Completely finished after 0.14 seconds.
```

Note, by default, the secrets are replaced with "`***REMOVED***`", but this can be customerized with options.

Now, when examining the commit history, note that the commits have been updated with new commit hashes:

```shell
$ git log
commit 9dc4c1f4739cfa6e07366954094185266cfacfff (HEAD -> main)
Merge: 9ac60a2 f9243d0
Author: Michael Mercurio <mmercurio@users.noreply.github.com>
Date:   Fri Aug 30 23:04:17 2024 -0400

    Merge pull request #2 from mmercurio/rm_git_token

    rm GIT_TOKEN

commit f9243d0ecef7953baf85a53ee435137ec8e5eea6 (rm_git_token)
Author: Michael Mercurio <mmercurio@users.noreply.github.com>
Date:   Fri Aug 30 23:02:45 2024 -0400

    rm GIT_TOKEN
```

Examining the details of the second commit removing the access token reveals the access token value has been replaced in the commit:

```
$ git show f9243d0ecef7953baf85a53ee435137ec8e5eea6
commit f9243d0ecef7953baf85a53ee435137ec8e5eea6 (rm_git_token)
Author: Michael Mercurio <mmercurio@users.noreply.github.com>
Date:   Fri Aug 30 23:02:45 2024 -0400

    rm GIT_TOKEN

diff --git a/files/config.txt b/files/config.txt
index 82de8af..68de0ea 100644
--- a/files/config.txt
+++ b/files/config.txt
@@ -3,5 +3,4 @@
 SOURCE_DIR=~/src
 GIT_REPO=git-filter-repo
 GIT_USER=git
-GIT_TOKEN=***REMOVED***
 GIT_BIN=/usr/bin/git
```

## Pushing Changes to the Remote

Last, before pushing changes to the remote, examine the contents of the repo carefully. You'll likely want to look at the `.git/filter-repo/commit-map` to examine the old and new commit hashes that have been rewritten.

Before pushing updated refs with rewritten history to the remote, you should review the [DISCUSSION](https://htmlpreview.github.io/?https://github.com/newren/git-filter-repo/blob/docs/html/git-filter-repo.html#DISCUSSION) section and warnings in the git-filter-repo(1) Manual Page for some important considerations.

When you're ready to push rewritten history, you'll need to add the remote back to the repo config because git-filter-repo removes the remote to prevent unintentially pushing rewritten history.

For the purposes of this demo, we'll push the changes to a new repo, `git-filter-repo-demo-filtered`, so that the before and after can be easily compared.

```shell
$ git push # TBD
```
