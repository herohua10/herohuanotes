---
title: AngularJS笔记 
tags: AngularJS,前端框架
grammar_cjkRuby: true
---


## AngularJS目录结构

``` javascript
app/                    --> 项目的源文件
app.css               --> 默认的css文件
  components/           --> 所有应用程序的特定模块
    version/              --> 相关的组件
      version.js                 --> 基本模块的声明
      version_test.js            --> 基本模块的测试
      version-directive.js       --> 用户定义的指令
      version-directive_test.js  --> 用户定义的指令测试
      interpolate-filter.js      --> 用户定义的过滤器
      interpolate-filter_test.js --> 用户定义的过滤器测试
  view1/                --> view1视图模板和控制器
    view1.html            --> 局部模板
    view1.js              --> 控制器
    view1_test.js         --> 控制器的测试
  view2/                --> view2视图模板和控制器
    view2.html            --> 局部模板
    view2.js              --> 控制器
    view2_test.js         --> 控制器的测试
  app.js                --> 主项目模块
  index.html            --> 项目布局模板
  index-async.html      --> 就像index.html，但异步加载JS文件
karma.conf.js         --> 用于运行karma单元测试的配置文件
e2e-tests/            --> 端对端测试
  protractor-conf.js    --> Protractor配置文件
  scenarios.js          --> Protractor端对端测试的运行文件
```