---
layout: post
category : nodejs
title: express的诡异错误
tags : [nodejs, express, ejs, compass]
---
{% include JB/setup %}

最近用 `express` 新建项目，发现一个很诡异的问题，出现的简直毫无理由。我的exprss 版本是`4.9.0`

先看 `express` 的参数使用。

	express -h

	Usage: express [options] [dir]

	Options:

	-h, --help          output usage information
	-V, --version       output the version number
	-e, --ejs           add ejs engine support (defaults to jade)
	    --hbs           add handlebars engine support
	-H, --hogan         add hogan.js engine support
	-c, --css <engine>  add stylesheet <engine> support (less|stylus|compass) (defaults to plain css)
	-f, --force         force on non-empty directory

新建项目，用 `ejs` 和 `sass` 。

    express -e -c compass test

	create : test
    create : test/package.json
    create : test/app.js
    create : test/public
    create : test/public/javascripts
    create : test/public/images
    create : test/public/stylesheets
    create : test/public/stylesheets/style.scss
    create : test/routes
    create : test/routes/index.js
    create : test/routes/users.js
    create : test/views
    create : test/views/index.ejs
    create : test/views/error.ejs
    create : test/bin
    create : test/bin/www

    install dependencies:
      $ cd test && npm install

    run the app:
      $ DEBUG=test ./bin/www

`npm install` 后，执行 `DEBUG=test ./bin/www` ,现在还是ok的
	
	 test Express server listening on port 3000 +0ms

然后用浏览器打开 `http://localhost:3000/`, 问题来了：

首先是浏览器报错：

![chrome-err](/images/2014/express_fail/20141203-1.png)

错误信息是：`(failed) net::ERR_CONNECTION_REFUSED` ，其实应该报404错误，因为没有这个样式文件。然后这个时候，控制台也会报错：

	  test Express server listening on port 3000 +0ms
	GET / 200 77.090 ms - 207

	events.js:72
	        throw er; // Unhandled 'error' event
	              ^
	Error: spawn ENOENT
	    at errnoException (child_process.js:1001:11)
	    at Process.ChildProcess._handle.onexit (child_process.js:792:34)

关键是我只是初始化了项目，代码完全没有改过。

其实问题出在 `node-compass` 模块,可以查看github上的用法说明 [node-compass][1]。

>node-compass requires the compass ruby gem in order to compile compass.

首先你的项目要安装ruby， 然后安装 compass。

	$ gem update --system
	$ gem install compass

好吧，其实是小问题，只是没想通。囧~

[1]: https://github.com/nathggns/node-compass






