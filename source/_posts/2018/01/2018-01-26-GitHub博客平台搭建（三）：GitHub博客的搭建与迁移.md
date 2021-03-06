---
title: GitHub博客平台搭建（三）：GitHub博客的搭建与迁移
categories:
  - 多元化技能储备
  - Git&GitHub
tags:
  - GitHub
abbrlink: 30aba86c
date: 2018-01-26 06:00:00
---
【引言】自从年前在GitHub上开了博，偶尔写写文章，不免会遇到换电脑或者重装系统的问题，这时候就得重新配置一遍环境了，当初摸索着搭建起来这么个博客，总得好好维护起来，所以呢，这里也做个搭建和迁移的操作记录，免得以后遗忘吧！[补充]：写完这篇总结后回头复看的时候，深觉有必要补充一篇关于Git的帮助文档，那就索性将Git定为下一篇博客的主题吧！
<div align=center><img src="https://github.com/ttfisher/images/raw/master/public/000013.jpg" width="55%"/></div>
<!-- more -->

# 基础软件准备

## Git
&emsp;&emsp;下载地址：https://git-scm.com/downloads ；下载完成后在本机安装（具体下载什么版本就看电脑操作系统和位数了）；一路Next，安装完成后就会有一个Git Bash的程序，正常运行即表示安装成功了；但是接下来还要完成一步设置：打开Git Bash，完成下面的两个命令录入（因为Git是分布式版本控制系统，所以，每个机器都必须自报家门：你的名字和Email地址。这个步骤也就是报家门的过程，这个我也是看别人的博客才知道这么使用的用意的，具体关于Git的内容，在下一篇文章里面再来详述吧。）
```
// 填写你的github用户名
$ git config --global user.name "your user name" 

// 填写你的github注册邮箱
$ git config --global user.email "your email address" 
```
<img style="clear: both;display: block;margin:auto;" src="https://github.com/ttfisher/images/raw/master/2018/2018-07-27-08.jpg" width="75%">

## Nodejs & npm
&emsp;&emsp;下载地址：https://nodejs.org/zh-cn/download/ ；下载完成后在本机安装（具体下载什么版本就看电脑操作系统和位数了）；一路Next，安装完成后验证一下是否安装成功（因为安装时安装程序默认就会向path追加写入nodejs的路径，所以也不用手工配置path了）
<img style="clear: both;display: block;margin:auto;" src="https://github.com/ttfisher/images/raw/master/2018/2018-07-27-04.jpg" width="75%">

# 账号环境准备
## 注册GitHub账号
> 作为一个能想到玩GitHub的同学，这个步骤还要写的话，就有点侮辱智商了...

## 创建博客仓库
&emsp;&emsp;在GitHub仓库管理页面建一个名为username.github.io的仓库，这里的username就是github登录用户名；比如说，我的github用户名是ttfisher，那么就新建一个名为ttfisher.github.io的仓库，最终生成的博客站点的访问地址就是 http://ttfisher.github.io ；注册完后就有下面图示的绿色区域效果（注：①账号的邮箱验证必须完成；② 务必按照这个命名规则来创建，这方面就不建议自由发挥了； ③ 仓库创建成功不一定会立即生效，如果发现没生效建议稍等片刻）
<img style="clear: both;display: block;margin:auto;" src="https://github.com/ttfisher/images/raw/master/2018/2018-07-27-01.jpg" width="75%">
<img style="clear: both;display: block;margin:auto;" src="https://github.com/ttfisher/images/raw/master/2018/2018-07-27-02.jpg" width="75%">

# SSH Key的配置
&emsp;&emsp;这里的配置，主要还是为了做代码提交免去用户名密码验证的过程，当然也为了安全考虑，所以还是很有必要配置的。

## 生成密钥
&emsp;&emsp;在Git命令窗口运行如下命令，然后3次enter之后即可生成密钥；密钥的存储位置就是图中打印的位置C:\Users\Administrator\.ssh；名为id_rsa的是私钥，名为id_rsa.pub的是公钥
```
ssh-keygen -t rsa -C "Github的注册邮箱地址"
```
<img style="clear: both;display: block;margin:auto;" src="https://github.com/ttfisher/images/raw/master/2018/2018-07-27-06.jpg" width="75%">

## 上传密钥
&emsp;&emsp;密钥配置的位置：用户个人设置里面的SSH and GPG keys菜单；打开之后在右上角有一个添加SSH key的按钮（下面第二个图），点击之后就会出现一个带有Title和Key的配置输入页面，将id_rsa.pub里面的内容原封不动的粘贴到Key区域里面，Title随意取名（便于自己识别），然后点击绿色的Add按钮即可完成密钥的上传（下图的VM-223就是新增的密钥）
<img style="clear: both;display: block;margin:auto;" src="https://github.com/ttfisher/images/raw/master/2018/2018-07-27-07.jpg" width="75%">
<img style="clear: both;display: block;margin:auto;" src="https://github.com/ttfisher/images/raw/master/2018/2018-07-27-05.jpg" width="75%">

## 验证密钥
```
// 直接输入如下命令确认公钥配置是否成功；遇到Are you sure you want to continue connecting (yes/no)?输入yes继续
$ ssh -T git@github.com 
```
<img style="clear: both;display: block;margin:auto;" src="https://github.com/ttfisher/images/raw/master/2018/2018-07-27-08.jpg" width="75%">

# Hexo的安装和配置

## 安装和初验
&emsp;&emsp;Hexo是一个相对来说封装的很好的简单又强大的，支持Github Pages的博客编写和发布工具，最主要的是它是支持Markdown的，而且网上也有很多分享的主题，这对于一个对卖相要求很高的处女座来说，绝对是福音啊！关于Hexo的知识，我这里就不赘述了，有兴趣的找度娘分分钟给你普及了。
&emsp;&emsp;关于不同主题的使用，后面有一节简短的说明，实际操作也很简单，就是下载包--配置_config.yml文件，然后基本就可以了，具体细节的微调就得自己去摸索了。这里有必要说明一下，在每个主题目录里面，也都有一个_config.yml文件，主要用来控制主题的（比如说：显示菜单啊，显示风格啊，图标啊等等；但这个的作用域比外面hexo那个同名配置文件的要小，不能搞混了）
<img style="clear: both;display: block;margin:auto;" src="https://github.com/ttfisher/images/raw/master/2018/2018-07-27-09.jpg" width="75%">

## 环境初始化（新用户）
&emsp;&emsp;对于没有历史存档的新用户，需要先通过如下命令即可完成Hexo环境的初始化，初始化完成后hexo就会将它运行所需要的基础文件都下载到对应的目录（如下图），初始化全部结束后，就可以开始最基础的博客环境配置了。（受限于网络或其他原因，初始化的过程稍稍有一些耗时）
```
Administrator@NRFS2V4EOT1XDKX MINGW64 ~
$ cd C:

Administrator@NRFS2V4EOT1XDKX MINGW64 /c
$ cd GitBlog/

Administrator@NRFS2V4EOT1XDKX MINGW64 /c/GitBlog
$ hexo init
......
```
<img style="clear: both;display: block;margin:auto;" src="https://github.com/ttfisher/images/raw/master/2018/2018-07-27-10.jpg" width="75%">

## 环境恢复（老用户）
&emsp;&emsp;针对已经在GitHub有存档的历史博客环境的，就没有必要安装上面的初始化步骤再一步步配置了，直接从GitHub恢复一套副本到本地就可以开展接下来的工作了
+ 首先，copy仓库源码到本地
```
git clone git@github.com:ttfisher/ttfisher.github.io.git
```
+ 然后到本地下载下来的仓库目录（比如：ttfisher.github.io）下通过Git bash依次执行以下安装和操作命令
```
npm install hexo  （hexo前面已经安装了，这里就可以省略了）
hexo init  （hexo初始化只是针对新用户的，这里也可以省略了）
npm install
hexo-deployer-git
```
<img style="clear: both;display: block;margin:auto;" src="https://github.com/ttfisher/images/raw/master/2018/2018-07-27-11.jpg" width="75%">
+ 可以先在本地启动测试一下，然后发布到GitHub；至此即完成了博客的恢复操作，可以愉快的开始继续接下来的博客之旅了
```
hexo g （生成静态页面）
hexo s （启动本地预览服务）  -- 一般启动完成后通过 http://localhost:4000/ 即可访问
hexo d （发布到GitHub）
```
<img style="clear: both;display: block;margin:auto;" src="https://github.com/ttfisher/images/raw/master/2018/2018-07-27-12.jpg" width="75%">
<img style="clear: both;display: block;margin:auto;" src="https://github.com/ttfisher/images/raw/master/2018/2018-07-27-13.jpg" width="75%">

## 偷懒的用法
&emsp;&emsp;可以安装一个Git小乌龟，通过小乌龟进行git clone的操作（对于不熟悉Git命令的同学来说，绝对是福音）。
<img style="clear: both;display: block;margin:auto;" src="https://github.com/ttfisher/images/raw/master/2018/2018-09-05-01.jpg" width="45%">
<img style="clear: both;display: block;margin:auto;" src="https://github.com/ttfisher/images/raw/master/2018/2018-09-05-02.jpg" width="45%">

# 其他参考信息

## hexo基本配置
&emsp;&emsp; Hexo本身主要的配置文件是_config.yml文件，其中包括整个网站的显示规则，提交内容等等都是要在这里面配置的，下面对其中比较重要的部分进行一个简单的说明
```
-- 注意1：本配置文件的每一项的冒号后面都要留下一个空格，再进行内容填写
-- 注意2：没有的项可以自己添加，保证格式正确即可

# Hexo Configuration
## Docs: https://hexo.io/docs/configuration.html
## Source: https://github.com/hexojs/hexo/

# Site（站点相关的配置）
title: 夏虫不可语冰
subtitle:
description:
author: Tiny Tiny Fisher
language: zh-Hans
timezone:

# URL（permalink：配置最终发布的链接生成的规则，这里用到了crc32这个算法，会算出一个比较短的值，避免URL过长和中文转义）
# [Note]：想要使用这种url压缩规则，需要在本地先安装一个依赖（npm install hexo-abbrlink --save）
## If your site is put in a subdirectory, set url as 'http://yoursite.com/child' and root as '/child/'
url: https://ttfisher.github.io/
root: /
#permalink: :year/:month/:day/:title/
permalink: post/:abbrlink.html
# 算法：crc16(default) and crc32
# 进制：dec(default) and hex
abbrlink:
  alg: crc32  
  rep: hex    
  
permalink_defaults:

# Directory（一些目录名称定义）
source_dir: source
public_dir: public
tag_dir: tags
archive_dir: archives
category_dir: categories
code_dir: downloads/code
i18n_dir: :lang
skip_render:

# Writing（文档的目录结构和文档默认初始化名称，跟hexo new配合）
# File name of new posts
new_post_name: :year/:year-:month-:day-:title.md 
default_layout: post
titlecase: false # Transform title into titlecase
external_link: true # Open external links in new tab
filename_case: 0
render_drafts: false
post_asset_folder: false
relative_link: false
future: true
highlight:
  enable: true
  line_number: true
  auto_detect: false
  tab_replace:

# Date / Time format（日期格式设置）
## Hexo uses Moment.js to parse and display date
## You can customize the date format as defined in
## http://momentjs.com/docs/#/displaying/format/
date_format: YYYY-MM-DD
time_format: HH:mm:ss

# Pagination（分页配置）
## Set per_page to 0 to disable pagination
per_page: 10
pagination_dir: page

# Extensions（主题，需要自己下载主题包）
## Plugins: https://hexo.io/plugins/
## Themes: https://hexo.io/themes/
## theme: yilia
theme: next

# Deployment（部署相关的配置，注意branch的选择必须要对应）
## Docs: https://hexo.io/docs/deployment.html
deploy:
  type: git
  repository: https://github.com/ttfisher/ttfisher.github.io.git
  branch: master

# 在线检索功能（需要自己去algolia注册配置） -- Refer： https://www.npmjs.com/package/hexo-algoliasearch
algolia:
  applicationID: 'XXXX'
  apiKey: 'XXXX'
  adminApiKey: 'XXXX'
  indexName: 'XXXX'
  chunkSize: 5000
```

## 引入新主题的步骤

+ 先下载主题到对应目录
```
主题推荐：https://www.zhihu.com/question/24422335
cd f:/blog
git clone https://github.com/iissnan/hexo-theme-next themes/next
git clone https://github.com/litten/hexo-theme-yilia themes/yilia
```

+ 下载完成后，会在themes目录下生成对应的主题包路径，然后到_config.yml指定你想使用的主题即可（这里配置的主题名称，必须和themes下面的包名称对应上，否则会出现异常错误；默认主题好像是yilia）
```
# Extensions（主题，需要自己下载主题包）
## Plugins: https://hexo.io/plugins/
## Themes: https://hexo.io/themes/
## theme: yilia
theme: next
```

+ 我选用的是next主题，下面附上一些关于这个主题的使用参考（next本身的和algolia的都有很详细的参考文档）
```
http://theme-next.iissnan.com/getting-started.html
http://theme-next.iissnan.com/third-party-services.html#algolia-search
```

## 代码分支的规划
+ 在仓库中创建两个分支：master 与 hexo；并设置hexo为默认分支（这个分支就是我们编辑的文件存档的，发布的分支实际上都是自动生成的，所以不用我们人为管理）
 + hexo分支用来存放网站的原始文件
 + master分支用来存放生成的静态网页
+ 在分支分配好之后，需要修改一下_config.yml中的deploy参数，deploy的默认分支应设置为master，这个也是最终博客发布后资源的分支
+ 依次执行如下命令，即可将本地修改提交到GitHub的仓库中（注：这里提交的是我们编写的未经过生成的原始文件，而不是最终的静态网页）；这里如果不习惯用命令的话，git版小乌龟也是个不错的选择
 + git add .
 + git commit -m “…”
 + git push origin hexo
+ 若是想将本地修改部署到GitHub上，只需要执行hexo generate -d就可以完成。

## 一些常用的命令
```
【Hexo相关】
hexo new fileName -- 新建
hexo g -- g=generate；生成（构建）
hexo s -- s=server；启动服务
hexo d -- d=deplot；发布到github
hexo algolia -- 重建index
hexo new draft "new draft" -- 创建草稿
hexo server --drafts -- 预览草稿模式启动（改配置也可实现render_drafts: true）
hexo publish <filename> -- 将草稿转为正式文章发布

【其他】
npm install XXX -- 安装npm依赖module的（也就是node_modules这个目录）
```

# FAQ

## WARN No layout: index.html?
&emsp;&emsp;运行git clone 指令获得主题后（假设是NEXT主题），在theme主题下保存文件夹的名称为：hexo-theme-next-0.4.0；那么如果在config里设置的是next，就会出现这样的WARN，http://localhost:4000/ 显示的是空白。只要把theme下的文件夹名称改为next就显示正常了。实际原因就是主题的名称配置和实际目录名称要对应。

## Git部分文件无法提交
&emsp;&emsp;先通过操作资源管理器显示出隐藏的目录和文件，然后删除需要提交的目录（比如：next）下的.git，然后通过客户端操作delete(keep local)，再通过客户端进行add做上添加之标记后，再进行commit和push。

## hexo d时卡死
&emsp;&emsp;通过将_config.yml文件中deploy节的提交仓库地址的形式做个修改即可解决此问题，亲测有效。
```
# Deployment
## Docs: https://hexo.io/docs/deployment.html
deploy:
  type: git
  # repository: https://github.com/ttfisher/ttfisher.github.io.git
  repository: ssh://git@github.com/ttfisher/ttfisher.github.io.git
  branch: master
```

## push报ssh.exe找不到
&emsp;&emsp;通过如下Git小乌龟配置可以查找一下GIT_SSH这个环境变量的值，发现和报错的位置是一致的，但是系统里面可能ssh.exe并不是在这个位置；这样就会导致push源文件的时候报ssh.exe not found；此时只需要修改一下注册表项即可解决（HKEY_CURRENT_USER->Software->TortoiseGit）
<img style="clear: both;display: block;margin:auto;" src="https://github.com/ttfisher/images/raw/master/2018/2018-08-17-01.jpg" width="60%">

## push时报permission denied
&emsp;&emsp;通常在push时会出现，比较暴力的方法，就是重新生成ssh key，配置到github管理端；目前还没有发现其他好的解决方法。

## 新装TortoiseGit报错
```
---------------------------
TortoiseGitPlink Fatal Error
---------------------------
Disconnected: No supported authentication methods available (server sent: publickey)
---------------------------
确定   
---------------------------
```
&emsp;&emsp;原因是因为TortoiseGit和Git的冲突，解决方法：将小乌龟的TortoiseGit->settings->Network->SSH Client的值由原来的TortoiseGitPlink.exe的地址改为本地安装的Git下的ssh.exe即可；比如我本机原来是C:\Program Files\TortoiseGit\bin\TortoiseGitPlink.exe，修改后为C:\Program Files\Git\usr\bin\ssh.exe。

## 国内GitHub访问慢的问题
&emsp;&emsp;某次想看看博客资料，差点就因为受不了网络加载的小圈圈一直转着却刷不出页面，都差点转投码云阵营了，后来无意间看到一个分享的方式，一试还挺好用（做人，真的要专一，困难总会有解决的办法的）：
- 打开 http://tool.chinaz.com/dns （这是一个查询域名映射关系的工具）
- 查询 github.global.ssl.fastly.net 和 assets-cdn.github.com 两个地址；多查几次，选择一个稳定，延迟较低的 ip 
- 将刚刚查到的IP地址配置到本地的C:\Windows\System32\drivers\etc\hosts文件中，重新打开浏览器，瞬间腾飞
```
# localhost name resolution is handled within DNS itself.
#	127.0.0.1       localhost
#	::1             localhost

# github
151.101.72.133 assets-cdn.github.com
151.101.109.194 github.global.ssl.fastly.net
```

## 关于博客图片的问题
&emsp;&emsp;为了这个博客图片存储的问题，兜兜转转走过几条路：最开始就是直接把图片放在了hexo博客的source目录里面，相对目录引用很方便，但是这样放总觉得有些怪怪的，于是就天天想着怎么优一下；后来花了些小工夫切到了七牛云，用了有那么三四个月吧，某一天突然收到邮件说测试域名将被回收，本来打算买个域名，不过发现还要一堆繁琐的认证，只得作罢；终于直到有一天发现博客图片都不显示了，就开始着手在网上找图床，试用了几个网站都不甚满意；于是转过头还是在github上开了个专用的图片仓库来存储吧，好在只写写博客图片的量也不大，上传之前先tinyPng压缩一下，访问速度也还可以，如此甚好。

## algolia生成索引报错
```
Lin.C@DESKTOP-DDQ99D1 MINGW64 /e/07-Github/ttfisher.github.io (hexo)
$ hexo algolia
INFO  Start processing
INFO  [Algolia] Identified 132 posts to index.
INFO  [Algolia] Clearing index...
INFO  [Algolia] Index cleared.
INFO  [Algolia] Starting indexation...

E:\07-Github\ttfisher.github.io\node_modules\hexo-algolia\lib\command.js:85
      throw err;
      ^
AlgoliaSearchError: Record at the position 0 objectID=a4e482ef7d11bd9cca40e1b35a11ff30e2608df1 is too big size=130212 bytes. Contact us if you need an extended quota
    at success (E:\07-Github\ttfisher.github.io\node_modules\algoliasearch\src\AlgoliaSearchCore.js:375:32)
    at process._tickCallback (internal/process/next_tick.js:68:7)
```
&emsp;&emsp;很明显的错误提示：too big size；推测基本上是因为文章超长导致索引出现了错误（毕竟用的是人家免费的东西，要求不要太高），鉴于检索也是个锦上添花的功能，检索全文内容也不大实用，所以考虑将内容不放入索引是不是就可以解决问题了呢？于是有了下面的尝试（大约在command.js文件的30行左右）：
```js
	.map(function(data) {
	  // 这是老的代码，删除其中的content，这样在上传到algolia生成索引时就不会上传内容了
	  // var storedPost = _.pick(data, ['title', 'date', 'slug', 'content', 'excerpt', 'permalink']);  
	  var storedPost = _.pick(data, ['title', 'date', 'slug', 'excerpt', 'permalink']); 
```
&emsp;&emsp;经过如此一番改动，重新试着生成了一次索引，完美解决之前的报错，perfect！唯一有点遗憾的是，这个文件是npm管理的，所以每次新装环境之后，又会恢复原来的样子，好在新装环境的时候不多，改动也不是多麻烦。
```
Lin.C@DESKTOP-DDQ99D1 MINGW64 /e/07-Github/ttfisher.github.io (hexo)
$ hexo algolia
INFO  Start processing
INFO  [Algolia] Identified 132 posts to index.
INFO  [Algolia] Clearing index...
INFO  [Algolia] Index cleared.
INFO  [Algolia] Starting indexation...
INFO  [Algolia] Import done.

Lin.C@DESKTOP-DDQ99D1 MINGW64 /e/07-Github/ttfisher.github.io (hexo)
$
```