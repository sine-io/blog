### 0. 前言

工作中git最常用的命令就是git add、git commit、git push:joy:。最近又接触了一些使用方法，在这里根据不同的使用场景做个记录。

### 1. 场景

#### 1.1 分支合并

```shell
# develop合并到main为例：
# 切换到main分支
sine@sine MINGW64 /g/sineio-projects/cosbench-sineio (develop)
$ git checkout main
Switched to branch 'main'
Your branch is up to date with 'origin/main'.

# 如果有他人共同编码，拉取代码
sine@sine MINGW64 /g/sineio-projects/cosbench-sineio (main)
$ git pull origin main
From https://github.com/sine-io/cosbench-sineio
 * branch            main       -> FETCH_HEAD
Already up to date.

# 合并
sine@sine MINGW64 /g/sineio-projects/cosbench-sineio (main)
$ git merge develop
Updating 3269d66..d1adec7
Fast-forward
 .../cosbench/api/sio/AsyncSIOStorage.java.bak      | 428 +++++++++++++++++++++
 ...
 docker/start-cosbench.sh                           |  87 +++++
 18 files changed, 1131 insertions(+)
 create mode 100644 dev/cosbench-sineio/bin/com/intel/cosbench/api/sio/AsyncSIOStorage.java.bak
 ...
 create mode 100644 docker/start-cosbench.sh

# 查看（按需）
sine@sine MINGW64 /g/sineio-projects/cosbench-sineio (main)
$ git status
On branch main
Your branch is ahead of 'origin/main' by 2 commits.
  (use "git push" to publish your local commits)

nothing to commit, working tree clean

# 推送到远端
sine@sine MINGW64 /g/sineio-projects/cosbench-sineio (main)
$ git push origin main
Total 0 (delta 0), reused 0 (delta 0), pack-reused 0
To https://github.com/sine-io/cosbench-sineio.git
   3269d66..d1adec7  main -> main
```

#### 1.2 tag操作

##### 1.2.1 列出

```shell
# 列出本地tag
sine@sine MINGW64 /g/sineio-projects/cosbench-sineio (develop)
$ git tag -l
0.4.7.5
0.4.7.6
0.4.7.7
0.4.7.8
0.4.7.9

# 列出远端tag
sine@sine MINGW64 /g/sineio-projects/cosbench-sineio (develop)
$ git ls-remote --tags origin
2bc3ae97c9bcc21c76a4a828279aab10265f824e        refs/tags/0.4.7.6
3f47cd6c34a0c84885695da40d4cf344a564d6a4        refs/tags/0.4.7.7
414ebb25a8e4e855329956b1dd34caac22962631        refs/tags/0.4.7.8
d1adec791cc0af18cdb68e5efeb2080f65fb0985        refs/tags/0.4.7.9
```

##### 1.2.2 删除

```shell
# 删除本地tag
sine@sine MINGW64 /g/sineio-projects/cosbench-sineio (develop)
$ git tag -d 0.4.7.5
Deleted tag '0.4.7.5' (was 983754d)

# 删除远端tag --- 待测试
git push origin :refs/tags/0.4.7.9

```

##### 1.2.3 创建 --- 待测试

```shell
# 创建本地tag
git tag 0.4.7.9

# 创建tag时添加comment
git tag -a 0.4.7.9 -m "xxx"

# 推送到远端
git push origin :0.4.7.9

# 推送所有tag到远端
git push origin --tags
```

```shell
# 基于某个commit创建tag

# 1. 查看当前分支的提交历史 里面包含 commit id
sine@sine MINGW64 /g/sineio-projects/cosbench-sineio (develop)
$ git log --pretty=oneline
d1adec791cc0af18cdb68e5efeb2080f65fb0985 (HEAD -> develop, origin/main, origin/develop, origin/HEAD, main) add docker feature
...
e1a640b5dc6c227b215fd62b1bbf19c37b6c4666 Merge pull request #23 from sine-io/develop


# 2. 创建tag --- 待测试
git tag -a <tagName> <commitId>

```

##### 1.2.4 查看

```shell
# 查看本地某个 tag 的详细信息
sine@sine MINGW64 /g/sineio-projects/cosbench-sineio (develop)
$ git show 0.4.7.9
tag 0.4.7.9
Tagger: sineio <sinecelia.wang@gmail.com>
Date:   Mon Oct 17 11:44:26 2022 +0800

release 0.4.7.9

commit 3269d660c92d2db4ce6330911ab5f10f790a9d87 (tag: 0.4.7.9)
Merge: 4688524 fdc13f8
Author: sineio <sinecelia.wang@gmail.com>
Date:   Mon Oct 17 11:43:58 2022 +0800

    Merge branch 'release/0.4.7.9'
    merge

```

```shell
# 查看所有本地tag
sine@sine MINGW64 /g/sineio-projects/cosbench-sineio (develop)
$ git tag
0.4.7.6
0.4.7.7
0.4.7.8
0.4.7.9

sine@sine MINGW64 /g/sineio-projects/cosbench-sineio (develop)
$ git tag -l
0.4.7.6
0.4.7.7
0.4.7.8
0.4.7.9
```

```shell
# 查看所有远程tag
sine@sine MINGW64 /g/sineio-projects/cosbench-sineio (develop)
$ git ls-remote --tags origin
2bc3ae97c9bcc21c76a4a828279aab10265f824e        refs/tags/0.4.7.6
3f47cd6c34a0c84885695da40d4cf344a564d6a4        refs/tags/0.4.7.7
414ebb25a8e4e855329956b1dd34caac22962631        refs/tags/0.4.7.8
d1adec791cc0af18cdb68e5efeb2080f65fb0985        refs/tags/0.4.7.9

```

