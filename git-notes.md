### Git cheat sheet

### Linking an existing local project to a remote github repository

`git remote add origin https://github.com/georgemillard/The-Hive.git`

Verify remote:
`git remote -v`

Set branch master to follow remote:
`git branch --set-upstream-to=origin/master master`

### Importing a Bitbucket repository into Github

On github, click the `+` in the top right of the screen, then click "Import Repository"

Paste in the Bitbucket clone link and fill out details,  then click begin!

**Note** You may get asked for credentials, these will be for the account you are importing it from (Ie, your bitbucket account).

Once done, issues will need to be manually imported.

#### Updating local / staging / live git repos
The git repositorys will need to have their origin's updated in order to use the new GitHub Repo.
```
$ git remote rename origin bitbucket
$ git remote add origin https://github.com/emfoundation/awesome-github-repo.git

$ git remote rm bitbucket
```

This will allow you to fetch, but not to pull. You will get this error:
```
$ git pull
There is no tracking information for the current branch.
Please specify which branch you want to merge with.
```
In order to do this, you will need to set the upstream branches for each branch (Normally `master` & `develop` but any other branches you currently need)

`git branch --set-upstream-to=origin/<branch> <branch>`

ie: `git branch --set-upstream-to=origin/develop develop`

#### Sourcetree config changes
The steps to swap over from bitbucket are similar to the terminal commands, but more graphical.
1. Go to your repository settings and then the "Remotes" tab

2. Click edit on remote that is called origin, and should be directed to a bitbucket url.

3. Change the remote name to "bitbucket" and click ok

4. Next create a new remote by clicking "Add"

5. Name this new remote "origin", and put the url as the clone link that can be found on your repo's github main page. It should automatically set the host type. Enter your github username. Press okay.

6. Now delete the remote labelled "bitbucket", and close the settings tab.

7. Now you can git fetch and git pull, and it will fetch from the new origin which is now pointed to the github repo.


### Creating a new branch

`git branch`
Show all branches, * indicates which is currently checked out

`git checkout master`

`git checkout -b feature`
will create and checkout feature branch

then add changes to feature and test. when you are ready to merge:

```
git checkout master
git pull origin master
git merge feature
git push origin master
```

`git checkout --track origin/branch`
Also useful for creating a local version of a remote branch...

### Git Stash

```
git stash
git stash list
git stash apply
```

### Feature branch

```
git checkout master
git pull
git checkout -b feature
```

Work on new branch:

```
git status
git add <file>
git commit
```

Push changes:

`git push -u origin feature`
(-u sets up a remote tracking branch, so `git push` alone will work)

Pull Request...

Checkout master and pull changes:

```
git checkout master
git pull
```

Delete feature branch:

```
git branch -d feature
```

### Diff between two branches

`git diff branch_1..branch_2`

### Useful git commands

`git reset <file>`

`git reset`

Undo `git add <file>` without losing changes.

`git log --oneline`

`git reset <commit hash>`

Reset to earlier commit
