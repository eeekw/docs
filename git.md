# git

## command line

```bash
### 添加远程仓库
git remote add origin <url>
### 克隆指定分支
git clone -b <branch> <repo>
### 创建空分支
git checkout --orphan <branch>
### 暂存区与上次commit比较
git diff --staged
### 取消跟踪，可传递文件、目录、file-glob模式
git rm --cached README
### 重命名文件
git mv file_from file_to
```

## git commit

[commitizen/cz-cli](https://github.com/commitizen/cz-cli)

## [git log](https://git-scm.com/book/en/v2/Git-Basics-Viewing-the-Commit-History)

```text
### 显示每次提交的差异
git log -p (-2)
### 显示每次提交的统计信息
git log --stat
### 一行显示
git log --pretty=oneline
### 指定格式输出
git log --pretty=format:"%h - %an, %ar : %s"
### 显示合并记录
git log --graph
```



| Specifier | Description of Output |
| :--- | :--- |
| %H | Commit hash |
| %h | Abbreviated commit hash |
| %T | Tree hash |
| %t | Abbreviated tree hash |
| %P | Parent hashes |
| %p | Abbreviated parent hashes |
| %an | Author name |
| %ae | Author email |
| %ad | Author date \(format respects the --date=option\) |
| %ar | Author date, relative |
| %cn | Committer name |
| %ce | Committer email |
| %cd | Committer date |
| %cr | Committer date, relative |
| %s | Subject |



