# Docker Compose SonarQube and PostgreSQL

## Docker 搭建代码质量检测中文平台 SonarQube 同时配套数据库选择 PostgreSQL

### 下载编排文件
```
# git clone https://github.com/AllenLiuGit/docker-compose-sonarqube-postgresql.git
```

### 使用

* 配置CentOS7内核参数
```
# vim /etc/sysctl.conf 
```
添加一行
```
vm.max_map_count=262144
```

重新加载参数 
```
# sysctl -p 
```

否则，如果不配置，则将会报错：
```
sonarqube_1  | [1]: max virtual memory areas vm.max_map_count [65530] is too low, increase to at least [262144]
```

* 构建并启动
```
# cd docker-compose-sonarqube-postgresql
# docker-compose build
# docker-compose up -d
```
或者
```
# cd docker-compose-sonarqube-postgresql
# docker-compose up -d --build
```

* 查看日志并验证
```
# docker-compose logs postgresql-db
```
期间可能会等一会儿再试，最终会看到如下日志输出：
```
postgresql-db_1  | waiting for server to shut down....2019-08-08 03:27:19.487 UTC [43] LOG:  received fast shutdown request
postgresql-db_1  | 2019-08-08 03:27:19.488 UTC [43] LOG:  aborting any active transactions
postgresql-db_1  | 2019-08-08 03:27:19.492 UTC [43] LOG:  background worker "logical replication launcher" (PID 50) exited with exit code 1
postgresql-db_1  | 2019-08-08 03:27:19.492 UTC [45] LOG:  shutting down
postgresql-db_1  | 2019-08-08 03:27:19.511 UTC [43] LOG:  database system is shut down
postgresql-db_1  |  done
postgresql-db_1  | server stopped
postgresql-db_1  | 
postgresql-db_1  | PostgreSQL init process complete; ready for start up.
postgresql-db_1  | 
postgresql-db_1  | 2019-08-08 03:27:19.601 UTC [1] LOG:  listening on IPv4 address "0.0.0.0", port 5432
postgresql-db_1  | 2019-08-08 03:27:19.601 UTC [1] LOG:  listening on IPv6 address "::", port 5432
postgresql-db_1  | 2019-08-08 03:27:19.606 UTC [1] LOG:  listening on Unix socket "/var/run/postgresql/.s.PGSQL.5432"
postgresql-db_1  | 2019-08-08 03:27:19.629 UTC [61] LOG:  database system was shut down at 2019-08-08 03:27:19 UTC
postgresql-db_1  | 2019-08-08 03:27:19.641 UTC [1] LOG:  database system is ready to accept connections
```
其中`database system is shut down`不用担心，这是数据库初始化进程在操作。

```
# docker-compose logs sonarqube
```
期间可能会等一会儿再试，最终会看到如下日志输出：
```
sonarqube_1      | 2019.08.08 03:28:10 INFO  ce[][o.s.p.ProcessEntryPoint] Starting ce
sonarqube_1      | 2019.08.08 03:28:10 INFO  ce[][o.s.ce.app.CeServer] Compute Engine starting up...
sonarqube_1      | 2019.08.08 03:28:11 INFO  ce[][o.e.p.PluginsService] no modules loaded
sonarqube_1      | 2019.08.08 03:28:11 INFO  ce[][o.e.p.PluginsService] loaded plugin [org.elasticsearch.join.ParentJoinPlugin]
sonarqube_1      | 2019.08.08 03:28:11 INFO  ce[][o.e.p.PluginsService] loaded plugin [org.elasticsearch.percolator.PercolatorPlugin]
sonarqube_1      | 2019.08.08 03:28:11 INFO  ce[][o.e.p.PluginsService] loaded plugin [org.elasticsearch.transport.Netty4Plugin]
sonarqube_1      | 2019.08.08 03:28:13 INFO  ce[][o.s.s.e.EsClientProvider] Connected to local Elasticsearch: [127.0.0.1:9001]
sonarqube_1      | 2019.08.08 03:28:13 INFO  ce[][o.sonar.db.Database] Create JDBC data source for jdbc:postgresql://postgresql-db:5432/sonar
sonarqube_1      | 2019.08.08 03:28:14 INFO  ce[][o.s.s.p.ServerFileSystemImpl] SonarQube home: /opt/sonarqube
sonarqube_1      | 2019.08.08 03:28:15 INFO  ce[][o.s.c.c.CePluginRepository] Load plugins
sonarqube_1      | 2019.08.08 03:28:15 INFO  ce[][o.s.c.p.PluginInfo] Plugin [l10nzh] defines 'l10nen' as base plugin. This metadata can be removed from manifest of l10n plugins since version 5.2.
sonarqube_1      | 2019.08.08 03:28:16 INFO  ce[][o.s.c.c.ComputeEngineContainerImpl] Running Community edition
sonarqube_1      | 2019.08.08 03:28:16 INFO  ce[][o.s.ce.app.CeServer] Compute Engine is operational
sonarqube_1      | 2019.08.08 03:28:16 INFO  app[][o.s.a.SchedulerImpl] Process[ce] is up
sonarqube_1      | 2019.08.08 03:28:16 INFO  app[][o.s.a.SchedulerImpl] SonarQube is up
```
其中可以识别连接的数据库`jdbc:postgresql://postgresql-db:5432/sonar`正式我们配置的数据库。

* 访问：localhost:9000
> username/password：admin/admin

* 如何删除
```
# cd docker-compose-sonarqube-postgresql
# docker-compose stop
# docker-compose rm
```
将会提示你是否删除sonarqube和postgresql两个容器，rm命令只能删除已停止的容器。

### FAQ

* 为什么运行脚本之后sonarqube使用的还是内置的数据库？
我们参考了`https://github.com/Hello-Nemo/docker-SonarQube.git`这个脚本，但是原始脚本环境变量`SONARQUBE_JDBC_URL:jdbc:postgresql://db:5432/sonar`中存在两处错误，我们已经修复：
* `SONARQUBE_JDBC_URL`与之后的值之间需要是`=`连接，而不是`:`
* 连接字符串中的【jdbc:postgresql://`db`:5432/sonar】中的`db`应该与`links`节点下面的值对应，最终应该对应到数据库Service配置的值。
例如，我们调整后均为`postgresql-db`：
```
version: '2'

services:

### SonarQube Container ###########################

    sonarqube:
        build: ./sonarqube
        ports:
            - "9000:9000"
        links:
            - postgresql-db
        environment:
            - SONARQUBE_JDBC_URL=jdbc:postgresql://postgresql-db:5432/sonar

            
### PostgreSQL Container #######################################

    postgresql-db:
      build: ./postgresql-db
      volumes:
        - ~/log/postgresql:/var/lib/postgresql/data
      ports:
        - "5432:5432"
      environment:
        - POSTGRES_USER=sonar
        - POSTGRES_PASSWORD=sonar
```

* links使用
链接到其它服务中的容器。使用服务名称(同时作为别名)或服务名称:服务别名 (`SERVICE:ALIAS`) 格式都可以。
```
links:
 - db
 - db:database
 - redis
```

使用的别名将会自动在服务容器中的 /etc/hosts 里创建。例如:
```
172.17.2.186  db
172.17.2.186  database
172.17.2.187  redis
```
被链接容器中相应的环境变量也将被创建。

* volumes使用
数据卷所挂载路径设置。可以设置宿主机路径 (`HOST:CONTAINER`) 或加上访问模式 (`HOST:CONTAINER:ro`)。
该指令中路径支持相对路径。例如
```
volumes:
 - /var/lib/mysql
 - cache/:/tmp/cache
 - ~/configs:/etc/configs/:ro
```

本例中，服务`postgresql-db`指定的是`~/log/postgresql:/var/lib/postgresql/data`，我们通过查看宿主机，得到如下目录：
可以看到，`~`宿主机Home目录下建立了`log`子目录，同时，再下一级为`postgresql`，与描述一致。
```
# ls -l ~/log/
total 4
drwx------ 19 systemd-bus-proxy root 4096 Aug  8 11:27 postgresql
```