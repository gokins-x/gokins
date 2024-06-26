
# Gokins文档(wiki)
https://github.com/gokins-x/gokins/wiki

# Gokins: *More Power*
![](https://static01.imgkr.com/temp/5ca8a54f7d6544b6a2c740d5f559e5c4.jpg)
Gokins一款由Go语言和Vue编写的款轻量级、能够持续集成和持续交付的工具.

* **持续集成和持续交付**

  作为一个可扩展的自动化服务器，Gokins 可以用作简单的 CI 服务器，或者变成任何项目的持续交付中心

* **简易安装**

  Gokins 是一个基于 Go 的独立程序，可以立即运行，包含 Windows、Mac OS X 和其他类 Unix 操作系统。


* **安全**

  绝不收集任何用户、服务器信息，是一个独立安全的服务

## Gokins 官网

**地址 : http://gokins.cn**

可在官网上获取最新的Gokins动态

## Gokins Demo
http://demo.gokins.cn
```
用户名: guest
密码: 123456
```

## Quick Start

It is super easy to get started with your first project.


#### Step 1: 环境准备

- Mysql
- Dokcer(非必要)
- Postgres（可选，[示例说明](document/gokins_postgres.md)）
#### Step 2: 下载
- Linux下载:http://bin.gokins.cn/gokins-linux-amd64
- Mac下载:http://bin.gokins.cn/gokins-darwin-amd64
> 我们推荐使用docker或者直接下载release的方式安装Gokins`

#### Step 3: 启动服务

```
./gokins
``` 

#### Step 3: 安装Gokins

访问 `http://localhost:8030`进入到Gokins安装页面

![](https://static01.imgkr.com/temp/e484d9747dec43108325c22283abe39f.png)

按页面上的提示填入信息

默认管理员账号密码

`username :gokins `

`pwd: 123456 `

#### Step 4:  新建流水线

- 进入到流水线页面

![](https://static01.imgkr.com/temp/ce383350056d4a63872b868c8f169c39.png)



- 点击新建流水线

![](https://static01.imgkr.com/temp/a3c2a870c9d94956bda2a685cc447077.png)


填入流水线基本信息

- 流水线配置

```
version: 1.0
vars:
stages:
  - stage:
    displayName: build
    name: build
    steps:
      - step: shell@sh
        displayName: test-build
        name: build
        env:
        commands:
          - echo Hello World

```

关于流水线配置的YML更多信息请访问 [YML文档](http://gokins.cn/%E5%B7%A5%E4%BD%9C%E6%B5%81%E8%AF%AD%E6%B3%95/)


- 运行流水线

![](https://static01.imgkr.com/temp/f002a22738644c8dbd40f0860c2bbb9e.png)


`这里可以选择输入仓库分支或者commitSha,如果不填则为默认分支`

- 查看运行结果

![](https://static01.imgkr.com/temp/681c8ea0a7dc45bcb9fe14234c5761be.png)

# 非官方补充说明
> 以下内容、脚本、镜像、可执行文件均为个人修改发布，使用自由、风险自担。
## docker部署
```
docker run -d -p 8030:8030 -p 8031:8031 -v /usr/data/gokins:/root/.gokins liohao/my-gokins:latest

docker run -d --name gokinsr-node \
    -e GOKINS_SERVHOST=172.17.0.1:8031 \
    -e GOKINS_SERVSECRET=maoguorui666 \
    -e GOKINS_PLUGIN=build@node \
    -v /var/run/docker.sock:/var/run/docker.sock \
    --entrypoint ./gokinsr-alpine \
    liohao/my-gokinsr:latest
```
## 编译
```
# 安装依赖
go mod download

# window环境编译可执行文件
$env:CGO_ENABLED="0"
$env:GOOS=" windows"
$env:GOARCH="amd64"
go build -o gokins.exe main.go

# window10环境编译Linux可执行文件
$env:CGO_ENABLED="0"
$env:GOOS="linux"
$env:GOARCH="amd64"
go build -o gokins main.go
```
## 流水线配置示例
> 使用制品部署静态网站资源到远程服务器
```yml
version: 1.0
vars: #不是所有地方都可以使用变量，注意参数大小写
  host: 127.0.0.1:22
stages:
  - stage:
    displayName: 构建
    name: simple
    steps:
      - step: shell@sh
        displayName: 打包
        name: build
        env:
        commands:
          - echo success!
      - step: shell@sh
        displayName: 生成制品
        name: art
        artifacts:
          - scope: repo #repo存档不会删除、pipe流水线、var流水线变量
            repository: ibepmjwt #制品库id
            name: simpleHtml #制品库名称
            path: ./dist #要打包的源码目录./dist ./public
        commands:
          - echo success!
  - stage:
    displayName: 发布
    name: publish
    steps:
      - step: shell@ssh
        displayName: 制品部署
        name: artPublish
        input:
          host: ${{host}}
          user: ${{user}}
          pass: ${{pwd}}
        useArtifacts:
          - scope: repo
            repository: ibepmjwt
            name: simpleHtml #name: 制品库中的制品名,可以使用name@xxx获取某个版本的制品(默认使用最新)
            alias: DIST #默认与name字段一致
            path: dist #要下载的制品目录，一般同artifacts.path
            isUrl: true #自动写入环境变量 $ARTIFACT_DOWNURL_${alias}
        commands:
          - echo $ARTIFACT_DOWNURL_DIST
          - wget -O public.zip $ARTIFACT_DOWNURL_DIST
          - unzip -o public.zip
          - cp -f ./dist/* www/sites/index
```

> 使用私有仓库到远程服务器部署docker，需要Dockerfile请自行了解
```yml
version: 1.0
vars:
  host: 192.168.1.100:22
  dockerHost: 192.168.1.101:5000
  projectName: v3temp
  projectPort: 80
  servePort: 3210
stages:
  - stage:
    displayName: 打包
    name: build
    steps:
      - step: shell@sh
        displayName: 打包流程
        name: build
        env:
        commands:
          - echo $(printenv)
          - echo $(ls -A)
          - docker build -t ${{projectName}}:latest .
          - docker tag ${{projectName}}:latest ${{dockerHost}}/${{projectName}}:latest
          - docker push ${{dockerHost}}/${{projectName}}:latest
  - stage:
    displayName: 发布
    name: publish
    steps:
      - step: shell@ssh
        displayName: 发布流程
        name: publish
        input:
          host: ${{host}}
          user: ${{user}}
          pass: ${{pwd}}
        commands:
          - echo $(printenv)
          - echo $(ls -A)
          - docker pull ${{dockerHost}}/${{projectName}}:latest
          - docker rm -f ${{projectName}}
          - |
            docker run -d -p ${{servePort}}:${{projectPort}} \
            --name ${{projectName}} ${{dockerHost}}/${{projectName}}:latest
```
### 额外节点部署
+ 使用简单无须使用SSH、制品库、私有镜像仓库
+ 直接通过gokinsr传递git分支到节点，通过shell打包部署
+ 需要在gokinsr中配置好节点信息
+ 这里我的gokinsr运行在docker内，所以使用的都是通过docker命令操作

> Dockerfile文件部署到docker
```yml
version: 1.0
vars:
  projectName: v3temp
  port: 2360
stages:
  - stage:
    displayName: build
    name: build
    steps:
      - step: shell@sh
        displayName: npm-build-1
        name: build
        env:
        commands:
          - echo pullSuccess
      - step: build@node
        displayName: deploy
        name: publish
        env:
          mode: development
        commands:
          - echo $(ls -A)
          - docker build -t ${{projectName}}/data .
          - docker rm -f ${{projectName}}
          - docker run -d -p ${{port}}:80 --name ${{projectName}} ${{projectName}}/data
```

> docker挂载目录部署静态资源
```yml
version: 1.0
vars:
stages:
  - stage:
    displayName: build
    name: build
    steps:
      - step: shell@sh
        displayName: npm-build-1
        name: build
        env:
        commands:
          - echo pullSuccess
      - step: build@node
        displayName: deploy
        name: publish
        env:
        commands:
          - echo $(ls -A)
          - docker run -d --name temp-copy -v /www/www.demo.com/index/:/data alpine tail -f /dev/null
          - docker cp . temp-copy:/data/
          - docker stop temp-copy
          - docker rm temp-copy
```
