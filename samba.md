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
        log level = 0
        vfs object = full_audit #开启审计功能
            full_audit:prefix = %S|%u|%I|%m
            full_audit:success = chdir mkdir open opendir read rename rmdir write link unlink
            full_audit:failure = none
            full_audit:facility = local7
            full_audit:priority = notice
        smb encrypt = mandatory #强制使用加密协议，防止自动协商到明文传输，如果客户端不支持加密则协商失败
        security = user
        passdb backend = tdbsam
        load printers = no
        cups options = raw
[AM]
        comment = AM理财
        browseable = yes
        writable = yes
        create mask = 664
        directory mask = 775
        force group = am
        valid users = +am
        path = /home/99bill/AM
[VAS]
        comment = VAS权益
        browseable = yes
        writable = yes
        create mask = 664
        directory mask = 775
        force group = vas
        valid users = +vas
        path = /home/99bill/VAS
[CSD]
        comment = CSD信用
        browseable = yes
        writable = yes
        create mask = 664
        directory mask = 775
        force group = csd
        valid users = +csd
        path = /home/99bill/CSD
[PCM]
        comment = PCM合规
        browseable = yes
        writable = yes
        create mask = 664
        directory mask = 775
        force group = pcm
        valid users = +pcm
        path = /home/99bill/PCM
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


## 配置审计功能
```
in /etc/rsyslog.d/50-smbd_audit.conf i have to tell rsyslogd to direct audit logs to a separate file:

if $programname == 'smbd_audit' then /var/log/samba/audit.log
if $programname == 'smbd_audit' then ~
in /etc/samba/smb.conf i tell samba to generate such information:

vfs object = full_audit
full_audit:prefix = %S|%u|%I|%m
full_audit:success = chdir mkdir open opendir read rename rmdir write link unlink
full_audit:failure = none
full_audit:facility = local7
full_audit:priority = notice
and finally tell logrotate to archive the files daily – /etc/logrotate.d/smbd_audit

/var/log/samba/audit.log
{
rotate 7
daily
missingok
notifempty
delaycompress
compress
postrotate
 invoke-rc.d rsyslog rotate > /dev/null
endscript
}
finally restart both samba and rsyslog and enjoy the logs:

service smbd restart
service rsyslogd restart
tail -f /var/log/samba/audit.log
```

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
`smbclient -L 127.0.0.1 -U% -m SMB3`

#### 登录一个用户进行操作
`smbclient //127.0.0.1/homes -U cuihengchun -m SMB3`


-----------------------------------------------------------
## 普通用户修改密码

1. 检查是否有smbpasswd命令，没有的话需要安装

    `yum install samba-common-tools`

2. 修改密码命令：

    `smbpasswd -r 10.214.96.85 -U zhouwenjun9`

-----------------------------------------------------------

## 使用

### linux

#### 普通使用

1. 检查是否有smbclient命令,没有的话需要安装

    `yum install samba-client`

2. 查看目录服务端的共享目录,SAMBASERVER替换为samba服务器地址，USERNAME替换为实际的用户名：

    `smbclient -L SAMBASERVER -U USERNAME -m SMB3`

3. 使用smbclient命令登录，SAMBASERVER替换为samba服务器地址，USERNAME替换为实际的用户名,homes替换为需要访问的目录:

    `smbclient //SAMBASERVER/homes -U USERNAME -m SMB3`

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
