# zzxia-kvm-manage

## 1 介绍
批量克隆、修改、删除、启动、自动启动KVM虚拟机。适合小企业使用。

### 1.1 功能：
1. 克隆虚拟机：通过编辑list.csv文件定义虚拟机信息，然后运行vm-clone.sh，选择克隆模板，然后按照list.csv清单克隆出想要的虚拟机
1. 修改虚拟机信息：【主机名、IP、IP子网掩码、网关、域名、DNS】，一般主要配合vm-clone.sh使用，也可以单独使用
1. 批量删除虚拟机指定虚拟机
1. 通过清单选择要删除的虚拟机
1. 通过清单、选择、参数设置要启动（或自动）的虚拟机

### 1.2 喜欢她，就满足她：
1. 【Star】她，让她看到你是爱她的；
3. 【Issue】她，告诉她有哪些小脾气，她会改的，手动小绵羊；
2. 【Fork】她，为她增加新功能，修Bug，让她更加卡哇伊；
4. 【打赏】她，为她买jk；
<img src="https://img-blog.csdnimg.cn/20210429155627295.jpg?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3poZl9zeQ==,size_16,color_FFFFFF,t_70#pic_center" alt="打赏" style="zoom:50%;" />


## 2 软件架构
Linux shell

## 3 安装教程
克隆到KVM服务器上即可

## 4 使用说明
请使用-h|--help参数运行sh脚本即可看到使用帮助
除了kvm，你还需要安装guestfs，在centos7上运行`yum install -y  libguestfs-tools`

### 4.1 list.csv
根据需要定制虚拟机信息，以逗号分隔，用#注释掉不需要的行
```csv
$ cat list.csv 
# VM_NAME,CPU(个),MEM(GB),NET名, IP1,IP_MASK1,GATEWAY1 ,DOMAIN,DNS1 DNS2
#################################################################################################
# test
v-192-168-11-190-deploy,1,2,br1,192.168.11.190,24,192.168.11.1,zjlh.lan,192.168.11.3 192.168.11.4
v-192-168-11-191-mast,4,8,br1,192.168.11.191,24,192.168.11.1,zjlh.lan,192.168.11.3 192.168.11.4
v-192-168-11-192-node,4,8,br1,192.168.11.192,24,192.168.11.1,zjlh.lan,192.168.11.3 192.168.11.4
v-192-168-11-193-node,4,8,br1,192.168.11.193,24,192.168.11.1,zjlh.lan,192.168.11.3 192.168.11.4
v-192-168-11-194-etcd,2,4,br1,192.168.11.194,24,192.168.11.1,zjlh.lan,192.168.11.3 192.168.11.4
#v-192-168-11-195-etcd,2,4,br1,192.168.11.195,24,192.168.11.1,zjlh.lan,192.168.11.3 192.168.11.4
#v-192-168-11-196-etcd,2,4,br1,192.168.11.196,24,192.168.11.1,zjlh.lan,192.168.11.3 192.168.11.4
v-192-168-11-197-repo,2,4,br1,192.168.11.197,24,192.168.11.1,zjlh.lan,192.168.11.3 192.168.11.4
```
### 4.2 克隆

**克隆前的建议：**
- 建议先制作好一个较为完美的模板虚拟机，然后在克隆时选择使用他
- 查看KVM环境变量文件`kvm.env`，看是否与你的实际情况相同，否则请修改它

```bash
$ ./vm-clone.sh --help

    用途：KVM上虚拟机克隆，并修改相关信息（主机名、IP、IP子网掩码、网关、域名、DNS）
    依赖：
        ./vm-img-modify.sh
    注意：本脚本在centos 7上测试通过
    用法：
        ./vm-clone.sh  [-h|--help]
        ./vm-clone.sh  <-f|--file {清单文件}>  < -q|--quiet  [-t|--template {虚拟机模板}] >
        ./vm-clone.sh  <-f|--file {清单文件}>  <-t|--template {虚拟机模板}>
    参数说明：
        $0   : 代表脚本本身
        []   : 代表是必选项
        <>   : 代表是可选项
        |    : 代表左右选其一
        {}   : 代表参数值，请替换为具体参数值
        %    : 代表通配符，非精确值，可以被包含
        #
        -h|--help      此帮助
        -f|--file      虚拟机清单文件，默认为【./list.csv】
            文件格式如下（字段之间用【,】分隔）：
            #VM_NAME,CPU(个),MEM(GB),NET名, IP1,IP_MASK1,GATEWAY1 ,DOMAIN,DNS1 DNS2
            v-192-168-1-2-nextcloud,2,4,br1, 192.168.1.2,24,192.168.11.1, zjlh.lan,192.168.11.3 192.168.11.4
            v-192-168-1-3-nexxxx,2,4,br1, 192.168.1.3,24,192.168.11.1, zjlh.lan,192.168.11.3
        -q|--quiet     静默方式
        -t|--templat   指定虚拟机模板
    示例:
        #
        ./vm-clone.sh  -h
        # 一般
        ./vm-clone.sh                       #--- 默认虚拟机清单文件【./list.csv】，非静默方式，手动选择模板
        ./vm-clone.sh  -t v-centos-1        #--- 默认虚拟机清单文件【./list.csv】，非静默方式，基于模板【v-centos-1】创建
        # 指定vm清单文件
        ./vm-clone.sh  -f vm.list                      #--- 使用虚拟机清单文件【vm.list】，非静默方式，手动选择模
        ./vm-clone.sh  -f vm.list  -t v-centos-1       #--- 使用虚拟机清单文件【vm.list】，非静默方式，基于模板【v-centos-1】创建
        # 静默方式
        ./vm-clone.sh  -q  -t v-centos-1               #--- 默认虚拟机清单文件【./list.csv】，静默方式，基于模板【v-centos-1】创建
        ./vm-clone.sh  -q  -t v-centos-1  -f vm.list   #--- 使用虚拟机清单文件【vm.list】，静默方式，基于模板【v-centos-1】创建
```
### 4.3 修改vm信息
```bash
$ ./vm-img-modify.sh 

    用途：修改KVM虚拟机主机名及网卡信息（主机名、IP、IP子网掩码、网关、域名、DNS）
    依赖：
    注意：本脚本在centos 7上测试通过
    用法：
        ./vm-img-modify.sh  [-h|--help]
        ./vm-img-modify.sh  <-q|--quiet>  [ {VM_NAME}  {NEW_IP}  {NEW_IP_MASK}  {NEW_GATEWAY} ]  <{NEW_DOMAIN}>  <{NEW_DNS1}>  <{NEW_DNS2}>
    参数说明：
        $0   : 代表脚本本身
        []   : 代表是必选项
        <>   : 代表是可选项
        |    : 代表左右选其一
        {}   : 代表参数值，请替换为具体参数值
        %    : 代表通配符，非精确值，可以被包含
        #
        -h|--help      此帮助
        -q|--quiet     静默方式
    示例:
        #
        ./vm-img-modify.sh  -h        #--- 帮助
        # 一般
        ./vm-img-modify.sh  v-192-168-1-3-nexxxx  192.168.1.3  24  192.168.11.1  zjlh.lan  192.168.11.3  192.168.11.4
        ./vm-img-modify.sh  v-192-168-1-3-nexxxx  192.168.1.3  24  192.168.11.1
        # 静默方式
        ./vm-img-modify.sh  -q  v-192-168-1-3-nexxxx  192.168.1.3  24  192.168.11.1  zjlh.lan  192.168.11.3  192.168.11.4
```

### 4.4 删除虚拟机
```bash
$ ./vm-rm.sh 

    用途：删除指定名称的kvm虚拟机
    注意：本脚本在centos 7上测试通过
    用法：
        ./vm-rm.sh  [-h|--help]
        ./vm-rm.sh  <-q|--quiet>  [{VM_NAME1}]  {VM_NAME2} ... {VM_NAMEn}
    参数说明：
        $0   : 代表脚本本身
        []   : 代表是必选项
        <>   : 代表是可选项
        |    : 代表左右选其一
        {}   : 代表参数值，请替换为具体参数值
        %    : 代表通配符，非精确值，可以被包含
        #
        -h|--help      此帮助
        -q|--quiet     静默方式
    示例:
        #
        ./vm-rm.sh  虚拟机1
        ./vm-rm.sh  虚拟机1  虚拟机2
        # 静默
        ./vm-rm.sh  -q  虚拟机1  虚拟机2
```

### 4.5 从列表中选择要删除的虚拟机
```bash
$ ./vm-rm-list.sh
```

### 4.6 启动（或自动启动）虚拟机
```bash
$ ./vm-start.sh -h

    用途：启动虚拟机；设置虚拟机自动启动
    依赖：
    注意：本脚本在centos 7上测试通过
    用法：
        ./vm-start.sh  [-h|--help]
        ./vm-start.sh  [ <-s|--start>  <-a|--autostart> ]  [ [-f|--file {清单文件}] | [-S|--select] | [-A|--ARG {虚拟机1} {虚拟机2} ... {虚拟机n}] ]
    参数说明：
        $0   : 代表脚本本身
        []   : 代表是必选项
        <>   : 代表是可选项
        |    : 代表左右选其一
        {}   : 代表参数值，请替换为具体参数值
        %    : 代表通配符，非精确值，可以被包含
        #
        -h|--help      此帮助
        -s|--start     启动虚拟机
        -a|--autostart 开启自动启动虚拟机
        -f|--file      从文件选择虚拟机（默认），默认文件为【./list.csv】
            文件格式如下（字段之间用【,】分隔）：
            #VM_NAME,CPU(个),MEM(GB),NET名, IP1,IP_MASK1,GATEWAY1 ,DOMAIN,DNS1 DNS2
            v-192-168-1-2-nextcloud,2,4,br1, 192.168.1.2,24,192.168.11.1, zjlh.lan,192.168.11.3 192.168.11.4
            v-192-168-1-3-nexxxx,2,4,br1, 192.168.1.3,24,192.168.11.1, zjlh.lan,192.168.11.3
        -S|--select    从KVM中选择虚拟机
        -A|--ARG       从参数获取虚拟机
    示例:
        #
        ./vm-start.sh  -h
        # 一般（默认从默认文件）
        ./vm-start.sh  -s                   #--- 启动默认虚拟机清单文件【./list.csv】中的虚拟机
        ./vm-start.sh  -s  -a               #--- 启动默认虚拟机清单文件【./list.csv】中的虚拟机，并设置为自动启动
        ./vm-start.sh  -a                   #--- 自动启动默认虚拟机清单文件【./list.csv】中的虚拟机
        # 从指定文件
        ./vm-start.sh  -s  -f my_vm.list    #--- 启动虚拟机清单文件【my_vm.list】中的虚拟机
        ./vm-start.sh  -a  -f my_vm.list    #--- 自动启动虚拟机清单文件【my_vm.list】中的虚拟机
        # 我选择
        ./vm-start.sh  -s  -S               #--- 启动我选择的虚拟机
        ./vm-start.sh  -a  -S               #--- 自动启动我选择的虚拟机
        # 指定虚拟机
        ./vm-start.sh  -s  -A  vm1 vm2      #--- 启动虚拟机【vm1、vm2】
        ./vm-start.sh  -a  -A  vm1 vm2      #--- 自动启动虚拟机【vm1、vm2】
```

### 4.7 启动（或自动启动）虚拟机
```bash
$ ./vm-shutdown.sh -h

    用途：shutdown虚拟机
    依赖：
    注意：本脚本在centos 7上测试通过
    用法：
        ./vm-shutdown.sh  [-h|--help]
        ./vm-shutdown.sh  [ [-f|--file {清单文件}] | [-S|--select] | [-A|--ARG {虚拟机1} {虚拟机2} ... {虚拟机n}] ]
    参数说明：
        $0   : 代表脚本本身
        []   : 代表是必选项
        <>   : 代表是可选项
        |    : 代表左右选其一
        {}   : 代表参数值，请替换为具体参数值
        %    : 代表通配符，非精确值，可以被包含
        #
        -h|--help      此帮助
        -f|--file      从文件选择虚拟机（默认），默认文件为【./list.csv】
            文件格式如下（字段之间用【,】分隔）：
            #VM_NAME,CPU(个),MEM(GB),NET名, IP1,IP_MASK1,GATEWAY1 ,DOMAIN,DNS1 DNS2
            v-192-168-1-2-nextcloud,2,4,br1, 192.168.1.2,24,192.168.11.1, zjlh.lan,192.168.11.3 192.168.11.4
            v-192-168-1-3-nexxxx,2,4,br1, 192.168.1.3,24,192.168.11.1, zjlh.lan,192.168.11.3
        -S|--select    从KVM中选择虚拟机
        -A|--ARG       从参数获取虚拟机
    示例:
        #
        ./vm-shutdown.sh  -h
        # 一般（默认从默认文件）
        ./vm-shutdown.sh                   #--- shutdown默认虚拟机清单文件【./list.csv】中的虚拟机
        # 从指定文件
        ./vm-shutdown.sh  -f my_vm.list    #--- shutdown虚拟机清单文件【my_vm.list】中的虚拟机
        # 我选择
        ./vm-shutdown.sh  -S               #--- shutdown我选择的虚拟机
        # 指定虚拟机
        ./vm-shutdown.sh  -A  vm1 vm2      #--- shutdown虚拟机【vm1、vm2】
```

## 5 参与贡献

1.  Fork 本仓库
2.  新建 Feat_xxx 分支
3.  提交代码
4.  新建 Pull Request


## 特技

1.  使用 Readme\_XXX.md 来支持不同的语言，例如 Readme\_en.md, Readme\_zh.md
2.  Gitee 官方博客 [blog.gitee.com](https://blog.gitee.com)
3.  你可以 [https://gitee.com/explore](https://gitee.com/explore) 这个地址来了解 Gitee 上的优秀开源项目
4.  [GVP](https://gitee.com/gvp) 全称是 Gitee 最有价值开源项目，是综合评定出的优秀开源项目
5.  Gitee 官方提供的使用手册 [https://gitee.com/help](https://gitee.com/help)
6.  Gitee 封面人物是一档用来展示 Gitee 会员风采的栏目 [https://gitee.com/gitee-stars/](https://gitee.com/gitee-stars/)

