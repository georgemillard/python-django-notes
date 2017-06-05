### Git cheat sheet

### Linking an existing local project to a remote github repository

`git remote add origin https://github.com/georgemillard/The-Hive.git`

Verify remote:
`git remote -v`

Set branch master to follow remote:
`git branch --set-upstream-to=origin/master master`

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

### Working on a new feature while waiting for a pull request to be approved
see https://stackoverflow.com/questions/35790561/working-while-waiting-for-pending-pr

Open pull request for `feature-1`
Create a new branch off the last feature branch:

`git checkout -b feature-2 feature-1`

Work on new branch:

```
git status
git add <file>
git commit
```

Push changes:

`git push -u origin feature-2`
(-u sets up a remote tracking branch, so `git push` alone will work)

Pull request approved, previous branch merged and deleted

Rebase next branch onto master:

`git rebase --onto master feature-2`

