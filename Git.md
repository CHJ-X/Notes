# Git

创建本地仓库

```shell
mkdir learngit
cd learngit
git init
```

如果要添加文件的话，要先把文件拉到作为本地仓库的目录或者子目录
比如说readme.txt

然后

```shell
git add readme.txt
git commit -m"注释/相关信息"
```

关联远程仓库

```shell
git remote add origin git@github.com
```

将本地的东西上传远程库

```shell
git push -u origin master
```

clone到本地

```shell
git clone git@github.com:CHJ-X/repo_name.git
```

如果clone失败可能是因为没有更新
用git pull更新先

添加文件夹

```shell
git add foldername/
```

添加目录内所有某类型type的文件

```shell
git add* .type
```

放弃本地修改
放弃所有修改

```shell
git checkout .
```

放弃某个文件的修改

```shell
git checkout -- filepathname
```
