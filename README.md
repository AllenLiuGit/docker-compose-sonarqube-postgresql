# Docker Compose SonarQube and PostgreSQL

## Docker 搭建代码质量检测中文平台 SonarQube 同时配套数据库选择 PostgreSQL

### 下载编排文件
```
$ git clone https://github.com/Hello-Nemo/docker-SonarQube.git
$ cd docker-SonarQube
```

### 使用
```
$ docker-compose build
$ docker-compose up -d
```
或者
```
$ docker-compose up -d --build
```

访问：localhost:9000
> username/password：admin/admin
