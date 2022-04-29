

```bash
# 版本控制的版本控制，即记录版本的操作记录
git reflog

# 删除历史中的文件，比如之前提交的不需要的大文件
git filter-branch --tree-filter 'rm -f passwords.txt' HEAD

git commit --amend

# 注意-i的使用，可以在rebase时，对commit做一些操作，如合并、修改等
git rebase -i origin/master

# 展示各个分支的情况
git show-branch

# 显示文件每行代码的变更时间
git blame

git bisect
```