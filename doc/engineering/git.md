## 撤销

git push 提交完数据后后悔了怎么办？

!> 写在前面的话重要：删除上次提交后本地和远程仓库的数据都将删除，所以删除上次提交前，记得备份备份备份数据！！！

### reset

reset 是直接删除上次提交，历史记录中不会出现放弃的提交记录。

```powershell
git reset --hard HEAD^
git push origin master -f
```

HEAD 是指向最新的提交，上一次提交是 HEAD^,上上次是 HEAD^^,也可以写成 HEAD ～ 2 ,依次类推。

### revert

revert 是放弃指定提交的修改，但是会生成一次新的提交，需要填写提交注释，以前的历史记录都在。

```powershell
git revert HEAD
git push origin master
```
