## 工作流程

- Workspace：工作空间
- Index：暂存区
- Repository：本地仓库
- Remote：远程仓库

![img](https:////upload-images.jianshu.io/upload_images/3985563-6b745d5fac15906c.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/508/format/webp)

## 命令

![img](https:////upload-images.jianshu.io/upload_images/3985563-c7f05348b711ebbe.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1000/format/webp)

HEAD始终指向当前所处分支的最新的提交点

### 配置

查看配置：git config --global --list

git config --global user.name "mason"

git config --global user.email "mason@gmail.com"

### add

| **git add .**           | **添加当前目录的所有文件到暂存区**        |
| ----------------------- | ----------------------------------------- |
| **git add \<dir/file>** | **添加指定目录/文件到暂存区，包括子目录** |

### commit

| **git commit -m \<message>**         | **提交暂存区到本地仓库,message代表说明信息** |
| ------------------------------------ | -------------------------------------------- |
| **git commit \<file> -m \<message>** | **提交暂存区的指定文件到本地仓库**           |
| **git commit --amend -m \<message>** | **使用一次新的commit，替代上一次提交**       |

### branch

| **git branch**                                               | **列出所有本地分支**                           |
| ------------------------------------------------------------ | ---------------------------------------------- |
| **git branch -r**                                            | **列出所有远程分支**                           |
| **git branch -a**                                            | **列出所有本地分支和远程分支**                 |
| **git branch \<branch-name>/git branch newbranch branch**    | **新建一个分支，但依然停留在当前分支**         |
| **git checkout -b \<branch-name>**                           | **新建一个分支，并切换到该分支**               |
| **git branch --track <branch><remote-branch>**？？？？？？？ | **新建一个分支，与指定的远程分支建立追踪关系** |
| **git checkout \<branch-name>**                              | **切换到指定分支，并更新工作区**               |
| **git branch -d \<branch-name>**                             | **删除分支**                                   |
| **git push origin --delete <branch-name>**？？？？？？？     | **删除远程分支**                               |
| **git branch -vv**                                           | **查看当前分支**                               |
| **git branch -m br1 newbranch**                              | **分支重命名**                                 |

### push

ssh-keygen -t rsa -C "mason@gmail.com"

github -> setting -> SSH and GPG keys 新增 id_rsa.pub 的内容

git remote add origin git@github.com:maruami/test.git 把本地库与远程库关联

git push -u origin master 第一次推送时

git push --set-upstream origin newbranch 提交新分支

### pull

拉取更新

### log

git log 显示当前分支的版本历史

git log --pretty=oneline 信息横向显示

git log -p （显示版本之间的代码差异）

git log 7b1558c （指定提交名称前7位）

### 其他

git mv oldfile newfile  文件重命名与移动

















### 2. 分支

- 合并分支
  - 合并(merge)方法
    - 1、直接合并：把两条分支上的历史轨迹合并，交汇到一起
    - 2、压合合并：一条分支上若干提交条目压合成一个提交条目，提交到另一条分支的末梢
    - 3、拣选合并：拣选另一条分支上的某个提交条目的改动带到当前分支上
  - 直接合并
    - git merge 分支名称  
      git checkout alternate  
      git add about.html  
      git commit -m "add about page"  
      git checkout master  
      git merge alternate
  - 压合合并
    - git merge --squash 分支名称  
      git checkout -b contact master  
      git add contact.html  
      git commit -m "add contact file"  
      git commit -m "add contact file 2" -a  
      git checkout master  
      git merge --squash contact  
      git status  
      git commit -m "add contact page" -m "has primary and secondary email"
  - 拣选合并
    - git cherry-pick 提交名称  
      git checkout contact  
      git commit -m "add contact 3" -a  
      [contact 6dbaf82]......  
      git checkout master  
      git cherry-pick 6dbaf82  /  git cherry-pick -n 6dbaf82
- 冲突处理
  - git merge  
    git checkout -b about master  
    编辑about.html  
    git add about.html  
    git commit -m "add about.html "  
    git branch about2 about  
    编辑about.html  
    git commit -m "add about.html 1" -a  
    git checkout about2  
    编辑about.html  
    git commit -m "add about.html 2" -a  
    git checkout about      
    git merge about2     
    git mergetool    
    git commit
  - 处理冲突软件（kdiff3）：git config --global merge.tool kdiff3
  - git mergetool

### 3. 查询历史记录

- \^：回溯一个版本
  - git log 18f822e^^
  - 注：1、windows系统下，^需要添加双引号 git log “18f822e^^”。
  - 注：2、当遇到某个节点（通常是版本合并后的节点）有并列的多个父节点时，“^1”代表第一个父节点，“^2”代表第二个，以此类推。而“^”是“^1”的简写。
- *~N：回溯N个版本
  - git log -1 HEAD^^^  /  git log -1 HEAD^~2  /  git log -1 HEAD~1^  /  git log -1 HEAD~3
  - git log -1 HEAD~10..HEAD
- 查看版本间差异
  - git diff 版本名称（与当前工作目录树的差异）
- 跟踪内容
  - 检查在同一个文件内移动或复制的代码行：git blame -M 文件名
  - 查看文件之间的复制：git blame -C -C 文件名
  - 查看显示代码的具体变动的历史记录：git log -C -C -1 -p
- 撤销修改
  - 增补提交：git commit -C HEAD -a --amend
    - --amend：增补提交
    - -C：复用指定提交的提交留言
    - -c：打开编辑器，在已有提交留言基础上修改
  - 反转提交：git revert -n 提交名称
    - 参数：--no-edit
  - 复位：git reset 提交名称
    - 提交名称默认值：HEAD
    - 提交名称可用^和~修饰符
    - 参数--soft：暂存所有因复位带来的差异，但不提交它
    - 参数--hard：慎用，从版本库和工作目录树中同时删除提交
- 重新改写历史记录
  - 重新排序提交：git rebase -i HEAD~3
  - 将多个提交压合成一个提交：git rebase -i 0bb3dfb^
  - 将一个提交分解成多个提交：git rebase --continue

### 4. 与远程版本库协作

- 版本库同步
  - 取来（fetch）：git fetch
  - 查看远程分支：git branch -r
  - 取来合并：git pull 远程版本库名称 须要拖入的远程分支名
  - 远程分支名前缀origin/表示远程版本库上的分支名称，origin是默认远程版本库别名
- 推入改动
  - 推入默认版本库origin：git push
  - 查看推入哪些提交：git push --dry-run
  - 推入指定版本库：git push <repository> <refspec>  
    git push origin mybranch:master
- 添加新的远程版本库
  - 一次拖入：git pull git://ourcompany.com/dev-erin.git
  - 使用别名：git remote add 别名 路径
  - 查看远程版本库详细信息：git remote show <name>
  - 删除别名：git remote rm

### 5. 管理本地版本库

- 使用标签标记里程碑

  - 标签只读、标签名不能包含空格
  - 查看已存在标签：git tag
  - 新建标签：git tag 标签名
  - git tag 标签名 提示名称/分支名称

- 发布分支的处理

  - 发布分支通常以RB_为前缀并包含版本号，RB_1.3
  - git branch RB_1.0.1 1.0

- 标签与分支的有效名称

  - 不能以“/”结尾
  - 不能以“.”开头
  - 不能使用特殊字符：空格~^:?*[控制符删除键
  - 不能出现“..”

- 记录和跟踪多个项目

  - 多个项目共享一个版本库
  - 多项目多版本库

- 使用Git子模块跟踪外部版本库

  - 添加新子模块

    - 查看该版本库的子模块：git submodule
    - 添加新子模块：git submodule add 源版本库 存储路径  
      git submodule add  git://github.com/tswicegood/hocus.git  hocus
    - 初始化子模块：git submodule init hocus

  - 克隆含子模块的版本库：git submodule update 子模块名  
    cd work  
    git clone magic new-magic  
    cd new-magic  

    git submodule   
    git submodule init hocus  
    git submodule update hocus

  - 改变子模块的版本

  - 使用子模块时要提防的错误

    - git add 确保结尾没有“\”
    - submodule update 先检查提交
    - 添加新内容到本地自模块版本库，要检出正确分支
    - 修改提交，确保改动被送回远程版本库

### 6. 高级功能

- 压缩版本库
  - git gc  整理版本库、优化Git内部存储历史记录
  - git gc <--aggressive>  重新计算增量存储单元
- 到处版本库
  - 创建版本快照:git archive 格式类型 指定版本
  - git archive --format=<tar/zip> <--prefix=父目录> 转换格式  
    git archive --format=zip --prefix=mysite-release/ HEAD > mysite-release.zip     
    git archive --format=tar --prefix=mysite-release/ HEAD | gzip > mysite-release.tar.gz
- 分支变基
  - git rebase --continue/--skip/--abort
  - git rebase --onto master contacts search
- 重现隐藏的历史：git reflog
- 二分查找
  - git bisect start
  - git bisect bad
  - git bisect good 1.0
  - git bisect reset
  - git bisect visualize
  - git bisect log
  - git bisect replay <文件>
  - git bisect run







- ssh -keygen -t rsa -C 'maruami@qq.com'
- copy key

git push origin master

git pull origin master 拉去远程修改会改本地

git fetch 获取远程分支状态



git remote -v 



git merge  branchname

git checkput . 回退

git flow



- git reset -- files 使用当前分支上的修改覆盖暂缓区，用来撤销最后一次 git add files
- git checkout -- files 使用暂存区的修改覆盖工作目录，用来撤销本地修改







### merge



![img](https:////upload-images.jianshu.io/upload_images/3985563-29417c3862d0599c.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/521/format/webp)



merge命令把不同的分支合并起来。如上图，在实际开放中，我们可能从master分支中切出一个分支，然后进行开发完成需求，中间经过R3,R4,R5的commit记录，最后开发完成需要合入master中，这便用到了merge。

| **git fetch <remote>** | **merge之前先拉一下远程仓库最新代码** |
| ---------------------- | ------------------------------------- |
| **git merge <branch>** | **合并指定分支到当前分支**            |

一般在merge之后，会出现conflict，需要针对冲突情况，手动解除冲突。主要是因为两个用户修改了同一文件的同一块区域。如下图所示，需要手动解除。



![img](https:////upload-images.jianshu.io/upload_images/3985563-70440791ecd54631.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1000/format/webp)



### rebase



![img](https:////upload-images.jianshu.io/upload_images/3985563-8d4e5fc624c0a23b.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1000/format/webp)



rebase又称为衍合，是合并的另外一种选择。

在开始阶段，我们处于new分支上，执行`git rebase dev`，那么new分支上新的commit都在master分支上重演一遍，最后checkout切换回到new分支。这一点与merge是一样的，合并前后所处的分支并没有改变。`git rebase dev`，通俗的解释就是new分支想站在dev的肩膀上继续下去。rebase也需要手动解决冲突。

**rebase与merge的区别**

现在我们有这样的两个分支,test和master，提交如下：

```
      D---E test
     /
A---B---C---F master
```

在master执行`git merge test`,然后会得到如下结果：

```
      D--------E
     /          \
A---B---C---F----G   test, master
```

在master执行`git rebase test`，然后得到如下结果：

```
A---B---D---E---C'---F'   test, master
```

可以看到，merge操作会生成一个新的节点，之前的提交分开显示。而rebase操作不会生成新的节点，是将两个分支融合成一个线性的提交。

如果你想要一个干净的，没有merge commit的线性历史树，那么你应该选择git rebase
 如果你想保留完整的历史记录，并且想要避免重写commit history的风险，你应该选择使用git merge

### reset



![img](https:////upload-images.jianshu.io/upload_images/3985563-2d41240c43bc3f2e.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/831/format/webp)



reset命令把当前分支指向另一个位置，并且相应的变动工作区和暂存区。

| **git reset —soft <commit>**  | 只改变提交点，暂存区和工作目录的内容都不改变               |
| ----------------------------- | ---------------------------------------------------------- |
| **git reset —mixed <commit>** | **改变提交点，同时改变暂存区的内容**                       |
| **git reset —hard <commit>**  | **暂存区、工作区的内容都会被修改到与提交点完全一致的状态** |
| **git reset --hard HEAD**     | **让工作区回到上次提交时的状态**                           |

### revert



![img](https:////upload-images.jianshu.io/upload_images/3985563-02aab40cb9b6efb1.jpg?imageMogr2/auto-orient/strip%7CimageView2/2/w/638/format/webp)



git revert用一个新提交来消除一个历史提交所做的任何修改。

**revert与reset的区别**



![img](https:////upload-images.jianshu.io/upload_images/3985563-93d402b6ebda56f8.jpg?imageMogr2/auto-orient/strip%7CimageView2/2/w/638/format/webp)



- git revert是用一次新的commit来回滚之前的commit，git reset是直接删除指定的commit。
- 在回滚这一操作上看，效果差不多。但是在日后继续merge以前的老版本时有区别。因为git revert是用一次逆向的commit“中和”之前的提交，因此日后合并老的branch时，导致这部分改变不会再次出现，减少冲突。但是git reset是之间把某些commit在某个branch上删除，因而和老的branch再次merge时，这些被回滚的commit应该还会被引入，产生很多冲突。关于这一点，不太理解的可以看[这篇文章](https://link.jianshu.com?t=http://yijiebuyi.com/blog/8f985d539566d0bf3b804df6be4e0c90.html)。
- git reset 是把HEAD向后移动了一下，而git revert是HEAD继续前进，只是新的commit的内容和要revert的内容正好相反，能够抵消要被revert的内容。

### push





| git push <remote><branch>     | 上传本地指定分支到远程仓库                 |
| ----------------------------- | ------------------------------------------ |
| **git push <remote> --force** | **强行推送当前分支到远程仓库，即使有冲突** |
| **git push <remote> --all**   | **推送所有分支到远程仓库**                 |

### 其他命令

​                **git cherry-pick \<commit>**  **选择一个commit，合并进当前分支**

