# Git Branch Maintenance

# Archiving a branch
To archive a branch, there are a few steps that need to be taken in order to fully remove local and remote branches:
```
# Create a tag named similarly to the branch but with the 'archive/' prefix
# Point it to the latest commit of the branch
git tag archive/<tagname> <branchname>

# Delete the branch locally
git branch -D <branchname>

# Delete the local reference of remote branch ("forget" the remote branch)
git branch -d -r origin/<branchname>

# Push local tags to remote
git push --tags

# Delete the remote branch
git push -d origin <branchname>
# or, similarly, 'git push origin :<branchname>'

```

# Rebasing a branch
As a better alternative than merging a branch onto an existing branch, rebasing can be done.

This is opposite to merging, instead of merging a feature branch onto the end of the main branch (updating the main branch), you rebase the feature branch onto the main branch:
```
# Checkout the main branch
git checkout <main-branch>

# Pull any changes
git pull origin <main-branch>

# Rebase the feature branch onto the main branch
git rebase <main-branch> <feature-branch>
# Equivalent to
# git checkout <feature-branch>
# git rebase <main-branch>
```

# Renaming a branch
To rename a branch properly, including changing references to it on both local and remote repositories:
```
# Checkout the branch to be renamed
# git checkout demo
git checkout <oldbranchname>

# Rename branch
# git branch -m test
git branch -m <newbranchname>

# Delete the old branch on remote repo
# git push origin --delete demo
git push <remoterepo> --delete <oldbranchname>

# Push the branch with the correct name, and reset the upstream branch
# git push origin -u test
git push <remoterepo> -u <newbranchname>

```
