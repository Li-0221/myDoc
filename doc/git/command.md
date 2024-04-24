# 前言

某天闲来无事，看看常用的后台管理模版有新需求，就挑了一个开始做起来。

1. 敲代码
2. 提交 pr
3. 管理员：你这个没有加 i18n
4. 敲代码
5. 提交 pr
6. 管理员：好像把这个东西单独抽出来比较好
7. 敲代码
8. 提交 pr
9. 管理员：你这个 commit 太多了，你把他们合并成一个 commit 吧
10. 找 git 命令...问 GTP...

# 命令

## fetch

从远程仓库下载最新的变更，但不会自动合并到你的本地分支。它会将远程仓库的变更保存到本人的 origin 分支中，你可以通过合并或者重新基于这些变更来更新你的本地分支。

## rebase

将当前分支的提交移动到另一个分支的最新提交之后。它会将你当前分支的提交逐个应用到目标分支上，并在每个提交应用完成后，将当前分支指向新的提交。  
这个命令通常用于保持提交历史的整洁性，尤其是在与主分支进行合并时。

> 当使用 git rebase 时，通常会涉及两个分支：当前分支和目标分支。

### 基本语法如下：

```bash
git rebase <目标分支>
```

### 示例：

假设有两个分支：feature 和 main。你正在 feature 分支上进行开发，并且想要将 feature 分支的提交整合到 main 分支上。

1. 切换到 feature 分支，进行 rebase 操作，将 feature 分支的提交整合到 main 分支上：

   ```bash
   git rebase main
   ```

2. 如果有冲突发生，Git 会停止 rebase 操作并提示你解决冲突。在解决完冲突后，使用 git add 命令将解决后的文件标记为已解决：

   ```bash
   git add <冲突解决文件>
   ```

3. 然后继续 rebase 操作：

   ```bash
   git rebase --continue
   ```

## cherry-pick

从一个分支中挑选一个或多个提交，并将它们应用到当前分支上。这个命令可以让你在不合并整个分支的情况下，将单个提交的更改引入到其他分支。

### 基本语法如下：

```bash
git cherry-pick <提交1的哈希值> <提交2的哈希值> ...
```

4. 如果需要，将修改推送到远程仓库
   ```bash
   git push origin feature
   ```

### 示例：

假设你有两个分支 feature 和 hotfix，并且想要将 hotfix 分支中的某个提交应用到 feature 分支上：

1. 切换到 feature 分支上
2. 使用 git log 命令查看 hotfix 分支上的提交历史，并找到你想要 cherry-pick 的提交的哈希值。

   ```bash
   git log hotfix
   ```

3. 执行 git cherry-pick 命令，将指定的提交应用到当前分支：

   ```bash
   git cherry-pick <提交的哈希值>
   #指定多个提交的哈希值
   git cherry-pick <提交 1 的哈希值> <提交 2 的哈希值>
   ```

4. 如果在 cherry-pick 过程中发生冲突，你需要解决冲突并手动完成 cherry-pick 操作。解决冲突后，使用 git add 命令将解决的文件标记为已解决，并继续 cherry-pick 操作：

   ```bash
   git add <冲突解决的文件>
   git cherry-pick --continue
   ```

5. 如果需要，将修改推送到远程仓库
   ```bash
   git push origin feature
   ```
