官网:www.git-scm.com

设置用户名和邮箱

~~~
//全局
git config --global user.email "sesame@163.com"
git config --global user.name "sesame"
仓库级
git config --local user.email "sesame@163.com"
git config --local user.name "sesame"
~~~

获得版本库：

```
git init
git clone
```

本部管理

~~~
git add
git commit -m ''
git commit -am '' add+commit
git rm
~~~

查看信息

~~~
git help
git log （-n 最近几条）（--pretty=oneline 提交日志一行显示）
git log --graph 加上图形化
git reflog  所有操作日志
git diff
~~~

远程协作

~~~
git pull
git push
~~~

查看工作目录状态

git status

从暂存区还原到修改状态

git rm --cached <file>



git rm
删除了一个文件
将被删除的文件纳入到暂存区 (stage, index)
若想恢复被删除的文件，需要进行两个动作:
git restore --staged <file>  将待删除的文件从暂存区恢复到工作区
git restore <file>  将工作区中的修改丢弃掉,保持与暂存区同步

rm
将test2.txt删除，这时，被删除的文件并未纳入翻存区当中。



git mv   重命名

git commit --amend -m '修改上次提交日志'



设置忽略提交文件

在项目目录创建 .gitignore 文件，在文件中配置忽略文件的表达式



查看文件修改历史

~~~
git blame <文件名>
~~~

diff比较不同

~~~
git diff:比较的是暂存区与工作区文件之间的差别。
git diff HEAD:比较的是最新的提交与工作区之间的差别。
git diff -cached:比较的是最新的提交与暂存区之间的差别。
~~~

