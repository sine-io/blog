### 0. 前言

近期有了创建个人博客的需求，进行了一番调研之后，由于我是go语言爱好者，所以我选择了hugo，下面就博客的搭建过程进行一个完整的记录，供参考。

### 1. 安装hugo

#### 1.1 Windows

我的办公电脑为Windows 10，由于我工作中经常接触Linux系统，所以我也想在Win 10上拥有像Linux一样的包管理工具，经过搜索，我选定了scoop这个工具，使用了一段时间后，还是很香的。

##### 1.1.1 安装scoop

> 前提：电脑可以访问外网
>
> 由于莫名的原因，可能会失败，请多尝试几次

```powershell
# 打开powershell
Set-ExecutionPolicy RemoteSigned -Scope CurrentUser
# 输入Y 后回车

irm get.scoop.sh | iex

# 测试
scoop help
```

##### 1.1.2 使用scoop安装hugo

```powershell
# 打开scoop官网，进行package搜索
https://scoop.sh/#/
```

安装命令如下图所示：

> 注：由于我选定的主题（Stack）需要hugo-extended版本，所以我安装的红框版本。
>
> hugo主题选择网站：https://themes.gohugo.io/

![](.\img\install-hugo-01.png)

```powershell
# 在cmd里执行这两条命令即可

scoop bucket add main
scoop install hugo-extended

# 注：网络原因，请耐心等待或多次尝试。

# 安装完毕后，可以进行一下测试
hugo version
```

### 2. 博客建设

#### 2.1 创建github项目

登录github，创建一个仓库即可，这里不再赘述。

#### 2.2 使用hugo搭建

```powershell
# 下载
git clone https://github.com/sine-io/sineio-docs.git
# 初始化
hugo new site sineio-docs --force

# 为了后面管理和定制化主题方便，咱们将主题fork一份到自己的仓库
# 我fork后主题的地址为：https://github.com/sine-io/hugo-theme-stack.git

# 配置主题，由于不可抗力会经常下载失败，请想个办法多试几次，不要放弃...
cd sineio-docs
git submodule add https://github.com/sine-io/hugo-theme-stack/ themes/hugo-theme-stack


```

