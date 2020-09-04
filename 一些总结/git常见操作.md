参考网址：https://www.atlassian.com/git/

`git init`: 初始化一个新的repo

`git add file`: 将file增加到repo staging area， 但只有你`git commit` 之后改变才会被记录。

`git commit -m "message": 描述做了什么改变`，commit the staged snapshot

`git commit --amend`: update 上次的commit，如果你上次commit的message不是你想要的，或者你又修改了某些地方。该命令可以将暂存区的文件自动加入。

`git reset`: undo a commit。

`git fetch <remote> <branch>`: fetch remote的branch 

`git push <remote-name> <branch-name>`: 将commited changes 发送到remote repo中。

Force push: `git push --force` ，原本git 会阻止push request overwrite central repo当它会导致一个non-fast-forward merge，但是--force，强制使得remote repo的branch与本地的相合。

`git remote add <name> <url>`:  创造一个连接到remote repo的新连接，之后就可以用`<name>` 来作为一个 `<url>`的一个short cut。

开发一个项目中，首先你在工working directory中编辑了一个文件，然后你准备将它保存作为项目的当前状态，这个时候使用`git add`，然后使用`git commit`将它commit 到项目的历史中。

.git文件夹下有存储两个库的commit ID，一个本地库，一个remote库。.git/refs/heads/master 存储的是上次commit 后的本地库最新的commit ID，.git/ref/remotes/origin/master 存储的是上次push 后remote库最新的commit ID。

假设此时，本地库 commit ID为0，remote库commit ID也为0。当一次git add 和 git commit 后，假设本地库commit ID更新为1，然后git push origin master 将remote库ID 变成1，对应的github上的库ID也会变成1。

假设此时github上的库被其他人更新了，commit ID变为2。choice 1: git fetch origin master 后使得本地上的remote库commit ID变为2，但此时本地库commit ID仍然是1，这时候要使用git merge 将本地库和remote库变为一致。choice 2: 或者直接用git pull origin master 一步到位让本地库，remote库commit ID都变为2。

