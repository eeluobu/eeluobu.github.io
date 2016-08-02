---
layout:     post
title:      "你好，我的博客"
subtitle:   "\"Hello World, Hello Blog\""
date:       "2016-04-25"
author:     "luobu"
header-img: "img/post.jpg"
catalog:    true
tags:
    - Jekyll
    - 学习
    - DNS
    - 网站
    - 博客
---

> —— 所有，从这里开始


就算是把这几天建这个博客的过程记录一下吧

-------

### 准备工作
不是专业搞前端的，而且对服务器也不是很熟悉，所以没有用wordpress在主机上搭，很感谢有GitHub Pages这个平台，开源项目Jekyll，还有[Hux Blog](https://github.com/Huxpro/huxblog-boilerplate)的模板。

#### 域名和个人主页绑定
为了能让eeluobu.com和www.eeluobu.com这两个域名都能访问我的博客，我在DNS解析处设置了两个`CNAME`，其中一个主机记录是`@`，指向的域名是`www.eeluobu.com`，另一个主机记录是`www`，指向的域名是`eeluobu.github.io`。这样的话在根目录下的CNAME中就要添加一个www.eeluobu.com的域名，才能解析到我的博客上。这样设置有一个问题，那就是我的域名的email等其他服务就用不了了。
我之所以这样在DNS解析处设置两个`CNAME`，而不是设置一个`CNAME`指向`eeluobu.github.io`和一个`A`指向IP`192.30.252.153`，我是在GitHub Pages的[官方文档](https://help.github.com/articles/about-supported-custom-domains/)看到一个说明，内容如下：

> We strongly recommend that you use a www subdomain for these reasons:
> It gives your GitHub Pages site the benefit of our **Content Delivery Network**.
> It is more stable because it is not affected by changes to the IP addresses of GitHub's servers.
> Pages will load significantly faster because Denial of Service attack protection can be implemented more efficiently.

这样就可以使用GitHub提供的CDN加速服务，也就是说访问我的两个域名时，解析到的都是离你最近的镜像主机，加快网站的访问速度，不过我发现ping的时候好像有无CND并没有什么区别，延迟差不多，毕竟都是在国外的服务器，也能理解。而直接使用`A`的话就只能解析到GitHub的IP了，而且如果IP发生变化，网站就访问不了了。
具体配置如下：

| 记录类型 |  主机记录 | 解析线路 | 记录值            |
| :------: | :--------:| :------: | :------------:    |
| CNAME    | @         |  默认    | www.eeluobu.com   |
| CNAME    | WWW       |  默认    | eeluobu.github.io |

相当于把顶级域名`eeluobu.com`解析为次级域名`www`，再把`www`解析到GitHub上的博客地址。这样配置可能会有问题，但好像是我目前发现的能用CDN，而且两个域名都能访问的唯一方法了。按照[GitHub的说法](https://help.github.com/articles/setting-up-an-apex-domain/)，如果你的DNS providers支持`ALIAS`或者`ANAME`，就可以直接用A记录指向域名而不是IP，同样可以使用CDN了。

> Some DNS providers support configuring apex domains with an ALIAS or ANAME record, but there is no industry standard for these. Only `DNS Made Easy` currently supports ANAME records and `DNSimple` is one of the few DNS providers that support ALIAS records.

不过从上面的内容中可以看到，目前只有**DNS Made Easy**和**DNSimple**这两家才支持这两个解析。可惜我用的是阿里云......


#### 安装和配置Jekyll
因为刚开始在网上看到jekyll比较麻烦，要安Ruby、 Python.....一大堆，这些我电脑上都没有，还有要配环境变量，而且在windows上工作又不怎么稳定，生成页面还很慢等等，差点就放弃了，准备用更简单的Hexo。不过自己亲自试了一下，并没有非常麻烦。不过大部分网上的教程都是几年前的，虽然能用，但是软件更新的很快，还是有一些变化。

##### 安装 Ruby
这个在windows下是图形界面，比较简单。唯一要注意的是安装目录不能有`空格`，比如`Program Files`这样的就不行。

##### 安装 DevKit
下载安装包，是一个压缩文件，直接双击解压就行。然后再目录下运行：

``` bash
ruby dk.rb init
ruby dk.rb install
```

##### 安装 Jekyll
因为最新的Ruby已经自带gem了，所以不用我们自己在安装了，直接安装jekyll即可。
首先把源换一下，之前都用的是淘宝的镜像，我开始也是用的这个源，但最后运行`gem install jekyll`的时候总是报错，最后才知道最新的RubyGems 2.6.3 淘宝已经不支持了，现在都由[Ruby China](https://gems.ruby-china.org/)来维护，网址是这个[https://gems.ruby-china.org](https://gems.ruby-china.org/)。不知为什么我用他们的`https`一直连不通，改成`http`就行了，可能是SSL证书问题，但是又没有报错。

``` bash
gem source -r https://rubygems.org/
gem source -a http://gems.ruby-china.org/
gem sources -l
gem install jekyll
```
最后检查一下jekyll是否安装成功

``` bash
jekyll -v
```

#### 在github上建repository
接下来就是比较熟悉的git的操作流程了。

在本地安装key

``` bash
ssh-keygen -t rsa -C "name@email.com"
git config --global user.name "luoyi"
git config --global user.email "name@email.com"
```
测试是否建立成功

``` bash
ssh -T git@github.com
```
将文件添加到仓库  

``` bash
git init
git pull origin master   
git add .
git remote add origin https://github.com/eeluobu/eeluobu.github.io.git
git commit -m "first post"
git push origin master
```

当然我在执行`add`和`push`命令的时候，我的博客上所有的内容都是已经配置和写好了的，这样访问域名就能看到博客界面了。如果要先在本地预览自己的博客，在文件夹目录下执行`jekyll serve`，然后访问`http://127.0.0.1:4000`即可。

### 博客框架

刚开始把这个模板从github上clone下来，解压后我也是一脸懵逼，好多文件和文件夹，花了好些时间才把每个文件的大概内容和功能搞清楚。

#### jekyll目录和原理

官方默认的模板的目录是这样的

```
.
├── _config.yml
├── _drafts
|   ├── begin-with-the-crazy-ideas.textile
|   └── on-simplicity-in-technology.markdown
├── _includes
|   ├── footer.html
|   └── header.html
├── _layouts
|   ├── default.html
|   └── post.html
├── _posts
|   ├── 2007-10-29-why-every-programmer-should-play-nethack.textile
|   └── 2009-04-26-barcamp-boston-4-roundup.textile
├── _site
├── .jekyll-metadata
└── index.html
```
下面是摘要的一些关于它们的介绍：

> _config.yml:
Stores configuration data. Many of these options can be specified from the command line executable but it’s easier to specify them here so you don’t have to remember them.  
> _includes:
These are the partials that can be mixed and matched by your layouts and posts to facilitate reuse.  
> _layouts:
These are the templates that wrap posts. Layouts are chosen on a post-by-post basis in the YAML Front Matter, which is described in the next section. 

#### 模板框架
clone的Hux Blog的模板的文件是下面这个结构，和官方的默认版本比较接近。

下面是具体的目录结构

```
.
├── _config.yml
├── index.html
├── about.html
├── tags.html
├── _includes
|   ├── footer.html
|   ├── head.html
|   └── nav.html
├── _layouts
|   ├── default.html
|   ├── page.html
|   └── post.html
├── _posts
|   ├── 2015-10-29-test1.markdown
|   └── 2016-04-26-test2.markdown
└── css&js&less
```

#### 改造模板
具体改造的内容有下面几个

- 添加目录页

- 把网页的头部标签换成中文

- 添加post页面的目录

然后把文件push到github上，访问[我的域名](http://www.eeluobu.com)就看到现在这个博客的样子了。

### 可以改进的地方

#### 加入评论统计

模板中原本是有评论和网站统计的代码，考虑到博客刚开始建，先一步步的完善就没有加这些功能。

#### 完善about页面

关于界面现在还没想好怎么写，等有好的想法之后加上。

#### 改进目录页面

目录界面用的标题和日期是主页的格式，感觉做目录不怎么好看，本人又不会css，只能将就着用一下，官方jekyll不支持目录的分页，就把所有的目录都放在一个页面了。

### 小结
博客到今天为止大体上算是建好了，接下来就可以开始用了。在这个过程中，中间也走了好多弯路，比如说开始想用jekyll发现网上说配置比较麻烦，就想着去用hexo，发现模板比较少，又回来了；关于选模板也花了一点时间，期间还试图用默认模板自己写，最后发现自己html水平太渣，也放弃了；直到在知乎上发现了这个模板，就直接clone过来用了。再次感谢[Hux Blog](https://github.com/Huxpro/huxblog-boilerplate)，GitHub和Jekyll

关于这个网站，还有好多不足需要后续去完善，继续努力吧！
