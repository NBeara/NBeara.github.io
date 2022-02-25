# 使用hugo+github pages+github action实现自动部署博客






## 前情提要：

本文所用到的工具以及相关插件：

- 静态博客搭建框架： Hugo_extended0.92.2.win64
- hugo主题：FeelIt
- 博客评论系统：Valine
- Github pages+github action实现自动部署
- 版本控制工具:git



## hugo部分

### 安装hugo

1. 下载hugo_extend_0.92.2_windows.64bits.zip
	<a href="https://github.com/gohugoio/hugo/releases">下载页面</a>

2. 将压缩包解压至以bin结尾的目录下：

	比如 D:\hugo\bin

3. 添加环境变量
	将D:\hugo\bin添加至环境变量中

4. 验证是否完成
	cmd命令行输入`hugo version`,如果成功安装会显示版本号



### 创建自己的站点

`hugo new site <example>`

`cd <example>`

*项目名<example>处随便写*

### 安装FeelIt主题

#### 三种方法：

1. 下载<a href="https://github.com/khusika/FeelIt/releases">压缩包</a>并解压至`theme`目录

2. 使用`git clone https://github.com/khusika/FeelIt.git themes/FeelIt`克隆到`theme`目录

3. 将主题仓库作为网站目录的子模块

	确保当前目录是站点根目录<example>

	`git init`
	`git submodule add https://github.com/khusika/FeelIt.git themes/FeelIt`



#### 基础配置：

- 打开`yourSites/config.toml`

	- 复制以下内容：

		```markdown
		baseURL = "http://example.org/"
		# [en, zh-cn, fr, ...] 设置默认的语言
		defaultContentLanguage = "zh-cn"
		# 网站语言, 仅在这里 CN 大写
		languageCode = "zh-CN"
		# 是否包括中日韩文字
		hasCJKLanguage = true
		# 网站标题
		title = "我的全新 Hugo 网站"
		
		# 更改使用 Hugo 构建网站时使用的默认主题
		theme = "FeelIt"
		
		[params]
		  # FeelIt 主题版本
		  version = "1.0.X"
		
		[menu]
		  [[menu.main]]
		    identifier = "posts"
		    # 你可以在名称 (允许 HTML 格式) 之前添加其他信息, 例如图标
		    pre = ""
		    # 你可以在名称 (允许 HTML 格式) 之后添加其他信息, 例如图标
		    post = ""
		    name = "文章"
		    url = "/posts/"
		    # 当你将鼠标悬停在此菜单链接上时, 将显示的标题
		    title = ""
		    weight = 1
		  [[menu.main]]
		    identifier = "tags"
		    pre = ""
		    post = ""
		    name = "标签"
		    url = "/tags/"
		    title = ""
		    weight = 2
		  [[menu.main]]
		    identifier = "categories"
		    pre = ""
		    post = ""
		    name = "分类"
		    url = "/categories/"
		    title = ""
		    weight = 3
		
		# Hugo 解析文档的配置
		[markup]
		  # 语法高亮设置 (https://gohugo.io/content-management/syntax-highlighting)
		  [markup.highlight]
		    # false 是必要的设置 (https://github.com/khusika/FeelIt/issues/158)
		    noClasses = false
		```

		

### 创建一篇文章

`hugo new posts/my-first-post.md`

这篇文章生成在站点根目录下`/contents`文件夹中



### 启动hugo

`hugo server -D`

打开浏览器输入http://localhost:1313/



如果出现found no layout file for "HTML" for kind "home"的错误，就是没有加载主题，导致没有模板进行创建



到此第一篇博客的生成已经完成，接下来将博客推送到github pages上。



## 准备自动部署



我用的是github pages+github action的方式实现的自动部署，用的两个仓库，一个私密仓库存放hugo根目录文件，一个公开仓库只用来存放public的内容也就是在github pages上显示需要的博客部分。



### Git链接远程仓库

1.创建SSH key

`ssh-keygen -t rsa -C "example@email.com" `
		后面的邮箱可以填写任意邮箱，与github账号邮箱不一致也是可以的



提示输入passphrase时直接回车，不需要这个



生成的密钥在`C:\users<用户名>\.ssh`中，公钥存放在`id_rsa.pub`里，私钥存放在`id_rsa`里。







在博客根目录下配置git:

```bash
git config user.name "example-name"
git config user.email "example@email.com"
```



建议将引号内的内容替换为准备部署博客的 github 账号用户名和密码，当然，用其他的名称和邮箱替代也可，只是这样每次本地提交 commit 时 github 上会无法查看提交者信息。



### 上传公钥

github头像下拉按钮进入settings，

左侧SSH and GPG keys

点击New SSH key,

名称随便，下方填入公钥id_rsa.pub的内容



测试一下：

`ssh -T git@github.com`

成功会有提示



### 生成Token

Settings->Developer settings->Personal access tokens->

Generate new token

如果这个token只用于发布博客可以设置成永久生效，

否则记得定期更新



select scopes

repo全勾选+workflow,  

一键生成后token上会浮现一串字符，复制存档。





### 建立存放站点根目录的仓库

new repository -> 设置为private->创建



进入git bash ，cd到站点根目录 D:\hugo\example

`git remote add origin git@github.com:<github-username>/<repository-name>.git`



### 将站点根目录下的文件都上传至仓库

`git checkout -b main`





```bash
git add . # 添加目录下全部内容
git commit -m "new blog" # 提交说明，出问题了可以回退到之前的commit
git push -u origin main # 将本地内容推送到远程

```



### 为根目录仓库生成secrets

进入刚建好的仓库->点进它的settings->secrets-> action->new repository secret ,命名为`PERSONAL_TOKEN`,内容为刚才备份的token.



### 建立存放博客目录的仓库

重新建一个repository

必须命名为yourGithubName.github.io格式，设为公开



### 创建Github Action

进入站点根目录仓库，点Actions,->new workflow->set up a workflow yourself.

此时在编辑/.github/workflows/main.yml文件

```yml
name: hugo-blog

on:
  push:
    branches:
      - main  # 博客根目录的默认分支，这里是main，有时也是master
  pull_request:

jobs:
  deploy:
    runs-on: ubuntu-20.04
    concurrency:
      group: ${{ github.workflow }}-${{ github.ref }}
    steps:
      - uses: actions/checkout@v2
#         with:               # 如果你安装主题时用的是git submodule add
#           submodules: true  # 那么这三行不必注释掉，这一行填写 true
#           fetch-depth: 0    # Fetch all history for .GitInfo and .Lastmod

      - name: Setup Hugo
        uses: peaceiris/actions-hugo@v2
        with:
          hugo-version: "latest"  # 使用最新版本hugo
          extended: true          # 如果你使用的不是extended版本的hugo，将true改为false

      - name: Build
        run: hugo --minify

      - name: Deploy
        uses: peaceiris/actions-gh-pages@v3
        if: ${{ github.ref == 'refs/heads/main' }}  # 注意填写main或者master
        with:
          personal_token: ${{ secrets.PERSONAL_TOKEN }} # 如果secret取了其他名称，将PERSONAL_TOKEN替换掉
          external_repository: <username>/<username>.github.io # 填写远程仓库，不一定是这个格式，按照自己的情况写 
          publish_dir: ./public
          #cname: blog.example.com        # 填写你的自定义域名。如果没有用自定义域名，注释掉这行
```



提交commit，如果action变绿则代表运行成功。

如果报错请看提示。



## 后续

等等再写

shortcode加载不出来给我整不会了



### 自定义FeelIt主题（可选）


