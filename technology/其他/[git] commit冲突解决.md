- git 使用总结

1. git log filename 显示某个文件的修改历史
2. git show commit_id filename 显示某次提交对某个文件的修改
3. git log --name-status commit_id 显示某次提交修改的文件列表



- commit冲突的解决
  - 本地commit ahead远程的时候，push 会报错； 所以push之前一定要pull
  - 如果pull的时候冲突，主要是因为本地和远程的修改冲突了；此时有两种情况：
    1. commit后pull显示冲突 -> 手动merge解决冲突 -> 重新commit -> push
    2. stash save(把自己的代码隐藏存起来) -> 重新pull -> stash pop(把存起来的隐藏的代码取回来 ) -> 代码文件会显示冲突 ->解决冲突，add ->commit ->push

