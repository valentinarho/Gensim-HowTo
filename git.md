# Help me with...Git!

## Git configurations

### Set the user of the current repository

    git config user.name "Valentina Rho"
    git config user.email "rv@mail.com"

Add the option --global to save these configuration globally

### Setup the colorful console

    git config --global color.ui auto


### Ignore file permissions changes from diffs

    git config --global core.fileMode false


## Draw the log tree

    git log --oneline --graph --color --all --decorate


## Discard all local changes

    git fetch
    git reset --hard


## Push a local branch to a remote branch with different name

    git push REMOTE_NAME LOCAL_BRANCH:REMOTE_BRANCH


## Delete the last commit if already pushed

!! The push --force is very dangerous if you have coworkers on the repo.

    git reset HEAD^ --hard
    git push origin --force


## Delete a file from the history

### Delete a file from all the history of the current branch

!! This command is dangerous if you have coworkers on the repo.

    git filter-branch --tree-filter 'rm FILEPATH' HEAD


### Delete a file from all branches and tags

!! This command is dangerous if you have coworkers on the repo.

    git filter-branch --force --index-filter 'git rm --cached --ignore-unmatch FILEPATH' --prune-empty --tag-name-filter cat -- --all


## Amending

!! The push --force is very dangerous if you have coworkers on the repo.


### Modify a file within an already pushed commit

    # edit your files
    git add FILE
    git commit --amend --no-edit
    git push REMOTE_NAME BRANCH_NAME --force


### Modify the last commit message

    git commit --amend -m "New message"
    git push REMOTE_NAME BRANCH_NAME --force
