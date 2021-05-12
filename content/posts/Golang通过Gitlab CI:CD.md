---
title: "Golang 通过Gitlab持续集成/持续部署（CI/CD"
date: 2018-03-16T14:52:41+08:00

---



gitlab自带的CI工具可以非常简单的实现持续集成和持续部署，无需繁琐的安装。
工作流程:
将代码提交到gitlab->gitlab进入CI工作流->GitLab-Runner从gitlab代码服务器拉下最新代码，根据.gitlab-ci.yml执行PIPELINE,
![](media/15211786315251.png)

# 安装GitLab-Runner:
https://docs.gitlab.com/runner/#install-gitlab-runner

将这个Runner注册到你运行的Gitlab实例上,这样你的代码每次发生变化Gitlab就会通知这个Runner来执行CI流程，打开项目的Settings->CI/CD->Runners settings,找到Specific Runners下面的Gitlab服务器地址和Token
![](media/15211798850111.jpg)

https://docs.gitlab.com/runner/#install-gitlab-runner
运行以下命令执行注册

```
sudo gitlab-runner register
```

输入你的GitLab实例URL：

```
Please enter the gitlab-ci coordinator URL (e.g. https://gitlab.com )
xxx
```

输入您获得的注册Runner的token：

```
Please enter the gitlab-ci token for this runner
xxx
```

输入Runner的描述，你可以稍后在GitLab的用户界面中进行更改：

```
Please enter the gitlab-ci description for this runner
my-runner
```

输入与Runner关联的标签，稍后可以在GitLab的用户界面中进行更改：

```
Please enter the gitlab-ci tags for this runner (comma separated):
go
```

选择Runner是否应该选择没有标签的作业，以后可以在GitLab的用户界面中更改它（默认为false）：

```
Whether to run untagged jobs [true/false]:
true
```

选择是否将Runner锁定到当前项目，您可以稍后在GitLab的用户界面中进行更改。Runner特定时有用（默认为true）：

```
Whether to lock Runner to current project [true/false]:
true
```

输入Runner执行者：
如果您选择Docker作为您的执行程序，则会要求您为默认图像用于未在其中定义一个的项目.gitlab-ci.yml

```
Please enter the executor: ssh, docker+machine, docker-ssh+machine, kubernetes, docker, parallels, virtualbox, docker-ssh, shell:
docker
```

编辑.gitlab-ci.yml文件, 这里使用docker运行环境，省去了构建编译环境的麻烦。

```
image: golang:latest

variables:
  REPO_NAME: project

before_script:
  - mkdir -p $GOPATH/src/$(dirname $REPO_NAME)
  - ln -svf $CI_PROJECT_DIR $GOPATH/src/$REPO_NAME
  - cd $GOPATH/src/$REPO_NAME
```

`image: golang:latest`这里告诉Runner将代码运行在哪个容器内运行。`variables: REPO_NAME: project` 配置变量，跟shell里的变量是一个意思。`before_script`在执行所有步骤之前执行的一段脚本。这里我们在go/src/目录下根据项目名称创建了一个文件夹，然后从$CI_PROJECT_DIR环境变量获取到代码现在存在的目录，将目录建立一个软链接链接到刚刚创建的go/src/project目录。这样我们就完成了go编译环境的搭建，

```
stages:
    - test
    - build
    - deploy
```
stages 用于定义执行步骤，每个步骤可以在job中使用。stages的排序定义了Job执行的顺序：
根据上面定义。首先所有被定义为test的job会并行执行，如果所有的test都执行成功了然后就会执行所有定义为build的job.如果任何先前的Job失败，则提交被标记为failed并且不执行下一步

```
test:
    stage: test
    script:
      - cd tests
      - go test -v

compile:
  stage: build
  script:
    - go build -o $CI_PROJECT_DIR/demo
  artifacts:
    paths:
      - demo
 
 deploy_develop:
  stage: deploy
  script:
    - echo "$SSH_PRIVATE_KEY" >> ~/.ssh/id_dsa
    - chmod 600 ~/.ssh/id_dsa
    - echo -e "Host *\n\tStrictHostKeyChecking no\n\n" > ~/.ssh/config
    - rsync $CI_PROJECT_DIR/demo root@xxxx:/root/program
    - ssh root@xxxx "/usr/bin/supervisorctl restart demo"
  only:
    - develop
 
deploy_production:
  stage: deploy
  script:
    - echo "$SSH_PRIVATE_KEY" >> ~/.ssh/id_dsa
    - chmod 600 ~/.ssh/id_dsa
    - echo -e "Host *\n\tStrictHostKeyChecking no\n\n" > ~/.ssh/config
    - rsync $CI_PROJECT_DIR/demo root@xxxx:/root/program
    - ssh root@xxxx "/usr/bin/supervisorctl restart demo"
  only:
    - master
```

这里定义了4个job分别是test,compile,deploy_production,deploy_develop,首先我们执行测试，测试成功后编译出一个可执行文件，然后根据不同的提交分支发布到不同的服务器上部署


fatal: repository 'http://gitlab-ci-token:xxxxxxxxxxxxxxxxxxxx@192.168.15.95/lintao/work-doc.git/' not found
错误处理
vim /etc/gitlab-runner/config.toml
clone_url = "http://192.168.15.95:8080/"

FROM golang:latest

RUN apt-get update -y && apt-get install openssh-client -y && apt-get update -y && apt-get install rsync -y && mkdir -p ~/.ssh

