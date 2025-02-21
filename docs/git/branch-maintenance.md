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
