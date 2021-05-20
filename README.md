# greenplum-ops

initialize.sh
```
# 登陆master节点
ssh -p 6222 gpadmin@127.0.0.1
# 或者 ssh -p 6222 gpadmin@0.0.0.0
# 密码: changeme

# 初始化配置文件
source /usr/local/greenplum-db/greenplum_path.sh

# 配置greenplum文件
artifact/prepare.sh -s 2 -n 2
# -s: segment(容器)的个数
# -n: 每个segment(容器)上primary的个数

# 初始化集群，会生成env.sh 文件(greenplum所需的环境变量)
gpinitsystem -a -c gpinitsystem_config
source env.sh

# 开启远程无密码访问
artifact/postinstall.sh

# 查看安装结果
ps -ef | grep postgres

# 查看集群状态
gpstate -s
```

repair.sh
```
#!/bin/bash

# whoami: gpadmin

# only run on master
host=`hostname`
if [ $host = "gp-0" ];then
   source /usr/local/greenplum-db/greenplum_path.sh

   # 注意下prepare.sh脚本，这里面有个删除master、data等目录的操作，需要注释掉
   artifact/prepare.sh -s 2 -n 2
   #gpinitsystem -a -c gpinitsystem_config
   source env.sh
   #artifact/postinstall.sh

   #  解决pod重建后，挂载目录权限问题导致启动失败
   sudo chmod 0700 -R $HOME/master/*
   for HOST in `cat $HOME/hostfile`; do
      ssh gpadmin@${HOST} "sudo chmod 0700 -R $HOME/data/*"
      ssh gpadmin@${HOST} "sudo chmod 0700 -R $HOME/master/*"
      ssh gpadmin@${HOST} "sudo chmod 0700 -R $HOME/mirror/*"
   done

   gpstart
fi
```
