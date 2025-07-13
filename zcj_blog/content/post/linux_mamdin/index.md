---
title: "Linux-mdadm工具学习"
date: 2022-06-12T22:00:38+08:00
draft: false
image: 1.png
categories:
    - computer
---

### mdadm

mdadm是linux下的一款标准的RAID的管理工具

#### 基本语法

mdadm [mode] <raid-device> [option] <componnet-devices>

#### 目前支持

 RAID0(striping), RAID1(mirroring), RAID4, RAID5, RAID6, RAID10, MULTIPATH和FAULTY

#### 模式

- Assemble：加入一个以前定义的阵列
- Build：创建一个没有超级块的阵列
- Create：创建一个新的阵列，每个设备具有超级块； -l,--level: RAID级别 -n,--raid-devices: 活动设备个数 -a {yes|no}: 是否自动为其创建设备文件 -c,--chunk: CHUNK大小, 默认为64K，重要的参数,决定了一次向阵列中每个磁盘写入数据的大小 -x,--spare-devices: 备用盘个数
- Manage： 管理阵列(如添加和删除)
- Misc：允许单独对阵列中的某个设备进行操作(如停止阵列)
- Follow or Monitor:监控RAID的状态
- Grow：改变RAID的容量或阵列中的设备数目 ；-n,--raid-devices=: 活动设备个数-x,--spare-devices=：备用盘个数-c,--chunk=: CHUNK大小, 默认为64K，重要的参数,决定了一次向阵列中每个磁盘写入数据的大小-z,--size=：阵列中从每个磁盘获取的空间总数-l,--level=: RAID级别-p,--layout=：设定raid5 和raid10的奇偶校验规则；并且控制故障的故障模式--parity: 类似于--layout--assume-clean:目前仅用于 --build 选项-R --run: 强制激活RAID，使用这个选项，设备上有旧的元数据信息的提示会被忽略-N --name=: 设定阵列的名称–-rounding：在linear array中的rounding factor，等于条带大小。
  

#### 可用的[options]

- -A，--assemble：加入一个以前定义的阵列
- -B,--build:Build a legacy array without superblocks .
- -C，--create：创建一个新的阵列
- -Q--query：查看一个device，判断它为一个md device或是一个md阵列的一部分
- -D，--detail：打印一个或多个md device的详细信息
- -E，--examine：打印device上的md superblock 的内容
- -F，--follow，--monitor：选择Monitor模式
- -G，--grow：改变在用阵列的大小或形态
- -h，--help：帮助信息，用在以上选项后，则显示该选项信息--help-options
- -V,--version
- -V，--verbose：显示细节
- -b,--brief：较少的细节。用于--detail和--examine选项
- -f,--force
- -c，--config=：指定配置文件，缺省为/etc/mdadm/mdadm. conf
- -s，--scan：扫描配置文件或/proc/mdstat以搜寻丢失的信息。配置文件/etc/mdadm/mdadm. conf
- create或build使用的选项：
- -c,--chunk=:Specify chunk size of kibibytes.缺省为64.
- --rounding=:Specify rounding factor for linear array(==chunk size)
- -I,--level=:设定raid level.
- --create可用：linear，raid 0，0，stripe，raid 1，1，mirror，raid 4，4，raid 5，5，raid 6，6，multipath, mp.
- --build可用：linear，raid 0，0，stripe.
- -p，--parity=：设定raid 5的奇偶校验规则：eft- asymmetric ，left-symmetric，right- asymmetric , right-symmetric, la, ra, ls, rs.缺省为left-symmetric--layout=：类似于--parity
- -n，--raid-devices=：指定阵列中可用device数目，这个数目只能由--grow修改-x，--spare-devices=：指定初始阵列的富余device数目
- -z，--size=：组建RAID1/4/5/6后从每个device获取的空间总数--assume-clean:目前仅用于--build选项
- -R，-run：阵列中的某一部分出现在其他阵列或文件系统中时，m dadm会确认该阵列。此选项将不作确认。
- -f, --force:通常mdadm不允许只用一个device创建阵列，而且创建raid5时会使用一个device作为missingdrive。此选项正相反
- -a, --auto{=no,yes,md,mdp,part,p}{NN}

### 测试

#### 创建RAID5阵列

- 执行命令: mdadm --create /dev/md0 --level=5 --chunk=64 --raid-devices=4 --spare-devices=1 /dev/sd[b-f]

- 使用cat /proc/mdstat查看RAID5阵列状态

- 结果：

  ```
  Personalities : [linear] [multipath] [raid0] [raid1] [raid6] [raid5] [raid4] [raid10] 
  md0 : active raid5 sde[5] sdf[4](S) sdd[2] sdc[1] sdb[0]
        328704 blocks super 1.2 level 5, 64k chunk, algorithm 2 [4/4] [UUUU]
        
  unused devices: <none>
  显示设备盘为sdf为热备盘， 使用fdisk –l查看磁盘阵列, 显示磁盘大小为：Disk /dev/md0: 320 MB
  ```

####  停止RAID5阵列

- 执行命令: sudo mdadm -Ss 或者 mdadm --stop /dev/md0

- 使用cat /proc/mdstat查看RAID5阵列状态

- 结果

  ```
  Personalities : [linear] [multipath] [raid0] [raid1] [raid6] [raid5] [raid4] [raid10] 
  unused devices: <none>
  // 磁盘已经停止 mdadm: stopped /dev/md0
  ```

#### 启动RAID5阵列

- 执行命令: sudo mdadm -As

- 使用cat /proc/mdstat查看RAID5阵列状态

- 结果

  ```
  mdadm: /dev/md/0 has been started with 4 drives and 1 spare.
  
  Personalities : [linear] [multipath] [raid0] [raid1] [raid6] [raid5] [raid4] [raid10] 
  md0 : active raid5 sdb[0] sdf[4](S) sde[5] sdd[2] sdc[1]
        328704 blocks super 1.2 level 5, 64k chunk, algorithm 2 [4/4] [UUUU]
        
  unused devices: <none>
  
  ```

  

#### 查看md状态

- 执行命令：mdadm -D /dev/md0

- 结果：

  ```
  /dev/md0:
             Version : 1.2
       Creation Time : Wed Jun  8 14:28:56 2022
          Raid Level : raid5
          Array Size : 328704 (321.00 MiB 336.59 MB)
       Used Dev Size : 109568 (107.00 MiB 112.20 MB)
        Raid Devices : 4
       Total Devices : 5
         Persistence : Superblock is persistent
  
         Update Time : Wed Jun  8 14:28:57 2022
               State : clean 
      Active Devices : 4
     Working Devices : 5
      Failed Devices : 0
       Spare Devices : 1
  
              Layout : left-symmetric
          Chunk Size : 64K
  
  Consistency Policy : resync
  
                Name : zcjubuntu-virtual-machine:0  (local to host zcjubuntu-virtual-machine)
                UUID : 03cfb72d:971a754f:8236aa23:21b23fd3
              Events : 18
  
      Number   Major   Minor   RaidDevice State
         0       8       16        0      active sync   /dev/sdb
         1       8       32        1      active sync   /dev/sdc
         2       8       48        2      active sync   /dev/sdd
         5       8       64        3      active sync   /dev/sde
  
         4       8       80        -      spare   /dev/sdf
  
  ```

  

#### 模拟损坏

- 执行命令：mdadm /dev/md0 -f /dev/sdf

- 使用cat /proc/mdstat查看RAID5阵列状态

- 结果

  ```
  Personalities : [linear] [multipath] [raid0] [raid1] [raid6] [raid5] [raid4] [raid10] 
  md0 : active raid5 sde[5] sdf[4](F) sdd[2] sdc[1] sdb[0]
        328704 blocks super 1.2 level 5, 64k chunk, algorithm 2 [4/4] [UUUU]
        
  unused devices: <none>
  
  /dev/md0:
             Version : 1.2
       Creation Time : Wed Jun  8 14:28:56 2022
          Raid Level : raid5
          Array Size : 328704 (321.00 MiB 336.59 MB)
       Used Dev Size : 109568 (107.00 MiB 112.20 MB)
        Raid Devices : 4
       Total Devices : 5
         Persistence : Superblock is persistent
  
         Update Time : Wed Jun  8 14:44:51 2022
               State : clean 
      Active Devices : 4
     Working Devices : 4
      Failed Devices : 1
       Spare Devices : 0
  
              Layout : left-symmetric
          Chunk Size : 64K
  
  Consistency Policy : resync
  
                Name : zcjubuntu-virtual-machine:0  (local to host zcjubuntu-virtual-machine)
                UUID : 03cfb72d:971a754f:8236aa23:21b23fd3
              Events : 19
  
      Number   Major   Minor   RaidDevice State
         0       8       16        0      active sync   /dev/sdb
         1       8       32        1      active sync   /dev/sdc
         2       8       48        2      active sync   /dev/sdd
         5       8       64        3      active sync   /dev/sde
  
         4       8       80        -      faulty   /dev/sdf
  ```

  

#### 移除损坏的磁盘

- 执行命令：mdadm /dev/md0 -r /dev/sdf

- 使用cat /proc/mdstat查看RAID5阵列状态

- 结果

  ```
  Personalities : [linear] [multipath] [raid0] [raid1] [raid6] [raid5] [raid4] [raid10] 
  md0 : active raid5 sde[5] sdd[2] sdc[1] sdb[0]
        328704 blocks super 1.2 level 5, 64k chunk, algorithm 2 [4/4] [UUUU]
        
  unused devices: <none>
  
  /dev/md0:
             Version : 1.2
       Creation Time : Wed Jun  8 14:28:56 2022
          Raid Level : raid5
          Array Size : 328704 (321.00 MiB 336.59 MB)
       Used Dev Size : 109568 (107.00 MiB 112.20 MB)
        Raid Devices : 4
       Total Devices : 4
         Persistence : Superblock is persistent
  
         Update Time : Wed Jun  8 14:49:13 2022
               State : clean 
      Active Devices : 4
     Working Devices : 4
      Failed Devices : 0
       Spare Devices : 0
  
              Layout : left-symmetric
          Chunk Size : 64K
  
  Consistency Policy : resync
  
                Name : zcjubuntu-virtual-machine:0  (local to host zcjubuntu-virtual-machine)
                UUID : 03cfb72d:971a754f:8236aa23:21b23fd3
              Events : 20
  
      Number   Major   Minor   RaidDevice State
         0       8       16        0      active sync   /dev/sdb
         1       8       32        1      active sync   /dev/sdc
         2       8       48        2      active sync   /dev/sdd
         5       8       64        3      active sync   /dev/sde
  
  ```

  

#### 添加新的硬盘到已有阵列

- 执行命令：mdadm /dev/md0 -a /dev/sdf

- 使用cat /proc/mdstat查看RAID5阵列状态

- 结果

  ```
  md0 : active raid5 sdf[4](S) sde[5] sdd[2] sdc[1] sdb[0]
        328704 blocks super 1.2 level 5, 64k chunk, algorithm 2 [4/4] [UUUU]
        
  unused devices: <none>
  
  /dev/md0:
             Version : 1.2
       Creation Time : Wed Jun  8 14:28:56 2022
          Raid Level : raid5
          Array Size : 328704 (321.00 MiB 336.59 MB)
       Used Dev Size : 109568 (107.00 MiB 112.20 MB)
        Raid Devices : 4
       Total Devices : 5
         Persistence : Superblock is persistent
  
         Update Time : Wed Jun  8 14:53:45 2022
               State : clean 
      Active Devices : 4
     Working Devices : 5
      Failed Devices : 0
       Spare Devices : 1
  
              Layout : left-symmetric
          Chunk Size : 64K
  
  Consistency Policy : resync
  
                Name : zcjubuntu-virtual-machine:0  (local to host zcjubuntu-virtual-machine)
                UUID : 03cfb72d:971a754f:8236aa23:21b23fd3
              Events : 21
  
      Number   Major   Minor   RaidDevice State
         0       8       16        0      active sync   /dev/sdb
         1       8       32        1      active sync   /dev/sdc
         2       8       48        2      active sync   /dev/sdd
         5       8       64        3      active sync   /dev/sde
  
         4       8       80        -      spare   /dev/sdf
  
  ```