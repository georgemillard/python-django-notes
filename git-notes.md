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

