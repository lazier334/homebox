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

## 本地构建docker镜像
> 基础目录列表如下:
> * homebox
> * Dockerfile
> * docker-compose.yml

### 1. 创建文件 Dockerfile
> 创建 Dockerfile 文件并写入以下内容

```Dockerfile
FROM debian:stable-slim
RUN mkdir -p /data
COPY homebox /app/api
RUN chmod +x /app/api
WORKDIR /app
EXPOSE 7745
ENTRYPOINT ["/app/api"]
CMD ["/data/config.yml"]
```
### 2. 创建文件 docker-compose.yml
> 创建 docker-compose.yml 文件并写入以下内容

```yml
version: '3.8'

services:
  api:
    build: .
    container_name: homebox
    restart: unless-stopped
    ports:
      - "7745:7745"
    volumes:
      - ./data:/data
    environment:
      - HBOX_LOG_LEVEL=info
      - HBOX_LOG_FORMAT=text
      - HBOX_WEB_MAX_FILE_UPLOAD=100
      - HBOX_OPTIONS_ALLOW_ANALYTICS=false
      - PATH=/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin
      - HBOX_MODE=production
      - HBOX_STORAGE_CONN_STRING=file:///?no_tmp_dir=true
      - HBOX_STORAGE_PREFIX_PATH=data
      - HBOX_DATABASE_SQLITE_PATH=/data/homebox.db?_pragma=busy_timeout=2000&_pragma=journal_mode=WAL&_fk=1&_time_format=sqlite
    labels:
      - "Name=homebox"
      - "Version=1.0.1"
```

### 3. 构建镜像 docker-compose up -d
> 使用命令构建并启动镜像

```shell
docker-compose up -d
```