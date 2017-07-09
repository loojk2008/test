## 分支开发

阅读前假设用户配置了
```
➜  test git:(feature-test-project) cat ~/.gitconfig
[alias]
        st = status
        ci = commit
        co = checkout
        br = branch
        df = diff
```

### **多人多分支同步开发**

假设线上有一个master分支，一个develop分支,两个分支一般都不直接拿来当作开发分支。
现在用户a,b需要协作开发一个独立功能，用户a分支管理者，用户b分支协同开发者：

#### 1： 建立开发分支
首先a用户,更新自己本地缓冲区，更新本地develop分支，从develop分支建立一个开发分支feature-test-project，
推送到远程中心服务器。

```
➜  test git:(develop) git fetch --all
Fetching origin

➜  test git:(develop) git pull origin develop  
From github.com:loojk2008/test
 * branch            develop    -> FETCH_HEAD

➜  test git:(develop) git co -b feature-test-project
Switched to a new branch 'feature-test-project'

➜  test git:(feature-test-project) git push origin feature-test-project  
Counting objects: 3, done.
Delta compression using up to 4 threads.
Compressing objects: 100% (3/3), done.
Writing objects: 100% (3/3), 421 bytes | 0 bytes/s, done.
Total 3 (delta 0), reused 0 (delta 0)
To github.com:loojk2008/test.git
 * [new branch]      feature-test-project -> feature-test-project

```

#### 2：协同开发者签出分支
b用户作为协同开发者，更新本地缓冲区。

```
➜  test git:(develop) git fetch --all
Fetching origin

```

使用git br -a可以看到a用户刚才建立的开发分支 feature-test-project

```
➜  test git:(develop) git br -a
* develop
  master
  remotes/origin/HEAD -> origin/master
  remotes/origin/develop
  remotes/origin/feature-test-project
  remotes/origin/master
```

现在要做的就是把远程开发分支签出到本地 -t 代表追踪远程分支

```
➜  test git:(develop) git co -t origin/feature-test-project
Branch feature-test-project set up to track remote branch feature-test-project from origin.
Switched to a new branch 'feature-test-project'
➜  test git:(feature-test-project) 
```

到这步a,b用户本地都有了一个用来开发的独立分支。

#### 3：分支请求合并

情景假设：a,b完成了分支功能开发

a用户更新缓冲区发现，已经有人先一步合并了一个分支到develop分支
```
➜  test git:(develop) git fetch --all
Fetching origin
remote: Counting objects: 6, done.
remote: Compressing objects: 100% (3/3), done.
remote: Total 6 (delta 1), reused 4 (delta 1), pack-reused 0
Unpacking objects: 100% (6/6), done.
From github.com:loojk2008/test
   e656bb0..288a8c3  develop    -> origin/develop

```

#### 代码rebase(衍合)
a用户需要在合并到develop分支前进行rebase操作

```
➜  test git:(feature-test-project) git fetch --all
Fetching origin
➜  test git:(feature-test-project) git pull --rebase origin develop  
From github.com:loojk2008/test
 * branch            develop    -> FETCH_HEAD
First, rewinding head to replay your work on top of it...
Applying: 分支开发
Applying: 协同开发分支建立
Applying: c user
Applying: 增加衍合操作

```
这里实验性的衍合代码并没有发生冲突，实际使用中代码冲突需要在分支管理者自己解决。
完成衍合之后，develop分支里最新的代码就合并到a用户本地feature-test-project分支，下一步就可以进行合并请求操作了。
我们需要做的就是

```
➜  test git:(feature-test-project) git push origin feature-test-project  -f
Counting objects: 18, done.
Delta compression using up to 4 threads.
Compressing objects: 100% (18/18), done.
Writing objects: 100% (18/18), 2.82 KiB | 0 bytes/s, done.
Total 18 (delta 11), reused 0 (delta 0)
remote: Resolving deltas: 100% (11/11), completed with 1 local object.
To github.com:loojk2008/test.git
 + b01e3a1...e1a16d7 feature-test-project -> feature-test-project (forced update)

```

note: -f 参数代码强制更新，注意两点

1：-f参数不是必须的，但是当衍合操作发生合并，冲突时，feature-test-project 和远程／origin／feature-test-project会发生偏离
不加-f参数，本地衍合后的代码是不能推送到远程对应的远程分支。

2：a进行衍合操作后，更新了远程origin／feature-test-project，b用户不应该再提交代码到feature-test-project分支。
（如果非要提交需要可以进行git pull --rebase origin/feature-test-project 或者删除本地feature-test-project分支，
重新git co -t origin/feature-test-project一个本地分支）


#### 代码merge(合并)
如果遵循上面步骤操作，这一步就简单了，直接merge到develop分支就可以了，不会有冲突了。


### **多个独立分支代码依赖解决**

假设有从develop分支签出的分支feature-a  feature-b

feature-a 分支需要依赖feature-b分支代码，可以执行

```
git fetch --all
git pull --rebase origin/feature-b
```

feature-a 就包含了 feature-b 代码，两个分支可以继续保持独立开发，或者进行合并。



### **cherry-pick使用**
一般应用场景：
假设有分支feature-a  feature-b
你在feature-b分支开发的时候，这个功能不需要了。你马上又要投入feature-a功能的开发，但是feature-b里修改的东西，feature-a里用的到。
可以进行下列操作，开发的时候建议单独文件的单独功能修改做一次commit.

```
➜  test git:(feature-b) touch b.txt
➜  test git:(feature-b) ✗ git add b.txt 
➜  test git:(feature-b) ✗ git commit -m '增加'

➜  test git:(feature-b) git log

commit 64f40c5c1b1b900a1cb0aba42efbc208049a6856 (HEAD -> feature-b)
Author: loojk <378504632@qq.com>
Date:   Sun Jul 9 12:58:39 2017 +0800

    增加

commit e656bb03d482950df778e2b89d2376002693fad9 (origin/master, origin/develop, origin/HEAD, master, feature-a, develop)
Author: loojk <378504632@qq.com>
Date:   Sun Jul 9 09:34:18 2017 +0800

    add .gitignore

commit b00fdbc5c2178327dede5cc9f59826239d9a66ce
Author: jinkuiluo <378504632@qq.com>
Date:   Wed Mar 23 15:58:52 2016 +0800

    Initial commit
(END)
```

通过查看log日志可以知道提交id e656bb03d482950df778e2b89d2376002693fad9

```
➜  test git:(feature-a) git cherry-pick 64f40c5c1b1b900a1cb0aba42efbc208049a6856
[feature-a 31941ec] 增加
 Date: Sun Jul 9 12:58:39 2017 +0800
 1 file changed, 0 insertions(+), 0 deletions(-)
 create mode 100644 b.txt
```

可以查看这次commit会从feature-b分支，被重新提交到feature-a分支里面，查看日志这会是一次新的提交
 
```
➜  test git:(feature-a) git log


commit b01e3a10ba1f53fa8c226a56f38b1e6d0cf67b64 (HEAD -> feature-test-project, origin/feature-test-project)
Author: loojk <378504632@qq.com>
Date:   Sun Jul 9 10:04:48 2017 +0800

    分支开发

commit e656bb03d482950df778e2b89d2376002693fad9 (origin/master, origin/develop, origin/HEAD, master, develop)
Author: loojk <378504632@qq.com>
Date:   Sun Jul 9 09:34:18 2017 +0800

    add .gitignore

commit b00fdbc5c2178327dede5cc9f59826239d9a66ce
Author: jinkuiluo <378504632@qq.com>
Date:   Wed Mar 23 15:58:52 2016 +0800

    Initial commit
(END)

```

 
