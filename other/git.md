# Git

```bash
git tag # 显示所有 tag
git checkout <tag> # 切换到 tag 所处的 commitId
git checkout -b <branch> <tag> # 切换到 tag，并创建分支

# 仅克隆某个 commitId 的内容
git clone --single-branch --depth 1 --branch v5.6.3 {url}
```

## Config

```bash
# 做此配置，输入一次密码后，再也不用输入密码
git config credential.helper store
```

## Stash

```bash
git stash # 暂存（存储在本地，并将项目本次操作还原）
git stash list # 查看所有的暂存
git stash pop # 使用上一次暂存，并将这个暂存删除
git stash apply # 使用某个暂存，但是不会删除这个暂存
git stash clear # 清空所有的暂存
```

## Remote

```bash
git remote -v # 查看 remote 详情

git remote set-url origin [url] # 重新设置 remote 的 url

git remote rm origin
git remote add origin [url]
```

