---
layout: post
tags: test
comments: true
---

그동안 mysql을 설치해서 사용할 일이 있으면 그냥 배포판에서 제공하는 저장소에서 내려받거나 윈도우를 쓸 때는 간편한 인스톨러를 사용하곤 했었다. 하지만 mysql을 좀 더 공부하려는 마음을 먹고 보니 처음부터 설치하고 설정하는 경험이 도움이 될 것 같다는 생각이 들었다. 그래서 소스 코드를 받아서 빌드하고 직접 설치해봤다.

## 소스 코드 내려받기, 빌드하기, 설치하기

github에 가 보면 mysql community 버전을 찾을 수 있다. [github mysql repository link](https://github.com/mysql/mysql-server)

```sh
$ hub clone mysql/mysql-server
$ cd mysql-server
$ git checkout 5.6
```

나는 5.6을 사용하기 위해서 `5.6` 으로 checkout 했다. 기본 브랜치는 현재 (2020-03-23) 기준으로 `8.0` 이다.

```sh
$ mkdir build_target && cd build_target
$ cmake \
  '-DCMAKE_INSTALL_PREFIX=/usr/local/mysql' \
  '-DINSTALL_SBINDIR=/usr/local/mysql/bin' \
  '-DINSTALL_BINDIR=/usr/local/mysql/bin' \
  '-DINSTALL_LAYOUT=STANDALONE' \
  '-DMYSQL_DATADIR=/usr/local/mysql/data' \
  '-DSYSCONFDIR=/usr/local/mysql/etc' \
  '-DINSTALL_SCRIPTDIR=/usr/local/mysql/bin' \
  '-DWITH_INNOBASE_STORAGE_ENGINE=1' \
  '-DWITH_ARCHIVE_STORAGE_ENGINE=1' \
  '-DWITH_BLACKHOLE_STORAGE_ENGINE=1' \
  '-DWITH_PERFSCHEMA_STORAGE_ENGINE=1' \
  '-DWITH_FEDERATED_STORAGE_ENGINE=1' \
  '-DWITH_PARTITION_STORAGE_ENGINE=1' \
  '-DENABLE_DEBUG_SYNC=0' \
  '-DENABLED_LOCAL_INFILE=1' \
  '-DENABLED_PROFILING=1' \
  '-DWITH_DEBUG=0' \
  '-DWITH_LIBWRAP=0' \
  '-DWITH_READLINE=1' \
  '-DWITH_SSL=0' \
  '-DCMAKE_BUILD_TYPE=RelWithDebInfo' \
  '-DCMAKE_C_FLAGS=-O2' \
  '-DCMAKE_CXX_FLAGS=-O2' \
  '-DWITH_SYSTEMD=1'
```

[mysql docs를 통해](https://dev.mysql.com/doc/refman/5.7/en/source-configuration-options.html) systemd를 사용하기 위해서는 `-DWITH_SYSTEMD` 플래그가 필요하다는 사실을 알아내고 (조금 뒤에 알고보니 저 문서는 5.7 버전용이었고, 5.6에서는 해당 사항이 없었다. 이런! [5.6 문서는 여기 있다](https://dev.mysql.com/doc/refman/5.6/en/source-configuration-options.html)) 해당 플래그끼지 붙인 다음, `make` 하고 `make install` 했다.

## 유저 그룹 및 유저 설정하기

쉘에서 필요한 권한 설정도 같이 해 준다.

```
# groupadd dba
# useradd -g dba mysql
# cd /usr/local
# mkdir data tmp logs
# chown -R root .
# chown -R mysql ./data
# chown -R mysql ./tmp
# chown -R mysql ./logs
# chgrp -R dba .
```

그런 다음 시스템 테이블을 준비한다.

```
# bin/mysql_install_db --user=mysql --basedir=/usr/local/mysql --datadir=/usr/local/mysql/data
```

## config 준비, 서비스 등록하기

`/etc/my.cnf` 를 적절한 내용으로 채워준 다음 systemd service description 파일을 준비한다.

```
[Unit]
Description=MySQL Community Server
After=network.target
After=syslog.target

[Install]
WantedBy=multi-user.target
Alias=mysql.service

[Service]
User=mysql
Group=dba

# Execute pre and post scripts as root
PermissionsStartOnly=true

# Start main service
ExecStart=/usr/local/mysql/bin/mysqld_safe

# Give up if ping don't get an answer
TimeoutSec=300

Restart=always
PrivateTmp=false
```

이 내용을 가진 파일을 `/etc/systemd/system/mysqld.service`에 저장하고 `sudo systemctl start mysqld.service` 하면 정상적으로 mysqld가 작동하는 것을 확인할 수 있다.

```
$ sudo systemctl status mysqld
● mysqld.service - MySQL Community Server
     Loaded: loaded (/etc/systemd/system/mysqld.service; disabled; vendor preset: disabled)
     Active: active (running) since Mon 2020-03-23 14:50:02 KST; 38min ago
   Main PID: 351224 (mysqld_safe)
      Tasks: 23 (limit: 19094)
     Memory: 892.8M
     CGroup: /system.slice/mysqld.service
             ├─351224 /bin/sh /usr/local/mysql/bin/mysqld_safe
             └─352350 /usr/local/mysql/bin/mysqld --basedir=/usr/local/mysql --datadir=/usr/local/mysql/data --pl>
```

## References

1. [MySQL Documentation](https://dev.mysql.com/doc/refman/5.6/en)
2. [Real MySQL (책)](https://wikibook.co.kr/real-mysql/)

{% include disqus_comments.html %}
