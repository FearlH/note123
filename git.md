### git rebase
出现branch过时的情况的时候，需要先切换回主分支，比如就是develop。之后再git pull 使得develop更新为远端的分支。之后切换为自己的分支，git checkout your_branch.之后使用rebase:  git rebase develop.之后强制推送到远端git push -u origin your_branch --force。

### git配置多个ssh
在~/.ssh下新建config文件，添加一个ssh
```shell
Host github.com
HostName github.com
PreferredAuthentications publickey
IdentityFile ~/.ssh/github_id_rsa
```

### github切换443端口
github的22端口可能被屏蔽，可以尝试切换到443端口。按照下面的方法其配置：
```shell
Host github.com
HostName ssh.github.com
Port 443
PreferredAuthentications publickey
IdentityFile ~/.ssh/github_ed25519
```

### 对ssh命令进行debug
可以使用ssh -v来打印ssh命令的执行过程比如使用`ssh -v -T git@github.com`来对链接进行测试

