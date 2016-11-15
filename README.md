# 代码部署系统

日常开发中，经常会有把本地的代码push到qa机器、worker机器、正式环境机器，无底线的手动重复操作为我们的工作增添了太多的枯燥。  

如果需要同步代码的机器有成百上千台，每次手动去操作实在是在给自己找麻烦，不但效率低下，而且容易出事故。 

如果你没有一套好用的代码推送系统、或者正在苦寻，那么这套基于shell deploy也许是你不错的选择！  

codedeploy 支持多项目、多集群、多模式的代码部署系统。

conf.sh 脚本是deploy配置文件，其它文件不要去改动它。

非正式的线上环境机器，谁最后一次部署的代码，需要本人才能解锁，防止代码被冲洗。  

codedeploy 采用轮询推送，如果某集群有100台机器，在推送这集群的时候，其中有几台机器推送失败了，会提示你重新推送失败的机器。

注：  
集群代码部署, 建议配置好无密码登录(RSA密钥认证)。否则部署过程中每一台机器都需要手动输入用户(conf.sh中变量SSH_USER)密码。

## 1. 命令手册

#### 1.1 git 模式部署代码
```Bash
sh deploy-qa.sh     分支名称 qa集群编号
sh deploy-ot.sh     分支名称 online集群编号
sh deploy-worker.sh 分支名称 worker集群编号
```

#### 1.2 file 模式部署代码
```Bash
sh deploy-qa.sh     file qa集群编号
sh deploy-ot.sh     file online集群编号
sh deploy-worker.sh file worker集群编号
```

#### 1.3 两种模式通用, 释放当前用户占用机器例子
```Bash
sh deploy-qa.sh     clean qa集群编号
sh deploy-ot.sh     clean online集群编号
sh deploy-worker.sh clean worker集群编号
```

#### 1.4 释放对机器的独占模式，让其他同事可以部署代码
```Bash
sh deploy-qa.sh     clean 集群编号
sh deploy-worker.sh clean 集群编号
sh deploy-ot.sh     clean 集群编号
```

### 1.5 特殊模式：rsync 代码推送，用于回归机上执行，不提供 clean 方法
```Bash
sh deploy-rsync.sh 集群编号
```

#### 多服务器批处理命令执行

##### 1.6 Debug模式
```Bash
sh exec.sh "command" debug
```

##### 1.7 非Dubug模式: 
```Bash
sh exec.sh "command"
```

## 2. 例子大全

##### 2.1 （file模式，代码部署例子）

首先打开文件conf.sh, 找到变量 **SSH_USER**，填入公共账号；   
继续找到变量 **project_name**、**qa**、**online**、**worker**、**local_web_path**、**remote_web_path**；（这些变量都在一起的）   
```Bash
[test@test01v ~/codedeploy]$ vim conf.sh

SSH_USER='public_user' # 部署代码, 用到的用户, 整个系统通用

number=1
project_name[$number]="test.codedeploy.cn"

# 顾名思义, 这是线上机器了, 可以填写IP地址、hostname 多台机器以空格分割
online[$number]=""

# 顾名思义, 这是测试机器了，可以填写IP地址、hostname 多台机器以空格分割
qa[$number]="test01v.add.net"

# 顾名思义, 这是worker机器了，可以填写IP地址、hostname 多台机器以空格分割
worker[$number]=""

# 项目本地跟目录, 权限(wr)
local_web_path[$number]="/home/test/codedeploy"

# 项目部署到远端机器的目录, 前提这个目录有读写(wr)权限
remote_web_path[$number]="/home/web/test/${project_name[1]}"
```

###### 2.2 file模式 执行结果如下(分支模式同理，把“file”参数换成“git分支名称”)
```Bash
[test@test01v ~/codedeploy]$  sh deploy-qa.sh file 1

[file模式] - [qa1 环境]

本地配置 [1] 开始打包 ... ...
本地配置 [1] 打包成功: /tmp/codedeploy_tmp/test/test.codedeploy.cn/codedeploy_test.codedeploy.cn_20160627153426.tar.gz
本地配置 [1] 文件大小: 172K

1. test01v.add.net => 代码上传中 ... ...
1. test01v.add.net => 代码上传成功: /home/public_user/codedeploy_history/test.codedeploy.cn/codedeploy_test.codedeploy.cn_20160627153426.tar.gz
1. test01v.add.net => 耗时: 150ms	[非常快]
1. test01v.add.net => 开始解压远程代码包 ... ...
1. test01v.add.net => 代码解压成功: /home/web/test/test.codedeploy.cn
1. test01v.add.net => 清理远程过期的tar包


[result] 机器数量: 1 全部部署成功!
```

##### 2.3 （rsync模式，代码部署例子）

首先打开文件conf.sh, 找到变量 **SSH_USER**，填入公共账号；   
继续找到变量 **project_name**、**rsync**、**local_web_path**、**remote_web_path**；（这些变量都在一起的）   
```Bash
[test@test01v ~/codedeploy]$ vim conf.sh

SSH_USER='public_user' # 部署代码, 用到的用户, 整个系统通用

number=1
project_name[$number]="test.codedeploy.cn"

# 顾名思义, 这是回归机机器了，可以填写IP地址、hostname 多台机器以空格分割
rsync[$number]="test01v.add.net"

# 项目本地跟目录, 权限(wr)
local_web_path[$number]="/home/test/codedeploy"

# 项目部署到远端机器的目录, 前提这个目录有读写(wr)权限
remote_web_path[$number]="/home/web/test/${project_name[1]}"
```

###### 2.4 rsync模式 执行结果如下
```Bash
[test@test01v ~/codedeploy]$  sh deploy-rsync.sh 1

[rsync模式] - [环境 1]

1. test01v.add.net => 代码同步中 ... ...
1. test01v.add.net => 代码同步成功: /home/web/test/test.codedeploy.cn
1. test01v.add.net => 耗时: 183ms	[非常快]


[result] 机器数量: 1 全部部署成功!

```

##### 2.5 (释放占用的环境，只支持模式：file、git)
```Bash
[test@test01v ~/codedeploy]$  sh deploy-qa.sh clean 1

[qa1 环境] - [执行操作 clean]

1. test01v.add.net => 环境释放成功.
```

##### 2.6 (清理当前用户登录日志)
```Bash
[test@test01v ~/codedeploy]$  sh clean-log.sh

clean log omplete!
```

##### 2.7 （多服务器批处理命令执行例子）

首先打开文件conf.sh, 找到变量 **EXEC_HOSTS**，填入需要执行的主机名称，以可以填写IP地址；  
继续找到变量 **SSH_USER**，填入公共账号；
```Bash
[test@test01v ~/codedeploy]$ vim conf.sh

EXEC_HOSTS='test01v.add.net 127.0.0.1' # 多个主机之间用空格分割
SSH_USER='public_user' # 部署代码, 用到的用户, 整个系统通用
```

###### 执行结果如下：
```Bash
[test@test01v ~/codedeploy]$ sh exec.sh 'hostname' debug

============== 服务器列表(开始) ==============

1. test01v.add.net
2. 127.0.0.1

============== 服务器列表(结束) ==============

确认服务器列表？ [y/n]: y

1. [ test01v.add.net ]

test01v.add.net

2. [ 127.0.0.1 ]

test02v.add.net

[result] 机器数量: 2 Done!
```

##### 2.8 集群与机器一对一怎么配置

这里假设有 8 台 QA机器，而且需要一对一推送场景。也就是说代码只部署其中一台QA机器；

```Bash
[test@test01v ~/codedeploy]$  vim conf.sh

number=8
 
for ((i = 1; i <= $number; i++))
do
    project_name[$i]="wifi.360.cn"
    online[$i]="" 
    worker[$i]=""
    local_web_path[$i]=`dirname $DEPLOY_DIRECTORY`
    remote_web_path[$i]="/home/www/"${project_name[$i]}
done
 
qa[1]="test01v.hostname"
qa[2]="test02v.hostname"
qa[3]="test03v.hostname"
qa[4]="test04v.hostname"
qa[5]="test05v.hostname"
qa[6]="test06v.hostname"
qa[7]="test07v.hostname"
qa[8]="test08v.hostname"
```

向 QA 1 服务器推送代码
```Bash
sh deploy/deploy-qa.sh master 1
```

向 QA 2 服务器推送代码，以此类推
```Bash
sh deploy/deploy-qa.sh master 2
```

## 3. 特别感谢

蔡玉光

## 4. 联系我们

[codedeploy 官网] (http://www.cdvphp.com/wiki/index.php?title=Cd_start)   
[Team: 随时随地，手机免费上网] (http://wifi.360.cn/)   
