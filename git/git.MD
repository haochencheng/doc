![git](https://static.oschina.net/uploads/space/2017/0910/101257_bfXs_2945455.jpg)

### git 初始化

```
$ git config user.name 'github用户名'
$ git config user.email '邮箱'
```

### git 提交

```
git add . 
git commit - m "提交备注"
git push
```

### 更新代码

```
git pull #更新远程代码到本地自动合并

git fetch #更新远程代码带本地仓库

```

### git 添加ssh

```
$ ssh-keygen -t rsa -C "your_email@example.com"

```
代码参数含义：

-t 指定密钥类型，默认是 rsa ，可以省略。
-C 设置注释文字，比如邮箱。
-f 指定密钥文件存储文件名。

以上代码省略了 -f 参数，因此，运行上面那条命令后会让你输入一个文件名，用于保存刚才生成的 SSH key 代码

### git删除远程文件
预览将要删除的文件

```
git rm -r -n --cached  文件/文件夹名称 
```

加上 -n 这个参数，执行命令时，是不会删除任何文件，而是展示此命令要删除的文件列表预览。
确定无误后删除文件


```
git rm -r --cached 文件/文件夹
```

提交到本地并推送到远程服务器


```
git commit -m "提交说明"
git push origin master
```

修改本地 .gitignore 文件 并提交


```
git commit -m "提交说明"
git push origin master
```

### git 查看最近分支
```
git describe --tags `git rev-list --tags --max-count=1`
```

