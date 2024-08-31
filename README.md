# git-filter-repo demo
Demo of git-filter-repo.

This repo demonstrates how to use [git-filter-repo](https://github.com/newren/git-filter-repo) to remove sensitive data such as passwords or API keys from a git repo.

In addition to removing sensitive data from individual files, we'll also ensure data are removed from git commit history.

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
