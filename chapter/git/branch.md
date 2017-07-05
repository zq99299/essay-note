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

## 在代码都相同的情况下怎么把修改的代码推向指定分支

以下是把 本地分支的prod修改的提交推向master，但是仅仅限于两个分支代码都是一样的，否则会出错
```bash
git push origin prod:master
```