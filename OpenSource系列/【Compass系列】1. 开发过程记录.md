## 1. Github上建立项目

略

## 2. 添加gitflow工作流

```shell
git clone https://github.com/sine-io/compass.git
cd compass
# 执行命令
sine@sine MINGW64 /g/sineio-projects/compass (main)
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
Hooks and filters directory? [G:/sineio-projects/compass/.git/hooks]
(base)
sine@sine MINGW64 /g/sineio-projects/compass (develop)
$

```

```shell
# 添加cobra feature
sine@sine MINGW64 /g/sineio-projects/compass (develop)
$ git flow feature start cobra
Switched to a new branch 'feature/cobra'

Summary of actions:
- A new branch 'feature/cobra' was created, based on 'develop'
- You are now on branch 'feature/cobra'

Now, start committing on your feature. When done, use:

     git flow feature finish cobra

(base)
sine@sine MINGW64 /g/sineio-projects/compass (feature/cobra)
$

```



## 3. 添加cobra



## 目录参考

https://github.com/golang-standards/project-layout/blob/master/README_zh.md