# 辅助说明

## 同步更新

1. `git remote -v` 查看是否有上游 `upstream        https://github.com/sysadminsmedia/homebox.git (fetch)`
2. `git fetch upstream main` 拉取上游的更新
3. `git checkout main` 切换到本地分支
4. `git merge upstream/main` 合并上游更新的分支

## 本地打包

### 前端
1. 安装 nodejs 
2. 在目录 `frontend/` 安装模块 `npm i` / `pnpm i` 
3. 打包 `npm run build` / `pnpm run build` 

### 后端
1. 安装 go 
2. 安装 goreleaser 使用命令 `go install github.com/goreleaser/goreleaser/v2@latest` 
3. 在目录 `backend/` 执行本地命令 `goreleaser build --snapshot --clean` 进行打包  

tip: `.goreleaser.yaml` 文件中仅打包 linux 的 amd64 和 arm64 两个版本，如果需要打包完整的版本，可以将 `.goreleaser.yaml.bak` 里面的完整配置复制到 `.goreleaser.yaml` 中进行打包  
