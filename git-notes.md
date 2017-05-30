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

