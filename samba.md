# Samba 文件服务器配置和使用方法

以下基于centos7.2
-----------------------------------------------


## 配置

### 安装
`yum install samba`

### 修改配置文件
`/etc/samba/smb.conf`
```ini
[global]
	workgroup = MYGROUP
	server string = Samba Server Version %v
	log file = /var/log/samba/log.%m
	max log size = 50
	security = user
	passdb backend = tdbsam
[homes]
	comment = Home Directories
	browseable = no
	writable = yes
	valid users = %S
    path = /home/%U
    root preexec = /usr/bin/su -l "%U" -c true #用于为ldap用户自动创建家目录，如果没有配合ldap，不需要这行配置
```

### 启动服务
`systemctl start smb`

### 添加开机自启动
`systemctl enable smb`

---------------------------------------------------

## 添加用户授权

假设系统上有cuihengchun用户，输入命令：
`pdbedit -a cuihengchun`

需要交互式的设置独立的密码，和系统登录密码不冲突

```
pdbedit -a cuihengchun
new password:
retype new password:
Unix username: cuihengchun
NT username: 
Account Flags: [U ]
User SID: S-1-5-21-4100736040-3121364991-669732443-1001
Primary Group SID: S-1-5-21-4100736040-3121364991-669732443-513
Full Name: cuihengchun
Home Directory: \\cpvl-samba-a01\cuihengchun
HomeDir Drive: 
Logon Script: 
Profile Path: \\cpvl-samba-a01\cuihengchun\profile
Domain: CPVL-SAMBA-A01
Account desc: 
Workstations: 
Munged dial: 
Logon time: 0
Logoff time: Wed, 06 Feb 2036 23:06:39 HKT
Kickoff time: Wed, 06 Feb 2036 23:06:39 HKT
Password last set: Tue, 11 Apr 2017 10:58:34 HKT
Password can change: Tue, 11 Apr 2017 10:58:34 HKT
Password must change: never
Last bad password : 0
Bad password count : 0
Logon hours : FFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFF
```

---------------------------------------------------------

## 验证

### smbclient

#### 安装
`yum install samba-client`

#### 查看服务器信息
`smbclient -L 127.0.0.1 -U%`

#### 登录一个用户进行操作
`smbclient //127.0.0.1/homes -U cuihengchun`


-----------------------------------------------------------

## 使用

### linux

#### 普通使用

1. 检查是否有smbclient命令,没有的话需要安装

    `yum install samba-client`

2. 使用smbclient命令登录，SAMBASERVER替换为samba服务器地址，USERNAME替换为实际的用户名:

    `smbclient //SAMBASERVER/homes -U USERNAME`

    *使用方法类似ftp命令行，输入help查看帮助菜单*

##### 常用命令

```
smb: \> help
?              allinfo        altname        archive        backup         
blocksize      cancel         case_sensitive cd             chmod          
chown          close          del            dir            du             
echo           exit           get            getfacl        geteas         
hardlink       help           history        iosize         lcd            
link           lock           lowercase      ls             l              
mask           md             mget           mkdir          more           
mput           newer          notify         open           posix          
posix_encrypt  posix_open     posix_mkdir    posix_rmdir    posix_unlink   
posix_whoami   print          prompt         put            pwd            
q              queue          quit           readlink       rd             
recurse        reget          rename         reput          rm             
rmdir          showacls       setea          setmode        scopy          
stat           symlink        tar            tarmode        timeout        
translate      unlock         volume         vuid           wdel           
logon          listconnect    showconnect    tcon           tdis           
tid            logoff         ..             !
```

##### 上传/下载文件
`put`/`get`

##### 浏览文件
`cd`/`ls`/`pwd`

##### 查看和修改文件权限属主(示例中test是文件名)

```
smb: \> stat test
File: \test
Size: 0           	Blocks: 0	directory
Inode: 278660	Links: 3
Access: (0755/drwxr-xr-x)	Uid: 1000	Gid: 100
Access: 2017-04-11 17:45:48 +0800
Modify: 2015-08-07 10:36:12 +0800
Change: 2017-04-17 15:46:53 +0800
smb: \> chown 1000 1001 test
smb: \> stat
stat file
smb: \> stat test
File: \test
Size: 0           	Blocks: 0	directory
Inode: 278660	Links: 3
Access: (0755/drwxr-xr-x)	Uid: 1000	Gid: 1001
Access: 2017-04-11 17:45:48 +0800
Modify: 2015-08-07 10:36:12 +0800
Change: 2017-04-17 15:47:04 +0800
```


#### mount

用mount命令挂载，类似nfs

1. 检查是否有mount.cifs命令，如果没有，安装软件包

    `yum install cifs-utils`

2. 挂载

    密码可以交互方式输入也可以非交互方式输入，SAMBASERVER替换为实际的samba服务器地址，USERNAME替换为实际的用户名,PASSWORD替换为实际的密码，homes代表本用户家目录，不用修改:

    交互:

        `mount //SAMBASERVER/homes /mnt -o user=USERNAME`

    非交互:

        `mount //SAMBASERVER/homes /mnt -o user=USERNAME,password=PASSWORD`

### windows
类似windows共享文件夹
