# git-filter-repo demo (results)

This repo contains the results of demonstrating how to use git-filter-repo to remove sensitive data such as passwords or API keys from a git repo.

For the complete demo see [git-filter-repo-demo](https://github.com/mmercurio/git-filter-repo-demo).

## Afterthroughts
After reading through [git-filter-repo-demo](https://github.com/mmercurio/git-filter-repo-demo), it's interesting to note that even the secret value documented in the README was replaced with "`***REMOVED***`" (as it should.)

Compare:

* Original: [README.md](https://github.com/mmercurio/git-filter-repo-demo/blob/main/README.md)
* Filtered: [README.md](https://github.com/mmercurio/git-filter-repo-demo-filtered/blob/readme/README.md)


## In Summary

git-filter-repo is a flexilble tool to quicklky and easily rewrite git history (perhaps *too easy*). It's much more flexible and powerful than the older, [BFG Repo-Cleaner](https://rtyley.github.io/bfg-repo-cleaner/). Between the two, I would stick with git-filter-repo. For demo of BFG, see [bfg-test-repo](https://github.com/mmercurio/bfg-test-repo). 

Rewriting history is a very dangerous operation and shoud be exercised with extreme caution. If used improperly, it's possible to destroy your repository and overwrite people's hard work. Even if work is not lost forwever, it's very easy to make a ton of extra work for anyone working on a repo where history is overwritten.

Before pushing changes with rewritten history, I urge you to:

1. Have an untouched backup of the entire repo prior to rewriting history.
1. Carefully read and understand the [DISCUSSION](https://htmlpreview.github.io/?https://github.com/newren/git-filter-repo/blob/docs/html/git-filter-repo.html#DISCUSSION) section and warnings in the git-filter-repo(1) Manual Page. 
1. Coordinate and communicate with anyone who may be working on repos with rewritten history. ***Once history is rewritten, no one should attempt to commit or push to the repository without first performing a fresh clone.***
