
## windows系统安装<br>
记录一些在安装系统时候，遇到的一些问题，如何通过windows命令行解决。<br>
* windows无法安装到这个磁盘。选中的磁盘采用GPT分区形式<br>
在将已安装着Linux系统的主分区或者其他系统的主分区，安装windows7系统的时候，就会遇到这种问题。因为要安装win7,主分区要求是MBR格式，不像win8,wind10的GPT格式。可以参考 <u>[diskpart分区](https://jingyan.baidu.com/article/92255446efce49851748f463.html)</u><br>
所以就通过以下命令将window主分区转换成MRB：<br>
```shell
## 1.0 在系统安装选择磁盘，界面，通过按键“shift+f10”，调出window的“cmd”命令符。
## 2.0 在cmd黑窗口中操作，先通过diskpark命令，进入分区操作模式:
path > diskpart

## 3.0 通过list disk命令，将系统中可识别的磁盘列出来:
path > list disk

磁盘    状态     大小    可用  ...
====   =====    =====   =====
磁盘0   联机     ....    ....

## 4.0 通常可以从3.0步骤操作后，看到列出来的磁盘信息，通常选择编码0的磁盘作为主分区安装系统:
path > select 0
磁盘0现在是所选磁盘

## 5.0 若是这个选中磁盘中安装有系统，需要先进行格式化删除数据：
path > clean
Diskpart 成功清除了磁盘。

## 6.0 在格式化完后，可以通过convert命令，将该分区转换成mbr格式，就可以安装系统了:
path > convert mbr
....等待diskpart转换格式...

## 7.0 最后，在回到选择磁盘安装系统的界面，通过界面的刷新按钮，刷新后，就可以在那个主分区安装系统了。
```
