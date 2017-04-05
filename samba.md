# Samba Server

-------------------------

以下基于centos7.2

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
```

### 启动服务
`systemctl start smb`

### 添加开机自启动
`systemctl enable smb`

### 添加用户
假设系统上有sm01用户：

`pdbedit -a sm01`

---------------------------

## 验证

### smbclient

#### 安装
`yum install samba-client`

#### 查看服务器信息
`smbclient -L 127.0.0.1 -U%`

#### 登录一个用户进行操作
`smbclient //127.0.0.1 -U sm01`

### mount

#### 安装
`yum install cifs-utils`

#### 挂载
交互:

`mount //127.0.0.1/homes /mnt -o user=sm01`

非交互:

`mount //127.0.0.1/homes /mnt -o user=sm01,password=***`

