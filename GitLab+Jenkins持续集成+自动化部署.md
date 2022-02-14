# GitLab+Jenkins持续集成+自动化部署

## 熟悉Jenkins

### （1）登录Jenkins webUI界面创建第一个项目

![avatar](image/devops01.png)

### （2）输入项目名称(My-freestyle-job)并选择构建一个自由风格的软件项目

![avatar](image/devops02.png)

### （3）上面创建完成后跳转进来后进行配置，选择丢弃旧的构建（下面保持天数一般在5~7天即可）

![avatar](image/devops03.png)

### （4）接着上面选择构建，然后选择Execute Shell 来执行shell命令

![avatar](image/devops04.png)

### （5）既然能执行shell命令，那么我们执行一个pwd，看下默认的工作目录在哪里

![avatar](image/devops05.png)

### （6）上面保存后点击立即构建，就会在下面生成一个build history，（出现蓝色即表示正常，若红色即表示有问题）

![avatar](image/devops06.png)

### （7）构建完成，我们可以点击build ID下拉框选择Console Output 来查看详细信息

![avatar](image/devops07.png)

### （8）通过输出信息我们可以看到Jenkins默认的工作目录在 /var/lib/jenkins/workspace/(项目名称)

![avatar](image/devops08.png)

### （9）既然能执行shell命令，那么我们创建一个文件试试；返回工作台，点击配置

![avatar](image/devops09.png)

### （10）创建一个test.txt文件并保存

![avatar](image/devops10.png)

### （11）同上面一样，我们点击立即构建

![avatar](image/devops11.png)

### （12）同样查看构建后的控制台输出，可以看到构建成功

![avatar](image/devops12.png)

### （13）既然构建成功，那么我们进入服务器进行验证是否创建成功

```console

[root@jenkins ~]# cd /var/lib/jenkins/workspace/My-freestyle-job/       #进入到项目目录

[root@jenkins My-freestyle-job]# 

[root@jenkins My-freestyle-job]# ls      #查看确实创建了文件

test.txt

[root@jenkins My-freestyle-job]# rm -rf test.txt     #因为没有什么用，测试完了，我们将其删除

[root@jenkins My-freestyle-job]#

```

## GitLab+Jenkins实现自动更新代码

### 流程图示例:

![avatar](image/devops流程简化版.png)

### 说明：

通过gitlab+Jenkins实现代码的自动更新同步代码到web服务器站点目录。此处示例后端web服务器使用nginx。本次项目示例使用码云上面的一个html项目（https://gitee.com/kangjie1209/monitor.git）

### 环境说明：

| IP地址 | 服务 |

| :----: | :---- |

| 192.168.1.21 | GitLab服务器 |

| 192.168.1.22 | Jenkins服务器 |

| 192.168.1.26 | Nginx服务器 |

https://gitee.com/kangjie1209/monitor.git 项目访问示意图：

![avatar](image/monitor.png)

## GitLab配置

### 说明：

　　首先我们在gitlab上面创建一个群组，并创建一个dev开发用户（用于提交代码等），同时在Jenkins服务器上面生成ssh秘钥并将key添加到新建用户dev的ssh认证下面，并创建一个代码仓库，并将代码copy进去。

### 具体操作步骤：

#### （1）登录gitlab点击项目，然后点击创建一个群组

![avatar](image/gitlab01.png)

#### （2）点击新建群组

![avatar](image/gitlab02.png)

#### （3）输入新建群组的相关信息并点击创建

![avatar](image/gitlab03.png)

#### （4）点击设置选择新建用户

![avatar](image/gitlab04.png)

#### （5）输入账号相关信息并点击创建用户

![avatar](image/gitlab05.png)

#### （6）点击编辑，为上面新创建的用户设置密码

![avatar](image/gitlab06.png)

#### （7）给这个账号设置密码并保存

![avatar](image/gitlab07.png)

#### （8）将上面创建的用户添加到devops组中并给与开发者权限

![avatar](image/gitlab08.png)

#### （9）找到刚刚创建的dev用户并给予开发人员权限，然后增加用户到群组

![avatar](image/gitlab09.png)

#### （10）回到Jenkins上面生成ssh秘钥

```console

[root@jenkins ~]# ssh-keygen

[root@jenkins ~]# cat .ssh/id_rsa.pub 

ssh-rsa AAAAB3NzaC1yc2EAAAADAQABAAABAQCt9+3rxKFGTEeT4F4q4AEc+So3A4jMBMpW6Ojoy5h1VQhNVV6meuOGp7ltJXtmY0Bm7tw8S/KDVXPSCvDi3QQgzWe2ZQmG+Y62SKcXpDxOJue98OHDNoxcm2kJl/xeFZddQZd6eohbBA9Au4SPMINLsGR6MEYXH4JHM6rXzf9QN4Q5GYYEFTTZvu/PxoDq46+A/mkO5aklBjaih6YSH3q7nr2rEhQJ64b6wEkfBjptfzKm54TZBJAPzycHrXe5P68tPZ2CNgJN+40XGrkg/MVjf9D9EwjyvsNcdLzTHDsClc3Jh8/8tfSFFzVFonyKyjal9amCvzdVbnUEqQSgRTWT root@jenkins

```

#### （11）回到gitlab服务器上面使用dev用户登录，然后点击头像选择设置

![avatar](image/gitlab10.png)

#### （12）上面的设置点进来后，选择SSH 秘钥， 然后复制Jenkins服务器上生成的秘钥然后点击增加秘钥

![avatar](image/gitlab11.png)

#### （13）切回root用户，新建一个项目

![avatar](image/gitlab12.png)

#### （14）选择导入项目并输入相关信息，然后点击创建项目

![avatar](image/gitlab13.png)

#### （15）导入成功后可以看见代码和导入成功提示，至此gitlab暂时部署到此

![avatar](image/gitlab14.png)

## Jenkins配置

### 说明：

　　还是使用前面示例创建的My-freestyle-job项目，配置导入上面gitlab创建的项目

### 具体操作步骤：

#### （1）点击项目名称开始配置

![avatar](image/jenkins01.png)

#### （2）上面点击进去后点击配置

![avatar](image/jenkins02.png)

#### （3）选择源码管理，选Git，输入仓库URL地址（如果下方出现红色及表示出错了），然后选择保存

![avatar](image/jenkins03.png)

#### （4）立即构建

![avatar](image/jenkins04.png)

#### （5）查看构建详细信息

![avatar](image/jenkins05.png)

#### （6）进入到服务器中查看项目目录是否将代码拉取成功

```console

[root@jenkins ~]# cd /var/lib/jenkins/workspace/My-freestyle-job/

[root@jenkins My-freestyle-job]# ls 

404.html              efficiencyAnalysis.html  js                       other-components.html

alerts.html           energy_consumption.html  keyInfo.html             profile-page.html

assets                file-manager.html        labels.html              QHME.iml

buttons.html          fonts                    LICENSE                  readme.md

calendar.html         form-components.html     list-view.html           real-time.html

charts.html           form-elements.html       login.html               sa.html

components.html       form-examples.html       media                    tables.html

content-widgets.html  form-validation.html     media.html               test.txt

css                   images-icons.html        messages.html            typography.html

deviceManager.html    img                      mstp_105_SuperAdmin.iml  userMng.html

dianfei.html          index.html               mstp_map.html

```

可以看到Jenkins已经成功从gitlab上面拉取代码，接下来我们先将Web站点配置好再看如何自动同步到Web服务器上面。



## 自动化同步代码

### 说明：

　　上面已经完成了Jenkins从gitlab上面拉取代码，及nginx web服务器站点的布置，现在需要实现的是如何Jenkins上面构建后自动同步到web服务器。想一下，在上面我们可以执行shell命令，那么一定也就可以执行shell脚本。so 我们编辑一个同步脚本，然后构建触发脚本同步到web服务器上面。

### 具体操作步骤：



#### （1）在Jenkins服务器上面编写同步脚本，由于是通过脚本拷贝到web服务器的站点目录，所以需要先做一个ssh秘钥认证

```console

[root@jenkins ~]# ssh-copy-id -i 192.168.1.26

[root@jenkins ~]# mkdir /server/scripts -p

[root@jenkins ~]# cd /server/scripts/

[root@jenkins scripts]# vim deploy.sh

#!/bin/bash



CODE_DIR="/var/lib/jenkins/workspace/My-freestyle-job/"     #项目目录

DATE_TIME=`date +%Y-%m-%d-%H-%M-%S`     #时间格式

TAR_NAME=web-${DATE_TIME}.tar.gz        #打包后的名字

WEB_ADDR=192.168.1.26                   #web服务器地址

WEB_DIR="/usr/share/nginx/"             #web服务器站点目录的上一级 "/usr/share/nginx/html"

WEB_NEWDIR_NAME=web-${DATE_TIME}        #web服务器新建的站点目录名字



#进入到项目目录并进行打包代码

tarcf_code(){

    cd $CODE_DIR && tar czf /opt/$TAR_NAME ./*

}



#拷贝到web服务器的站点目录的上一级

scp_code(){

    scp /opt/$TAR_NAME $WEB_ADDR:$WEB_DIR

}



#连接web服务器进行解压压缩包到新的一个已时间命名的站点目录

tarxf_code(){

    ssh $WEB_ADDR "cd $WEB_DIR && mkdir $WEB_NEWDIR_NAME && tar xf $TAR_NAME -C $WEB_NEWDIR_NAME"

}



#将新建的站点目录与html站点目录做一个软链接

ln_code(){

    ssh $WEB_ADDR "cd $WEB_DIR && rm -rf html && ln -s $WEB_NEWDIR_NAME html"

}



del_code(){

    ssh $WEB_ADDR "cd $WEB_DIR && rm -rf $TAR_NAME"

}



main(){

    tarcf_code;

    scp_code;

    tarxf_code;

    ln_code;

    del_code;

}

main

```

#### （2）配置Jenkins，使用Jenkins调用部署脚本（此处写脚本全路径脚本名称）

![avatar](image/job01.png)

#### （3）配置自动触发构建、需要设置安全令牌Secret token，进入项目选择配置，设置相关信息，然后生成token，复制token（需要填写到gitlab上面）和Build when a chang上面提示的URL地址（http://192.168.1.22:8080/project/My-freestyle-job）

![avatar](image/job02.png)

#### （4）配置gitlab，添加token

![avatar](image/job03.png)

#### （5）测试Web钩子

![avatar](image/job04.png)

#### （6）上面已经配置完成，接下来就是测试，我们知道目前web服务站点访问得到的一个“Web Server”， 在另外一台机器上面克隆代码并进行修改（此处我就使用Jenkins顺便来做测试了），然后推送gitlab仓库

```console

[root@jenkins ~]# git clone git@192.168.1.21:devops/monitor.git

[root@jenkins ~]#

[root@jenkins ~]# cd monitor/

[root@jenkins monitor]#

[root@jenkins monitor]# vim index.html    //将里面的”移动能效管理平台“ 全部改成”GitLab+Jenkins自动化“

[root@jenkins monitor]#

[root@jenkins monitor]# git add .

[root@jenkins monitor]#

[root@jenkins monitor]# git commit -m "update index.html"

[root@jenkins monitor]#

[root@jenkins monitor]# git push origin master

```

#### （7）访问web站点，刷新页面

![avatar](image/job05.png)

#### （8）从上面页面可以看见已经自动更新了代码，接下来我们查看web服务器上面的web站点目录

```console

[root@web-nginx ~]# ll /usr/share/nginx

总用量 4

lrwxrwxrwx. 1 root root   23 3月  29 12:21 html -> web-2019-03-29-12-20-51

drwxr-xr-x. 2 root root  170 3月  27 22:03 modules

drwxr-xr-x. 8 root root 4096 3月  29 12:20 web-2019-03-29-12-20-51

[root@web-nginx ~]# ls /usr/share/nginx/html/

404.html              efficiencyAnalysis.html  js                       other-components.html

alerts.html           energy_consumption.html  keyInfo.html             profile-page.html

assets                file-manager.html        labels.html              QHME.iml

buttons.html          fonts                    LICENSE                  readme.md

calendar.html         form-components.html     list-view.html           real-time.html

charts.html           form-elements.html       login.html               sa.html

components.html       form-examples.html       media                    tables.html

content-widgets.html  form-validation.html     media.html               test.txt

css                   images-icons.html        messages.html            typography.html

deviceManager.html    img                      mstp_105_SuperAdmin.iml  userMng.html

dianfei.html          index.html               mstp_map.html

```

从上面我们可以看到站点目录多了一个“web-2019-03-29-12-20-51”目录， 且与html目录做了一个软链接，这样每次更新代码都会新建一个目录与站点目录做软链接，（好处，如果开发发现代码有bug，还可以及时回滚到前面的版本）至此，就完成了自动更新代码。

## 配置构建Jenkins返回构建状态到gitlab

#### （1）先在gitlab上面生成访问令牌token，点击用户处的设置

![avatar](image/gitlab15.png)

#### （2）访问令牌——>输入名称——>选择范围为api——>创建个人访问令牌，将生成的令牌token复制

![avatar](image/gitlab16.png)

#### （3）登录Jenkins——>选择系统管理——>选择系统设置——>选择Gitlab——输入名字——>输入URL——点击Add

![avatar](image/gitlab17.png)

#### （4）上面点击Add进来后，kind处选择Gitlab API token，粘贴上面复制的token，自定义ID和描述

![avatar](image/gitlab18.png)

#### （5）上面添加后，回到这里选择刚刚生成的这个token。然后点击保存

![avatar](image/gitlab19.png)

#### （6）上面保存后，进入到项目里选择配置，配置构建后操作，选择Publish ... to ...Gitlab，选择后直接保存就ok

![avatar](image/gitlab20.png)

![avatar](image/gitlab21.png)

#### （7）测试，点击立即构建

![avatar](image/gitlab22.png)

#### （8）回到gitlab上面查看，那里生成了一个对号

![avatar](image/gitlab23.png)

#### （9）点击对号，进来后可以看到流水线，下面一些信息，比如状态，提交等。

![avatar](image/gitlab24.png)

#### （10）点击上面Status下面的对号，可以看到如下图一样的有个jenkins-success

![avatar](image/gitlab25.png)

#### （11）再点击jenkins就跳转到jenkins上面去了，也可以查看输出，还可以查看工作空间，可以查看到所有代码，返回的话只能选择后退。

![avatar](image/gitlab26.png)

![avatar](image/gitlab27.png)

![avatar](image/gitlab28.png)

#### 以上就完成了配置构建状态返回到gitlab