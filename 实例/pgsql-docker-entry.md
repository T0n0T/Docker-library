> 在docker中，pgsql等数据库在启动容器时就会创建实例，一些配置项需要重新启动或者在启动数据库前就进行，则需要修改或者在docker-compose中指定新的进入点脚本

在pgsql的进入脚本`docker-entrtypoint.sh`中，主要需要注意的是主函数：
```shell
_main() {
	# 第一个参数 $1 是否具有前缀 - ，有则将其替换为postgres
	if [ "${1:0:1}" = '-' ]; then
		set -- postgres "$@"
	fi

	if [ "$1" = 'postgres' ] && ! _pg_want_help "$@"; then
		docker_setup_env
		# 作为root设置权限
		docker_create_db_directories
		if [ "$(id -u)" = '0' ]; then
			# 从这里开始作为 postgres 帐号登入
			exec su-exec postgres "$BASH_SOURCE" "$@"
		fi

		# 以下初始化只能在未初始化的空文件夹中进行
		if [ -z "$DATABASE_ALREADY_EXISTS" ]; then
			docker_verify_minimum_env

			# 用于检查 /docker-entrypoint-initdb.d/ 中文件的权限是否正确
			ls /docker-entrypoint-initdb.d/ > /dev/null

			docker_init_database_dir
			pg_setup_hba_conf "$@"

			# PGPASSWORD is required for psql when authentication is required for 'local' connections via pg_hba.conf and is otherwise harmless
			# e.g. when '--auth=md5' or '--auth-local=md5' is used in POSTGRES_INITDB_ARGS
			export PGPASSWORD="${PGPASSWORD:-$POSTGRES_PASSWORD}"
			docker_temp_server_start "$@"

			docker_setup_db
			docker_process_init_files /docker-entrypoint-initdb.d/*

			docker_temp_server_stop
			unset PGPASSWORD

			cat <<-'EOM'

				PostgreSQL init process complete; ready for start up.

			EOM
		else
			cat <<-'EOM'

				PostgreSQL Database directory appears to contain a database; Skipping initialization

			EOM
		fi
	fi

	exec "$@"
}
```

可以看出，整个脚本的运行逻辑：
1. `docker_setup_env` 先以一开始登入docker的root<font color="#c300ff">设置文件权限</font>
2. `docker_create_db_directories` 以root创建`$PGDATA`，`$WALDIR`(如果设置了)
3. `docker_verify_minimum_env` 检查`$POSTGRES_PASSWORD`和`$POSTGRES_HOST_AUTH_METHOD`是否至少设置了一项，保证数据库的可登入
4. 检查`/docker-entrypoint-initdb.d/`中的文件权限情况
5. `docker_init_database_dir` 会调用pgsql的`initdb`脚本，并根据设定的参数设置一些内容，包括了`--waldir`，`--username`，以及其他`$POSTGRES_INITDB_ARGS`
6. `pg_setup_hba_conf` 设置连接认真方式
7. `docker_temp_server_start` 开启数据库
8. `docker_setup_db` 选取或者创建docker-compose中预设的数据库
9. `docker_process_init_files` 使用`/docker-entrypoint-initdb.d/*`中的脚本进行补充设置
10. `docker_temp_server_stop` 停止数据库工作，等待正式使用
> 在官方的Dockerfile中，这个进入点脚本传入了参数为`postgres`

为了在docker中修改pgsql的配置文件位置，需要修改默认的进入点文件，或者在`/docker-entrypoint-initdb.d/*`中增加修改配置的脚本，主要任务如下：
- 停止数据库服务
- 删除按默认配置启动
- 指定启动时的进入点文件路径
- 根据配置文件对新的需要操作文件夹赋权给postgres
- 开启数据库
- 在docker-compose中指定启动时配置文件的路径
pg_ctl -D $PGDATA stop -mf

查询配置是否需要重启

```shell
# true or false
psql -c "select name, setting, pending_restart from pg_settings
```


附`initdb`参数表：
```shell
$ /usr/pgsql-10/bin/initdb --help
initdb initializes a PostgreSQL database cluster.

Usage:
  initdb [OPTION]... [DATADIR]

Options:
  -A, --auth=METHOD         default authentication method for local connections
      --auth-host=METHOD    default authentication method for local TCP/IP connections
      --auth-local=METHOD   default authentication method for local-socket connections
 [-D, --pgdata=]DATADIR     location for this database cluster
  -E, --encoding=ENCODING   set default encoding for new databases
      --locale=LOCALE       set default locale for new databases
      --lc-collate=, --lc-ctype=, --lc-messages=LOCALE
      --lc-monetary=, --lc-numeric=, --lc-time=LOCALE
                            set default locale in the respective category for
                            new databases (default taken from environment)
      --no-locale           equivalent to --locale=C
      --pwfile=FILE         read password for the new superuser from file
  -T, --text-search-config=CFG
                            default text search configuration
  -U, --username=NAME       database superuser name
  -W, --pwprompt            prompt for a password for the new superuser
  -X, --waldir=WALDIR       location for the write-ahead log directory

Less commonly used options:
  -d, --debug               generate lots of debugging output
  -k, --data-checksums      use data page checksums
  -L DIRECTORY              where to find the input files
  -n, --no-clean            do not clean up after errors
  -N, --no-sync             do not wait for changes to be written safely to disk
  -s, --show                show internal settings
  -S, --sync-only           only sync data directory

Other options:
  -V, --version             output version information, then exit
  -?, --help                show this help, then exit

If the data directory is not specified, the environment variable PGDATA
is used.

Report bugs to <pgsql-bugs@postgresql.org>.
```
