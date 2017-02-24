# 推荐一个好用的 makedown在线编辑器  

https://www.zybuluo.com/mdeditor

# Cento7下开发环境搭建(mysql 5.6.35+ redis)

## 安装依赖库
yum -y install wget vim git texinfo patch make cmake gcc gcc-c++ gcc-g77 flex bison file libtool libtool-libs autoconf kernel-devel libjpeg libjpeg-devel libpng libpng-devel libpng10 libpng10-devel gd gd-devel freetype freetype-devel libxml2 libxml2-devel zlib zlib-devel glib2 glib2-devel bzip2 bzip2-devel libevent libevent-devel ncurses ncurses-devel curl curl-devel e2fsprogs e2fsprogs-devel krb5 krb5-devel libidn libidn-devel openssl openssl-devel vim-minimal nano fonts-chinese gettext gettext-devel ncurses-devel gmp-devel pspell-devel unzip libcap diffutils

（gcc-g77，libpng10， libpng10-devel， gd-devel， libevent-devel， krb5，libidn-devel，fonts-chinese ，pspell-devel缺没有关系）


## 安装mysql
安装依赖
```
yum install net-tools perl
yum install perl-Module-Install.noarch
rpm -qa |grep -i mariadb
rpm -e mariadb-libs-5.5.44-2.el7.centos.x86_64 --nodeps
```
先创建组和用户再安装
```
groupadd mysql
useradd -s /sbin/nologin -M -g mysql mysql
```
安装mysql5.6.35
下载 MySQL-5.6.35-1.el7.x86_64.rpm-bundle.tar 解压得到下面几个rpm
```
[root@localhost mysql-5.6.35]# ls
MySQL-client-5.6.35-1.el7.x86_64.rpm
MySQL-devel-5.6.35-1.el7.x86_64.rpm
MySQL-embedded-5.6.35-1.el7.x86_64.rpm
MySQL-server-5.6.35-1.el7.x86_64.rpm
MySQL-shared-5.6.35-1.el7.x86_64.rpm
MySQL-shared-compat-5.6.35-1.el7.x86_64.rpm
MySQL-test-5.6.35-1.el7.x86_64.rpm
[root@localhost mysql-5.6.35]# rpm -ivh *.rpm
```

看安装时输出的提示内容：
```
	A RANDOM PASSWORD HAS BEEN SET FOR THE MySQL root USER !
	You will find that password in '/root/.mysql_secret'.

	You must change that password on your first connect,
	no other statement but 'SET PASSWORD' will be accepted.
	See the manual for the semantics of the 'password expired' flag.

	Also, the account for the anonymous user has been removed.

	In addition, you can run:

	  /usr/bin/mysql_secure_installation

	which will also give you the option of removing the test database.
	This is strongly recommended for production servers.

	See the manual for more instructions.

	Please report any problems at http://bugs.mysql.com/

	The latest information about MySQL is available on the web at

	  http://www.mysql.com

	Support MySQL by buying support/licenses at http://shop.mysql.com

	New default config file was created as /usr/my.cnf and
	will be used by default by the server when you start it.
	You may edit this file to change server settings
```

初始密码在： /root/.mysql_secret 这个文件里有
默认配置文件： /usr/my.cnf 

查看生成的随机密码
vim /root/.mysql_secret

```
# The random password set for the root user at Mon Feb 20 10:28:39 2017 (local time): 9lQbqBuyz7bz57x6
```
那么默认密码就是 9lQbqBuyz7bz57x6

修改root密码
mysqladmin -u root -p password 新密码

默认的配置本人持鄙视态度，所以要自己配置

创建配置： /etc/my.cnf

```

# Example MySQL config file for small systems.
#
# This is for a system with little memory (<= 64M) where MySQL is only used
# from time to time and it's important that the mysqld daemon
# doesn't use much resources.
#
# MySQL programs look for option files in a set of
# locations which depend on the deployment platform.
# You can copy this option file to one of those
# locations. For information about these locations, see:
# http://dev.mysql.com/doc/mysql/en/option-files.html
#
# In this file, you can use all long options that a program supports.
# If you want to know which options a program supports, run the program
# with the "--help" option.

# The following options will be passed to all MySQL clients
[client]
#password    = your_password
port        = 3306
socket        = /var/lib/mysql/mysql.sock
default-character-set=utf8
#character-set-client = gbk

# Here follows entries for some specific programs

# The MySQL server
[mysqld]
port        = 3306
socket        = /var/lib/mysql/mysql.sock
datadir = /usr/mysql/data
skip-external-locking
lower_case_table_names=1
character-set-server = utf8

#skip_name_resolve  

#key_buffer_size = 16M
#他人经验值，查询排序时所能使用的缓冲区大小。所以，对于内存在4GB左右的服务器推荐设置为6-8M。  
#sort_buffer_size = 1M


max_allowed_packet = 32M


#使用查询缓冲，MySQL将SELECT语句和查询结果存放在缓冲区中，今后对于同样的SELECT语句（区分大小写），将直接从缓冲区中读取结果
query_cache_size = 32M
#query_cache_type 0 代表不使用缓冲， 1 代表使用缓冲，2 代表根据需要使用。
#设置 1 代表缓冲永远有效，如果不需要缓冲，就需要使用如下语句：
#SELECT SQL_NO_CACHE * FROM my_table WHERE ...
#如果设置为 2 ，需要开启缓冲，可以用如下语句：
#SELECT SQL_CACHE * FROM my_table WHERE ...
query_cache_type= 1

#要求MySQL能有的连接数量。当主要MySQL线程在一个很短时间内得到非常多的连接请求，这就起作用，然后主线程花些时间(尽管很短)检查连接并且启动一个新线程。
#back_log值指出在MySQL暂时停止回答新请求之前的短时间内多少个请求可以被存在堆栈中
back_log=100

#临时表的大小（4G配置）
tmp_table_size = 256M  

#并发连接数目最大，
#指定MySQL允许的最大连接进程数。如果在访问论坛时经常出现Too Many Connections的错误提 示，则需要增大该参数值。
max_connections=500

#没找到具体说明，不过设置为32后 20天才创建了400多个线程 而以前一天就创建了上千个线程 所以还是有用的
thread_cache_size = 32

#目的待定，他人经验值，
#指定一个请求的最大连接时间，对于4GB左右内存的服务器可以设置为5-10。
interactive_timeout=28800
wait_timeout =28800


#设置为你的cpu数目x2,例如，只有一个cpu,那么thread_concurrency=2
#有2个cpu,那么thread_concurrency=4
thread_concurrency = 8

#增加table_open_cache，会增加文件描述符，当把table_open_cache设置为很大时，如果系统处理不了那么多文件描述符，那么就会出现客户端失效
#如果我们把table_open_cache设置小一点，那么mysql会随着table cache的不足，而关闭不用或者少用的表的cache，这样会释放文件描述符
#指定表高速缓存的大小。每当MySQL访问一个表时，如果在表缓冲区中还有空间，该表就被打开并放入其中，这样可以更快地访问表内容。
#通过检查峰值时间的状态值Open_tables和Opened_tables，可以决定是否需要增加table_cache的值。如果你发现open_tables等于table_cache，
#并且opened_tables在不断增长，那么你就需要增加table_cache的值了（上述状态值可以使用SHOW STATUS LIKE ‘Open%tables’获得）。
#注意，不能盲目地把table_cache设置成很大的值。如果设置得太高，可能会造成文件描述符不足，从而造成性能不稳定或者连接失败。对于有1G内存的机器，推荐值是128－256。
table_open_cache = 256

#读查询操作所能使用的缓冲区大小。和sort_buffer_size一样，该参数对应的分配内存也是每连接独享。
read_buffer_size = 4M   

#联合查询操作所能使用的缓冲区大小，和sort_buffer_size一样，该参数对应的分配内存也是每连接独享。  
join_buffer_size = 8M   

read_rnd_buffer_size = 256K
net_buffer_length = 2K
thread_stack = 128K

# Don't listen on a TCP/IP port at all. This can be a security enhancement,
# if all processes that need to connect to mysqld run on the same host.
# All interaction with mysqld must be made via Unix sockets or named pipes.
# Note that using this option without enabling named pipes on Windows
# (using the "enable-named-pipe" option) will render mysqld useless!
#
#skip-networking
server-id    = 2

# Uncomment the following if you want to log updates
#log_bin指定日志文件，如果不提供文件名，MySQL将自己产生缺省文件名。MySQL会在文件名后面自动添加数字引，每次启动服务时，都会重新生成一个新的二进制文件。
#此外，使用log-bin-index可以指定索引文件；使用binlog-do-db可以指定记录的数据库；使用binlog-ignore-db可以指定不记录的数据库。
#注意的是：binlog-do-db和binlog-ignore-db一次只指定一个数据库，指定多个数据库需要多个语句。而且，MySQL会将所有的数据库名称改成小写，在指定数据库时必须全部使用小写名字，否则不会起作用。
#关掉这个功能只需要在他前面加上#号
#log-bin=mysql-bin

# binary logging format - mixed recommended
#binlog_format=mixed

# Causes updates to non-transactional engines using statement format to be
# written directly to binary log. Before using this option make sure that
# there are no dependencies between transactional and non-transactional
# tables such as in the statement INSERT INTO t_myisam SELECT * FROM
# t_innodb; otherwise, slaves may diverge from the master.
#binlog_direct_non_transactional_updates=TRUE

# Uncomment the following if you are using InnoDB tables
#innodb_data_home_dir = /opt/database/mysql/data
#innodb_data_file_path = ibdata1:10M:autoextend
#innodb_log_group_home_dir = /opt/database/mysql/data
# You can set .._buffer_pool_size up to 50 - 80 %
# of RAM but beware of setting memory usage too high
#这是InnoDB最重要的设置，对InnoDB性能有决定性的影响。默认的设置只有8M，所以默认的数据库设置下面InnoDB性能很差。在只有InnoDB存储引擎的数据库服务器上面，可以设置60-80%的内存。
#更精确一点，在内存容量允许的情况下面设置比InnoDB tablespaces大10%的内存大小。
innodb_buffer_pool_size = 1G

#作用：用来存放Innodb的内部目录
#这个值不用分配太大，系统可以自动调。不用设置太高。通常比较大数据设置16Ｍ够用了，如果表比较多，可以适当的增大。如果这个值自动增加，会在error log有中显示的。
innodb_additional_mem_pool_size = 16M

#作用：指定日值的大小
#分配原则：几个日值成员大小加起来差不多和你的innodb_buffer_pool_size相等。上限为每个日值上限大小为4G.一般控制在几个ＬＯＧ文件相加大小在２Ｇ以内为佳。具体情况还需要看你的事务大小，数据大小为依据。
#说明：这个值分配的大小和数据库的写入速度，事务大小，异常重启后的恢复有很大的关系。
#该参数决定了recovery speed。太大的话recovery就会比较慢，太小了影响查询性能，一般取256M可以兼顾性能和recovery的速度
innodb_log_file_size = 256M

#磁盘速度是很慢的，直接将log写道磁盘会影响InnoDB的性能，该参数设定了log buffer的大小，一般4M。如果有大的blob操作，可以适当增大。
#作用：事务在内存中的缓冲。
#分配原则：控制在2-8M.这个值不用太多的。他里面的内存一般一秒钟写到磁盘一次。具体写入方式和你的事务提交方式有关。在Ｏｒａｃｌｅ等数据库了解这个，一般最大指定为３Ｍ比较合适。
innodb_log_buffer_size = 5M

#设置为0就是等到innodb_log_buffer_size列队满后再统一储存,默认为1
# 该参数设定了事务提交时内存中log信息的处理。
#1) =1时，在每个事务提交时，日志缓冲被写到日志文件，对日志文件做到磁盘操作的刷新。Truly ACID。速度慢。
#2) =2时，在每个事务提交时，日志缓冲被写到文件，但不对日志文件做到磁盘操作的刷新。只有操作系统崩溃或掉电才会删除最后一秒的事务，不然不会丢失事务。
#3) =0时， 日志缓冲每秒一次地被写到日志文件，并且对日志文件做到磁盘操作的刷新。任何mysqld进程的崩溃会删除崩溃前最后一秒的事务
innodb_flush_log_at_trx_commit = 2

#默认参数:innodb_lock_wait_timeout设置锁等待的时间是50s,
#因为有的锁等待超过了这个时间,所以抱错.
#你可以把这个时间加长,或者优化存储过程,事务避免过长时间的等待.
innodb_lock_wait_timeout = 600

#你的服务器CPU有几个就设置为几,建议用默认一般为8
innodb_thread_concurrency=8            


[mysqldump]
quick
max_allowed_packet = 16M

[mysql]
no-auto-rehash
# Remove the next comment character if you are not familiar with SQL
#safe-updates

[myisamchk]
key_buffer_size = 64M
sort_buffer_size = 4M

[mysqlhotcopy]
interactive-timeout

```

看我的配置里面 datadir路径是：
datadir = /usr/mysql/data

所以要设置权限：
chown -R mysql /usr/mysql/data
chgrp -R mysql /usr/mysql/.

默认的 datadir改了 启动脚本也要跟着改

vim /etc/init.d/mysql

```
#!/bin/sh
# Copyright Abandoned 1996 TCX DataKonsult AB & Monty Program KB & Detron HB
# This file is public domain and comes with NO WARRANTY of any kind

# MySQL daemon start/stop script.

# Usually this is put in /etc/init.d (at least on machines SYSV R4 based
# systems) and linked to /etc/rc3.d/S99mysql and /etc/rc0.d/K01mysql.
# When this is done the mysql server will be started when the machine is
# started and shut down when the systems goes down.

# Comments to support chkconfig on RedHat Linux
# chkconfig: 2345 64 36
# description: A very fast and reliable SQL database engine.

# Comments to support LSB init script conventions
### BEGIN INIT INFO
# Provides: mysql
# Required-Start: $local_fs $network $remote_fs
# Should-Start: ypbind nscd ldap ntpd xntpd
# Required-Stop: $local_fs $network $remote_fs
# Default-Start:  2 3 4 5
# Default-Stop: 0 1 6
# Short-Description: start and stop MySQL
# Description: MySQL is a very fast and reliable SQL database engine.
### END INIT INFO
 
# If you install MySQL on some other places than /usr, then you
# have to do one of the following things for this script to work:
#
# - Run this script from within the MySQL installation directory
# - Create a /etc/my.cnf file with the following information:
#   [mysqld]
#   basedir=<path-to-mysql-installation-directory>
# - Add the above to any other configuration file (for example ~/.my.ini)
#   and copy my_print_defaults to /usr/bin
# - Add the path to the mysql-installation-directory to the basedir variable
#   below.
#
# If you want to affect other MySQL variables, you should make your changes
# in the /etc/my.cnf, ~/.my.cnf or other MySQL configuration files.

# If you change base dir, you must also change datadir. These may get
# overwritten by settings in the MySQL configuration files.

basedir=/usr
datadir=/usr/mysql/data

# Default value, in seconds, afterwhich the script should timeout waiting
# for server start. 
# Value here is overriden by value in my.cnf. 
# 0 means don't wait at all
# Negative numbers mean to wait indefinitely
service_startup_timeout=900

# Lock directory for RedHat / SuSE.
lockdir='/var/lock/subsys'
lock_file_path="$lockdir/mysql"

# The following variables are only set for letting mysql.server find things.

# Set some defaults
mysqld_pid_file_path=
if test -z "$basedir"
then
  basedir=/usr
  bindir=/usr/bin
  if test -z "$datadir"
  then
    datadir=/var/lib/mysql
  fi
  sbindir=/usr/sbin
  libexecdir=/usr/sbin
else
  bindir="$basedir/bin"
  if test -z "$datadir"
  then
    datadir="$basedir/data"
  fi
  sbindir="$basedir/sbin"
  libexecdir="$basedir/libexec"
fi

# datadir_set is used to determine if datadir was set (and so should be
# *not* set inside of the --basedir= handler.)
datadir_set=

#
# Use LSB init script functions for printing messages, if possible
#
lsb_functions="/lib/lsb/init-functions"
if test -f $lsb_functions ; then
  . $lsb_functions
else
  log_success_msg()
  {
    echo " SUCCESS! $@"
  }
  log_failure_msg()
  {
    echo " ERROR! $@"
  }
fi

PATH="/sbin:/usr/sbin:/bin:/usr/bin:$basedir/bin"
export PATH

mode=$1    # start or stop

[ $# -ge 1 ] && shift


other_args="$*"   # uncommon, but needed when called from an RPM upgrade action
           # Expected: "--skip-networking --skip-grant-tables"
           # They are not checked here, intentionally, as it is the resposibility
           # of the "spec" file author to give correct arguments only.

case `echo "testing\c"`,`echo -n testing` in
    *c*,-n*) echo_n=   echo_c=     ;;
    *c*,*)   echo_n=-n echo_c=     ;;
    *)       echo_n=   echo_c='\c' ;;
esac

parse_server_arguments() {
  for arg do
    case "$arg" in
      --basedir=*)  basedir=`echo "$arg" | sed -e 's/^[^=]*=//'`
                    bindir="$basedir/bin"
		    if test -z "$datadir_set"; then
		      datadir="$basedir/data"
		    fi
		    sbindir="$basedir/sbin"
		    libexecdir="$basedir/libexec"
        ;;
      --datadir=*)  datadir=`echo "$arg" | sed -e 's/^[^=]*=//'`
		    datadir_set=1
	;;
      --pid-file=*) mysqld_pid_file_path=`echo "$arg" | sed -e 's/^[^=]*=//'` ;;
      --service-startup-timeout=*) service_startup_timeout=`echo "$arg" | sed -e 's/^[^=]*=//'` ;;
    esac
  done
}

wait_for_pid () {
  verb="$1"           # created | removed
  pid="$2"            # process ID of the program operating on the pid-file
  pid_file_path="$3" # path to the PID file.

  i=0
  avoid_race_condition="by checking again"

  while test $i -ne $service_startup_timeout ; do

    case "$verb" in
      'created')
        # wait for a PID-file to pop into existence.
        test -s "$pid_file_path" && i='' && break
        ;;
      'removed')
        # wait for this PID-file to disappear
        test ! -s "$pid_file_path" && i='' && break
        ;;
      *)
        echo "wait_for_pid () usage: wait_for_pid created|removed pid pid_file_path"
        exit 1
        ;;
    esac

    # if server isn't running, then pid-file will never be updated
    if test -n "$pid"; then
      if kill -0 "$pid" 2>/dev/null; then
        :  # the server still runs
      else
        # The server may have exited between the last pid-file check and now.  
        if test -n "$avoid_race_condition"; then
          avoid_race_condition=""
          continue  # Check again.
        fi

        # there's nothing that will affect the file.
        log_failure_msg "The server quit without updating PID file ($pid_file_path)."
        return 1  # not waiting any more.
      fi
    fi

    echo $echo_n ".$echo_c"
    i=`expr $i + 1`
    sleep 1

  done

  if test -z "$i" ; then
    log_success_msg
    return 0
  else
    log_failure_msg
    return 1
  fi
}

# Get arguments from the my.cnf file,
# the only group, which is read from now on is [mysqld]
if test -x ./bin/my_print_defaults
then
  print_defaults="./bin/my_print_defaults"
elif test -x $bindir/my_print_defaults
then
  print_defaults="$bindir/my_print_defaults"
elif test -x $bindir/mysql_print_defaults
then
  print_defaults="$bindir/mysql_print_defaults"
else
  # Try to find basedir in /etc/my.cnf
  conf=/etc/my.cnf
  print_defaults=
  if test -r $conf
  then
    subpat='^[^=]*basedir[^=]*=\(.*\)$'
    dirs=`sed -e "/$subpat/!d" -e 's//\1/' $conf`
    for d in $dirs
    do
      d=`echo $d | sed -e 's/[ 	]//g'`
      if test -x "$d/bin/my_print_defaults"
      then
        print_defaults="$d/bin/my_print_defaults"
        break
      fi
      if test -x "$d/bin/mysql_print_defaults"
      then
        print_defaults="$d/bin/mysql_print_defaults"
        break
      fi
    done
  fi

  # Hope it's in the PATH ... but I doubt it
  test -z "$print_defaults" && print_defaults="my_print_defaults"
fi

#
# Read defaults file from 'basedir'.   If there is no defaults file there
# check if it's in the old (depricated) place (datadir) and read it from there
#

extra_args=""
if test -r "$basedir/my.cnf"
then
  extra_args="-e $basedir/my.cnf"
else
  if test -r "$datadir/my.cnf"
  then
    extra_args="-e $datadir/my.cnf"
  fi
fi

parse_server_arguments `$print_defaults $extra_args mysqld server mysql_server mysql.server`

#
# Set pid file if not given
#
if test -z "$mysqld_pid_file_path"
then
  mysqld_pid_file_path=$datadir/`hostname`.pid
else
  case "$mysqld_pid_file_path" in
    /* ) ;;
    * )  mysqld_pid_file_path="$datadir/$mysqld_pid_file_path" ;;
  esac
fi

case "$mode" in
  'start')
    # Start daemon

    # Safeguard (relative paths, core dumps..)
    cd $basedir

    echo $echo_n "Starting MySQL"
    if test -x $bindir/mysqld_safe
    then
      # Give extra arguments to mysqld with the my.cnf file. This script
      # may be overwritten at next upgrade.
      $bindir/mysqld_safe --datadir="$datadir" --pid-file="$mysqld_pid_file_path" $other_args >/dev/null &
      wait_for_pid created "$!" "$mysqld_pid_file_path"; return_value=$?

      # Make lock for RedHat / SuSE
      if test -w "$lockdir"
      then
        touch "$lock_file_path"
      fi

      exit $return_value
    else
      log_failure_msg "Couldn't find MySQL server ($bindir/mysqld_safe)"
    fi
    ;;

  'stop')
    # Stop daemon. We use a signal here to avoid having to know the
    # root password.

    if test -s "$mysqld_pid_file_path"
    then
      mysqld_pid=`cat "$mysqld_pid_file_path"`

      if (kill -0 $mysqld_pid 2>/dev/null)
      then
        echo $echo_n "Shutting down MySQL"
        kill $mysqld_pid
        # mysqld should remove the pid file when it exits, so wait for it.
        wait_for_pid removed "$mysqld_pid" "$mysqld_pid_file_path"; return_value=$?
      else
        log_failure_msg "MySQL server process #$mysqld_pid is not running!"
        rm "$mysqld_pid_file_path"
      fi

      # Delete lock for RedHat / SuSE
      if test -f "$lock_file_path"
      then
        rm -f "$lock_file_path"
      fi
      exit $return_value
    else
      log_failure_msg "MySQL server PID file could not be found!"
    fi
    ;;

  'restart')
    # Stop the service and regardless of whether it was
    # running or not, start it again.
    if $0 stop  $other_args; then
      $0 start $other_args
    else
      log_failure_msg "Failed to stop running server, so refusing to try to start."
      exit 1
    fi
    ;;

  'reload'|'force-reload')
    if test -s "$mysqld_pid_file_path" ; then
      read mysqld_pid <  "$mysqld_pid_file_path"
      kill -HUP $mysqld_pid && log_success_msg "Reloading service MySQL"
      touch "$mysqld_pid_file_path"
    else
      log_failure_msg "MySQL PID file could not be found!"
      exit 1
    fi
    ;;
  'status')
    # First, check to see if pid file exists
    if test -s "$mysqld_pid_file_path" ; then 
      read mysqld_pid < "$mysqld_pid_file_path"
      if kill -0 $mysqld_pid 2>/dev/null ; then 
        log_success_msg "MySQL running ($mysqld_pid)"
        exit 0
      else
        log_failure_msg "MySQL is not running, but PID file exists"
        exit 1
      fi
    else
      # Try to find appropriate mysqld process
      mysqld_pid=`pidof $libexecdir/mysqld`

      # test if multiple pids exist
      pid_count=`echo $mysqld_pid | wc -w`
      if test $pid_count -gt 1 ; then
        log_failure_msg "Multiple MySQL running but PID file could not be found ($mysqld_pid)"
        exit 5
      elif test -z $mysqld_pid ; then 
        if test -f "$lock_file_path" ; then 
          log_failure_msg "MySQL is not running, but lock file ($lock_file_path) exists"
          exit 2
        fi 
        log_failure_msg "MySQL is not running"
        exit 3
      else
        log_failure_msg "MySQL is running but PID file could not be found"
        exit 4
      fi
    fi
    ;;
    *)
      # usage
      basename=`basename "$0"`
      echo "Usage: $basename  {start|stop|restart|reload|force-reload|status}  [ MySQL server options ]"
      exit 1
    ;;
esac

exit 0

```

然后执行mysql_install_db 就会根据my.cnf安装db
```
/usr/bin/mysql_install_db
```

如果是成功的话看到类似下面的信息

```
To start mysqld at boot time you have to copy
support-files/mysql.server to the right place for your system

PLEASE REMEMBER TO SET A PASSWORD FOR THE MySQL root USER !
To do so, start the server, then issue the following commands:

  /usr/bin/mysqladmin -u root password 'new-password'
  /usr/bin/mysqladmin -u root -h localhost.localdomain password 'new-password'

Alternatively you can run:

  /usr/bin/mysql_secure_installation

which will also give you the option of removing the test
databases and anonymous user created by default.  This is
strongly recommended for production servers.

See the manual for more instructions.

You can start the MySQL daemon with:

  cd /usr ; /usr/bin/mysqld_safe &

You can test the MySQL daemon with mysql-test-run.pl

  cd mysql-test ; perl mysql-test-run.pl

Please report any problems at http://bugs.mysql.com/

The latest information about MySQL is available on the web at

  http://www.mysql.com

Support MySQL by buying support/licenses at http://shop.mysql.com

WARNING: Found existing config file /usr/my.cnf on the system.
Because this file might be in use, it was not replaced,
but was used in bootstrap (unless you used --defaults-file)
and when you later start the server.
The new default config file was created as /usr/my-new.cnf,
please compare it with your file and take the changes you need.

WARNING: Default config file /etc/my.cnf exists on the system
This file will be read by default by the MySQL server
If you do not want to use this, either remove it, or use the
--defaults-file argument to mysqld_safe when starting the server
```


1. 启动mysql服务
```
/etc/init.d/mysql start
```
2. 设置开机启动
```
chkconfig --add mysql 
chkconfig mysql on
```
3. 打开远程访问权限
```
mysql -uroot -p
mysql> GRANT ALL PRIVILEGES ON *.* TO 'root'@'%' IDENTIFIED BY '123456' WITH GRANT OPTION;
mysql> flush privileges;
```

# 安装redis
```
yum install jemalloc-3.6.0-1.el7.x86_64.rpm
rpm -ivh redis-2.8.19-2.el7.x86_64.rpm

systemctl start redis.service
systemctl enable redis.service
```
开发和维护时可以开启redis的远程连接
vim /etc/redis.conf
注释掉 bind 127.0.0.1
如果是redis3.2之后，redis增加了protected-mode 即使注释掉bind 127.0.0.1，再访问redis时还是会报错

# mysql 开启远程
```sql
GRANT ALL PRIVILEGES ON *.* TO 'myuser'@'%'IDENTIFIED BY 'mypassword' WITH GRANT OPTION; 
```
如果是固定ip就这么写
```sql
grant all privileges on *.* to 'root'@'192.168.0.49'identified by '123' with grant option;
```
推送设置到内存或重启服务器也行
```sql
FLUSH PRIVILEGES;
```

# Mysql处理海量数据时的一些优化查询速度方法

由于在参与的实际项目中发现当mysql表的数据量达到百万级时，普通SQL查询效率呈直线下降，而且如果where中的查询条件较多时，其查询速度简直无法容忍。曾经测试对一个包含400多万条记录（有索引）的表执行一条条件查询，其查询时间竟然高达40几秒，相信这么高的查询延时，任何用户都会抓狂。因此如何提高sql语句查询效率，显得十分重要。以下是网上流传比较广泛的30种SQL查询语句优化方法：

	1、应尽量避免在 where 子句中使用!=或<>操作符，否则将引擎放弃使用索引而进行全表扫描。
	
	2、对查询进行优化，应尽量避免全表扫描，首先应考虑在 where 及 order by 涉及的列上建立索引。
	
	3、应尽量避免在 where 子句中对字段进行 null 值判断，否则将导致引擎放弃使用索引而进行全表扫描，如：
	select id from t where num is null
	可以在num上设置默认值0，确保表中num列没有null值，然后这样查询：
	select id from t where num=0
	
	4、尽量避免在 where 子句中使用 or 来连接条件，否则将导致引擎放弃使用索引而进行全表扫描，如：
	select id from t where num=10 or num=20
	可以这样查询：
	select id from t where num=10
	union all
	select id from t where num=20
	
	5、下面的查询也将导致全表扫描：(不能前置百分号)
	select id from t where name like '%abc%'
	若要提高效率，可以考虑全文检索。
	
	6、in 和 not in 也要慎用，否则会导致全表扫描，如：
	select id from t where num in(1,2,3)
	对于连续的数值，能用 between 就不要用 in 了：
	select id from t where num in(1,2,3)
	
	7、如果在 where 子句中使用参数，也会导致全表扫描。因为SQL只有在运行时才会解析局部变量，但优化程序不能将访问计划的选择推迟到运行时；它必须在编译时进行选择。然 而，如果在编译时建立访问计划，变量的值还是未知的，因而无法作为索引选择的输入项。如下面语句将进行全表扫描：
	select id from t where num=@num
	可以改为强制查询使用索引：
	select id from t with(index(索引名)) where num=@num
	
	8、应尽量避免在 where 子句中对字段进行表达式操作，这将导致引擎放弃使用索引而进行全表扫描。如：
	select id from t where num/2=100
	应改为:
	select id from t where num=100*2
	
	9、应尽量避免在where子句中对字段进行函数操作，这将导致引擎放弃使用索引而进行全表扫描。如：
	select id from t where substring(name,1,3)=’abc’–name以abc开头的id
	select id from t where datediff(day,createdate,’2005-11-30′)=0–’2005-11-30′生成的id
	应改为:
	select id from t where name like ‘abc%’
	select id from t where createdate>=’2005-11-30′ and createdate<’2005-12-1′
	
	10、不要在 where 子句中的“=”左边进行函数、算术运算或其他表达式运算，否则系统将可能无法正确使用索引。
	
	11、在使用索引字段作为条件时，如果该索引是复合索引，那么必须使用到该索引中的第一个字段作为条件时才能保证系统使用该索引，否则该索引将不会被使用，并且应尽可能的让字段顺序与索引顺序相一致。
	
	12、不要写一些没有意义的查询，如需要生成一个空表结构：
	select col1,col2 into #t from t where 1=0
	这类代码不会返回任何结果集，但是会消耗系统资源的，应改成这样：
	create table #t(…)
	
	13、很多时候用 exists 代替 in 是一个好的选择：
	select num from a where num in(select num from b)
	用下面的语句替换：
	select num from a where exists(select 1 from b where num=a.num)
	
	14、并不是所有索引对查询都有效，SQL是根据表中数据来进行查询优化的，当索引列有大量数据重复时，SQL查询可能不会去利用索引，如一表中有字段 sex，male、female几乎各一半，那么即使在sex上建了索引也对查询效率起不了作用。
	
	15、索引并不是越多越好，索引固然可以提高相应的 select 的效率，但同时也降低了 insert 及 update 的效率，因为 insert 或 update 时有可能会重建索引，所以怎样建索引需要慎重考虑，视具体情况而定。一个表的索引数最好不要超过6个，若太多则应考虑一些不常使用到的列上建的索引是否有 必要。
	
	16.应尽可能的避免更新 clustered 索引数据列，因为 clustered 索引数据列的顺序就是表记录的物理存储顺序，一旦该列值改变将导致整个表记录的顺序的调整，会耗费相当大的资源。若应用系统需要频繁更新 clustered 索引数据列，那么需要考虑是否应将该索引建为 clustered 索引。
	
	17、尽量使用数字型字段，若只含数值信息的字段尽量不要设计为字符型，这会降低查询和连接的性能，并会增加存储开销。这是因为引擎在处理查询和连接时会 逐个比较字符串中每一个字符，而对于数字型而言只需要比较一次就够了。
	
	18、尽可能的使用 varchar/nvarchar 代替 char/nchar ，因为首先变长字段存储空间小，可以节省存储空间，其次对于查询来说，在一个相对较小的字段内搜索效率显然要高些。
	
	19、任何地方都不要使用 select * from t ，用具体的字段列表代替“*”，不要返回用不到的任何字段。
	
	20、尽量使用表变量来代替临时表。如果表变量包含大量数据，请注意索引非常有限（只有主键索引）。
	
	21、避免频繁创建和删除临时表，以减少系统表资源的消耗。
	
	22、临时表并不是不可使用，适当地使用它们可以使某些例程更有效，例如，当需要重复引用大型表或常用表中的某个数据集时。但是，对于一次性事件，最好使 用导出表。
	
	23、在新建临时表时，如果一次性插入数据量很大，那么可以使用 select into 代替 create table，避免造成大量 log ，以提高速度；如果数据量不大，为了缓和系统表的资源，应先create table，然后insert。
	
	24、如果使用到了临时表，在存储过程的最后务必将所有的临时表显式删除，先 truncate table ，然后 drop table ，这样可以避免系统表的较长时间锁定。
	
	25、尽量避免使用游标，因为游标的效率较差，如果游标操作的数据超过1万行，那么就应该考虑改写。
	
	26、使用基于游标的方法或临时表方法之前，应先寻找基于集的解决方案来解决问题，基于集的方法通常更有效。
	
	27、与临时表一样，游标并不是不可使用。对小型数据集使用 FAST_FORWARD 游标通常要优于其他逐行处理方法，尤其是在必须引用几个表才能获得所需的数据时。在结果集中包括“合计”的例程通常要比使用游标执行的速度快。如果开发时 间允许，基于游标的方法和基于集的方法都可以尝试一下，看哪一种方法的效果更好。
	
	28、在所有的存储过程和触发器的开始处设置 SET NOCOUNT ON ，在结束时设置 SET NOCOUNT OFF 。无需在执行存储过程和触发器的每个语句后向客户端发送 DONE_IN_PROC 消息。
	
	29、尽量避免向客户端返回大数据量，若数据量过大，应该考虑相应需求是否合理。
	
	30、尽量避免大事务操作，提高系统并发能力。


# 对数据库参数进行调优及测试结果
	调整 innodb_flush_log_at_trx_commit=0。重启。truncate test;
	（其实直接将存储引擎改成MyISAM，速度可直接到达9000条/秒~10000条/秒）
	平均速度：7363.8条/秒
	
	删除索引
	此表在建表之初加入索引，是为了提高其后的组函数查询效率，但是在如果只考虑插入数据的速度，索引显示是个累赘。试试把索引删掉会如何。此处删除testId上索引的同时也删除了主键。使用代码来模拟自增主键：
	平均速度：8150.6条/秒。每秒写入增加近1000

	多线程
	使用多个线程，模拟多连接。
	平均速度：15891条/秒。又有了进一倍的提升。


# mysql的优化方法及实现例子

## 1.添加索引
普通索引   添加INDEX
```
ALTER TABLE `table_name` ADD INDEX index_name ( `column` )
```
主键索引   添加PRIMARY KEY
```
ALTER TABLE `table_name` ADD PRIMARY KEY ( `column` )
```
唯一索引    添加UNIQUE
```
ALTER TABLE `table_name` ADD UNIQUE ( `column` )
```
全文索引    添加FULLTEXT
```
ALTER TABLE `table_name` ADD FULLTEXT ( `column`)
```
如何添加多列索引
```
ALTER TABLE `table_name` ADD INDEX index_name ( `column1`, `column2`, `column3` )
```
如大家所知道的，Mysql目前主要有以下几种索引类型：FULLTEXT，HASH，BTREE，RTREE。
那么，这几种索引有什么功能和性能上的不同呢？

### FULLTEXT

即为全文索引，目前只有MyISAM引擎支持。其可以在CREATE TABLE ，ALTER TABLE ，CREATE INDEX 使用，不过目前只有 CHAR、VARCHAR ，TEXT 列上可以创建全文索引。值得一提的是，在数据量较大时候，现将数据放入一个没有全局索引的表中，然后再用CREATE INDEX创建FULLTEXT索引，要比先为一张表建立FULLTEXT然后再将数据写入的速度快很多。

全文索引并不是和MyISAM一起诞生的，它的出现是为了解决WHERE name LIKE “%word%"这类针对文本的模糊查询效率较低的问题。在没有全文索引之前，这样一个查询语句是要进行遍历数据表操作的，可见，在数据量较大时是极其的耗时的，如果没有异步IO处理，进程将被挟持，很浪费时间，当然这里不对异步IO作进一步讲解，想了解的童鞋，自行谷哥。

全文索引的使用方法并不复杂：

创建ALTER TABLE table ADD INDEX `FULLINDEX` USING FULLTEXT(`cname1`[,cname2…]);

使用SELECT * FROM table WHERE MATCH(cname1[,cname2…]) AGAINST ('word' MODE );

其中， MODE为搜寻方式（IN BOOLEAN MODE ，IN NATURAL LANGUAGE MODE ，IN NATURAL LANGUAGE MODE WITH QUERY EXPANSION / WITH QUERY EXPANSION）。

关于这三种搜寻方式，愚安在这里也不多做交代，简单地说，就是，布尔模式，允许word里含一些特殊字符用于标记一些具体的要求，如+表示一定要有，-表示一定没有，*表示通用匹配符，是不是想起了正则，类似吧；自然语言模式，就是简单的单词匹配；含表达式的自然语言模式，就是先用自然语言模式处理，对返回的结果，再进行表达式匹配。

对搜索引擎稍微有点了解的同学，肯定知道分词这个概念，FULLTEXT索引也是按照分词原理建立索引的。西文中，大部分为字母文字，分词可以很方便的按照空格进行分割。但很明显，中文不能按照这种方式进行分词。那又怎么办呢？这个向大家介绍一个Mysql的中文分词插件Mysqlcft，有了它，就可以对中文进行分词，想了解的同学请移步Mysqlcft，当然还有其他的分词插件可以使用。

### HASH

Hash这个词，可以说，自打我们开始码的那一天起，就开始不停地见到和使用到了。其实，hash就是一种（key=>value）形式的键值对，如数学中的函数映射，允许多个key对应相同的value，但不允许一个key对应多个value。正是由于这个特性，hash很适合做索引，为某一列或几列建立hash索引，就会利用这一列或几列的值通过一定的算法计算出一个hash值，对应一行或几行数据（这里在概念上和函数映射有区别，不要混淆）。在java语言中，每个类都有自己的hashcode()方法，没有显示定义的都继承自object类，该方法使得每一个对象都是唯一的，在进行对象间equal比较，和序列化传输中起到了很重要的作用。hash的生成方法有很多种，足可以保证hash码的唯一性，例如在MongoDB中，每一个document都有系统为其生成的唯一的objectID（包含时间戳，主机散列值，进程PID，和自增ID）也是一种hash的表现。额，我好像扯远了-_-!

由于hash索引可以一次定位，不需要像树形索引那样逐层查找,因此具有极高的效率。那为什么还需要其他的树形索引呢？

在这里愚安就不自己总结了。引用下园子里其他大神的文章：来自 14的路 的MySQL的btree索引和hash索引的区别

（1）Hash 索引仅仅能满足"=","IN"和"<=>"查询，不能使用范围查询。 
由于 Hash 索引比较的是进行 Hash 运算之后的 Hash 值，所以它只能用于等值的过滤，不能用于基于范围的过滤，因为经过相应的 Hash 算法处理之后的 Hash 值的大小关系，并不能保证和Hash运算前完全一样。 
（2）Hash 索引无法被用来避免数据的排序操作。 
由于 Hash 索引中存放的是经过 Hash 计算之后的 Hash 值，而且Hash值的大小关系并不一定和 Hash 运算前的键值完全一样，所以数据库无法利用索引的数据来避免任何排序运算； 
（3）Hash 索引不能利用部分索引键查询。 
对于组合索引，Hash 索引在计算 Hash 值的时候是组合索引键合并后再一起计算 Hash 值，而不是单独计算 Hash 值，所以通过组合索引的前面一个或几个索引键进行查询的时候，Hash 索引也无法被利用。 
（4）Hash 索引在任何时候都不能避免表扫描。 
前面已经知道，Hash 索引是将索引键通过 Hash 运算之后，将 Hash运算结果的 Hash 值和所对应的行指针信息存放于一个 Hash 表中，由于不同索引键存在相同 Hash 值，所以即使取满足某个 Hash 键值的数据的记录条数，也无法从 Hash 索引中直接完成查询，还是要通过访问表中的实际数据进行相应的比较，并得到相应的结果。 
（5）Hash 索引遇到大量Hash值相等的情况后性能并不一定就会比B-Tree索引高。 
对于选择性比较低的索引键，如果创建 Hash 索引，那么将会存在大量记录指针信息存于同一个 Hash 值相关联。这样要定位某一条记录时就会非常麻烦，会浪费多次表数据的访问，而造成整体性能低下。


愚安我稍作补充，讲一下HASH索引的过程，顺便解释下上面的第4,5条：

当我们为某一列或某几列建立hash索引时（目前就只有MEMORY引擎显式地支持这种索引），会在硬盘上生成类似如下的文件：

hash值 	存储地址    
1db54bc745a1	77#45b5 
4bca452157d4	76#4556,77#45cc…
…

hash值即为通过特定算法由指定列数据计算出来，磁盘地址即为所在数据行存储在硬盘上的地址（也有可能是其他存储地址，其实MEMORY会将hash表导入内存）。

这样，当我们进行WHERE age = 18 时，会将18通过相同的算法计算出一个hash值==>在hash表中找到对应的储存地址==>根据存储地址取得数据。

所以，每次查询时都要遍历hash表，直到找到对应的hash值，如（4），数据量大了之后，hash表也会变得庞大起来，性能下降，遍历耗时增加，如（5）。

### BTREE

BTREE索引就是一种将索引值按一定的算法，存入一个树形的数据结构中，相信学过数据结构的童鞋都对当初学习二叉树这种数据结构的经历记忆犹新，反正愚安我当时为了软考可是被这玩意儿好好地折腾了一番，不过那次考试好像没怎么考这个。如二叉树一样，每次查询都是从树的入口root开始，依次遍历node，获取leaf。

BTREE在MyISAM里的形式和Innodb稍有不同

在 Innodb里，有两种形态：一是primary key形态，其leaf node里存放的是数据，而且不仅存放了索引键的数据，还存放了其他字段的数据。二是secondary index，其leaf node和普通的BTREE差不多，只是还存放了指向主键的信息.

而在MyISAM里，主键和其他的并没有太大区别。不过和Innodb不太一样的地方是在MyISAM里，leaf node里存放的不是主键的信息，而是指向数据文件里的对应数据行的信息.

### RTREE
RTREE在mysql很少使用，仅支持geometry数据类型，支持该类型的存储引擎只有MyISAM、BDb、InnoDb、NDb、Archive几种。

相对于BTREE，RTREE的优势在于范围查找.

各种索引的使用情况

（1）对于BTREE这种Mysql默认的索引类型，具有普遍的适用性

（2）由于FULLTEXT对中文支持不是很好，在没有插件的情况下，最好不要使用。其实，一些小的博客应用，只需要在数据采集时，为其建立关键字列表，通过关键字索引，也是一个不错的方法，至少愚安我是经常这么做的。

（3）对于一些搜索引擎级别的应用来说，FULLTEXT同样不是一个好的处理方法，Mysql的全文索引建立的文件还是比较大的，而且效率不是很高，即便是使用了中文分词插件，对中文分词支持也只是一般。真要碰到这种问题，Apache的Lucene或许是你的选择。

（4）正是因为hash表在处理较小数据量时具有无可比拟的素的优势，所以hash索引很适合做缓存（内存数据库）。如mysql数据库的内存版本Memsql，使用量很广泛的缓存工具Mencached，NoSql数据库redis等，都使用了hash索引这种形式。当然，不想学习这些东西的话Mysql的MEMORY引擎也是可以满足这种需求的。

（5）至于RTREE，至今还没有使用过，它具体怎么样，我就不知道了。有RTREE使用经历的同学，到时可以交流下！

# explain分析sql语句执行效率

Explain命令在解决数据库性能上是第一推荐使用命令，大部分的性能问题可以通过此命令来简单的解决，Explain可以用来查看SQL语句的执行效 果，可以帮助选择更好的索引和优化查询语句，写出更好的优化语句。
	
Explain语法：explain select … from … [where …]
	
```
mariadb> explain select name from tb_1 where age<22;
+----+-------------+-------+-------+---------------+-----------+---------+------+------+-----------------------+
| id | select_type | table | type  | possible_keys | key       | key_len | ref  | rows | Extra                 |
+----+-------------+-------+-------+---------------+-----------+---------+------+------+-----------------------+
|  1 | SIMPLE      | tb_1  | range | index_age     | index_age | 4       | NULL |    1 | Using index condition |
+----+-------------+-------+-------+---------------+-----------+---------+------+------+-----------------------+
1 row in set
```

结果值从好到坏依次是：

	system > const > eq_ref > ref > fulltext > ref_or_null > index_merge > unique_subquery > index_subquery > range > index > ALL
	一般来说，得保证查询至少达到range级别，最好能达到ref，否则就可能会出现性能问题
	
下面对各个属性进行了解：
	
	1、id：这是SELECT的查询序列号
	
	2、select_type：select_type就是select的类型，可以有以下几种：
	
	SIMPLE：简单SELECT(不使用UNION或子查询等)
	
	PRIMARY：最外面的SELECT
	
	UNION：UNION中的第二个或后面的SELECT语句
	
	DEPENDENT UNION：UNION中的第二个或后面的SELECT语句，取决于外面的查询
	
	UNION RESULT：UNION的结果。
	
	SUBQUERY：子查询中的第一个SELECT
	
	DEPENDENT SUBQUERY：子查询中的第一个SELECT，取决于外面的查询
	
	DERIVED：导出表的SELECT(FROM子句的子查询)
	
	3、table：显示这一行的数据是关于哪张表的
	
	4、type：这列最重要，显示了连接使用了哪种类别,有无使用索引，是使用Explain命令分析性能瓶颈的关键项之一。
	结果值从好到坏依次是：
	system > const > eq_ref > ref > fulltext > ref_or_null > index_merge > unique_subquery > index_subquery > range > index > ALL
	一般来说，得保证查询至少达到range级别，最好能达到ref，否则就可能会出现性能问题。
	
	5、possible_keys：列指出MySQL能使用哪个索引在该表中找到行
	
	6、key：显示MySQL实际决定使用的键（索引）。如果没有选择索引，键是NULL
	
	7、key_len：显示MySQL决定使用的键长度。如果键是NULL，则长度为NULL。使用的索引的长度。在不损失精确性的情况下，长度越短越好
	
	8、ref：显示使用哪个列或常数与key一起从表中选择行。
	
	9、rows：显示MySQL认为它执行查询时必须检查的行数。
	
	10、Extra：包含MySQL解决查询的详细信息，也是关键参考项之一。
	
	Distinct
	一旦MYSQL找到了与行相联合匹配的行，就不再搜索了
	
	Not exists
	MYSQL 优化了LEFT JOIN，一旦它找到了匹配LEFT JOIN标准的行，
	
	就不再搜索了
	
	Range checked for each
	
	Record（index map:#）
	没有找到理想的索引，因此对于从前面表中来的每一 个行组合，MYSQL检查使用哪个索引，并用它来从表中返回行。这是使用索引的最慢的连接之一
	
	Using filesort
	看 到这个的时候，查询就需要优化了。MYSQL需要进行额外的步骤来发现如何对返回的行排序。它根据连接类型以及存储排序键值和匹配条件的全部行的行指针来 排序全部行
	
	Using index
	列数据是从仅仅使用了索引中的信息而没有读取实际的行动的表返回的，这发生在对表 的全部的请求列都是同一个索引的部分的时候
	
	Using temporary
	看到这个的时候，查询需要优化了。这 里，MYSQL需要创建一个临时表来存储结果，这通常发生在对不同的列集进行ORDER BY上，而不是GROUP BY上
	
	Using where
	使用了WHERE从句来限制哪些行将与下一张表匹配或者是返回给用户。如果不想返回表中的全部行，并且连接类型ALL或index， 这就会发生，或者是查询有问题
	
	其他一些Tip：
	
	当type 显示为 “index” 时，并且Extra显示为“Using Index”， 表明使用了覆盖索引。	
	
# 常用的sql优化

在数据库日常维护中，最常做的事情就是SQL语句优化，因为这个才是影响性能的最主要因素。当然还有其他方面的，比如OS优化，硬件优化，MySQL Server优化，数据类型优化，应用层优化，但是这些都没有SQL语句优化来的重要。下面将介绍INSERT,GROUP BY,LIMIT等的优化方法。

## 1.优化大批量插入数据

当用load命令导入数据的时候，适当的设置可以提高导入的速度。对于MyISAM存储引擎的表，可以通过如下方式快速导入大量的数据。
```
ALTER TABLE tablename DISABLE KEYS；
loading the data；
ALTER TABLE tablename ENABLE KEYS；
```
DISABLE KEYS和ENABLE KEYS用来关闭或者打开MyISAM表非唯一索引的更新。在导入大量的数据到一个非空的MyISAM表示，通过设置这两个命令，可以提高导入的效率。对于导入大量数据到一个空的MyISAM表时，默认就是先导入数据然后才创建索引的，所以不用设置。

对于InnoDB存储引擎表，上面的方式并不能提高导入数据的效率。可以有以下几种方式提高Innodb表的导入效率

>(1)因为InnoDB类型的表是按照主键的顺序保存的，所以将导入的数据按照主键的顺序排列，可以有效的提高导入数据的效率。

>(2)在导入数据前执行 SET UNIQUE_CHECKS=0，关闭唯一性效验，在导入数据结束以后执行SET UNIQUE_CHECKS=1，恢复唯一性效验，可以提高导入效率。

>(3)如果使用自动提交的方式，建议在导入前执行SET AUTOCOMMIT=0，关闭自动提交，导入结束后再执行SET AUTOCOMMIT=1，打开自动提交，也可以提高导入的效率。

>(4)对于有外键约束的表，我们在导入数据之前也可以忽略外键检查，因为innodb的外键是即时检查的，所以对导入的每一行都会进行外键的检查。
set foreign_key_checks = 0;
load data ............
set foreign_key_checks = 1;

## 2.优化INSERT语句

(1)如果同时从同一客户端插入大量数据，应该尽量使用多个值的表的INSERT 语句，这种方式将大大减少客户端与数据库服务器之间的连接，关闭等消耗，使得效率比分开执行的单个INSERT语句快(大部分情况下，使用多个值表的INSERT语句能比单个INSERT语句快上好几倍），比如下面一次插入多行：
```
INSERT INTO VALUES ('yayun',23),('tom',26),('atlas',32),('david',25).......
```
(2)插入延迟。如果从不同客户端插入很多行，可以通过使用INSERT  DELAYED语句得到更高的速度。DELAYED的意思是让INSERT语句马上执行，其实数据都被放在内存的队列中，并没有真正写入磁盘，这比每条语句分别插入要快的多；LOW_PRIORITY刚好相反，在所有其他用户对表的读写完成后才进行插入。

(3)将索引文件和数据文件放在不同的磁盘（利用建表中的选项）

(4)如果进行批量插入，可以通过增加bulk_insert_buffer_size 变量值的方法来提高速度，这只对MyISAM表有用。

(5)当从一个文本文件装载一个表时，使用LOAD DATA INFILE。通常比使用很多的INSERT语句快。

 无法使用索引的情况

(1).以%开头的like查询
(2).数据类型出现隐式转换的时候也不会使用索引，特别是当列类型是字符串，那么一定记得在where条件中把字符串常量值用引号引起来，否则即便这个列上有索引，MySQL也不会用到，因为MySQL默认把输入的常量值进行转换以后才进行检索
(3).复合索引的情况下，如果查询条件不包含索引列的最左边部分，即不满足最左前缀原则，则不会使用索引
(4).如果mysql估计使用索引扫描比全表扫描更慢，则不使用索引。(扫描数据超过30%，都会走全表)
(5).用or分割开的条件，如果 or前的条件中的列有索引，而后面的列中没有索引，那么涉及的索引都不会被用到
(6).字段使用函数，将无法使用索引
(7).Join 语句中 Join 条件字段类型不一致的时候 MySQL 无法使用索引

## 3.优化ORDER BY语句

通过索引排序是性能最好的，通常如果SQL语句不合理，就无法使用索引排序，以下几种情况是无法使用索引排序的。

(1).查询使用了两种不同的排序方向，但是索引列都是正序排序的;
(2).查询的where和order by中的列无法组合成索引的最左前缀;
(3).查询在索引列的第一列上是范围条件;
(4)查询条件上有多个等于条件。对排序来说，这也是一种范围查询
 在优化ORDER BY语句之前，先来看看MySQL中排序的方式。先看看MySQL官方提供的示例数据库sakila中customer表上的索引情况。
 
```
mysql> show index from customer;
+----------+------------+-------------------+--------------+-------------+-----------+-------------+----------+--------+------+------------+---------+---------------+
| Table    | Non_unique | Key_name          | Seq_in_index | Column_name | Collation | Cardinality | Sub_part | Packed | Null | Index_type | Comment | Index_comment |
+----------+------------+-------------------+--------------+-------------+-----------+-------------+----------+--------+------+------------+---------+---------------+
| customer |          0 | PRIMARY           |            1 | customer_id | A         |         577 |     NULL | NULL   |      | BTREE      |         |               |
| customer |          1 | idx_fk_store_id   |            1 | store_id    | A         |           3 |     NULL | NULL   |      | BTREE      |         |               |
| customer |          1 | idx_fk_address_id |            1 | address_id  | A         |         577 |     NULL | NULL   |      | BTREE      |         |               |
| customer |          1 | idx_last_name     |            1 | last_name   | A         |         577 |     NULL | NULL   |      | BTREE      |         |               |
+----------+------------+-------------------+--------------+-------------+-----------+-------------+----------+--------+------+------------+---------+---------------+
4 rows in set (0.00 sec)

mysql> 
```

###1.MySQL中有两种排序方式

第一种通过有序索引顺序扫描直接返回有序数据，这种方式在使用explain分析查询时显示为Using Index，不需要额外的排序，性能是最优的。

```
mysql> explain select customer_id from customer order by store_id ;
+----+-------------+----------+-------+---------------+-----------------+---------+------+------+-------------+
| id | select_type | table    | type  | possible_keys | key             | key_len | ref  | rows | Extra       |
+----+-------------+----------+-------+---------------+-----------------+---------+------+------+-------------+
|  1 | SIMPLE      | customer | index | NULL          | idx_fk_store_id | 1       | NULL |  577 | Using index |
+----+-------------+----------+-------+---------------+-----------------+---------+------+------+-------------+
1 row in set (0.00 sec)

mysql> 
```
因为查询主键，然后store_id列是辅助索引(二级索引)，辅助索引上存放了索引键值+对应行的主键，所以直接扫描辅助索引返回有序数据。

```
mysql> explain select * from customer order by customer_id;
+----+-------------+----------+-------+---------------+---------+---------+------+------+-------+
| id | select_type | table    | type  | possible_keys | key     | key_len | ref  | rows | Extra |
+----+-------------+----------+-------+---------------+---------+---------+------+------+-------+
|  1 | SIMPLE      | customer | index | NULL          | PRIMARY | 2       | NULL |  577 |       |
+----+-------------+----------+-------+---------------+---------+---------+------+------+-------+
1 row in set (0.00 sec)

mysql> 
```
这种排序方式直接使用了主键，也可以说成是使用了聚集索引。因为innodb是索引组织表（index-organized table），通过主键聚集数据，数据都是按照主键排序存放。而聚集索引就是按照没张表的主键构造一颗B+树，同时叶子节点中存放的即为正张表的行记录数据，也将聚集索引的叶子节点称为数据页。聚集索引的这个特性决定了索引组织表中的数据也是索引的一部分。

第二种是通过对返回数据进行排序，也就是通常说的Filesort排序，所有不是通过索引直接返回排序结果的排序都叫Filesort排序。Filesort并不代表通过磁盘文件进行排序，而只是说明进行了一个排序操作，至于排序操作是否使用了磁盘文件或者临时表，取决于mysql服务器对排序参数的设置和需要排序数据的大小。

```
mysql> explain select * from customer order by store_id ;                                      
+----+-------------+----------+------+---------------+------+---------+------+------+----------------+
| id | select_type | table    | type | possible_keys | key  | key_len | ref  | rows | Extra          |
+----+-------------+----------+------+---------------+------+---------+------+------+----------------+
|  1 | SIMPLE      | customer | ALL  | NULL          | NULL | NULL    | NULL |  671 | Using filesort |
+----+-------------+----------+------+---------------+------+---------+------+------+----------------+
1 row in set (0.00 sec)

mysql> 
```
那么这里优化器为什么不使用store_id列上的辅助索引进行排序呢？

当通过辅助索引来查找数据时，innodb存储引擎会遍历辅助索引并通过叶级别的指针获得指向主键索引的主键，然后通过主键索引来找到一个完整的行记录。举例来说，如果在一棵高度为3的辅助索引树中查找数据，那么需要对这个辅助索引树遍历3次找到指定主键，如果聚集索引树的高度同样为3，那么还需要对聚集索引树进行3次查找，最终找到一个完整的行数所在的页，因此一共需要6次逻辑IO访问以得到最终的一个数据页。

使用mysql5.6 的trace功能来查看一下强制使用辅助索引和全表扫描的开销。（mysql5.6的trace真心不错，给个赞^_^）

```
 {
   "rows_estimation": [
     {
       "table": "`customer` FORCE INDEX (`idx_fk_store_id`)",
       "table_scan": {
         "rows": 599,
         "cost": 5
       } /* table_scan */
     }
   ] /* rows_estimation */
 },
 {
   "considered_execution_plans": [
     {
       "plan_prefix": [
       ] /* plan_prefix */,
       "table": "`customer` FORCE INDEX (`idx_fk_store_id`)",
       "best_access_path": {
         "considered_access_paths": [
           {
             "access_type": "scan",
             "rows": 599,
             "cost": 719.8,
          "chosen": true,
             "use_tmp_table": true
           }
```
可以清楚的看见优化器使用全表扫描开销更小。

再来看一种情况
```
mysql> alter table customer add key idx_stored_email ( store_id , email );
Query OK, 0 rows affected (0.19 sec)
Records: 0  Duplicates: 0  Warnings: 0

mysql> 
mysql> explain select store_id , email from customer order  by email ;            
+----+-------------+----------+-------+---------------+------------------+---------+------+------+-----------------------------+
| id | select_type | table    | type  | possible_keys | key              | key_len | ref  | rows | Extra                       |
+----+-------------+----------+-------+---------------+------------------+---------+------+------+-----------------------------+
|  1 | SIMPLE      | customer | index | NULL          | idx_stored_email | 154     | NULL |  671 | Using index; Using filesort |
+----+-------------+----------+-------+---------------+------------------+---------+------+------+-----------------------------+
1 row in set (0.00 sec)

mysql> 
```
这里为什么又是filesort呢？不是使用了using index吗？虽然使用了覆盖索引(只访问索引的查询，即查询只需要访问索引，而无须访问数据行，最简单的理解，比如翻开一本书，从目录页查找某些内容，但是目录就写的比较详细，我们在目录就找到了自己想看的内容)。但是请别忘记了，idx_stored_email是复合索引，必须遵循最左前缀的原则。

我们改成如下SQL，就可以看见效果了：

```
mysql> explain select store_id , email from customer order  by store_id ;              
+----+-------------+----------+-------+---------------+------------------+---------+------+------+-------------+
| id | select_type | table    | type  | possible_keys | key              | key_len | ref  | rows | Extra       |
+----+-------------+----------+-------+---------------+------------------+---------+------+------+-------------+
|  1 | SIMPLE      | customer | index | NULL          | idx_stored_email | 154     | NULL |  671 | Using index |
+----+-------------+----------+-------+---------------+------------------+---------+------+------+-------------+
1 row in set (0.00 sec)

mysql> 
```
Filesort是通过相应的排序算法，将取得的数据在sort_buffer_size系统变量设置的内存排序区中进行排序，如果内存装载不下，它就会将磁盘上的数据进行分块，再对各个数据块进行排序，然后将各个块合并成有序的结果集。sort_buffer_size设置的排序区是每个线程独占的，所以在同一个时刻，mysql中存在多个sort buffer排序区。该值不要设置的太大，避免耗尽服务器内存。

简单来说，尽量减少额外排序，通过索引直接返回有序数据。where条件和order by使用相同的索引，并且order by的顺序和索引顺序相同。并且order by的字段都是升序或者降序。否则肯定需要额外的排序操作，这样就会出现Filesort。

```
mysql> explain select store_id , email , customer_id from customer where store_id =1 order by email desc;
+----+-------------+----------+------+----------------------------------+------------------+---------+-------+------+--------------------------+
| id | select_type | table    | type | possible_keys                    | key              | key_len | ref   | rows | Extra                    |
+----+-------------+----------+------+----------------------------------+------------------+---------+-------+------+--------------------------+
|  1 | SIMPLE      | customer | ref  | idx_fk_store_id,idx_stored_email | idx_stored_email | 1       | const |  325 | Using where; Using index |
+----+-------------+----------+------+----------------------------------+------------------+---------+-------+------+--------------------------+
1 row in set (0.00 sec)

mysql> 
```
针对Filesort优化MySQL Server

通过创建合适的索引当然能够减少Filesort的出现。但是在某些特殊情况下，条件限制不能让Filesort消失，那就需要想办法加快Filesort的操作。对于Filesort，mysql有两种排序算法。

1.两次扫描算法(Two Passes)

首先 根据条件取出排序字段和行指针信息，之后在排序区sort buffer中排序。如果sort buffer不够，则在临时表Temporary Table中存储排序结果。完成排序后根据行指针回表读取记录。该算法是MySQL 4.1之前采用的算法，需要两次访问数据，第一次获取排序字段和行指针信息，第二次根据行指针记录获取记录，尤其是第二次读取操作可能导致大量随机I/O操作，优点是排序的时候内存开销比较小。

2.一次扫描算法(Single passes)

一次性取出满足条件的行的所有字段，然后在排序区sort buffer中排序后直接输出结果集。排序的时候内存开销比较大，但是排序效率比两次扫描算法高。

MySQL通过比较系统变量max_length_for_sort_data的大小和Query语句取出的字段总大小来判断使用哪种排序算法。如果max_length_for_sort_data更大，那么使用第二种排序算法，否则使用第一种。

适当加大系统变量max_length_for_sort_data的值，能够让MySQL选择更优化的排序算法，即第二种算法。当然设置max_length_for_sort_data 过大，会造成CPU利用率过低和磁盘I/O过高，CPU和I/O利用平衡就足够了。

适当加大sort_buffer_size排序区，尽量让排序在内存中完成，而不是通过创建临时表放在文件中进行，当然也不能无限制加大sort_buffer_size排序区，因为sort_buffer_szie参数是每个线程独占，设置过大，会导致服务器SWAP严重。

尽量只使用必要的字段，SELECT具体的字段名称，而不是SELECT * 选择所有字段，这样可以减少排序区的使用。提高SQL性能。

## 4.优化GROUP BY 语句

默认情况下，mysql对所有GROUP BY col1，col2，的字段进行排序。这与在查询中指定ORDER BY col1，col2类似。因此，如果显式包括一个 包含相同列的ORDER BY子句，则对mysql的实际性能没有什么影响。如果查询包括GROUP BY,但我们想要避免排序带来的性能损耗，则可以指定ORDER BY NULL禁止排序，示例如下：

```
mysql> explain select payment_date , sum(amount) from payment group by payment_date;                  
+----+-------------+---------+------+---------------+------+---------+------+-------+---------------------------------+
| id | select_type | table   | type | possible_keys | key  | key_len | ref  | rows  | Extra                           |
+----+-------------+---------+------+---------------+------+---------+------+-------+---------------------------------+
|  1 | SIMPLE      | payment | ALL  | NULL          | NULL | NULL    | NULL | 15422 | Using temporary; Using filesort |
+----+-------------+---------+------+---------------+------+---------+------+-------+---------------------------------+
1 row in set (0.00 sec)

mysql> 
```
可以看见使用了Filesort，还使用了内存临时表，这条SQL严重影响性能，所以需要优化：

首先禁止排序，ORDER BY NULL

```
mysql> explain select payment_date , sum(amount) from payment group by payment_date order by null;    
+----+-------------+---------+------+---------------+------+---------+------+-------+-----------------+
| id | select_type | table   | type | possible_keys | key  | key_len | ref  | rows  | Extra           |
+----+-------------+---------+------+---------------+------+---------+------+-------+-----------------+
|  1 | SIMPLE      | payment | ALL  | NULL          | NULL | NULL    | NULL | 15422 | Using temporary |
+----+-------------+---------+------+---------------+------+---------+------+-------+-----------------+
1 row in set (0.00 sec)

mysql> 
```
可以看见已经没有使用Filesort，但是还是使用了内存临时表，这是我们可以创建一个复合索引来优化性能

```
mysql> alter table payment add key idx_pal (payment_date,amount,last_update);                                     
Query OK, 0 rows affected (0.13 sec)
Records: 0  Duplicates: 0  Warnings: 0

mysql> explain select payment_date , sum(amount) from payment group by payment_date;              
+----+-------------+---------+-------+---------------+---------+---------+------+-------+-------------+
| id | select_type | table   | type  | possible_keys | key     | key_len | ref  | rows  | Extra       |
+----+-------------+---------+-------+---------------+---------+---------+------+-------+-------------+
|  1 | SIMPLE      | payment | index | NULL          | idx_pal | 15      | NULL | 15422 | Using index |
+----+-------------+---------+-------+---------------+---------+---------+------+-------+-------------+
1 row in set (0.01 sec)

mysql> 
```
## 5.优化子查询

MySQL 4.1开始支持SQL的子查询。这个技术可以使用SELECT语句来创建一个单列的查询结果，然后把这个结果作为过滤条件用在另外一个SELECT语句中。使用子查询可以一次性完成很多逻辑上需要多个步骤才能完成的SQL操作，同时也可以避免事务或者表锁死，并且写起来也非常easy，but，在有些情况下，子查询效率非常低下，我们可以使用比较高大上的写法，那就是连接（JOIN）取而代之.^_^,下面是一个列子：

```
mysql> explain select * from customer where customer_id not in ( select customer_id from payment);
+----+--------------------+----------+----------------+--------------------+--------------------+---------+------+------+-------------+
| id | select_type        | table    | type           | possible_keys      | key                | key_len | ref  | rows | Extra       |
+----+--------------------+----------+----------------+--------------------+--------------------+---------+------+------+-------------+
|  1 | PRIMARY            | customer | ALL            | NULL               | NULL               | NULL    | NULL |  671 | Using where |
|  2 | DEPENDENT SUBQUERY | payment  | index_subquery | idx_fk_customer_id | idx_fk_customer_id | 2       | func |   12 | Using index |
+----+--------------------+----------+----------------+--------------------+--------------------+---------+------+------+-------------+
2 rows in set (0.00 sec)

mysql> 
```
我解释一下这里的执行计划：

第二行，id为2，说明优先级最高，最先执行，DEPENDENT SUBQUERY子查询中的第一个SELECT(意味着select依赖于外层查询中的数据)，type为index_subquery,与unique_subquery类似，区别在于in的后面是查询非唯一索引字段的子查询,using index使用了覆盖索引。

第一行，id为1，说明优先级最低，可以看见select_type列是PRIMARY，意思是最外层的SELECT查询，可以看见使用了全表扫描。

如果使用连接(join)来完成这个查询，速度将会快很多。尤其是连接条件有索引的情况下：

```
mysql> explain select * from customer left join payment on customer.customer_id = payment.customer_id where  payment.customer_id is null;
+----+-------------+----------+------+--------------------+--------------------+---------+-----------------------------+------+-------------------------+
| id | select_type | table    | type | possible_keys      | key                | key_len | ref                         | rows | Extra                   |
+----+-------------+----------+------+--------------------+--------------------+---------+-----------------------------+------+-------------------------+
|  1 | SIMPLE      | customer | ALL  | NULL               | NULL               | NULL    | NULL                        |  671 |                         |
|  1 | SIMPLE      | payment  | ref  | idx_fk_customer_id | idx_fk_customer_id | 2       | sakila.customer.customer_id |   12 | Using where; Not exists |
+----+-------------+----------+------+--------------------+--------------------+---------+-----------------------------+------+-------------------------+
2 rows in set (0.00 sec)

mysql> 
```
从执行计划看出查询关联类型从index_subquery调整为了ref，在mysql5.5(包含mysql5.5)，子查询效率还是不如关联查询(join),连接之所以更有效率，是因为MySQL不需要在内存中创建临时表来完成这个逻辑上需要两个步骤的查询工作。

## 6.优化OR条件

对于含有OR的查询语句，则无法使用单列索引，但是可以使用复合索引

```
mysql> explain select * from film where language_id=1 or title ='ACADEMY DINOSAUR';
+----+-------------+-------+------+------------------------------+------+---------+------+------+-------------+
| id | select_type | table | type | possible_keys                | key  | key_len | ref  | rows | Extra       |
+----+-------------+-------+------+------------------------------+------+---------+------+------+-------------+
|  1 | SIMPLE      | film  | ALL  | idx_title,idx_fk_language_id | NULL | NULL    | NULL | 1133 | Using where |
+----+-------------+-------+------+------------------------------+------+---------+------+------+-------------+
1 row in set (0.00 sec)

mysql> 
```
下面看一个较简单明了的例子：

```
mysql> show create table tt\G
*************************** 1. row ***************************
       Table: tt
Create Table: CREATE TABLE `tt` (
  `id` int(11) DEFAULT NULL,
  `age` int(11) DEFAULT NULL,
  KEY `id` (`id`),
  KEY `age` (`age`)
) ENGINE=InnoDB DEFAULT CHARSET=utf8
1 row in set (0.00 sec)

mysql> select * from tt;
+------+------+
| id   | age  |
+------+------+
|    1 |   23 |
|    2 |   36 |
+------+------+
2 rows in set (0.00 sec)

mysql> 
```
可以看见表tt有两个单列索引，我们使用如下SQL查询，看是否会使用索引

```
mysql> explain select * from tt where id=1 or age=36;
+----+-------------+-------+------+---------------+------+---------+------+------+-------------+
| id | select_type | table | type | possible_keys | key  | key_len | ref  | rows | Extra       |
+----+-------------+-------+------+---------------+------+---------+------+------+-------------+
|  1 | SIMPLE      | tt    | ALL  | id,age        | NULL | NULL    | NULL |    2 | Using where |
+----+-------------+-------+------+---------------+------+---------+------+------+-------------+
1 row in set (0.01 sec)

mysql> 
```
可以看见虽然显示有id，age索引可用，但是没有使用，即全表扫描。我们可以这样优化：

```
mysql> explain select * from tt where id=1 union all select *  from tt where age=36;
+----+--------------+------------+------+---------------+------+---------+-------+------+-------------+
| id | select_type  | table      | type | possible_keys | key  | key_len | ref   | rows | Extra       |
+----+--------------+------------+------+---------------+------+---------+-------+------+-------------+
|  1 | PRIMARY      | tt         | ref  | id            | id   | 5       | const |    1 | Using where |
|  2 | UNION        | tt         | ref  | age           | age  | 5       | const |    1 | Using where |
| NULL | UNION RESULT | <union1,2> | ALL  | NULL          | NULL | NULL    | NULL  | NULL |             |
+----+--------------+------------+------+---------------+------+---------+-------+------+-------------+
3 rows in set (0.00 sec)

mysql> 
```
可以看见已经使用了索引，至于这里的执行计划，我就不再说明。有机会我会写一篇mysql执行计划的文章。

看看使用复合索引查询的情况：

```
mysql> show create table tt\G
*************************** 1. row ***************************
       Table: tt
Create Table: CREATE TABLE `tt` (
  `id` int(11) DEFAULT NULL,
  `age` int(11) DEFAULT NULL,
  KEY `idx_id_age` (`id`,`age`)
) ENGINE=InnoDB DEFAULT CHARSET=utf8
1 row in set (0.00 sec)

mysql> explain select * from tt where id=1 or age=36;
+----+-------------+-------+-------+---------------+------------+---------+------+------+--------------------------+
| id | select_type | table | type  | possible_keys | key        | key_len | ref  | rows | Extra                    |
+----+-------------+-------+-------+---------------+------------+---------+------+------+--------------------------+
|  1 | SIMPLE      | tt    | index | idx_id_age    | idx_id_age | 10      | NULL |    2 | Using where; Using index |
+----+-------------+-------+-------+---------------+------------+---------+------+------+--------------------------+
1 row in set (0.00 sec)

mysql> 
```
## 7.优化分页查询(LIMIT)

一般分页查询时，通过创建覆盖索引能够比较好的提高性能。比较常见的一个分页查询是limit 1000,20，这种最蛋碎了，此时mysql排序出前1020条记录后仅仅返回第1001到1020条记录，前1000条记录都会被抛弃，查询和排序的代价非常高。

在索引上完成排序分页操作，最后根据主键关联回表查询所需要的其他列内容。（使用到了自连接）例如下面的SQL语句：

```
mysql> explain select film_id , description from film order by title limit 50,5;
+----+-------------+-------+------+---------------+------+---------+------+------+----------------+
| id | select_type | table | type | possible_keys | key  | key_len | ref  | rows | Extra          |
+----+-------------+-------+------+---------------+------+---------+------+------+----------------+
|  1 | SIMPLE      | film  | ALL  | NULL          | NULL | NULL    | NULL | 1133 | Using filesort |
+----+-------------+-------+------+---------------+------+---------+------+------+----------------+
1 row in set (0.00 sec)

mysql> 
```
可以看见实际上使用了全表扫描，如果表有上百万记录，那么这将是一条致命SQL

我们改写成按照索引分页后回表读取行的方式，从执行计划中看不到全表扫描了

```
mysql> explain select a.film_id , a.description from film a inner join (select film_id from film order by title limit 50,5) b on a.film_id=b.film_id; 
+----+-------------+------------+--------+---------------+-----------+---------+-----------+------+-------------+
| id | select_type | table      | type   | possible_keys | key       | key_len | ref       | rows | Extra       |
+----+-------------+------------+--------+---------------+-----------+---------+-----------+------+-------------+
|  1 | PRIMARY     | <derived2> | ALL    | NULL          | NULL      | NULL    | NULL      |    5 |             |
|  1 | PRIMARY     | a          | eq_ref | PRIMARY       | PRIMARY   | 2       | b.film_id |    1 |             |
|  2 | DERIVED     | film       | index  | NULL          | idx_title | 767     | NULL      |   55 | Using index |
+----+-------------+------------+--------+---------------+-----------+---------+-----------+------+-------------+
3 rows in set (0.00 sec)

mysql> 
```
这里我大概解释一下执行计划：

第三行：

id为2，优先级最高，最先执行
select_type为DERIVED 用来表示包含在from子句中的子查询的select，mysql会递归执行并将结果放到一个临时表中。服务器内部称为“派生表”，因为该临时表是从子查询中派生出来的。
type列为index表示索引树全扫描，mysql遍历整个索引来查询匹配的行，这里就把film_id查询出来了。
Extra列为using index 表示使用覆盖索引

第二行：
select_type为PRIMARY，即复杂查询的最外层，当然这里还不算是最最外层。 
table列为a，即film表的别名a，
type列为eq_ref，类似ref，区别就在使用的索引是唯一索引，对于每个索引键值，表中只有一条记录匹配，简单来说，就是多表连接中使用primary key或者 unique key作为关联条件

第一行：
select_type列的primary表示该查询为外层查询
table列被标记为<derived2>，表示查询结果来自一个衍生表，其中2代表该查询衍生自第2个select查询，即id为2的select

## 7.其他优化手段

当然还有其他的优化手段，比如索引提示，我这里简单列举一下就行了，因为大部分的时候mysql 优化器都工作的很好。

USE INDEX

提供给优化器参考的索引列表(优化器不一定给你面子哦)

IGNORE INDEX

提示优化器忽略一个或者多个索引

FORCE INDEX

强制优化器使用某个特定索引

总结一下：

其实SQL语句优化的过程中，无非就是对mysql的执行计划理解，以及B+树索引的理解，其实只要我们理解执行计划和B+树以后，优化SQL语句还是比较简单的，当然还有特别复杂的SQL，我这里只是一些简单例子，当然再复杂的SQL，还是逃脱不了原理性的东西。呵呵。^_^
