### 0. 前言

在某个项目开发一段时间之后，我发现分支操作有点复杂，后来进行查询，发现了有git flow，在此做一下使用记录。

### 1. Git Flow 流程图

<img src=".\img\git-flow-01.png" style="zoom: 35%;" />

### 2. 各个分支基本概念

- main（只允许存在一个），当前生产代码所在分支，除项目创建之初提交一次代码之外，不能再对该分支主动提交代码，后续只能通过合并`release`分支的方式，获取代码的变更。一致保持当前最新稳定的发布版本代码，在 CI/CD 建设时，你可以只在该分支做签名、公证等一系列自动化流程
- develop（只允许存在一个），主开发分支。基于`main`分支创建，不能直接向该分支提交代码，只能通过合并`feature`分支的方式得到最新的代码变更。再此基础集成，它名字可能是 release/0.4.7.9，release/0.4.8.0 等。
- feature/\* 可删，新功能分支。 feature分支都是基于develop创建的，开发完成后会合并到`develop`分支上. 项目成员在拿到自己的工作任务之后，就新建一个自己的`feature`分支，功能开发完毕，再将该分支合并到`develop`
- release/* 可删，进入预发布阶段时基于 develop 创建的分支，再此基础集成，它名字可能是 release/2.8.0，release/2.9.0 等。
- bugfix/\* 可删，这种分支一般情况下基于 develop 或者 release/* 分支开出，作为简单的缺陷修复分支，最终合并到 develop 或 release/* 分支中
- hotfix/\* 可删，是对线上最新版本或长期服务版本做紧急修复时使用的分支，他不是常驻的

### 3. 使用

#### 3.1 git flow init

```shell
sine@sine MINGW64 /g/sineio-projects/cosbench-sineio (main)
$ git flow init

Which branch should be used for bringing forth production releases?
   - main
Branch name for production releases: [main]
Branch name for "next release" development: [develop]

How to name your supporting branch prefixes?
Feature branches? [feature/]
Bugfix branches? [bugfix/]
Release branches? [release/]
Hotfix branches? [hotfix/]
Support branches? [support/]
Version tag prefix? []
Hooks and filters directory? [G:/sineio-projects/cosbench-sineio/.git/hooks]
(base)
sine@sine MINGW64 /g/sineio-projects/cosbench-sineio (develop)
$
```

#### 3.2 git flow feature --- 让功能独立拆分

以往的开发我们经常是基于 main 开启一个分支，命名为 dev/* 或者 release/* 等来区分不同版本的迭代，但当迭代节奏加快，团队人员增加，不同的人做不同的功能，都工作在一个分支时互相 rebase 代码的时间会变得非常多，更重要的是在临近发布前一些功能还在出现各种各样的缺陷，影响整个版本的发布。

如果我们能将每个相对独立的功能分开分支开发，在临近发布时将稳定的功能分支合并进发布分支，那些不稳定的功能可以延后至下个迭代中，这非常符合现在敏捷开发的团队需求，刚提到的问题也都很好的解决了。

使用 git-flow 模型可以基于 develop 分支开启一个 feature/* 的分支，来对一个功能进行开发

```shell
# 当我们执行完 git flow init之后，会在develop本地分支下
sine@sine MINGW64 /g/sineio-projects/cosbench-sineio (develop)
$ git branch -a
* develop
  main
  remotes/origin/HEAD -> origin/main
  remotes/origin/main

sine@sine MINGW64 /g/sineio-projects/cosbench-sineio (develop)
$ git flow feature start docker
Switched to a new branch 'feature/docker'

Summary of actions:
- A new branch 'feature/docker' was created, based on 'develop'
- You are now on branch 'feature/docker'

Now, start committing on your feature. When done, use:

     git flow feature finish docker

sine@sine MINGW64 /g/sineio-projects/cosbench-sineio (feature/docker)
$
```

此时已经切换到了feature/docker本地分支下

```shell
sine@sine MINGW64 /g/sineio-projects/cosbench-sineio (feature/docker)
$ git branch -a
  develop
* feature/docker
  main
  remotes/origin/HEAD -> origin/main
  remotes/origin/main
```

当在此分支完成了所有关于 docker 的功能后，并进行了一部分冒烟测试，那么可以使用如下命令将该 feature 合并到 develop 分支。

```shell
# 将修改内容添加并commit
sine@sine MINGW64 /g/sineio-projects/cosbench-sineio (feature/docker)
$ git add .


sine@sine MINGW64 /g/sineio-projects/cosbench-sineio (feature/docker)
$ git commit -m "add docker feature"
[feature/docker d1adec7] add docker feature
 18 files changed, 1131 insertions(+)
 create mode 100644 dev/cosbench-sineio/bin/com/intel/cosbench/api/sio/AsyncSIOStorage.java.bak
 create mode 100644 dev/cosbench-sineio/bin/com/intel/cosbench/api/sio/AsyncSIOStorageFactory.java.bak
 create mode 100644 docker/Dockerfile-alpine
 create mode 100644 docker/Dockerfile-centos
 create mode 100644 docker/Dockerfile-ubuntu
 create mode 100644 docker/README.md
 create mode 100644 docker/build-image.sh
 create mode 100644 docker/examples/docker-compose-alpine.yml
 create mode 100644 docker/examples/docker-compose-centos.yml
 create mode 100644 docker/examples/docker-compose-ubuntu.yml
 create mode 100644 docker/examples/run-controller-alpine.sh
 create mode 100644 docker/examples/run-controller-centos.sh
 create mode 100644 docker/examples/run-controller-ubuntu.sh
 create mode 100644 docker/examples/run-driver-alpine.sh
 create mode 100644 docker/examples/run-driver-centos.sh
 create mode 100644 docker/examples/run-driver-ubuntu.sh
 create mode 100644 docker/push-image.sh
 create mode 100644 docker/start-cosbench.sh

sine@sine MINGW64 /g/sineio-projects/cosbench-sineio (feature/docker)
$ git flow feature finish
Switched to branch 'develop'
Updating e971e17..d1adec7
Fast-forward
 .../cosbench/api/sio/AsyncSIOStorage.java.bak      | 428 +++++++++++++++++++++
 .../api/sio/AsyncSIOStorageFactory.java.bak        |  35 ++
 docker/Dockerfile-alpine                           |  27 ++
 docker/Dockerfile-centos                           |  26 ++
 docker/Dockerfile-ubuntu                           |  25 ++
 docker/README.md                                   | 118 ++++++
 docker/build-image.sh                              |  20 +
 docker/examples/docker-compose-alpine.yml          |  98 +++++
 docker/examples/docker-compose-centos.yml          |  98 +++++
 docker/examples/docker-compose-ubuntu.yml          | 113 ++++++
 docker/examples/run-controller-alpine.sh           |   7 +
 docker/examples/run-controller-centos.sh           |   7 +
 docker/examples/run-controller-ubuntu.sh           |   7 +
 docker/examples/run-driver-alpine.sh               |   8 +
 docker/examples/run-driver-centos.sh               |   8 +
 docker/examples/run-driver-ubuntu.sh               |   8 +
 docker/push-image.sh                               |  11 +
 docker/start-cosbench.sh                           |  87 +++++
 18 files changed, 1131 insertions(+)
 create mode 100644 dev/cosbench-sineio/bin/com/intel/cosbench/api/sio/AsyncSIOStorage.java.bak
 create mode 100644 dev/cosbench-sineio/bin/com/intel/cosbench/api/sio/AsyncSIOStorageFactory.java.bak
 create mode 100644 docker/Dockerfile-alpine
 create mode 100644 docker/Dockerfile-centos
 create mode 100644 docker/Dockerfile-ubuntu
 create mode 100644 docker/README.md
 create mode 100644 docker/build-image.sh
 create mode 100644 docker/examples/docker-compose-alpine.yml
 create mode 100644 docker/examples/docker-compose-centos.yml
 create mode 100644 docker/examples/docker-compose-ubuntu.yml
 create mode 100644 docker/examples/run-controller-alpine.sh
 create mode 100644 docker/examples/run-controller-centos.sh
 create mode 100644 docker/examples/run-controller-ubuntu.sh
 create mode 100644 docker/examples/run-driver-alpine.sh
 create mode 100644 docker/examples/run-driver-centos.sh
 create mode 100644 docker/examples/run-driver-ubuntu.sh
 create mode 100644 docker/push-image.sh
 create mode 100644 docker/start-cosbench.sh
Deleted branch feature/docker (was d1adec7).

Summary of actions:
- The feature branch 'feature/docker' was merged into 'develop'
- Feature branch 'feature/docker' has been locally deleted
- You are now on branch 'develop'

# 已切换到develop分支
sine@sine MINGW64 /g/sineio-projects/cosbench-sineio (develop)

```

执行以上命令后，feature/docker 分支会自动合并到 develop 分支，并且将自动删除临时的 feature 分支。他类似与你自己执行了如下的 git 原生命令：

```shell
# 当前在 develop 分支
git branch -b feature/docker

# 编写业务功能代码后一系列提交
git commit -a -s -m "feature commit message"

# 完成功能
git checkout develop
git merge --no-ff feature/docker
git branch -d feature/docker
```

如果您对主仓库的 develop 有推送权限，那么可以直接推送代码到仓库中:

```shell
git push origin develop
```

但某些场景下，团队是需要 code review 的，您可以将 feature/* 分支推送到你 fork 的子仓库中，并以 Pull Request 或 Merge Request 的方式提交到主仓库的 develop 分支进行 code review。当然 code review 的方式有很多种，我们可以根据自己团队的需求来做一些改变。

#### 3.3 git flow release --- 让版本发布自动化

当进入一个发布窗口期，我们需要考量一下哪些功能可以在准备发布的版本进行发布了，这些功能首先会被合并到 develop 分支，这里避免不了会有一些代码的冲突，需要指定功能的负责人进行合并，在确认无误后我们开启一个准备发布的分支：

```shell
# 在develop分支下执行 git flow release start 0.4.7.9

sine@sine MINGW64 /g/sineio-projects/cosbench-sineio (develop)
$ git flow release start 0.4.7.9
Switched to a new branch 'release/0.4.7.9'

Summary of actions:
- A new branch 'release/0.4.7.9' was created, based on 'develop'
- You are now on branch 'release/0.4.7.9'

Follow-up actions:
- Bump the version number now!
- Start committing last-minute fixes in preparing your release
- When done, run:

     git flow release finish '0.4.7.9'

sine@sine MINGW64 /g/sineio-projects/cosbench-sineio (release/0.4.7.9)
$
```

执行以上命令后，我们就基于 develop 分支开启了一个 release/0.4.7.9 的分支，你可以使用该分支做整体预发布功能的回归、做一些版本号修改、文档更正等简单的工作。这个分支不在进行大规模的代码调整，仅做一些回归时发现的小缺陷修复，这个周期通常要 1~2 天的时间。当确认该分支代码稳定可以发布时，执行如下命令进行发布：

```shell
# 当前在 release/0.4.7.9 分支执行 git flow release finish

sine@sine MINGW64 /g/sineio-projects/cosbench-sineio (release/0.4.7.9)
$ git flow release finish '0.4.7.9'
Switched to branch 'main'
Your branch is up to date with 'origin/main'.
Merge made by the 'ort' strategy.
 dev/cosbench-sineio/bin/.gitignore | 1 -
 1 file changed, 1 deletion(-)
 delete mode 100644 dev/cosbench-sineio/bin/.gitignore
Already on 'main'
Your branch is ahead of 'origin/main' by 2 commits.
  (use "git push" to publish your local commits)
Switched to branch 'develop'
Merge made by the 'ort' strategy.
Deleted branch release/0.4.7.9 (was fdc13f8).

Summary of actions:
- Release branch 'release/0.4.7.9' has been merged into 'main'
- The release was tagged '0.4.7.9'
- Release tag '0.4.7.9' has been back-merged into 'develop'
- Release branch 'release/0.4.7.9' has been locally deleted
- You are now on branch 'develop'

sine@sine MINGW64 /g/sineio-projects/cosbench-sineio (develop)
```

该命令执行了如下几个操作：

- 合并 release/0.4.7.9 到 main 分支
- 创建 0.4.7.9 tag
- 合并 0.4.7.9 tag 到 develop 分支

整个 release 版本发布的阶段，转化为 git 的原生指令他是这样一个步骤：

```shell
# 当前工作与 develop 分支
git checkout -b release/0.4.7.9

# 做一些简单修复或者版本更改，提交代码
git commit -a -s -m "Update version..."

# 完成版本发布
git checkout main
git merge --no-ff release/0.4.7.9
git tag -a 0.4.7.9
git checkout develop
git merge --no-ff 0.4.7.9
git branch -d release/0.4.7.9
```

整个发布过程的指令就是这些，看似简单，但是当我们自己操作时很难不出错误，特别是版本发布和线上缺陷（hotfix）修复同时进行的时候，如果有这些辅助指令可以大大加快我们的工作效率且不容易出错。

在 release 阶段你可以让你的自动构建系统配合 git-flow 的固有规则进行构建，比如我们可以根据分支名称或者当前最新 tag 自动给产出物做版本号的修改，而不需要我们在代码中新建一个提交去修改这些内容。

另外我更建议在 release/* 分支做版本发布，而不是 main 分支仅作为一个备份分支，它在一些开源项目中显得比较重要，因为他代表着最新稳定分支。但公司内部的一些开发团队中来看 main 分支可有、可无。但是你不能在 main 分支随便产生一个提交，这样会打乱 git flow 的工作流程，你要来来回回合并好几次才能保证各个协作分支正常工作。

之所以建议在 release/* 分支做发布操作，是因为有些时候在你执行完 `git flow release finish` 后还会发现有一些非常简单的错误需要修复，比如对外文档中的一个符号错误、一个错别字、甚至版本号错误等，这些都是避免不了的。如果你已经合并到 main 分支，你需要基于 develop 重来一遍发布流程，这会产生非常多的麻烦。何不在 release 分支就都完成了呢？当产品上线、部署到官网后，下一个迭代开始前，在执行 `git flow release finish` 更好。

#### 3.4 git flow hotfix --- 线上缺陷紧急修复

谁都不愿意看到线上出现紧急问题，出问题不要怕，解决它并告诉自己不要再犯同样的错误，这也是我为什么使用 git flow 一个很重要的原因。以往的线上紧急问题修复中，我们通常是基于 main 或者最新 tag 拉取一个分支，在这个分支中做缺陷的紧急修复，分支名称比较随意，有时带版本、有时不带版本，不同人的做法不一导致这个流程出现很多问题。在紧急问题修复后我们要把这些修复的问题合并回 main，但同时我们需要将这个修复合并到我们正在开发或者准备发布的分支中，这一步是经常容易忘记的，无论你是新来的同事还是老同学都可能在这里犯错。

而使用 git-flow 则可以非常简单的避免这些问题，它有非常完善的 hotfix 流程，确保你在修复问题时不影响常规迭代，当线上发生紧急问题时，你需要基于 main 分支执行如下命令：

```shell
git flow hotfix start 0.4.7.10
```

该命令会基于 main 创建一个 hotfix/0.4.7.10 的分支，在进行一系列缺陷修复并通过测试后，使用如下命令完成这个紧急修复：

```shell
git flow hotfix finish
```

git-flow 命令行工具会自动根据当前分支获取要使用的版本号，它将执行如下功能：

- 将修复合并到 main 分支确保主干为最新得到修复的内容
- 新建 0.4.7.10 的 tag
- 将修复同时合并到 develop 分支，确保当前开发分支也同样得到修复而不是被遗忘
- 删除临时的 hotfix 分支

两条命令帮助我们做了非常多我们容易忘记的事情，同时版本号的管理也更加严谨不会轻易让我们出错，自动根据版本号创建 tag 也让我们的紧急修复可以被追溯。上面简单的两条命令还原为 git 的原生命令是如下代码：

```shell
# 基于 main 开启分支
git checkout -b hotfix/0.4.7.10

# 修复内容
git commit -a -s -m "Fixed something"

# 合并修复到主干和开发分支
git checkout main
git merge --no-ff hotfix/0.4.7.10
git tag -a 0.4.7.10
git checkout develop
git merge --no-ff 0.4.7.10 # merge tag
git branch -d hotfix/0.4.7.10
```

机器代码最容易解决的就是流程上的问题，但最难解决的是变更。当你在准备下一个 release 版本比如 release/0.4.7.11 时，此时线上又出现了紧急的缺陷待修复必须马上发版本解决。团队决定又不准备做版本回滚，那么就要有一些变更了。我们需要在完成修复代码后将修复内容合并到 release/0.4.7.11 分支，而不是 develop 分支，因为在 release/0.4.7.11 完成后会自动合并到 develop，确保我们的代码不会被丢失。这些场景我们的确有碰到。但我更建议在流程上避免这些事情，release 分支的新建代表下一个迭代版本即将发布，如不是非常紧急的问题可以等待新版本发布，否则可能对现有发布流程产生影响。

#### 3.5 git flow support --- 长期服务分支维护

私有化版本在我们的团队中是“家常便饭”，这些私有化版本常常无法与主版本代码保持一致，包括 hotfix 也无法覆盖到这些版本中。通常的情况是我们最新的版本已经发布到 0.5.8.11 版本，但外部还有使用 0.4.7.0 或 0.5.7.0 版本的客户，他们因为业务稳定性的要求，很难升级 SDK 至最新版本，你不得不把一些主版本已经修复的问题单独合并到这些长期维护分支中，它很像 Linux 的 LTS 版本。

在 git-flow 模型中，使用 `support/` 前缀来管理这些长期维护版本分支，当我们确定某个版本的代码是需要长期维护的，并且客户在这个版本中提到了一些已知问题，我们需要对这些问题进行修复时，首先基于该版本开启一个 support 分支。如下：

```shell
git flow support start 0.4.7.x 0.4.7.0
```

以上命令是基于 0.4.7.0 的 tag 开启了一个新的分支 support/0.4.7.x 分支，这个分支就长期存在了，你不能删除它，它的级别与 main、develop 是一样重要的，比 release、hotfix 等更重要。因为他保存了这个长期支持分支的所有修复内容。接下来我们基于这个长期服务分支进行问题修复：

```shell
git flow hotfix start 0.4.7.1 support/0.4.7.x
```

此命令代表我们要基于 `support/0.4.7.x` 分支开启一个 hotfix 修复，修复后的版本号我们定在 0.4.7.1，在新的 hotfix 分支上我们进行问题代码修复，修复完成后执行：

```shell
git flow hotfix finsih '0.4.7.1'
```

执行此命令后会有如下几个操作：

- 合并 hotfix/0.4.7.1 到 support/0.4.7.x 分支
- 新建 tag 0.4.7.1
- 删除 hotifx/0.4.7.1 的分支

这样基于 support/0.4.7.x 分支开启的所有修复都会合并回该分支中，它一直保持最新。在团队协作过程中，hotfix/* 分支开启后，需要在该分支中提测和测试，在确保无误后再合并到 support/* 分支确保。

我们来总结一下 git flow support 拆分成单独的 git 原生指令是如何工作的：

```shell
# 新建分支
git checkout 0.4.7.0
git branch -b support/0.4.7.x

# 基于 support/0.4.7.x 新建 hotfix
git checkout -b hotfix/0.4.7.x

# 解决问题并合并修复
git commit -a -s -m "fix: somthing"
git checkout support/0.4.7.x
git merge --no-ff hotfix/0.4.7.1
git tag 0.4.7.1
git branch -D hotfix/0.4.7.1
```

#### 3.6 推送分支到远端

```shell
# 3.2~3.5均在本地分支操作，当我们执行完后，需要推送需要的分支到远端（比如develop分支）
git push origin develop
```

### 4. 总结

git-flow 模型和工具链给我们团队协作带来很大的方便，但是它有学习成本，即使是最简单的几条命令很多人也不愿意去理解它。现有的一些 GUI 管理工具如 SourceTree 则通过界面交互的方式让开发者少敲一些命令相信学习成本会有一些降低。但过度依赖 GUI 工具或现有 git-flow 工具链的命令并不是什么好事儿，容易变成“教条”或者“真理”让团队生厌。理解它的原理，当有一天你可以不依赖 git-flow 工具链能完整的做一个开发、发布、修复、支持等流程时，才算真的理解 git-flow。

### 5. 致谢

本文大部分基于[链接](https://cloud.tencent.com/developer/article/1800951)进行的实践，侵删联系，在此表示感谢。