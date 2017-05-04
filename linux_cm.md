## Linux操作命令使用场景代码库<br>
记录一些linux操作的常用命令，在开发中或者日常使用中经常遇到的命令。<br>
* **linux 进程相关：**<br>
  * 查看系统tcp连接详情命令:<br>
  可以查看具体ip地址，端口发送的tcp连接信息，tcp连接状态:`# netstat -np|grep tcp` <br>
  *  统计tcp连接状态数量信息:<br>
  查看每个tcp连接状态的数量:`# netstat -n | awk '/^tcp/ {++S[$NF]} END {for(a in S) print a, S[a]}`<br>
  * 查看某个线程下的所有子线程系统资源占用信息:<br>
  该命令可以查看系统上的某个进行下的每个子进程占用的系统资源，就是使用**pidstat**命令:`# pidstat -p pid -t 1 n`,n代表显示次数。<br>
* **查看Linux系统配置：**<br>
   列举出一些用于查看Linux配置信息的命令集合<br>
  * 查看系统CPU配置详情命令:<br>
  可以查看系统上与CPU相关的所有配置信息：`cat /proc/cpuinfo`。若是想要从中过滤或者抓取一些关键数据可以配合使用grep命令。<br>
  eg:若是想查看CPU大小:` cat /proc/cpuinfo |grep "model name" && cat /proc/cpuinfo |grep "physical id`,Linux下可以在/proc/cpuinfo中看到每个cpu的详细信息。但是对于双核的cpu，在cpuinfo中会看到两个cpu。常常会让人误以为是两个单核的cpu。<br>
  * 查看系统的内存大小命令详情:<br>
  若是想要查看系统的内存详细信息，可以使用命令:`cat /proc/meminfo`。<br>
  若是仅仅想查看系统的总内存大小，则可以使用命令:`cat /proc/meminfo |grep MemTotal`。<br>
  * 查看系统磁盘大小<br>
  查看系统磁盘大小信息，使用命令:`fdisk -l |grep Disk`,不过要是在root管理员权限下操作。<br>
  * 查看内核/操作系统/CPU信息的linux系统信息命令:<br>
  使用命令：`uname -a`即可查看。<br>
  * 查看操作系统版本<br>
  若是想直接查看操作系统版本，使用命令:`head -n 1 /etc/issue`。<br>
  * 查看系统环境变量资源<br>
  Linux系统上会有很多环境变量，包括我们自己创建的变量。查看直接使用指令:`env`即可。<br>
<br>
 关于查看Linux系统配置的更多命令，可以参考:[查看linux系统常用的命令，Linux查看系统配置常用命令](http://www.cnblogs.com/xuchunlin/p/5671572.html)<br>