## 同步更新

1. `git remote -v` 查看是否有上游 `upstream        https://github.com/sysadminsmedia/homebox.git (fetch)`
2. `git fetch upstream main` 拉取上游的更新
3. `git checkout main` 切换到本地分支
4. `git merge upstream/main` 合并上游更新的分支