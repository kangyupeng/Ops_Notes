#  Yum  


 > 本章节主要是介绍了yum源相关的各方面使用，例如如何使用yum来管理软件包，如何创建yum源，自建yum仓库等



---

### 创建一个本地yum源

- **安装 createrepo**

  createrepo：用于创建yum源（软件仓库），即为存放于本地特定位置的众多rpm包建立索引，描述各包所需依赖信息，并形成元数据。

  ```bash
  [root@apt-1 ~]# yum -y install createrepo
  ```

- **将软件包放入指定路径**

  需要创建一个路径用来专门做我们的软件包仓库，之后的所有rpm包都可以放在这个路径下

- **使用 createrepo 创建依赖关系**

  具体可以参考下面的同步yum源


---

### 同步yum源

​	在CentOS或者RHEL中，我们的本地yum源是需要定期更新的，所以应该要如何去同步yum源呢？  
使用`reposync`命令可以用来同步yum源，需要安装`yum-utils`包。  
yum-utils：与Yum整合以多种方式扩大其机功能实用程序的集合，从而使其功能更强大，使用更方便。  
yum-utils 的更多功能介绍参考： [yum-utils](https://blog.csdn.net/xiaoxiao_22/article/details/7044583)



- **安装 yum-utils 与 createrepo**

  ```bash
  [root@apt-1 ~]# yum -y install yum-utils createrepo
  ```

- **使用 reposync 下载指定源的rpm包**

  reposync -r `<repoid>` -p `<down_path>`

  ```bash
  [root@apt-1 ~]# reposync -r base -p /test/repo/
  (1/10019): 389-ds-base-devel-1.3.8.4-15.el7.x86_64.rpm       | 271 kB  00:00:00     
  (2/10019): 389-ds-base-libs-1.3.8.4-15.el7.x86_64.rpm        | 698 kB  00:00:00     
  (3/10019): 389-ds-base-1.3.8.4-15.el7.x86_64.rpm             | 1.7 MB  00:00:00 
  ...
  ```

  具体参数的意思可以查看 `reposync --help`

- **下载完毕后创建依赖关系**

  通过 createrepo 也可以自己创建元数据信息，但是不会自动生成组之类的信息，也就是不能使用 yum grouplist   

  ```bash
  [root@apt-1 ~]# createrepo -d /test/repo/base/
  Spawning worker 0 with 5010 pkgs
  Spawning worker 1 with 5009 pkgs
  Workers Finished
  Saving Primary metadata
  Saving file lists metadata
  Saving other metadata
  Generating sqlite DBs
  Sqlite DBs complete
  
  [root@apt-1 ~]# ls /test/repo/base/
  Packages  repodata
  # 可以看到已经生成了一个 repodata 的目录来存放软件包之间的依赖关系
  ```

- **下载 metadata** 

  下载了 metadata  之后才能直接那些组的意思吧

  ```bash
  [root@apt-1 repodata]# reposync -r base -p /test/repo/ --downloadcomps --download-metadata 
  [root@apt-1 base]# ll
  total 1752
  -rw-r--r--. 1 root root 169676 Jul  2 21:13 bc140c8149fc43a5248fccff0daeef38182e49f6fe75d9b46db1206dc25a6c1c-c7-x86_64-comps.xml.gz
  -rw-r--r--. 1 root root 913932 Jul  2 21:13 comps.xml
  drwxr-xr-x. 2 root root 557056 Jul  2 20:00 Packages
  drwxr-xr-x. 2 root root   4096 Jul  2 20:17 repodata
  ```

- **加载元数据信息**

  ```bash
  [root@apt-1 base]# createrepo /test/repo/base/ -g comps.xml  
  Spawning worker 0 with 5010 pkgs
  Spawning worker 1 with 5009 pkgs
  Workers Finished
  Saving Primary metadata
  Saving file lists metadata
  Saving other metadata
  Generating sqlite DBs
  Sqlite DBs complete
  ```

- **加载组信息repodata里面的文件对比**

  自己生成依赖关系后的repodata目录：

  ```bash
  [root@apt-1 base]# ll repodata/
  total 27876
  -rw-r--r--. 1 root root 2951663 Jul  2 20:16 25cd2c29e5adb2b6f8a5b091237dd9f760e97815b37f2557f2cd1c12a5f294f0-primary.xml.gz
  -rw-r--r--. 1 root root 7514424 Jul  2 20:16 48632a48107dce6d5b7ffc08a404085e0c19a1a675492cc02b1bd166299ab51c-filelists.xml.gz
  -rw-r--r--. 1 root root 6289348 Jul  2 20:17 6614b3605d961a4aaec45d74ac4e5e713e517debb3ee454a1c91097955780697-primary.sqlite.bz2
  -rw-r--r--. 1 root root 1629864 Jul  2 20:16 7da504ebb833ae8a240e126d12e591c90a7b2066be4625b6760ef2cb895916f1-other.xml.gz
  -rw-r--r--. 1 root root 7461122 Jul  2 20:17 a0ec5a4708a1026db100d4799c404c9ed48a9371a4bab234a1355f86628a244a-filelists.sqlite.bz2
  -rw-r--r--. 1 root root 2684613 Jul  2 20:16 fbebcd3de05e22bd1cd526e594f235968401471d4a9aef3c1ad356b6d1965365-other.sqlite.bz2
  -rw-r--r--. 1 root root    3012 Jul  2 20:17 repomd.xml
  ```

  通过加载 comps.xml 生成后的repodata目录：

  ```
  [root@apt-1 base]# ll repodata/
  total 28940
  -rw-r--r--. 1 root root 2951663 Jul  2 21:23 25cd2c29e5adb2b6f8a5b091237dd9f760e97815b37f2557f2cd1c12a5f294f0-primary.xml.gz
  -rw-r--r--. 1 root root 7514424 Jul  2 21:23 48632a48107dce6d5b7ffc08a404085e0c19a1a675492cc02b1bd166299ab51c-filelists.xml.gz
  -rw-r--r--. 1 root root 6289348 Jul  2 21:24 6614b3605d961a4aaec45d74ac4e5e713e517debb3ee454a1c91097955780697-primary.sqlite.bz2
  -rw-r--r--. 1 root root 1629864 Jul  2 21:23 7da504ebb833ae8a240e126d12e591c90a7b2066be4625b6760ef2cb895916f1-other.xml.gz
  -rw-r--r--. 1 root root 7461122 Jul  2 21:23 a0ec5a4708a1026db100d4799c404c9ed48a9371a4bab234a1355f86628a244a-filelists.sqlite.bz2
  -rw-r--r--. 1 root root  913932 Jul  2 21:24 aced7d22b338fdf7c0a71ffcf32614e058f4422c42476d1f4b9e9364d567702f-comps.xml
  -rw-r--r--. 1 root root  169676 Jul  2 21:24 bc140c8149fc43a5248fccff0daeef38182e49f6fe75d9b46db1206dc25a6c1c-comps.xml.gz
  -rw-r--r--. 1 root root 2684613 Jul  2 21:23 fbebcd3de05e22bd1cd526e594f235968401471d4a9aef3c1ad356b6d1965365-other.sqlite.bz2
  -rw-r--r--. 1 root root    3716 Jul  2 21:24 repomd.xml
  ```

  加载comps.xml文件的话，能看到grouplist的分组信息

- **修改yum源为本地yum源**

  ```bash
  [root@apt-1 yum.repos.d]# tar zcvf repo-bk.tar.gz CentOS-*
  [root@apt-1 yum.repos.d]# vim test-base.repo 
  [test-base]
  name=CentOS-$releasever - Test - Base
  baseurl=file:///test/repo/base/
  enabled=1
  gpgcheck=0
  ```

- **清除cache，创建cache**

  ```bash
  [root@apt-1 yum.repos.d]# yum clean all 
  [root@apt-1 yum.repos.d]# yum makecache 
  ```

- **查看 yum grouplist**

  ```bash
  [root@apt-1 yum.repos.d]# yum grouplist 
  Loaded plugins: fastestmirror, product-id, search-disabled-repos, subscription-manager
  This system is not registered with an entitlement server. You can use subscription-manager to register.
  There is no installed groups file.
  Maybe run: yum groups mark convert (see man yum)
  Loading mirror speeds from cached hostfile
  Available Environment Groups:
     Minimal Install
     Compute Node
     Infrastructure Server
     File and Print Server
     Basic Web Server
     Virtualization Host
     Server with GUI
     GNOME Desktop
     KDE Plasma Workspaces
     Development and Creative Workstation
  Available Groups:
     Compatibility Libraries
     Console Internet Tools
     Development Tools
     Graphical Administration Tools
     Legacy UNIX Compatibility
     Scientific Support
     Security Tools
     Smart Card Support
     System Administration Tools
     System Management
  Done
  ```


### 通过ftp的方式让局域网内其他机器共享本地yum源





### 通过HTTP的方式让局域网内其他机器共享本地yum源