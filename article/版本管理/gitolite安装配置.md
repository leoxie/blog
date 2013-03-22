gitolite安装配置
================

安装
---------
1. 创建新用户git

		sudo adduser --system --shell /bin/bash --group git

2. 配置git用户的sudo权限

		chmod u+w /etc/sudoers
		root ALL=(ALL) ALL
		git  ALL=(ALL) ALL
		chmod u-w /etc/sudoers

3. 配置SSH密钥访问

4. 安装git-core

		su - git
		sudo apt-get install git-core

5. 配置git用户语言

		vi ~/.bashrc
		export LANG=en_US:zh_CN.UTF-8
		export LC_ALL=C
		source ~/.bashrc

6. 将客户端的公钥(admin.pub)拷贝到git用户目录下

		sudo chown -R git admin.pub

7. 安装gitolite

		git clone git://github.com/sitaramc/gitolite
        mkdir -p $HOME/bin
        gitolite/install -to $HOME/bin
        $HOME/bin/gitolite setup -pk admin.pub

配置
---------
1. 克隆安装后的gitolite-admin

		git clone git@host:gitolite-admin

2. 添加用户

		将用户的xxx.pub(xxx为用户名)复制到gitolite-admin/keydir/下
		然后git push origin master即可

3. 用户组和版本库组
	在 conf/gitolite.conf 授权文件中，可以定义用户组或者版本库组。
	组名称以 @ 字符开头，可以包含一个或多个成员。成员之间用空格分开。
	- 用户组

			@admin = jiangxin wangsheng
			@test = bob leo
	- 版本库组

			@myrepos    =   foo
			@myrepos    =   bar

			repo @myrepos
    		RW      =   alice
    - 任何组可以嵌套

    		@staff = @admin @engineers tester1

4. 配置访问权限
	- 普通的库

			repo foo
            RW+         =   alice
            RW          =   bob
            R           =   carol
    > `RW+`：  读写权限(强制push)
    > `RW`：   读写权限(不能强制push)
    > `R`：    只读权限
	- 各用户的个人库

			repo users/CREATOR/.+$
			C   =  @all
    		RW+ =  CREATOR
			RW  =  WRITERS
			R   =  READERS
	> `C`：      创建权限
	> `@all`：   所有有权限访问的用户
	> `CREATOR`: 创建该库的用户角色
	> `WRITERS`：有读写权限的用户角色
	> `READERS`: 有只读权限的用户角色
	> 该库只要用户在客户端git init后，push到服务器上就自动创建
	> 创建库的用户可以给自己的库再分配权限(见第5节)
	- 控制分支和标签的库

			repo foo
            RW+                     =   alice
            -   master              =   bob
            -   refs/tags/v[0-9]    =   bob
            RW                      =   bob
            RW  refs/heads/feature/ =   carol
            R                       =   dave
    > `-`：        禁止写操作
    > `master`:    以master开头的所有分支
    > `refs/tags/v[0-9]`: 表示所有的tag
    > `refs/heads/feature/`：表示以 feature开头的分支 等同直接写 feature
    - 用户分支库

    		repo test/repo4
    		RW+CD = @administrators
    		RW+CD refs/personal/USER/  = @all
    		RW+    master = @dev
    > `D`:     删除分支
    > `refs/personal/USER/`： USER会替换成用户名，即用户可以自由创建和删除自己的分支，不能修改别人的分支

5. 配置用户项目下的访问权限
	在Git Bash下，执行

		ssh git@host perms users/xxx/yyy + READERS user
		ssh git@host perms users/xxx/yyy - WRITERS user
	> `+-`： 代表增加和删除
	> `READERS`: 有读权限的角色
	> `WRITERS`: 有写权限的角色
	> `user`： 用户名，可以多个，使用空格隔开


	

GitWeb集成
-----------
1. 安装
	
		sudo apt-get install highlight gitweb
2. 配置
		
		sudo vi /etc/gitweb.conf
		# change $projectroot to /home/git/repositories
		# change $projects_list to /home/git/projects.list
		# Add this at the end
		---
		$feature{'blame'}{'default'} = [1];
		$feature{'blame'}{'override'} = 1;

		$feature{'pickaxe'}{'default'} = [1];
		$feature{'pickaxe'}{'override'} = 1;

		$feature{'snapshot'}{'default'} = [1];
		$feature{'snapshot'}{'override'} = 1;

		$feature{'search'}{'default'} = [1];

		$feature{'grep'}{'default'} = [1];
		$feature{'grep'}{'override'} = 1;

		$feature{'show-sizes'}{'default'} = [1];
		$feature{'show-sizes'}{'override'} = 1;

		$feature{'avatar'}{'default'} = ['gravatar'];
		$feature{'avatar'}{'override'} = 1;

		$feature{'highlight'}{'default'} = [1];
		$feature{'highlight'}{'override'} = 1;
		---

		sudo usermod -a -G git www-data
		sudo chmod g+r /home/git/projects.list
		sudo chmod -R g+rx /home/git/repositories
		sudo service apache2 restart

3. 加载样式

		sudo mv /usr/share/gitweb/static/ /usr/share/gitweb/static-bak
		sudo mkdir /usr/share/gitweb/static/
		cd /tmp
		git clone git://github.com/kogakure/gitweb-theme.git
		cd gitweb-theme
		sudo cp * /usr/share/gitweb/static/

4. 应用到版本库

		repo testing
  		  RW+ = @all
  		  R = gitweb
