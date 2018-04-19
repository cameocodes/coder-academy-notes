## Github Git Cheat Sheet

### MAKE CHANGES

- `$ git status` 
    - Lists all new or modified files to be committed
- `$ git add [file]` 
    - Snapshots the file in preparation for versioning
- `$ git reset [file]` 
    - Unstages the file, but preserve its contents
- `$ git diff` 
    - Shows file differences not yet staged
- `$ git diff --staged` 
    - Shows file differences between staging and the last file version
- `$ git commit -m "[descriptive message]"` 
    - Records file snapshots permanently in version history

### GROUP CHANGES

- `$ git branch` Lists all local branches in the current repository
- `$ git branch [branch-name]` Creates a new branch
- `$ git checkout [branch-name]` Switches to the specified branch and updates the working directory
- `$ git merge [branch]` Combines the specified branch’s history into the current branch
- `$ git branch -d [branch-name]` Deletes the specified branch

### SYNCHRONIZE CHANGES

- `$ git fetch [bookmark]`
    - Downloads all history from the repository bookmark
- `$ git merge [bookmark]/[branch]`
    - Combines bookmark’s branch into current local branch
- `$ git push [alias] [branch]`
    - Uploads all local branch commits to GitHub
- `$ git pull`
    - Downloads bookmark history and incorporates changes

### REFACTOR FILENAMES

- `$ git rm [file]`
    - Deletes the file from the working directory and stages the deletion
- `$ git rm --cached [file]`
    - Removes the file from version control but preserves the file locally
- `$ git mv [file-original] [file-renamed]`
    - Changes the file name and prepares it for commit

### REDO COMMITS

- `$ git reset [commit]`
    - Undoes all commits after [commit], preserving changes locally
- `$ git reset --hard [commit]`
    - Discards all history and changes back to the specified commit

### REVIEW HISTORY

- `$ git log`
    - Lists version history for the current branch
- `$ git log --follow [file]`
    - Lists version history for a file, including renames
- `$ git diff [first-branch]...[second-branch]`
    - Shows content differences between two branches
- `$ git show [commit]`
    - Outputs metadata and content changes of the specified commit