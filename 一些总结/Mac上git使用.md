Mac上git使用：

## 一.配置git,创建ssh key

1.设置username和email:

git config --local username "lovelyfrog"

git config --local user.email "15307110114@fudan.edu.cn"

2.创建ssh key:

由于本地Git仓库和Github仓库是通过SSH加密的，所以连接时需要设置：

先查看 .ssh是否存在：

cd ~/.ssh

不管存不存在都可以新建ssh key:

ssh-keygen -t rsa -C "15307110114@fudan.edu.cn"

然后打开生成的id_rsa.pub文件，复制

cat /.ssh/id_rsa.pub

## 二.将本地与github连接

打开GitHub中的settings, 点击 SSH and GPG keys中的New SSH key,然后将上一步复制的ssh key 粘贴进来。

## 三.提交本地项目到github

先在Github上创建一个仓库，在本地仓库中的命令行中输入来添加远程仓库地址：

git remote add origin http://github.com/lovelyfrog/....git

关联好之后就可以把本地仓库的所以内容上传到github中：

git push -u origin master

如果出现

```bash
remote: Permission to lovelyfrog/cs61B.git denied to liudajie.
fatal: unable to access 'https://github.com/lovelyfrog/cs61B.git/': The requested URL returned error: 403
```

这是因为之前git登录的用户没有访问它的权限，我们需要将之前登录的密码删除：打开钥匙串访问，找到github.com删除即可。

然后又会报错：

```bash
 ! [rejected]        master -> master (fetch first)
error: failed to push some refs to 'https://github.com/lovelyfrog/cs61B.git'
hint: Updates were rejected because the remote contains work that you do
hint: not have locally. This is usually caused by another repository pushing
hint: to the same ref. You may want to first integrate the remote changes
hint: (e.g., 'git pull ...') before pushing again.
hint: See the 'Note about fast-forwards' in 'git push --help' for details.
```

需要先从远端库中把远端库的代码拉下来：

git pull origin [master](https://www.centos.bz/tag/master/) --allow-unrelated-histories

后面加上的 --allow-unrelated-histories 是将两段不相干的分支强行合并

然后再：

git push -u origin master

大功告成！