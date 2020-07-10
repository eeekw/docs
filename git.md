# git

## command line

```bash
### 添加远程仓库
git remote add origin <url>
### 克隆指定分支
git clone -b <branch> <repo>
### 克隆时截取历史commit为指定个数
git clone <repo> --depth 1
### 创建空分支
git checkout --orphan <branch>
### 暂存区与上次commit比较
git diff --staged
### 取消跟踪，可传递文件、目录、file-glob模式
git rm --cached README
### 重命名文件
git mv file_from file_to
### 清除暂存区
git reset HEAD <file>
### 丢弃更改
git checkout -- <file>
```

## git commit

[commitizen/cz-cli](https://github.com/commitizen/cz-cli)

## [git log](https://git-scm.com/book/en/v2/Git-Basics-Viewing-the-Commit-History)

```bash
### 显示每次提交的差异
git log -p -2
### 显示每次提交的统计信息
git log --stat
### 一行显示
git log --pretty=oneline
### 指定格式输出
git log --pretty=format:"%h - %an, %ar : %s"
### 显示合并记录
git log --graph
### 显示最近提交的n个commit
git log -<n>
### 显示指定日期后的commit
git log --since,--after
### 显示指定日期前的commit
git log --until,--before
### 显示指定作者的提交
git log --author
### 显示指定commit message包含的string的提交
git log --grep
### 显示指定代码的提交
git log -s
```

 用于`git log --pretty=format`的格式指定

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



