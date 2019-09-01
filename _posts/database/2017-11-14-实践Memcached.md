---
layout: post
title: "实践Memcached"
date: 2017-11-14 14:05:00 +0800
categories: 数据库
tags: memcached nosql
---



## 命令参考

```shell
$ memcached -h
memcached 1.4.33
-p <num>      TCP port number to listen on (default: 11211)
-U <num>      UDP port number to listen on (default: 11211, 0 is off)
-s <file>     UNIX socket path to listen on (disables network support)
-A            enable ascii "shutdown" command
-a <mask>     access mask for UNIX socket, in octal (default: 0700)
-l <addr>     interface to listen on (default: INADDR_ANY, all addresses)
              <addr> may be specified as host:port. If you don't specify
              a port number, the value you specified with -p or -U is
              used. You may specify multiple addresses separated by comma
              or by using -l multiple times
-d            run as a daemon
-r            maximize core file limit
-u <username> assume identity of <username> (only when run as root)
-m <num>      max memory to use for items in megabytes (default: 64 MB)
-M            return error on memory exhausted (rather than removing items)
-c <num>      max simultaneous connections (default: 1024)
-k            lock down all paged memory.  Note that there is a
              limit on how much memory you may lock.  Trying to
              allocate more than that would fail, so be sure you
              set the limit correctly for the user you started
              the daemon with (not for -u <username> user;
              under sh this is done with 'ulimit -S -l NUM_KB').
-v            verbose (print errors/warnings while in event loop)
-vv           very verbose (also print client commands/reponses)
-vvv          extremely verbose (also print internal state transitions)
-h            print this help and exit
-i            print memcached and libevent license
-V            print version and exit
-P <file>     save PID in <file>, only used with -d option
-f <factor>   chunk size growth factor (default: 1.25)
-n <bytes>    minimum space allocated for key+value+flags (default: 48)
-L            Try to use large memory pages (if available). Increasing
              the memory page size could reduce the number of TLB misses
              and improve the performance. In order to get large pages
              from the OS, memcached will allocate the total item-cache
              in one large chunk.
-D <char>     Use <char> as the delimiter between key prefixes and IDs.
              This is used for per-prefix stats reporting. The default is
              ":" (colon). If this option is specified, stats collection
              is turned on automatically; if not, then it may be turned on
              by sending the "stats detail on" command to the server.
-t <num>      number of threads to use (default: 4)
-R            Maximum number of requests per event, limits the number of
              requests process for a given connection to prevent 
              starvation (default: 20)
-C            Disable use of CAS
-b <num>      Set the backlog queue limit (default: 1024)
-B            Binding protocol - one of ascii, binary, or auto (default)
-I            Override the size of each slab page. Adjusts max item size
              (default: 1mb, min: 1k, max: 128m)
-S            Turn on Sasl authentication
-F            Disable flush_all command
-o            Comma separated list of extended or experimental options
              - maxconns_fast: immediately close new
                connections if over maxconns limit
              - hashpower: An integer multiplier for how large the hash
                table should be. Can be grown at runtime if not big enough.
                Set this based on "STAT hash_power_level" before a 
                restart.
              - tail_repair_time: Time in seconds that indicates how long to wait before
                forcefully taking over the LRU tail item whose refcount has leaked.
                Disabled by default; dangerous option.
              - hash_algorithm: The hash table algorithm
                default is jenkins hash. options: jenkins, murmur3
              - lru_crawler: Enable LRU Crawler background thread
              - lru_crawler_sleep: Microseconds to sleep between items
                default is 100.
              - lru_crawler_tocrawl: Max items to crawl per slab per run
                default is 0 (unlimited)
              - lru_maintainer: Enable new LRU system + background thread
              - hot_lru_pct: Pct of slab memory to reserve for hot lru.
                (requires lru_maintainer)
              - warm_lru_pct: Pct of slab memory to reserve for warm lru.
                (requires lru_maintainer)
              - expirezero_does_not_evict: Items set to not expire, will not evict.
                (requires lru_maintainer)
              - idle_timeout: Timeout for idle connections
              - (EXPERIMENTAL) slab_chunk_max: Maximum slab size. Do not change without extreme care.
              - watcher_logbuf_size: Size in kilobytes of per-watcher write buffer.
              - worker_logbuf_Size: Size in kilobytes of per-worker-thread buffer
                read by background thread. Which is then written to watchers.
              - modern: Enables 'modern' defaults. See release notes (highly recommended!).
              - track_sizes: Enable dynamic reports for 'stats sizes' command.
```

样例

```shell
$ /usr/local/bin/memcached -P /app/software/memcached.pid -d -u ctsapp -m 128 -l 0.0.0.0 -p 11211 -vvv > /app/log/memcached.log 2>&1
```

## deepin

```shell
$ apt install memcached
$ sudo systemctl start memcached
```

## RedHat

安装memcached（使用root用户）

```shell
$ cd libevent-2.1.8-stable/ && ./configure --prefix=/usr && make && make install

$ cd memcached-1.5.3 && ./configure --with-libevent=/usr/lib && make && make install

$ ln -s /usr/lib/libevent-2.1.so.6 /usr/lib64/libevent-2.1.so.6 && ldd /usr/local/bin/memcached

$ cp memcached /etc/init.d/memcached

$ cp memcached.conf /app/config/

$ chmod +x /etc/init.d/memcached

$ chkconfig --add memcached

$ chkconfig memcached on

# 非root用户
$ service memcached start
$ service memcached stop
$ service memcached restart
$ service memcached status
```

### memcached脚本

备注：指定memcached.conf配置文件

```shell
#!/bin/bash  
# v.0.0.1  
# create by snowolf at 2012.5.25  
#  
# memcached  - This shell script takes care of starting and stopping memcached.  
#  
# chkconfig: - 90 10  
# description: Memcache provides fast memory based storage.  
# processname: memcached  
  
source /app/config/memcached.conf
  
# Source function library.  
#. /etc/rc.d/init.d/functions  
TEXTDOMAIN=initscripts

# Make sure umask is sane
umask 022

# Set up a default search path.
PATH="/sbin:/usr/sbin:/bin:/usr/bin"
export PATH

# Get a sane screen width
[ -z "${COLUMNS:-}" ] && COLUMNS=80

[ -z "${CONSOLETYPE:-}" ] && CONSOLETYPE="`/sbin/consoletype`"

if [ -f /etc/sysconfig/i18n -a -z "${NOLOCALE:-}" ] ; then
  . /etc/profile.d/lang.sh
fi

# Read in our configuration
if [ -z "${BOOTUP:-}" ]; then
  if [ -f /etc/sysconfig/init ]; then
      . /etc/sysconfig/init
  else
    # This all seem confusing? Look in /etc/sysconfig/init,
    # or in /usr/doc/initscripts-*/sysconfig.txt
    BOOTUP=color
    RES_COL=60
    MOVE_TO_COL="echo -en \\033[${RES_COL}G"
    SETCOLOR_SUCCESS="echo -en \\033[1;32m"
    SETCOLOR_FAILURE="echo -en \\033[1;31m"
    SETCOLOR_WARNING="echo -en \\033[1;33m"
    SETCOLOR_NORMAL="echo -en \\033[0;39m"
    LOGLEVEL=1
  fi
  if [ "$CONSOLETYPE" = "serial" ]; then
      BOOTUP=serial
      MOVE_TO_COL=
      SETCOLOR_SUCCESS=
      SETCOLOR_FAILURE=
      SETCOLOR_WARNING=
      SETCOLOR_NORMAL=
  fi
fi

if [ "${BOOTUP:-}" != "verbose" ]; then
   INITLOG_ARGS="-q"
else
   INITLOG_ARGS=
fi

# Interpret escape sequences in an fstab entry
fstab_decode_str() {
	fstab-decode echo "$1"
}

# Check if $pid (could be plural) are running
checkpid() {
	local i

	for i in $* ; do
		[ -d "/proc/$i" ] && return 0
	done
	return 1
}

__readlink() {
    ls -bl "$@" 2>/dev/null| awk '{ print $NF }'
}

# __umount_loop awk_program fstab_file first_msg retry_msg umount_args
# awk_program should process fstab_file and return a list of fstab-encoded
# paths; it doesn't have to handle comments in fstab_file.
__umount_loop() {
	local remaining sig=
	local retry=3

	remaining=$(LC_ALL=C awk "/^#/ {next} $1" "$2" | sort -r)
	while [ -n "$remaining" -a "$retry" -gt 0 ]; do
		if [ "$retry" -eq 3 ]; then
			action "$3" fstab-decode umount $5 $remaining
		else
			action "$4" fstab-decode umount $5 $remaining
		fi
		sleep 2
		remaining=$(LC_ALL=C awk "/^#/ {next} $1" "$2" | sort -r)
		[ -z "$remaining" ] && break
		fstab-decode /sbin/fuser -k -m $sig $remaining >/dev/null
		sleep 5
		retry=$(($retry -1))
		sig=-9
	done
}

# Similar to __umount loop above, specialized for loopback devices
__umount_loopback_loop() {
	local remaining devremaining sig=
	local retry=3

	remaining=$(awk '$1 ~ /^\/dev\/loop/ && $2 != "/" {print $2}' /proc/mounts)
	devremaining=$(awk '$1 ~ /^\/dev\/loop/ && $2 != "/" {print $1}' /proc/mounts)
	while [ -n "$remaining" -a "$retry" -gt 0 ]; do
		if [ "$retry" -eq 3 ]; then
			action $"Unmounting loopback filesystems: " \
				fstab-decode umount $remaining
		else
			action $"Unmounting loopback filesystems (retry):" \
				fstab-decode umount $remaining
		fi
		for dev in $devremaining ; do
			losetup $dev > /dev/null 2>&1 && \
				action $"Detaching loopback device $dev: " \
				losetup -d $dev
		done
		remaining=$(awk '$1 ~ /^\/dev\/loop/ && $2 != "/" {print $2}' /proc/mounts)
		devremaining=$(awk '$1 ~ /^\/dev\/loop/ && $2 != "/" {print $1}' /proc/mounts)
		[ -z "$remaining" ] && break
		fstab-decode /sbin/fuser -k -m $sig $remaining >/dev/null
		sleep 5
		retry=$(($retry -1))
		sig=-9
	done
}

# __proc_pids {program} [pidfile]
# Set $pid to pids from /var/run* for {program}.  $pid should be declared
# local in the caller.
# Returns LSB exit code for the 'status' action.
__pids_var_run() {
	local base=${1##*/}
	local pid_file=${2:-/var/run/$base.pid}

	pid=
	if [ -f "$pid_file" ] ; then
	        local line p
		read line < "$pid_file"
		for p in $line ; do
			[ -z "${p//[0-9]/}" -a -d "/proc/$p" ] && pid="$pid $p"
		done
	        if [ -n "$pid" ]; then
	                return 0
	        fi
		return 1 # "Program is dead and /var/run pid file exists"
	fi
	return 3 # "Program is not running"
}

# Output PIDs of matching processes, found using pidof
__pids_pidof() {
	pidof  -o $$ -o $PPID -o %PPID -x "$1" || \
		pidof  -o $$ -o $PPID -o %PPID -x "${1##*/}"
}


# A function to start a program.
daemon() {
	# Test syntax.
	local gotbase= force= nicelevel corelimit
	local pid base= user= nice= bg= pid_file=
	nicelevel=0
	while [ "$1" != "${1##[-+]}" ]; do
	  case $1 in
	    '')    echo $"$0: Usage: daemon [+/-nicelevel] {program}"
	           return 1;;
	    --check)
		   base=$2
		   gotbase="yes"
		   shift 2
		   ;;
	    --check=?*)
	    	   base=${1#--check=}
		   gotbase="yes"
		   shift
		   ;;
	    --user)
		   user=$2
		   shift 2
		   ;;
	    --user=?*)
	           user=${1#--user=}
		   shift
		   ;;
	    --pidfile)
		   pid_file=$2
		   shift 2
		   ;;
	    --pidfile=?*)
		   pid_file=${1#--pidfile=}
		   shift
		   ;;
	    --force)
	    	   force="force"
		   shift
		   ;;
	    [-+][0-9]*)
	    	   nice="nice -n $1"
	           shift
		   ;;
	    *)     echo $"$0: Usage: daemon [+/-nicelevel] {program}"
	           return 1;;
	  esac
	done

        # Save basename.
        [ -z "$gotbase" ] && base=${1##*/}

        # See if it's already running. Look *only* at the pid file.
	__pids_var_run "$base" "$pid_file"

	[ -n "$pid" -a -z "$force" ] && return

	# make sure it doesn't core dump anywhere unless requested
	corelimit="ulimit -S -c ${DAEMON_COREFILE_LIMIT:-0}"
	
	# if they set NICELEVEL in /etc/sysconfig/foo, honor it
	[ -n "${NICELEVEL:-}" ] && nice="nice -n $NICELEVEL"
	
	# Echo daemon
        [ "${BOOTUP:-}" = "verbose" -a -z "${LSB:-}" ] && echo -n " $base"

	# And start it up.
	if [ -z "$user" ]; then
	   $nice /bin/bash -c "$corelimit >/dev/null 2>&1 ; $*"
	else
	   $nice runuser -s /bin/bash - $user -c "$corelimit >/dev/null 2>&1 ; $*"
	fi
	[ "$?" -eq 0 ] && success $"$base startup" || failure $"$base startup"
}

# A function to stop a program.
killproc() {
	local RC killlevel= base pid pid_file= delay

	RC=0; delay=3
	# Test syntax.
	if [ "$#" -eq 0 ]; then
		echo $"Usage: killproc [-p pidfile] [ -d delay] {program} [-signal]"
		return 1
	fi
	if [ "$1" = "-p" ]; then
		pid_file=$2
		shift 2
	fi
	if [ "$1" = "-d" ]; then
		delay=$2
		shift 2
	fi
        

	# check for second arg to be kill level
	[ -n "${2:-}" ] && killlevel=$2

        # Save basename.
        base=${1##*/}

        # Find pid.
	__pids_var_run "$1" "$pid_file"
	if [ -z "$pid_file" -a -z "$pid" ]; then
		pid="$(__pids_pidof "$1")"
	fi

        # Kill it.
        if [ -n "$pid" ] ; then
                [ "$BOOTUP" = "verbose" -a -z "${LSB:-}" ] && echo -n "$base "
		if [ -z "$killlevel" ] ; then
		       if checkpid $pid 2>&1; then
			   # TERM first, then KILL if not dead
			   kill -TERM $pid >/dev/null 2>&1
			   usleep 100000
			   if checkpid $pid && sleep 1 &&
			      checkpid $pid && sleep $delay &&
			      checkpid $pid ; then
                                kill -KILL $pid >/dev/null 2>&1
				usleep 100000
			   fi
		        fi
			checkpid $pid
			RC=$?
			[ "$RC" -eq 0 ] && failure $"$base shutdown" || success $"$base shutdown"
			RC=$((! $RC))
		# use specified level only
		else
		        if checkpid $pid; then
	                	kill $killlevel $pid >/dev/null 2>&1
				RC=$?
				[ "$RC" -eq 0 ] && success $"$base $killlevel" || failure $"$base $killlevel"
			elif [ -n "${LSB:-}" ]; then
				RC=7 # Program is not running
			fi
		fi
	else
		if [ -n "${LSB:-}" -a -n "$killlevel" ]; then
			RC=7 # Program is not running
		else
			failure $"$base shutdown"
			RC=0
		fi
	fi

        # Remove pid file if any.
	if [ -z "$killlevel" ]; then
            rm -f "${pid_file:-/var/run/$base.pid}"
	fi
	return $RC
}

# A function to find the pid of a program. Looks *only* at the pidfile
pidfileofproc() {
	local pid

	# Test syntax.
	if [ "$#" = 0 ] ; then
		echo $"Usage: pidfileofproc {program}"
		return 1
	fi

	__pids_var_run "$1"
	[ -n "$pid" ] && echo $pid
	return 0
}

# A function to find the pid of a program.
pidofproc() {
	local RC pid pid_file=

	# Test syntax.
	if [ "$#" = 0 ]; then
		echo $"Usage: pidofproc [-p pidfile] {program}"
		return 1
	fi
	if [ "$1" = "-p" ]; then
		pid_file=$2
		shift 2
	fi
	fail_code=3 # "Program is not running"

	# First try "/var/run/*.pid" files
	__pids_var_run "$1" "$pid_file"
	RC=$?
	if [ -n "$pid" ]; then
		echo $pid
		return 0
	fi

	[ -n "$pid_file" ] && return $RC
	__pids_pidof "$1" || return $RC
}

status() {
	local base pid pid_file=

	# Test syntax.
	if [ "$#" = 0 ] ; then
		echo $"Usage: status [-p pidfile] {program}"
		return 1
	fi
	if [ "$1" = "-p" ]; then
		pid_file=$2
		shift 2
	fi
	base=${1##*/}

	# First try "pidof"
	__pids_var_run "$1" "$pid_file"
	RC=$?
	if [ -z "$pid_file" -a -z "$pid" ]; then
		pid="$(__pids_pidof "$1")"
	fi
	if [ -n "$pid" ]; then
	        echo $"${base} (pid $pid) is running..."
	        return 0
	fi

	case "$RC" in
		0)
			echo $"${base} (pid $pid) is running..."
			return 0
			;;
		1)
	                echo $"${base} dead but pid file exists"
	                return 1
			;;
	esac
	# See if /var/lock/subsys/${base} exists
	if [ -f /var/lock/subsys/${base} ]; then
		echo $"${base} dead but subsys locked"
		return 2
	fi
	echo $"${base} is stopped"
	return 3
}

echo_success() {
  [ "$BOOTUP" = "color" ] && $MOVE_TO_COL
  echo -n "["
  [ "$BOOTUP" = "color" ] && $SETCOLOR_SUCCESS
  echo -n $"  OK  "
  [ "$BOOTUP" = "color" ] && $SETCOLOR_NORMAL
  echo -n "]"
  echo -ne "\r"
  return 0
}

echo_failure() {
  [ "$BOOTUP" = "color" ] && $MOVE_TO_COL
  echo -n "["
  [ "$BOOTUP" = "color" ] && $SETCOLOR_FAILURE
  echo -n $"FAILED"
  [ "$BOOTUP" = "color" ] && $SETCOLOR_NORMAL
  echo -n "]"
  echo -ne "\r"
  return 1
}

echo_passed() {
  [ "$BOOTUP" = "color" ] && $MOVE_TO_COL
  echo -n "["
  [ "$BOOTUP" = "color" ] && $SETCOLOR_WARNING
  echo -n $"PASSED"
  [ "$BOOTUP" = "color" ] && $SETCOLOR_NORMAL
  echo -n "]"
  echo -ne "\r"
  return 1
}

echo_warning() {
  [ "$BOOTUP" = "color" ] && $MOVE_TO_COL
  echo -n "["
  [ "$BOOTUP" = "color" ] && $SETCOLOR_WARNING
  echo -n $"WARNING"
  [ "$BOOTUP" = "color" ] && $SETCOLOR_NORMAL
  echo -n "]"
  echo -ne "\r"
  return 1
}

# Inform the graphical boot of our current state
update_boot_stage() {
  if [ "$GRAPHICAL" = "yes" -a -x /usr/bin/rhgb-client ]; then
    /usr/bin/rhgb-client --update="$1"
  fi
  return 0
}

# Log that something succeeded
success() {
  #if [ -z "${IN_INITLOG:-}" ]; then
  #   initlog $INITLOG_ARGS -n $0 -s "$1" -e 1
  #fi
  [ "$BOOTUP" != "verbose" -a -z "${LSB:-}" ] && echo_success
  return 0
}

# Log that something failed
failure() {
  local rc=$?
  #if [ -z "${IN_INITLOG:-}" ]; then
  #   initlog $INITLOG_ARGS -n $0 -s "$1" -e 2
  #fi
  [ "$BOOTUP" != "verbose" -a -z "${LSB:-}" ] && echo_failure
  [ -x /usr/bin/rhgb-client ] && /usr/bin/rhgb-client --details=yes
  return $rc
}

# Log that something passed, but may have had errors. Useful for fsck
passed() {
  local rc=$?
  #if [ -z "${IN_INITLOG:-}" ]; then
  #   initlog $INITLOG_ARGS -n $0 -s "$1" -e 1
  #fi
  [ "$BOOTUP" != "verbose" -a -z "${LSB:-}" ] && echo_passed
  return $rc
}  

# Log a warning
warning() {
  local rc=$?
  #if [ -z "${IN_INITLOG:-}" ]; then
  #   initlog $INITLOG_ARGS -n $0 -s "$1" -e 1
  #fi
  [ "$BOOTUP" != "verbose" -a -z "${LSB:-}" ] && echo_warning
  return $rc
}  

# Run some action. Log its output.
action() {
  local STRING rc

  STRING=$1
  echo -n "$STRING "
  if [ "${RHGB_STARTED:-}" != "" -a -w /etc/rhgb/temp/rhgb-console ]; then
      echo -n "$STRING " > /etc/rhgb/temp/rhgb-console
  fi
  shift
  "$@" && success $"$STRING" || failure $"$STRING"
  rc=$?
  echo
  if [ "${RHGB_STARTED:-}" != "" -a -w /etc/rhgb/temp/rhgb-console ]; then
      if [ "$rc" = "0" ]; then
      	echo_success > /etc/rhgb/temp/rhgb-console
      else
        echo_failure > /etc/rhgb/temp/rhgb-console
	[ -x /usr/bin/rhgb-client ] && /usr/bin/rhgb-client --details=yes
      fi
      echo > /etc/rhgb/temp/rhgb-console
  fi
  return $rc
}

# returns OK if $1 contains $2
strstr() {
  [ "${1#*$2*}" = "$1" ] && return 1
  return 0
}

# Confirm whether we really want to run this service
confirm() {
  [ -x /usr/bin/rhgb-client ] && /usr/bin/rhgb-client --details=yes
  while : ; do 
      echo -n $"Start service $1 (Y)es/(N)o/(C)ontinue? [Y] "
      read answer
      if strstr $"yY" "$answer" || [ "$answer" = "" ] ; then
         return 0
      elif strstr $"cC" "$answer" ; then
	 rm -f /var/run/confirm
	 [ -x /usr/bin/rhgb-client ] && /usr/bin/rhgb-client --details=no
         return 2
      elif strstr $"nN" "$answer" ; then
         return 1
      fi
  done
}

# resolve a device node to its major:minor numbers in decimal or hex
get_numeric_dev() {
(
    fmt="%d:%d"
    if [ "$1" == "hex" ]; then
        fmt="%x:%x"
    fi
    ls -lH "$2" | awk '{ sub(/,/, "", $5); printf("'"$fmt"'", $5, $6); }'
) 2>/dev/null
}

# find the working name for a running dm device with the same table as one
# that dmraid would create
resolve_dm_name() {
(
    name="$1"

    line=$(/sbin/dmraid -ay -t --ignorelocking | \
        egrep -iv "no block devices found|No RAID disks" | \
        awk -F ':' "{ if (\$1 ~ /^$name$/) { print \$2; }}")
    for x in $line ; do
        if [[ "$x" =~ "^/dev/" ]] ; then
            majmin=$(get_numeric_dev dec $x)
            line=$(echo "$line" | sed -e "s,$x\( \|$\),$majmin\1,g")
        fi
    done
    line=$(echo "$line" | sed -e 's/^[ \t]*//' -e 's/[ \t]*$//' \
               -e 's/ core [12] [[:digit:]]\+ / core [12] [[:digit:]]\\+ /')
    /sbin/dmsetup table | \
        sed -n -e "s/.*\(no block devices found\|No devices found\).*//" \
               -e "s/\(^[^:]\+\): $line\( \+$\|$\)/\1/p"
) 2>/dev/null
}

# Check whether file $1 is a backup or rpm-generated file and should be ignored
is_ignored_file() {
    case "$1" in
	*~ | *.bak | *.orig | *.rpmnew | *.rpmorig | *.rpmsave)
	    return 0
	    ;;
    esac
    return 1
}
# A sed expression to filter out the files that is_ignored_file recognizes
__sed_discard_ignored_files='/\(~\|\.bak\|\.orig\|\.rpmnew\|\.rpmorig\|\.rpmsave\)$/d'
  
[ -x $memcached_path ] || exit 0  
  
RETVAL=0  
prog="memcached"  
  
# Start daemons.  
start() {  
    if [ -e $memcached_pid -a ! -z $memcached_pid ];then  
        echo $prog" already running...."  
        exit 1  
    fi  
  
    echo -n $"Starting $prog "  
    # Single instance for all caches  
	$memcached_path -P $memcached_pid $memcached_options >> $memcached_log 2>&1
    RETVAL=$?  
    [ $RETVAL -eq 0 ] && {  
        success $"$prog"  
    }  
    echo  
    return $RETVAL  
}  
  
  
# Stop daemons.  
stop() {  
    echo -n $"Stopping $prog "  
    killproc -d 10 $memcached_path  
    echo  
    [ $RETVAL = 0 ] && rm -f $memcached_pid
  
    RETVAL=$?  
    return $RETVAL  
}  
  
# See how we were called.  
case "$1" in  
        start)  
            start  
            ;;  
        stop)  
            stop  
            ;;  
        status)  
            status $prog  
            RETVAL=$?  
            ;;  
        restart)  
            stop  
            start  
            ;;  
        *)  
            echo $"Usage: $0 {start|stop|status|restart}"  
            exit 1  
esac  
exit $RETVAL
```

### memcached.conf配置文件

```
memcached_path="/usr/local/bin/memcached"  
memcached_pid="/app/software/memcached/memcached.pid"  
memcached_options="-d -u ctsapp -m 256 -l 0.0.0.0 -p 11211"
memcached_log=/app/log/memcached.log
export memcached_path memcached_pid memcached_options memcached_log
```