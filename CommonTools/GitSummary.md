## 一、克隆

1. 克隆带有子模块项目的两种情况
   - 未克隆父项目时：`git clone --recursive https://github.com/example/example.git`
   - 已经克隆了父项目：①`git submodule init` ② `git submodule update`

2. 克隆项目报错
   * ` RPC failed; curl 56 OpenSSL SSL_read: SSL_ERROR_SYSCALL, errno 10054 fatal: the remote end hung up `。 解决方法1:① `git config http.postBuffer 524288000` ② `git config http.sslVerify "false"` 。 解决方法2:① `env GIT_SSL_NO_VERIFY=true git clone https://<host_name/git/project.git` ② `git config http.sslVerify "false"`

3. github克隆缓慢，如果是自己的仓库，则可以通过码云：新建项目->导入已有项目解决该问题

## 二、回滚

1. 回滚单个文件到指定版本
   * 命令模式
     * `git reset VERSIONNUM FILENAME` 执行回滚命令(版本号可在log中查看)
     * `git checkout FILENAME` 用旧版本文件覆盖掉工作区中现有的文件(此步骤最关键)
     * 该操作可以通过revert恢复
2. 回滚整个分支

* `git reset --soft` 
* `git reset --mix` 不触及工作树，重置索引。
* `git reset --hard VERSIONNUM` 硬回滚 **本地所有的修改以及Log**都回退到指定版本，且指定版本后的修改都查看不到

## 三、其他问题

1. `git pull`时提示"Auto packing the repository for optimum performance.You may also run 'git gc' manually."
   * 方法1：执行命令 `git gc`
   * 方法2:执行命令 `git fsck --lost-found`、`git gc --prune=now`
2. 回滚commit但没push的修改(会从本地log记录删除)
   * `git reset --soft HEAD^`
   * *注意:HEAD表示头指针，永远指向本地仓库的当前最新位置。不要用hard，因为本地修改也会被抹掉*
3. 想恢复到之前某个提交的版本，且那个版本之后提交的版本我们都不要了
   
   * `git reset --hard 目标版本号`
4. 撤销之前的reset --hard的方法
   * `git reflog` 查看所有当前本地仓库的日志，找到reset的前一条的版本号
   * `git reset --hard ***` 就能回滚我们的回滚操作了
5.  分支之间的修改想仅仅合并指定comID的文件的方法(适用于文本文件，不适用于资源文件)
   * 首先得到原分支修改的commitID，然后切换到目标分支操作
   * git cherry-pick commitID` 执行该命令 如果没发生冲突，则会自动执行commit
   * 最后记得push一下

6. git想恢复某一个或多个指定文件，到指定版本
   * 找到想恢复版本的上一个版本的commitId
   * git checkout commitId file1.txt file2.txt(多个文件之间用空格隔开)
   * 最后commit+push即可

7. git开启代理vpn，命令如下

   * 开启`git config --global https.proxy http://127.0.0.1:10808
     git config --global https.proxy https://127.0.0.1:10808
     git config --global http.proxy 'socks5://127.0.0.1:10808' 
     git config --global https.proxy 'socks5://127.0.0.1:10808'`

   * 关闭 `git config --global --unset http.proxy`

     `git config --global --unset https.proxy`

     