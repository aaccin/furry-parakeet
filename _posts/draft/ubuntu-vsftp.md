title: Ubuntu VSFTP
date: 2015-9-24 12:37:15
tags: [ftp, ubuntu, build]
---

## 目标

    * 多用户
    * 权限控制在制定目录

<!-- more -->

## vsftp 探坑

  安装：

````
    sudo apt-get install vsftpd
````

  配置：

````
# When "listen" directive is enabled, vsftpd runs in standalone mode and
# listens on IPv4 sockets. This directive cannot be used in conjunction
# with the listen_ipv6 directive.
# 设定该vsftpd服务工作在 StandAlone 模式下。
# 顺便展开说明一下，所谓 StandAlone 模式就是该服务拥有自己的守护进程支持，
# 在ps -A命令下我们将可用看到vsftpd的守护进程名。
# 如果不想工作在StandAlone模式下，则可以选择SuperDaemon模式，
# 在该模式下 vsftpd 将没有自己的守护进程，而是由超级守护进程 xinetd 全权代理，
# 与此同时，vsftp服务的许多功能将得不到实现。
listen=YES

#默认FTP服务器端口号是21，出于安全目的，有时需修改默认端口号
listen_port=18881

# 只监听来访问192.168.1.100(适用本机多网卡、多IP的情况)的FTP服务请求
#listen_address=192.168.1.100

# Allow anonymous FTP? (Beware - allowed by default if you comment this out).
# 设定不允许匿名访问
anonymous_enable=NO

# Uncomment this to allow local users to log in.
# 设定本地用户可以访问。
# 注意：主要是为虚拟宿主用户，如果该项目设定为NO那么所有虚拟用户将无法访问
local_enable=YES

# Uncomment this to enable any form of FTP write command.
# 设定可以进行写操作
write_enable=YES

# Default umask for local users is 077. You may wish to change this to 022,
# if your users expect that (022 is used by most other ftpd's)
# 设定上传后文件的权限掩码
local_umask=022

# Uncomment this to allow the anonymous FTP user to upload files. This only
# has an effect if the above global write enable is activated. Also, you will
# obviously need to create a directory writable by the FTP user.
# 禁止匿名用户上传
anon_upload_enable=NO

# Uncomment this if you want the anonymous FTP user to be able to create
# new directories.
# 禁止匿名用户建立目录
anon_mkdir_write_enable=NO

# Activate directory messages - messages given to remote users when they
# go into a certain directory.
# 设定开启目录标语功能
dirmessage_enable=YES

# Activate logging of uploads/downloads.
# 表明FTP服务器记录上传下载的情况
# 设定开启 wu-ftp 服务器格式记录功能
xferlog_enable=YES
xferlog_file=/var/log/xferlog

# You may override where the log file goes if you like. The default is shown below.
# vsftpd_log_file所指定的文件，即/var/log/vsftpd.log也将用来记录服务器的传输情况
# 设定 vsftpd 的服务日志保存路径。
# 注意，该文件默认不存在。必须要手动touch出来，
# 并且由于这里更改了vsftpd的服务宿主用户为手动建立的vsftpd。
# 必须注意给与该用户对日志的写入权限，否则服务将启动失败。
dual_log_enable=YES
vsftpd_log_file=/var/log/vsftpd.log

# 是否将原本输出到/var/log/vsftpd.log中的日志，输出到系统日志
# syslog_enable

# Make sure PORT transfer connections originate from port 20 (ftp-data).
# 设定端口20进行数据连接
connect_from_port_20=YES

# If you want, you can arrange for uploaded anonymous files to be owned by
# a different user. Note! Using "root" for uploaded files is not
# recommended!
# 设定禁止上传文件更改属主
chown_uploads=YES
chown_username=root

# If you want, you can have your log file in standard ftpd xferlog format
# 设定日志使用标准的记录格式。
xferlog_std_format=YES

# You may change the default value for timing out an idle session.
# 设定空闲连接超时时间，这里使用默认。
# 将具体数值留给每个具体用户具体指定，当然如果不指定的话，还是使用这里的默认值600，单位秒。
idle_session_timeout=600

# You may change the default value for timing out a data connection.
# 设定单次最大连续传输时间，这里使用默认。
# 将具体数值留给每个具体用户具体指定，当然如果不指定的话，还是使用这里的默认值120，单位秒。
#data_connection_timeout=120

# It is recommended that you define on your system a unique user which the
# ftp server can use as a totally isolated and unprivileged user.
# 设定支撑vsftpd服务的宿主用户为手动建立的vsftpd用户。
# 注意，一旦做出更改宿主用户后，必须注意一起与该服务相关的读写文件的读写赋权问题。
# 比如日志文件就必须给与该用户写入权限等。
nopriv_user=ftp

# Enable this and the server will recognise asynchronous ABOR requests. Not
# recommended for security (the code is non-trivial). Not enabling it,
# however, may confuse older FTP clients.
# 是否识别异步ABOR请求
# 如果FTP client会下达“async ABOR”这个指令时，这个设定才需要启用
# 而一般此设定并不安全，所以通常将其取消
#async_abor_enable=YES

# By default the server will pretend to allow ASCII mode but in fact ignore
# the request. Turn on the below options to have the server actually do ASCII
# mangling on files when in ASCII mode.
# Beware that on some FTP servers, ASCII support allows a denial of service
# attack (DoS) via the command "SIZE /big/file" in ASCII mode. vsftpd
# predicted this attack and has always been safe, reporting the size of the
# raw file.
# ASCII mangling is a horrible feature of the protocol.
# 是否以ASCII方式传输数据。默认情况下，服务器会忽略ASCII方式的请求
# 启用此选项将允许服务器以ASCII方式传输数据
# 不过，这样可能会导致由"SIZE /big/file"方式引起的DoS攻击
#ascii_upload_enable=YES
#ascii_download_enable=YES

# You may fully customise the login banner string:
# 设定ftp的登陆标语。
ftpd_banner=Welcome to login the FTP server ^_^
#
# You may specify a file of disallowed anonymous e-mail addresses. Apparently
# useful for combatting certain DoS attacks.
#deny_email_enable=YES
# (default follows)
#banned_email_file=/etc/vsftpd/banned_emails

# You may specify an explicit list of local users to chroot() to their home
# directory. If chroot_local_user is YES, then this list becomes a list of
# users to NOT chroot().
# 将使用者限制在自己的家目录之内
chroot_local_user=YES
# 启用不被 chroot 的使用者账号(如果 chroot_local_user=NO 目的相反)
chroot_list_enable=YES
# 不被 chroot 的使用者账号的列表文件
chroot_list_file=/etc/vsftpd/chroot_list ## is empty file

# You may activate the "-R" option to the builtin ls. This is disabled by
# default to avoid remote users being able to cause excessive I/O on large
# sites. However, some broken FTP clients such as "ncftp" and "mirror" assume
# the presence of the "-R" option, so there is a strong case for enabling it.
# 是否允许递归查询。默认为关闭，以防止远程用户造成过量的I/O
ls_recurse_enable=NO

# This directive enables listening on IPv6 sockets. To listen on IPv4 and IPv6
# sockets, you must run two copies of vsftpd whith two configuration files.
# Make sure, that one of the listen options is commented !!
# 设定是否支持IPV6。如要同时监听IPv4和IPv6端口
# 则必须运行两套vsftpd，采用两套配置文件
# 同时确保其中有一个监听选项是被注释掉的
#listen_ipv6=YES

# 以下这些是关于vsftpd虚拟用户支持的重要配置项目。
# 默认 vsftpd.conf 中不包含这些设定项目，需要自己手动添加配置。

# 设定启用虚拟用户功能。
guest_enable=YES

# 指定虚拟用户的属主用户。
guest_username=ftp168

# 设定虚拟用户的权限符合他们的属主用户。
virtual_use_local_privs=YES

# 设定虚拟用户个人vsftp的配置文件存放路径。
# 也就是说，这个被指定的目录里，将存放每个Vsftp虚拟用户个性的配置文件，
# 一个需要注意的地方就是这些配置文件名必须和虚拟用户名相同。
user_config_dir=/etc/vsftpd/vusers_conf ## is dir for user

# 设定PAM服务下vsftpd的验证配置文件名。
# 因此，PAM验证将参考/etc/pam.d/下的vsftpd文件配置。
pam_service_name=ftp
#pam_service_name=/etc/pam.d/vsftpd

# 设定userlist_file中的用户将不得使用FTP
userlist_enable=YES
userlist_deny=YES
userlist_file=/etc/vsftpd/user_list  ## is file empty

# The following entries are added for supporting virtual ftp users.
# 是否使用tcp_wrappers作为主机访问控制方式
# 可以实现linux系统中网络服务的基于主机地址的访问控制
# /etc/hosts.allow控制可以访问本机的IP地址
# /etc/hosts.deny控制禁止访问本机的IP
# 如果两个文件的配置有冲突，以/etc/hosts.deny为准。
tcp_wrappers=YES

````

    用户配置：

* 只读模式

````
local_root=/var/ftpvuser/vuser_name
anonymous_enable=NO
write_enable=NO
local_umask=022
anon_upload_enable=NO
anon_mkdir_write_enable=NO
#idle_session_timeout=300
#data_connection_timeout=90
#max_clients=6
#max_per_ip=6
#local_max_rate=25000
````

* 读写模式

````
local_root=/var/ftpvuser/vuser_name
write_enable=YES
anon_umask=022
anon_world_readable_only=NO
anon_upload_enable=YES
anon_mkdir_write_enable=YES
anon_other_write_enable=YES
#idle_session_timeout=30
#data_connection_timeout=30
#max_clients=6
#max_per_ip=6
#local_max_rate=25000
````

<abbr title="End of file">EOF</abbr>
