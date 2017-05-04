# git

## 解决GIT提交，文件名太长问题(filename too long)
```bash
git config --system core.longpaths true  
```


# 使用错误处理，技巧
## 让服务器上的版本回退到某一个指定版本，提交历史也干掉
### 问题起因是：

    1. 由于错误的命令操作，导致本地代码更新到错误的代码，然后上传到了服务器。
    2. 其他使用者更新了服务端的代码发现了问题


### 解决问题
解决问题的思路如下：

1. 本地回退到某一个提交点
2. 强制push到服务端
3. 其他仓库使用者再fetch 和 merge 让本地代码的版本强制合并到服务器的版本

命令如下：

1. checkout 某一个版本，或则reset到某一个版本（具体方案以后再自测解决）
2. git push -u -f origin master （某些仓库有强推权限需要打开）
3. 让本地代码版本与服务器版本更新到一致
    1. git fetch origin master
    2. git merge origin master

    上面两行代码貌似是 git pull 的一个合并，但是在这种情况下，git pull 不能解决问题。

 