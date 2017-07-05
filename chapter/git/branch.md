# 分支管理

查看远程分支
```bash
git branch -a  
```

查看本地分支
```bash
git branch
```

创建分支
```bash
git branch test
```

把本地分支推送到远程
```bash
git push origin test  
```

切换分支
```bash
$ git checkout test  
```

删除本地分支
```bash
Git branch -d xxxxx
```

删除远程分支
```bash
git push origin :br-1.0.0  
```

## 把修改代码推向多个分支

以下是把 本地分支的prod修改的提交推向master，**但是仅仅限于两个分支代码都是一样的，否则会出错**
```bash
git push origin prod:master
```

怎么切换分支之后，发现 git pull 报错了。要加上远程分支完整名称才可以
```bash
git pull origin prod
```