---
layout: post
title:  "Jekyll 博客搭建记录"
date:   2016-05-17 12:51:40 +0800
categories: jekyll update
---
现在搭建blog可以用Hexo、Octopress、Ghost，为什么要用Jekyll？据说可定制性很强，试一下。
文档在[这里][jekyll-help]。

- ruby、rbenv

	Mac 自带了 ruby 2.0.0，但后面安装其他东西的时候会要求sudo，所以这里不使用系统的ruby版本。

	rbenv是一个ruby管理工具，方便安装和版本切换，这是它的项目[地址][rbenv]。

		// 安装rbenv
		$ brew update
		$ brew install rbenv

		// 安装 ruby
		$ rbenv install 2.3.0
		$ gem install bundler

		// 创建 Gemfile，编辑
		// source 'https://rubygems.org'
		// gem 'github-pages', group: :jekyll_plugins

		$ bundle install
		$ bundle exec jekyll new . --force

	
[jekyll-help]: https://help.github.com/articles/using-jekyll-as-a-static-site-generator-with-github-pages/
[rbenv]: https://github.com/rbenv/rbenv#readme
