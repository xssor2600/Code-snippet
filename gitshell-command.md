## Github命令操作<br>
记录一些github在命令行操作任务的代码块。<br>

* 添加文件，写入内容并推送到远程库

```shell
echo "# Code-snippet" >> README.md
git init
git add README.md
git commit -m "first commit"
git remote add origin https://github.com/arden2600/Code-snippet.git
git push -u origin master

```
