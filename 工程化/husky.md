# [GitHook 工具 —— husky介绍及使用](https://www.cnblogs.com/jiaoshou/p/12222665.html)

## 名称

githooks-Git使用的挂钩。([githook在官网的介绍](https://git-scm.com/docs/githooks))

## 描述

如同其他许多的版本控制系统一样，Git 也具有在特定事件发生之前或之后执行特定脚本代码功能（从概念上类比，就与监听事件、触发器之类的东西类似）。Git Hooks 就是那些在Git执行特定事件（如commit、push、receive等）后触发运行的脚本，挂钩是可以放置在挂钩目录中的程序，可在git执行的某些点触发动作。没有设置可执行位的钩子将被忽略。

默认情况下，`hooks`目录是`$GIT_DIR/hooks`，但是可以通过`core.hooksPath`配置变量来更改（请参见 [git-config [1\]](https://git-scm.com/docs/git-config)）。

## Git Hooks 能做什么

Git Hooks是定制化的脚本程序，所以它实现的功能与相应的git动作相关,如下几个简单例子：
1.多人开发代码语法、规范强制统一
2.commit message 格式化、是否符合某种规范
3.如果有需要，测试用例的检测
4.服务器代码有新的更新的时候通知所有开发成员
5.代码提交后的项目自动打包（git receive之后） 等等...

更多的功能可以按照生产环境的需求写出来