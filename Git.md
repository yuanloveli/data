# Git

##	一、常用指令

获取帮助：git help <verb>   #例如 git help config       常用： **git add -h 也行**



* .gitignore文件是将不需要管理的文件排除

* git add . #将文件添加至暂存区

* git commit -m "注释"      #提交暂存区中的文件

* git status   #查看文件状态

  

* **git push origin master:master   #origin远程仓库名 master本地分支：远程分支名（不写默认同名）**

* git pull origin master          #从远程仓库拉取并且合并

  

* git log  #查看日志

* git reflog    # 查看详细日志 

* git reset --hard commitID  #版本回退

* 分支

* git branch #查看所有分支

* git branch 分支名   #创建分支

* git checkout 分支名    #移至该分支

* git checkout -b 分支名     #创建并移到该分支

* git branch -d 分支名 #删除分支

* git branch -D 分支名  #强制删除分支

解决冲突：

1. 手动选择更改内容
2. git add .    #添加至暂存区

##二、远程仓库

* git remote add 仓库名(origin)  仓库地址          #添加远程仓库
* git remote   -v   #查看仓库 并且显示URL
* git remote show origin  #查看更详细的仓库信息
* git remote rename origin  Hello  #更改仓库名称
* git remote remove origin    #删除远程仓库
* **git push origin master:master   #origin远程仓库名 master本地分支：远程分支名（不写默认同名）**

* git clone 仓库地址 [本地目录]

**注意：push 之前需要保证工作区有东西**

## 三.、ssh公钥

* ```console
  ssh-keygen -o          #生成公钥
  ```

* ```console
  cat ~/.ssh/id_rsa.pub          #查看公钥
  ```

##四、代码冲突问题解决方法：

在仓库下进入命令行输入  ***git pull origin master --allow-unrelated-histories***

https://kisugitakumi.com/2022/01/18/Git%E5%AD%A6%E4%B9%A0%E7%AC%94%E8%AE%B0/
