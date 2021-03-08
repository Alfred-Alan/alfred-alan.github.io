---
layout: post
title: '在Ubuntu上使用Nginx代理jekyll'
description: '如何在Ubuntu上使用Nginx代理jekyll静态文件'
categories: [Ubuntu,Docker,Nginx]
tags: [jekyll]
image:  /assets/img/blog/nginx_jekyll.png
# accent_image: /assets/img/blog/RabbitMQ-logo.png
# invert_sidebar: true
related_posts:
  - _posts/2020-04-20-github+jekyll for blog.md
  - _posts/2020-04-27-docker-use.md

---

- Table of Contents
{:toc .large-only}

## 首先更新 apt-get
因为刚购买的服务器可能源还不是最新的
所以我们需要更新一下安装源
```shell
sudo apt-get update
```

## 下载ruby
jekyll 需要ruby环境才能构建 我们先下载ruby
```shell
sudo apt-get install ruby-full build-essential zlib1g-dev
```

## 设置gem安装目录
gem 相当于 vue 的npm python 的pip 
是一个下载包的工具
使用gem要设置一下安装目录
```shell
echo '# Install Ruby Gems to ~/gems' >> ~/.bashrc
echo 'export GEM_HOME="$HOME/gems"' >> ~/.bashrc
echo 'export PATH="$HOME/gems/bin:$PATH"' >> ~/.bashrc
source ~/.bashrc
```

## 安装Jekyll和Bundler：
安装 jekyll 和bundler 
bundler 能够跟踪并安装所需的特定版本的 gem，以此来为 Ruby 项目提供一致的运行环境。

```shell
gem install jekyll bundler
```

## git clone 博客代码

```shell
git clone https://github.com/....git
```

## build 打包
这里使用bundle 来协助jekyll 打包项目
```shell
bundle exec jekyll build
```

## 注意

发现遇到报错
{:.note title="注意错误"}
这时候我遇到了一个问题 
```shell
root@:~/alfred-alan.github.io# bundle exec jekyll build
Configuration file: /root/alfred-alan.github.io/_config.yml
 Theme Config file: /root/alfred-alan.github.io/#jekyll-theme-hydejack/_config.yml
            Source: /root/alfred-alan.github.io
       Destination: /root/alfred-alan.github.io/_site
 Incremental build: disabled. Enable with --incremental
      Generating... 
       Jekyll Feed: Generating feed for posts
bundler: failed to load command: jekyll (/root/gems/bin/jekyll)
Traceback (most recent call last):
	62: from /root/gems/bin/bundle:23:in `<main>'
	61: from /root/gems/bin/bundle:23:in `load'
	60: from /root/gems/gems/bundler-2.2.11/exe/bundle:37:in `<top (required)>'
	59: from /root/gems/gems/bundler-2.2.11/lib/bundler/friendly_errors.rb:130:in `with_friendly_errors'
	58: from /root/gems/gems/bundler-2.2.11/exe/bundle:49:in `block in <top (required)>'
	57: from /root/gems/gems/bundler-2.2.11/lib/bundler/cli.rb:24:in `start'
	56: from /root/gems/gems/bundler-2.2.11/lib/bundler/vendor/thor/lib/thor/base.rb:485:in `start'
	55: from /root/gems/gems/bundler-2.2.11/lib/bundler/cli.rb:30:in `dispatch'
	54: from /root/gems/gems/bundler-2.2.11/lib/bundler/vendor/thor/lib/thor.rb:392:in `dispatch'
	53: from /root/gems/gems/bundler-2.2.11/lib/bundler/vendor/thor/lib/thor/invocation.rb:127:in `invoke_command'
	52: from /root/gems/gems/bundler-2.2.11/lib/bundler/vendor/thor/lib/thor/command.rb:27:in `run'
	51: from /root/gems/gems/bundler-2.2.11/lib/bundler/cli.rb:494:in `exec'
	50: from /root/gems/gems/bundler-2.2.11/lib/bundler/cli/exec.rb:28:in `run'
	49: from /root/gems/gems/bundler-2.2.11/lib/bundler/cli/exec.rb:63:in `kernel_load'
	48: from /root/gems/gems/bundler-2.2.11/lib/bundler/cli/exec.rb:63:in `load'
	47: from /root/gems/bin/jekyll:23:in `<top (required)>'
	46: from /root/gems/bin/jekyll:23:in `load'
	45: from /root/gems/gems/jekyll-4.2.0/exe/jekyll:15:in `<top (required)>'
	44: from /root/gems/gems/mercenary-0.4.0/lib/mercenary.rb:21:in `program'
	43: from /root/gems/gems/mercenary-0.4.0/lib/mercenary/program.rb:44:in `go'
	42: from /root/gems/gems/mercenary-0.4.0/lib/mercenary/command.rb:221:in `execute'
	41: from /root/gems/gems/mercenary-0.4.0/lib/mercenary/command.rb:221:in `each'
	40: from /root/gems/gems/mercenary-0.4.0/lib/mercenary/command.rb:221:in `block in execute'
	39: from /root/gems/gems/jekyll-4.2.0/lib/jekyll/commands/build.rb:18:in `block (2 levels) in init_with_program'
	38: from /root/gems/gems/jekyll-4.2.0/lib/jekyll/command.rb:91:in `process_with_graceful_fail'
	37: from /root/gems/gems/jekyll-4.2.0/lib/jekyll/command.rb:91:in `each'
	36: from /root/gems/gems/jekyll-4.2.0/lib/jekyll/command.rb:91:in `block in process_with_graceful_fail'
	35: from /root/gems/gems/jekyll-4.2.0/lib/jekyll/commands/build.rb:36:in `process'
	34: from /root/gems/gems/jekyll-4.2.0/lib/jekyll/commands/build.rb:65:in `build'
	33: from /root/gems/gems/jekyll-4.2.0/lib/jekyll/command.rb:28:in `process_site'
	32: from /root/gems/gems/jekyll-4.2.0/lib/jekyll/site.rb:79:in `process'
	31: from /root/gems/gems/jekyll-4.2.0/lib/jekyll/site.rb:191:in `generate'
	30: from /root/gems/gems/jekyll-4.2.0/lib/jekyll/site.rb:191:in `each'
	29: from /root/gems/gems/jekyll-4.2.0/lib/jekyll/site.rb:193:in `block in generate'
	28: from /root/gems/gems/jekyll-titles-from-headings-0.5.3/lib/jekyll-titles-from-headings/generator.rb:42:in `generate'
	27: from /root/gems/gems/jekyll-titles-from-headings-0.5.3/lib/jekyll-titles-from-headings/generator.rb:42:in `each'
	26: from /root/gems/gems/jekyll-titles-from-headings-0.5.3/lib/jekyll-titles-from-headings/generator.rb:46:in `block in generate'
	25: from /root/gems/gems/jekyll-titles-from-headings-0.5.3/lib/jekyll-titles-from-headings/generator.rb:71:in `title_for'
	24: from /root/gems/gems/jekyll-titles-from-headings-0.5.3/lib/jekyll-titles-from-headings/generator.rb:81:in `strip_markup'
	23: from /root/gems/gems/jekyll-titles-from-headings-0.5.3/lib/jekyll-titles-from-headings/generator.rb:81:in `reduce'
	22: from /root/gems/gems/jekyll-titles-from-headings-0.5.3/lib/jekyll-titles-from-headings/generator.rb:81:in `each'
	21: from /root/gems/gems/jekyll-titles-from-headings-0.5.3/lib/jekyll-titles-from-headings/generator.rb:82:in `block in strip_markup'
	20: from /root/gems/gems/jekyll-titles-from-headings-0.5.3/lib/jekyll-titles-from-headings/generator.rb:82:in `public_send'
	19: from /root/gems/gems/jekyll-4.2.0/lib/jekyll/filters.rb:19:in `markdownify'
	18: from /root/gems/gems/jekyll-4.2.0/lib/jekyll/converters/markdown.rb:84:in `convert'
	17: from /root/gems/gems/jekyll-4.2.0/lib/jekyll/converters/markdown.rb:15:in `setup'
	16: from /root/gems/gems/jekyll-4.2.0/lib/jekyll/converters/markdown.rb:37:in `get_processor'
	15: from /root/gems/gems/jekyll-4.2.0/lib/jekyll/converters/markdown.rb:37:in `new'
	14: from /root/gems/gems/jekyll-4.2.0/lib/jekyll/converters/markdown/kramdown_parser.rb:89:in `initialize'
	13: from /root/gems/gems/jekyll-4.2.0/lib/jekyll/converters/markdown/kramdown_parser.rb:131:in `load_dependencies'
	12: from /root/gems/gems/jekyll-4.2.0/lib/jekyll/external.rb:57:in `require_with_graceful_fail'
	11: from /root/gems/gems/jekyll-4.2.0/lib/jekyll/external.rb:57:in `each'
	10: from /root/gems/gems/jekyll-4.2.0/lib/jekyll/external.rb:60:in `block in require_with_graceful_fail'
	 9: from /root/gems/gems/jekyll-4.2.0/lib/jekyll/external.rb:60:in `require'
	 8: from /root/gems/gems/kramdown-math-katex-1.0.1/lib/kramdown-math-katex.rb:10:in `<top (required)>'
	 7: from /root/gems/gems/kramdown-math-katex-1.0.1/lib/kramdown-math-katex.rb:10:in `require'
	 6: from /root/gems/gems/kramdown-math-katex-1.0.1/lib/kramdown/converter/math_engine/katex.rb:10:in `<top (required)>'
	 5: from /root/gems/gems/kramdown-math-katex-1.0.1/lib/kramdown/converter/math_engine/katex.rb:10:in `require'
	 4: from /root/gems/gems/katex-0.6.1/lib/katex.rb:4:in `<top (required)>'
	 3: from /root/gems/gems/katex-0.6.1/lib/katex.rb:4:in `require'
	 2: from /root/gems/gems/execjs-2.7.0/lib/execjs.rb:4:in `<top (required)>'
	 1: from /root/gems/gems/execjs-2.7.0/lib/execjs.rb:5:in `<module:ExecJS>'
/root/gems/gems/execjs-2.7.0/lib/execjs/runtimes.rb:58:in `autodetect': Could not find a JavaScript runtime. See https://github.com/rails/execjs for a list of available runtimes. (ExecJS::RuntimeUnavailable)
```

意思是 我们没有ExecJS 环境打包失败



## 安装nodejs
上网查了一下 安装node.js 就能把execjs 附带了 
```shell
sudo apt-get install nodejs
```



打包完成之后可以看见在根目录有一个``_site``的文件夹

可以看到里面是一些静态文件 而且还有``index.html``访问入口

接下来直接使用 nginx指向这个__site 文件夹就能访问了



## 安装docker 
这里我使用docker来启动nginx服务
如果小伙伴不会使用可以看下面的关联帖子

```shell
sudo apt-get install -y docker.io
```
## 拉取Nginx镜像

```
docker pull nginx
```

## 创建目录
创建挂载目录(分别对应日志目录、配置目录、网站文件目录)

```
mkdir -p /home/nginx/log
mkdir -p /home/nginx/config
```

## 启动一个Nginx容器实例

```
docker run --name mynginx -p 80:80 -d nginx
```

## 查看启动的容器

```
docker ps -a
```
## 拷贝配置
先将容器内的配置复制出来

取出默认的配置文件复制到宿主机目录中

```
docker cp 容器名称:/var/log/nginx /home/nginx/log
docker cp 容器名称:/etc/nginx/conf.d /home/nginx/config/conf.d
docker cp 容器名称:/etc/nginx/nginx.conf /home/nginx/nginx.conf
```

## 删除容器
删除刚刚创建的Nginx实例

```
docker rm 容器id/容器名
```

## 修改nginx配置
简单修改一下nginx的配置

```shell
vim /home/nginx/config/conf.d/default.conf
```

```shell
server {
    listen       80;
    listen      443;
    listen  [::]:80;
    listen  [::]:443;
    server_name  localhost;

    #charset koi8-r;
    #access_log  /var/log/nginx/host.access.log  main;

    location / {
        root   /usr/share/nginx/html;
        index  index.html index.htm;
    }

    #error_page  404              /404.html;

    # redirect server error pages to the static page /50x.html
    #
    error_page   500 502 503 504  /50x.html;
    location = /50x.html {
        root   /usr/share/nginx/html;
    }

    # proxy the PHP scripts to Apache listening on 127.0.0.1:80
    #
    #location ~ \.php$ {
    #    proxy_pass   http://127.0.0.1;
    #}

    # pass the PHP scripts to FastCGI server listening on 127.0.0.1:9000
    #
    #location ~ \.php$ {
    #    root           html;
    #    fastcgi_pass   127.0.0.1:9000;
    #    fastcgi_index  index.php;
    #    fastcgi_param  SCRIPT_FILENAME  /scripts$fastcgi_script_name;
    #    include        fastcgi_params;
    #}

    # deny access to .htaccess files, if Apache's document root
    # concurs with nginx's one
    #
    #location ~ /\.ht {
    #    deny  all;
    #}
}

```

## 重新启动nginx容器
重新创建一个拥有挂载目录的，能够自动重启的Nginx容器

```shell
docker run --name nginx -d -p 80:80 -p 443:443 \
--restart always \
-v /home/nginx/log:/var/log/nginx \
-v /home/nginx/config/conf.d:/etc/nginx/conf.d \
-v /home/nginx/nginx.conf:/etc/nginx/nginx.conf \
-v /root/alfred-alan.github.io/_site:/usr/share/nginx/html nginx
```

注意最后一行 ``/root/alfred-alan.github.io/_site``

这个就是刚刚打包好的静态文件地址

然后访问本机ip 就能看到博客的内容了
