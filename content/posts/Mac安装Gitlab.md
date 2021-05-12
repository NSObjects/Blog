---
title: "Mac安装Gitlab"
date: 2018-02-22T22:53:00+08:00
draft: false
---

安装环境:
Docker
[Gitlab中文镜像](https://hub.docker.com/r/twang2218/gitlab-ce-zh/)
4GB内存

Docker安装:

Mac版本下载地址
https://store.docker.com/editions/community/docker-ce-desktop-mac

下载完docker后添加Daocloud加速器
http://www.daocloud.io/mirror#accelerator-doc


![加速器_DaoCloud_-_业界领先的容器云平台](http://7u2kla.com1.z0.glb.clouddn.com/%E5%8A%A0%E9%80%9F%E5%99%A8_DaoCloud_-_%E4%B8%9A%E7%95%8C%E9%A2%86%E5%85%88%E7%9A%84%E5%AE%B9%E5%99%A8%E4%BA%91%E5%B9%B3%E5%8F%B0.png)

下载中文版Gitlab镜像

```
docker pull twang2218/gitlab-ce-zh
```

启动容器:

```
docker run -d \
    --hostname gitlab.example.com \
    -p 80:80 \
    -p 443:443 \
    -p 22:22 \
    --name gitlab \
    --restart unless-stopped \
    -v gitlab-config:/etc/gitlab \
    -v gitlab-logs:/var/log/gitlab \
    -v gitlab-data:/var/opt/gitlab \
    --network gitlab-net \
    twang2218/gitlab-ce-zh:10.4.3
```
进入容器修改配置

```
docker exec -it gitlab bash
vi /etc/gitlab/gitlab.rb
```

修改时区

```
 gitlab_rails['time_zone'] = 'Asia/Shanghai'
```

配置邮箱:

```
### Email Settings
 gitlab_rails['smtp_enable'] = true
 gitlab_rails['smtp_address'] = "smtp.qq.com"
 gitlab_rails['smtp_port'] = 465
 gitlab_rails['smtp_user_name'] = "xxxx@qq.com"
 gitlab_rails['smtp_password'] = "xxxx"
 gitlab_rails['smtp_authentication'] = "login"
 gitlab_rails['smtp_enable_starttls_auto'] = true
 gitlab_rails['smtp_tls'] = true
 gitlab_rails['gitlab_email_from'] = 'xxxx@qq.com'
```
修改完后更新下配置，[更多邮箱配置参考](https://docs.gitlab.com/omnibus/settings/smtp.html)

```
gitlab-ctl reconfigure
```
在浏览器打开http://127.0.0.1 首次安装会要几分钟初始化。可能会出现500的情况，初始化完成后就能使用了。
![星标项目_·_仪表盘_·_GitLab_和_2018-02-22_md](http://7u2kla.com1.z0.glb.clouddn.com/%E6%98%9F%E6%A0%87%E9%A1%B9%E7%9B%AE_%C2%B7_%E4%BB%AA%E8%A1%A8%E7%9B%98_%C2%B7_GitLab_%E5%92%8C_2018-02-22_md.png)

Gitlab文档
https://docs.gitlab.com/omnibus/README.html




