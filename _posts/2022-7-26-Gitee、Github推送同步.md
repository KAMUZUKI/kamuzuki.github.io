## Gitee、Github推送同步 

更改仓库.git文件下的config文件

 自August 13, 2021,Github只能通过token进行身份认证

```
[core]
	repositoryformatversion = 0
	filemode = false
	bare = false
	logallrefupdates = true
	symlinks = false
	ignorecase = true
[remote "github"]
	url = https://Your_Token@github.com/KAMUZUKI/Web.git
	fetch = +refs/heads/*:refs/remotes/github/*
[remote "gitee"]
	url = https://gitee.com/kamuzuki_admin/web.git
	fetch = +refs/heads/*:refs/remotes/gitee/*
[branch "master"]
	remote = origin
	merge = refs/heads/master
```

对两个仓库分别执行一次push命令，也就是多次推送

```shell
git push github matser  #推送至github
git push gitee master  #推送至gitee
```

执行命令，可以看到配置的两个仓库

```shell
git remote 
```

推送代码时，需要对两个仓库分别执行一次push命令，也就是多次推送

```shell
git push github matser 
git push gitee master  
```