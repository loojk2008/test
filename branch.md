## 分支开发


### 多人多分支同步开发

假设线上有一个master分支，一个develop分支,两个分支一般都不直接拿来当作开发分支。
现在用户a,b需要协作开发一个独立功能，用户a分支管理者，用户b分支协同开发者：

#### 建立开发分支
首先a用户,更新自己本地缓冲区，更新本地develop分支，从develop分支建立一个开发分支feature-test-project，
推送到远程中心服务器。

```
➜  test git:(develop) git fetch --all
Fetching origin

➜  test git:(develop) git pull origin develop  
From github.com:loojk2008/test
 * branch            develop    -> FETCH_HEAD

➜  test git:(master) git co -b feature-test-project
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

#### 协同开发者签出分支
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

到这步a,b用户本地都用了一个用来开发的独立分支。
