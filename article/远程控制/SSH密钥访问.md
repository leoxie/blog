SSH密钥访问
===========

服务端配置
----------------
1. 	备份配置文件
	
		cp /etc/ssh/sshd_config /etc/ssh/sshd_config.bak

2. 	编辑/etc/ssh/sshd_config，找到 Authentication 配置

		RSAAuthentication yes 
   		PubkeyAuthentication yes 
   		AuthorizedKeysFile %h/.ssh/authorized_keys    //此处对应名称可以

3.  切换用户到需要密钥访问的用户下

		su - xxx  

4.  生成私钥和公钥(.pub) 私钥复制到客户端后可删除
		
		ssh-keygen

客户端设置
----------------
#### PuTTy设置
1.  将私钥拷贝到客户端
2.  使用PuTTyGEN生成ppk格式
		
		Load，选择私钥，输入passphrase（如果有的话），Save PrivateKey

3.  配置PuTTy

		Session 中输入服务器的 IP 地址，在 Connection->SSH->Auth 下点击 Browse按钮，
		选择刚才生成好的私钥。然后回到 Connection 选项，在 Auto-login username 中
		输入证书所属的用户名。回到 Session 选项卡，输入个名字点 Save 保存下这个 Session。
		点击底部的 Open 应该就可以通过证书认证登录到服务器了。
		如果有 passphrase 的话，登录过程中会要求输入 passphrase，否则将会直接登录到服务器上

SSH主机别名
----------------
在实际应用中，有时需要使用多套公钥/私钥对，例如:

* 使用缺省的公钥访问 git 帐号，获取 shell，进行管理员维护工作。
* 使用单独创建的公钥访问 git 帐号，执行 git 命令。
* 访问 github（免费的Git服务托管商）采用其他公钥。

如何创建指定名称的公钥/私钥对呢？还是用 ssh-keygen 命令，如下:

	$ ssh-keygen -f ~/.ssh/<filename>

> 注：
  - 将 `<filename>` 替换为有意义的名称。
  - 会在 ~/.ssh 目录下创建指定的公钥/私钥对。
  - 文件`<filename>` 是私钥，文件 `<filename>.pub` 是公钥。

将新生成的公钥添加到远程主机的 .ssh/authorized_keys 文件中，建立新的公钥认证。例如:
	
	$ ssh-copy-id -i .ssh/<filename>.pub user@server

这样，就有两个公钥用于登录主机 server，那么当执行下面的 ssh 登录指令，用到的是那个公钥呢？
	
	$ ssh user@server

当然是缺省公钥 ~/.ssh/id_rsa.pub 。  
那么如何用新建的公钥连接 server 呢？
SSH 的客户端配置文件 ~/.ssh/config 可以通过创建主机别名，在连接主机时，使用特定的公钥。
	
	chmod 600 ~/.ssh/config
例如 ~/.ssh/config 文件中的下列配置：

    Host github.com
     IdentityFile ~/.ssh/github
     User git

当执行
	
	$ ssh github

或者执行

	$ git clone github:path/to/repo.git

> 含义为：
  - 登录的 SSH 主机为 github.com 。
  - 登录时使用的用户名为 git 。
  - 认证时使用的公钥文件为 ~/.ssh/github.pub 。
