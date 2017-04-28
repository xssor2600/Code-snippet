## Linux操作命令使用场景代码库<br>
记录一些linux操作的常用命令，在开发中或者日常使用中经常遇到的命令。<br>
* **linux 进程相关：**<br>
  * 查看系统tcp连接详情命令:<br>
  可以查看具体ip地址，端口发送的tcp连接信息，tcp连接状态:`# netstat -np|grep tcp` <br>
  *  统计tcp连接状态数量信息:<br>
  查看每个tcp连接状态的数量:`# netstat -n | awk '/^tcp/ {++S[$NF]} END {for(a in S) print a, S[a]}`<br>
  * 查看某个线程下的所有子线程系统资源占用信息:<br>
  该命令可以查看系统上的某个进行下的每个子进程占用的系统资源，就是使用**pidstat**命令:`# pidstat -p pid -t 1 n`,n代表显示次数。<br>
