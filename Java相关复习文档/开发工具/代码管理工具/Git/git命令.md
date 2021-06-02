# 新增文件推送至远程仓库

1、我在项目文档文件夹下新增了一个界面原型图文件（变更）
2、进入本地仓库，将变更增加到缓存

用cd进入到本地仓库的目录下

```
# 将当前目录及其子目录的所有变更文件添加到缓存
git add . 
#  将当前目录下的变更文件添加到缓存
git add *
# 将指定变更文件添加到缓存
git add 指定文件路径 
```

3、切换一下分支，将变更提交

```
# 切换分支
git checkout 分支名
# 提交变更
git commit -m "说明提交内容" 
```

4、将变更push到远程仓库

```
# 查看远程仓库，一般clone下来的仓库默认远程仓库名为origin
git remote -v 
# push到远程仓库
git push 远程仓库名 分支名 
```

![](image\4355294-4ebbfaeae7d8a87f.webp)

# 创建本地仓库并同步到远程

1.进入你想要同步到远程的项目的根目录，初始化为本地仓库

```
#cd 你的项目路径
git init
```

2、在本地仓库添加远程仓库链接

```
git remote add 远程仓库名 远程仓库链接
```

3、将文件都add进缓存，提交，然后push到远程（同上）

```
git add .
git commit -m "说明"
git push 远程仓库名 分支名
```

![](image\4355294-645ed2ff0f95bf6e.webp)

# 分支与合并

切换分支

```
# 创建并切换
git checkout -b branch_name 
git checkout branch_name
```

合并目标分支到当前分支

```
git merge branch_name 
```



```
#查看本地分支
git branch
#比较分支差异
git diff branch
#查看合并冲突文件
git status
#查看日志  按q推出
git log
#撤销commit
git reset 版本号
git reset -hard 版本号 # 同时将代码恢复到前一个commit_id 对应的版本
#合并回退
git reset --merge 
```

