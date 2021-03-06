Git命令集
===========
[配置](http://blog.jobbole.com/25775/)
-----------
1. 文件说明
	- /etc/gitconfig ：系统中对所有用户都普遍适用的配置。

			git config --system
	- ~/.gitconfig ：用户目录下的配置文件只适用于该用户。

			git config --global
	- .git/config : 工作目录中的配置文件只适用于该项目。

			git config --local
2. 用户信息
	
		git config --global user.name "John Doe"
		git config --global user.email johndoe@example.com

3. 文本编辑器

		git config --global core.editor emacs

4. [差异分析工具](http://blog.jobbole.com/26131/#id372)

		git config --global merge.tool vimdiff

5. 查看配置信息

		git config --list

6. 获取帮助

		git help config

7. 别名设置

		git config --global alias.co checkout
		git config --global alias.br branch
		git config --global alias.ci commit
		git config --global alias.st status 

8. [设置全局忽略文件](https://help.github.com/articles/ignoring-files)

		git config --global core.excludesfile ~/.gitignore_global

9. 着色

		git config --global color.ui true

[基础](http://blog.jobbole.com/25808/)
-----------
1. 初始化项目

		git init

2. 克隆项目

		git clone [url] [NewName]
		git clone git://github.com/schacon/grit.git
		git clone git://github.com/schacon/grit.git newgrit

3. 查看状态

		git status

4. 将文件加入跟踪
		
		git add [file]

5. 查看差异

		git diff [--staged]

6. 提交更新

		git commit [-m xxx]

7. 移除文件

		git rm [--cache] xxx

8. 移动文件

		git mv file_from file_to

9. 查看日志

		git log [-p] [-n] [--stat]

10. 修改最后一次提交

		git commit --amend

[远程仓库](http://blog.jobbole.com/25808/#id259)
-----------
1. 查看

		git remote -v
		git remote show [remote-name]

2. 添加

		git remote add [shortname] [url]
		git remote add xxx_name git://github.com/xxx/xxx.git

3. 抓取数据

		git fetch [remote-name]

4. 推送数据

		git push [remote-name] [branch-name]
		git push origin master

5. 重命名和删除

		git remote rename xxx xxx_new
		git remote rm xxx

[标签Tag](http://blog.jobbole.com/25808/#id266)
-----------
1. 新建标签

		git tag -a v1.4 -m 'my version 1.4' //带附注
		git tag v1.4 //轻量级

2. 查看标签

		git tag 
		git tag show v1.4t

3. 推送标签

		git push origin [tagname] //指定的tag
		git push origin --tags //所有tag

[分支](http://blog.jobbole.com/25877/)
-----------
1. 新建分支

		git branch branch-name 

2. 切换分支

		git checkout [-b] branch-name // -b 新建并切换分支

3. 合并分支

		git merge branch-name //将指定分支合并到当前分支

4. 删除分支

		git branch -d /[-D] branch-name // -D 强制删除

5. 查看分支

		git branch [-v] [–-merge] [–-no-merged]

6. 远程分支
	- 推送远程分支

			git push remote-name branch-name[:remote-branch-name]
			git push origin xxx
			git push origin xxx:newxxx
	- 抓取远程分支

			git fetch remote-name
	- 将远程分支合并到当前分支

			git merge remote-name/branch-name
	- 分化出新的远程分支（跟踪分支）

			git chechout -b newbranch-name remote-name/branch-name
			git chechout --track remote-name/branch-name
	- 推拉数据（当前是跟踪分支的情况下）

			git pull // 拉取数据，并且合并到当前分支(fetch 拉取数据不合并)
			git push // 同远程仓库的推送数据)
	- 删除远程分支

			git push remote-name (此处为空格):remote-branch-name //类似推送远程分支，即用空分支来删除远程分支
7. 衍合(rebase)分支  
	把一个分支整合到另一个分支的办法有两种：merge 和 rebase。
	merge是将两者的结果合并，rebase是将一个分支的提交在另一分支上重新播放一次。  

	**一旦分支中的提交对象发布到公共仓库，就千万不要对该分支进行衍合操作。**  

	假设有3个分支(master、server、client)

		git rebase --onto master server client

	即：取出 client 分支，找出 client 分支和 server 分支的共同祖先之后 client 的变化，
	然后把它们在master 上重演一遍（切换到master分支，合并client）：

		git checkout master
		git merge client

	此时再rebase server分支:

		git rebase master server
		git checkout master
		git merge server

	最后删除server、client分支即可。  
	衍合只是在推送之前清理本地提交历史的手段，即本地整理分支，不能对已提交的对象进行rebase。

[分支范围](http://blog.jobbole.com/26112/#id344)
-------------
1. 双点

		git log master..experiment  //所有在 master 而不在 experiment 中的提交
		git log origin/master..     //查看要推送哪些commit去远程服务器

2. 三点

		git log master...experiment //两个引用中的一个包含但又不被两者同时包含

[储藏Stashing](http://blog.jobbole.com/26112)
-------------
切换分支时，当前分支有未提交的，但是又不想立即提交，可以暂存起来。

1. 暂存当前分支

		git stash
2. 查看暂存

		git stash list
3. 应用储存，默认是最近的，也可以指定
		
		git stash apply [stash@{2}]
4. 移除指定的储存
		
		git stash drop [stash@{2}]
5. 应用储存,并移除

		git stash pop
6. 从储藏中创建分支

		git stash branch branch-name

[子模块](http://blog.jobbole.com/26112/)
-------------
可以在项目下管理子模块，该子模块不会提交到仓库。

1. 添加子模块

		git submodule add git://xxx.git dirxxx
2. 初始化子模块

		git submodule init (第一次clone项目的时候可以进行)
3. 更新子模块
		
		git submodule update (会从仓库拉起最新的数据)
4. 切换子模块tag
		
		cd dirxxx
		git checkout xxx //xxx为tag或者分支名
 



