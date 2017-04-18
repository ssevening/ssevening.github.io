---
title: GitHub Pages MAC 下搭建个人博客
category: 网站技术
feature_image: "https://ssevening.github.io/assets/android.png"
image: "https://ssevening.github.io/assets/android.png"
---

今天开启第一步，搭一个github pages 个人博客

<!-- more -->


### 1. 第一步参照英文文档直接创建page
 链接地址：[https://pages.github.com/](https://pages.github.com/)
 * Create a repository
 * clone 到本地。
 * 编写 index.html 然后上传到线上即可.

 可以用命令行或 sourceTree 工具进行文件上传或管理。文档介绍很详细。按照做就是了。


### 2. 制作漂亮的页面，我们选：Jekyll

* 安装GEM

		##检查gem版本
		gem -v
		##更新Gem(提示权限)
		gem update --system

如果遇到无法下载的问题，请直接翻墙解决。		

* 安装jekyll
		
		安装jekyll(提示权限)
		$ gem install jekyll
		安装成功之后，查看版本号
		$ jekyll -v

	* 遇到的问题：
	  * 无权限问题:Permission denied - /Library/Ruby/Gems/2.0.0/gems/jekyll-3.4.3/.rubocop.yml，请直接 sudo chmod -R 777 /Library/Ruby/
	  * 还是权限问题：sudo gem install jekyll
ERROR:  While executing gem ... (Errno::EPERM)
    Operation not permitted - /usr/bin/jekyll，可以通过：sudo gem install -n /usr/local/bin/ jekyll 来解决。
    
### 3. 使用Jekyll
* 去找你喜欢的模板吧。网址：[下载地址](http://jekyllthemes.org/)
* 下载其中的Demo，解压后，上传到仓库，然后修改 _posts 中的md文件即可。

### 4. 域名指向
* 申请个域名，添加一个DNS解析记录到 用户名.github.io。
* 在github中 新建CNAME文件名，然后写上你要解析的域名即可。



